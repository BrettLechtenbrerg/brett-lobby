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

State (July 14, 2026, v1.2 — Door 3 is now The AI Strategist, teal accent, silver-bomber
outfit, matching ai-hero.jpg; Author persona retired but assets kept): walk-through door cinematic (3 beats + creak + fade-to-black
threshold), DOOR 1–4 labels, idle door-crack animation, uniform-size door photos with
distinct AI-edited outfits, "which Brett" quiz (lobby pill + dossier links), mid-screen
pulsing hint, top-pinned dossiers with scroll cue, face favicon, full mobile support
(portrait 2×2 door cards, short-landscape/in-app browsers 4-across, `MQ_MOBILE`
media query gates the 3D path; renderer paused on mobile).

Outstanding / next candidates:
1. speaker/coach/trainer `{key}-hero.jpg` backdrops still show Brett's OLD blue-tee
   outfit — regenerate with gpt-image-2 to match each door's outfit (ai-hero.jpg done).
2. `{key}-idle.mp4` / `{key}-beat.mp4` clone videos not shot yet (specs in README).
3. Watch quiz-pill pulse annoyance; consider capping at 3 cycles.
4. Tester feedback loop on door affordance (hint + breathing glow + enter chip).
5. If mobile users report a black screen anyway: likely WebGL blocked — mobile path
   is DOM-only so should be immune, but check the in-app browser involved.

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
