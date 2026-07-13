# Brett Lechtenberg — The Lobby

**Live:** https://brett-lobby.vercel.app
**Repo:** https://github.com/BrettLechtenbrerg/brett-lobby

Character-select home screen (inspired by adandelacruz.com). Standalone project —
does NOT touch the live brettlechtenberg.com site.

## Run it

```bash
cd ~/brett-lobby
python3 -m http.server 8080
# open http://localhost:8080
```

Works today — lobby plate and dossier hero images are already generated.
Hover a panel, click it (or press 1–4), Esc / "back to the lobby" to return.
Debug: `?go=1`..`?go=4` auto-opens a dossier.

Pages: `index.html` (lobby) · `about.html` · `speaking.html` · `books.html` · `contact.html`
(all interior pages share `site.css` and carry REAL copy pulled from brettlechtenberg.com).

## The four personas (edit in `index.html` → `CLASSES` array)

| # | Key       | Persona                | Accent    |
|---|-----------|------------------------|-----------|
| 1 | `speaker` | The Keynote Speaker    | Gold      |
| 2 | `coach`   | The Performance Coach  | Cranberry |
| 3 | `author`  | The Author             | Blue      |
| 4 | `trainer` | The Corporate Trainer  | Silver    |

All persona copy is REAL (sourced from brettlechtenberg.com). Tweak freely.

## Assets to drop into `assets/` (all optional; site upgrades automatically)

| File                  | Status | What it is                                              | Spec |
|-----------------------|--------|---------------------------------------------------------|------|
| `lobby.jpg`           | ✅ DONE (AI-generated high-rise dojo) | Background plate | 16:10, dark |
| `{key}-hero.jpg`      | ✅ DONE (AI-generated, all 4) | Dossier hero backdrop / video poster | 16:9 |
| `{key}-still.png`     | ✅ DONE (real photos of Brett, composited) | Photo panel shown in each door until video exists | tall 1:4.26 |
| `{key}-idle.mp4`      | ⏳ needs Brett footage | Looping "alive" clip per persona (replaces still) | 3–8s loop, vertical ~1:3 crop OK, <5 MB, muted |
| `{key}-beat.mp4`      | ⏳ needs Brett footage | Short intro/clone video per persona (dossier) | 6–12s, landscape 16:9, <4 MB |
| `ambient-music.mp3`   | ⏳ pick a track | Lobby ambience (home screen only) | loopable, quiet |

Note: the four glass-door panes are UV-mapped to the doors in `lobby.jpg`
(measured by brightness-run detection). If you regenerate `lobby.jpg`, re-measure
and update `PANEL_U` / `PANEL_V0` / `PANEL_V1` in `index.html`.

`{key}` = speaker / coach / author / trainer.
(Old `sensei-*` assets remain in `assets/` in case the mystery persona returns.)

## Photo shoot checklist (for the AI clone videos)

- Full-body, standing, facing camera, feet visible
- Even, soft lighting; plain or dark background (easy to cut out)
- One outfit per persona:
  - Speaker: sharp suit / stage look
  - Coach: smart casual (blazer, no tie)
  - Author: relaxed professional (book in hand works great)
  - Sensei: gi or black training wear (hood/shadow optional for mystery)
- Shoot 4K video too if possible: 10s of standing still + one small gesture
  per persona (point, arms crossed, bow, page turn) — makes far better
  AI idle/beat clips than stills alone.

## Reference: how Adan's works (from source analysis)

- Static HTML + Three.js r128 CDN, hosted on Vercel
- Background = 4K plate image + ambient video texture
- Each pane = video texture (`{key}-idle.mp4`) mapped onto measured UV regions
- Click → raycast → camera lerp to door → white flare → fullscreen dossier
  playing `{key}-beat.mp4`, auto-scrolls to bio when video ends
- Ambient music home-screen only, fades on select
- Contact form: Web3Forms; analytics: GA4
