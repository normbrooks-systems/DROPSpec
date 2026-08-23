# DROPSpec Head Interface Specification

**Draft 0.2 — pre-release working draft**

Part of DROPSpec — the Deposition Retrofit Open Print Standard.

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

- **[HP]** — Documented behavior of the HP 45-class cartridge as established hardware fact, drawn from HP's own patent disclosures (principally US 6,332,677 B1) and from the open reverse-engineering lineage (Yvo de Haas / YTEC HP45 controller work and successors). This specification does not claim authorship of these behaviors; it codifies them as the compatibility baseline. Patent disclosures describe a preferred embodiment rather than shipping hardware, and are marked [HP] on the basis that the disclosed architecture matches the cartridge in production.
- **[DROPSpec]** — Conventions original to this specification: the auxiliary interface, the umbilical exit zone, conformance language, and keep-out definitions.
- **[UNDEFINED]** — Territory reserved from definition in this revision. Future revisions or future electrical personalities may define it. Implementations SHALL NOT assume behavior in undefined territory.

## 2. Conformance Classes

**Class C (Contained)** — Head carries its entire fluid supply within the envelope. The factory TIJ 2.5 cartridge is the reference Class C head. No umbilical. [DROPSpec]

**Class T (Tanked)** — Head is supplied by an external reservoir via a flexible umbilical exiting per §7. [DROPSpec]

A machine claiming DROPSpec compatibility SHALL accept Class C heads in all slots. A machine SHOULD accept Class T heads in all slots; a machine that restricts Class T heads to particular slots SHALL document the restriction prominently. Slot-position asymmetry is considered a defect of the machine design, not of the head.

## 3. Mechanical Envelope and Datums

### 3.1 Envelope

The conforming envelope is the HP 45-class cartridge body. [HP]

Envelope dimensions divide into two classes, and take different kinds of limit.

**Datum features** (§3.2) take a **bilateral tolerance**: a conforming head presents them within a stated ± band.

**All other envelope dimensions** take a **unilateral maximum**: a conforming head SHALL NOT exceed them, and MAY be smaller anywhere. Undersize costs the machine nothing; oversize collides with the stall, the neighbouring slot, or a service volume.

| Parameter | Limit type | Value | Status |
|---|---|---|---|
| Overall height | Maximum | TBD mm | [HP] — to be captured from reference hardware |
| Overall width | Maximum | TBD mm | [HP] |
| Overall depth | Maximum | TBD mm | [HP] |
| Thumb handle geometry | Maximum | TBD | [HP] — the handle is a datum landmark for §7 |
| Nozzle plate position and extent | Bilateral | TBD | [HP] |
| Pad film position and extent (rear face) | Bilateral | TBD | [HP] |
| Datum A / B / C | Bilateral | TBD | [DROPSpec] — see §3.2 |
| Mass, filled (reference Class C head) | — | 120–125 g | [HP] — measured |
| Mass, maximum (any conforming head, excluding umbilical) | Maximum | TBD g | [DROPSpec] — machine carriage design load derives from this × slot count |

Final envelope dimensions will be published as a measured drawing set derived from reference cartridge hardware.

**Mass note (normative once TBD resolves):** A reference filled Class C head is 120–125 g; a four-slot machine therefore carries ~500 g of heads before carriage hardware. The maximum-mass line exists so future heavy heads (tanked instruments, dense fluids) declare themselves against a known ceiling rather than surprising a carriage designed to the reference mass. A head exceeding the maximum does not conform, full stop — the scan dynamics budget is machine property.

**Design note (non-normative):** The split above exists so that shell-to-shell dimensional variation is absorbed by the limit type rather than by clearance. Only the datum features need a two-sided band; everywhere else, a head that comes in under the ceiling is simply conforming, and the machine never has to care how far under.

### 3.2 Datums

The head SHALL present the following datum features, which the machine's clamping system references [HP, codified as DROPSpec convention]:

- **Datum A** — TBD (primary seating surface)
- **Datum B** — TBD (lateral registration feature)
- **Datum C** — TBD (depth stop)

Datum features SHALL be selected from surfaces that the factory carriage itself references to seat and contact the cartridge.

Clamp designs SHALL preload the datum features consistently.

**Design note (non-normative):** The datum-selection rule above is what makes the tolerance band affordable. Surfaces the factory carriage references are held tight across the entire cartridge ecosystem by market pressure rather than by any published tolerance — a cartridge that does not seat in a factory printer does not sell. Those are the only features on the shell whose dimensional stability can be relied upon without a controlled drawing, and they are therefore the only sensible candidates for a datum.

**Design note (non-normative):** Nozzle die placement relative to the shell is deterministic. The die's truncated end portions are supported by molded surface portions and central peninsulas within the cartridge's headland, and the TAB head assembly is adhesively located against that molded headland pattern — so die position is referenced to molded features formed in the same tool as the shell's exterior. [HP] Die-to-datum registration is a tooling relationship, set once at the mold and inherited unchanged by any party that fills or refills the resulting cartridge.

Machines therefore do not store per-cartridge registration offsets, and cartridge replacement is not a calibration event.

Stitch registration in a multi-slot gang — the offset between one stall and the next — is set by the stall gang's own geometry at the time it is manufactured. It is not an operator-facing calibration, not machine state, and not a per-cartridge quantity. Its accuracy is bounded by the shrink-compensation strategy used to produce the stall, which is why shrink compensation is applied at datum surfaces rather than globally: global scaling moves the stagger pitch, and the stagger pitch is the dimension that sets stitch quality.

### 3.3 Head identification

A conforming Class C head implementing Electrical Personality 1 carries a **thermal sense resistor (TSR)** at contact position 27 and a **10× resistor (R10X)** at position 28, per Appendix A. [HP]

A machine MAY read these for per-head energy calibration and for head-presence detection.

A head declaring an electrical personality other than Personality 1 leaves positions 27 and 28 bare (§5.4). A machine probing this circuit therefore reads an open for any head that is not a Personality 1 head, and SHALL treat that open as a valid negative result rather than a fault.

Machines SHOULD detect head removal and insertion events, and SHOULD respond to an insertion by offering a **nozzle health check** — a test pattern exercising every nozzle, evaluated by the operator or by a sensor. A machine MAY allow the operator to skip it. A machine SHALL NOT require a registration calibration on insertion.

**Design note (non-normative):** The observed failure mode of this consumable is nozzle health, not placement. Cartridges arrive or become partially dead; they do not arrive misregistered. The insertion check is aimed accordingly.

### 3.4 Keep-out zones (head-side obligations)

The following machine-owned volumes SHALL remain clear of any head protrusion, in any conformance class:

- **Capping engagement volume** — the travel path and seated position of the machine's capping gasket against the nozzle plate region. TBD dimensions. [DROPSpec]
- **Wipe access volume** — the frontal clearance required for in-place manual wiping of the nozzle face at the machine's maintenance position. TBD dimensions. [DROPSpec]
- **Adjacent-slot volume** — no feature of a head may extend laterally beyond the envelope width. In an inline gang, a head's side faces are its neighbors' property. [DROPSpec]

## 4. Electrical Personality 1: TIJ 2.5 Pad Film

### 4.1 Status

The TIJ 2.5 flex-film pad array on the rear face, exactly as found on the HP 45-class cartridge, is **Electrical Personality 1** and is the *complete* electrical interface of this revision. [HP]

This specification adds nothing to the film, removes nothing from it, and reinterprets nothing on it. A conforming machine drives it per documented TIJ 2.5 practice; a conforming Class C head presents it per factory convention.

### 4.2 Personality 1 is defined by reference [HP]

Electrical Personality 1 is the TIJ 2.5 interface as produced by the existing HP 45-compatible cartridge industry. That industry is mature, multi-vendor, and in volume production. This specification does not define Personality 1, does not restate it, and claims no authority over it.

Firing voltage, pulse energy and timing, nozzle-to-driver mapping, thermal sensing, calibration, and every other drive property of a Personality 1 head are properties of that ecosystem. A machine implementing Personality 1 conforms to existing practice, not to this document.

What this specification records is the **contact position address space** — which positions exist on the film, and what Personality 1 natively uses each of them for. It is recorded for one reason: §5.4 requisitions positions by index, and an index is meaningless without a published address space. See Appendix A.

Orientation only, not normative: 300 nozzles, 600 dpi native pitch, one-half inch swath, address/primitive multiplexed firing. Aqueous TIJ decap behavior in this cartridge format is short enough that capping is a machine-side lifecycle requirement, not an option.

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

After successful handshake, a negotiated personality MAY requisition the following eight contact positions for auxiliary data and encoder passthrough:

**Positions 22, 23, 24, 25, 26, 29, 30, 31.**

A head declaring an electrical personality other than Personality 1 **SHALL NOT** present conductive material at any contact position outside this set. The remaining forty-four machine-side contacts land on insulator.

Positions **27 and 28** lie between 26 and 29 and are excluded. A negotiated head SHALL leave them bare.

This is what makes the film *control pads* rather than TIJ pads (§4.3): function is assigned by negotiated personality. Encoder passthrough exists so a head with onboard drive intelligence can self-time firing against the machine's position truth. Requisition never occurs for Personality 1 heads, which cannot handshake and whose film use is unchanged and unchangeable.

**Design note (non-normative):** Positions 22–31 are the only run of ten consecutive positions on this film carrying neither a primitive select nor a common — the only run that never carries firing current under Personality 1 operation. PS1 at 21 and PS14 at 32 bound it; no other gap between power positions exceeds four. This is not the operating rationale, since a machine that has detected a TOOMAH does not drive the film as Personality 1 at all. It is a fault-tolerance property: should a machine erroneously drive the film while a negotiated head is seated, the requisitioned positions are the only ones where there is no firing current to deliver. The head is protected by which positions were chosen, not by the machine behaving correctly.

**Design note (non-normative):** Positions 27 and 28 are the thermal sense resistor and the 10× resistor, and a Personality 1 machine may probe that circuit to detect insertion before any handshake can exist. Leaving them bare on negotiated heads makes the probe answer correctly by itself: it reads an open, which unambiguously means *not a Personality 1 head* — exactly what the machine needs to know before it does anything else. The cost is two conductors. The TOOMAH already carries ground, logic supply, and the I2C pair on blades, so the film handles only auxiliary data and encoder passthrough. Eight is a working budget, not a comfortable one; a personality needing more will need a different mechanism, not a wider requisition.

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

A head meeting this specification inherits, from any conforming machine: the carriage and clamping system, the service station lifecycle (capping, spitting, wipe access), encoder-synchronized firing infrastructure, per-head energy calibration, nozzle health checking, and the raster pipeline. A machine meeting this specification inherits every conforming head — which today means the entire industrial TIJ 2.5 cartridge and ink ecosystem, and tomorrow means whatever anyone chooses to build inside the envelope.

That trade is the point of the document.

## Appendix A — Contact position index (informative)

Contact positions and their native Personality 1 functions, after US 6,332,677 B1 (Steinfield et al., Hewlett-Packard) Table II. [HP]

This table is informative. It is reproduced so that §5.4 can name positions unambiguously. Nothing in this specification governs what a Personality 1 head or machine does with any of these positions.

| Pos | Native | Pos | Native |
|---|---|---|---|
| 1 | A9 | 2 | G6 |
| 3 | PS7 | 4 | PS6 |
| 5 | G7 | 6 | A11 |
| 7 | PS5 | 8 | A13 |
| 9 | G5 | 10 | G4 |
| 11 | G3 | 12 | PS4 |
| 13 | PS3 | 14 | A15 |
| 15 | A7 | 16 | A17 |
| 17 | A5 | 18 | G2 |
| 19 | G1 | 20 | PS2 |
| 21 | PS1 | 22 | A19 |
| 23 | A3 | 24 | A21 |
| 25 | A1 | 26 | A22 |
| 27 | TSR | 28 | R10X |
| 29 | A2 | 30 | A20 |
| 31 | A4 | 32 | PS14 |
| 33 | PS13 | 34 | G14 |
| 35 | G13 | 36 | A18 |
| 37 | A6 | 38 | A16 |
| 39 | A8 | 40 | PS12 |
| 41 | PS11 | 42 | G12 |
| 43 | G11 | 44 | G10 |
| 45 | A10 | 46 | PS10 |
| 47 | A12 | 48 | G8 |
| 49 | PS9 | 50 | PS8 |
| 51 | G9 | 52 | A14 |

`A` = address select · `PS` = primitive select · `G` = common · `TSR` = thermal sense · `R10X` = 10× resistor

The interface comprises 22 address select lines, 14 primitive selects, 14 commons, and the two resistor positions — 52 in total. Odd positions occupy one side of the film, even positions the other. Position numbering runs as a serpentine across the film: positions `2h−1` and `2h` share a height level, and consecutive same-side positions alternate between columns.

**Figure A-1** (`figure-a-1-contact-position-index.svg`) shows the position layout and the §5.4 requisition set. It is schematic. Position pitch, pad size, column spacing, and film extent are UNDEFINED in this revision and will be captured by measurement alongside §3.1. No dimension may be inferred from the figure.

## 9. Open Items

- Measured envelope drawing set, with the tolerance band for datum features and the maximum for all other dimensions (§3.1)
- Datum feature selection and definition, constrained per §3.2 to factory-carriage-referenced surfaces
- Contact position pitch, pad dimensions, and film extent (Appendix A, Figure A-1)
- The TOOMAH: position on the rear face, receptacle block and blade geometry, blade family selection (commodity, multi-vendor), engagement stagger lengths, shutter mechanism, vH voltage envelope, vL voltage, and I2C protocol and parameter set (§5)
- Exit zone dimensions for Zones H and F, and the shared protrusion limit (§7.1); Zone H clearance to be captured by measurement from C6119A factory hardware
- Keep-out volume dimensions pending service station design freeze (§3.4)
- Specification versioning and personality-numbering scheme for future revisions

## 10. License

CERN-OHL-S (hardware definitions) / CC BY-SA 4.0 (this document), per project licensing. Final notice text TBD.

---

*Draft 0.2. Geometry gospel; contents wild west. Comments, measurements, and dissent welcome via the project repository.*
