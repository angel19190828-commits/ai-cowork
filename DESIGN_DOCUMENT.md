# AI COWORK ® STUDIO — Design Document
**Project:** AI COWORK Visual Intelligence Website  
**Stack:** Vanilla HTML / CSS / JavaScript · Three.js · GSAP ScrollTrigger  
**Live:** `https://angel19190828-commits.github.io/ai-cowork/`  
**Local:** `localhost:3000` via `npx serve`

---

## 1. Project Vision

A single-page portfolio website for an AI product studio. The design language is rooted in **editorial minimalism** — inspired by nutrition labels, receipt printers, and financial data terminals. Three.js 3D objects float in the background as ambient motion. Everything feels considered, slow, and intentional.

**Design principles agreed on:**
- Off-white background `#f4f2ef` — not pure white, feels like paper
- Ultra-light Inter typeface (weight 100) for all large display text
- PT Mono for all labels, metadata, and UI text
- No decorative gradients — depth through opacity and layering
- 3D objects are atmosphere, not hero content

---

## 2. Architecture

### Single Scene, Two Modes
One Three.js `WebGLRenderer` handles everything. It runs in two modes:

| Mode | Behavior |
|------|----------|
| `hero` | 3 torus knots orbit in a horizontal circle at 30° top-down camera |
| `service` | Single knot floats to the LEFT of the active product card |

A `heroInView` flag controls whether the orbital animation runs — set to `false` once the hero scrolls off-screen, preventing knots from ghosting into the intro section.

### Camera
```
position: (0, 5.0, 8.66)   ← 30° top-down angle
lookAt: (0, 0, 0)
FOV: 38°
```
tan(30°) = 0.577 → y=5, z=8.66. This angle makes the orbital ring read as elliptical from the viewer's perspective — more dynamic than flat top-down.

### Metal Material
```javascript
MeshStandardMaterial {
  color: 0xb0b4c4,
  metalness: 1.0,
  roughness: 0.08,
  envMapIntensity: 3.5
}
```
Paired with a canvas-generated equirectangular env map: warm key light at top, cool blue reflection at bottom. PMREMGenerator processes it. Result: chrome-like surface that reads as light metal on the off-white background.

---

## 3. Components

### Hero
- 4-line headline: DIGITAL / PRODUCTS / FOR AI / BUILDERS
- Words animate in with `yPercent: 105 → 0` on load
- 3 torus knots orbit at `ORBIT_SPEED: 0.22 rad/s`, spaced 120° apart
- `ORBIT_CX: 0.8` shifts the ring right so it doesn't cover the headline text
- On scroll: knots exit upward (`exitY = heroScrollProg * 7.0`)

### Intro / Description Section
- Full-viewport section between hero and product cards
- Large headline: THE NEW WAY THINGS GET BUILT
- Two body paragraphs + 4-step workflow grid (Define → Orchestrate → Iterate → Ship)
- Background `∞` glyph at `opacity: 0.025`, `font-size: 38vw`
- Parallax on glyph and headline via ScrollTrigger scrub

### Product Cards (Nutrition Label Style)
Each card is a `card-flip` container with front and back faces:

**Front:** Nutrition label aesthetic
- Large `#01` number at top
- Title bar
- Empty body zone (space for the 3D knot to "live behind")
- Footer with barcode SVG + stats grid

**Back:** Dark flip face (`#0d0d10`)
- "Opening Project" label
- Title + loading bar animation
- Redirects to `work1.html` on complete

**Interactions:**
- `mouseenter` → subtle shake `±2px / ±0.25°` over 0.39s
- `click` → GSAP `rotateY(180)` → loading bar → `window.location.href`

### Receipt (Bottom Right)
Fixed position, PT Mono, uppercase. Behaves like a real receipt printer:
- Starts with `ENTER TO DISCOVER`
- Each visited product **appends** a new line — never replaces, never resets
- Separator only between header and items
- Items accumulate: Product 1 stays when Product 2 prints, etc.
- `printedItems` Set prevents re-printing on scroll-back

### Lateral Text
Fixed left-side vertical text (`writing-mode: vertical-rl`)
- Default: `AI COWORK`
- Changes to product name on section enter
- Character split animation: `yPercent: 110 → 0` on in, `yPercent: -110` on out

---

## 4. What Failed (and Why)

### ❌ GLTFLoader CDN
**Problem:** `THREE.GLTFLoader` wasn't available from the CDN path used.  
**Result:** 3D models couldn't load.  
**Fix:** Graceful fallback to `TorusKnotGeometry`. Code checks `typeof THREE.GLTFLoader !== 'undefined'` before instantiating.

### ❌ Fast-Scroll Overlap
**Problem:** Scrolling quickly through sections caused multiple torus knots to scale in simultaneously — they stacked on top of each other.  
**Root cause:** Async GSAP callback chains from `enter()` and `leave()` running out of order.  
**Fix:** `showKnot(i)` now **synchronously** kills all other tweens and `gsap.set` scales them to 0.001 before animating the target knot in. No async gap.

### ❌ CSS Flip Conflict
**Problem:** CSS rule `.card-inner.flipped { transform: rotateY(180deg) !important }` conflicted with GSAP's inline transform.  
**Fix:** Removed the CSS rule entirely. Flip is handled 100% by GSAP `rotateY(180)`.

### ❌ Card Design Rejected (3D Pane Split)
**Problem:** Rebuilt cards as a split layout — 3D model on left pane, info on right pane inside the card.  
**User feedback:** "NO. Rewind. The GLBs are separate from the card — they're in the background to the LEFT."  
**Fix:** Restored original nutrition label card. 3D models float independently in the canvas behind/beside the card.

### ❌ heroInView Ghost Models
**Problem:** Scrolling back up through the intro section would show the orbital torus knots floating over the description text.  
**Fix:** Added `heroInView` boolean flag. `onLeave` of hero ScrollTrigger hides all knots. `onEnterBack` restores them. The intro section has no knowledge of the 3D scene.

### ❌ Python Server Can't Handle Extension-less URLs
**Problem:** `localhost:3000/work1` returned 404. Python's `http.server` only serves exact file paths.  
**Fix:** Switched to `npx serve` which maps `/work1` → `work1.html` automatically.

### ❌ WebGL Context Conflict (work1.html)
**Problem:** Script called `canvas.getContext('webgl')` on the actual background canvas to test for WebGL support. This locked the canvas to a raw context. When Three.js tried to create its own context on the same canvas, it failed: *"existing context of a different type"*.  
**Fix:** WebGL detection now uses a **throwaway canvas** (`document.createElement('canvas')`) — the real canvas is untouched until Three.js claims it.

### ❌ overflow-x: hidden Scrollbar Bug
**Problem:** Changed `.section { overflow: hidden }` to `overflow-x: hidden` to stop background numbers being clipped vertically. Browser spec: setting `overflow-x: hidden` implicitly sets `overflow-y: auto` → scrollbar appeared on every section.  
**Fix:** Reverted to `overflow: hidden`. Instead moved the background numbers up via `translateY(-51%)` so they sit higher and the bottom isn't clipped.

### ❌ Root-Relative Paths Breaking GitHub Pages
**Problem:** Changed card links to `/work1.html` (absolute). On GitHub Pages, the site lives at `github.io/ai-cowork/` — so `/work1.html` resolves to `github.io/work1.html` (404).  
**Fix:** All links use relative paths (`work1.html`, `index.html`). Works on both localhost and GitHub Pages.

### ❌ html.to.design Figma Import
**Problem:** Tried to import the site into Figma using html.to.design Chrome extension. Appeared to work but nothing showed up in Figma.  
**Root cause:** The page uses WebGL canvas (Three.js). html.to.design cannot capture GPU-rendered canvas content — the 3D elements are blank/missing in the export.  
**Workaround:** Chrome DevTools → `Ctrl+Shift+P` → *Capture full size screenshot* → import PNG into Figma.

---

## 5. What Worked Well

| Feature | Why It Worked |
|---------|--------------|
| Single Three.js scene | One renderer, mode switching — no sync issues between hero and service 3D |
| GSAP `overwrite: true` | Eliminated tween conflicts on fast scroll entirely |
| `gsap.set()` for initial states | More reliable than inline `style.transform` — GSAP owns the transform from creation |
| Receipt as accumulated state | `Set` + append pattern is simple, never breaks, naturally grows upward from fixed bottom |
| `preserveDrawingBuffer: true` | Required for screenshot tools to capture the WebGL canvas without black frames |
| Fallback geometries | TorusKnotGeometry as GLB fallback means the site always has 3D content even without model files |
| npx serve over Python | Handles extension-less URLs, cleaner routing, no manual restart needed |

---

## 6. Typography Scale (Desktop)

| Element | Size |
|---------|------|
| Hero headline | `9.5vw` |
| Intro headline | `7.8vw` |
| Background section numbers | `44vw` |
| Background intro glyph ∞ | `38vw` |
| Lateral text AI COWORK | `8.8vw` |
| Card #01 number | `5.2vw` |
| Card title | `1.6vw` |
| Receipt / UI labels | `0.72vw` |
| Stats / mono data | `0.6vw` |

---

## 7. Latest Work1 Background Update

`work1.html` originally used a simple Three.js particle field as its fixed background. During the latest iteration, that background was replaced with a **LineWaves-style shader** based on the React Bits `LineWaves` component.

Important implementation note: the project is still a pure static HTML/CSS/JS site. Even though the reference component uses React + `ogl`, the effect was ported into the existing Three.js canvas instead of adding a React build step. This keeps local preview and GitHub Pages deployment simple.

### Current `work1.html` Background
- Uses the existing fixed `<canvas id="work-canvas">`
- Renders a full-screen shader plane with an orthographic camera
- Implements the LineWaves fragment shader logic directly in `work1.html`
- Uses the requested settings:
  - `speed: 0.3`
  - `innerLineCount: 32`
  - `outerLineCount: 36`
  - `warpIntensity: 1.0`
  - `rotation: -45`
  - `edgeFadeWidth: 0.0`
  - `colorCycleSpeed: 1.0`
  - `brightness: 0.2`
  - `color1/color2/color3: #ffffff`
  - `enableMouseInteraction: true`
  - `mouseInfluence: 2.0`

### What Changed From Previous Experiments
- `ColorBends` was tested first as a soft green-blue animated background.
- `DotField` was then layered over it as a second canvas.
- The final current version removes the extra DotField canvas and uses **only LineWaves** on `#work-canvas`.
- Mouse and touch movement are smoothed into shader uniforms so the lines react gently to cursor position.
- The WebGL support check still uses a throwaway canvas, preserving the earlier fix that prevents context conflicts.

---

## 8. Deployment

```
Local dev:    npx serve . --listen 3000   (from C:\Users\angel\ai-cowork)
GitHub Pages: https://angel19190828-commits.github.io/ai-cowork/

Update workflow:
  git add .
  git commit -m "description"
  git push
```

GitHub Pages auto-deploys ~30s after push. No build step. Pure static files.

---

## 9. File Structure

```
ai-cowork/
├── index.html        ← Main site (hero + intro + 3 product cards)
├── work1.html        ← Project detail page (AI Workspace Infrastructure)
├── .gitignore        ← Excludes portfolio/ folder and system files
└── .claude/
    └── launch.json   ← npx serve config for local preview
```

`portfolio/` is a separate personal project living in the same parent directory — excluded from this repo intentionally.

---

*Document generated from live build session · AI COWORK ® STUDIO · 2026*
