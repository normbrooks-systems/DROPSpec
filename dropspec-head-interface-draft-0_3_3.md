# DROPSpec Head Interface Specification

**Draft 0.3.3 (proposed) — pre-release working draft**

Part of DROPSpec — the Deposition Raster Open Print Specification.

---

### Changes from 0.3.1 (non-normative)

The 0.3.3 revision is the TOOMAH interface redefinition. §5.5 (Stable) was reopened for it; the committee voted 1–0. This summary does not govern; the sections do.

1. The TOOMAH's transverse blade is replaced by a **USB-C receptacle** carrying power (USB PD) and data (USB 2.0). The four longitudinal blades now carry the machine's gantry wheel-encoder **quadrature** and are equal length: engagement sequencing is inherited from the USB-C connector standard rather than from blade stagger (§5.1, §5.2).
2. Block proudness **5.0 → 8.0**, providing full USB-C plug engagement without the mated plug protruding into the stall envelope. The §5.3 standoff argument carries at the larger figure.
3. Block lowered **2.0 mm**: top clearance to the fillet tangent 1.0 → 3.0, resolving the 0.3.1 open item on tangent clearance versus moulding tolerance. Consequences, all recorded in §3.4 and §5.1: block occupies 59.65–74.65 from AC; X-clamp band 47.85 → **45.85** (13.8 to 59.65); film-top clearance 19.25 → 17.25; topmost-pad clearance 23.25 → 21.25.
4. **Film requisition withdrawn** (§5.5). Data and encoder passthrough live on the TOOMAH; a head declaring a personality other than Personality 1 now presents no conductive material at any film position. The §4.5 presence-probe logic is unchanged and no longer needs a 27/28 carve-out.
5. §5.4 re-founded on USB: the handshake is USB enumeration, and the high-voltage interlock is the PD contract requirement — VBUS follows USB-C source rules, with above-default voltage only under an explicit PD contract. vH and vL as named rails are withdrawn.
6. Cross-references to the requisition mechanism repaired (§4.2, §4.4, §4.5, §5.6, Appendix A).
7. Open items updated: stagger margins, vH/vL envelopes, and the I2C protocol are superseded; the tangent-clearance item is resolved. Opened: PD power profile ceiling, USB device class, quadrature electrical specification, and blade family for the four longitudinal blades.
8. Project subtitle corrected: Deposition Raster Open Print **Specification**.

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

A subsection is Stable only when every TBD within it is filled and a minimum of half the governance committee agrees it's ready to be made stable. A section is Stable only when every subsection is; where they differ the section is **Mixed**, even at eleven-to-one. **Reserved** requires a stated reason for the deferral, and is not a synonym for unfinished.

Stable means harder to change, not impossible. §4 rests on a reverse-engineering lineage; if that lineage is wrong, the door must open.

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
| J | Lower-right corner radius | R14.25 | scan fit — 325 boundary points, 0.28 px residual; the radius is trusted as relational and robust to scan-axis blur, the fitted centre position is not used |
| K | Upper-right corner radius | R14.25 | constructed |
| L | Upper-left corner radius | R14.25 | constructed |
| — | Mass, filled (reference Class C head) | 120–125 g | [HP] measured |
| — | Mass, maximum (any conforming head, excluding umbilical) | **TBD** | [DROPSpec] |

**N is the flying-height chain dimension.** Seating Y on AD places the nozzle plate at AD + N. It requires no support (§3.6), but it requires stating, because it is the only dimension linking the Y datum to the nozzle plate.

**O caveat:** the lower-right fillet goes tangent to AD at B − J = **45.85 mm** from AB (constructed), so 14.25 mm of O — exactly J, as tangency to AG requires — is radius. **The usable AD seating land is 19.05 mm**, not 33.30. Draft 0.3 published 45.75 and 18.95, taken from the scan fit's centre position; the radius is trusted as relational, the centre is not, and the constructed figures govern.

**On K and L:** J is the only measured radius. K measured independently at R14.20 from roughly 30° of visible arc before the thumb grip occludes it — consistent with J within the uncertainty of a short-arc fit — and has been set equal to J. L is unmeasurable on the reference article for reasons given in §3.5. Both are **constructed**, placed by tangency to the fitted edge lines, and are assumptions rather than observations.

**Mass note (normative once TBD resolves):** A reference filled Class C head is 120–125 g; a four-slot machine therefore carries ~500 g of heads before carriage hardware. The maximum-mass line exists so future heavy heads (tanked instruments, dense fluids) declare themselves against a known ceiling rather than surprising a carriage designed to the reference mass. A head exceeding the maximum does not conform, full stop — the scan dynamics budget is machine property.

### 3.3 Envelope growth rule — *Stable* [DROPSpec]

Growth beyond the reference article, **of order 0.1 mm**, is permitted **only on non-datum faces**.

- **AB and AD are frozen.** They are the X and Y datums (§3.4); moving them moves the nozzle plate relative to the stall.
- **AC is frozen.** It carries the nozzles.
- **Growth on AA and AG** is an implementer's choice within the allowance.
- **Growth in Z** is permitted only by specification revision: it multiplies by slot count and is the dimension §3.6 calls the adjacent-slot volume.

**Rationale (non-normative):** growth away from the datums is invisible to registration — a factory cartridge and a grown head land on the same two faces and place their nozzle plates in the same position. Growth *toward* a datum face would create two classes of head that do not register alike.

### 3.4 Datums — *Stable* [DROPSpec]

The head SHALL present the following datum features, which the machine's clamping system references:

- **Datum X — face AB**, hard seat
- **Datum Y — face AD**, hard seat
- **Datum Z — the thickness axis**, self-centering between compliant elements bearing on both broad faces

A conforming clamp SHALL seat X and Y against hard stops with sufficient preload to resist scan-motion loads without relying on friction from the Z centering elements. Z centering force is a machine-side quantity; reference practice is of order 1–1.5 N total.

**Seating land availability:** AB's flat runs from AC to the fillet tangent at 77.65 (§3.1). Of it, the band from AC to AD level (0 to 13.8) lies in the no-load zone (§3.6), and the TOOMAH occupies 59.65 to 74.65 with a further 3.0 clearance to the tangent (§5.1). The X-clamp band is therefore **13.8 to 59.65 — 45.85 mm**. The pad field spans 14.5 to 38.4 within that band (§4.3): the **contact pads SHALL NOT receive clamping load**; film area outside the pad field MAY serve as clamp land. AD's flat land is 19.05 mm (§3.2). Three of the four profile corners are R14.25; radius-on-radius contact slides rather than locates, so clamps SHALL bear on the flats and not on the corner between them.

**Design note (non-normative):** a seat that bears partly on film and partly on bare shell carries the film thickness in its X chain on one side only; a clamp should bear on one or the other, not both.

**This revision departs from draft 0.2.2**, which required datums be selected from surfaces the factory carriage itself references. That constraint was written before those surfaces had been examined. HP's locating feature is a fin lying on the mould parting line — the least trustworthy surface on a moulding, subject to mould-half register error and to flash. Referencing chosen surfaces on the outer envelope avoids inheriting that tolerance entirely.

**Design note (non-normative):** the secondary consequence is the more important one. A conforming head no longer needs to reproduce the locating fin or the bottom features. It needs two flats and a thickness. That is the difference between cloning HP's shell and fitting inside an envelope, and only the second of those is an open standard.

**Design note (non-normative):** The cartridge self-locates its own die. The nozzle die is supported and positioned by molded features of the shell — the headland's wall openings, support surfaces, and central peninsulas — so die-to-datum registration is fixed by the mold, not by assembly technique. [HP] Registration variance is therefore a manufacturing defect rather than an expected quantity: a conforming cartridge places its die where the geometry says it goes. Machines do not store per-cartridge registration offsets, and cartridge replacement is not a calibration event.

The die-to-datum relationship is located by internal moulded features that cannot be reached, measured, or designed around from outside the shell. The assertion above is therefore **untested**. §10 records the test.

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

**Design note (non-normative):** the shell is compliant. Measured dimensions vary with caliper jaw force, and the same compliance appears under clamp preload. Flying height is therefore a function of clamp force as well as of geometry, and preload may need to become a specified quantity rather than an implementer's choice. §10 records the test.

## 4. Electrical Personality 1: TIJ 2.5 Pad Film — *Mixed*

### 4.1 Status — *Stable*

The TIJ 2.5 flex-film pad array on face AB, exactly as found on the HP 45-class cartridge, is **Electrical Personality 1** and is the *complete* electrical interface of this revision. [HP]

This specification adds nothing to the film, removes nothing from it, and reinterprets nothing on it. A conforming machine drives it per documented TIJ 2.5 practice; a conforming Class C head presents it per factory convention.

### 4.2 Personality 1 is defined by reference — *Stable* [HP]

Electrical Personality 1 is the TIJ 2.5 interface as produced by the existing HP 45-compatible cartridge industry. That industry is mature, multi-vendor, and in volume production. This specification does not define Personality 1, does not restate it, and claims no authority over it.

Firing voltage, pulse energy and timing, nozzle-to-driver mapping, thermal sensing, calibration, and every other drive property of a Personality 1 head are properties of that ecosystem. A machine implementing Personality 1 conforms to existing practice, not to this document.

What this specification records is the **contact position address space** — which positions exist on the film, where they are, and what Personality 1 natively uses each of them for. It is recorded so that positions can be named unambiguously — §4.5 names two of them, and future revisions may need to name others. See Appendix A.

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
| Single-column height, lowest pad bottom to highest pad top | 23.5 |
| Film width | UNDEFINED — the film carries lateral tolerance and is not a datum |

The array is **centred on Z** and is **mirror symmetric** about that centreline.

Fifty-two pads in eleven rows. Rows 1–7 carry two columns per side; rows 8–11 carry three. The wider 0.6 row separation begins exactly where the third column appears.

All values are caliper measurements of reference hardware except the **highest pad edge**, which is **constructed** — 14.5 + 23.5 + two chevron steps — and whose caliper check reads 38.3, within §0.2's observed spread.

**Derived, for checking:** gap between opposed mid-column positions is 8.9 in rows other than 7; in row 7 it is 8.6, because the 0.45 column separation displaces positions 27 and 28 inboard by 0.15 each. Gap between opposed inner-column positions is 5.3.

**Measurement note (non-normative):** a flatbed scan was used to establish structure — which positions exist, in which columns, in what pattern — and contributed no dimension. Scanner geometry is not calibrated and the instrument's motion blur inflates measured feature extents along its travel axis while leaving centroids unaffected, which makes it useful for counting and for relative layout and useless for size.

**Design note (non-normative):** the row separation widens where the column count increases, which is consistent with trace clearance: the upper rows route three columns' worth of conductors through the same film width. Row 7's 0.45 column separation displaces positions 27 and 28 inboard by 0.15 each — see §4.5.

### 4.4 Contact pads are control pads — *Stable* [DROPSpec]

Throughout this specification the film contacts are termed **control pads**, not firing pads. The name is deliberate: the pads are the machine's control interface to *a head*, of which the thermal-resistive TIJ 2.5 implementation is version one. The specification's compatibility promise attaches to the contact geometry and the negotiated personality — not to the assumption that a resistor lives behind every pad. A negotiated non-thermal personality takes its data and encoder passthrough through the TOOMAH instead, and presents no conductive material on the film at all (§5.5); the film remains the control interface of the thermal personality that defined it.

### 4.5 Head identification — *Stable*

A conforming Class C head implementing Electrical Personality 1 carries a **thermal sense resistor (TSR)** at contact position 27 and a **10× resistor (R10X)** at position 28, per Appendix A. [HP]

A machine MAY read these for per-head energy calibration and for head-presence detection.

A head declaring an electrical personality other than Personality 1 presents no conductive material on the film at all (§5.5), positions 27 and 28 included. A machine probing this circuit therefore reads an open for any head that is not a Personality 1 head, and SHALL treat that open as a valid negative result rather than a fault.

Machines SHOULD detect head removal and insertion events. A machine SHALL NOT require a registration calibration on insertion.

**Design note (non-normative):** A Personality 1 head reports no per-nozzle state. The thermal sense resistor reads substrate temperature, not individual nozzles, and nothing in the head can indicate a failed jet. Nozzle condition is observable only by printing and looking; what a machine does about that is machine business and outside this specification.

**Design note (non-normative):** positions 27 and 28 sit in row 7, the only row using 0.45 column separation, which places them 0.15 mm further inboard than their columns elsewhere. The two positions the specification already treats as special are the two the mould also treats as special. Whether that is deliberate on HP's part is unknown; it is recorded because it is measurable and because someone reading a pad map will otherwise take it for an error.

## 5. The TOOMAH (Auxiliary Interface) — *Mixed*

### 5.1 Definition and geometry — *Draft*

The auxiliary interface is the **TOOMAH**: a plastic receptacle block standing **8 mm proud of face AB**, on the same face as the Personality 1 control-pad film, carrying a **USB-C receptacle** and **four blade receptacles**. [DROPSpec]

The name is a proper noun, not an acronym. The TOOMAH is, definitionally, a growth on the side of a standard cartridge. It is not a tumor.

| Parameter | Value | Grade |
|---|---|---|
| Face | AB | settled |
| Footprint on AB | 12.0 wide × 15.0 tall | proposed |
| Proudness | 8.0 | settled |
| Top edge, below the upper fillet tangent | 3.0 | proposed |
| USB-C receptacle position | transverse, at block top | settled |
| Longitudinal blade count | 4 | settled |
| Blade length | equal, value **TBD** | proposed |
| Blade arrangement | four longitudinal, below the receptacle | settled |
| Blade family (commodity, multi-vendor) | **TBD** | |

Resulting clearances on AB: 17.25 to the film top edge, 21.25 to the topmost pad, 3.40 to each edge of Z, and 59.65 mm of AB below the block — of which 45.85 mm lies above the no-load zone and is available to the X clamp (§3.4).

| Contact | Function |
|---|---|
| **USB-C receptacle** | Power by USB PD contract; data at USB 2.0. VBUS follows USB-C source rules (§5.3, §5.4). |
| **Blades 1–4** | Machine gantry wheel-encoder quadrature passthrough. Logic-level; assignment TBD. |

Mate and break sequencing is inherited from the USB-C connector standard — ground-first pin staggering is a property of the connector, not of this document. The four quadrature blades carry logic-level signals only and are **equal length**; no sequencing requirement attaches to them.

**Design note (non-normative):** the 8.0 proudness is set by USB-C engagement. A mated plug requires approximately 6.5 mm of insertion depth; at the former 5.0 the plug body would have ended its engagement roughly 1.5 mm inside the stall envelope. At 8.0 the full engagement completes with margin before the stall face, and the mated overmold never touches the shell. The proudness remains an out-of-plane standoff from the film plane, per §5.3. Drive power now arrives through the connector's own conductors under a PD contract; a head with a fast-edged driver will still want local bulk capacitance at the receptacle.

### 5.2 Engagement — *Draft* [DROPSpec]

**The head does not mate on insertion.** The cartridge seats and clamps on AB and AD first; the machine-side contact carrier then advances into the TOOMAH receptacle.

A conforming machine's contact carrier travel SHALL be **linear and parallel to the blade axis** over the engagement stroke.

This is what preserves the connector's own sequencing and wipe. USB-C insertion is an axial engagement: on an arced path the plug enters the receptacle at an angle, wipe geometry degrades, and the connector's internal pin staggering — which this specification relies on for mate order — is no longer exercised as designed. The quadrature blades have the same dependency for wipe, though not for sequencing.

**Design note (non-normative):** separating the two motions means nothing electrical happens until the head is fully seated and clamped, so clamp preload is not fighting connector insertion force. This supersedes draft 0.2.2's requirement that blade insertion force be accounted for in stall preload; that concern applied to a design where mating happened during insertion.

Carrier alignment must keep the plug within the USB-C connector family's own gathering allowance at receptacle entry, with the quadrature blades inside their receptacle lead-ins over the same stroke. The carrier budget against those allowances is TBD and is a measurement, not an estimate.

### 5.3 Keying and protection — *Draft* [DROPSpec]

The TOOMAH's exclusion properties rest on **two geometric layers and one electrical one**. This revision states them separately because they are not equivalent and draft 0.2.2's presentation of three independent mechanical layers no longer holds.

1. **Head side (geometric, operator-independent):** the TOOMAH's position and 8 mm proudness SHALL interfere with the reference (TIJ-only) stall envelope, such that a TOOMAH-bearing head cannot seat in a stall lacking the auxiliary slot. A high-voltage head in a thermal-drive stall is not forbidden; it is *unconstructable*.
2. **Machine omission (geometric):** a machine MAY omit the auxiliary system entirely; omission mechanically rejects TOOMAH-bearing heads by rule 1, which is the intended behavior.
3. **Electrical interlock:** VBUS follows USB-C source rules — de-energized until sink attach is detected, and above-default voltage only under an explicit PD contract (§5.4). An exposed contact is not a live contact.

**The machine-side blades are protected by a manually removed cap**, not by a shutter. The cap provides **ingress and contamination protection** — it keeps debris and ink aerosol out of an unused slot — and it is **not an interlock**, because it depends on an operator having refitted it. Draft 0.2.2 claimed a shutter opened by conforming geometry, which would have excluded a finger or a dropped fastener regardless of machine state. It was dropped on cost. Nothing in this revision claims that guarantee.

What replaces it is rule 3. A machine SHALL NOT present VBUS on an unmated slot, and SHALL NOT offer above-default voltage outside a PD contract with an enumerated head — behavior inherited from the USB-C source rules rather than invented here. This is the stronger guarantee of the two in any case, since it does not depend on a mechanical part being present and correct.

**Design note (non-normative):** rule 1 does not address a Personality 1 head in a TOOMAH-equipped stall, and nothing needs to — §2 requires such a head be accepted in every slot, and it seats normally. An erroneous carrier advance against it is a null operation by construction: the receptacles the plug and blades are sized to enter live inside the block, 8 mm proud of AB, so with no block present the plug and blade tips end their stroke in free air, 8 mm short of the shell, contacting nothing — with VBUS absent by rule 3 regardless. The 8 mm proudness is therefore a standoff as well as a key. The property belongs to the carrier's stroke: a carrier whose travel could push the plug or blade tips past the receptacle entry plane forfeits it. Do not build one.

### 5.4 Handshake — *Draft*

The handshake is **USB enumeration**. The head presents as a USB 2.0 device on the TOOMAH's receptacle; power negotiation is **USB PD** on the same connector. What 0.3.1 reserved as an I2C protocol to be invented is re-founded on a mass-market standard whose sequencing, electrical rules, and failure behavior are already specified, implemented, and tested at scale. The protocol problem this section deferred no longer needs inventing — only profiling.

The following is normative and is not deferred:

- VBUS SHALL follow USB Type-C source rules: no VBUS until sink attach is detected, default VBUS only until a PD contract exists, above-default voltage only under an explicit PD contract.
- A head SHALL enumerate and declare its electrical personality and drive power requirements before requesting an above-default PD contract.
- On loss of device presence the machine SHALL revert the port to the unattached state — a backup behavior, since physical removal breaks the connector first.
- Enumeration and default-power operation are the only electrical activity permitted on a non-negotiated TOOMAH.

Device class, descriptor and parameter set, and the PD power profile ceiling a head may request are **UNDEFINED** in this revision. **Deferral reason (unchanged in spirit):** a profile invented with no implementer gets details wrong in ways nobody discovers until someone tries to use it; it will be specified when a real head needs it.

### 5.5 Film requisition — *Withdrawn* [DROPSpec]

The requisition mechanism of drafts 0.2.2 through 0.3.1 is **withdrawn**. Data and encoder passthrough — the two functions requisition existed to carry — live on the TOOMAH: data on the USB-C receptacle, encoder quadrature on the four blades (§5.1). No film position is requisitionable under any personality.

A head declaring an electrical personality other than Personality 1 **SHALL NOT** present conductive material at **any** contact position on the film. All fifty-two machine-side contacts land on insulator.

Positions 27 and 28 remain bare on such a head by the general rule rather than by carve-out, and §4.5's presence-probe logic is unchanged: a machine probing the TSR circuit reads an open, which unambiguously means *not a Personality 1 head*.

**Design note (non-normative):** withdrawal simplifies the fault-tolerance story rather than weakening it. Under requisition, an erroneously driven film was safe because the requisitioned positions were chosen from the address class and carried no firing current. Under withdrawal there is no contact to deliver anything to; the property no longer depends on which class of position was chosen, because no position is presented.

**Design note (non-normative):** the 0.3.1 requisition rationale — address positions only, the top-9.0-mm proximity argument, the ten-position budget — is preserved in that draft for the record. It was sound; it is no longer needed.

### 5.6 Personality 1 obligations — *Stable*

A Personality 1 head has no TOOMAH and presents no auxiliary contacts; no head-side rule is required beyond conforming to the envelope. It seats normally in a TOOMAH-equipped stall (§2); an advanced carrier finds no block there and its plug and blades reach nothing (§5.3 design note). A machine SHALL NOT drive the auxiliary interface beyond USB-C attach behavior, nor drive any film position outside Personality 1 practice, in the absence of a completed handshake.

## 6. Service Interaction Requirements — *Draft*

A conforming head SHALL tolerate the machine's default service lifecycle: [DROPSpec]

- **Capping** as the idle-default state: the nozzle plate region will be engaged by an elastomer gasket whenever the machine is idle, for indefinite durations.
- **Spitting**: firmware-initiated maintenance ejection into a spittoon. Heads whose contents must not be spat (none are currently contemplated) do not conform.
- **Wiping**: a solenoid-actuated, spring-loaded silicone wiper blade makes a straight pass across the nozzle face at the maintenance position, with the head clamped. **Head features SHALL NOT obstruct a straight wipe pass across the nozzle face.**

The head-side obligation is unchanged from draft 0.2.2's manual wiping; only the mechanism differs. The wipe access volume in §3.6 is correspondingly better defined: it is the blade's swept volume, and a solenoid throw has a known stroke, where hand clearance was an estimate.

**Design note (non-normative):** the wiper blade is deliberately not a consumable. Silicone does not absorb; ink sits on the surface and is removed with a wet wipe. An automated service station is precisely where a proprietary consumable would otherwise be reintroduced into an otherwise open supply chain, and the reference design does not acquire a part number here. Aqueous ink means water is the solvent of choice; avoid ammonia-based cleaners near the nozzle plate.

**Reference-implementation note (non-normative):** early DDP-01 prototypes will likely wipe manually while the service station matures. The head-side obligation is identical under either regime: tolerate a straight wipe pass with the head clamped.

## 7. Umbilical Interface (Class T) — *Draft*

### 7.1 Exit zones

A Class T head's umbilical SHALL exit the envelope through one of two conforming zones. Exit from the broad faces, bottom region, or face AB is non-conforming: the broad faces belong to adjacent slots in the gang, the bottom to the nozzle plate, capping gasket, and wipe path, and AB to the control pads and the TOOMAH. Both conforming zones are per-slot symmetric — tanks work in any position (§2).

**Zone H — through the handle void.** [HP] The factory convention: HP's own C6119A bulk ink delivery system routes its supply tube through the thumb-handle opening, with the handle loop serving as captive routing. Natural fit for overhead, Bowden-style drape. Heads using Zone H inherit compatibility with factory HP bulk hardware conventions.

**Zone F — face AG, below the thumb grip.** [DROPSpec] A defined band on face AG immediately below the grip's lower extent. Natural fit for drape toward the machine's maintenance access window, where the operator already works.

| Parameter | Value |
|---|---|
| Zone H clearance envelope through handle void | TBD |
| Zone F band vertical extent (below the grip's lower extent) | TBD mm |
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

- Changes to a section marked **Stable** require a committee vote of at least 2/3rds majority to reopen, followed by a version increment.
- The review process binds all committee members, **including the author**.
- The committee is presently one person. The mechanism is therefore formal rather than operative.
- The numbering scheme itself, and the personality-numbering scheme for future electrical personalities, are **UNDEFINED** in this revision.

**Managing the Governance:** The creator of this specification, David Norman Brooks, is the initial and sole committee member until the eventual onboarding of other members. During the initial development of this standard through version 2.0 or 2 years after version 1.0 of the specification, whichever comes first, he will be the guarantor of standard integrity on this project, provided a future committee charter is established. What this means:

- David Norman Brooks must divest his guarantor position upon the sunset provision unless a committee charter cannot be established. At that point, he will offer an alternative to be voted on within 30 to 60 days, after which he must fully divest his guarantor position.
- Members must ***not*** attempt corporate capture of this specification.
    - This means that attempts to single-source parts will not be tolerated (Ignoring the obvious single source nature of TIJ 2.5, which is off-patent)
    - This means that attempts to wrest control of the standard from the community will not be tolerated.
- The guarantor will make determinations regarding accusations relating to the above. The committee member who is suspected of wrongdoing will be allowed to make their case in a closed session of the committee.
- Penalties will include the following:
    - Forced recusal from specific actions
    - Removal from the committee
- Penalties will result in a public release detailing the action leading to the penalty and the rationale behind the consequence.
- Version 1.0's automatic creation will be detailed to occur at a specific threshold, not contingent upon committee direction.

Eventually, it is Mr. Brooks's desire to have the committee self-govern. This means that he will be focused on eventually divesting himself of the guarantor role.

## 10. Open Items

**Measurement**

- Maximum mass for any conforming head (§3.2)
- Flip-scan or equivalent to establish maximum envelope under draft, since a contact-plane section reads smaller than the maximum (§3.2)
- Tolerance band for datum features, requiring samples beyond a single SKU. Both reference articles examined are C8842A; mould codes should be compared before any spread is treated as characterising rather than understating (§3.4)
- Whether clamp preload moves flying height, via shell compliance. Vary preload, print, observe (§3.6)
- Die-to-datum repeatability: print a registration pattern from two cartridges in the same stall, swap, reprint. Disagreement beyond a nozzle pitch falsifies §3.4's registration assertion (§3.4)
- Grip lower extent: measurement schedule letters H and I were withdrawn (grip bottom undefined); Zone F cannot be dimensioned until it is established (§7.1)
- Whether EXC-1's 0.50 mm projection past AB lies wholly above the 77.65 tangent; if any of it lies below, §3.1's flatness statement acquires an explicit carve-out (§3.1, §3.5)

**TOOMAH**

- Blade family for the four quadrature blades — commodity, multi-vendor (§5.1)
- Blade length, and the carrier alignment budget against the USB-C family's gathering allowance and the blade lead-ins (§5.1, §5.2)
- Quadrature electrical specification: single-ended or differential, logic level, and pair assignment across the four blades (§5.1)
- PD power profile ceiling a head may request; USB device class, descriptors, and parameter set (§5.4)
- USB-C receptacle grade and retention spec for a scanning-carriage duty cycle (§5.1)
- Whether rule 3 plus the 8 mm standoff fully replace the deleted shutter — believed yes; held open until the carrier design confirms its stroke cannot carry the plug or blade tips past the receptacle entry plane (§5.3)

Resolved since 0.3.1: tangent clearance (block lowered 2.0 mm, clearance now 3.0 — §5.1); the clamp-band consequence is recorded in §3.4. Superseded: engagement stagger lengths (connector-internal), vH/vL envelopes and the I2C protocol (USB PD and USB enumeration — §5.4).

**Other**

- Exit zone dimensions for Zones H and F, and the shared protrusion limit; Zone H clearance to be captured by measurement from C6119A factory hardware (§7.1)
- Keep-out volume dimensions pending service station design freeze (§3.6)
- Reconciling EXC-2's exclusion with capping and wipe access (§3.5)
- Film width, and whether it needs stating at all given it is not a datum (§4.3)
- Versioning and personality-numbering scheme (§9, Reserved)

## Appendix A — Contact position index and geometry

Contact positions and their native Personality 1 functions, after US 6,332,677 B1 (Steinfield et al., Hewlett-Packard) Table II. [HP]

The position/function table is informative. It is reproduced so that positions can be named unambiguously (§4.5). Nothing in this specification governs what a Personality 1 head or machine does with any of these positions. **The layout in §A.2 is normative**, with dimensions per §4.3, since a stall connector is built to them.

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

The provisional Figure A-1 circulated alongside the 0.2.x drafts carries that revision's eight-position requisition outlines and is superseded by this section and §4.3; the requisition mechanism itself is withdrawn (§5.5).

### A.3 Contact position dimensions

Dimensions for the pad field are normative in **§4.3** and are deliberately not restated here: a single copy cannot drift against itself.

## 11. License

CERN-OHL-S (hardware definitions) / CC BY-SA 4.0 (this document), per project licensing. Final notice text TBD.

---

*Draft 0.3.3 (proposed). Geometry gospel; contents wild west. Comments, measurements, and dissent welcome via the project repository.*
