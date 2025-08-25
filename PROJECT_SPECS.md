# VoxJoin System — Scalable Proposal v1.0 (for review)

**Objective**
Create a future‑proof, sustainable, and extensible voxel‑joinery platform—spanning printable kits, a simulation‑first design toolchain, and an ecosystem/standard that others can build upon.

---

## A) Core Principles (frozen v1)

1. **Grid Rule — 5 mm Unit (U):** All geometry aligns to a cubic grid of **5 mm** voxels.
2. **Piece Integrity — 6‑Connected Polycubes:** Each *piece* is a single, **face‑connected** union of whole voxels (no partials).
3. **Material Freedom — Physics‑Gated:** Pieces may use **any material or combination** (rigid, flexible, conductive, magnetic, fluid‑carrying, dense/light) **inside the voxel envelope**. Every design includes a **digital twin** and must pass **assembly + physics + manufacturability** checks to be labeled *manufacturable*; otherwise it is *prototype‑only* or *digital‑only*.

**Assembly Motions Allowed:** axis‑aligned translations; optional 90° rotations. Elastic‑assisted fits permitted only when flagged `flex-fit`.

---

## B) Product System Architecture

**B1. Standards & Data**

* **VES‑5 (Voxel Envelope Standard):** Defines the 5 mm grid, connectivity, join classes, fit classes, and rounding rules.
* **VJT‑JSON Schemas:** `voxjoin-piece.json` and `voxjoin-assembly.json` with **material maps**, **tolerance profiles**, and **digital‑twin physics blocks** (structural, electrical, fluid, magnetic).
* **Versioning & Backcompat:** Semantic versioning (e.g., 1.0.0). Deprecated fields supported for 24 months; migration CLI provided.

**B2. Software Stack (user‑facing)**

* **Creator App (Desktop/Web):** Voxel & material editor, joinery presets, assembly solver, physics fast‑checks, BOM generator, printer profiles.
* **Simulator:** Play/solve assemblies, visualize stresses, electrical continuity, fluids; export animations/instructions.
* **Exporter:** STL/3MF + per‑process offsets; QR metadata embedded in meshes; slice presets for common printers.

**B3. Services & APIs**

* **Validation API:** Design uploads → returns status `manufacturable | prototype‑only | digital‑only` + reasons & fix hints.
* **Market API:** Listing metadata, dependency checks, license enforcement.
* **Education API:** Structured lessons, assessment of builds, classroom rosters.

---

## C) Joinery & Fit (Design Rules)

* **Join Classes:** Mortise‑tenon (voxel), dovetail (voxelized), shoulder locks, rotational pegs, sequential locks.
* **Fit Classes:** `Snug`, `Standard`, `Loose` with process‑specific clearances (per printer profile).
* **Minimum Robust Section:** Any load path ≥ **2×2 voxels**; `lint` fails 1‑voxel necks unless explicitly waivered for *non‑load*.
* **Edge Easing:** Default edge radius 0.2–0.5 mm on contact edges.
* **Cavities & Channels:** Grid‑aligned; min wall defaults: **FDM 1U**, **SLA 0.6U**, **MJF/SLS 0.6U** (overridable by process profile).

---

## D) Materials & Sustainability Program

**D1. Material Taxonomy (Presets)**

* **Rigid:** PLA (recycled), PETG, Nylon, Bio‑based resins.
* **Flexible:** TPU/TPE (shore presets) for `flex-fit` hinges & gaskets.
* **Conductive:** Conductive PLA/TPU (not for low‑resistance power), metal inserts where allowed (encapsulated).
* **Magnetic:** Ferrite/neo inserts **fully encapsulated**, or printable magnetics for Pro.
* **Fluid‑Safe:** SLA/MJF watertight resins/nylon with validated wall/pressure tables.

**D2. Sustainability by Design**

* **Open standard → longevity:** VJT‑JSON + 3MF ensures future tool compatibility.
* **Repairability:** All sets ship with **standardized replacement voxel keys**; broken parts reprintable locally.
* **Take‑Back & Regrind:** Offer prepaid returns for PLA/PETG; certify recycled content in future runs.
* **Energy & Emissions:** Provide **per‑piece print time & kWh** estimates; encourage low‑temp filaments for kits.
* **Minimal supports:** Library of supportless joints; solver flags trapped support risks.
* **Packaging:** Flat‑pack, recyclable, instruction‑lite (QR to digital).

---

## E) Safety & Compliance Guardrails (consumer line)

* **Age Grading:** Core line 8+; Pro Lab 14+.
* **Magnets:** Consumer line = **no exposed magnets**; only **encapsulated** inserts, pull force capped; Pro line allows stronger configurations with warnings.
* **Liquids:** Consumer line excludes liquid‑bearing pieces; Pro Lab permits with explicit ratings.
* **Chemicals:** Restrict to food‑safe/contact‑safe only where claimed; publish MSDS links.
* **Standards Target:** EN 71‑1/2/3, ASTM F963, CPSIA labeling; CE/UKCA where applicable.

---

## F) Ecosystem & Business Model

**F1. SKUs & Offerings**

* **Starter Kits (3 levels):** Foundations (rigid), Motion (adds TPU hinges), Logic (sequential locks).
* **Pro Lab Packs:** Conductive demos, magnetics, fluids (sim‑first).
* **Education Bundles:** Classroom licenses + printable kits + curriculum.
* **Marketplace:** Creator designs (revenue share), verified by Validation API.

**F2. Monetization**

* Hardware kits; **digital plans** (STL/3MF + VJT files); **Creator Pro** subscription (solver, sim, export); **Education** seats; **B2B licensing** (branding/custom sets).

**F3. IP & Licensing**

* **Standard:** VJT schemas under a permissive license to drive adoption.
* **Brand & curated libraries:** Trademarked; premium content under commercial license.
* **Open‑Core Strategy:** Free tool basics; paid advanced physics & manufacturability checks.

---

## G) Roadmap (12 months)

**Q1 — MVP (Weeks 0–12)**

* Lock VES‑5 & VJT‑JSON 1.0.
* Creator App alpha (grid editor, join presets, STL export).
* Calibration kit + three demo joints (rigid slider, TPU flex‑lock, sequential 4‑piece puzzle).
* Validation API: connectivity, fit, manufacturability basics.

**Q2 — Beta (Weeks 13–24)**

* Assembly solver + instruction generator.
* Physics fast‑checks (structural, electric continuity, fluid heuristics).
* Education alpha with 4 lesson plans (tolerances, mechanics, circuits, fluids).
* Pilot with 3 schools + 2 makerspaces.

**Q3 — Launch (Weeks 25–36)**

* Marketplace v1 (creator onboarding, rev share).
* Pro Lab material pack (magnetic, conductive, fluid).
* Certification testing start for consumer kits.

**Q4 — Scale (Weeks 37–52)**

* Manufacturing partnerships (MJF/SLA bureaus).
* Internationalization (ES first); curriculum expansion; take‑back logistics.
* LTS schemas 1.1 with migration tools.

---

## H) Risk Register & Mitigations

1. **Printer variability → poor fits.**
   *Mitigation:* Calibration kit, per‑printer profiles, adjustable clearance in exporter.
2. **Conductive parts underperform.**
   *Mitigation:* Position as **signal/logic** demos; offer metal pad inserts in Pro with full encapsulation.
3. **Leaks in fluid pieces.**
   *Mitigation:* Limit to SLA/MJF; require pressure tests; rating labels.
4. **Magnet safety/compliance.**
   *Mitigation:* Encapsulation, size limits, Pro line gating, compliance lab early.
5. **Solver complexity for big puzzles.**
   *Mitigation:* Piece count caps; authoring lint; benchmark suite.
6. **E‑waste from failed prints.**
   *Mitigation:* Supportless designs; repair libraries; take‑back & regrind.
7. **Standard fragmentation.**
   *Mitigation:* Open standard governance; public test suite; compatibility badges.

---

## I) Success Metrics (12‑month)

* **Adoption:** 10k Creator App downloads; 1k marketplace listings; 100 classroom seats.
* **Quality:** ≥90% first‑fit success after calibration; ≤2% RMA.
* **Sustainability:** ≥30% of kits printed with recycled filament; 20% take‑back participation.
* **Learning outcomes:** Pre/post assessments show ≥25% improvement in tolerance/DFM literacy.

---

## J) Operating Model & Governance

* **Stewardship:** Small standards committee (internal + 2 external advisors) meeting quarterly.
* **Release Cadence:** Minor schema/tool updates monthly; LTS each 6 months.
* **Transparency:** Public changelogs, deprecation timelines, compliance checklists.

---

## K) Immediate Next Actions (2 weeks)

1. Finalize **Core Principles** wording + compliance tests.
2. Build **calibration kit** STLs and print on 3 printer/process combos.
3. Draft **VJT‑JSON 1.0** (piece + assembly + material map).
4. Design **3 demo joints** (rigid slider, TPU flex‑lock, sequential puzzle) and run sim checks.
5. Define **Pro vs Consumer** gating rules; write safety one‑pager.

---

## L) Resumen breve (ES)

* **Tres reglas:** malla 5 mm, piezas 6‑conectadas, libertad de material con **gemelo digital** y **filtro de manufacturabilidad**.
* **Arquitectura:** estándar abierto (VJT), app de creación, simulador, API de validación y mercado.
* **Sustentabilidad:** materiales reciclados, diseño sin soportes, reparación, programa de retorno.
* **Lanzamiento en 12 meses** con MVP en 12 semanas; métricas claras de adopción, calidad y medio ambiente.
