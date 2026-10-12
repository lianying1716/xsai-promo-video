---
name: promo-video-pipeline
description: >
  Make a product promo / feature / launch video entirely from code plus real product
  screenshots — no editor, no manual cutting. Use when someone wants a promo, feature
  highlight, product demo, or launch film built as a reproducible pipeline: define the
  acceptance bar first, write the script and storyboard, capture the real UI, build and
  render the composition, then audit the result with measurable signals instead of
  eyeballing it. Also use when re-running, updating, or QC-ing an existing video made this
  way. Defaults to HyperFrames but the workflow and the audit tooling work with any
  renderer that outputs an MP4.
license: MIT
---

# Promo Video Pipeline

A repeatable way to produce a sharp product promo video **with code and real screenshots only** —
built for people who don't edit video. The value is not the finished film; it is that **every
judgment about the film can be disproven by a command**.

## The core idea

"Good-looking" is decomposed into measurable quantities — energy ratios, active-frame rates,
zoom scale factors, block-difference ratios. That makes the workflow scriptable and reviewable,
and it means a film can be driven to a real quality bar even when nobody can sit and watch it
frame by frame.

Three commitments carry the whole method:

1. **Define the pass line before touching the render.** Otherwise "is it better now" is pure
   vibes and you loop forever.
2. **Use real product UI and real numbers as the evidence.** Generated mock UI and invented
   stats are the fastest way to make a promo that nobody trusts.
3. **Study references twice — with numbers and with a vision model.** Numbers tell you *which
   dial is wrong*; a vision model tells you *what to do about it*. Both are in this skill.

## Workflow

Run the steps in order. Each step's output is the next step's input, and **each step has a
machine-checkable gate** — do not advance past a failed gate.

```
1 Define pass line -> 2 Script -> 3 Real assets -> 3.5 Study references
      -> 4 Build -> 5 Render -> 6 Adversarial audit   (fail -> back to 4)
```

**1. Define the pass line.** Write the acceptance doc first: a one-line definition of "excellent",
weighted scoring dimensions, and **hard-veto items**. Read `references/acceptance-rubric.md` — the
reusable bar including the veto list. Adapt weights and vetoes to the product; keep the shape.

**2. Write the script.** One core claim, a per-shot copy table, the visual language, a storyboard.
The hook (first ~3s) must use the **audience's own words**; each shot does exactly one job; any shot
that can't be traced to the claim gets cut. Read `references/creative-brief.md`.

**3. Prepare real assets.** Hard rule: **if the product has a real screen, screenshot the real
screen; if it has real data, use the real number.** Make the demo dataset fully real first, or the
film fills with placeholders. Capture at `deviceScaleFactor=2`; capture motion as motion. Read
`references/asset-production.md`.

**3.5. Study references — twice.** Take apart **at least 3** big-company films (one is taste, three
are a rule):

- **Quantitatively** — `references/reference-teardown.md`: shot count/length, in-shot motion, beat
  alignment. Turns "the motion feels weak" into a specific dial.
- **Semantically** — `references/multimodal-teardown.md`: timestamped contact sheets + a vision model
  across 7 angles, ending with `improve_ours` (reference vs your film). Turns "it looks like a PPT"
  into concrete edits.

`references/design-rules.md` holds the rules that repeated across every reference film tested — read
it before designing anything; it is cheaper than re-deriving them.

**4. Build the composition.** One main timeline; scenes with explicit start/duration; **beat-snap to
the real pulse of the music** (analyze the track, never trust a nominal BPM); numbers interpolated
per-frame; every scene carries ambient motion. Start from `assets/starter/my-film/` — a runnable
skeleton that already encodes the proven motion, camera, and layout patterns. If using HyperFrames,
load the `hyperframes` skill tree for framework mechanics; this skill owns the *pipeline*.

**5. Render.** Pin the render command. Use a sensible CRF (a slow full-frame zoom balloons file size
at very low CRF). Gate: the renderer's own `check` passes with **zero errors**.

**6. Adversarial audit — the real core.** Don't ask "does it look good"; ask **"which frame proves it
is dead / fake / wrong"**, then measure. Details and thresholds in `references/audit-method.md`.

```bash
python3 scripts/audit_video.py renders/video.mp4 \
  --total <seconds> --grid-json assets/beat-grid.json --bgm assets/bgm.mp3 \
  --countup "t0:t1:x:y:w:h" --playback "..." --scale-windows "..." \
  --json evidence/audit.json          # exit 0 = passed, 1 = below the line
```

## Non-negotiables

- Define the pass line before building — it is the step everyone skips and the most expensive to skip.
- Real UI, real numbers. No generated mock UI passed off as the product.
- Verify music/asset licensing and attribute when required. No unclear provenance.
- Every scene boundary and beat-snap derives from the **analyzed** pulse, not a stated BPM.
- Keep the audit reproducible: every claim in the final report backed by a re-runnable command.
- **The protagonist is a person or one person's action, not the UI.** Across every reference film
  tested this held; a film with no human throughline reads as a slide deck.
- **One claim per screen, and hold the close.** One headline + at most one supporting line (≥2×
  type-size gap); fast cuts earn urgency, but a reveal or closing line holds 2–4s of near-stillness.
- **A placeholder is not a stand-in for a real asset.** If a card shows a cover, it is a real image;
  a box reading "cover" ships a film that looks broken.

See `references/design-rules.md` for the full evidence-backed rule set.

## Using it on another project

This skill is project-agnostic. To apply it elsewhere:

1. Copy `assets/starter/my-film/` to your project and swap the copy, captions, cover images, and BGM.
2. Write your own acceptance doc and creative brief from the templates in `references/`.
3. Replace the starter's placeholder assets with that product's real captures.
4. Reuse every script in `scripts/` unchanged.

Read `references/pipeline-checklist.md` for the end-to-end run order.

## References

- `references/acceptance-rubric.md` — the reusable pass line: definition, weighted dimensions, hard-veto list, checkpoints.
- `references/creative-brief.md` — script / storyboard / visual-language format and copy anti-patterns.
- `references/asset-production.md` — real UI, real motion, real data; how to keep them honest.
- `references/design-rules.md` — **the evidence-backed rule set** distilled from tearing down reference films: structure, motion, type, colour, proof, sound.
- `references/multimodal-teardown.md` — the semantic half of reference study: contact sheets + vision model, the exact call shape, the gateway traps, and the shared rules.
- `references/reference-teardown.md` — the quantitative half: many single-question passes, and the two measurements that make "it feels weak" specific.
- `references/audit-method.md` — every measurable check, its threshold, and **how the metric itself can lie**.
- `references/pipeline-checklist.md` — the shortest end-to-end run order for a new film.

## Scripts

- `scripts/audit_video.py` — adversarial auditor (exit 0/1, JSON evidence).
- `scripts/make_beat_grid.py` — real-pulse beat grid from a music file.
- `scripts/make_frame_sheets.py` — video → timestamped contact sheets for vision-model study.
- `scripts/vision_teardown.py` — feed those sheets to a vision model across 7 angles, incl. `improve_ours`.
- `scripts/teardown_video.py` — six-dimension quantitative teardown (structure / motion / text / sound / beat / colour) for comparing against reference films.

## Starter

- `assets/starter/my-film/` — a runnable 12s composition: design tokens, real-cover cards, grow-in
  motion, a 3-layer camera system, and a closing hold. `npx hyperframes@0.8.147 check` passes with
  0 errors as shipped; render it to confirm your toolchain before writing anything.
