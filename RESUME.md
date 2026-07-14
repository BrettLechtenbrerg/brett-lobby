# RESUME PROMPT — Brett Lobby project

Paste this into a fresh agent/chat session to continue work with zero context loss.

---

## Prompt

You are working on **~/brett-lobby** — a standalone "character-select lobby" website
for Brett Lechtenberg, inspired by adandelacruz.com.

- **Live:** https://lobby.brettlechtenberg.com · **Repo:** github.com/BrettLechtenbrerg/brett-lobby
- **Deploy:** push to `main` → Vercel auto-deploys in ~35s. Verify with
  `curl -s -o /dev/null -w "%{http_code}" https://lobby.brettlechtenberg.com`
- **Local dev:** `cd ~/brett-lobby && python3 -m http.server 8080`
- **Test shortcuts:** `?go=1..4` auto-enters a door; `?peek=1..4` freezes a door half-open.
  Screenshot after changes; wait ~3s for the 3D scene to settle.
- **Stack:** single-file `index.html` (Three.js r128 CDN, all CSS/JS inline),
  interior pages share `site.css`. No build step. Read README.md first.

State (July 14, 2026, v1.4 — SIX live concepts, all sharing the same six-persona
SUITES data + dossier layout, each a standalone single-file page):

| Concept | File / URL | Metaphor |
|---|---|---|
| A — Lobby doors | index.html · / | walk through a door (Three.js 3D) |
| B — Tower | tower.html · /tower | lit windows in a building at night |
| C — Character select | select.html · /select | fighting-game roster |
| D — Elevator | elevator.html · /elevator | brass elevator, floors L–6 |
| E — BLTV | broadcast.html · /broadcast | TV studio, channel surfing |
| F — Switchboard | switchboard.html · /switchboard | 1940s operator exchange |

Concept notes:
- D Elevator: "The Brett Lechtenberg Building" brass plaque (plaque-face.png medallion),
  portrait floor buttons, floors top-down 6 Author / 5 Sensei / 4 Trainer / 3 AI /
  2 Coach / 1 Speaker (no basement — Sensei moved up per user), stepping indicator,
  hum + WebAudio ding, keys 1–6/esc.
- E BLTV: studio control room — physical monitor w/ bezel + stand + ON AIR tally,
  preview monitor wall (six live hero-jpg feeds), lower-third + ticker inside screen,
  static burst + hiss on channel change, keys ↑↓/1–6/enter/esc.
- F Switchboard: "The Lechtenberg Exchange" — wooden cabinet, engraved brass nameplate,
  centered face chips ({key}-face.png), jack sockets, animated patch cord + ring burr,
  idle cords onto operator desk, generated exchange-hall.jpg backdrop (source png in
  source-assets/) with slow lamp-glow breathing.
- Persona faces: assets/{key}-face.png (auto-centered head chips, script in git history
  commit a501ac4); assets/plaque-face.png is the branded medallion (full head).
- index.html (Concept A) unchanged from v1.3: 3D walk-through cinematic, Foundation
  overlay w/ Author+Sensei, quiz pill, mobile DOM fallback.

Outstanding / next candidates:
1. USER DECISION: run testers across all six concepts, watch which one they finish,
   then promote the winner to / and retire the rest (or keep as easter eggs).
2. speaker/coach/trainer `{key}-hero.jpg` backdrops still show Brett's OLD blue-tee
   outfit — regenerate with gpt-image-2 to match each door's outfit (ai-hero.jpg done).
3. `{key}-idle.mp4` / `{key}-beat.mp4` clone videos not shot yet (specs in README).
4. Watch quiz-pill pulse annoyance; consider capping at 3 cycles.
5. If mobile users report a black screen on /: likely WebGL blocked — mobile path
   is DOM-only so should be immune, but check the in-app browser involved.
6. Possible: cross-links between concepts (currently each links back to / only).

Rules: keep edits small, screenshot-verify, commit+push after each accepted change,
never touch the main brettlechtenberg.com site.

---

## How-to: regenerate the four door stills

Sources in `source-assets/` (originals — do not delete):
`brett-fullbody-nobg.webp` (speaker, transparent), `brett-{coach,ai,trainer}-outfit2.png`
(AI outfit variants on black), `brett-headshot-nobg.webp` (favicon source).
(`brett-author-outfit2.png` kept in case the Author door returns.)

```bash
cd ~/brett-lobby && python3 - <<'EOF'
from PIL import Image, ImageDraw, ImageFilter
from collections import deque

def extract_figure(path):  # flood-fill near-black bg from edges -> transparent
    im = Image.open(path).convert('RGBA'); px = im.load(); W, H = im.size
    THR = 26; seen = bytearray(W * H); q = deque()
    for x in range(W): q.append((x, 0)); q.append((x, H - 1))
    for y in range(H): q.append((0, y)); q.append((W - 1, y))
    while q:
        x, y = q.popleft()
        if x < 0 or y < 0 or x >= W or y >= H: continue
        i = y * W + x
        if seen[i]: continue
        r, g, b, a = px[x, y]
        if max(r, g, b) > THR: continue
        seen[i] = 1; q.extend(((x+1,y),(x-1,y),(x,y+1),(x,y-1)))
    for y in range(H):
        for x in range(W):
            if seen[y * W + x]: px[x, y] = (0, 0, 0, 0)
    return im

AURAS = {'speaker': (212,175,55), 'coach': (196,50,74), 'ai': (63,193,201), 'trainer': (150,160,155)}
SRC = {'speaker': 'source-assets/brett-fullbody-nobg.webp',
       'coach': 'source-assets/brett-coach-outfit2.png',
       'ai': 'source-assets/brett-ai-outfit2.png',
       'trainer': 'source-assets/brett-trainer-outfit2.png'}
W, H = 512, 2180; BASE_Y = int(H * 0.97); MAX_W = int(W * 0.88)

figs = {}
for key, src in SRC.items():
    fig = Image.open(src).convert('RGBA') if key == 'speaker' else extract_figure(src)
    figs[key] = fig.crop(fig.getchannel('A').getbbox())

FIG_H = int(H * 0.55)  # uniform height, clamped by widest figure
for fig in figs.values():
    if fig.width * (FIG_H / fig.height) > MAX_W:
        FIG_H = min(FIG_H, int(fig.height * (MAX_W / fig.width)))

for key, fig in figs.items():
    s = FIG_H / fig.height
    fig = fig.resize((int(fig.width * s), FIG_H), Image.LANCZOS)
    ar, ag, ab = AURAS[key]
    canvas = Image.new('RGB', (W, H), (8, 7, 9)); d = ImageDraw.Draw(canvas)
    for y in range(H):
        f = max(0, 1 - abs(y / H - 0.55) * 2.1) * 0.42
        d.line([(0, y), (W, y)], fill=(int(8 + ar * f), int(7 + ag * f), int(9 + ab * f)))
    rim = Image.new('L', (W, H), 0); rd = ImageDraw.Draw(rim)
    cx, cy, R = W // 2, int(H * 0.72), int(W * 0.95)
    rd.ellipse((cx - R, cy - R, cx + R, cy + R), fill=90)
    rim = rim.filter(ImageFilter.GaussianBlur(120))
    canvas = Image.composite(Image.blend(canvas, Image.new('RGB', (W, H), (ar, ag, ab)), 0.35), canvas, rim)
    canvas.paste(fig, ((W - fig.width) // 2, BASE_Y - FIG_H), fig)
    canvas.save(f'assets/{key}-still.png')
EOF
```

## How-to: outfit variants (gpt-image-2)

Edit `source-assets/brett-fullbody-nobg.webp` with a prompt like:
"Change ONLY his clothing: replace the dark suit jacket and blue t-shirt with …
Keep his face, mustache, gray hair, pose, hands-in-pockets stance, body proportions,
and the plain black background exactly the same. Photorealistic, full body visible."
Size 1024x1536, quality high. Pick LIGHT garment colors — dark ones vanish into the doors.

## Favicon regeneration

Circular crop of `source-assets/brett-headshot-nobg.webp` on #070607 with gold ring
(#D4AF37), sizes 32/192/512 + multi-size `favicon.ico`. Script in git history
(commit "Cinematic 3-beat door walk-through … + face favicon on all pages").
