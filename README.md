# For Lalima

A single-page birthday site. No build step, no dependencies, no framework.
Everything lives in `index.html`; photos live in `uploads/`.

## Deploy to GitHub Pages

```bash
git init
git add -A
git commit -m "For Lalima"
git branch -M main
git remote add origin git@github.com:<you>/<repo>.git
git push -u origin main
```

Then: **Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)`**.

`.nojekyll` is present so Pages serves every file as-is.

## Photos

`uploads/` holds 12 processed JPEGs, all 900x1200 (3:4), ~1.8 MB total:

| file | where it appears | from |
|---|---|---|
| `hero.jpg` | screen 1, the big polaroid | `20260124_162804.jpg` |
| `trio-1..3.jpg` | screen 2, the three tilted frames | `20260125_153330`, `IMG-20250308-WA0058`, `PXL_20250309_191136638` |
| `gallery-1..8.jpg` | screen 4, the 2-column grid | the remaining eight |

Originals came from `~/Downloads/Photos-1-001`. Each was EXIF-rotated,
centre-cropped to 3:4, resized to 900x1200, and saved at quality 78 with
**all metadata stripped** — the originals carried GPS coordinates, and this
page is public.

`gallery-8.jpg` is the three-person selfie. Its subjects span 71% of the
frame and a 3:4 window only covers 56%, so it is anchored at 55% to keep
Lalima centred; the two on the ends are deliberately half-cropped.

To swap a photo, overwrite the file in `uploads/` at 3:4 — no code change
needed. Re-crop with:

```python
from PIL import Image, ImageOps
im = ImageOps.exif_transpose(Image.open("new.jpg")).convert("RGB")
w, h = im.size; r = 0.75
nw, nh = (int(h*r), h) if w/h > r else (w, int(w/r))
cx = 0.5                                  # 0=left edge, 1=right edge
l = min(max(int(cx*w - nw/2), 0), w - nw)
im.crop((l, (h-nh)//2, l+nw, (h-nh)//2+nh)) \
  .resize((900, 1200), Image.LANCZOS) \
  .save("uploads/hero.jpg", quality=78, optimize=True, progressive=True)
```

Only `hero.jpg` loads up front; the rest are lazy-loaded per screen.

## The "send it" button (screen 7)

GitHub Pages is static hosting — no server, no database, nothing that can
receive a POST. So her note goes to a third-party endpoint. Default is
**Web3Forms**, which emails it to you.

**Setup, ~2 minutes, no account needed:**

1. Go to <https://web3forms.com>
2. Type your email into "Create Access Key"
3. They email you a key
4. Paste it into `CONFIG.send.key` at the top of the `<script>` in `index.html`

The key is safe in public source: it maps to your email on their server, so
your address never appears on this page.

Also set `fallbackPhone` (digits, country code first). If the request fails —
dead wifi, service down — the page keeps her note and offers a WhatsApp link
so it still reaches you instead of vanishing.

```js
send: {
  mode: 'web3forms',
  key: 'your-key-here',
  fallbackPhone: '91XXXXXXXXXX'
}
```

**Until you paste a key, the button says delivery is not set up.** It does not
claim the message was sent.

Behaviour, verified end-to-end:

| state | what she sees | what happens |
|---|---|---|
| sending | `sending…`, button locked | 12s timeout so it can't hang on mobile data |
| success | `sent` + confirmation, confetti | draft cleared from `localStorage` |
| failure | `try again` + WhatsApp link | note kept in the box *and* in `localStorage` |

The draft is only deleted once delivery is confirmed. Repeated taps while a
request is in flight are ignored, so you can't get duplicates.

Other modes if you'd rather: `whatsapp` (zero setup, opens WhatsApp
pre-filled) or `formspree` (needs an account, 50 submissions/month free).

## Notes

- `#skip` in the URL jumps straight past the candle intro (`.../index.html#skip`).
- Music is a small Web Audio melody, off by default — browsers block autoplay
  until a tap, so it's a manual toggle in the bottom bar.
- Tap the small dot in the top bar four times.
- Nav: swipe left/right, arrow keys, the dots, or the buttons.
