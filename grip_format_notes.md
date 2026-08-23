# .grip — cliff notes

**DROPSpec** (Deposition Raster Open Print Spec): the standard.
**`.grip`**: its job container.
**MDP-01**: reference implementation.

Working notes for the DROPSpec job container. Decisions made, not a finished
spec. Where this document and a decision below disagree, the decision governs
— but nothing here is frozen until v1.

## What it is

A zip archive. Extension `.grip` (g-rip: gcode + rip).

```
manifest.ini
passes.csv
command.gcode
rip/
  001_001_00.ppp
  001_001_01.ppp
  001_001_02.ppp
  001_002_00.ppp
  ...
```

Four members, each with an obvious owner:

- **manifest.ini** — what this job is. Constant size, whatever the job length
- **passes.csv** — where each pass lands. Grows with the job
- **command.gcode** — the motion plan, Klipper-consumable
- **rip/** — the swaths, in print order

`passes.csv` exists so that `manifest.ini` can stay constant-size. Per-pass
origin is a table, not a field, and a 500-page job has thousands of rows.
Isolating the one part of the container that scales with job length is what
lets the manifest genuinely be *what this job is*.

Zip specifics: **deflate**, zip64 where required, `rip/` as a path prefix
with no directory entries. Directory entries are omitted by enough writers
that requiring them only creates false negatives in validation.

## Naming and field widths

`PPP_SSS_NN.ppp` — `page_pass_pen`.

- **Pages** 1-based, three digits, **999 maximum**
- **Passes** 1-based, three digits, **999 maximum**, **counted per page**
- **Pens** 0-based, two digits

Pens are 0-based because a pen index is a hardware address and `T00` already
forces it. Pages and passes are 1-based because they are print ordinals, and
`page 000` reads wrong in every log line anyone will ever grep. Mixed base is
deliberate; it needs stating rather than inferring.

Passes are counted **per page**, not job-global. A single page is therefore
extractable as a self-contained unit, and `passes.csv` keys on `(page, pass)`
— which it needs to do anyway once pages can have differing pass counts.

**Overflow of either 999 is a hard fault. Fields never widen.** Widening
silently destroys the lexical sort order that triple-padding exists to
protect, which is worse than refusing the job. Same posture as underrun:
fail loudly, do not improvise.

Pen-major within a pass, so everything a single pass needs is contiguous in
the archive.

Headroom check, so the caps are on the record as arithmetic rather than
guesswork: CMYK quality mode is the worst known case at roughly 56 passes
per page. 999 leaves ~18×, enough to absorb decisions not yet made (wider
media, more overlap nozzles, finer advance).

## Coordinate systems

Two of them. They disagree in two places, both intentionally. State them
together or nobody will get this right.

**Page space** — origin **bottom-left**, +x along scan, +y along media
advance. Bottom-left because Klipper is bottom-left, and the machine's own
coordinate system is the one people will debug in. A manifest that disagrees
with `printer.cfg` means everyone reading a fault message performs a mental
flip forever.

All geometry is **millimetres**. The manifest already carries origins and
tool offsets in mm; page size in inches would put two units in one file
describing one coordinate space, which produces a 25.4× error exactly once,
spectacularly, in someone else's implementation. Dots appear only as raster
dimensions in the `.ppp` header.

**File space** — within a `.ppp`:

- **Line 0 is the first column fired.** On `DIR:XNEG` passes, line index
  therefore runs from high x to low x
- **Character 0 is the nozzle nearest AA** (the top face), so character index
  runs in **decreasing y**

**The mapping.** `origin_x` and `origin_y` in `passes.csv` are the swath's
**lowest-x, bottom** corner in page space, direction-independent. Character 0
of every line lies at `origin_y` plus the swath height. On `XNEG`, line 0
lies at the swath's **far** end — `origin_x` plus swath width.

A renderer therefore does exactly one conditional thing: **reverse line order
on `XNEG`**. Everything else is unconditional. A renderer that ignores `dir`
entirely still places every swath in the right position and only gets internal
column order wrong — a visible, diagnosable failure rather than a silent
misplacement.

## Swath data — `.ppp`

**Plain text.** One line per column, one character per nozzle.

```
# ROWS:300 COLUMNS:5100 PAGE:001 PASS:001 PEN:00 ACTIVE:8 INACTIVE:. DIR:XPOS
..........8888888888888888..........;
..........8888888888888888..........;
```

- `.` inactive, `8` active, `;` terminator
- Line length exactly **ROWS + 1**. Character at index ROWS is `;`
- LF line endings; parsers tolerate CRLF, since `;` is the true terminator
- Extension `.ppp` (page/pass/pen)

**Column-major** because that is the consumption axis — one column per fire
event. Note this is a property of the *loaded buffer* and the fire loop, not
of the archive member: the member sits inside a zip, so something already
decompresses it into a buffer, and the buffer is what the fire loop reads.
The DMA argument never applied to the on-disk encoding.

**Why `8`.** It tiles — its strokes reach the cell edges, so adjacent glyphs
join into continuous mass rather than reading as a texture of separated dots.
`@` has higher raw ink coverage and is worse for exactly that reason.

**Why `.` rather than space.** Space is more legible, and was the original
choice. `.` wins on safety: nothing anywhere strips a period. The `;`
terminator already makes trailing clear-nozzles interior, so space was
defensible — but a format that outlives its authors should not depend on
every downstream tool respecting whitespace. Space remains the documented
alternate for hand-authored fixtures, never for RIP output.

**Why not `#` for active.** `#` is the comment prefix. A data line whose
first nozzle is set would be ambiguous with a header unless the parser reads
to end-of-line before deciding what it is holding — a subtle rule somebody
implements wrong.

**Resist the empty-column shorthand.** Full-clear columns are common, since
the swath spans the whole commanded move with leading and trailing columns
clear. Someone will propose a bare `;` for "300 inactive". Refuse it: it
compresses to nothing anyway, and it destroys the fixed-width invariant that
is the entire integrity story.

**Note on `8`.** On data lines it means a firing nozzle. In the header line,
the digits in `300`, `5100`, `001` are just digits. No parser can be
confused; a loose `grep` can be.

### Header line

`#`-prefixed, so it is skippable with `grep -v '^#'` and leaves room for a
second comment line later without a format revision.

`KEY:VALUE`, space-delimited, **colon with no space**, strict parse.

| Field | Purpose |
|---|---|
| `ROWS` | Nozzles per column |
| `COLUMNS` | Fire events in this pass |
| `PAGE` `PASS` `PEN` | Identity, ordered to match the filename left-to-right |
| `ACTIVE` `INACTIVE` | Glyph set, so a `.ppp` needs no external context to read |
| `DIR` | `XPOS` or `XNEG` |

`ROWS` and `COLUMNS` are what make a `.ppp` readable standalone — extracted
from the archive and mailed to you, it is still fully self-describing. That
is the whole reason a header exists at all.

`PAGE`, `PASS`, `PEN`, and `DIR` duplicate `passes.csv` and the filename.
**`passes.csv` is authoritative; disagreement is a fault.** That turns
redundancy into a check that catches a misfiled or misnamed swath at load,
rather than as a mystery registration error later.

`DIR` uses the same tokens as `passes.csv` so the check is string equality,
not a mapping. Alphabetic (`XPOS`/`XNEG`) rather than `X+`/`X-` because it
parses faster for a human and carries no sign character for a sloppy
tokenizer to mangle. Axis-relative rather than FOR/REV because it is
self-defining against the coordinate system the container already declares,
and it names the axis explicitly.

### Nozzle order

**Geometric, not firing-address order.** Character 0 is the nozzle nearest
**AA**; index increases toward **AC**.

Address mapping and the scan-axis offset between a pen's two nozzle columns
are machine facts, absorbed by per-pen firing delay. They are not job data.
Firmware carries the permutation table — which is the correct place for it,
and the difference between a `.grip` being portable across head personalities
and not.

Character 0 at AA also means the file reads right-side-up when opened. Given
that legibility is the entire justification for a text format, that is worth
the one stated inversion against +y.

### Size

Text costs about 8× uncompressed against packed binary, and **1.95×
compressed** — measured, not estimated, on a 300 × 5100 swath at roughly 2%
coverage:

| | uncompressed | gzip -9 |
|---|---|---|
| text | 1,540,250 B | 14,641 B |
| packed binary, 38 B stride | 193,800 B | 7,512 B |

That sparse case is close to the **worst** ratio, not the best. At high
coverage LZ77 finds little in either encoding and Huffman on a two-symbol
alphabet converges toward one bit per character, so text approaches ~1.1×.
The penalty shrinks as content densifies.

Accepted. What this project is short of is contributors who can read the
format, not disk.

## manifest.ini

Lowercase keys with underscores. `version` first.

### `[grip]`

| Key | Notes |
|---|---|
| `version` | Format version |
| `profile` | `sheet`. `roll` is enumerated and **[RESERVED]** |
| `job_name` | Free text, for the queue UI and log lines |
| `created` | ISO 8601. First thing you want when two files disagree |
| `pages` | Count |
| `passes_total` | Across the job. Redundant against `passes.csv` row count — a check, not a source |
| `pens` | Count |

**On `profile`.** It exists before it varies, deliberately. A container kind
that will fork is a version boundary, and those are cheapest installed before
the fork. A v1 reader seeing `profile = roll` **refuses the file** rather
than guessing field widths.

`roll` is out of scope and likely a different container (`.gripi`) rather
than a mode of this one. Two reasons: roll needs a six-digit pass field
(`PPP_SSSSSS_NN` — 999,999 passes ≈ 3.9 miles of web at CMYK quality-mode
advance, where five digits caps out around 2,080 ft, an ordinary label roll),
and a filename grammar cannot be announced by a field inside a member the
reader must enumerate filenames to find. The deeper problem is that a
continuous-web job's raster may not be finite at job start, while zip is a
closed, indexed archive.

### `[geometry]`

| Key | Notes |
|---|---|
| `nozzles_per_pen` | Compatibility assertion |
| `dpi_scan` | Cross-axis; encoder ticks per fire |
| `dpi_media` | Down-axis; nozzle pitch |
| `page_width` | mm, float |
| `page_height` | mm, float |

`nozzles_per_pen` is no longer needed to parse anything — `ROWS` is in every
`.ppp` header. Its job now is to let a machine **reject a job built for a
different head before it moves**. Different justification than the one that
originally put it here; recorded so it is not inherited by accident.

`dpi_scan`/`dpi_media` rather than x/y because x and y swap meaning the day
someone builds a portrait-fed machine or bolts a head to a Voron gantry.
Scan and media are architectural and cannot rotate.

**Page size is the media extent, not the image extent.** It is the PDF
MediaBox. A pass may legally sit outside it — bleed, overspray beyond trim —
so a renderer **clips rather than faults**. Without this stated, someone
validates pass extents against page bounds and rejects a good full-bleed job.

### `[pen.00]` … `[pen.nn]`

| Key | Notes |
|---|---|
| `colorant` | A name: `black`, `cyan`, `uv_615`, `micr`, `varnish` |
| `srgb` | Optional hint, omissible |
| `offset_x` `offset_y` | mm |
| `planned_droplets` | Integer, exact, derived from the rasters |
| `planned_consumption` | Millilitres, float |
| `drop_volume_pl` | The assumption behind the line above |

**All of this is provenance, not instruction.** Registration compensation is
already baked into the raster by the RIP; a consumer that re-applies
`offset_x` double-corrects. Colorant is a record of which pen received which
mask. This has to be unambiguous in spec text, not merely implied by the
section name.

Consequence: **the PDF render is a proof, not a conformance artifact.** It
consumes informative data to choose fill colours, so a PDF mismatch is not a
spec violation.

**Colorant is a name plus an optional hint, not a colour.** A 615 nm
fluorescent has no honest sRGB value, and neither does MICR or a clear
varnish. The format should not be made to pretend it knows what an ink looks
like.

**"Planned" is doing real work.** It is not a claim about reality — it is
what the RIP intended. Divergence between planned and actual is the
informative signal: nozzle dropout, energy trim drift, a depletion mask
biting harder than expected. `drop_volume_pl` makes the arithmetic auditable
and recomputable against a better number; ~30 pL nominal is a datasheet
figure, not a measurement.

Per-pen rather than per-job because four pens may carry entirely different
fluids, and the operationally interesting question is which cartridge runs
dry first. Job totals are trivially summed by anything that cares.

### `[rip]` — informative

`generator`, `generator_version`, `halftone`, `overlap_nozzles`, `interleave`.

**Consumers SHALL NOT alter behaviour based on anything in `[rip]`.**

It exists for the human reading a bug report who needs to know whether a
visible stitch came from the RIP or the machine. Overlap, interleave, and
direction remain **implied** for rendering purposes — a swath paints at its
origin regardless — and this section records intent without joining the
conformance surface.

## passes.csv

Comma-delimited, LF, header row required, no quoting (no field can contain a
comma). Keys on `(page, pass)`.

```
page,pass,origin_x,origin_y,dir
1,1,12.7,266.7,XPOS
1,2,12.7,215.9,XNEG
1,3,12.7,165.1,XPOS
```

| Column | Notes |
|---|---|
| `page` | 1–999 |
| `pass` | 1–999, per page |
| `origin_x` | mm. Lowest-x edge, **direction-independent** |
| `origin_y` | mm. Bottom edge |
| `dir` | `XPOS` / `XNEG`. Authoritative; `.ppp` header carries a copy |

`page` and `pass` are written **unpadded**. CSV is parsed, not sorted; the
triple-padding exists for filename lexical sort and does not apply here.

Rows **SHOULD** be in print order and **MUST** be keyed. A reader can walk it
sequentially, but order is not load-bearing for correctness.

**Deliberately absent:**

- **Swath extent** — derivable as `COLUMNS ÷ dpi_scan`, and `COLUMNS` is in
  every `.ppp` header. Storing it creates a value that can disagree with the
  rasters, and anything that can disagree with the rasters eventually will
- **Pen** — all pens in a pass share origin and direction by physics; they
  are on one carriage in one sweep. A pen column would quadruple every row
  and create a state that can be internally inconsistent

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

### Two buffers

- **Go buffer** — what the fire loop reads. Filled explicitly by `Q1nn`
- **Stage buffer** — filled asynchronously by `Q2nn`

**On go-buffer exhaustion, stage is promoted to go.** `Q3nn` verifies that
promotion occurred.

**Promotion is gated on the stage transfer being complete.** If the `Q2nn`
DMA is still in flight when the pass ends, promotion does **not** happen: go
stays empty and `Q3nn` catches it. Refusing to promote is the safe failure — a
torn buffer is worse than an empty one, because an empty one is detectable
and a half-filled one looks structurally valid and fires garbage.

**Buffers carry identity tags** — pen, page, pass, source filename —
and promotion carries the tag along. Without them `Q3nn` is only a liveness
check and passes when promotion moved a *stale* buffer. With them it asserts
that the **right** thing is there, and the fault report is meaningful.

### Commands

`Qxnn`, where `nn` is the pen. The digit is urgency, descending:

- `Q1nn` — **load** into the go buffer. Blocking: does not yield until the
  buffer is real
- `Q2nn` — **stage**. Async: fires the transfer and returns immediately
- `Q3nn` — **verify** that promotion occurred, and make it true if it did
  not. **Blocking.** Discards and reloads synchronously on failure; dumps
  the job after the retry budget is spent. Reports pen / pass / file through
  the respond mechanism on every attempt
- `Q4nn` — **discard**. Clears **both** buffers for that pen and drops their
  identity tags. Legal any time; the defined way to reach a known-empty state

`Q3nn` **treats a missing `Q2nn` as a failed promotion** — it discards,
reloads, and re-checks like any other failure. A missing stage is precisely
the bug the assertion exists to catch; treating it as a no-op makes `Q3`
silent in the one case where it matters most.

**`Q3nn` blocks by design, and that is not a contradiction of what an
assertion is for.** The objection to a waiting assertion is that it waits
*quietly* and forever, which turns a check into a hang. This one is bounded,
loud, and terminal: it reports every attempt, spends a fixed retry budget,
and dumps the job. It asserts, and then it acts on the assertion.

Blocking is affordable here specifically because `Q3nn` fires **before** the
sweep begins. The cost is time, not substrate.

`Q1nn` is **legal anywhere but expected only at cold start and after a
fault.** Steady state runs `Q3` then `Q2` and never touches `Q1` again. The
example below is the only documentation of that, and examples get read as
grammar, so it needs saying.

### Recovery

Recovery is **internal to `Q3nn`**, not a macro the job author writes. On a
failed promotion `Q3nn` discards both buffers, reloads synchronously, and
re-checks — up to its retry budget, then dumps the job.

```
Q300              ; verify. On failure, internally:
                  ;   discard both buffers (as Q4nn)
                  ;   synchronous reload from passes.csv + filename
                  ;   re-check
                  ; two attempts total, then job dump
```

**Two attempts total**, not one plus two. State it as attempts rather than
retries — off-by-one on a retry budget is the kind of thing two
implementations get differently.

**The pass never starts unloaded.** `Q3nn` sits at the boundary before the
sweep, so there is no ruined sheet in this path — the pass simply does not
begin until the go buffer is correct, or the job ends.

**`Q4nn` clears both buffers, not just stage.** After a failed promotion the
go buffer is precisely what is in question — it may be empty, may hold the
previous pass, may hold a torn transfer. Clearing only stage leaves the
ambiguous buffer in place, and the reload would have to overwrite it anyway.
Clearing both makes machine state unambiguous before reload, which is the
entire point of having a discard command.

`Q4nn` remains separately useful: **it is the correct call on job dump and on
fault-stop generally.** A machine that stops with stale swaths sitting in
buffers is a machine that can resume into garbage.

**Job dump** means: `Q4nn` every pen, park, cap, report pen / page / pass /
file. Capping is not optional — see below.

**Decap is the price of blocking.** A stopped carriage at a pass boundary is
an uncapped head, and TIJ decap runs in seconds. Two synchronous reload
attempts may be long enough to matter. Firmware policy, not container policy,
but it follows directly from this design: a block that exceeds a threshold
should spit, or park and cap, before continuing. Recorded here so the
consequence is not discovered on a nozzle plate.

### Scheduling — the thing that silently doesn't work

**Klipper's gcode processing runs ahead of motion.** The lookahead queue
exists so motion never starves, which means a command after `G1` is parsed
and executed long before the carriage physically finishes that move.

`Q3nn` must therefore be **scheduled against the toolhead's print time, not
executed at parse time.** A parse-time check runs before the pass it is
supposed to be verifying, always passes, and asserts nothing. Same for the
promotion event: "on go-buffer exhaustion" is a motion-time event.

This is the only thing in the design that fails *silently* if an implementer
misses it. Everything else fails loudly.

It also validates the command split. `Q1nn` blocking at parse time stalls
the lookahead queue — correct at cold start, wrong mid-job, which is exactly
why `Q1` is cold-start-and-recovery only. `Q2nn` async at parse time is fine,
since it only kicks off a transfer. `Q3nn` blocks, but at print time and at a
pass boundary, where a stall costs a pause between sweeps rather than a
starved move. Three commands, three different relationships to time. Not an
accident.

### Syntax

Klipper takes the whole first whitespace-delimited token as the command name,
so `Q100=file` does not parse. The form is:

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

**A pass cannot start unloaded.** `Q1nn` blocks, and `Q3nn` blocks and
reloads at the boundary, so the only remaining starvation case is hardware
failing *during* a sweep. That is not a late load; it is a machine breaking
mid-motion.

This section governs that case alone, and it is now the only path in the
design where a sheet is ruined.

If it happens: **fire blanks, complete the pass, fault at the pass
boundary, log loudly.** Never abort with the head over substrate — an abort
ruins the sheet *and* leaves the machine in an undefined position; blanks
ruin the sheet recoverably and keep motion sane.

## PDF as a validation target

1-bit masks map onto PDF `ImageMask` + `CCITTFaxDecode` almost directly: set
bits paint the fill colour, clear bits leave the page alone. One mask per pen,
composited by overprint, no flattening to RGB.

Render `.grip` → PDF, and if the PDF looks right the file is right — before a
drop of ink. Proofing the press without the press.

This is why per-pass origin belongs in `passes.csv` rather than the gcode:
otherwise the renderer has to parse motion, i.e. simulate the machine. Page
size supplies the MediaBox, which cannot be honestly inferred from pass
extents.

The renderer's full conditional logic: **reverse line order on `XNEG`.** That
is all.

## Validation

Cheap checks, in the order they fail usefully:

1. `profile` is `sheet`, else refuse
2. Every `.ppp` line is exactly `ROWS + 1` characters, terminated `;`,
   containing only the declared `ACTIVE` and `INACTIVE` glyphs. Failure
   reports a **line number**
3. `.ppp` header `PAGE`/`PASS`/`PEN` agree with the filename
4. `.ppp` header `DIR` agrees with `passes.csv`
5. `ROWS` agrees with `nozzles_per_pen`
6. `passes_total` agrees with the `passes.csv` row count
7. Every `(page, pass)` in `passes.csv` has a full set of `pens` swath files

Text validation is **stronger** than the binary equivalent, not weaker.
Packed binary gives "file length is a multiple of stride"; text gives a
specific corrupt line number.

## Open

- Whether these notes split into a normative spec plus a rationale companion.
  Most of the value above is *why*, and the whys are what stop a future
  contributor unwinding a decision they don't understand — but mixed into a
  spec they make it harder to implement against
- Freeze nothing else until v1
