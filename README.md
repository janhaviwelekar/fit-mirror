# 🪞 FitMirror

> **Paste any clothing link. See it on you. Live.**

FitMirror is a browser-native virtual try-on app — no installation, no backend, no build step. Paste a product URL from Amazon, Myntra, Zara, ASOS, or 10+ other sites, and Claude AI extracts the garment details while MediaPipe overlays it on your live camera feed in real time.

---

## ✦ Demo

```
Paste URL  →  Claude AI reads it  →  MediaPipe finds your body  →  Garment draws live at 60fps
```

**Supported garment types:** Maxi dress · Midi dress · Mini dress · Top · Crop top · Jacket · Shirt

---

## 🛍️ Supported Sites

| Site | Domain |
|---|---|
| Amazon India | amazon.in |
| Amazon Global | amazon.com |
| Myntra | myntra.com |
| Flipkart | flipkart.com |
| Zara | zara.com |
| ASOS | asos.com |
| Ajio | ajio.com |
| Meesho | meesho.com |
| H&M | hm.com |
| Uniqlo | uniqlo.com |
| Nykaa Fashion | nykaa.com |
| SHEIN | shein.com |
| Mango | mango.com |
| Forever 21 | forever21.com |
| **Any other site** | Claude still attempts extraction |

---

## ⚡ Features

- **Universal URL input** — paste any product link and Claude infers the garment from the URL slug, path segments, and its knowledge of each retailer's catalog
- **Real-time body tracking** — MediaPipe Pose detects 33 body keypoints at ~15fps; garment anchors to live shoulder and hip positions
- **7 garment renderers** — each shape drawn with gradients, seams, necklines, buttons, and fold details via HTML5 Canvas
- **Color swatches** — switch between up to 3 product colors extracted by Claude; overlay updates instantly
- **Fit controls** — opacity slider, scale slider, vertical position nudge, size selector (XS–XXL)
- **Session history** — last 8 tried items stored in-session; one-click reload
- **Photo capture** — download the current frame as a PNG with a single click
- **Zero dependencies** — no npm, no build step, no server; just 11 files in a folder

---

## 🚀 Quick Start

**1. Clone the repo**
```bash
git clone https://github.com/yourusername/fitmirror.git
cd fitmirror
```

**2. Open in Chrome or Edge**
```bash
# Simply double-click index.html
# OR serve locally to avoid any file:// restrictions:
npx serve .
```

**3. Paste a link and try it on**
- Click **Enable Camera** and allow access
- Paste any clothing product URL into the box
- Press **Enter** or click **Analyse & Try On**
- Wait 1–3 seconds for Claude to extract the garment
- See it live on you — move around!

> **Note:** Requires an active internet connection for MediaPipe CDN, Google Fonts, and the Anthropic API.

---

## 🔑 API Key Setup

The prototype calls the Anthropic API directly from the browser — fine for local development.

For any public deployment, proxy the call through a backend:

```js
// Current (dev only — exposes key in browser)
fetch('https://api.anthropic.com/v1/messages', { ... })

// Production — create a POST /extract endpoint on your server
fetch('https://your-backend.com/extract', { body: JSON.stringify({ url, site }) })
```

Your backend stores the key in an environment variable:
```bash
ANTHROPIC_API_KEY=sk-ant-...
```

---

## 🗂️ Project Structure

```
fitmirror/
├── index.html          ← HTML skeleton only — zero inline logic
├── style.css           ← All CSS, 12 sections, fully commented
└── js/
    ├── state.js        ← All shared variables (single source of truth)
    ├── main.js         ← Entry point — boots app, owns the Try On button
    ├── camera.js       ← getUserMedia + MediaPipe Pose + 60fps render loop
    ├── renderer.js     ← Canvas drawing engine — all 7 garment shapes
    ├── api.js          ← Claude API call — URL → garment JSON
    ├── garment.js      ← Apply / clear / toggle garment state
    ├── ui.js           ← DOM updates — cards, sliders, progress, errors
    ├── history.js      ← Session history — add, render, reload
    └── utils.js        ← Helpers — clipboard, getSite, shiftColor, toast
```

Each file has exactly one job. No file mixes concerns.

---

## 🧠 How It Works

### The two flows

```
URL Flow:
  User pastes URL
    → main.js validates
      → api.js calls Claude
        → garment JSON returned
          → garment.js writes to state
            → renderer.js draws on next frame

Camera Flow (60fps):
  Camera stream
    → camera.js sends frame to MediaPipe every 66ms
      → 33 landmarks stored in state.currentPose
        → renderer.js reads on every animation frame
```

### Garment anchor algorithm

```js
// 1. Read key landmarks from MediaPipe's 33-point model
const ls = landmarks[11];  // left shoulder
const rs = landmarks[12];  // right shoulder

// 2. Convert normalised (0–1) coords to canvas pixels
const shoulderWidth = Math.abs(rs.x - ls.x) * canvas.width;
const midX          = ((ls.x + rs.x) / 2) * canvas.width;
const midY          = ((ls.y + rs.y) / 2) * canvas.height;

// 3. Scale garment to body
const gW = shoulderWidth * garment.widthRatio * scale;
const gH = computeHeight(garment.type, ...);  // 7 different formulas

// 4. Draw at anchor
ctx.globalAlpha = opacity;
renderShape(midX - gW/2, midY - gH*0.06 + offset, gW, gH, ...);
```

### Claude prompt

Claude receives only the URL and site name — no image, no camera data:

```
Extract clothing product details from this URL.
Return ONLY valid JSON: { name, brand, price, type, colors, widthRatio, fit, emoji, description }
type must be exactly one of: maxi | midi | mini | top | crop | jacket | shirt
```

---

## ⚙️ Technical Details

| Metric | Value |
|---|---|
| Canvas render rate | 60fps |
| Pose detection rate | ~15fps (throttled to 66ms) |
| MediaPipe model | Pose v0.5, complexity:1 |
| WASM load time | ~1.8s on first open |
| AI extraction latency | 1–3 seconds |
| Claude model | claude-sonnet-4-20250514 |
| Camera mirror | `ctx.scale(-1,1)` + landmark x-flip |
| Privacy | Camera data never leaves browser |

---

## 🔧 Extending

### Add a new garment type

1. Add the type to the Claude prompt in `api.js`
2. Write a draw function in `renderer.js`: `function drawSaree(x, y, w, h, hex, light, dark) { ... }`
3. Add a case in `renderShape()`: `case 'saree': drawSaree(...); break;`
4. Add height logic in `computeHeight()`: `case 'saree': return remaining * 0.95 * scale;`

### Add a new shopping site

In `utils.js`, add one line to the `getSite()` map:
```js
'newsite.com': 'New Site Name',
```

That's it. Claude will already attempt extraction from any URL.

---

## 🗺️ Roadmap

- [ ] Real product image overlay (needs backend CORS proxy + image warping)
- [ ] MediaPipe Selfie Segmentation — arms naturally in front of garment
- [ ] Multi-garment layering (top + bottom simultaneously)
- [ ] Save looks to account (Supabase/Firebase)
- [ ] AI photo mode — IDM-VTON or OOTDiffusion for photorealistic still images
- [ ] Mobile app (React Native / Capacitor)

---

## 🌐 Browser Support

| Browser | Support |
|---|---|
| Chrome 90+ | ✅ Full |
| Edge 90+ | ✅ Full |
| Firefox 88+ | ⚠️ Partial (slower WASM) |
| Safari 15+ | ⚠️ Partial (iOS camera limits) |

---

## 📄 License

MIT — see [LICENSE](LICENSE)

---

<div align="center">
  <sub>Built with MediaPipe Pose · Anthropic Claude · HTML5 Canvas · Zero frameworks</sub>
</div>
