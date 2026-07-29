# Reference Implementation — Deliverable Register

**The Desktop Digital Press**
Working draft — v0.1, July 2026

The reference implementation of DROPSpec. This document enumerates what must exist for the machine to be buildable by someone who is not us, and marks what each deliverable blocks or is blocked by.

Status keys: **NOT STARTED** · **IN PROGRESS** · **DRAFT** · **FROZEN**
Dependency keys: **BLOCKED BY** (cannot start) · **GATES** (others wait on this)

---

## 0. Standing Decisions

Decisions already made that every deliverable below inherits. Changing one of these invalidates work downstream, so they are listed separately from the open items in §10.

| # | Decision | Consequence |
|---|---|---|
| D1 | The stall is **designed, not purchased** — printed body, aftermarket contacts | Head assembly becomes a DROPSpec deliverable; mass and structural class reopen |
| D2 | Gang is **staggered on the media axis**, ~half-inch pitch, four pens spanning ~2 in | Mono = four banded pens; CMYK = four passes, order fixed by geometry |
| D3 | Position truth is a **sealed encoder wheel on a gantry raceway** | Firing fires from encoder edges; steppers provide motion only |
| D4 | Motion is **Klipper, unforked**, extended via klippy extras | Machine defined in `printer.cfg`; Moonraker/Mainsail intact |
| D5 | Capping is **passive, automatic, and the idle default** | No service motor; carriage motion actuates the cap sled |
| D6 | Wiping is **manual, in place**, cartridge never leaves clamp | Stall design must not obstruct a straight wipe pass |

---

## 1. Track A — Head Assembly

The largest change from the original proposal and the highest-leverage track. Everything here is new work; none of it is inherited from a commercial article.

### A1. Parametric stall library (OpenSCAD) — **GATES A2–A6**

The core mechanical deliverable. Architecture is a parameter/geometry split so that no dimension is stated twice.

| File | Kind | Import | Provenance | Contents |
|---|---|---|---|---|
| `params/hp45-envelope.scad` | parameter | `include` | [HP] | Measured cartridge dimensions — single source of truth, fed by A0 |
| `params/contacts.scad` | parameter | `include` | [sourced] | Aftermarket contact part geometry and pocket dimensions |
| `params/material.scad` | parameter | `include` | [build] | Shrink compensation, clearance fits, material class |
| `params/machine.scad` | parameter | `include` | [build] | Overlap nozzles, staircase arrangement, slot count |
| `lib/stall.scad` | geometry | `use` | [DROPSpec] | Single-pen stall: body, datums, latch interface, keep-outs |
| `lib/latch.scad` | geometry | `use` | [DROPSpec] | Preload mechanism |
| `lib/keepouts.scad` | geometry | `use` | [DROPSpec] | Cap engagement and wipe access volumes as subtractable solids |
| `lib/gauges.scad` | geometry | `use` | [DROPSpec] | Metrology artifacts (A7) |
| `gang.scad` | top level | — | [DROPSpec] | Arrays stalls per `params/machine.scad` |

**Convention: `include` parameter files, `use` geometry files.** Parameter files carry only assignments and no geometry, so a build variant is a short file that includes the defaults and overrides three lines.

| # | Requirement | Rationale |
|---|---|---|
| A1.1 | Single-pen stall generates standalone | Bench rigs, single-head experiments, Track D bring-up — the single-pen case is first-class, not a degenerate gang of one |
| A1.2 | Slot count, stagger pitch, overlap, and staircase arrangement are parameters | O1–O3 remain open; the library must not need editing when they close |
| A1.3 | Keep-out volumes subtracted by construction | Spec §3.4 conformance verified by geometry, not by eye |
| A1.4 | `assert()` guards on non-conforming parameter combinations | Negative clearance, overlap exceeding array length, stagger pitch below envelope width without staircase offset |
| A1.5 | Shrink compensation applied at datum surfaces only | Global scaling moves the stagger pitch, which is the tolerance that matters |

**Status:** NOT STARTED · **BLOCKED BY:** A0 (measured envelope)

### A0. Measured envelope drawing set — **GATES A1, and spec §3.1**

Reference cartridge hardware measured; cross-manufacturer tolerance study. Feeds `params/hp45-envelope.scad` directly and resolves the spec's §3.1 TBD table. This is the true head of the critical path.

**Status:** NOT STARTED

### A2. Contact interface

Aftermarket contact part identified, sourced, characterized: contact force, wipe distance, retention method, pocket geometry, service life. Contact-to-pad alignment is a datum problem, not a fastening problem — the pocket references the same datums as the shell seat.

**Status:** NOT STARTED · **BLOCKED BY:** A1

### A3. Latch and preload mechanism

Consistent datum preload per spec §3.2, without disturbing seating when the TOOMAH blade insertion force is present (spec §5.2 design note). Insertion cycle life is a stated number, not a hope.

**Status:** NOT STARTED

### A4. Carrier / backbone

Ties four stalls into one coordinate system. **Open decision:** monolithic four-slot body versus discrete stalls on a common plate (see §10, O1). If a metal backbone is used for thermal stability, this is where it lives.

**Status:** NOT STARTED · **BLOCKED BY:** O1

### A5. Material and thermal validation

The deliverable that replaces the argument the spec used to borrow from steel fixtures. Required data:

- Creep under sustained latch preload, per candidate material, at elevated ambient
- Datum-face wear across insertion cycles
- Coefficient of thermal expansion effect on stagger pitch across the working ambient range, measured not calculated
- Layer-orientation anisotropy at datum surfaces

Acceptance criterion is repeatability against a nozzle pitch (~42 µm), not accuracy — per spec §3.2, software owns the absolute offset.

**Status:** NOT STARTED · **GATES:** the README and glossary language corrections in §9

### A6. TOOMAH provision

With the stall designed rather than purchased, the shuttered blade slot can be native to v1 hardware. Deliverable is the machine-side receptacle, shutter, and the reference envelope interference geometry that makes rule 2 of spec §5.2 true by construction.

**Status:** NOT STARTED · **BLOCKED BY:** spec §5 TBDs (blade family, position, stagger lengths)

### A7. Printable metrology gauges

Datum go/no-go, stagger pitch verification, contact alignment check. Cheap to print, and the thing that lets a builder discover a bad print before ink is involved.

**Status:** NOT STARTED

---

## 2. Track B — Motion Platform

### B1. Scan axis structural spec — **GATES B2, B3**

Re-derive from actual head assembly mass. The README's ~2 kg design load and 4080 gantry followed from a steel commercial stall; a printed gang plausibly lands sub-kilo. Deliverable is the load, acceleration, and deflection budget, and the extrusion/rail selection that falls out of it.

**Status:** NOT STARTED · **BLOCKED BY:** A4 (mass estimate)

### B2. Gantry and frame

Includes scan travel budget: media width + staircase span + overspray + service station. The staircase arrangement (O2) directly sizes this.

**Status:** NOT STARTED

### B3. Encoder wheel and raceway

Sealed wheel, raceway surface, mounting, preload. Slip is the new failure mode this architecture accepts in exchange for aerosol immunity — characterize it, and define the detection behavior.

**Status:** NOT STARTED

### B4. Media transport

Dual-pinch grit metering, star-wheel downstream set, vacuum print zone, mesh web loop ganged to the metering shaft. Advance repeatability target is set by stitch tolerance; note the steady-state advance is now ~2 in per pass, so the tolerance is a *fraction*, not a distance.

**Status:** NOT STARTED

### B5. Vacuum system

Platen, permeable web, plenum, fan selection with measured static pressure margin, waste tray and filter, cleaning station on the return run.

**Status:** NOT STARTED

---

## 3. Track C — Service Station

### C1. Passive cap sled

Ramp-and-latch engaging four caps from carriage motion alone. Idle means capped, always. Per spec §6 this is a head-facing obligation, so its engagement volume must be in `lib/keepouts.scad`.

**Status:** NOT STARTED · **GATES:** spec §3.4 keep-out dimensions

### C2. Spittoon

Absorbent fill, capacity, service interval, firmware spit scheduling interface.

**Status:** NOT STARTED

### C3. Maintenance position

Front access window presenting all four nozzle faces for a straight wipe pass without unclamping. Interlock behavior.

**Status:** NOT STARTED

### C4. Automated wiper module *(optional, community track)*

Documented mounting interface, not a v1 dependency.

**Status:** NOT STARTED

---

## 4. Track D — Electronics

### D1. Firing board

Deterministic timing MCU, address/primitive multiplexing, boost stage to nominal ~11 V, per-pen energy calibration from the ID resistor, encoder edge input, per-pen firing delay (fixed integer offsets from the staircase geometry).

**Status:** NOT STARTED

### D2. Contact routing PCB

From the aftermarket contact block to the firing board. Replaces the proposal's custom pogo interposer.

**Status:** NOT STARTED · **BLOCKED BY:** A2

### D3. Data path budget

Sustained nozzle data rate versus carriage speed; SPI-from-host versus per-pass double buffering. Bounds honest scan-speed claims.

**Status:** NOT STARTED

### D4. Power and safety

24 V rail, boost stage, door interlock, solvent-ink ventilation posture.

**Status:** NOT STARTED

---

## 5. Track E — Firmware

| # | Deliverable | Notes |
|---|---|---|
| E1 | Firing firmware | Encoder-synced pulse generation, per-pen delay, energy trim |
| E2 | `[tij_printhead]` klippy extras module | Arms firing board, streams per-pass raster, sequences passes |
| E3 | Service choreography | Cap engage/disengage, spit scheduling, maintenance position, homing |
| E4 | Head ID / insertion detection | Triggers verification print per spec §3.3 |

---

## 6. Track F — Host Software

| # | Deliverable | Notes |
|---|---|---|
| F1 | CUPS filter chain | Rasterization, universal client compatibility |
| F2 | Halftoning | Error diffusion baseline; depletion masking |
| F3 | Band assignment and stitch feathering | Per-pen band split with overlap-zone feathering; dead-nozzle substitution in overlap |
| F4 | Registration compensation | Applies stored per-pen offsets, both axes |
| F5 | Reference ICC profile | Blessed CMYK ink set on reference media |
| F6 | Community profiling toolkit | Deferred |

---

## 7. Track G — Calibration and Verification

The routines that make the mechanical tolerances survivable. Each is both firmware behavior and a printed artifact with a documented reading procedure.

| # | Routine | Measures |
|---|---|---|
| G1 | Per-pen offset calibration | Media-axis and scan-axis position of each pen's array |
| G2 | Bidirectional calibration | Scan-offset error appears at 2× between directions — this is the bidirectional problem on this architecture |
| G3 | Stitch verification | Band boundary quality across the four-pen swath |
| G4 | Media advance calibration | Doubles as Klipper `rotation_distance` |
| G5 | Nozzle health test print | Dead nozzle map feeding F3 substitution |
| G6 | Insertion verification print | Fast pass/fail arbiter per spec §3.3 |

---

## 8. Track H — Documentation

| # | Deliverable | Notes |
|---|---|---|
| H1 | Re-costed BOM | The $400–500 band must be re-derived, not repeated. D1 helps it; 4080/rails/base hurt it |
| H2 | Build documentation | Assembly, sourcing, first-run |
| H3 | Ink handling and safety guide | Per README safety posture; solvent ventilation |
| H4 | Calibration manual | Reading the G-series artifacts |
| H5 | Disposition of `open-tij-printer-proposal.md` | Retire, or rewrite as RI design doc — it currently contradicts the README in several places |

---

## 9. Document Corrections Required

Existing repo documents that state the stall is a purchased commercial article and are now wrong:

- `README.md` — "used here as the commercial article it is"
- `glossary-voron-to-press.md` — **Hotend / toolhead**: "bolted on rather than designed"
- `glossary-press-to-voron.md` — **Stall**: describes a steel catalog fixture. The passage about infrastructure that never fails disappearing from vocabulary is worth keeping, reframed as the standard the printed part must meet rather than a description of what it is.
- `dropspec-head-interface-draft-0.1.md` §3.2 design note — the industrial-pen-stall repeatability argument is now something the RI must demonstrate (A5) rather than inherit.

---

## 10. Open Decisions Gating Work

| # | Decision | Gates |
|---|---|---|
| O1 | Monolithic four-slot body vs. discrete stalls on a common plate | A1, A4 — pitch tolerance stack vs. serviceability and print volume |
| O2 | Staircase arrangement: inline four vs. two-by-two | A1, B2 — scan travel and dead travel vs. carriage depth and per-pen delay |
| O3 | Stitch overlap, in nozzles | A1, F3 — sets stagger pitch; recommend non-zero |
| O4 | Stall material class | A5 — thermal stability vs. printability |
| O5 | Metal backbone: yes/no/where | A4, A5, B1 |
| O6 | Maximum media width; roll support in v1 | B2, B4 |

---

## 11. Critical Path

```
A0 measured envelope
  └─ A1 parametric stall library
       ├─ A2 contact interface ──> D2 routing PCB ──> D1 firing board
       └─ A4 carrier ──> B1 structural spec ──> B2 gantry
                          └─ A5 material validation ──> stall design freeze
```

Everything else parallelizes. A0 is a measurement session with reference hardware and a set of calipers, and it is holding up the entire mechanical track.
