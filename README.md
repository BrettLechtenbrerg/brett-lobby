# Brett Lechtenberg — The Lobby

**Live:** https://lobby.brettlechtenberg.com (also https://brett-lobby.vercel.app)
**Repo:** https://github.com/BrettLechtenbrerg/brett-lobby
**Deploys:** Vercel, auto-deploy on push to `main` (~35s)

Character-select home screen (inspired by adandelacruz.com). Standalone project —
does NOT touch the live brettlechtenberg.com site.

## Run it

```bash
cd ~/brett-lobby
python3 -m http.server 8080
# open http://localhost:8080
```

Hover a door, click it (or press 1–4), Esc / "back to the lobby" to return.
Debug: `?go=1`..`?go=4` auto-opens a dossier · `?peek=1`..`?peek=4` freezes a door half-open.

Pages: `index.html` (lobby) · `about.html` · `speaking.html` · `training.html` · `books.html` · `contact.html`
(interior pages share `site.css`; all copy is REAL, pulled from brettlechtenberg.com).

## Current feature set (July 2026)

- **3D lobby** (Three.js r128 CDN): background plate, 4 hinged glass doors labeled
  DOOR 1–4, idle bob, breathing frame glow, occasional idle "door crack" with light sliver
- **Walk-through cinematic** (3 beats, ~3.4s): approach & stop off-axis → door slab
  (BoxGeometry with lit edge) swings ~86° toward viewer + hinge-creak sound →
  footstep-bob dolly through → fade-to-black threshold → dossier lights up
- **Dossier**: pins at top, bouncing SCROLL cue (no auto-scroll), per-persona hero +
  optional beat video, quiet quiz link under the CTA
- **"Which Brett" quiz**: pulsing gold pill above the nav (lobby) + text link in every
  dossier; 4 questions, keyboard 1–4; result opens the winning door (switches rooms
  if you're already inside one); links to the full Rockstar Team Quiz on the main site
- **Hint pill** mid-screen: "← click a door to step inside →", pulses, gone after first click
- **Face favicon** (circular headshot, gold ring): `favicon.ico` + `assets/favicon-*.png`, all pages
- **Sound**: WebAudio blips + hinge creak; ambient music with pause toggle
- Mobile portrait shows a rotate-your-device gate

## The four personas (edit in `index.html` → `CLASSES` array)

| # | Key       | Persona                | Accent    | Outfit (still) |
|---|-----------|------------------------|-----------|----------------|
| 1 | `speaker` | The Keynote Speaker    | Gold      | dark suit + blue tee (original photo) |
| 2 | `coach`   | The Performance Coach  | Cranberry | light heather-gray quarter-zip |
| 3 | `author`  | The Author             | Gold-yellow | camel tweed + cream turtleneck |
| 4 | `trainer` | The Corporate Trainer  | Silver    | white shirt + navy vest |

Outfits 2–4 are AI-edited (gpt-image-2) from the same original photo — face/pose identical.

## Assets

| File                  | Status | Notes |
|-----------------------|--------|-------|
| `assets/lobby.jpg`    | ✅ | AI-generated high-rise dojo plate. If regenerated, re-measure `PANEL_U`/`PANEL_V0`/`PANEL_V1` in `index.html` |
| `assets/{key}-hero.jpg` | ✅ | Dossier hero backdrops. ⚠️ Still show the OLD blue-tee outfit — regenerate to match new outfits if desired |
| `assets/{key}-still.png` | ✅ | Door photo panels, uniform figure size (feet on same baseline) |
| `assets/favicon-*.png`, `favicon.ico` | ✅ | Face favicon |
| `assets/ambient-music.mp3` | ✅ | Lobby ambience |
| `assets/{key}-idle.mp4` | ⏳ | Looping "alive" clip per door (replaces still). 3–8s, vertical, <5 MB, muted |
| `assets/{key}-beat.mp4` | ⏳ | Dossier intro/clone video. 6–12s, 16:9, <4 MB |
| `source-assets/`      | ✅ | ORIGINALS — cutouts + AI outfit variants. Needed to regenerate stills. Do not delete |

`{key}` = speaker / coach / author / trainer. (Old `sensei-*` assets kept in `assets/`.)

## Regenerating the door stills

The stills are built from `source-assets/` by a Python/PIL pipeline (flood-fill background
removal → uniform figure height → aura gradient + rim glow → paste at shared baseline).
The script lives in git history (commit "Uniform figure size…") and in RESUME.md §How-to.

## Photo shoot checklist (for real AI clone videos)

- Full-body, standing, facing camera, feet visible; soft even light, plain/dark background
- One outfit per persona (match the table above so doors stay consistent)
- Shoot 4K video: 10s standing still + one small gesture per persona
  (point, arms crossed, book in hand, whiteboard gesture)

## Reference: how Adan's works (from source analysis)

- Static HTML + Three.js r128 CDN, hosted on Vercel
- Background = 4K plate image + ambient video texture
- Each pane = video texture mapped onto measured UV regions
- Click → raycast → camera lerp → flare → fullscreen dossier with beat video
- Ambient music home-screen only, fades on select
- Contact form: Web3Forms; analytics: GA4
