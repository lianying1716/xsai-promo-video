# xsai · 产品宣传视频制作

`xsai-promo-video` — a code-only product promo video pipeline with measurable, vision-model-assisted audit.

Build a product promo / feature / launch video **entirely from code plus real product screenshots** —
no editor, no timeline scrubbing, no manual cutting. Built for people who can write code but don't
edit video.

The point is not the finished film. The point is that **every judgment about the film can be
disproven by a command**: "the motion feels weak" becomes a frame-difference number, "it looks like a
PPT" becomes a shot-length and in-shot-motion count, "the beat is off" becomes a millisecond offset.

```text
1 Define pass line -> 2 Script -> 3 Real assets -> 3.5 Study references
      -> 4 Build -> 5 Render -> 6 Adversarial audit   (fail -> back to 4)
```

## What makes it different

- **A pass line before a single frame.** The acceptance rubric (weighted dimensions + hard-veto
  items) is written first, so "is it better now" is never a matter of taste.
- **An adversarial auditor, not a vibe check.** `audit_video.py` measures spec, opening audio energy,
  per-block motion (splitting *frozen* from *low-motion*), blank/black frames, beat drift, push-in
  scale via region template matching, real-video playback, and count-up continuity — and exits 0/1.
- **Reference study done twice.** Quantitatively (`teardown_video.py`) and semantically
  (`make_frame_sheets.py` + `vision_teardown.py`: timestamped contact sheets fed to a vision model
  across 7 angles). One reference is taste; three are a rule.
- **A runnable starter** that already encodes the proven patterns and passes the renderer's check
  with 0 errors as shipped.

## Quick start

```bash
cp -r assets/starter/my-film /path/to/your/film && cd /path/to/your/film
npx hyperframes@0.8.147 check                 # 0 errors as shipped
npx hyperframes@0.8.147 render -o renders/smoke.mp4 --crf 24
```

Then read `SKILL.md` and work the six steps. `references/pipeline-checklist.md` is the short version.

## Layout

```text
xsai-产品宣传视频制作/
├── SKILL.md                     the entrypoint: workflow, non-negotiables, routing
├── references/
│   ├── acceptance-rubric.md     the reusable pass line (dimensions + hard vetoes)
│   ├── creative-brief.md        script / storyboard / visual-language format
│   ├── asset-production.md      real UI, real motion, real data
│   ├── design-rules.md          the four rules every reference film obeys, with the measurements
│   ├── multimodal-teardown.md   semantic reference study via a vision model
│   ├── reference-teardown.md    quantitative reference study
│   ├── audit-method.md          every check, its threshold, and how the metric lies
│   └── pipeline-checklist.md    end-to-end run order
├── scripts/
│   ├── audit_video.py           adversarial auditor (exit 0/1, JSON evidence)
│   ├── make_beat_grid.py        real-pulse beat grid from a music file
│   ├── make_frame_sheets.py     video → timestamped contact sheets
│   ├── vision_teardown.py         contact sheets → 7-angle vision-model teardown
│   └── teardown_video.py        six-dimension quantitative teardown
└── assets/
    ├── fonts/                   JetBrains Mono (OFL)
    └── starter/my-film/         a runnable 12s composition
```

## Requirements

- **Node** (for `npx hyperframes`) — the default renderer. The pipeline is framework-agnostic; any
  renderer that outputs an MP4 works, only steps 4–5 touch the framework.
- **Python 3.9+** with `numpy`, `scipy`, `librosa`, `Pillow`.
- **ffmpeg / ffprobe** on `PATH`.
- Optional: an OpenAI-compatible vision model for `vision_teardown.py` (set `VISION_ENDPOINT`,
  `VISION_MODEL`, `VISION_API_KEY`). The semantic teardown degrades gracefully to the
  quantitative one if you don't have one.

## Design rules in one paragraph

The protagonist is a person or one person's action, never the UI by itself. Something moves inside
every shot (camera + ambient + content). One claim per screen, and the close holds 2–4s. Colour is a
closed system of a low-saturation base plus one accent that carries meaning. Everything else follows
from those four — see `references/design-rules.md` for the measurements behind them.

## License

MIT — see `LICENSE`. Third-party assets are listed in `THIRD_PARTY_NOTICES.md`.
