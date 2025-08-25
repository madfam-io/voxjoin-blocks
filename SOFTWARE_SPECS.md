# SOFTWARE\_SPEC.md — VoxJoin v1.0

**Status:** Draft for review
**Owners:** Platform (Core Engine), Creator App, Validation/Cloud
**Last updated:** 2025‑08‑25

---

## 1. Purpose & Scope

This document specifies the **software architecture, data models, algorithms, services, and quality gates** for the VoxJoin platform. It operationalizes the three core rules (5 mm voxel grid, 6‑connected pieces, material freedom with digital‑twin/physics gate) and defines interfaces for clients (desktop/web), services, and third‑party extensions.

Non‑goals (covered elsewhere): detailed manufacturing processes, materials procurement, mechanical test rigs (see HARDWARE\_SPEC.md).

---

## 2. Product Goals

* **Create + Validate:** Let users design voxel‑joinery pieces/assemblies and validate manufacturability.
* **Simulate:** Provide first‑order physics (structural, fluid, electrical, magnetic) sufficient to gate designs.
* **Export:** Output clean meshes (STL/3MF) and metadata for fabrication and sharing.
* **Ecosystem:** Support a marketplace and education workflows via open schemas and APIs.
* **Longevity:** Open standard (VJT‑JSON), semantic versioning, migration tools, offline‑first.

Success KPI targets are in the Program Proposal; this spec covers the *how*.

---

## 3. Core Rules (Software Interpretation)

1. **Grid rule:** All geometry is defined on a cubic grid of **U = 5 mm**. All coordinates are integers in units of **U**.
2. **Piece integrity:** A piece is a single **6‑connected** set of voxels (face adjacency only). Edge/corner contact alone is invalid.
3. **Material freedom + physics gate:** Pieces may use any materials/composites **inside the voxel envelope**. Every design has a **digital twin**. A design is labeled **manufacturable** only if it passes: (a) geometry checks, (b) assembly solvability, and (c) process & physics checks. Otherwise it is **prototype‑only** or **digital‑only**.

**Allowed assembly motions:** axis‑aligned translations; optional 90° rotations about grid axes. Elastic‑assisted insertion allowed only when piece or joint is flagged `flex-fit` with limits.

---

## 4. System Architecture

### 4.1 High‑Level Components

* **Core Engine (Rust, compiled to native & WASM):** voxel kernel, geometry ops, clearance engine, solver, physics adapters.
* **Creator App (TypeScript/React + WASM core):** editor UI, authoring tools, presets, instruction generator.
* **Simulator (in‑app module):** runtime visualization of assembly, stresses, flows, circuits, magnet interactions.
* **Exporter:** STL/3MF generation with tolerance offsets; QR/metadata embedding.
* **Validation Service (Cloud, Rust/Actix or Node/Fastify):** headless checks at scale; returns reports.
* **Marketplace & Education APIs:** listing, licensing, class mgmt; see §12.

### 4.2 Data Flow (ASCII)

```
Authoring → VJT Piece/Assembly JSON → Validation (local/WASM) → Physics fast‑checks →
Status {manufacturable | prototype‑only | digital‑only} → Exporter → STL/3MF + QR → Print/Share
                                                     ↘ Marketplace/Education APIs
```

### 4.3 Supported Platforms

* **Desktop:** Windows/macOS/Linux (Electron + WASM).
* **Web:** Modern browsers (WASM, WebGL2). Offline‑first via IndexedDB.

---

## 5. Data Models & Schemas (VJT‑JSON 1.0)

All numeric distances unless noted are in **units of U = 5 mm**. Angles in degrees.

### 5.1 voxjoin‑piece.json

```json
{
  "schema": "voxjoin-piece-1.0",
  "unit_mm": 5,
  "id": "urn:vj:piece:TenonA:v1",
  "name": "Tenon A",
  "voxels": [[x,y,z], ...],
  "flags": { "flex-fit": false },
  "default_material": "PLA_rigid",
  "material_map": {
    "encoding": "rle",
    "runs": [ {"material": "TPU_flex", "from": [x,y,z], "to": [x2,y2,z2]} ]
  },
  "inserts": [
    {"type": "magnet", "shape": "cube", "min_wall": 1, "bbox": {"from": [x,y,z], "to": [x2,y2,z2]}, "encapsulated": true}
  ],
  "tolerance": { "fit_class": "standard", "clearance_mm": 0.30 },
  "safety": { "age_grade": "8+", "warnings": [] },
  "metadata": { "author": "", "license": "CC-BY-NC", "tags": ["tenon"] },
  "content_hash": "sha256-…"
}
```

**Notes**

* `voxels`: integer lattice coordinates; must form one 6‑connected set.
* `material_map`: optional overrides; default storage is **sparse RLE** over axis‑aligned boxes; alternative `sparse_list` allowed.
* `inserts`: virtual volumes fully inside the voxel envelope; used for magnets/weights; **must** be encapsulated by walls ≥ `min_wall` voxels.

### 5.2 voxjoin‑assembly.json

```json
{
  "schema": "voxjoin-assembly-1.0",
  "unit_mm": 5,
  "id": "urn:vj:asm:Demo4:v1",
  "pieces": [
    {"ref": "urn:vj:piece:TenonA:v1", "id": "P1", "pose": {"t": [0,0,0], "r": [0,0,0]}},
    {"ref": "urn:vj:piece:MortiseA:v1", "id": "P2", "pose": {"t": [0,0,0], "r": [0,0,0]}}
  ],
  "allowed_motions": { "translate": true, "rotate90": true },
  "sequence_hint": [
    {"piece": "P1", "motion": {"type": "translate", "axis": "+X", "distance_U": 8}}
  ],
  "fit_policy": { "class": "standard" },
  "validation_profile": "consumer_v1",
  "metadata": { "name": "Demo 4‑Piece", "author": "" }
}
```

### 5.3 printer‑profile.json (excerpt)

```json
{
  "schema": "voxjoin-printer-1.0",
  "process": "FDM",
  "material": "PLA_rigid",
  "nominal_clearance_mm": {"snug": 0.25, "standard": 0.30, "loose": 0.35},
  "shrink_xy_ppm": 0,
  "shrink_z_ppm": 0,
  "min_wall_U": 1,
  "edge_ease_mm": 0.3
}
```

### 5.4 validation‑report.json (excerpt)

```json
{
  "schema": "voxjoin-validation-1.0",
  "status": "manufacturable",
  "checks": [
    {"name": "6-connectivity", "pass": true},
    {"name": "assembly-solvable", "pass": true},
    {"name": "min-wall", "pass": true},
    {"name": "electrical-continuity", "pass": false, "detail": "open circuit between pads A and B"}
  ],
  "recommendations": ["Increase wall to 2U around cavity"],
  "process": "SLA"
}
```

---

## 6. Core Engine (Rust) — Modules & APIs

### 6.1 Modules

* **vox:** voxel grid, sets, morphology (dilate/erode), connectivity.
* **geom:** poses, transforms, swept volumes, collision.
* **join:** interface detection, complementary surface matching.
* **fit:** clearance computation per printer profile.
* **solve:** assembly solver (search over motion space).
* **phys:** adapters for structural, fluid, electrical, magnetic checks.
* **io:** JSON/3MF/STL import/export, QR metadata.
* **validate:** rule gates, reports, linting.

### 6.2 Public WASM API (TypeScript signatures)

```ts
init(wasmBytes: ArrayBuffer): Promise<void>;
loadPiece(json: string): Piece;
loadAssembly(json: string): Assembly;
validatePiece(p: Piece, profile: PrinterProfile): ValidationReport;
validateAssembly(a: Assembly, profile: PrinterProfile): ValidationReport;
solveAssembly(a: Assembly, opts?: SolveOptions): SolveResult;
exportMeshes(a: Assembly, profile: PrinterProfile): Promise<MeshBundle>; // STL/3MF
simulate(a: Assembly, sim: SimConfig): SimResult; // structural/electric/fluids/magnetics
```

---

## 7. Algorithms (deterministic & bounded)

### 7.1 6‑Connectivity Check

* Model as an unweighted graph where nodes = occupied voxels; edges for face adjacency.
* **Method:** BFS/DFS from any voxel; pass if visited count == |voxels|.

**Pseudo‑code**

```python
visited = set([seed])
stack = [seed]
while stack:
  v = stack.pop()
  for n in neighbors6(v):
    if n in voxels and n not in visited:
      visited.add(n); stack.append(n)
return len(visited) == len(voxels)
```

### 7.2 Clearance Computation (Interface Fit)

* Convert mating pieces into voxel occupancy at target resolution; dilate one by `clearance_mm / unit_mm` and test for intersection along mating surfaces.
* **Output:** min/avg clearance, collision flags, hotspots.

### 7.3 Assembly Solver

* **State:** Pose of each piece on the grid.
* **Moves:** `{piece, translate ±X/±Y/±Z by kU}` and optionally `{piece, rotate 90° about axis at integer grid center}`.
* **Search:** A\* or IDA\* over motion primitives; collision checked by swept‑volume occupancy.
* **Goal:** final poses match target; optional minimal move cost.
* **Caps:** piece count ≤ 12 (configurable); move depth ≤ 200 in MVP.

### 7.4 Collision & Swept Volume

* Discretize motion into unit steps; build union of occupied voxels during move; detect intersection with static pieces.

### 7.5 Physics Fast‑Checks

* **Structural:** voxel spring‑lattice approximation; report max strain vs. material limit with safety factor.
* **Electrical:** graph of conductive voxels; continuity & resistance between labeled pads.
* **Fluid:** cavity detection by flood‑fill; wall thickness; simple pressure rating heuristic.
* **Magnetic:** dipole approximations; check unintended attraction/repulsion stability.

---

## 8. Creator App (TS/React)

### 8.1 Key Features

* Voxel editor (paint/add/remove, box fill, mirror, array).
* Material painter (per‑piece default + overrides).
* Joinery presets (mortise/tenon, dovetail, rotational).
* Linting overlay (thin necks, trapped support, min‑wall).
* Validation panel with live status & fix‑its.
* Instruction generator (step‑by‑step assembly).

### 8.2 UX Conventions

* Grid snapping, ghost preview while moving, keyboard nudges (±1U).
* Warnings are actionable with one‑click fixes (e.g., “Increase wall to 2U”).

---

## 9. Simulator

* Real‑time playback of assembly sequence.
* Overlays: stress heatmap, fluid fill path, electrical continuity highlights, magnetic field hints.
* Export: GIF/MP4 of assembly; printable instruction PDF.

---

## 10. Exporter

* Outputs **STL** and **3MF** per piece.
* Applies **tolerance offsets** from printer profile; adds **edge easing**.
* Embeds QR/JSON metadata with `id`, `hash`, and validation status.

---

## 11. Validation Profiles

Profiles bundle rules by audience.

* **consumer\_v1:** no liquids, no exposed magnets, single‑material per piece, piece count ≤ 8, no 1‑voxel necks on load paths.
* **pro\_v1:** allows `flex-fit`, encapsulated magnets, conductive maps, fluids (SLA/MJF only).
* **edu\_v1:** relaxed limits but verbose warnings.

---

## 12. Cloud Services & APIs (REST+JSON)

### 12.1 Validation API

`POST /validate` → {piece|assembly JSON, profile} → `validation‑report.json`
`POST /solve` → assembly JSON → `SolveResult`
`POST /simulate` → assembly + SimConfig → `SimResult`

### 12.2 Marketplace API

`POST /list` (with validation status), `GET /search`, `GET /asset/:id`, license checks.

### 12.3 Education API

Classrooms, assignments, analytics; exports CSV/JSON.

**Auth:** OAuth2; tokens scoped to read/write assets. **Rate limits** and **content hashing** enforced.

---

## 13. Performance Budgets

* Load piece ≤ 100k voxels in < 200 ms (WASM).
* Validate assembly of 8 pieces in < 2 s on mid‑tier laptop.
* Solver: find path for 6‑piece sequential in < 5 s typical, 30 s cap.
* Memory: keep working set < 500 MB for 12‑piece assemblies.

---

## 14. Security, Privacy, Telemetry

* Local files remain local unless user opts into cloud validation/marketplace.
* Telemetry is opt‑in, anonymized, and documented.
* Assets hashed; signatures optional for paid marketplace content.
* Sandboxed WASM; never executes arbitrary code from models.

---

## 15. Accessibility & Internationalization

* UI WCAG 2.1 AA; keyboard‑first operations; color‑blind safe overlays.
* i18n: English & Spanish in v1; string resources externalized; RTL‑ready.

---

## 16. Testing Strategy

* **Unit tests:** vox/geom/solve/phys modules.
* **Property tests:** connectivity invariants, solver reversibility.
* **Golden files:** validation‑report snapshots for fixtures.
* **Fuzzing:** JSON loaders and solver inputs.
* **Perf tests:** solver and exporter benchmarks.

---

## 17. Versioning & Migration

* Semantic versioning for schemas and APIs (`MAJOR.MINOR.PATCH`).
* Deprecations supported for **24 months**; migration CLI maps 1.x → 2.x.
* Files include `schema` URN; loader rejects unknown MAJOR.

---

## 18. Error Handling

* Validation returns actionable errors with codes (e.g., `E_CONN_6`, `E_WALL_MIN`, `E_COLLISION_MOVE`).
* Creator App shows inline fix‑its; always retain user data (no destructive ops without undo).

---

## 19. Acceptance Criteria (Release Gates)

**MVP**

* Create/edit pieces on grid; 6‑connectivity validation; export STL; basic solver; consumer\_v1 profile; docs.

**Beta**

* Physics fast‑checks; instruction generator; 3MF export with QR metadata; Validation API live.

**1.0 Launch**

* Marketplace integration; Education API; migration tool; full i18n ES; accessibility audit pass.

---

## 20. Glossary

* **U:** Base unit = 5 mm.
* **6‑connected:** Face‑adjacent connectivity of voxels.
* **Digital twin:** Material + physics metadata sufficient for checks.
* **Manufacturable:** Passed geometry, assembly, and process/physics checks for a declared route.
* **Prototype‑only/Digital‑only:** Fails some checks; allowed in sim; not for consumer fabrication.
