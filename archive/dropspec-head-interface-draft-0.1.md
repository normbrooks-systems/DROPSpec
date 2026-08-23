# DROPSpec Head Interface Specification

**Draft 0.1 — pre-release working draft**

Part of DROPSpec — the Deposition Retrofit Open Print Standard.

#***THIS DOCUMENT HAS BEEN SUPERCEDED***

---

## 0. Reading This Document

**SHALL** denotes a requirement for conformance. **SHOULD** denotes a strong recommendation. **MAY** denotes a permitted option. **UNDEFINED** denotes territory this revision deliberately makes no claims about — see §1.2, because UNDEFINED is not filler in this document; it is a load-bearing term.

**TBD** marks values not yet finalized. A TBD is a placeholder for a number, not for a decision — the decisions in this draft are settled; the dimensions await measurement and verification.

## 1. Scope and Philosophy

### 1.1 What this specifies

This document defines the mechanical, fluidic, and electrical interface between a DROPSpec-compatible machine and a DROPSpec-compatible print head, using the HP 45-class (TIJ 2.5) cartridge envelope as its foundation.

The specification covers:

- The mechanical envelope, datums, and keep-out zones of a conforming head
- The fluidic interface geometry, including the two umbilical exit zones (Zone H and Zone F) for tank-fed heads
- The electrical contact interface: the TIJ 2.5 pad film (Electrical Personality 1) and the TOOMAH auxiliary interface (§5)
- Interaction requirements with machine service functions (capping, wiping, maintenance access)

### 1.2 What this deliberately does not specify

**The geometry is gospel. The contents are the wild west.**

This specification defines the *shape* of a conforming head — its envelope, datums, contact positions, and fluid exit geometry — as normative and stable. It says nothing about what is inside the shell. Ink chemistry, jetting technology, internal reservoir design, filtration, and drive electronics beyond the contact interface are all outside the scope of this document, permanently and on purpose.

A conforming head is any object that:

1. Fits the envelope and presents the datums (§3),
2. Respects the keep-out zones (§3.4, §6),
3. If it makes electrical contact, does so per a defined electrical personality (§4) or the auxiliary field rules (§5),
4. If it carries an umbilical, exits it per §7.

A factory TIJ 2.5 cartridge conforms. A refilled cartridge conforms. A cartridge filled with TiO₂ white, MICR magnetic ink, or a UV-fluorescent emissive conforms. A future head containing an entirely different jetting technology conforms, provided it honors the geometry and does not touch what it has not negotiated. The specification's job is to guarantee that all of these interoperate with any conforming machine without either party knowing the other in advance.

### 1.3 Provenance

This document mixes three kinds of content, and marks which is which:

- **[HP]** — Documented behavior of the HP 45-class cartridge as established hardware fact, largely via the open reverse-engineering lineage (Yvo de Haas / YTEC HP45 controller work and successors). This specification does not claim authorship of these behaviors; it codifies them as the compatibility baseline.
- **[DROPSpec]** — Conventions original to this specification: the auxiliary pad field, the umbilical exit zone, conformance language, and keep-out definitions.
- **[UNDEFINED]** — Territory reserved from definition in this revision. Future revisions or future electrical personalities may define it. Implementations SHALL NOT assume behavior in undefined territory.

## 2. Conformance Classes

**Class C (Contained)** — Head carries its entire fluid supply within the envelope. The factory TIJ 2.5 cartridge is the reference Class C head. No umbilical. [DROPSpec]

**Class T (Tanked)** — Head is supplied by an external reservoir via a flexible umbilical exiting per §7. [DROPSpec]

A machine claiming DROPSpec compatibility SHALL accept Class C heads in all slots. A machine SHOULD accept Class T heads in all slots; a machine that restricts Class T heads to particular slots SHALL document the restriction prominently. Slot-position asymmetry is considered a defect of the machine design, not of the head.

## 3. Mechanical Envelope and Datums

### 3.1 Envelope

The conforming envelope is the HP 45-class cartridge body. [HP]

| Parameter | Value | Status |
|---|---|---|
| Overall height | TBD mm | [HP] — to be captured from reference hardware |
| Overall width | TBD mm | [HP] |
| Overall depth | TBD mm | [HP] |
| Thumb handle geometry | TBD | [HP] — the handle is a datum landmark for §7 |
| Nozzle plate position and extent | TBD | [HP] |
| Pad film position and extent (rear face) | TBD | [HP] |
| Mass, filled (reference Class C head) | 120–125 g | [HP] — measured, cross-manufacturer tolerance TBD |
| Mass, maximum (any conforming head, excluding umbilical) | TBD g | [DROPSpec] — machine carriage design load derives from this × slot count |

Final envelope dimensions will be published as a measured drawing set derived from reference cartridge hardware, with tolerances reflecting observed variation across manufacturers.

**Mass note (normative once TBD resolves):** A reference filled Class C head is 120–125 g; a four-slot machine therefore carries ~500 g of heads before carriage hardware. The maximum-mass line exists so future heavy heads (tanked instruments, dense fluids) declare themselves against a known ceiling rather than surprising a carriage designed to the reference mass. A head exceeding the maximum does not conform, full stop — the scan dynamics budget is machine property.

### 3.2 Datums

The head SHALL present the following datum features, which the machine's clamping system references [HP, codified as DROPSpec convention]:

- **Datum A** — TBD (primary seating surface)
- **Datum B** — TBD (lateral registration feature)
- **Datum C** — TBD (depth stop)

**Design note (non-normative):** Registration variance has two distinct sources, and the design treats them separately. *Cart-to-cart* variance — nozzle die placement relative to shell datums — is real and requires a printed calibration per cartridge (HP's own consumer printers run an alignment page on new cartridges for exactly this reason). *Insertion-to-insertion* variance of the same cartridge is small when the clamp consistently preloads the shell datums — industrial pen stalls demonstrate this daily, with production floors reseating pens across shifts without per-insertion realignment. Machines therefore store per-cartridge offsets and treat reinsertion as a *verification* event, not a recalibration event (§3.3). Clamp designs SHALL preload the datum features consistently; software owns the per-cartridge absolute offset, mechanics own its repeatability.

### 3.3 Head identification

A conforming Class C head implementing Electrical Personality 1 carries the standard TIJ 2.5 ID/calibration resistor, which the machine reads for per-head energy calibration. [HP]

Machines SHOULD detect head removal/insertion events (via the ID resistor circuit or equivalent) and SHOULD respond to any insertion with a registration verification print. Because the ID resistor bins cartridges rather than serializing them, the machine cannot electronically distinguish the same cartridge reseated from a new cartridge of the same type — the verification print is the arbiter: if it passes, stored per-cartridge offsets stand; if it fails, full calibration runs.

### 3.4 Keep-out zones (head-side obligations)

The following machine-owned volumes SHALL remain clear of any head protrusion, in any conformance class:

- **Capping engagement volume** — the travel path and seated position of the machine's capping gasket against the nozzle plate region. TBD dimensions. [DROPSpec]
- **Wipe access volume** — the frontal clearance required for in-place manual wiping of the nozzle face at the machine's maintenance position. TBD dimensions. [DROPSpec]
- **Adjacent-slot volume** — no feature of a head may extend laterally beyond the envelope width. In an inline gang, a head's side faces are its neighbors' property. [DROPSpec]

## 4. Electrical Personality 1: TIJ 2.5 Pad Film

### 4.1 Status

The TIJ 2.5 flex-film pad array on the rear face, exactly as found on the HP 45-class cartridge, is **Electrical Personality 1** and is the *complete* electrical interface of this revision. [HP]

This specification adds nothing to the film, removes nothing from it, and reinterprets nothing on it. A conforming machine drives it per documented TIJ 2.5 practice; a conforming Class C head presents it per factory convention.

### 4.2 Summary of documented behavior [HP]

Normative electrical detail will be published as an appendix derived from the open HP45 lineage. Summary for orientation:

- 300 nozzles, 600 dpi native pitch, ~30 pL binary drops
- Address/primitive multiplexed firing (not per-nozzle discrete channels)
- Nominal firing voltage ~11 V, with per-head pulse energy trimmed via the ID/calibration resistor
- Aqueous TIJ decap behavior measured in tens of seconds; capping is a machine-side lifecycle requirement, not an option

| Pad map, timing diagrams, resistance ranges | TBD — Appendix A | [HP] |
|---|---|---|

### 4.3 Contact pads are control pads

Throughout this specification the film contacts are termed **control pads**, not firing pads. [DROPSpec] The name is deliberate: the pads are the machine's control interface to *a head*, of which the thermal-resistive TIJ 2.5 implementation is version one. The specification's compatibility promise attaches to the contact geometry and the negotiated personality — not to the assumption that a resistor lives behind every pad. (A negotiated non-thermal personality may requisition a bounded subset of these positions for data and encoder passthrough; see §5.4.)

## 5. The TOOMAH (Auxiliary Interface)

### 5.1 Definition

The auxiliary interface is the **TOOMAH**: a plastic receptacle block standing **5 mm proud** of the rear face, on the same face as the Personality 1 control-pad film, at a position TBD, carrying five blade receptacles. The machine side is a **shuttered blade slot**: five blade contacts behind a passive cover that is opened only by insertion of a conforming TOOMAH. [DROPSpec]

The name is a proper noun, not an acronym. The TOOMAH is, definitionally, a growth on the side of a standard cartridge. It is not a tumor.

| Blade | Function | Engagement |
|---|---|---|
| **G** | Ground reference | Longest — first mate, last break |
| **vL** | Logic supply; powers the head's controller for enumeration before vH exists. Voltage TBD. | Mid-length |
| **D+ / D−** | Data pair carrying the I2C handshake bus (SDA/SCL assignment TBD) | Mid-length |
| **vH** | High-voltage drive rail; de-energized by default, energized only after handshake (§5.3). Envelope TBD. | **Shortest — last mate, first break** |

Blade lengths SHALL be staggered as tabled, so the connector sequences itself: a head is grounded and enumerable before vH can physically connect, and on removal vH breaks mechanically before logic and ground do. Blade engagement additionally provides contact wipe on insertion and vibration-proof retention in service — properties a pad-and-spring interface cannot match on a scanning carriage.

### 5.2 Keying (normative)

The TOOMAH is a mechanical interlock with three independent exclusion layers:

1. **Shutter:** machine-side blades live behind a cover opened only by conforming TOOMAH geometry. A flat-faced Personality 1 head — or a finger, or a dropped fastener — cannot reach any blade, vH in particular, regardless of machine state. The slot is additionally sealed against ink aerosol whenever no TOOMAH-bearing head is seated.
2. **Head side:** the TOOMAH's position and 5 mm proudness SHALL interfere with the reference (TIJ-only) stall envelope, such that a TOOMAH-bearing head cannot seat in a stall lacking the auxiliary slot. A high-voltage head in a thermal-drive stall is not forbidden; it is *unconstructable*.
3. **Machine omission:** a machine MAY omit the auxiliary system entirely; omission mechanically rejects TOOMAH-bearing heads by rule 2, which is the intended behavior.

**Design note (non-normative):** wrong matings are prevented by geometry and shutter, not by rule or firmware — and the 5 mm elevation provides high-voltage creepage and clearance separation from the film plane without this specification naming a distance. Blade insertion force SHALL be accounted for in stall preload so seating datums are not disturbed (§3.2).

### 5.3 Handshake (normative)

vH SHALL NOT be energized until an I2C handshake completes on D+/D−, in which the head enumerates, declares its electrical personality, and states its drive requirements (protocol and parameter set TBD). vH SHALL de-energize on loss of I2C device presence — a backup behavior, since the engagement stagger already breaks vH first on any removal. vL and the handshake are the only electrical activity permitted on an unenumerated TOOMAH.

### 5.4 Film requisition

After successful handshake, a negotiated personality MAY requisition **up to ten** designated Personality 1 film positions for auxiliary data and encoder passthrough (which ten, TBD). This is what makes the film *control pads* rather than TIJ pads (§4.3): function is assigned by negotiated personality. Encoder passthrough exists so a head with onboard drive intelligence can self-time firing against the machine's position truth. Requisition never occurs for Personality 1 heads, which cannot handshake and whose film drive is unchanged and unchangeable.

### 5.5 Personality 1 obligations

A Personality 1 head has no TOOMAH and, by §5.2, cannot contact the auxiliary system; no head-side rule is required beyond conforming to the envelope. A machine SHALL NOT energize auxiliary blades, nor requisition film positions, in the absence of a completed handshake.

## 6. Service Interaction Requirements

A conforming head SHALL tolerate the machine's default service lifecycle [DROPSpec]:

- **Capping** as the idle-default state: the nozzle plate region will be engaged by an elastomer gasket whenever the machine is idle, for indefinite durations.
- **Spitting**: firmware-initiated maintenance ejection into a spittoon. Heads whose contents must not be spat (none are currently contemplated) do not conform.
- **In-place manual wiping**: a human will wipe the nozzle face with a lint-free wipe at the maintenance position without unclamping the head. Head features SHALL NOT obstruct a straight wipe pass across the nozzle face.

## 7. Umbilical Interface (Class T)

### 7.1 Exit zones

A Class T head's umbilical SHALL exit the envelope through one of two conforming zones. Exit from the side faces, bottom region, or rear face is non-conforming: the sides belong to adjacent slots in the gang, the bottom to the nozzle plate, capping gasket, and wipe path, and the rear to the control pads. Both conforming zones are per-slot symmetric — tanks work in any position (§2).

**Zone H — through the handle void.** [HP] The factory convention: HP's own C6119A bulk ink delivery system routes its supply tube through the thumb-handle opening, with the handle loop serving as captive routing. Natural fit for overhead, Bowden-style drape. Heads using Zone H inherit compatibility with factory HP bulk hardware conventions.

**Zone F — front face, below the handle.** [DROPSpec] A defined band immediately below the handle datum. Natural fit for drape toward the machine's maintenance access window, where the operator already works.

| Parameter | Value |
|---|---|
| Zone H clearance envelope through handle void | TBD |
| Zone F band vertical extent (below handle datum) | TBD mm |
| Zone F band lateral extent | TBD mm |
| Maximum perpendicular protrusion before the tube must turn (either zone) | TBD mm |

**Machine-side obligation:** a conforming machine's clamp and carriage SHALL keep the handle void clear of obstruction. This makes Zone H heads — including HP's factory bulk delivery hardware — conforming by construction rather than by accommodation.

**Design note (non-normative):** an exhausted 45-class pen is itself a conforming-envelope 42 mL vessel: a community bulk-feed implementation may use a donor pen as the reservoir, docked in any 45-pattern mount, plumbed to the live head with commodity PTFE Bowden tube and push-fit couplings. The donor pen's internal backpressure regulation may serve the feed system's meniscus-control function natively; this is plausible, not established, and awaits bench verification. Backpressure management — keeping slight negative pressure at the live nozzle plate — is the governing fluidics problem for any Class T implementation, factory or DIY.

### 7.2 Head-side obligations

- **Strain relief is the head's job.** The umbilical SHALL be strain-relieved at the head such that normal drape loads and scan accelerations transmit no force to the fluidic joint. The machine provides routing; the head owns its own tail. [DROPSpec]
- The umbilical and its strain relief SHALL remain clear of the capping engagement volume and the wipe access volume (§3.4) at all times, including under scan-motion tube deflection.
- The protrusion limit (§7.1) exists so the tail cannot act as a lever into the access window envelope during carriage motion.

### 7.3 Advisory (non-normative)

A front-exiting umbilical on a scanning carriage is a continuously flexing umbilical, cycling once per pass for the life of the head. Continuous-flex-rated tubing is strongly recommended; static-rated tubing in this application is a fatigue failure awaiting its cycle count.

## 8. What Conformance Buys You

A head meeting this specification inherits, from any conforming machine: the carriage and clamping system, the service station lifecycle (capping, spitting, wipe access), encoder-synchronized firing infrastructure, per-head calibration, registration compensation, and the raster pipeline. A machine meeting this specification inherits every conforming head — which today means the entire industrial TIJ 2.5 cartridge and ink ecosystem, and tomorrow means whatever anyone chooses to build inside the envelope.

That trade is the point of the document.

## 9. Open Items

- Measured envelope drawing set with cross-manufacturer tolerance study (§3.1)
- Datum feature selection and definition (§3.2)
- Appendix A: Personality 1 electrical detail (pad map, timing, calibration resistor ranges) compiled from the open HP45 lineage with attribution (§4.2)
- The TOOMAH: position on the rear face, receptacle block and blade geometry, blade family selection (commodity, multi-vendor), engagement stagger lengths, shutter mechanism, vH voltage envelope, vL voltage, I2C protocol and parameter set, and the ten designated requisitionable film positions (§5)
- Exit zone dimensions for Zones H and F, and the shared protrusion limit (§7.1); Zone H clearance to be captured by measurement from C6119A factory hardware
- Keep-out volume dimensions pending service station design freeze (§3.4)
- Specification versioning and personality-numbering scheme for future revisions

## 10. License

CERN-OHL-S (hardware definitions) / CC BY-SA 4.0 (this document), per project licensing. Final notice text TBD.

---

*Draft 0.1. Geometry gospel; contents wild west. Comments, measurements, and dissent welcome via the project repository.*
