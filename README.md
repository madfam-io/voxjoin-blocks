# VoxJoin

> **Voxel joinery puzzle system** — Create interlocking, 3D‑printable assemblies that combine LEGO‑like combinatorics, Minecraft‑style voxels, and Japanese joinery—**no glue, no screws**. Designs are **simulation‑gated**: only those that pass physics & manufacturability checks are marked *manufacturable*; others remain *digital‑only* for play in the simulator.

---

## ✨ What is VoxJoin?

VoxJoin is an open, simulation‑first platform for designing, validating, and sharing **voxel joinery** parts and puzzles. It includes:

* A **5 mm voxel grid** rule set and joinery guidelines.
* A **Creator App** (editor + solver + simulator).
* A **Core Engine** (Rust/WASM) for validation, fit, assembly solving, and physics fast‑checks.
* **Schemas (VJT‑JSON)** for portable pieces/assemblies/printer profiles.
* Exporters for **STL/3MF** with embedded metadata.

---

## 🧭 Core rules (frozen v1)

1. **Grid** — All geometry aligns to a **5 mm** cubic grid (unit **U**).
2. **Piece integrity** — Each piece is a single **6‑connected** union of whole voxels (face adjacency only).
3. **Material freedom** — Pieces may use any material(s) **inside the voxel envelope**. Every design ships with a **digital twin** and must pass **geometry + assembly + process/physics** checks to be labeled **manufacturable**; otherwise **prototype‑only** or **digital‑only**.

**Allowed motions:** Axis translations; optional 90° rotations. `flex-fit` (elastic‑assist) only when explicitly flagged.

---

## 📚 Repo structure

```text
/docs/
  PROJECT_SPEC.md               # Vision, principles, roadmap (Project spec)
  SOFTWARE_SPEC.md              # Architecture, schemas, APIs, algorithms
  HARDWARE_SPEC.md              # Dimensions, tolerances, materials, QA
  SCHEMA_REFERENCE/
    voxjoin-piece.schema.json
    voxjoin-assembly.schema.json
    printer-profile.schema.json
    validation-report.schema.json
  VALIDATION_PROFILES/
    consumer_v1.json
    pro_v1.json
    edu_v1.json
  TUTORIALS/
    quickstart.md
    joinery-cookbook.md
/packages/
  core-engine/                  # Rust → WASM library (validation, solver, physics)
  creator-app/                  # TypeScript/React UI (Electron/Web)
  validator-service/            # Headless validation API (cloud)
  exporter/                     # Mesh export & tolerance offsets
  schemas/                      # JSON Schemas as an installable package
/examples/
  calibration-kit/              # Fit couplers and printer calibration
  demo-joints/                  # Starter mortise/tenon, dovetail, rotational
/tools/
  migration-cli/                # Schema migrations & validators
  scripts/                      # Dev scripts (format, lint, release)
```

> Tip: The **docs** are the source of truth. Implementations must match the schemas and acceptance tests.

---

## 🚀 Quickstart

### Prerequisites

* **Rust** (stable) + `wasm-pack`
* **Node.js** (LTS) + **pnpm**
* Git, C/C++ build tools (platform default)

### Clone & build

```bash
git clone https://github.com/<org>/voxjoin.git
cd voxjoin

# Build core engine (native + WASM)
pnpm i
pnpm -w run build

# Run Creator App (web dev server)
pnpm -w run dev
```

Then open the URL shown in your terminal. For desktop builds, see `/packages/creator-app/README.md`.

### Validate & export an example

```bash
# Validate a sample assembly against the consumer profile
pnpm run validate ./examples/demo-joints/assembly.json --profile consumer_v1

# Export meshes with tolerance offsets
pnpm run export ./examples/demo-joints/assembly.json --out ./out
```

Artifacts will include STL/3MF files with embedded QR/JSON metadata.

---

## 🧪 Testing

* **Unit/Property tests:** `pnpm test`
* **Golden validation reports:** `pnpm test:validation`
* **Performance benches (solver/exporter):** `pnpm bench`

---

## 🧩 Design & joinery guidance

See `/docs/joinery-cookbook.md` for minimum sections, fit classes, voxelized dovetails, rotational locks, and supportless patterns. Calibration guidance & printer profiles live in `/examples/calibration-kit` and `/docs/`.

---

## 🔐 Security & privacy

* Local designs stay local unless you opt into cloud validation or marketplace features.
* Vulnerability reports: **security\@yourdomain** (please include steps to reproduce).
* Telemetry is **opt‑in** and anonymized; details in `/docs/security-privacy.md`.

---

## 🧭 Roadmap

Milestones and acceptance gates are in `/docs/PROJECT_SPEC.md`. Highlights: MVP (Creator + validation), Beta (physics checks + instructions), 1.0 (marketplace + education).

---

## 🤝 Contributing

We welcome issues and PRs. Start with:

1. Read `/docs/SOFTWARE_SPEC.md` and `/docs/HARDWARE_SPEC.md`.
2. Follow coding standards (`pnpm run lint` / `cargo fmt`).
3. Ensure tests pass (`pnpm -w test`).

See `/CONTRIBUTING.md` and `/CODE_OF_CONDUCT.md`.

---

## 📝 Licensing

License files live at the repo root. Until finalized, code and content are published under a **temporary evaluation license**; do not redistribute outside the organization. (Replace with final SPDX identifiers at launch.)

---

## 🌿 Sustainability

* Prefer recycled polymers where feasible; publish **kWh/part** estimates.
* Encourage supportless joinery and local re‑print repairs. See `/docs/HARDWARE_SPEC.md`.

---

## 🌎 Español (resumen)

VoxJoin es una plataforma de **ensambles voxel** en malla de 5 mm. Cada pieza es un policubo **6‑conectado** y puede usar **cualquier material** dentro del envolvente voxel. Los diseños se validan con **gemelo digital** y sólo los que aprueban física + manufactura son **fabricables**. Empieza en `/docs/` y usa los ejemplos en `/examples/`.

---

## 🙌 Acknowledgments

Inspired by traditional **kumiki** joinery, modern **voxel** modeling, and the open‑source maker community.
