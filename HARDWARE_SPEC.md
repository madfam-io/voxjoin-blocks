# HARDWARE\_SPEC.md — VoxJoin v1.0

**Status:** Draft for review
**Owners:** Hardware/DFM, Materials, Compliance
**Last updated:** 2025‑08‑25 (TZ: America/Mexico\_City)

---

## 1. Purpose & Scope

Defines the **physical rules, dimensions, tolerances, materials, manufacturing routes, QA tests, and safety constraints** for VoxJoin parts and kits. This is the authoritative reference for printable production, third‑party manufacturing, and compliance.

**Out of scope:** Software, schemas, and algorithms (see SOFTWARE\_SPEC.md).

---

## 2. Core Principles (frozen v1)

1. **Grid Rule — U = 5 mm:** All external geometry aligns to a cubic grid of **5 mm** voxels (U).
2. **Piece Integrity — 6‑Connected:** Each piece is a single **face‑connected** union of whole voxels (no partial voxels; edge/corner‑only contact is invalid).
3. **Material Freedom — Physics‑Gated:** Pieces may use **any materials or composites** (rigid, flexible, conductive, magnetic, fluid‑carrying, dense/light) **inside the voxel envelope**. Every design ships with a **digital twin**; it is **manufacturable** only if it passes geometry, assembly, and process/physics checks; else **prototype‑only** or **digital‑only**.

**Assembly motions allowed:** Axis‑aligned translations; optional **90° rotations** about grid axes. **Elastic‑assisted fits** allowed only when flagged `flex-fit` with defined limits.

---

## 3. Dimensional System

* **Base unit (U):** 5.00 mm ± 0.02 mm (reference).
* **Coordinate system:** Integer lattice (x,y,z) in U.
* **Voxel cube extents:** \[nU, (n+1)U] along each axis.
* **Datums:** Each piece defines a local origin at its minimum (x,y,z) voxel corner unless otherwise documented.
* **Rounding:** All nominal dims are integer × U; fillets/chamfers are exceptions (see §5.3).

---

## 4. Fits, Tolerances & Edge Treatment

### 4.1 Fit Classes (clearance between mating faces)

| Process                   | Snug C (mm) | Standard C (mm) | Loose C (mm) |
| ------------------------- | ----------: | --------------: | -----------: |
| **FDM – PLA**             |        0.25 |            0.30 |         0.35 |
| **FDM – PETG**            |        0.30 |            0.35 |         0.40 |
| **FDM – ABS/ASA**         |        0.30 |            0.35 |         0.45 |
| **FDM – TPU (as socket)** |        0.15 |            0.20 |         0.25 |
| **SLA/DLP resins**        |        0.10 |            0.15 |         0.20 |
| **MJF/SLS (PA12/PA11)**   |        0.20 |            0.25 |         0.30 |

> Select the **fit class** per printer profile. Default = **Standard**.

### 4.2 Edge Easing

* **Contact edges:** Add **0.2–0.5 mm** fillet or chamfer (off‑grid OK) to reduce wear and insertion forces.
* **Exterior aesthetic edges:** Optional; must not violate voxel envelope.

### 4.3 Dimensional Compensation

* Per‑process **XY/Z scale** compensation allowed at export. Document factors in the print profile attached to the kit.

---

## 5. Minimum Features & DFM Limits

### 5.1 Structural Sections

* **Minimum load path:** ≥ **2×2 voxels** (10×10 mm) cross‑section for pieces expected to carry assembly loads.
* **Thin necks:** 1‑voxel necks are **disallowed** in consumer kits unless flagged `non-load`.

### 5.2 Walls, Pins, Holes (by process)

| Feature                     | FDM            | SLA/DLP         | MJF/SLS              |
| --------------------------- | -------------- | --------------- | -------------------- |
| **Min wall (rigid)**        | **1U (5 mm)**  | **0.6U (3 mm)** | **0.6U (3 mm)**      |
| **Min wall (flex/gaskets)** | 0.6U (3 mm)    | 0.4U (2 mm)     | 0.6U (3 mm)          |
| **Min pin/tenon width**     | **2U (10 mm)** | **1U (5 mm)**   | **1U (5 mm)**        |
| **Min through‑hole Ø**      | 1U (5 mm)      | 0.6U (3 mm)     | 0.6U (3 mm)          |
| **Powder escape (MJF/SLS)** | —              | —               | ≥ **2U** (two holes) |

### 5.3 Overhangs & Bridges (FDM)

* Prefer orientations that avoid internal supports in joints.
* **Bridges:** ≤ 2U (10 mm) unsupported; longer requires arching or ribbing.
* **Overhangs:** ≤ 45° unless supports are fully removable from non‑functional areas.

### 5.4 Surface Texture

* Functional faces should be printed for **lowest roughness** (SLA/MJF preferred). If FDM, orient so layer lines are **perpendicular** to sliding direction when possible.

---

## 6. Joinery Patterns (Voxelized)

All joints must be expressible as unions/subtractions of whole voxels (external envelope) with optional off‑grid edge easing.

### 6.1 Mortise & Tenon (Slider)

* **Tenon:** width ≥ **2U**; shoulder ≥ **1U**.
* **Mortise:** depth ≥ **2U**; relief at ends ≥ **1U** to avoid collision.
* **Locking:** Directional; removal along insertion axis only unless key piece is removed.

### 6.2 Voxel Dovetail (Uni‑axis Retention)

* Stair‑stepped trapezoid on grid; **root width ≥ 2U**, **neck ≥ 1U**; flank rise ≥ **1U per 1U** run.
* Prevents pull‑out except along axis. Use **SLA/MJF** where possible for smoother engagement.

### 6.3 Rotational Lock (Quarter‑Turn)

* Cylindrical approximation on grid: use **octagonal voxel profile**.
* Clearance: add **+0.1 mm** vs. slider tables.
* Add **hard stop** voxel shoulders to signal full engagement.

### 6.4 Sequential Locks (Kumiki‑Style)

* At least one **key** piece must remove first.
* Author must provide sequence; solver must validate **no deadlocks**.

---

## 7. Materials Program

### 7.1 Standard Material Presets (consumer line)

| Code            | Material            | Use                        | Notes                              |
| --------------- | ------------------- | -------------------------- | ---------------------------------- |
| **PLA\_rigid**  | PLA (≥10% recycled) | General rigid              | Low warp, low temp; indoor use     |
| **PETG\_rigid** | PETG                | Rigid with toughness       | Slightly tacky; increase clearance |
| **TPU\_flex**   | TPU 95A             | `flex-fit` hinges, gaskets | Keep sections ≥ 0.6U               |
| **SLA\_tough**  | Tough resin         | Precision joints           | Post‑cure per vendor               |
| **MJF\_PA12**   | PA12                | Durable, fine fits         | Porous; dyeable                    |

> **Conductive** and **magnetic** are **Pro Lab** defaults (see 7.2/7.3). Consumer kits restrict to above.

### 7.2 Conductive Pieces (Pro Lab)

* **Conductive filaments** typically **0.5–30 Ω·cm**; suitable for **signals/logic**, not power.
* Provide **test pads** (≥ **1U×1U** face).
* Continuity spec (kit level): ≤ **100 Ω** end‑to‑end for demo circuits unless otherwise stated.
* For low‑resistance needs, use **encapsulated metal inserts** fully within voxel envelope; min wall per §5.2.

### 7.3 Magnetic Pieces (Pro Lab)

* **Inserts:** Neodymium/ferrite **encapsulated** by ≥ **1U** wall all around.
* **Maximum pull force (consumer):** limit so two pieces cannot pinch skin at > 15 N.
* **Polarity marks:** Embossed **N/S** on a **1U** face area.

### 7.4 Fluid‑Carrying Pieces (Pro Lab)

* **Process:** SLA or MJF only; **FDM not approved** for sealed fluids in consumer kits.
* **Min wall:** SLA **0.6U**, MJF **0.6U**; recommend **2U** where pressurized.
* **Pressure rating:** Default ≤ **50 kPa** unless validated.
* **Clean‑out/vent:** Provide at least one **2U** access or threaded cap (printed thread on grid).
* **Sealing:** Prefer printed **TPU gaskets** in mating piece; adhesives move the design to **prototype‑only**.

### 7.5 Density & Balance

* Declare **mass** per piece (±5%) in BOM.
* **Weighted cores** allowed if encapsulated with ≥ **1U** wall.

### 7.6 Temperature & Environment

* **PLA:** ≤ 50 °C.
* **PETG/PA12:** ≤ 70 °C.
* UV exposure may embrittle; note for outdoor use.

---

## 8. Manufacturing Routes & Default Print Profiles

### 8.1 FDM (consumer baseline)

* **Nozzle:** 0.4 mm (0.6 mm for speed on large parts).
* **Layer height:** 0.20 mm (standard), 0.16 mm (fine).
* **Perimeters:** ≥ 3; **Top/Bottom:** ≥ 4 layers.
* **Infill:** 20–40% grid/gyroid (non‑functional).
* **Orientation:** Layer lines perpendicular to sliding direction where possible.
* **Supports:** Avoid inside joints; if required, use **tree supports** with **0.25–0.30 mm** Z‑gap.

### 8.2 SLA/DLP

* **Layer:** 50–100 µm.
* **Orientation:** 30–45° to minimize suction; **no supports** on mating faces.
* **Post‑cure:** As per resin; rinse thoroughly to avoid tacky joints.

### 8.3 MJF/SLS

* **Wall:** ≥ 0.6U; **Powder escape holes:** ≥ 2U (two or more).
* **Bead blasting:** Light to preserve fits; check clearance post‑finish.

### 8.4 CNC (Advanced)

* Maintain external voxel envelope; **micro‑draft** (≤ 2°) allowed on **internal** features only.
* **Corner radii:** Use fillets that do not protrude outside the voxel envelope.

### 8.5 Multi‑Material

* **Consumer:** Single material per piece.
* **Pro Lab:** MMU allowed with purge towers; or **hybrid** (print shells, pause, place insert, resume) if encapsulation rules are met.

---

## 9. Calibration & QA

### 9.1 Calibration Kit (per printer)

Includes 8 couplers covering **C = 0.10–0.40 mm**. Procedure:

1. Print all couplers with intended material.
2. Test fit; record the **lowest C** that achieves smooth insertion without force.
3. Set printer profile clearance to selected **C**.

### 9.2 Inspection & Acceptance

* **Go/No‑Go Gauges:** Printed gauges for mortise width and tenon width at chosen C.
* **Metrology:** Calipers ±0.02 mm; measure three points per mating face.
* **FAI Report:** Record dims, weight (±5%), visual defects.
* **Wear Test:** 200 insertion cycles; no gross wear or cracking.

### 9.3 Classification Gate

* **Manufacturable:** Meets §3–§8 and passes QA tests.
* **Prototype‑only:** Minor deviations or use of adhesives/unsupported materials.
* **Digital‑only:** Passes geometry but fails physics/process—do not fabricate for consumer use.

---

## 10. Safety & Compliance

* **Age grading:** Consumer line **8+**; Pro Lab **14+**.
* **Magnets:** Encapsulated; comply with **ASTM F963**, **EN 71‑1** small parts/magnet rules.
* **Chemicals:** Resins/filaments must meet regional safety where claimed (e.g., **CPSIA**, **REACH**).
* **Electrical:** Demo circuits ≤ **5 V** and ≤ **500 mA**; no mains interfaces.
* **Liquids:** No hazardous liquids in consumer line; water/air only.

---

## 11. Post‑Processing & Finishing

* **Deburr:** Knife/scrape only on non‑functional faces.
* **SLA:** IPA rinse + full post‑cure; ensure no resin residue in joints.
* **MJF/SLS:** Media tumble (light).
* **Anneal (optional):** PLA anneal may distort—revalidate fits if used.
* **Coloring:** Dyes/paints must not add thickness on mating faces.

---

## 12. Identification & Traceability

* **Embossed ID:** `URN` on a **1U×3U** face strip recessed 0.5 mm.
* **QR Tile:** Optional **2U×2U** flat face reserved; QR encodes `id`, `hash`, and validation status.
* **BOM:** Each kit lists piece IDs, materials, mass, and printer profile.

---

## 13. Packaging & Kitting

* **Flat‑pack** in recyclable media.
* Include **calibration card**, safety sheet, and QR to digital instructions.
* Small parts bag must be **child‑resistant** where required.

---

## 14. Environmental & Sustainability

* Prefer **recycled PLA/PETG** where feasible; declare % recycled.
* **Take‑Back & Regrind:** Offer returns for end‑of‑life parts; separate by polymer.
* **Energy Disclosure:** Publish **kWh/part** estimates for standard profiles.
* **Supportless Design:** Prioritize joints that print without supports.

---

## 15. Test Plans (per kit)

1. **Fit Validation:** Print test coupons; achieve target **C**.
2. **Drop Test:** Assembled model from **1.0 m**, 3 orientations; no functional failure.
3. **Cycle Test:** 200 insert/remove cycles; fit remains within +0.10 mm clearance change.
4. **Magnet Retention (if any):** Pull test ≥ **2×** expected service load; no exposure.
5. **Leak Test (fluid kits):** Hold **30 kPa** for 10 min; no visible loss.
6. **Electrical Continuity:** Circuit demo meets resistance spec.
7. **Thermal Soak:** 40 °C for 2 h; no warping that blocks assembly.

---

## 16. Acceptance Criteria (Ship/No‑Ship)

* **Ship:** All tests in §15 pass; QA acceptance; compliance doc set completed.
* **Hold:** Any failure in magnet/leak/fit/thermal tests; rework design or route.
* **Reject:** Violates core rules or safety constraints.

---

## 17. Appendices

### 17.1 Unit Conversion Quick Reference

* **1U = 5 mm**; **2U = 10 mm**; **3U = 15 mm**; **4U = 20 mm**; **8U = 40 mm**.

### 17.2 Default Printer Profiles (starter)

* **FDM‑PLA:** 0.4 mm nozzle, 0.20 mm layer, 3 perimeters, C = **0.30 mm**.
* **SLA‑Tough:** 50 µm layers, C = **0.15 mm**.
* **MJF‑PA12:** C = **0.25 mm**, escape holes ≥ **2U**.

---

**End of HARDWARE\_SPEC.md v1.0**
