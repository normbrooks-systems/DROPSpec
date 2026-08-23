# Voron → Press: A Glossary for Print People Wandering In

*Part of DROPSpec. You know what decap is. You do not know what a Voron is. This document fixes that.*

You arrive fluent in ink, media, and service rituals. The half of this machine you don't know is the half that moves — a motion platform borrowed wholesale from the open-source 3D printing world, along with its parts supply, its firmware, and its very particular culture. Each entry gives the term, what it means, the nearest thing in your world, and where the analogy breaks.

## The Tribes (yes, the project names are load-bearing)

**RepRap.** The 2005-era academic project that started open-source 3D printing, with the founding conceit that the machine should print its own parts. Every open printer since is descended from it. Cultural analog: it is to this world what desktop publishing was to yours — the moment the means of production escaped into civilian hands. When DROPSpec's proposal says "RepRap-style economy," it means that lineage of commons.

**Ender.** The Creality Ender 3 — the cheap, ubiquitous, endlessly cloned consumer 3D printer. Tens of millions of people own one. Its significance to DROPSpec is purely practical: its carriage conventions, parts, and tooling are a commodity supply chain. Analog: it's the machine in this world that the desktop laser printer was in yours — not the best, but the one *everywhere*, whose parts are consequently free-ish.

**Voron.** A community-designed, no-corporation, high-performance 3D printer you cannot buy assembled — you source or kit the parts and build it. Renowned for engineering rigor and documentation quality. The "Voron-style economy": the design is free, and an ecosystem of independent kit vendors, parts sellers, and mod authors makes a living around it. This is the commercial-commons model DROPSpec explicitly wants, and the reason its license permits commercial use.

**Prusa.** The company proving open hardware can be a real business — they publish designs, sell excellent machines, and the community respects both halves. Analog: the closest thing this world has to a beloved incumbent, which in your industry is not a sentence that parses.

**Klipper.** The dominant open-source motion firmware, and DROPSpec's motion brain, used unmodified. Its defining architecture: a small microcontroller executes precisely-timed moves while a Linux computer does all the thinking. Analog: think of it as the machine-control half of a DFE, if DFEs were free, community-maintained, and configured with a text file.

**Mod.** A community-published modification to an open machine design — printed parts, alternate components, documented and shared. Mods are how these platforms evolve between official releases. Loose analog: press options and retrofits, except designed by users, for free, and installed without a service contract.

## Reading the Machine's Spec Sheet

**NEMA 17.** A stepper motor *frame-size standard* (1.7-inch bolt pattern), and shorthand for the commodity motor class every desktop machine uses. It says nothing about torque or quality — only that it bolts into the ecosystem. Analog: a standard cartridge envelope. The point isn't performance; the point is *interchangeable from forty vendors*.

**Stepper motor.** Moves in discrete steps counted open-loop — no feedback, position by faith. Your world's servo people are now wincing: correctly. Which is why DROPSpec fires from a dedicated encoder wheel and demotes the steppers to providing mere motion. The steppers position the carriage; the encoder tells the truth.

**TMC drivers.** The chips that drive the steppers — near-silent, smart, and standard in decent machines. What you need to know: "TMC-class" on a BOM means "the good, quiet, normal choice," not something exotic.

**GT2 belt.** The 2 mm-pitch toothed belt that transmits motion in essentially every desktop 3D printer. Analog: think timing belt in a press drivetrain, miniaturized and available for pocket change by the meter.

**T-slot extrusion (2020 / 2040).** Slotted aluminum framing rail, named by cross-section in millimeters — the Lego of machine frames. 2020 (20×20 mm) is the desktop-printer commodity and forms DROPSpec's general frame; the gantry carrying the head assembly is specced for 2040 (20×40 mm), because up to two kilos of printhead scanning at a meter per second is not a 2020 job. The head assembly's final mass is TBD — a printed gang plausibly lands well under the ceiling — and the gantry selection gets confirmed against the weighed article, not assumed.

**V-slot / wheels.** The budget linear-motion option in the desktop-printer world: plastic-tired wheels rolling in the extrusion's grooves. Cheap, quiet, adequate — for light loads. DROPSpec's scan axis is not one: up to two kilos reversing at speed rides a proper linear rail, and V-slot survives in this project only on lighter duties, if anywhere. Analog: bushings vs. bearings — and this machine needs the bearings.

**MGN12 / linear rail.** The precision option: a ground steel rail and recirculating-ball carriage, 12 mm wide. In hobby machines it's the upgrade path; on DROPSpec's scan axis it's the baseline, because the head assembly's mass and speed left the wheel option behind. If a DROPSpec build sheet names an MGN rail, read "the slide the printhead actually rides."

**Hotend / toolhead.** The business end of a 3D printer — melter and nozzle and its carriage-mounted assembly. DROPSpec borrows the *motion-side* conventions, not the melting: where an Ender's carriage carries a hotend, DROPSpec's shuttle carries a four-pen stall gang — designed in the open and printed, in the industrial configuration, holding commodity industrial cartridges. When motion people say "toolhead," they mean what you'd call the head carriage.

**Endstop / homing.** The switch and ritual establishing where zero is at power-on. Steppers are open-loop and wake up amnesiac; homing is how the machine finds itself. Analog: registration, performed by the axes on themselves.

**Mesh leveling.** Probing a grid of surface heights and compensating in software rather than demanding the surface be flat. Included because it's this community's reflexive answer to "the substrate isn't flat" — relevant the day someone mounts a DROPSpec head over an uneven object.

## The Software Stack

**printer.cfg.** Klipper's single human-readable configuration file, defining the whole machine — motors, geometry, features. There is no vendor service menu; there is a text file you own. DROPSpec machines are defined in one.

**G-code.** The ancient, universal command language of machine motion ("move here at this speed"). DROPSpec uses it for carriage sweeps and media advances while firing rides the encoder. Analog: it is to motion what PostScript was to pages — old, everywhere, outliving its replacements.

**Klippy extras / [tij_printhead].** Klipper's plugin mechanism, and the specific DROPSpec module that arms the firing board and sequences passes. The design point: DROPSpec *extends* Klipper without forking it, staying compatible with the upstream ecosystem forever. Your analog, loosely: an ICC-workflow plugin rather than a hacked driver.

**Moonraker / Mainsail.** The API server and web interface layered on Klipper — job queue, machine control, monitoring, from any browser. Analog: the press console / job front end, free and self-hosted.

**RP2040 / STM32.** Commodity microcontroller families; DROPSpec uses one for deterministic firing-pulse timing. What matters: dollars-cheap, unlocked, programmed by anyone. The contrast with your world's controller boards, priced like organs, is the point.

**SBC / Raspberry Pi.** The credit-card Linux computer hosting the print pipeline and network face. Analog: the RIP station, at 1/100th the price.

**Firmware.** In your world, the sealed thing a service tech flashes. Here: open source, on a public repository, rebuilt by hobbyists before breakfast. Same word, opposite power relationship — and the entire reason this project can exist.

## The Culture's Load-Bearing Vocabulary

**BOM (bill of materials).** The published parts list with sources and prices, treated as a first-class deliverable. An open machine *is* its BOM plus its documentation. Your analog is a service parts catalog — if it were public, priced, and designed for self-sourcing.

**Kit vendor.** An independent business selling the parts bundle for an open design they don't own. Legitimate, encouraged, plural. The existence of competing kit vendors for one design is a *success signal* in this culture, not infringement.

**Printed parts.** Structural components of the machine, 3D-printed — either by you or by the community member selling them. DROPSpec uses printed parts for carriers, clamps, gaskets (in flexible TPU), bezels — and, flagship of the class, the **stall gang itself**: the precision fixture that holds the pens is a printed, parametric part, with a validation program to prove a printed part can hold datum surfaces to the tolerances steel takes for granted. The RepRap conceit, still alive and promoted to load-bearing: machines whose parts reproduce.

**TPU / PETG / filament.** 3D-printing materials, referenced in DROPSpec for specific supporting roles: TPU is the rubbery one (gaskets, tires, web clamps), PETG the tough clear-ish one (and, as sheet, the light-guide substrate for edge-lit panels). "Filament" generically = 3D printer feedstock; it will appear on the BOM as a consumable *for building the machine*, not for printing pages.

**Open source hardware / CERN-OHL.** Design files published under licenses guaranteeing the right to build, modify, and sell. CERN-OHL-S is the copyleft flavor DROPSpec uses: improvements must be shared back. Contrast entry — **NC (non-commercial)** clauses, which this project explicitly rejects because they strangle the kit-vendor economy that makes open machines sustainable.

**Fork.** Taking a project's files and developing a divergent version — a *right*, guaranteed by license, sometimes rude, always legal. Your world's analog is... nothing. That's rather the point of the whole exercise.

## Words You Already Know That Mean Something Else Here

**Web.** You mean a continuous substrate through a press; DROPSpec's transport does use a mesh web, so context-check every use. A Voron person, meanwhile, means a website. Three communities, one syllable, budget accordingly.

**Bed.** Their platen. Usually heated in their world; in DROPSpec it's perforated and vacuum-drawn instead. If a motion person says "bed," point at the platen and you'll both be right.

**Slicer.** Their RIP: software converting a 3D model into machine instructions. DROPSpec replaces the slicer's seat with an actual RIP (CUPS + filter chain). If a motion person asks "what slicer does it use," the honest answer is "its slicer is a RIP."

**Purge / prime.** In their world, a brief startup ritual after changing material. Do not let them assume ink works the same way — decap will teach them otherwise, expensively. (This warning appears in mirror image in the companion glossary. It's that important.)

**Layer.** Their unit of vertical progress. DROPSpec has no layers; its unit of work is the swath. If you hear "layer" around this machine, someone is either talking about the 3D-printed parts — or about the day someone bolts a DROPSpec head onto a Voron, at which point both vocabularies apply at once and God help the documentation team.
