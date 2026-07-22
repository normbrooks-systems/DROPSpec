# Press → Voron: A Glossary for Motion-Platform Refugees

*Part of DROPSpec. You know what a MGN12 rail is. You do not know what decap is. This document fixes that.*

You arrive fluent in steppers, belts, and Klipper. The half of this machine you don't know is the half that touches ink — a vocabulary built over five centuries of presswork and fifty years of inkjet engineering. Each entry gives the term, what it means, the nearest thing in your world, and — most importantly — where the analogy *breaks*, because false friends cause worse crashes than total strangers.

## The Head

**Pen / cartridge / head.** Used near-interchangeably here for the HP 45-class TIJ 2.5 cartridge. "Pen" is HP's own internal jargon and industrial habit. The critical mental shift: this is your hotend *and* your material supply *and* most of your precision, in one $15 disposable object. When it wears out, you throw away the entire print engine and clip in a new one. There is no nozzle to unclog with a needle. The consumable *is* the machine.

**TIJ (thermal inkjet).** Drop ejection by micro-boiling: a resistor under each nozzle flash-vaporizes a film of ink, the bubble kicks a droplet out, the bubble collapses, capillary action refills. Millions of tiny steam explosions per second. Nearest analog: none, and that's the point — there is no motion-system equivalent of the fluid being the moving part.

**Stall.** The catalog name for the spring-loaded holder a pen clicks into: a steel fixture whose latch preloads the cartridge against fixed datum surfaces, carrying the pen's electrical contacts with it. Gang stalls in a half-inch-stepped stack and you have the industrial 2-inch print head — the commercial assembly DROPSpec mounts on its shuttle. Nearest analog: a toolhead dock, if docking were the whole job. The telling detail: production floors don't actually *say* "stall" — operators just say "pen 1" through "pen 4," because the pens are what you touch and swap while the fixture never fails, never drifts, and therefore never needs a name. Infrastructure that works disappears from vocabulary; that linguistic invisibility is decades of registration reliability, audible as silence. DROPSpec inherits the dialect with one ambiguity engineered out: multi-head installations never agreed on whether head 2's first pen is "pen 5" or "head 2, pen 1," so DROPSpec's canon is hierarchical — head N, pen 1–4 — with the single-head machine's "head 1" going politely unsaid. You keep saying "pen 3"; the config file always knew which head you meant.

**TIJ 2.5.** The specific HP head generation this platform standardizes on — the HP 45 cartridge class, in industrial production since the late '90s. Think of it the way you think of NEMA 17: not the newest thing, but the *standardized* thing, with a massive multi-vendor ecosystem precisely because it stopped changing.

**Nozzle plate.** The gold-orange face on the bottom of the cartridge with 300 laser-drilled holes. Treat it like you treat a PEI sheet's surface: the sacred face. Touch it with anything other than a lint-free wipe and you will regret it.

**Drop volume (picoliters).** The inkjet equivalent of layer height as a quality driver. This head fires ~30 pL binary drops — "binary" meaning fire or don't-fire, no in-between. Consumer photo printers do 1–4 pL with variable sizes; we trade drop finesse for speed, ink freedom, and industrial durability.

**Primitive / address multiplexing.** How 300 nozzles are driven over ~50 pads instead of 300 wires: nozzles are organized in a matrix (groups called *primitives*, selected by *address* lines), fired one address at a time, fast. Same idea as a multiplexed LED matrix or keyboard scan. This is why the firing electronics are "a modest electrical problem, not 1,200 discrete channels."

**Calibration resistor / pen ID.** A resistor built into each cartridge that tells the machine what firing energy this specific head wants. The machine reads it and trims the pulse. Nearest analog: reading a thermistor table for a hotend — per-unit calibration, done electrically, automatically.

## Keeping the Head Alive (the part with no Voron analog)

**Decap.** *The* word to learn first. Aqueous ink in an open nozzle starts drying in **tens of seconds**, skinning over and killing jetting. Decap time is how long a nozzle survives uncapped and idle. There is no FDM equivalent — filament doesn't cure in the nozzle while you go make coffee. Decap is why the service station exists and why it's described as the graveyard of DIY inkjet.

**Capping.** Sealing the nozzle plate against an elastomer gasket whenever idle, making a tiny humid chamber where ink can't dry. On DROPSpec this is passive and automatic: parked means capped, always. Mental model: it's the Z-endstop of ink — not a feature, a survival requirement.

**Spitting.** Firing waste drops into an absorbent-filled bin (the **spittoon**) to purge nozzles that have begun to dry. Pre-print spits, mid-job maintenance spits. Analog: a prime line or purge bucket — except mandatory, scheduled by firmware, forever.

**Wiping.** Physically cleaning the nozzle face with a lint-free wipe. Industrial TIJ operators do this by hand as ritual; DROPSpec keeps it manual but in-place so the cartridge never leaves its clamp. Analog: cleaning your nozzle with a brass brush, if skipping it bricked your printer by Monday.

**Service station.** The parking structure implementing all of the above: cap sled, spittoon, maintenance position. Analogous in spirit to a toolchanger dock, but its job is keeping fluid alive, not swapping tools.

**Kogation.** Baked-on residue accumulating on the firing resistors over a head's life — one reason heads are consumables. No analog; closest vibe is nozzle wear from abrasive filament, but it's chemical, not mechanical.

## Putting Ink Where You Meant To

**Swath.** The band of image laid down in one carriage pass — this machine's fundamental unit of work, as the layer is FDM's. Four ganged heads: ~2-inch swath in mono.

**Stitching.** Making adjacent swaths meet invisibly. The precision problem of this machine, as first-layer adhesion is FDM's. A seam error of one nozzle pitch (~42 µm) is visible; your eye is a crueler metrology instrument than any caliper.

**Interleave.** Splitting raster rows across the four ganged heads (mono mode) or across multiple passes (quality mode) so their outputs mesh like fingers. Loose analog: interlaced video, or splitting perimeters across extruders on an IDEX — but at 42 µm registration.

**Registration.** Any alignment of one printed thing to another: head-to-head, color-to-color, pass-to-pass, front-to-back. Where you say "offset calibration" for a probe, a press operator says registration — and has said it since Gutenberg.

**Encoder strip / encoder wheel.** In consumer inkjets, position truth is a clear plastic strip with fine dark stripes stretched along the scan axis, read optically — and firing is triggered from its edges, *not* from step counting. This is the load-bearing difference from your world: in FDM the steppers are the truth; in inkjet **the encoder is the truth** and the stepper merely provides motion. Open-loop firing is how DIY inkjets die. DROPSpec keeps the principle but swaps the implementation for the coding industry's version: a sealed encoder *wheel* riding a raceway on the gantry — the same friction-wheel reference that fires industrial coders on conveyor lines — because exposed strips are slowly blinded by ink aerosol, and sealed optics with a mechanical contact patch are not.

**Media advance.** The paper-feed axis step between swaths — this machine's Z-hop-equivalent in importance. Advance error appears directly as stitch banding. The tightest tolerance in the machine (~21 µm in quality mode) lives here, not on the scan axis.

**Cockle.** Paper warping from absorbed water as aqueous ink hits it — the sheet buckles *while you're printing on it*, into a 1–2 mm flying height. Analog: warping, except mid-print and in seconds. The vacuum platen exists substantially because of this word.

**Flying height / head-to-media gap.** 1–2 mm, and quality dies beyond it. Analog: Z-offset, except across the *entire sheet at once*, enforced by vacuum instead of mesh compensation.

## The Machine's Paper-Handling Vocabulary

**Media / stock.** What's being printed on. "Media" is the machine's word; "stock" is the press world's ("card stock," "coated stock"). You say "build surface"; they say substrate, media, or stock depending on how long they've worked in the trade.

**Pinch roller / grit roller.** The drive system metering the sheet: a shaft with an abrasive surface (grit) pinching the media against idlers. The grit bites paper like a GT2 belt bites a pulley — this is the media axis's "belt," and its repeatability is why it's trusted with tens-of-microns advances.

**Star wheels (spur wheels).** Toothed wheels touching freshly printed ink downstream on points so tiny they leave no mark. The answer to "how do you pinch wet ink."

**Platen.** The flat surface under the print zone. Yours is heated and called a bed; this one is perforated, vacuum-drawn, and web-covered. Same job: hold the workpiece flat.

**Full bleed.** Printing to all four edges with zero margin — requires printing *past* the sheet edge and catching the overspray. The reason the vacuum web doubles as a gutter. Press people consider bleed table stakes; consumer printers mostly can't do it. No FDM analog; nobody demands prints overhang the bed.

**Duty cycle.** How hard and continuously the machine can run. Industrial TIJ runs production lines 24/7; consumer printers are engineered for bursts. Same word as motor/heater duty, applied to the whole machine's stamina.

## Ink Itself

**Aqueous / solvent.** The two great ink families: water-based (safe, cheap, needs porous media) and solvent-based (bites into plastics and coated surfaces, needs ventilation). Loose analog: PLA vs. ABS — the friendly default vs. the one with fumes and superpowers.

**Dye / pigment.** Colorant dissolved in the fluid (dye: vivid, fade-prone) vs. solid particles suspended in it (pigment: durable, lightfast). No motion-world analog; it's chemistry. Matters because it drives decap behavior, media compatibility, and longevity.

**Fast-decap formulation.** Ink engineered to survive longer uncapped. Exists because industrial coders sit idle between products on a conveyor. One of many ways ink chemistry, not hardware, defines capability on this platform.

**CMYK / process color.** Cyan, magenta, yellow, black — the subtractive primaries. Full color from four inks by halftoning. The K is "key," a letterpress term.

**Spot color.** A single premixed ink printed as itself rather than built from CMYK — industrial TIJ's native mode (lot codes don't need color management). DROPSpec's mono mode is spot-color thinking; its CMYK mode is process thinking.

**"I blue up my hand."** Not a typo. The floor's term for the rite of passage that finds everyone who refills, remans, or unclogs: dye ink does not wash off skin — it *wears* off, over days, and the classic first offender is spot blue on refill day. Your world's equivalents are the brass-nozzle burn scar and the resin-printer glove ritual; ours announces itself in meetings for a week. Nitrile gloves cost four cents. The entry exists so you learn this the four-cent way.

**MICR.** Magnetic ink character recognition — the magnetizable ink checks are printed with. Exists in 45-class bodies today. Mentioned because it's the canonical example of "capability delivered by chemistry, not hardware."

## Making Images Out of Dots

**RIP (raster image processor).** The pipeline turning a page description into per-nozzle firing decisions. The slicer of this world — same seat in the toolchain, same "the software defines the machine" energy.

**Halftoning.** Faking continuous tone with binary dots by controlling dot *density*. The core rendering trick of all printing since the 1880s.

**Error diffusion.** The baseline halftoning algorithm: quantize each pixel, push the error onto neighbors. Produces the characteristic organic dot scatter. Analog in spirit: dithering in retro graphics — because it literally is dithering.

**dpi vs. addressability.** 600 dpi native means 600 possible dot *positions* per inch; it does not mean 600 *distinct* dots — a ~30 pL drop makes a ~60 µm splat. 1200 dpi mode is honest addressability, not optical resolution. Same skepticism you apply to "50-micron resolution!" microstepping claims: positions are cheap, resolvable features are not.

**ICC profile.** A measured color fingerprint of a specific ink-on-media combination, letting software compensate so colors match intent. Analog: filament tuning profiles, if filament tuning had an ISO standard and a measurement instrument.

**Depletion mask.** Rendering-stage limit on total ink per area so heavy coverage doesn't drown the sheet. Analog: max volumetric flow limits — same idea, applied to liquid on paper instead of plastic through a nozzle.

## Words You Already Know That Mean Something Else Here

**Extrusion.** Here: T-slotted aluminum framing, always — 2020 profile for the general frame, a beefier 4080 profile for the gantry that slings the head assembly. Nobody in this project extrudes ink. If a DROPSpec document says "extrusion," someone is talking about structure.

**Purge.** Your purge is priming after a filament swap; ink's purge (spitting) is a *perpetual scheduled obligation*. Same word, different tense: you purge *sometimes*, this machine spits *always*.

**Resolution.** In FDM marketing this word is already abused; in inkjet it's abused *differently* (see dpi vs. addressability). In both worlds, ask "resolvable by whom, measured how" and watch the number shrink.

**Calibration.** You calibrate flow, PID, input shaping — mostly once per machine. This machine calibrates per *cartridge*: a printed alignment measures where each new pen's nozzles actually sit relative to its shell (the same reason HP printers have always run an alignment page on new carts), stored and confirmed by a fast verification print whenever a pen is reseated. Same lifecycle shape as your per-spool filament tuning: per consumable, not per boot.
