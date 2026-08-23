# Reference Implementation — Deliverable Register

**The Desktop Digital Press (DDP-01)**
Working draft — v0.2, August 2026

The reference implementation of DROPSpec. This document enumerates what must exist for the machine to be buildable by someone who is not us, and marks what each deliverable blocks or is blocked by.

This register tracks **spec draft 0.3.3** (`dropspec-head-interface-draft-0_3_3.md`). All section citations below refer to that draft.

Status keys: **NOT STARTED** · **IN PROGRESS** · **DRAFT** · **FROZEN**
Dependency keys: **BLOCKED BY** (cannot start) · **GATES** (others wait on this)

---

### Changes from v0.1 (non-normative)

v0.1 was written against spec drafts 0.1–0.2.2 assumptions. The 0.3.3 TOOMAH redefinition and the completed envelope measurement invalidate several entries. This summary does not govern; the entries do.

1. **A0 reclassified IN PROGRESS.** The single-article envelope measurement is complete and published (spec §3.2, reference article C8842A, drawing set `hp45_ab_face_final.svg` / `hp45_profile_final.svg`). What remains is the cross-manufacturer tolerance study and the spec §10 measurement items. A1 is unblocked at nominal dimensions.
2. **A3 rewritten.** Spec §5.2 separates head seating from TOOMAH mating — the contact carrier advances after clamping, so clamp preload no longer fights insertion force. The 0.2.2-era requirement that preload survive blade insertion is superseded by the spec itself. A3 now carries the §3.4 clamp bearing rules instead.
3. **A6 rewritten.** The TOOMAH is now a USB-C receptacle plus four quadrature blades (spec §5.1); the shutter is gone, replaced by a manually removed cap and the USB-C electrical interlock (§5.3). The deliverable and its blockers are restated accordingly.
4. **D6 restated; C4 promoted.** Spec §6 makes the solenoid-actuated wiper the reference mechanism, with manual wiping as the bring-up interim. The wiper module is a v1 deliverable, not a community option.
5. **E4/G6 re-founded on §4.5.** The insertion verification print is RI behavior, not a spec citation — §4.5 makes nozzle condition machine business and forbids requiring registration calibration on insertion.
6. **G7 added.** Spec §10's die-to-datum swap test needs a bench, and the RI is the bench.
7. All spec section numbers updated to the 0.3.3 schedule (datums §3.2 → §3.4, keep-outs §3.4 → §3.6, and so on).
8. **H5 flagged**: `open-tij-printer-proposal.md` is not present in the repository.

---

## 0. Standing Decisions

Decisions already made that every deliverable below inherits. Changing one of these invalidates work downstream, so they are listed separately from the open items in §10.

| # | Decision | Consequence |
|---|---|---|
| D1 | The stall is **designed, not purchased** — printed body, aftermarket contacts | Head assembly becomes a DROPSpec deliverable; mass and structural class reopen |
| D2 | Gang is **staggered on the media axis**, ~half-inch pitch, four pens spanning ~2 in | Mono = four banded pens; CMYK = four passes, order fixed by geometry |
| D3 | Position truth is a **sealed encoder wheel on a gantry raceway** | Firing fires from encoder edges; steppers provide motion only. The TOOMAH's four blades carry this quadrature to heads that negotiate for it (spec §5.1) |
| D4 | Motion is **Klipper, unforked**, extended via klippy extras | Machine defined in `printer.cfg`; Moonraker/Mainsail intact |
| D5 | Capping is **passive, automatic, and the idle default** | No service motor; carriage motion actuates the cap sled |
| D6 | Wiping is a **straight pass with the head clamped** — solenoid-actuated wiper in the reference design, manual during bring-up (spec §6) | The head-side obligation is identical under either regime; the stall must not obstruct the blade's swept volume |

---

## 1. Track A — Head Assembly

The largest change from the original proposal and the highest-leverage track. Everything here is new work; none of it is inherited from a commercial article.

### A0. Measured envelope drawing set — **GATES A1 design freeze**

The single-article measurement is done: spec §3.2 carries the measured dimension schedule for the C8842A reference article, and the drawing set (`hp45_ab_face_final.svg`, `hp45_profile_final.svg`) publishes it. `params/hp45-envelope.scad` can be populated at nominal today.

What remains, per spec §10:

- Cross-manufacturer tolerance study — both articles examined so far are C8842A; mould codes must be compared before any spread is treated as characterising rather than understating
- Maximum-mass figure for a conforming head (§3.2)
- Flip-scan or equivalent to establish the maximum envelope under draft
- Grip lower extent (schedule letters H and I withdrawn) — gates Zone F dimensioning (§7.1)

**Status:** IN PROGRESS — nominal complete, tolerance bands outstanding

### A1. Parametric stall library (OpenSCAD) — **GATES A2–A6**

The core mechanical deliverable. Architecture is a parameter/geometry split so that no dimension is stated twice.

| File | Kind | Import | Provenance | Contents |
|---|---|---|---|---|
| `params/hp45-envelope.scad` | parameter | `include` | [HP] | Measured cartridge dimensions — single source of truth, fed by A0 / spec §3.2 |
| `params/contacts.scad` | parameter | `include` | [sourced] | Aftermarket contact part geometry and pocket dimensions |
| `params/material.scad` | parameter | `include` | [build] | Shrink compensation, clearance fits, material class |
| `params/machine.scad` | parameter | `include` | [build] | Overlap nozzles, staircase arrangement, slot count |
| `lib/stall.scad` | geometry | `use` | [DROPSpec] | Single-pen stall: body, datums, latch interface, keep-outs |
| `lib/latch.scad` | geometry | `use` | [DROPSpec] | Preload mechanism |
| `lib/keepouts.scad` | geometry | `use` | [DROPSpec] | Cap engagement and wipe volumes as subtractable solids — the wipe volume is the wiper blade's swept volume, with a known solenoid stroke (spec §6, §3.6) |
| `lib/gauges.scad` | geometry | `use` | [DROPSpec] | Metrology artifacts (A7) |
| `gang.scad` | top level | — | [DROPSpec] | Arrays stalls per `params/machine.scad` |

**Convention: `include` parameter files, `use` geometry files.** Parameter files carry only assignments and no geometry, so a build variant is a short file that includes the defaults and overrides three lines.

| # | Requirement | Rationale |
|---|---|---|
| A1.1 | Single-pen stall generates standalone | Bench rigs, single-head experiments, Track D bring-up — the single-pen case is first-class, not a degenerate gang of one |
| A1.2 | Slot count, stagger pitch, overlap, and staircase arrangement are parameters | O1–O3 remain open; the library must not need editing when they close |
| A1.3 | Exclusion and keep-out volumes subtracted by construction | Spec §3.5/§3.6 conformance verified by geometry, not by eye |
| A1.4 | `assert()` guards on non-conforming parameter combinations | Negative clearance, overlap exceeding array length, stagger pitch below envelope width without staircase offset, clamp land encroaching on the pad field |
| A1.5 | Shrink compensation applied at datum surfaces only | Global scaling moves the stagger pitch, which is the tolerance that matters — now the spec's own stated rationale (§3.4) |

**Status:** NOT STARTED · **BLOCKED BY:** nothing at nominal — A0's tolerance bands gate the freeze, not the start

### A2. Contact interface

Aftermarket contact part identified, sourced, characterized: contact force, wipe distance, retention method, pocket geometry, service life. Contact-to-pad alignment is a datum problem, not a fastening problem — the pocket references the same datums as the shell seat. Pad field geometry is Stable at spec §4.3; the pads are control pads (§4.4), and the pocket must keep clamping load off them (§3.4).

**Status:** NOT STARTED · **BLOCKED BY:** A1

### A3. Latch and preload mechanism

Consistent datum preload per spec §3.4: X and Y seated against hard stops with preload sufficient to resist scan-motion loads without borrowing friction from the Z centering elements (reference Z force of order 1–1.5 N total). Clamp bearing rules, all from §3.4: bear on the flats, never the corner radii; the X-clamp band is 13.8 to 59.65 from AC — 45.85 mm — with the TOOMAH above it and the no-load zone below; the contact pads receive no clamping load; a seat bears on film or on bare shell, not both. Insertion cycle life is a stated number, not a hope.

TOOMAH mating force is *not* this mechanism's problem: the head seats and clamps first, and the contact carrier advances afterward (spec §5.2). The 0.2.2-era requirement that preload survive blade insertion is superseded by the spec itself.

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

Acceptance criterion is repeatability against a nozzle pitch (~42 µm), not accuracy — spec §3.4 puts stitch registration in the stall gang's as-manufactured geometry, bounded by the shrink-compensation strategy. The as-built residual is captured once by G1 at build time; it is a property of the gang, not of any cartridge, and nothing about it recurs at insertion (§4.5).

**Status:** NOT STARTED · **GATES:** the README and glossary language corrections in §9

### A6. TOOMAH provision

With the stall designed rather than purchased, the auxiliary system can be native to v1 hardware. The head-side block is a USB-C receptacle plus four blade receptacles standing 8 mm proud of AB (spec §5.1); the machine side is this deliverable:

- **Auxiliary slot relief** in the stall body accepting the block at 59.65–74.65 from AC, and the reference-envelope interference geometry that makes §5.3 rule 1 true by construction — a TOOMAH-bearing head cannot seat in a TIJ-only stall
- **Contact carrier** — USB-C plug plus four blade pins on a linear travel parallel to the mating axis (§5.2), advancing only after seat-and-clamp, with a stroke that cannot carry the plug or blade tips past the receptacle entry plane (§5.3 design note — the 8 mm standoff is only a standoff if the carrier honors it)
- **Slot cap**, manually removed — ingress protection, explicitly not an interlock (§5.3)
- **Electrical posture** — VBUS per USB-C source rules: de-energized until sink attach, above-default voltage only under a PD contract with an enumerated head (§5.3 rule 3, §5.4)

**Status:** NOT STARTED · **BLOCKED BY:** spec §10 TOOMAH items — blade family, blade length and carrier alignment budget, quadrature electrical spec, PD profile ceiling and USB device class, receptacle grade for scanning-carriage duty

### A7. Printable metrology gauges

Datum go/no-go, stagger pitch verification, contact alignment check. Cheap to print, and the thing that lets a builder discover a bad print before ink is involved.

**Status:** NOT STARTED

---

## 2. Track B — Motion Platform

### B1. Scan axis structural spec — **GATES B2, B3**

Re-derive from actual head assembly mass. The README's ~2 kg design load and 4080 gantry followed from a steel commercial stall; a printed gang plausibly lands sub-kilo. Deliverable is the load, acceleration, and deflection budget, and the extrusion/rail selection that falls out of it. The spec-side ceiling this must eventually respect — maximum mass of a conforming head — is itself still TBD (spec §3.2, §10).

**Status:** NOT STARTED · **BLOCKED BY:** A4 (mass estimate)

### B2. Gantry and frame

Includes scan travel budget: media width + staircase span + overspray + service station. The staircase arrangement (O2) directly sizes this.

**Status:** NOT STARTED

### B3. Encoder wheel and raceway

Sealed wheel, raceway surface, mounting, preload. Slip is the new failure mode this architecture accepts in exchange for aerosol immunity — characterize it, and define the detection behavior. The quadrature this wheel produces is also what the TOOMAH's four blades carry to heads that ask (spec §5.1), so its electrical specification is co-owned with the spec §10 item.

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

**Status:** NOT STARTED · **GATES:** spec §3.6 keep-out dimensions (spec §10 holds them open pending this design freeze)

### C2. Spittoon

Absorbent fill, capacity, service interval, firmware spit scheduling interface.

**Status:** NOT STARTED

### C3. Maintenance position

Front access window presenting all four nozzle faces at the wipe station without unclamping. Interlock behavior.

**Status:** NOT STARTED

### C4. Wiper module

Solenoid-actuated, spring-loaded silicone blade making a straight pass across the nozzle faces with the heads clamped — the spec's reference mechanism (§6), no longer a community option. The blade is deliberately not a consumable: silicone doesn't absorb, ink wipes off wet, and no part number gets reintroduced into the supply chain here (§6 design note). Deliverable includes the blade swept volume for `lib/keepouts.scad`, since spec §3.6's wipe volume is defined as exactly that. Early DDP-01 prototypes wipe manually while this matures (§6 RI note); the head-side obligation is identical under either regime.

**Status:** NOT STARTED

---

## 4. Track D — Electronics

### D1. Firing board

Deterministic timing MCU, address/primitive multiplexing, boost stage to nominal ~11 V, per-pen energy calibration from the ID resistor, encoder edge input, per-pen firing delay (fixed integer offsets from the staircase geometry).

**Status:** NOT STARTED

### D2. Contact routing PCB

From the aftermarket contact block to the firing board. Replaces the proposal's custom pogo interposer. The presence-probe circuit reads TSR/R10X at positions 27/28; an open is a valid negative — a non-Personality-1 head presents no conductive material at any film position — and must not be treated as a fault (spec §4.5).

**Status:** NOT STARTED · **BLOCKED BY:** A2

### D3. Data path budget

Sustained nozzle data rate versus carriage speed; SPI-from-host versus per-pass double buffering. Bounds honest scan-speed claims.

**Status:** NOT STARTED

### D4. Power and safety

24 V rail, boost stage, door interlock, solvent-ink ventilation posture. TOOMAH slots inherit the USB-C source posture from A6: no VBUS on an unmated slot, no above-default voltage outside a PD contract (spec §5.3, §5.4).

**Status:** NOT STARTED

---

## 5. Track E — Firmware

| # | Deliverable | Notes |
|---|---|---|
| E1 | Firing firmware | Encoder-synced pulse generation, per-pen delay, energy trim |
| E2 | `[tij_printhead]` klippy extras module | Arms firing board, streams per-pass raster, sequences passes |
| E3 | Service choreography | Cap engage/disengage, spit scheduling, wiper solenoid, maintenance position, homing |
| E4 | Head ID / insertion detection | TSR/R10X probe per spec §4.5; open = valid negative, not a fault. May offer the G6 health print on insertion — offer, not require: §4.5 forbids requiring registration calibration on insertion, and nozzle condition is machine business |

---

## 6. Track F — Host Software

| # | Deliverable | Notes |
|---|---|---|
| F1 | CUPS filter chain | Rasterization, universal client compatibility |
| F2 | Halftoning | Error diffusion baseline; depletion masking |
| F3 | Band assignment and stitch feathering | Per-pen band split with overlap-zone feathering; dead-nozzle substitution in overlap |
| F4 | Registration compensation | Applies stored per-slot offsets, both axes — properties of the as-built gang from G1, not per-cartridge state (spec §3.4, §4.5) |
| F5 | Reference ICC profile | Blessed CMYK ink set on reference media |
| F6 | Community profiling toolkit | Deferred |

---

## 7. Track G — Calibration and Verification

The routines that make the mechanical tolerances survivable. Each is both firmware behavior and a printed artifact with a documented reading procedure. G-series offsets are captured at build or service time and stored as gang properties; nothing in this track runs, or is required to run, at cartridge insertion (spec §4.5).

| # | Routine | Measures |
|---|---|---|
| G1 | Per-slot offset calibration | Media-axis and scan-axis position of each slot's array, as built |
| G2 | Bidirectional calibration | Scan-offset error appears at 2× between directions — this is the bidirectional problem on this architecture |
| G3 | Stitch verification | Band boundary quality across the four-pen swath |
| G4 | Media advance calibration | Doubles as Klipper `rotation_distance` |
| G5 | Nozzle health test print | Dead nozzle map feeding F3 substitution |
| G6 | Insertion health print | Fast pass/fail nozzle arbiter, offered on insertion per E4 — a health check, not a registration calibration |
| G7 | Die-to-datum swap test | Spec §10's falsification experiment for §3.4's registration assertion: registration pattern from two cartridges in the same stall, swap, reprint. Disagreement beyond a nozzle pitch falsifies the assertion — and the RI is the only bench that can run it |

---

## 8. Track H — Documentation

| # | Deliverable | Notes |
|---|---|---|
| H1 | Re-costed BOM | The $400–500 band must be re-derived, not repeated. D1 helps it; 4080/rails/base hurt it. TOOMAH adds a USB-C receptacle, carrier, and cap per slot |
| H2 | Build documentation | Assembly, sourcing, first-run |
| H3 | Ink handling and safety guide | Per README safety posture; solvent ventilation |
| H4 | Calibration manual | Reading the G-series artifacts |
| H5 | Disposition of `open-tij-printer-proposal.md` | The file is not in the repository. If it exists privately, retire it or rewrite as an RI design doc; if it is gone, close this item |

---

## 9. Document Corrections Required

Repo documents now out of step, in two groups.

**Stall-provenance corrections** — documents that state the stall is a purchased commercial article:

- `README.md` — "used here as the commercial article it is"
- `glossary-voron-to-press.md` — **Hotend / toolhead**: "bolted on rather than designed"
- `glossary-press-to-voron.md` — **Stall**: describes a steel catalog fixture. The passage about infrastructure that never fails disappearing from vocabulary is worth keeping, reframed as the standard the printed part must meet rather than a description of what it is.

**Spec-adoption corrections** — housekeeping from moving to 0.3.3:

- `README.md` — subtitle reads "Open Print Standard"; 0.3.3 corrects it to "Open Print Specification" (change item 8). Repository Contents still links `dropspec-head-interface-draft-0.1.md`.
- Commit the drawing set (`hp45_ab_face_final.svg`, `hp45_profile_final.svg`) and reference it from spec §3.2 and Appendix A.
- *Resolved by adoption:* the v0.1 flag on the old §3.2 design note — 0.3.3 §3.4 already reframes the registration argument as an untested assertion with §10's swap test (G7 here) as its arbiter.

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
A0 measured envelope ── nominal DONE (spec §3.2) · tolerance bands outstanding
  └─ A1 parametric stall library ── unblocked at nominal
       ├─ A2 contact interface ──> D2 routing PCB ──> D1 firing board
       └─ A4 carrier ──> B1 structural spec ──> B2 gantry
                          └─ A5 material validation ──> stall design freeze
                                                          ▲
                              A0 tolerance bands gate the freeze, not the start
```

Everything else parallelizes. The measurement session that was holding up the entire mechanical track has happened; what holds up the *freeze* is now the cross-manufacturer study and the spec §10 measurement items — a slower burn that no longer blocks A1 from starting.
