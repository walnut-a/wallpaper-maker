# Space Cinematic Style Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a new original `太空电影感` style by introducing a warp-speed background, a tethered orbital-pod shape module, a star-spark texture module, and a matching recommended composition.

**Architecture:** Keep the current modular system intact. Extend `backgroundModes`, `shapeModes`, `textureModes`, and `compositionPresets` with a new cinematic trio instead of adding a standalone mode. Reuse the existing Canvas 2D rendering pipeline, adding one background renderer, one shape renderer, one texture renderer, and one palette tuned for deep-space contrast.

**Tech Stack:** HTML, CSS, vanilla JavaScript, Canvas 2D API, local static single-file app, Playwright browser checks

---

## File Structure

- Modify: `index.html`
  - Add new palette, background mode, shape mode, texture mode, preset mapping, random-mix compatibility, and rendering functions.
  - Keep the existing single-file structure and module-driven UI.
- Modify: `README.md`
  - Document the new cinematic style, modules, and palette.
- Reference: `docs/superpowers/specs/2026-03-28-space-cinematic-style-design.md`
  - Use this as the contract for naming, structure, and visual intent.

## Notes Before Execution

- Keep all new work inside the current module system.
- Do not import external images or other assets.
- Do not attempt to reproduce the exact reference composition.
- Use the existing `file://` Playwright checks as the main verification path.

## Chunk 1: Register New Modules and Preset

### Task 1: Add the new palette, background, shape, texture, and recommended composition

**Files:**
- Modify: `index.html`
- Modify: `README.md`

- [ ] **Step 1: Write the failing browser check**

Run:

```python
from pathlib import Path
from playwright.sync_api import sync_playwright

url = Path("/Users/zhaolixing/Documents/GitHub/wallpaper-maker/index.html").resolve().as_uri()

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto(url, wait_until="load")
    page.wait_for_timeout(500)
    data = page.evaluate("""
      () => ({
        presetLabels: [...document.querySelectorAll('#presetButtons button')].map(node => node.textContent.trim()),
        backgrounds: [...document.querySelectorAll('#backgroundModeSelect option')].map(node => node.textContent.trim()),
        shapes: [...document.querySelectorAll('#shapeModeSelect option')].map(node => node.textContent.trim()),
        textures: [...document.querySelectorAll('#textureModeSelect option')].map(node => node.textContent.trim())
      })
    """)
    print(data)
    browser.close()
```

Expected initial result:

- no `太空电影感`
- no `跃迁星幕`
- no `轨舱系绳`
- no `星尘碎光`

- [ ] **Step 2: Add a new palette**

Add a palette tuned for this style, for example:

```js
starlight: {
  label: "跃迁霓夜",
  backgroundA: "#140f2d",
  backgroundB: "#040814",
  surface: "#f0b480",
  ink: "#02040d",
  accent: "#ff7a3c",
  accentSoft: "#6d7dff",
}
```

- [ ] **Step 3: Add the background module**

Add a `backgroundModes["warp-canopy"]` entry:

- label: `跃迁星幕`
- defaults:
  - `warpAngle`
  - `warpDensity`
  - `flareIntensity`

Suggested defaults:

```js
defaults: {
  warpAngle: 0.66,
  warpDensity: 0.62,
  flareIntensity: 0.64,
}
```

- [ ] **Step 4: Add the shape module**

Add a `shapeModes["orbital-tether"]` entry:

- label: `轨舱系绳`
- defaults:
  - `podScale`
  - `tetherTension`
  - `driftOffset`

Suggested defaults:

```js
defaults: {
  podScale: 0.58,
  tetherTension: 0.72,
  driftOffset: 0.46,
}
```

- [ ] **Step 5: Add the texture module**

Add a `textureModes["stardust-sparks"]` entry:

- label: `星尘碎光`
- defaults:
  - `sparkDensity`
  - `sparkSharpness`

Suggested defaults:

```js
defaults: {
  sparkDensity: 0.42,
  sparkSharpness: 0.68,
}
```

- [ ] **Step 6: Add the recommended composition**

Add:

```js
"space-cinematic": {
  label: "太空电影感",
  backgroundMode: "warp-canopy",
  shapeMode: "orbital-tether",
  textureMode: "stardust-sparks",
  defaults: {
    palette: "starlight",
    density: 0.62,
    complexity: 0.66,
    advanced: {
      softness: 0.42,
      glowRadius: 0.74,
      tonalSplit: 0.76,
      grainFine: 0.12,
      warpAngle: 0.66,
      warpDensity: 0.62,
      flareIntensity: 0.64,
      podScale: 0.58,
      tetherTension: 0.72,
      driftOffset: 0.46,
      sparkDensity: 0.42,
      sparkSharpness: 0.68,
    },
  },
}
```

- [ ] **Step 7: Update README wording**

Add the new style and module names to:

- feature overview
- style description
- parameter description

- [ ] **Step 8: Re-run the browser check**

Expected:

- all four new names appear in the UI

## Chunk 2: Build the Warp-Speed Background

### Task 2: Render a directional deep-space warp background

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing visual check**

Run:

```python
from pathlib import Path
from playwright.sync_api import sync_playwright

url = Path("/Users/zhaolixing/Documents/GitHub/wallpaper-maker/index.html").resolve().as_uri()
out = Path("/Users/zhaolixing/Documents/GitHub/wallpaper-maker/playwright-output")
out.mkdir(exist_ok=True)

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={"width": 1280, "height": 900})
    page.goto(url, wait_until="load")
    page.wait_for_timeout(700)
    page.select_option("#backgroundModeSelect", "warp-canopy")
    page.wait_for_timeout(700)
    page.screenshot(path=str(out / "warp-canopy-before.png"))
    browser.close()
```

Expected initial result: the background still looks like an existing mode or remains empty because the new branch is not implemented.

- [ ] **Step 2: Add the background renderer**

Create a renderer such as:

- `drawWarpCanopyBackground()`

It should include:

- deep-space dark base
- long diagonal speed streaks
- 2 to 4 large warm flare clouds
- sparse cool neon accents

- [ ] **Step 3: Route the new background mode**

Update `drawBackground()` or the relevant branch so `warp-canopy` uses the new renderer.

- [ ] **Step 4: Verify parameter response**

Change:

- `warpAngle`
- `warpDensity`
- `flareIntensity`

Expected: each change visibly alters the canvas signature.

## Chunk 3: Build the Orbital-Tether Shape

### Task 3: Render the cinematic pod, distant body, and tether line

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing visual check**

Run:

```python
from pathlib import Path
from playwright.sync_api import sync_playwright

url = Path("/Users/zhaolixing/Documents/GitHub/wallpaper-maker/index.html").resolve().as_uri()
out = Path("/Users/zhaolixing/Documents/GitHub/wallpaper-maker/playwright-output")
out.mkdir(exist_ok=True)

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={"width": 1280, "height": 900})
    page.goto(url, wait_until="load")
    page.wait_for_timeout(700)
    page.select_option("#shapeModeSelect", "orbital-tether")
    page.wait_for_timeout(700)
    page.screenshot(path=str(out / "orbital-tether-before.png"))
    browser.close()
```

Expected initial result: no dedicated cinematic subject appears.

- [ ] **Step 2: Add the shape renderer**

Create a renderer such as:

- `drawOrbitalTetherGeometry()`

It should draw:

- a near-corner pod body
- a small distant drifting body
- a thin connecting tether

Visual rules:

- pod stays abstract and geometric
- distant body is tiny and bright
- tether is thin and tense, not cartoonish

- [ ] **Step 3: Route the new shape mode**

Update `drawShapeMode()` so `orbital-tether` uses the new renderer.

- [ ] **Step 4: Add accent support if needed**

If the shape needs extra highlight shaping, add a focused branch in `drawAccentLayer()`.

- [ ] **Step 5: Verify parameter response**

Change:

- `podScale`
- `tetherTension`
- `driftOffset`

Expected: each change visibly alters the canvas signature and subject layout.

## Chunk 4: Build the Stardust Texture and Compatibility

### Task 4: Render sparse directional spark texture and include it in mixing

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing visual check**

Run:

```python
from pathlib import Path
from playwright.sync_api import sync_playwright

url = Path("/Users/zhaolixing/Documents/GitHub/wallpaper-maker/index.html").resolve().as_uri()
out = Path("/Users/zhaolixing/Documents/GitHub/wallpaper-maker/playwright-output")
out.mkdir(exist_ok=True)

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={"width": 1280, "height": 900})
    page.goto(url, wait_until="load")
    page.wait_for_timeout(700)
    page.select_option("#textureModeSelect", "stardust-sparks")
    page.wait_for_timeout(700)
    page.screenshot(path=str(out / "stardust-sparks-before.png"))
    browser.close()
```

Expected initial result: no dedicated star-spark texture appears.

- [ ] **Step 2: Add the texture renderer**

Extend `drawTextureLayer()` with a `stardust-sparks` branch.

It should draw:

- sparse bright points
- short directional streaks
- very small amount of color-separated sparks

- [ ] **Step 3: Verify parameter response**

Change:

- `sparkDensity`
- `sparkSharpness`

Expected: canvas signature changes while the texture remains sparse.

- [ ] **Step 4: Include the new modules in random mix**

Update the random-mix compatibility logic so:

- `warp-canopy`
- `orbital-tether`
- `stardust-sparks`

can be selected, with reasonable pairings.

## Chunk 5: Final Integration and Verification

### Task 5: Prove the new cinematic style works end to end

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Verify the full recommended composition**

Run a Playwright script that applies `space-cinematic`, waits for render, and captures:

- canvas signature
- bottom summary label
- screenshot

Expected:

- summary reflects the new modules
- screenshot reads as a cinematic space scene

- [ ] **Step 2: Verify export**

Trigger `导出 PNG` with the new preset active.

Expected: a `.png` download starts successfully.

- [ ] **Step 3: Verify mobile layout still holds**

Open at a narrow width and confirm preview remains above the controls.

- [ ] **Step 4: Verify console stays clean**

Capture `pageerror` and `console` messages.

Expected:

- no page errors
- no new JavaScript exceptions

- [ ] **Step 5: Review diff**

Run:

```bash
cd /Users/zhaolixing/Documents/GitHub/wallpaper-maker
git diff -- index.html README.md docs/superpowers/specs/2026-03-28-space-cinematic-style-design.md docs/superpowers/plans/2026-03-28-space-cinematic-style-implementation.md
```

- [ ] **Step 6: Commit**

```bash
git add index.html README.md docs/superpowers/specs/2026-03-28-space-cinematic-style-design.md docs/superpowers/plans/2026-03-28-space-cinematic-style-implementation.md
git commit -m "feat: add space cinematic style"
```
