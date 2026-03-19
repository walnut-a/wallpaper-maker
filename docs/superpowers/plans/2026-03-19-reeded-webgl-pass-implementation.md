# Reeded WebGL Pass Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a fullscreen WebGL pass for `reeded-column` so the long-ribbed glass style reads like a real medium field while preview and `PNG` export stay aligned.

**Architecture:** Keep the existing Canvas 2D renderer as the main pipeline. Only when `state.shapeMode === "reeded-column"`, render the background into an intermediate 2D canvas, run a small WebGL fragment pass on that bitmap, draw the processed result back into the target 2D canvas, then continue with the existing accent and texture layers. If WebGL is unavailable or initialization fails, fall back to the current 2D reeded-field implementation.

**Tech Stack:** HTML, CSS, vanilla JavaScript, Canvas 2D API, WebGL 1.0, hidden canvas, local static server, Playwright browser checks

---

## File Structure

- Modify: `index.html`
  - Add a tiny WebGL renderer dedicated to `reeded-column`
  - Route `renderArtwork()` and `drawShapeMode()` so only the reeded mode uses the new pass
  - Keep export working by reusing the same pass for preview and full-size rendering
- Modify: `README.md`
  - Briefly note that long-ribbed glass now uses a shader-style fullscreen pass with graceful fallback
- Reference: `docs/superpowers/specs/2026-03-19-reeded-webgl-pass-design.md`
  - Use this as the contract for scope, fallback, and preview/export consistency

## Notes Before Execution

- Do not change the public shape key: keep `reeded-column`
- Do not move the project off the single-file architecture
- Do not add external dependencies
- Serve locally before running checks:

```bash
cd /Users/zhaolixing/Documents/GitHub/wallpaper-maker
python3 -m http.server 8123
```

## Chunk 1: WebGL Hook and Fallback

### Task 1: Add a dedicated reeded-glass renderer with safe fallback

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing support check**

Run:

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("http://127.0.0.1:8123", wait_until="networkidle")
    value = page.evaluate("typeof window.__wallpaperDebug?.renderReededGlassPass")
    print(value)
    browser.close()
```

Expected initial result: `undefined`

- [ ] **Step 2: Add hidden WebGL canvas creation**

Create helpers in `index.html`:

- `getReededShaderRenderer()`
- `createReededShaderRenderer()`

The renderer should cache:

- hidden canvas
- `gl` context
- compiled shaders
- fullscreen quad buffer
- texture handle
- uniform locations

- [ ] **Step 3: Add graceful fallback state**

Store an internal renderer status such as:

```js
{
  ready: true,
  failed: false,
  reason: "",
}
```

If setup fails at any stage, mark it failed and let later calls fall back to the 2D path.

- [ ] **Step 4: Expose a small debug hook**

Add:

```js
window.__wallpaperDebug.renderReededGlassPass = renderReededGlassPass;
```

Expected: the browser check now prints `function`

## Chunk 2: Fullscreen Reeded Shader Pass

### Task 2: Build the bitmap-in, bitmap-out shader path

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing visual check**

Run:

```python
from pathlib import Path
from playwright.sync_api import sync_playwright

out = Path("playwright-output")
out.mkdir(exist_ok=True)

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={"width": 1080, "height": 1920})
    page.goto("http://127.0.0.1:8123", wait_until="networkidle")
    page.select_option("#shapeModeSelect", "reeded-column")
    page.screenshot(path=str(out / "reeded-webgl-before.png"))
    browser.close()
```

Expected initial result: image still reads like the current 2D approximation.

- [ ] **Step 2: Create `renderReededGlassPass()`**

Function contract:

```js
function renderReededGlassPass(sourceCanvas, targetCtx, width, height, palette, currentState, composition)
```

Responsibilities:

- upload `sourceCanvas` as a texture
- run a fullscreen fragment shader
- draw the shader output into `targetCtx`
- return `true` on success, `false` when fallback should be used

- [ ] **Step 3: Implement the fragment shader**

Use uniforms for:

- resolution
- source texture
- `ridgeDensity`
- `ridgeBreath`
- `refractionField`
- `focusField`
- `fieldBlend`
- focus zone data

The shader should do:

- longitudinal periodic displacement
- slight frequency breathing
- one or two stronger focus bands
- brightness and contrast shaping that reads as glass thickness

- [ ] **Step 4: Keep the output soft**

Avoid hard high-frequency stripes. Use:

- mixed sine modulation
- softened highlight weighting
- low-amplitude distortion that still survives on a phone screen

- [ ] **Step 5: Verify whole-frame coverage**

Take the screenshot again and confirm:

- the glass treatment reaches the frame edges
- no centered closed object silhouette remains
- local enhancement still exists without becoming a separate object

## Chunk 3: Pipeline Integration

### Task 3: Route only `reeded-column` through the new pass

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing pipeline check**

Run:

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("http://127.0.0.1:8123", wait_until="networkidle")
    values = page.evaluate("""
      () => ({
        mode: document.querySelector('#shapeModeSelect').value,
        hasDebug: typeof window.__wallpaperDebug?.renderReededGlassPass === 'function'
      })
    """)
    print(values)
    browser.close()
```

Expected initial result before routing change: debug function exists, but the main render path may still not be calling it.

- [ ] **Step 2: Add a dedicated background-to-pass path**

Refactor the reeded branch so it does this:

1. render background into an intermediate 2D canvas
2. run `renderReededGlassPass(...)`
3. if that returns `false`, call the current 2D `drawReededGlassGeometry(...)`

- [ ] **Step 3: Keep accent and texture layers after the pass**

Do not move:

- `drawAccentMode(...)`
- `drawTextureMode(...)`

They should still run after the shape phase so existing style layering survives.

- [ ] **Step 4: Ensure export uses the same path**

Verify that `renderArtwork(exportContext, state.width, state.height, state)` reaches the WebGL pass for `reeded-column`, not a preview-only shortcut.

## Chunk 4: Verification and Docs

### Task 4: Prove preview, export, and fallback behavior

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Verify parameter response**

Run a browser script that changes:

- `ridgeDensity`
- `refractionField`
- `focusField`

Expected: the canvas signature changes after each adjustment.

- [ ] **Step 2: Verify export still downloads**

Run a Playwright script that clicks `导出 PNG` in `reeded-column` mode.

Expected: a `.png` file download starts successfully.

- [ ] **Step 3: Verify fallback path**

Temporarily simulate failure by short-circuiting renderer creation in dev code or by monkey-patching the debug object during the test.

Expected:

- page still renders
- no uncaught errors
- `reeded-column` falls back to the current 2D path

- [ ] **Step 4: Update README**

Add a short note that the long-ribbed glass mode now uses a fullscreen shader-style pass with graceful fallback, while the rest of the generator stays on Canvas 2D.

- [ ] **Step 5: Final verification**

Run:

```bash
cd /Users/zhaolixing/Documents/GitHub/wallpaper-maker
git diff -- index.html README.md docs/superpowers/specs/2026-03-19-reeded-webgl-pass-design.md docs/superpowers/plans/2026-03-19-reeded-webgl-pass-implementation.md
```

Then confirm in browser:

- `reeded-column` looks more like a full-frame medium
- preview still updates live
- export still works
- console stays clean

- [ ] **Step 6: Commit**

```bash
git add index.html README.md docs/superpowers/specs/2026-03-19-reeded-webgl-pass-design.md docs/superpowers/plans/2026-03-19-reeded-webgl-pass-implementation.md
git commit -m "feat: add webgl pass for reeded glass mode"
```
