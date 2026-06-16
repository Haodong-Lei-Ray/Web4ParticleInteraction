# Web4ParticleInteraction

[中文](README.md)

A high-performance, zero-dependency **ASCII fluid interactive background library**. When the user moves the mouse, fluid is injected to produce a visible "ink" flow effect, with support for custom shapes, parameter tuning, content avoidance, and easy integration into any webpage.

## ✨ Features

- **Fluid simulation**: Based on the Jos Stam stable-fluids algorithm, running directly at glyph-grid resolution to keep shapes crisp
- **Interactive cursor**: A brush follows the cursor (default: a cat-head shape), advecting into a flowing trail while moving, and settling back into a gently floating shape while still
- **Content avoidance**: Automatically detects page text, buttons, images, and other elements, and skips rendering the effect over them to keep content readable
- **Responsive**: Automatically adapts to window resizing, scrolling, and font loading; supports touch devices
- **Accessible**: Respects the `prefers-reduced-motion` system setting, gracefully degrading to a static texture
- **Zero dependencies**: Pure JavaScript + Canvas 2D, no external libraries
- **High performance**: Simulates at 45Hz, renders at 60fps; measured ~60FPS sustained at 1280×820 resolution
- **Customizable**: Swap the shape (edit the bitmap), tune parameters (fluid feel, color, range), or change colors (CSS variables)

## 🚀 Quick Start

### 1. Copy the script

Copy `js/background.js` into your project.

### 2. Edit your HTML

Add two containers at the very start of `<body>`:

```html
<body>
  <div id="bg-gradient" aria-hidden="true"></div>
  <canvas id="bg-canvas" aria-hidden="true"></canvas>
  <!-- your page content... -->
</body>
```

### 3. Add CSS

Add the following to your stylesheet (or a `<style>` block in `<head>`):

```css
:root {
  /* gradient glow colors */
  --bg-grad-1: #c7d0ff;
  --bg-grad-2: #d9ccf2;
  --bg-grad-3: #cfe0f5;
  /* background base color */
  --bg-grad-base: #e3e7ea;
  /* glyph color (RGB triplet) */
  --bg-glyph: 70, 84, 150;
  /* optional: monospace font */
  --font-mono: 'Courier New', monospace;
}

#bg-gradient {
  position: fixed;
  inset: 0;
  z-index: -2;
  pointer-events: none;
  background:
    radial-gradient(60% 60% at 78% 18%, var(--bg-grad-1) 0%, transparent 62%),
    radial-gradient(55% 55% at 18% 30%, var(--bg-grad-2) 0%, transparent 60%),
    radial-gradient(70% 70% at 50% 92%, var(--bg-grad-3) 0%, transparent 64%),
    var(--bg-grad-base);
  opacity: 0.55;
}

#bg-canvas {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
  pointer-events: none;
}
```

**Important**: both background layers must use `position: fixed`, a negative `z-index`, and `pointer-events: none`, so they always stay behind your content and never block user interaction.

### 4. Include the script

Add this just before `</body>`:

```html
<script src="path/to/js/background.js"></script>
```

Done! Refresh the page and move your mouse to see the ASCII fluid effect.

## ⚙️ Tuning Parameters

Edit the constants at the top of `js/background.js` to tune the effect.

### Appearance

| Parameter | Default | Description |
|------|--------|------|
| `CELL_PX` | `8` | Size of the character grid (px); smaller = finer detail |
| `RAMP` | `[' ','·','_','-','>','○']` | Brightness ramp: characters from dark to bright |
| `CONTRAST` | `1.9` | Density-to-brightness contrast; higher = more pronounced |
| `GAMMA` | `0.6` | Brightness curve; smaller = darker areas show up more easily |
| `THRESHOLD` | `0.12` | Cells below this value draw nothing (keeps things clean) |

### Opacity

| Parameter | Default | Description |
|------|--------|------|
| `ALPHA_MAX` | `0.45` | Overall opacity cap; smaller = more subtle |
| `ALPHA_BASE` | `0.05` | Opacity floor for the dimmest glyphs |
| `ALPHA_GAIN` | `0.40` | How much brighter glyphs get near the ink |

### Cursor Brush Shape

Edit the `BRUSH_SHAPE` array (`#` = ink, space = empty) to swap in any shape:

```javascript
const BRUSH_SHAPE = [
  '##   ##',     // two pointed ears
  '### ###',
  '#######',
  '# ### #',     // eyes
  ' ##### ',     // mouth
];
// or try a heart:
// const BRUSH_SHAPE = [
//   ' ##### ',
//   '#######',
//   '#######',
//   ' ##### ',
//   '  ###  ',
// ];
```

| Parameter | Default | Description |
|------|--------|------|
| `BRUSH_SHAPE` | cat-head bitmap | Shape of the cursor brush |
| `BRUSH_SCALE` | `2` | How many character cells each bitmap pixel occupies; bigger = larger shape |
| `STAMP_VALUE` | `0.9` | Density strength of the brush |

### Fluid Feel

These parameters control the ink's flow, decay, and floating motion:

| Parameter | Default | Description |
|------|--------|------|
| `flow` | `0.7` | Advection strength: cells the ink moves per unit velocity; higher = spreads out more easily |
| `iter` | `4` | Pressure-solve iterations; higher = smoother swirls but more expensive |
| `densDecay` | `0.85` | Trail fade rate; smaller = shorter trail |
| `velDecay` | `0.55` | Velocity damping; smaller = stops faster. Also determines the steady-state amplitude of `idleAmp` (≈`idleAmp/(1-velDecay)`); increasing it makes the floating motion progressively more exaggerated, even divergent |
| `force` | `0.6` | Scale from cursor speed to injected velocity; higher = more dramatic flow |
| `velStop` | `0` | Velocities below this snap to zero; keep at `0` and let `velDecay` damp naturally, to avoid conflicting with `idleAmp` |
| `idleAmp` | `0.07` | A tiny continuous "floating force" injected while at rest; keeps the cat from freezing solid, giving it a subtle breathing motion |
| `idleFreq` | `0.0015` | Oscillation speed of the floating force; higher = faster, more noticeable floating |

### Ambient Layer

| Parameter | Default | Description |
|------|--------|------|
| `AMBIENT_AMP` | `0.08` | Amplitude of the faint resting texture (multi-frequency sine noise); set to `0` for a fully blank resting state |

### Content Avoidance / Range Control

| Parameter | Default | Description |
|------|--------|------|
| `SAFE_SELECTOR` | multiple selectors | The effect is not drawn above these elements (headings, paragraphs, buttons, images, …) |
| `SAFE_PAD` | `6` | Hard padding expanding each content box (px) |
| `SAFE_FEATHER` | `16` | Width of the soft feather band at the mask edge (px) |
| `SAFE_REBUILD_MS` | `140` | How often the mask is recomputed (ms) |
| `CUTOFF_SELECTOR` | `'#about'` | The effect only renders down to the bottom of this element; use a non-existent selector to cover the whole page |
| `CUTOFF_FEATHER` | `70` | Width of the fade-out band above the cutoff line (px) |

**Example**: keep the effect off certain elements by editing `SAFE_SELECTOR`:

```javascript
const SAFE_SELECTOR = 'h1, h2, .logo, .nav-bar, .floating-widget';
```

**Example**: make the effect cover the entire viewport:

```javascript
const CUTOFF_SELECTOR = '#__nonexistent__';
```

## 🎨 Color Themes

All colors are driven by CSS variables and support automatic light/dark theme switching:

```css
/* Light theme */
:root {
  --bg-grad-1: #c7d0ff;     /* purple-blue */
  --bg-grad-2: #d9ccf2;     /* light purple */
  --bg-grad-3: #cfe0f5;     /* blue-white */
  --bg-grad-base: #e3e7ea;  /* base color */
  --bg-glyph: 70, 84, 150;  /* glyph color RGB */
}

/* Dark theme */
@media (prefers-color-scheme: dark) {
  :root {
    --bg-grad-1: #2b343b;
    --bg-grad-2: #313442;
    --bg-grad-3: #283640;
    --bg-grad-base: #1a1f24;
    --bg-glyph: 141, 163, 153;
  }
}
```

## 📐 Technical Details

### Algorithm

Based on Jos Stam's classic stable fluid solver (Stable Fluids), using semi-Lagrangian advection plus Gauss-Seidel pressure projection:

1. **Velocity injection**: while the mouse moves, velocity is injected within the brush footprint, scaled by `force`
2. **Pressure projection**: ensures the velocity field is divergence-free (so the fluid doesn't expand)
3. **Advection**: traces backward along the velocity field to compute the new density (ink) position
4. **Decay**: each step applies exponential decay to density and velocity via `densDecay` / `velDecay`
5. **Rendering**: maps density to a RAMP character and computes the corresponding opacity

### Performance Optimizations

- Runs directly at glyph-grid resolution (1280×820@8px ≈ 160×103 cells), avoiding upsampling blur
- Simulation fixed at 45Hz, rendering at 60fps (sim and render are decoupled)
- Mask detection throttled to 140ms, only recomputed when necessary (scroll/resize/load/font-ready)
- Single-pass rendering, no multiple buffering

### Browser Compatibility

Requires support for:
- Canvas 2D Context
- `requestAnimationFrame`
- CSS custom properties (CSS variables)
- `matchMedia` for responsive detection

**Minimum requirement**: modern browsers (Chrome 49+, Firefox 31+, Safari 9.1+, Edge 15+)

## 📝 Integration Tips

### 1. Tune the safe zone

Adjust `SAFE_SELECTOR` to match your page's structure:

```javascript
const SAFE_SELECTOR = 'h1,h2,h3,h4,h5,h6,p,li,a,button,img,.header,.nav';
```

### 2. Tune the effect range

Change `CUTOFF_SELECTOR` to decide where the effect stops:

```javascript
const CUTOFF_SELECTOR = '.footer'; // only render down to before the footer
// or
const CUTOFF_SELECTOR = '#__none__'; // cover the whole page
```

### 3. Match your colors

Change the CSS variables to match your site's palette, or inherit from existing CSS variables:

```css
:root {
  --bg-grad-base: var(--primary-bg);
  --bg-glyph: var(--primary-text-rgb);
}
```

### 4. Disable/enable

If a specific page doesn't need the effect, check a condition in `<head>`:

```html
<script>
  if (location.pathname === '/admin') {
    // Don't load the background effect
    document.addEventListener('DOMContentLoaded', () => {
      const canvas = document.getElementById('bg-canvas');
      if (canvas) canvas.parentNode.removeChild(canvas);
    });
  }
</script>
<script src="js/background.js"></script>
```

## 📦 File Structure

```
Web4ParticleInteraction/
├── README.md                   # this file
├── background-effect.md        # detailed parameter documentation
└── js/
    └── background.js           # core library (495 lines, pure JS)
```

## 🔧 FAQ

**Q: The effect is covering my text**
A: Adjust `SAFE_SELECTOR`, or increase `SAFE_PAD` / `SAFE_FEATHER`.

**Q: I want to swap in a different shape**
A: Edit the `BRUSH_SHAPE` array. Each line is a string; `#` means ink, space means empty.

**Q: The flow is too slow/too fast**
A: Increase/decrease `velDecay` (smaller = stops faster) and `force` (bigger = more responsive).

**Q: The floating is too noticeable, making the cat look jittery**
A: Decrease `idleAmp`, or adjust `velDecay`.

**Q: I want the effect to be more subtle**
A: Decrease `ALPHA_MAX` / `ALPHA_GAIN`, or increase `THRESHOLD`.

**Q: The effect stutters on low-end devices**
A: Increase `CELL_PX` (coarser grid, but better performance), or lower `iter` (less swirl smoothness).

## 📄 License

MIT

## 👤 Author

Original design concept inspired by the OpenAI Codex official demo page; this library is implemented from scratch, with added extensions for fluid floating motion, content avoidance, and parameter tuning.
