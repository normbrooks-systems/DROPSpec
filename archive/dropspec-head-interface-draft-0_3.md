# DROPSpec Head Interface Specification

**Draft 0.3 — pre-release working draft**

Part of DROPSpec — the Deposition Raster Open Print Standard.

---

## 0. Reading This Document

**SHALL** denotes a requirement for conformance. **SHOULD** denotes a strong recommendation. **MAY** denotes a permitted option. **UNDEFINED** denotes territory this revision deliberately makes no claims about — see §1.2, because UNDEFINED is not filler in this document; it is a load-bearing term.

**TBD** marks values not yet finalized. A TBD is a placeholder for a number, not for a decision — the decisions in this draft are settled; the dimensions await measurement and verification.

**EXC** marks an exclusion zone: a region this specification declares out of bounds and will not characterise. An EXC is not a TBD. Nothing about it is pending.

### 0.1 Maturity marking

Every section carries a maturity label. Three different things are being tracked and readers will conflate them unless the distinction is stated: **provenance** ([HP] / [DROPSpec] / [UNDEFINED]) says where content came from; **TBD** says a decided thing awaits a number; **maturity** says how settled the process around it is.

| Label | Meaning |
|---|---|
| **Stable** | Contains no TBDs. Reopening requires a committee vote and increments the version. |
| **Draft** | Actively being written. May change without ceremony. |
| **Reserved** | Deliberately deferred, with a stated reason. |
| **Mixed** | Subsections differ — see subsections. |

A subsection is Stable only when every TBD within it is filled. A section is Stable only when every subsection is; where they differ the section is **Mixed**, even at eleven-to-one. **Reserved** requires a stated reason for the deferral, and is not a synonym for unfinished.

Locked means harder to change, not impossible. §4 rests on a reverse-engineering lineage; if that lineage is wrong, the door must open.

### 0.2 Dimensional convention

Dimensions in this document are **nominal design values**, stated to the precision a designer would specify — not to the precision of any one measurement of any one sample.

Measured departures from nominal on reference hardware run to approximately **±0.25 mm**. That figure is an observation about the parts examined, not a conformance requirement. Where a tolerance is required it will be stated explicitly; absent that, do not infer one from the number of decimal places.

Measurements are reported with their grade: **measured** (from reference hardware, method stated), **constructed** (placed by geometric construction from measured values), or **inferred** (reasoned from indirect evidence).

## 1. Scope and Philosophy — *Stable*

### 1.1 What this specifies

This document defines the mechanical, fluidic, and electrical interface between a DROPSpec-compatible machine and a DROPSpec-compatible print head, using the HP 45-class (TIJ 2.5) cartridge envelope as its foundation.

The specification covers:

- The mechanical envelope, datums, exclusion zones, and keep-out zones of a conforming head
- The fluidic interface geometry, including the two umbilical exit zones (Zone H and Zone F) for tank-fed heads
- The electrical contact interface: the TIJ 2.5 pad film (Electrical Personality 1) and the TOOMAH auxiliary interface (§5)
- Interaction requirements with machine service functions (capping, wiping, maintenance access)

### 1.2 What this deliberately does not specify

**The geometry is gospel. The contents are the wild west.**

This specification defines the *shape* of a conforming head — its envelope, datums, contact positions, and fluid exit geometry — as normative and stable. It says nothing about what is inside the shell. Ink chemistry, jetting technology, internal reservoir design, filtration, and drive electronics beyond the contact interface are all outside the scope of this document, permanently and on purpose.

A conforming head is any object that:

1. Fits the envelope and presents the datums (§3),
2. Respects the exclusion zones and keep-out zones (§3.5, §3.6, §6),
3. If it makes electrical contact, does so per a defined electrical personality (§4) or the auxiliary interface rules (§5),
4. If it carries an umbilical, exits it per §7.

A factory TIJ 2.5 cartridge conforms. A refilled cartridge conforms. A cartridge filled with TiO₂ white, MICR magnetic ink, or a UV-fluorescent emissive conforms. A future head containing an entirely different jetting technology conforms, provided it honors the geometry and does not touch what it has not negotiated. The specification's job is to guarantee that all of these interoperate with any conforming machine without either party knowing the other in advance.

### 1.3 Provenance

This document mixes three kinds of content, and marks which is which:

- **[HP]** — Documented behavior of the HP 45-class cartridge as established hardware fact, drawn from HP's own patent disclosures (principally US 6,332,677 B1) and from the open reverse-engineering lineage (Yvo de Haas / YTEC HP45 controller work and successors). This specification does not claim authorship of these behaviors; it codifies them as the compatibility baseline. Patent disclosures describe a preferred embodiment rather than shipping hardware, and are marked [HP] on the basis that the disclosed architecture matches the cartridge in production.
- **[DROPSpec]** — Conventions original to this specification: the auxiliary interface, the umbilical exit zone, conformance language, datum selection, and keep-out definitions.
- **[UNDEFINED]** — Territory reserved from definition in this revision. Future revisions or future electrical personalities may define it. Implementations SHALL NOT assume behavior in undefined territory.

**Where this document and the cartridge disagree, the cartridge is correct.** This is a description of an existing article plus conventions layered on it; the article is the arbiter.

## 2. Conformance Classes — *Stable*

**Class C (Contained)** — Head carries its entire fluid supply within the envelope. The factory TIJ 2.5 cartridge is the reference Class C head. No umbilical. [DROPSpec]

**Class T (Tanked)** — Head is supplied by an external reservoir via a flexible umbilical exiting per §7. [DROPSpec]

A machine claiming DROPSpec compatibility SHALL accept Class C heads in all slots. A machine SHOULD accept Class T heads in all slots; a machine that restricts Class T heads to particular slots SHALL document the restriction prominently. Slot-position asymmetry is considered a defect of the machine design, not of the head.

## 3. Mechanical Envelope and Datums — *Mixed*

### 3.1 Face nomenclature — *Stable* [DROPSpec]

Faces are named independently of the dimension schedule. Dimension letters are consumed once measured; face labels are referenced for the life of the document.

| Label | Face |
|---|---|
| **AA** | Top |
| **AB** | Left — **X datum**. Carries the Personality 1 pad film and the TOOMAH. |
| **AC** | Bottom, nozzle-end land — **protected**, see §3.6 |
| **AD** | Bottom, main body land — **Y datum** |
| **AG** | Right |

AC and AD are both bottom faces, at different levels, doing opposite jobs. Prose SHALL NOT refer to "the bottom face."

**AB is flat from AC up to 77.65 mm**, where it goes tangent into the upper corner fillet. Every feature mounted on AB — pad film, TOOMAH — lies below that line.

### 3.2 Envelope — *Draft* (mass maximum outstanding)

The conforming envelope is a **superset within which the HP 45-class cartridge body fits**. It is no longer identical to that body: the TOOMAH (§5) stands proud of AB, and growth of order 0.1 mm is permitted on non-datum faces per §3.3. Conformance is unaffected — §1.2's test is that a head fits the envelope and presents the datums, which a smaller object passes.

Reference article: HP **C8842A** Versatile Black.

| Dim | Feature | mm | Grade |
|---|---|---|---|
| A | Overall width, including thumb grip | 68.60 | measured |
| B | Body width, AB to AG | 60.10 | measured |
| M | Grip protrusion past AG | 8.50 | derived (A − B) |
| C | AB to first bottom step | 19.00 | measured |
| D | AB to second bottom step | 26.80 | measured |
| E | Overall height, AA to AC | 91.90 | measured |
| F | AA to intermediate step level | 82.10 | measured |
| G | AA to AD | 78.10 | measured |
| N | AD to AC step | 13.80 | derived (E − G) |
| O | D to AG along AD | 33.30 | derived (B − D) |
| Z | Thickness, AB face width | 18.80 | measured (18.75 as found) |
| J | Lower-right corner radius | R14.25 | measured — 325 boundary points, 0.28 px residual |
| K | Upper-right corner radius | R14.25 | constructed |
| L | Upper-left corner radius | R14.25 | constructed |
| — | Mass, filled (reference Class C head) | 120–125 g | [HP] measured |
| — | Mass, maximum (any conforming head, excluding umbilical) | **TBD** | [DROPSpec] |

**N is the flying-height chain dimension.** Seating Y on AD places the nozzle plate at AD + N. It requires no support (§3.6), but it requires stating, because it is the only dimension linking the Y datum to the nozzle plate.

**O caveat:** the lower-right fillet goes tangent to AD at 45.75 mm from AB, so 14.35 mm of O is radius. **The usable AD seating land is 18.95 mm**, not 33.30.

**On K and L:** J is the only measured radius. K measured independently at R14.20 from roughly 30° of visible arc before the thumb grip occludes it — consistent with J within the uncertainty of a short-arc fit — and has been set equal to J. L is unmeasurable on the reference article for reasons given in §3.5. Both are **constructed**, placed by tangency to the fitted edge lines, and are assumptions rather than observations.

**Mass note (normative once TBD resolves):** A reference filled Class C head is 120–125 g; a four-slot machine therefore carries ~500 g of heads before carriage hardware. The maximum-mass line exists so future heavy heads (tanked instruments, dense fluids) declare themselves against a known ceiling rather than surprising a carriage designed to the reference mass. A head exceeding the maximum does not conform, full stop — the scan dynamics budget is machine property.

### 3.3 Envelope growth rule — *Stable* [DROPSpec]

Growth beyond the reference article is permitted **only on non-datum faces**.

- **AB and AD are frozen.** They are the X and Y datums (§3.4); moving them moves the nozzle plate relative to the stall.
- **AC is frozen.** It carries the nozzles.
- **Growth is permitted on AA, AG, and Z.**

Growth in Z multiplies by slot count and is the dimension §3.6 calls the adjacent-slot volume. Growing it is a specification edit, not an implementer's free choice.

**Rationale (non-normative):** growth away from the datums is invisible to registration — a factory cartridge and a grown head land on the same two faces and place their nozzle plates in the same position. Growth *toward* a datum face would create two classes of head that do not register alike.

### 3.4 Datums — *Stable* [DROPSpec]

The head SHALL present the following datum features, which the machine's clamping system references:

- **Datum X — face AB**, hard seat
- **Datum Y — face AD**, hard seat
- **Datum Z — the thickness axis**, self-centering between compliant elements bearing on both broad faces

A conforming clamp SHALL seat X and Y against hard stops with sufficient preload to resist scan-motion loads without relying on friction from the Z centering elements. Z centering force is a machine-side quantity; reference practice is of order 1–1.5 N total.

**Seating land availability:** AB provides approximately 64 mm of flat between fillet tangent points, less the TOOMAH footprint (§5.1), leaving 61.65 mm below the tangent and above the nozzle plate. This will be reduced by the TOOMAH's presence, eventually, and compressed to approximately 45 mm. AD's flat land is 18.95 mm (§3.2). Three of the four profile corners are R14.25; radius-on-radius contact slides rather than locates, so clamps SHALL bear on the flats and not on the corner between them.

**This revision departs from draft 0.2.2**, which required datums be selected from surfaces the factory carriage itself references. That constraint was written before those surfaces had been examined. HP's locating feature is a fin lying on the mould parting line — the least trustworthy surface on a moulding, subject to mould-half register error and to flash. Referencing chosen surfaces on the outer envelope avoids inheriting that tolerance entirely.

**Design note (non-normative):** the secondary consequence is the more important one. A conforming head no longer needs to reproduce the locating fin or the bottom features. It needs two flats and a thickness. That is the difference between cloning HP's shell and fitting inside an envelope, and only the second of those is an open standard.

**Design note (non-normative):** The cartridge self-locates its own die. The nozzle die is supported and positioned by molded features of the shell — the headland's wall openings, support surfaces, and central peninsulas — so die-to-datum registration is fixed by the mold, not by assembly technique. [HP] Registration variance is therefore a manufacturing defect rather than an expected quantity: a conforming cartridge places its die where the geometry says it goes. Machines do not store per-cartridge registration offsets, and cartridge replacement is not a calibration event.

The die-to-datum relationship is located by internal moulded features that cannot be reached, measured, or designed around from outside the shell. The assertion above is therefore **untested**. §9 records the test.

Stitch registration in a multi-slot gang is set by the stall gang's own geometry at the time it is manufactured. It is not an operator-facing calibration, not machine state, and not a per-cartridge quantity. Its accuracy is bounded by the shrink-compensation strategy used to produce the stall, which is why shrink compensation is applied at datum surfaces rather than globally: global scaling moves the stagger pitch, and the stagger pitch is the dimension that sets stitch quality.

### 3.5 Exclusion zones — *Stable* [DROPSpec]

Two regions of the reference article's profile are declared **exclusion zones**. Their internal geometry is uncharacterised and this specification will not characterise it. A conforming machine SHALL NOT rely on any feature within them.

| Zone | Location | Known contents |
|---|---|---|
| **EXC-1** | Upper-left, at the AA/AB corner | A triangular locating feature lying on the mould parting line, with an adjacent feature that appears to set height. Sloped top face. Projects 0.50 mm past AB. |
| **EXC-2** | Bottom centre, adjacent to AC | A ribbed rectangular boss, a stepped notch, and a slotted tab. Multi-level. |

Declaring these excluded is what permits §3.4 to place the datums elsewhere. It converts geometry nobody has measured into a bounded promise, and it removes the parting-line fin from the tolerance chain entirely.

**EXC-2 interacts with service access.** It is adjacent to AC, which the capping gasket engages and the wipe path crosses (§6). A machine's exclusion of EXC-2 SHALL NOT be drawn so as to make capping or wiping non-conforming; the two obligations must be reconciled at service station design freeze.

### 3.6 Keep-out zones — *Draft* (dimensions outstanding)

Two kinds of keep-out exist and conflating them would either forbid capping or permit clamping on the nozzle plate.

**Machine-owned volumes.** The following SHALL remain clear of any head protrusion, in any conformance class: [DROPSpec]

- **Capping engagement volume** — the travel path and seated position of the machine's capping gasket against the nozzle plate region. TBD dimensions.
- **Wipe access volume** — the swept volume of the machine's wiper blade across the nozzle face at the maintenance position (§6). TBD dimensions.
- **Adjacent-slot volume** — no feature of a head may extend laterally beyond the envelope Z dimension. In an inline gang, a head's broad faces are its neighbours' property.

**Head-owned no-load zones.** The following SHALL NOT receive clamping load from the machine: [DROPSpec]

- **Face AC**, the nozzle-end land, and the region of the head between AD and AC.

A machine is *permitted* in the AC region — the capping gasket is deliberately there and the wipe path crosses it. What is forbidden is clamping force. The head is cantilevered from AD across the N dimension; loading AC deflects that cantilever and moves the nozzle plate.

**Design note (non-normative):** the shell is compliant. Measured dimensions vary with caliper jaw force, and the same compliance appears under clamp preload. Flying height is therefore a function of clamp force as well as of geometry, and preload may need to become a specified quantity rather than an implementer's choice. §9 records the test.

## 4. Electrical Personality 1: TIJ 2.5 Pad Film — *Mixed*

### 4.1 Status — *Stable*

The TIJ 2.5 flex-film pad array on face AB, exactly as found on the HP 45-class cartridge, is **Electrical Personality 1** and is the *complete* electrical interface of this revision. [HP]

This specification adds nothing to the film, removes nothing from it, and reinterprets nothing on it. A conforming machine drives it per documented TIJ 2.5 practice; a conforming Class C head presents it per factory convention.

### 4.2 Personality 1 is defined by reference — *Stable* [HP]

Electrical Personality 1 is the TIJ 2.5 interface as produced by the existing HP 45-compatible cartridge industry. That industry is mature, multi-vendor, and in volume production. This specification does not define Personality 1, does not restate it, and claims no authority over it.

Firing voltage, pulse energy and timing, nozzle-to-driver mapping, thermal sensing, calibration, and every other drive property of a Personality 1 head are properties of that ecosystem. A machine implementing Personality 1 conforms to existing practice, not to this document.

What this specification records is the **contact position address space** — which positions exist on the film, where they are, and what Personality 1 natively uses each of them for. It is recorded for one reason: §5.5 requisitions positions by index, and an index is meaningless without a published address space. See Appendix A.

Orientation only, not normative: 300 nozzles, 600 dpi native pitch, one-half inch swath, address/primitive multiplexed firing. Aqueous TIJ decap behavior in this cartridge format is short enough that capping is a machine-side lifecycle requirement, not an option.

### 4.3 Pad field geometry — *Stable* [HP]

The pad field is a physical interface independent of what Personality 1 does electrically, and is specified here so that a stall connector can be built without reference to §4.2.

| Parameter | Value |
|---|---|
| Pad size | 1.5 across × 1.7 vertical |
| Column separation | 0.3 |
| Column separation, row 7 | 0.45 |
| Row separation, rows 1–7 | 0.4 |
| Row separation, rows 7–11 | 0.6 |
| Centre gap, inner column to inner column | 5.3 |
| Outermost pad edge to outermost pad edge | 15.5 |
| Chevron rise per column step, inward | 0.2 |
| Lowest pad edge, from AC | 14.5 |
| Highest pad edge, from AC | 38.4 |
| Film top edge, from AC | 42.4 |
| Film width | UNDEFINED — the film carries lateral tolerance and is not a datum |

The array is **centred on Z** and is **mirror symmetric** about that centreline.

Fifty-two pads in eleven rows. Rows 1–7 carry two columns per side; rows 8–11 carry three. The wider 0.6 row separation begins exactly where the third column appears.

**Design note (non-normative):** the row separation widens where the column count increases, which is consistent with trace clearance: the upper rows route three columns' worth of conductors through the same film width. Row 7's 0.45 column separation displaces positions 27 and 28 inboard by 0.15 each — see §4.5.

### 4.4 Contact pads are control pads — *Stable* [DROPSpec]

Throughout this specification the film contacts are termed **control pads**, not firing pads. The name is deliberate: the pads are the machine's control interface to *a head*, of which the thermal-resistive TIJ 2.5 implementation is version one. The specification's compatibility promise attaches to the contact geometry and the negotiated personality — not to the assumption that a resistor lives behind every pad. A negotiated non-thermal personality may requisition a bounded subset of these positions for data and encoder passthrough; see §5.5.

### 4.5 Head identification — *Stable*

A conforming Class C head implementing Electrical Personality 1 carries a **thermal sense resistor (TSR)** at contact position 27 and a **10× resistor (R10X)** at position 28, per Appendix A. [HP]

A machine MAY read these for per-head energy calibration and for head-presence detection.

A head declaring an electrical personality other than Personality 1 leaves positions 27 and 28 bare (§5.5). A machine probing this circuit therefore reads an open for any head that is not a Personality 1 head, and SHALL treat that open as a valid negative result rather than a fault.

Machines SHOULD detect head removal and insertion events. A machine SHALL NOT require a registration calibration on insertion.

**Design note (non-normative):** A Personality 1 head reports no per-nozzle state. The thermal sense resistor reads substrate temperature, not individual nozzles, and nothing in the head can indicate a failed jet. Nozzle condition is observable only by printing and looking; what a machine does about that is machine business and outside this specification.

**Design note (non-normative):** positions 27 and 28 sit in row 7, the only row using 0.45 column separation, which places them 0.15 mm further inboard than their columns elsewhere. The two positions the specification already treats as special are the two the mould also treats as special. Whether that is deliberate on HP's part is unknown; it is recorded because it is measurable and because someone reading a pad map will otherwise take it for an error.

## 5. The TOOMAH (Auxiliary Interface) — *Mixed*

### 5.1 Definition and geometry — *Draft*

The auxiliary interface is the **TOOMAH**: a plastic receptacle block standing **5 mm proud of face AB**, on the same face as the Personality 1 control-pad film, carrying five blade receptacles. [DROPSpec]

The name is a proper noun, not an acronym. The TOOMAH is, definitionally, a growth on the side of a standard cartridge. It is not a tumor.

| Parameter | Value | Grade |
|---|---|---|
| Face | AB | settled |
| Footprint on AB | 12.0 wide × 15.0 tall | proposed |
| Proudness | 5.0 | settled |
| Top edge, below the upper fillet tangent | 1.0 | proposed |
| Blade length | 8 | proposed |
| Blade count | 5 | settled |
| Blade arrangement | one transverse, four longitudinal below it | proposed |
| Blade family (commodity, multi-vendor) | **TBD** | |
| Engagement stagger lengths | **TBD** | |

Resulting clearances on AB: 19.25 to the film top edge, 23.35 to the topmost pad, 3.40 to each edge of Z, and 61.65 mm of AB left below the block for the X clamp.

| Blade | Function | Engagement |
|---|---|---|
| **G** | Ground reference | Longest — first mate, last break |
| **vL** | Logic supply; powers the head's controller for enumeration before vH exists. Voltage TBD. | Mid-length |
| **D+ / D−** | Data pair carrying the I2C handshake bus (SDA/SCL assignment TBD) | Mid-length |
| **vH** | High-voltage drive rail; de-energized by default, energized only after handshake (§5.4). Envelope TBD. | **Shortest — last mate, first break** |

Blade lengths SHALL be staggered as tabled, so the connector sequences itself: a head is grounded and enumerable before vH can physically connect, and on removal vH breaks mechanically before logic and ground do.

**Design note (non-normative):** engagement stagger runs along the carrier's travel axis (§5.2), which is normal to AB. Contact arrangement *on the face* is therefore independent of sequencing, and may be chosen on creepage and packaging grounds alone. Every contact wipes along its own insertion travel regardless of its orientation on the face. Placing G and vH at opposite extremes maximises creepage but also maximises the drive-current return loop; a head with a fast-edged driver will want local bulk capacitance at the blades. The 5 mm proudness provides out-of-plane separation from the film plane without this specification naming a distance.

### 5.2 Engagement — *Draft* [DROPSpec]

**The head does not mate on insertion.** The cartridge seats and clamps on AB and AD first; the machine-side contact carrier then advances into the TOOMAH receptacle.

A conforming machine's contact carrier travel SHALL be **linear and parallel to the blade axis** over the engagement stroke.

This is what makes the stagger sequence the connector. On an arced path, blade tips sweep different radii and meet contacts at different angles, mating order ceases to follow from blade length, and the vH-last-to-mate guarantee is lost. Blade wipe and any cammed geometry have the same dependency.

**Design note (non-normative):** separating the two motions means nothing electrical happens until the head is fully seated and clamped, so clamp preload is not fighting connector insertion force. This supersedes draft 0.2.2's requirement that blade insertion force be accounted for in stall preload; that concern applied to a design where mating happened during insertion.

Engagement stagger must exceed the sum of carrier misalignment, blade compliance, and part tolerance. That margin is TBD and is a measurement, not an estimate.

### 5.3 Keying and protection — *Draft* [DROPSpec]

The TOOMAH's exclusion properties rest on **two geometric layers and one electrical one**. This revision states them separately because they are not equivalent and draft 0.2.2's presentation of three independent mechanical layers no longer holds.

1. **Head side (geometric, operator-independent):** the TOOMAH's position and 5 mm proudness SHALL interfere with the reference (TIJ-only) stall envelope, such that a TOOMAH-bearing head cannot seat in a stall lacking the auxiliary slot. A high-voltage head in a thermal-drive stall is not forbidden; it is *unconstructable*.
2. **Machine omission (geometric):** a machine MAY omit the auxiliary system entirely; omission mechanically rejects TOOMAH-bearing heads by rule 1, which is the intended behavior.
3. **Electrical interlock:** vH is de-energized until handshake completes (§5.4). An exposed blade is not a live blade.

**The machine-side blades are protected by a manually removed cap**, not by a shutter. The cap provides **ingress and contamination protection** — it keeps debris and ink aerosol out of an unused slot — and it is **not an interlock**, because it depends on an operator having refitted it. Draft 0.2.2 claimed a shutter opened by conforming geometry, which would have excluded a finger or a dropped fastener regardless of machine state. It was dropped on cost. Nothing in this revision claims that guarantee.

What replaces it is rule 3. A machine SHALL NOT energize vH on an uncapped, unmated slot; vH exists only after a completed handshake with an enumerated head. This is the stronger guarantee of the two in any case, since it does not depend on a mechanical part being present and correct.

**Design note (non-normative):** with the cap removed and the carrier under machine control, nothing mechanical prevents a carrier advancing against a flat-faced Personality 1 head. Rule 1 makes such a head unable to seat in a TOOMAH-equipped stall in the first place, so the exposure should be nil — but it now rests on rule 1 alone, where previously a second layer covered it. Implementers of the carrier drive should treat rule 1 as load-bearing.

### 5.4 Handshake — *Reserved*

**Reserved reason:** the interlock is mechanical (§5.3 rule 1) and electrical (vH de-energized by default), and neither depends on what the bus conversation says. A protocol invented with no implementer gets details wrong in ways nobody discovers until someone tries to use it. Enumeration will be specified when a real head needs it.

The following is normative and is not deferred:

vH SHALL NOT be energized until an I2C handshake completes on D+/D−, in which the head enumerates, declares its electrical personality, and states its drive requirements. vH SHALL de-energize on loss of I2C device presence — a backup behavior, since the engagement stagger already breaks vH first on any removal. vL and the handshake are the only electrical activity permitted on an unenumerated TOOMAH.

Protocol, parameter set, vH envelope, and vL voltage are **UNDEFINED** in this revision.

### 5.5 Film requisition — *Stable* [DROPSpec]

After successful handshake, a negotiated personality MAY requisition the following **ten** contact positions for auxiliary data and encoder passthrough:

**Positions 29, 30, 31, 36, 37, 38, 39, 45, 47, 52.**

Equivalently and normatively: **the address-select positions within the three-column region of the pad field** (rows 8–11, §4.3). There are twenty-four positions in that region and exactly ten of them are address-select.

A head declaring an electrical personality other than Personality 1 **SHALL NOT** present conductive material at any contact position outside this set. The remaining forty-two machine-side contacts land on insulator.

Positions **27 and 28** are excluded. A negotiated head SHALL leave them bare.

This is what makes the film *control pads* rather than TIJ pads (§4.4): function is assigned by negotiated personality. Encoder passthrough exists so a head with onboard drive intelligence can self-time firing against the machine's position truth. Requisition never occurs for Personality 1 heads, which cannot handshake and whose film use is unchanged and unchangeable.

**Design note (non-normative) — why address positions:** a Personality 2 head omits these contacts entirely, so it cannot carry current there. Omitting an address line costs addressing resolution and nothing else, and addressing only matters when the *machine* is doing the addressing — a head with its own driver has taken that job in-house. Omitting a primitive select or a common would cut a firing-current path, and a head missing those is not degraded, it is broken. That is the operating rationale.

The fault-tolerance property follows from the same fact: should a machine erroneously drive the film while a negotiated head is seated, the requisitioned positions carry no firing current under Personality 1, so there is nothing to deliver. The head is protected by which *class* of position was chosen, not by the machine behaving correctly.

**This supersedes draft 0.2.2's rationale**, which argued that positions 22–31 formed the only run of ten consecutive positions free of primitive selects and commons. That argument over-constrained the choice — any address positions satisfy the property, and there are twenty-two of them — and the run it selected contained 27 and 28, which had to be carved out, yielding eight rather than ten.

**Design note (non-normative) — why 27 and 28 stay bare:** they are the thermal sense resistor and the 10× resistor, and a Personality 1 machine may probe that circuit to detect insertion before any handshake can exist. Leaving them bare on negotiated heads makes the probe answer correctly by itself: it reads an open, which unambiguously means *not a Personality 1 head* — exactly what the machine needs to know before it does anything else. The cost is two conductors.

**Design note (non-normative) — proximity:** the requisitioned set occupies the top 7.2 mm of the pad array, roughly 23 mm from the TOOMAH block. On a negotiated head that is the shortest available internal routing between the auxiliary connector and the requisitioned pads. Ten is a working budget, not a comfortable one; a personality needing more will need a different mechanism, not a wider requisition.

### 5.6 Personality 1 obligations — *Stable*

A Personality 1 head has no TOOMAH and, by §5.3, cannot contact the auxiliary system; no head-side rule is required beyond conforming to the envelope. A machine SHALL NOT energize auxiliary blades, nor requisition film positions, in the absence of a completed handshake.

## 6. Service Interaction Requirements — *Draft*

A conforming head SHALL tolerate the machine's default service lifecycle: [DROPSpec]

- **Capping** as the idle-default state: the nozzle plate region will be engaged by an elastomer gasket whenever the machine is idle, for indefinite durations.
- **Spitting**: firmware-initiated maintenance ejection into a spittoon. Heads whose contents must not be spat (none are currently contemplated) do not conform.
- **Wiping**: a solenoid-actuated, spring-loaded silicone wiper blade makes a straight pass across the nozzle face at the maintenance position, with the head clamped. **Head features SHALL NOT obstruct a straight wipe pass across the nozzle face.**

The head-side obligation is unchanged from draft 0.2.2's manual wiping; only the mechanism differs. The wipe access volume in §3.6 is correspondingly better defined: it is the blade's swept volume, and a solenoid throw has a known stroke, where hand clearance was an estimate.

**Design note (non-normative):** the wiper blade is deliberately not a consumable. Silicone does not absorb; ink sits on the surface and is removed with a wet wipe. An automated service station is precisely where a proprietary consumable would otherwise be reintroduced into an otherwise open supply chain, and the reference design does not acquire a part number here. Aqueous ink means water is the solvent of choice; avoid ammonia-based cleaners near the nozzle plate.

## 7. Umbilical Interface (Class T) — *Draft*

### 7.1 Exit zones

A Class T head's umbilical SHALL exit the envelope through one of two conforming zones. Exit from the broad faces, bottom region, or face AB is non-conforming: the broad faces belong to adjacent slots in the gang, the bottom to the nozzle plate, capping gasket, and wipe path, and AB to the control pads and the TOOMAH. Both conforming zones are per-slot symmetric — tanks work in any position (§2).

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
- The umbilical and its strain relief SHALL remain clear of the capping engagement volume and the wipe access volume (§3.6) at all times, including under scan-motion tube deflection.
- The protrusion limit (§7.1) exists so the tail cannot act as a lever into the access window envelope during carriage motion.

### 7.3 Advisory (non-normative)

A front-exiting umbilical on a scanning carriage is a continuously flexing umbilical, cycling once per pass for the life of the head. Continuous-flex-rated tubing is strongly recommended; static-rated tubing in this application is a fatigue failure awaiting its cycle count.

## 8. What Conformance Buys You — *Stable*

A head meeting this specification inherits, from any conforming machine: the carriage and clamping system, the service station lifecycle (capping, spitting, wipe access), encoder-synchronized firing infrastructure, per-head energy calibration, and the raster pipeline. A machine meeting this specification inherits every conforming head — which today means the entire industrial TIJ 2.5 cartridge and ink ecosystem, and tomorrow means whatever anyone chooses to build inside the envelope.

That trade is the point of the document.

## 9. Versioning and Governance — *Reserved*

**Reserved reason:** a versioning scheme communicates compatibility, and compatibility claims are meaningless until someone external is building against a version. The mechanism is recorded now so that nothing changes later except membership.

- Changes to a section marked **Stable** require a committee vote to reopen, followed by a version increment.
- The review process binds all committee members, **including the author**.
- The committee is presently one person. The mechanism is therefore formal rather than operative.
- The numbering scheme itself, and the personality-numbering scheme for future electrical personalities, are **UNDEFINED** in this revision.

## 10. Open Items

**Measurement**

- Maximum mass for any conforming head (§3.2)
- Flip-scan or equivalent to establish maximum envelope under draft, since a contact-plane section reads smaller than the maximum (§3.2)
- Tolerance band for datum features, requiring samples beyond a single SKU. Both reference articles examined are C8842A; mould codes should be compared before any spread is treated as characterising rather than understating (§3.4)
- Whether clamp preload moves flying height, via shell compliance. Vary preload, print, observe (§3.6)
- Die-to-datum repeatability: print a registration pattern from two cartridges in the same stall, swap, reprint. Disagreement beyond a nozzle pitch falsifies §3.4's registration assertion (§3.4)

**TOOMAH**

- Blade family selection — commodity, multi-vendor (§5.1)
- Engagement stagger lengths, and the margin against carrier misalignment, blade compliance and part tolerance (§5.2)
- vH voltage envelope, vL voltage (§5.1)
- I2C protocol and parameter set (§5.4, Reserved)
- Whether the block's 1.0 mm clearance below the fillet tangent should increase — it strands AB too narrow to clamp on, and moulding tolerance on the tangent could put a flat-bottomed boss onto the start of the curve (§5.1)
- Whether §5.3 rule 1 alone is sufficient protection now that the shutter is gone (§5.3)

**Other**

- Exit zone dimensions for Zones H and F, and the shared protrusion limit; Zone H clearance to be captured by measurement from C6119A factory hardware (§7.1)
- Keep-out volume dimensions pending service station design freeze (§3.6)
- Reconciling EXC-2's exclusion with capping and wipe access (§3.5)
- Film width, and whether it needs stating at all given it is not a datum (§4.3)
- Versioning and personality-numbering scheme (§9, Reserved)

## Appendix A — Contact position index and geometry

Contact positions and their native Personality 1 functions, after US 6,332,677 B1 (Steinfield et al., Hewlett-Packard) Table II. [HP]

The position/function table is informative. It is reproduced so that §5.5 can name positions unambiguously. Nothing in this specification governs what a Personality 1 head or machine does with any of these positions. **The geometry in §A.2 is normative**, since a stall connector is built to it.

### A.1 Position index (informative)

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

The interface comprises 22 address select lines, 14 primitive selects, 14 commons, and the two resistor positions — 52 in total.

### A.2 Figure A-1a — contact position layout (normative)

Viewed head-on toward face AB, with AC down. Odd positions occupy one side of the film, even positions the other. Position 1 is in the outermost column of row 1.

Eleven rows. Rows 1–7 carry two columns per side (outer, mid); rows 8–11 carry three (outer, mid, inner). Within a row, positions ascend outer-left, outer-right, mid-left, mid-right, inner-left, inner-right.

Each column is offset vertically from the one outboard of it by the chevron, giving the array its characteristic form: the columns rise toward the centreline of the film, on both sides, symmetrically.

### A.3 Table A-1b — contact position dimensions (normative)

| Parameter | Value (mm) |
|---|---|
| Pad size | 1.5 across × 1.7 vertical |
| Column separation | 0.3 |
| Column separation, row 7 | 0.45 |
| Row separation, rows 1–7 | 0.4 |
| Row separation, rows 7–11 | 0.6 |
| Centre gap (inner column to inner column) | 5.3 |
| Outermost pad edge to outermost pad edge | 15.5 |
| Chevron rise per column step, inward | 0.2 |
| Lowest pad edge, from AC | 14.5 |
| Highest pad edge, from AC | 38.4 |
| Single-column height, lowest pad bottom to highest pad top | 23.5 |
| Film top edge, from AC | 42.4 |
| Film width | UNDEFINED |

The array is centred on Z and mirror symmetric about that centreline.

**Derived, for checking:** gap between opposed mid-column positions is 8.9 in rows other than 7; in row 7 it is 8.6, because the 0.45 column separation displaces positions 27 and 28 inboard by 0.15 each. Gap between opposed inner-column positions is 5.3.

**Measurement note (non-normative):** these figures are caliper measurements of reference hardware. A flatbed scan was used to establish structure — which positions exist, in which columns, in what pattern — and contributed no dimension. Scanner geometry is not calibrated and the instrument's motion blur inflates measured feature extents along its travel axis while leaving centroids unaffected, which makes it useful for counting and for relative layout and useless for size.

## 11. License

CERN-OHL-S (hardware definitions) / CC BY-SA 4.0 (this document), per project licensing. Final notice text TBD.

---

*Draft 0.3. Geometry gospel; contents wild west. Comments, measurements, and dissent welcome via the project repository.*
