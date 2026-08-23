# .grip — cliff notes

**DROPSpec** (Deposition Raster Open Print Spec): the standard.
**`.grip`**: its job container.
**MDP-01**: reference implementation.

Working notes for the DROPSpec job container. Decisions made, not a finished spec.

## What it is

A zip archive. Extension `.grip` (g-rip: gcode + rip).

```
manifest.ini
command.gcode
rip/
  001_001_01.ppp
  001_001_02.ppp
  001_002_01.ppp
  ...
```

Three members, each with an obvious owner:

- **manifest.ini** — what this job is
- **command.gcode** — the motion plan, Klipper-consumable
- **rip/** — the swaths, in print order

## Naming

`page_pass_pen`

Triple-padded page and pass, so sort order stays correct past 999.
Pen-major within a pass: everything a single pass needs is contiguous in
the archive.

## Swath data

1-bit, **column-major**, one file per pen per pass.

Column-major because that is the consumption axis — one column per fire
event, so the DMA source is a straight sequential read with no stride math.

Extension `.ppp` (page/pass/pen) rather than `.bmp`. If it is stored
column-major it is not a BMP, and calling it one invites someone to open it,
see garbage, and file a bug. A ten-line debug script that transposes to PBM
covers the eyeball case.

## manifest.ini

Minimum fields:

- format version
- page count
- pen count
- total passes
- native dpi — **both axes, separately** (cross-axis is set by encoder ticks
  per fire, down-axis by nozzle pitch; they are rarely the same number)
- per-pass page origin offset (x/y) — lets a renderer place a pass without
  parsing the gcode
- per-tool offset — so anything that isn't the printer can reason about where
  ink lands without reading the Klipper config

## Why the structure is the spec

Everything that would otherwise be a header field is structural:

- 1-bit means screening already happened
- per-pen means nozzle-space
- per-pass means the pass structure *is* the file layout
- gcode in the same container means motion and bitmaps cannot desync

Not underspecified. Decided.

## Arming and sync

Toolchange gcode (`T00`–`T99`) arms the RIP: it names the pen and is already
understood by Klipper, senders, and log readers. The toolchange is standard
vocabulary; only the data plane is custom, and it is invisible to generic
tooling by design.

**T is the control plane. Q is the data plane.**

Queue gcode (`Qxnn`, where `nn` is the pen) — the digit is urgency,
descending:

- `Q1nn` — **load** swath for the next pass. Blocking: does not yield until
  the buffer is real.
- `Q2nn` — **stage** swath for the pass after that. Async: fires the
  transfer and returns immediately.
- `Q3nn` — **verify** the staged swath landed. Cheap assertion at the pass
  boundary: no-op when the async load finished (the normal case), immediate
  fault when it didn't. Failure reports pen / pass / file through the
  respond mechanism. A waiting flavour, if ever needed, is an explicit
  `Q3nn WAIT=1` — never the default. An assertion that quietly waits isn't
  asserting anything.

Syntax: Klipper takes the whole first whitespace-delimited token as the
command name, so `Q100=file` does not parse. The form is:

```
Q100 FILE=001_001_00.ppp
```

`FILE=` is optional. Pen is named by the command digits and pass by the
running pass counter, so a bare `Q100` resolves implicitly; `FILE=` is the
explicit override and the audit form. Terse by default, greppable on demand.

An initial 4-pen job opens like this:

```
Q100 FILE=001_001_00.ppp
Q101 FILE=001_001_01.ppp
Q102 FILE=001_001_02.ppp
Q103 FILE=001_001_03.ppp
T00
T01
T02
T03
Q200 FILE=001_002_00.ppp
Q201 FILE=001_002_01.ppp
Q202 FILE=001_002_02.ppp
Q203 FILE=001_002_03.ppp
G1 Xn F37500
Q300
Q301
Q302
Q303
Q200 FILE=001_003_00.ppp
...
```

Swath spans the **full commanded move**, leading and trailing columns clear.
Array sized by geometry, nothing computed at runtime, no case where the pass
ends early with data still in the buffer.

Trigger picks where column zero lands. **Encoder clocks every column after
that** — so trigger jitter shifts the whole swath by a fraction of a column
instead of accumulating, and accel/decel/velocity ripple stop mattering. Ink
is placed against position, not time.

## Underrun — decided

A pass cannot start unloaded: `Q1nn` blocks and `Q3nn` faults at the
boundary, so mid-pass starvation means hardware is actively failing, not
that a load was late.

If it happens anyway: **fire blanks, complete the pass, fault at the pass
boundary, log loudly.** Never abort with the head over substrate — an abort
ruins the sheet *and* leaves the machine in an undefined position; blanks
ruin the sheet recoverably and keep motion sane.

## PDF as a validation target

1-bit masks map onto PDF `ImageMask` + `CCITTFaxDecode` almost directly: set
bits paint the fill colour, clear bits leave the page alone. One mask per pen,
composited by overprint, no flattening to RGB.

Render `.grip` → PDF, and if the PDF looks right the file is right — before a
drop of ink. Proofing the press without the press.

This is why per-pass origin belongs in the manifest: otherwise the renderer
has to parse motion, i.e. simulate the machine.

## Open

- exact ini key names (leaning: snake_case, one `[grip]` section,
  `version` first, freeze nothing else until v1)
- whether pass overlap/interleave/direction needs explicit fields or stays
  implied by pass ordering (leaning: implied — the renderer doesn't need
  direction, and direction-dependent flight-time offset is machine
  calibration, not job data)
