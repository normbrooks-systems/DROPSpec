# DROPSpec

**D**eposition **R**aster **O**pen **P**rint **S**tandard — an open standard for putting industrial inkjet on open motion platforms, with a desktop press as its reference implementation.

## Status

Design phase, as of July 2026. This repository launches with three things: this readme (the *why*), a pair of glossaries (the *translation layer*), and the head interface specification (the *contract*). The machine itself follows as the reference implementation of that contract.

## The Problem

Consumer inkjet is a hostile ecosystem: chipped cartridges, region locks, firmware that disables third-party ink, hardware priced as a loss-leader for a fluid subscription. The designs themselves are mostly remixes on solved technology, sold with 1-picoliter party tricks that exceed what most duties require, at prices that reflect the lock-in rather than the engineering.

Meanwhile, industrial coding and marking has run the same fundamental technology for over twenty-five years in the opposite direction: chipless commodity cartridges, a competitive multi-vendor ink industry, and operator-serviceable equipment — because industrial customers would never tolerate anything else. In that world, this is legitimately a solved problem.

These two worlds barely communicate. DROPSpec is the bridge.

## The Approach

**The head.** HP's TIJ 2.5 technology (the HP 45 cartridge class) has been in industrial production since the late 1990s and has been properly reverse-engineered in the open (the YTEC HP45 lineage). The cartridges are chipless, hold 42 mL, and routinely sell under $25 for versatile black. The ink catalog behind them is an entire industry's: aqueous and solvent, dye and pigment, CMY process sets from first-party partner suppliers, plus specialty chemistries — MICR, food-grade, UV-fluorescent emissives — that consumer printing has never offered.

**The gang.** Four cartridges on a scanning shuttle, staggered half an inch apart *vertically* — along the media axis — in the standard construction of the industrial 2-inch ganged print head, used here as the commercial article it is: the four pens span a full two inches of page height. This one geometric choice defines both operating modes. In mono, each black pen owns its own half-inch band: a stitched ~2-inch swath per pass, the fast workhorse mode. In CMYK, the media advance walks every band of the page under each pen in turn, so any given spot of paper receives its colors on four successive passes, always in slot order, with a full pass period of dry time between inks. Color lay-down order is therefore fixed by the aluminum, not the firmware: printing is bidirectional at full speed in every mode, with no direction-dependent hue shift — the artifact consumer inkjet drivers quietly slow down to avoid cannot occur on this architecture. Switching modes is a cartridge swap and a settings change.

**The control.** Motion is Klipper — used, not forked — with the standard ecosystem (TMC drivers, Moonraker, web UI) intact, so the machine is defined in a printer.cfg any Voron builder can read. Nozzle firing is *not* trusted to step counting: a sealed encoder wheel riding a raceway on the gantry — the same friction-wheel position reference that fires industrial coders on conveyor lines — is the position truth, and drops fire from encoder edges. The steppers move the carriage; the encoder tells the truth.

**The leverage.** HP cannot abandon this platform, because its primary customers are industrial clients who buy these cartridges and inks by the case. The supply chain's stability is guaranteed by customers with far more purchasing power than any consumer — we simply ride behind them.

## Safety, Up Front

DROPSpec's ink freedom includes solvent, MICR, UV-reactive, and other industrial chemistries. Aqueous ink is about as safe as printing gets. Everything else is a different matter: **you are stepping into an industrial environment the moment you load a non-aqueous solution.** These fluids ship with safety data sheets and handling requirements written for production floors — ventilation, gloves, storage, disposal. Read, and more importantly, ***obey***, the safety language. This platform hands you the industry's catalog; the industry's responsibilities come with it.

As a side note, the author of this document also wants you to know this:
>[!CAUTION]
>*Regulations are written in blood, and some of it is my own.* 

Do with that knowledge what you will.

## The Motion Platform

Numbers worth stating precisely, because they set the machine's structural class:

- **Head assembly mass:** a filled TIJ 2.5 cartridge is 120–125 g; the four-gang alone approaches 500 g, and the full printhead assembly is comfortably a kilogram before pens. Design load for the scan axis is ~2 kg.
- **Gantry:** specced for 4080 T-slot extrusion, because 2020 does not get to throw two kilograms around at a meter per second. A side benefit: industrial TIJ head units ship with 40-series T-slot mounting, so commercial print hardware bolts to this gantry natively.
- **Linear motion:** MGN-class linear rail is the scan-axis baseline, not an upgrade. Wheel-in-slot options are not rated for this duty.
- **Speed, honestly stated:** the gantry is structurally specced to 1 m/s. Printing speed is bounded by firing frequency, and the anchor is industrial practice: TIJ 2.5 coders routinely run 50 in/s at 300 dpi — a 15 kHz sustained firing rate. The pen counts firings, not dpi, so the same 15 kHz at 600 dpi column pitch yields ~25 in/s (~635 mm/s) — inside the gantry's structural envelope with margin. Datasheet-nominal figures (~12 kHz) are the conservative floor; one honest caveat is that dense raster coverage is a harder thermal duty than the sparse text industrial coders typically print, so full-coverage sustained speed awaits bench verification. Any document should still quote travel speed and print speed as separate numbers.

## The Platform Argument

The head interface specification's rule is: **the geometry is gospel; the contents are the wild west.** The spec defines the cartridge envelope, datums, contact interface, and umbilical geometry as stable and normative — and deliberately says nothing about what lives inside the shell. A factory black cartridge conforms. A refilled fluorescent conforms. Things not yet built conform, the day someone builds them to the envelope.

That opens a branch space — possibilities the contract permits, not commitments on a roadmap:

- **Direct-on-object printing** — a DROPSpec head on a Voron-class gantry, marking and decorating FDM parts.
- **Binder jetting** — the aqueous head plus a powder bed is the jetting half of the open binder-jet machine the 3D printing world has wanted for a decade.
- **TiO₂ white and other head-hostile fluids** — the nightmare inks of production inkjet become tractable when the entire print engine is a $15 consumable.
- **Future electrical personalities** — the spec's TOOMAH interface defines the power delivery, handshake, and keying for head technologies beyond thermal, without ever naming any of them.
- **Business forms and quick printing** — spot blue or spot red alongside black turns the mono gang into a forms press for specific business contexts.
- **Additive emissive art** — RGB UV-fluorescent ink sets exist in this cartridge format today, making wall art that emits its image rather than reflecting it: printing that directly mimics a sensor's capture rather than fighting it.

The branches get walked when someone shows up with the itch. The trunk — open envelope, open firing stack, open motion integration — is this project.

## Repository Contents

- `dropspec-head-interface-draft-0.1.md` — the head interface specification (draft): the core document of the standard
- `glossary-press-to-voron.md` — press and inkjet vocabulary for motion-platform people
- `glossary-voron-to-press.md` — motion-platform vocabulary and culture for print-industry people

## Roadmap

See the following file:
- `deliverables-reference-implementation.md`

## Contributing / Contact

normbrooks.systems@gmail.com

## License

- **Hardware:** CERN-OHL-S
- **Software:** GPLv3
- **Documentation:** CC BY-SA 4.0

Commercial use is explicitly welcome. Kit vendors, ink suppliers, mod authors, and service businesses are the point, not a problem.
