# Getting a document into a `.grip`

Design note, August 2026. Covers deliverable **F1 (CUPS filter chain)** and
the shorter path that exists today. No filter code ships with this note —
that is deliberate; see "Why not build the filter yet" at the end.

There are three ways a document becomes a `.grip`, and they are worth
separating because they fail differently.

| | Effort | Works today | "Select an option" in LibreOffice | Right long-term |
|---|---|---|---|---|
| **A.** `grip-write` on the command line | none | yes | no | — |
| **B.** LibreOffice macro on a toolbar button | ~20 lines of Basic | yes | **yes** | as a stopgap |
| **C.** CUPS queue, `File → Print → DROPSpec` | a filter, a PPD, a backend | not yet | **yes** | yes |

Path B gives you the thing you actually asked for — open a document, click a
thing, get a `.grip` — without touching system printing configuration. Path C
is the same experience routed through the print system, which is what buys
you every *other* application on the machine for free.

---

## A. The command line

`grip-write` already handles the document → PDF step itself, by shelling out
to LibreOffice headless:

```
grip-write quarterly-report.odt -o quarterly-report.grip
grip-validate quarterly-report.grip
grip-render  quarterly-report.grip -o proof.pdf
```

`.odt`, `.docx`, `.rtf`, `.txt`, `.html`, `.xlsx`, `.pptx` — anything
LibreOffice opens — goes through `soffice --convert-to pdf` into a temporary
file, and PDFs and raster images are read directly. A private user profile
is used per conversion, so this is safe to run while LibreOffice is open.

---

## B. LibreOffice macro — the option in the menu

`examples/libreoffice/ExportGrip.bas` is a Basic macro that exports the
current document to a temporary PDF using LibreOffice's own PDF filter, runs
`grip-write` on it, and reports where the container landed.

Install:

1. **Tools → Macros → Edit Macros**, expand *My Macros → Standard*,
   right-click *Module1* → *Insert → BASIC Module*, name it `DROPSpec`
2. Paste the contents of `ExportGrip.bas`
3. **Tools → Customize → Menus** (or *Toolbars*, or *Keyboard*), pick
   *Macros → My Macros → Standard → DROPSpec → ExportGrip*, and add it
   wherever you want it — *File* menu, a toolbar button, a shortcut

The macro deliberately uses LibreOffice's PDF export rather than its print
path: the PDF export is deterministic, is not affected by whatever printer
happens to be selected, and preserves the document's page size, which becomes
the `.grip` MediaBox.

**Its limits, honestly.** It shells out, so `grip-write` has to be on `PATH`
as LibreOffice sees it (which on a Flatpak or Snap LibreOffice, it is not).
It handles one document at a time. It knows nothing about a print queue, so
there is no job spooling, no "print 3 copies", and no other application on
the machine can use it. Those are exactly the things path C buys.

---

## C. The CUPS queue

### Where CUPS actually is, in 2026

This matters because the obvious design is the one being deprecated.

CUPS 2.4.19 is the current stable release; there is no CUPS 3.0 yet, and the
roadmap still has a 2.5.x series in front of it. So the classic architecture
— a PPD, a filter chain, a backend — still works on every distribution you
would actually install this on today.

But it is on a published deprecation path: raw queues were deprecated in CUPS
2.2, printer drivers in 2.3, and CUPS 3.0 ends driver support outright. The
replacement is the **Printer Application**: a daemon that speaks IPP to CUPS
on one side and does the rasterisation itself on the other, typically built
on PAPPL. Distributions lag the deprecation by years, so both are live
choices right now — but only one of them has a future.

The recommendation is therefore to **write the conversion once, as a library
entry point, and wrap it twice.**

### C1. The classic chain (build this first)

```
LibreOffice ──IPP──▶ cupsd ──▶ pdftoraster ──▶ rastertogrip ──▶ backend ──▶ press
                                (cups-filters)   (ours)          (ours)
```

**`rastertogrip`** reads CUPS Raster on stdin and writes a `.grip`. It gets
its arguments the way every CUPS filter does — `argv[1..6]` being job id,
user, title, copies, options, filename — and CUPS Raster hands it a page
header per page carrying resolution, media size, colour space and bit depth.
That header is very nearly a `manifest.ini` already:

| CUPS Raster header | `.grip` |
|---|---|
| `cupsPageSize` / `PageSize` | `[geometry] page_width`, `page_height` |
| `HWResolution` | `dpi_scan`, `dpi_media` |
| `cupsBitsPerPixel`, `cupsColorSpace` | which pens, and whether we screen |
| `argv[3]` (job title) | `[grip] job_name` |
| `argv[1]`, `argv[2]` | log lines, not container fields |

Three decisions inside the filter, all of which the reference implementation
has already made and can simply be called:

1. **Ask for 8-bit grey, not 1-bit.** Declare `cupsBitsPerColor 8` and
   `cupsColorSpace CUPS_CSPACE_SW` in the PPD so `pdftoraster` hands us
   continuous tone and *we* screen it. If CUPS dithers, halftoning lives in
   two places and F2's depletion masking has nowhere to go. This is the single
   most important line in the PPD.
2. **Stream, do not buffer.** A 500-page A4 job at 600 dpi is roughly 4 MB of
   1-bit mask per page per pen. `GripWriter` already streams swaths into the
   archive as they are produced; the filter should feed it page by page and
   never hold the job.
3. **Emit one container per job, not per page.** The container is what keeps
   motion and bitmaps from desyncing; splitting it per page throws that away.

**The PPD** is small. The parts that carry weight:

```ppd
*cupsFilter2: "application/vnd.cups-raster application/vnd.cups-raster 50 rastertogrip"
*cupsManualCopies: True
*DefaultResolution: 600x600dpi
*OpenUI *Resolution/Resolution: PickOne
*Resolution 600x600dpi/600 dpi: "<</HWResolution[600 600]>>setpagedevice"
*Resolution 300x300dpi/300 dpi: "<</HWResolution[300 300]>>setpagedevice"
*CloseUI: *Resolution
*OpenUI *DSHalftone/Screening: PickOne
*DSHalftone FS/Error diffusion: ""
*DSHalftone Bayer/Ordered: ""
*CloseUI: *DSHalftone
```

Press-specific options — screening method, stitch overlap, bidirectional or
not — become `*OpenUI` groups, arrive in `argv[5]` as `key=value` pairs, and
map one-to-one onto `RipOptions` and `MachineProfile` fields. Nothing new has
to be invented; the CLI flags and the PPD options are the same set.

**The backend** decides where the container goes. Three options, increasing
in ambition:

- `file:///var/spool/dropspec/` — CUPS' own `file` backend, needs
  `FileDevice Yes` in `cups-files.conf`. Fine for bring-up: the queue
  produces containers, you inspect them with `grip-info`, nothing moves.
- A `grip://` backend that `POST`s to Moonraker's `/server/files/upload`
  and then starts the job. This is the one that makes the machine feel like
  a printer.
- Direct to a watched directory on the Pi over a share. Works, but has no
  error path back to the queue, so a failed job looks successful in the UI.

### C2. The Printer Application (the version that survives CUPS 3)

Same conversion, different wrapper: a PAPPL-based daemon advertises the press
as an IPP Everywhere printer over DNS-SD. CUPS discovers it driverlessly —
no PPD, no filter, no backend, nothing installed system-wide — and every
client on the network can print to it, including phones. The document format
arriving is PDF or PWG Raster rather than CUPS Raster, which if anything is
easier, since `dropspec.raster` already reads PDF.

The reason not to start here is that it is a daemon with a web UI, a
discovery story, and a state machine, and none of that helps you find out
whether the raster you produce is correct. Start with C1, keep the conversion
in a library, move the wrapper when the distributions force it.

### What CUPS cannot give you

CUPS' job model ends when the backend accepts the file. It has no idea what
a pass is, so:

- **Progress** is "sent to printer", not "page 4 of 12, pass 3". Real progress
  has to come back from Moonraker.
- **Cancel** cancels the spool, not the sweep. Cancelling a running job is a
  Klipper concern, and it must end in `Q4nn` on every pen, park, and cap — a
  machine that stops with stale swaths in its buffers can resume into garbage.
- **Media handling** — the press's sheet feed, the vacuum platen, capping
  state — has no CUPS vocabulary. `printer-state-reasons` can carry a few
  standard strings (`media-empty`, `marker-supply-low`) and that is about it.

None of that is an argument against the queue. It is an argument for the
queue being a *submission* path, with Moonraker remaining the machine's
actual interface.

---

## What the machine end still needs

A `.grip` in a spool directory is not printing. The missing piece is
firmware-side and is deliverable **E2**, the `[tij_printhead]` klippy extras
module. Concretely it needs a command roughly like:

```
GRIP_LOAD FILE=quarterly-report.grip
```

which opens the container, reads `manifest.ini`, checks `nozzles_per_pen`
against the machine's own geometry and **refuses the job before it moves** if
they disagree, then executes `command.gcode` from inside the archive with
`Qxnn` resolving `FILE=` against the archive's `rip/` prefix.

That last point is the whole reason the container is one file: the gcode and
the rasters it names cannot desync, because they arrived together. Any
integration that unpacks the archive and hands Klipper a loose `.gcode` has
thrown that guarantee away on the first step.

Two things E2 must get right, both already stated in the format notes and
both worth repeating where an implementer will see them:

1. **`Q3nn` schedules against toolhead print time, not parse time.** Klipper's
   gcode processing runs ahead of motion. A parse-time `Q3` runs before the
   pass it is meant to verify, always passes, and asserts nothing. This is the
   only thing in the design that fails *silently*.
2. **Buffers carry identity tags** — pen, page, pass, source filename — and
   promotion carries the tag along. Without them `Q3nn` is a liveness check
   that passes when promotion moved a stale buffer.

## Why not build the filter yet

Because the filter is the easy half and the wrong thing to freeze first.

`rastertogrip` is about 150 lines around a library call. What it commits you
to is the *option vocabulary* — the names and semantics of every press
setting a user can pick from a print dialog — and that vocabulary is
downstream of decisions still open: stitch overlap in nozzles (O3), the
staircase arrangement (O2), maximum media width and roll support (O6). Ship a
PPD now and those names are in users' `~/.cups/lpoptions` forever.

The order that costs least: stabilise the container, get proofs that look
right, close O2/O3/O6, then write a filter whose options describe a machine
that exists.
