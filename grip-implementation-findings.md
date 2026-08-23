# What implementing `.grip` turned up

Written against `grip_format_notes.md`. Every item here is something the
reference implementation had to decide in order to compile. Nothing below is
a proposed redesign — they are places where two sentences in the notes are
individually clear and jointly ambiguous, plus the interpretation this code
adopted so that the ambiguity is at least *recorded* rather than silently
resolved differently by the next implementer.

Three of these are worth a spec edit before v1. They are marked **[EDIT]**.

---

## 1. Per-pen placement has no normative home — **[EDIT]**

**The conflict.** Validation check 5 pins `ROWS` to `nozzles_per_pen`, so a
`.ppp` covers one pen's array and nothing more. `passes.csv` deliberately has
no pen column, because "all pens in a pass share origin and direction by
physics". And `[pen.nn]` says of `offset_x` / `offset_y`: "All of this is
provenance, not instruction. Registration compensation is already baked into
the raster by the RIP; a consumer that re-applies `offset_x` double-corrects."

Take those three together and a renderer has no way to know where pen 01's
300 rows land. Every pen would paint the same 300 rows at `origin_y`, which
is wrong in both operating modes: in mono the pens own different half-inch
bands, and in CMYK the gang is still physically staggered, so within any one
pass the four pens are still at four different media positions.

The sentence "Character 0 of every line lies at `origin_y` plus the swath
height" is true only for the AA-most pen. For every other pen it is false,
and the notes do not say what replaces it.

**What this implementation does.** It reads `[pen.nn] offset_x` / `offset_y`
as the pen's *nominal geometric position in the gang*, in page-space sense,
measured from the swath origin — and the renderer places pen *p*'s character 0
at `origin_y + offset_y[p] + ROWS / dpi_media`. On the reference four-pen
gang that makes `offset_y` = `[37.084, 24.723, 12.361, 0.0]` mm for an
8-nozzle overlap, and pen 00's character 0 lands at exactly
`origin_y + swath_height`, which is the notes' sentence, recovered.

**Why not the alternative.** The other consistent reading is that pens stack
contiguously by index — pen *p* occupies rows `[p × ROWS, (p+1) × ROWS)` of
the gang, derivable from `pens` and `ROWS` alone with no per-pen field. That
is simpler, and it is wrong the moment `overlap_nozzles` is non-zero, because
then the gang span is less than `pens × ROWS`. It would also make the gang's
geometry re-derivable from the container in a way that silently disagrees
with the machine, which is the class of thing the notes are otherwise careful
to avoid.

**Suggested edit.** Split the `[pen.nn]` paragraph in two. The coarse
stagger is *instruction* — the renderer must apply it or it cannot place a
swath at all. Fine registration compensation is *provenance* — already baked
into the raster, never re-applied. Both currently live under one sentence
that says "not instruction", and only the second half of that is true.

If you would rather keep `offset_x`/`offset_y` purely as provenance, the
minimal alternative is a new normative key — `band_dy` or similar, in mm,
relative to the pass origin — and this implementation would switch to it in
about ten lines. The important thing is that *something* in the container is
normative for placement.

---

## 2. Space cannot be declared as the `INACTIVE` glyph — **[EDIT]**

The notes name space as "the documented alternate for hand-authored
fixtures". But the header line is `KEY:VALUE`, **space-delimited**, so
`INACTIVE: ` does not survive tokenisation: it parses as a key with an empty
value, and then the next token gets read as a value.

**What this implementation does.** Reserves the two-character token `SP` for
the space glyph, in both directions: it writes `INACTIVE:SP` and reads `SP`
back as `" "`. Unambiguous, because every real glyph is one character.

**Suggested edit.** One sentence in the header-line section naming `SP`, or
alternatively dropping space as a documented alternate. Either is fine; the
current text promises something the grammar cannot express.

---

## 3. `origin_x` + swath width is off by one column

"On `XNEG`, line 0 lies at the swath's **far** end — `origin_x` plus swath
width." Swath width is `COLUMNS ÷ dpi_scan`, so `origin_x + width` is one
column pitch *past* the last column. The last column's own position is
`origin_x + (COLUMNS − 1) ÷ dpi_scan`.

At 600 dpi this is a 42 µm error — about one nozzle pitch, which is exactly
the scale at which registration arguments happen. Two implementations that
each read the sentence literally, in opposite directions, produce a
one-column relative shift between XPOS and XNEG passes, which presents as a
bidirectional registration error and gets chased into the hardware.

**What this implementation does.** Treats `origin_x` and `origin_y` as the
low corner of the *occupied* raster and indexes columns `0 … COLUMNS−1`
inside it, in both directions. Same for the media axis: character 0 sits at
`origin_y + offset_y + (ROWS − 1) ÷ dpi_media`, not `+ ROWS ÷ dpi_media`.

**Suggested edit.** State the extent convention once — whether the origin is
the low edge of the first cell or the low corner of the bounding box — and
let both the `x` and `y` sentences inherit it.

---

## 4. `[rip] interleave` and `overlap_nozzles` are informative, but stitch
   quality depends on them

The notes are firm and, I think, right: `[rip]` is informative and consumers
SHALL NOT alter behaviour on it. But `overlap_nozzles` is not only a record
of what the RIP did — it is a fact about the *head* that a receiving machine
might reasonably want to compare against its own geometry, the same way
`nozzles_per_pen` lets it "reject a job built for a different head before it
moves".

This implementation writes `overlap_nozzles` to `[rip]` as specified and
consumes it nowhere. Flagging only that `nozzles_per_pen` earned its way into
`[geometry]` for exactly the reason `overlap_nozzles` might.

Not an edit request. A question for v1.

---

## 5. Nothing says a pass's pens must agree on `COLUMNS`

`passes.csv` omits swath extent because it is derivable from `COLUMNS`, and
"all pens in a pass share origin and direction by physics" — they are on one
carriage in one sweep. The same physics means they share the sweep *length*,
so all pens in a pass must have identical `COLUMNS`. The notes never say so,
and the seven checks do not test it.

This implementation refuses to write such a container and reports it as a
fault when reading one. Cheap check, real failure mode: a job where one pen's
swath is a different length is a job where one pen's registration is wrong by
a growing amount.

**Suggested edit.** An eighth check.

---

## 6. Small provenance values round to zero

Not a spec issue, a spec-adjacent trap. `planned_consumption` is millilitres.
A light page on a four-pen gang is single-digit microlitres per pen, so any
fixed-decimal-places formatter writes `0.0000` and the provenance is gone.
This implementation writes floats with nine significant figures and accepts
exponent notation on read. Worth a sentence, because the first person to
write a manifest by hand will use `%.4f`.

---

## 7. The `Q3nn` retry budget interacts with decap, and the container cannot
   express the tradeoff

The notes already flag this — "decap is the price of blocking", firmware
policy not container policy. Recording it here only because the reference
gcode emitter has no way to hint at it: `command.gcode` emits `Q3nn` and the
firmware decides how long to block and whether to spit. If a job wants a
tighter budget, there is nowhere to say so. That is probably correct — it is
a machine property, not a job property — but it means the "two attempts
total" number is normative firmware behaviour and belongs in the firmware
spec, not only in the container notes.
