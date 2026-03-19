# Modular Wallpaper Composition Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Refactor the current single-file wallpaper generator so recommended presets remain available, while background, shape, and texture become independently selectable modules that can be freely combined.

**Architecture:** Keep the shipped app inside `index.html`, but split the current preset-driven logic into three registries: `compositionPresets`, `backgroundModes`, `shapeModes`, and `textureModes`. Reuse the existing drawing functions by mapping them into module render stages first, then rebuild state syncing, advanced controls, and randomization around the new module model.

**Tech Stack:** HTML, CSS, vanilla JavaScript, Canvas 2D API, browser download APIs, local static server, headless Chrome or Playwright MCP for verification

---

## File Structure

- Modify: `index.html`
  - Add composition module registries, extend the state model, add the free-combination controls, refactor rendering to run by module, update randomization, reset behavior, and browser automation helpers.
- Modify: `README.md`
  - Update the feature description after the modular composition UI ships so the public repo matches the actual product.
- Reference: `docs/superpowers/specs/2026-03-19-modular-wallpaper-composition-design.md`
  - Use this as the behavioral contract for presets, modules, randomization, and custom-combination status.
- Reference: `PRINCIPLES.md`
  - Keep the implementation single-file, static, and result-first.

## Notes Before Execution

- There is still no package-based test runner in this repo. Treat browser verification as the test harness.
- Serve the page locally before each browser test:

```bash
cd /Users/zhaolixing/Documents/GitHub/wallpaper-maker
python3 -m http.server 8123
```

- Use browser checks through `window.__app` for behavior verification. If the needed helper does not exist yet, the first failing check should prove that.
- Keep commits small. One task or one tightly related pair of tasks per commit is the target.

## Chunk 1: State Model And UI Entry Points

### Task 1: Expand app state from a single preset into preset + module composition

**Files:**
- Modify: `index.html`
- Reference: `docs/superpowers/specs/2026-03-19-modular-wallpaper-composition-design.md`

- [ ] **Step 1: Write the failing browser check**

Start the local server, open `http://127.0.0.1:8123/`, and run this in the browser automation console:

```js
(() => {
  const state = window.__app.getState();
  return {
    preset: state.preset,
    backgroundMode: state.backgroundMode,
    shapeMode: state.shapeMode,
    textureMode: state.textureMode,
  };
})()
```

Expected: FAIL because `backgroundMode`, `shapeMode`, and `textureMode` are missing.

- [ ] **Step 2: Add the module registries and state shape**

In `index.html`, introduce:

```js
const compositionPresets = {
  "minimal-grid": {
    label: "极简秩序感",
    backgroundMode: "soft-atmosphere",
    shapeMode: "minimal-form",
    textureMode: "light-grid",
    defaults: { /* palette, density, complexity, advanced */ },
  },
};

const backgroundModes = {
  "soft-atmosphere": { label: "柔雾渐变", defaults: { /* ... */ }, controls: [/* ... */] },
};

const shapeModes = {
  "minimal-form": { label: "极简大形体", defaults: { /* ... */ }, controls: [/* ... */] },
};

const textureModes = {
  "light-grid": { label: "轻网格", defaults: { /* ... */ }, controls: [/* ... */] },
};
```

Update `buildDefaultState()` so it returns:

```js
{
  preset: presetKey,
  backgroundMode: preset.backgroundMode,
  shapeMode: preset.shapeMode,
  textureMode: preset.textureMode,
  palette: preset.defaults.palette,
  density: preset.defaults.density,
  complexity: preset.defaults.complexity,
  seed: 314159,
  advanced: { ...preset.defaults.advanced },
}
```

- [ ] **Step 3: Run the browser check again**

Run the same browser code.

Expected: PASS with non-empty values for:
- `preset`
- `backgroundMode`
- `shapeMode`
- `textureMode`

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "refactor: expand wallpaper state to modular composition"
```

### Task 2: Add the free-combination controls under the preset area

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Open the page and run:

```js
(() => {
  const ids = ["backgroundModeSelect", "shapeModeSelect", "textureModeSelect"];
  return ids.map((id) => ({ id, exists: Boolean(document.getElementById(id)) }));
})()
```

Expected: FAIL because these controls do not exist yet.

- [ ] **Step 2: Add the UI and option builders**

Add a new control section below preset selection:

```html
<section class="panel-section" aria-labelledby="composition-heading">
  <h2 id="composition-heading" class="section-title">自由组合</h2>
  <div class="field">
    <label for="backgroundModeSelect">背景气氛</label>
    <select id="backgroundModeSelect"></select>
  </div>
  <div class="field">
    <label for="shapeModeSelect">主体图形</label>
    <select id="shapeModeSelect"></select>
  </div>
  <div class="field">
    <label for="textureModeSelect">纹理覆盖</label>
    <select id="textureModeSelect"></select>
  </div>
</section>
```

Extend the existing option-building logic so these three selects are filled from `backgroundModes`, `shapeModes`, and `textureModes`.

- [ ] **Step 3: Wire the controls into state**

On each select change:

```js
state.backgroundMode = event.currentTarget.value;
state.preset = "custom";
queueRender();
```

Do the equivalent for `shapeMode` and `textureMode`. Keep `syncControlValues()` responsible for showing the actual selected values.

- [ ] **Step 4: Verify the controls exist and update state**

Run:

```js
(() => {
  document.getElementById("textureModeSelect").value = "honeycomb";
  document.getElementById("textureModeSelect").dispatchEvent(new Event("change", { bubbles: true }));
  return window.__app.getState();
})()
```

Expected:
- `textureMode === "honeycomb"`
- `preset === "custom"`

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add free-composition controls"
```

## Chunk 2: Render Pipeline Refactor

### Task 3: Split rendering into background, shape, and texture module stages

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

With the page open, record one signature, then change only `textureMode`:

```js
(() => {
  const before = window.__app.getCanvasSignature();
  const select = document.getElementById("textureModeSelect");
  select.value = "honeycomb";
  select.dispatchEvent(new Event("change", { bubbles: true }));
  return new Promise((resolve) => requestAnimationFrame(() => {
    resolve({
      before,
      after: window.__app.getCanvasSignature(),
      state: window.__app.getState(),
    });
  }));
})()
```

Expected initial failure:
- changing `textureMode` does not reliably change the image because rendering still depends on `preset`

- [ ] **Step 2: Add module render dispatch**

Refactor the current preset-based rendering into:

```js
function drawBackgroundMode(ctx, width, height, palette, currentState, rng) {}
function drawShapeMode(ctx, width, height, palette, currentState, rng) {}
function drawAccentMode(ctx, width, height, palette, currentState, rng) {}
function drawTextureMode(ctx, width, height, palette, currentState, rng) {}
```

Map existing functions into module keys:

- `soft-atmosphere`
- `deep-cold`
- `warm-cool-split`
- `glass-diffuse`
- `minimal-form`
- `soft-blocks`
- `flow-ribbons`
- `reeded-column`
- `none`
- `light-grid`
- `honeycomb`
- `fine-grain`
- `prism-glass`

Then update `renderArtwork()` to use:

```js
drawBackgroundMode(...)
drawShapeMode(...)
drawAccentMode(...)
drawTextureMode(...)
```

- [ ] **Step 3: Verify that module-only changes affect rendering**

Run the same browser check after implementation.

Expected:
- `after !== before`
- state shows the changed module
- the preview remains non-blank

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "refactor: render wallpapers by composition modules"
```

### Task 4: Remap the five preset cards into recommended combinations

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Verify that each preset card sets a specific module combination:

```js
(() => {
  window.__app.switchPreset("reeded-glass");
  return window.__app.getState();
})()
```

Expected initial failure:
- the state switches `preset`, but module keys do not update to the recommended combination

- [ ] **Step 2: Implement preset-to-module mapping**

Define the recommended combinations exactly as in the spec:

- `minimal-grid` → `soft-atmosphere + minimal-form + light-grid`
- `bold-blocks` → `warm-cool-split + soft-blocks + fine-grain`
- `tech-texture` → `deep-cold + minimal-form + honeycomb`
- `organic-flow` → `soft-atmosphere + flow-ribbons + fine-grain`
- `reeded-glass` → `glass-diffuse + reeded-column + prism-glass`

Update `mergePresetDefaults()` and `switchPreset()` so they set all three module keys and rebuild advanced defaults.

- [ ] **Step 3: Verify the mapping**

Run one check per preset through `window.__app.switchPreset(...)`.

Expected:
- the preset label is correct
- the three module keys match the documented combination
- preview metadata still shows the preset name instead of `custom`

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: remap presets as recommended wallpaper combinations"
```

## Chunk 3: Controls, Randomization, And Product Behavior

### Task 5: Rebuild advanced controls from shared + module-specific parameter groups

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Check that advanced controls do not react to module changes yet:

```js
(() => {
  document.getElementById("textureModeSelect").value = "prism-glass";
  document.getElementById("textureModeSelect").dispatchEvent(new Event("change", { bubbles: true }));
  const text = document.getElementById("advancedContent").textContent;
  return text.includes("玻璃肌理") && !text.includes("蜂窝尺度");
})()
```

Expected initial failure:
- controls are still derived from the old preset-only grouping, so the wrong labels remain visible

- [ ] **Step 2: Implement advanced group composition**

Replace the current preset-only logic with merged groups:

```js
function getAdvancedGroups(currentState) {
  return [
    sharedAdvancedGroup,
    backgroundModes[currentState.backgroundMode].group,
    shapeModes[currentState.shapeMode].group,
    textureModes[currentState.textureMode].group,
  ].filter(Boolean);
}
```

Rules:
- keep one shared “组合控制” group
- only render controls for the active background, shape, and texture modules
- preserve current values when switching between compatible groups
- apply module defaults only when the user chooses a new preset or a new module

- [ ] **Step 3: Verify label and control switching**

Check at least these transitions:
- `honeycomb` shows `蜂窝尺度`
- `prism-glass` shows `玻璃肌理`
- `flow-ribbons` shows `漂移幅度` and `流向拉伸`
- inactive module controls disappear

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: build advanced controls from active wallpaper modules"
```

### Task 6: Split random actions into random parameters and random mix

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Add checks for the two expected behaviors:

```js
(() => {
  const before = window.__app.getState();
  return {
    hasRandomizeMix: Boolean(document.getElementById("randomizeMixButton")),
    before,
  };
})()
```

Expected initial failure:
- there is no separate random-mix action
- the current random action cannot preserve module selection versus mix modules on demand

- [ ] **Step 2: Add the second action and implement both behaviors**

UI:

```html
<button id="randomizeButton" type="button">随机参数</button>
<button id="randomizeMixButton" type="button">随机混搭</button>
```

Behavior:

- `randomizeCurrentCombination()`
  - keeps `backgroundMode`, `shapeMode`, `textureMode`
  - randomizes palette, density, complexity, seed, and visible advanced values

- `randomizeMix()`
  - chooses module combinations from a compatibility matrix
  - sets `preset = "custom"`
  - seeds a matching parameter set for the chosen modules

Use a compatibility table like:

```js
const mixCompatibility = {
  "reeded-column": {
    backgrounds: ["glass-diffuse", "soft-atmosphere"],
    textures: ["prism-glass", "fine-grain"],
  },
};
```

- [ ] **Step 3: Verify both random actions**

Checks:
- after `randomizeCurrentCombination()`, module keys are unchanged
- after `randomizeMix()`, at least one module key changes
- `randomizeMix()` always produces a combination present in the compatibility matrix

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add random parameter and random mix actions"
```

### Task 7: Finish custom-combination behavior, reset behavior, and public docs

**Files:**
- Modify: `index.html`
- Modify: `README.md`

- [ ] **Step 1: Write the failing browser check**

Expected product behavior:
- manual module changes set `preset` to `custom`
- clicking a preset card returns the state to a named recommendation
- reset restores the current combination defaults instead of arbitrarily jumping to another preset

Verify with:

```js
(() => {
  window.__app.switchPreset("tech-texture");
  document.getElementById("shapeModeSelect").value = "flow-ribbons";
  document.getElementById("shapeModeSelect").dispatchEvent(new Event("change", { bubbles: true }));
  return window.__app.getState();
})()
```

Expected initial failure:
- one or more of these transitions still follows the old preset-only behavior

- [ ] **Step 2: Implement final interaction rules**

In `index.html`:
- ensure manual module edits set `preset = "custom"`
- ensure clicking a preset card restores the recommended composition and defaults
- update preview metadata to show `自定义组合` when appropriate
- make reset restore the current active module defaults when `preset === "custom"`
- extend `window.__app` with any helper needed for browser verification:

```js
window.__app = {
  getState,
  switchPreset,
  randomizeCurrentCombination,
  randomizeMix,
  renderPreview,
  getCanvasSignature,
};
```

In `README.md`:
- describe presets as “推荐组合”
- mention free combination of background, shape, and texture
- mention both random actions if they ship in this pass

- [ ] **Step 3: Run the final regression pass**

Browser checks:
- all five preset cards still work
- the three module selects change the image
- advanced controls track active modules
- export still downloads a PNG
- mobile layout still keeps preview above controls

Manual screenshot checks:
- one screenshot for a named preset
- one screenshot for a custom mixed combination

- [ ] **Step 4: Commit**

```bash
git add index.html README.md
git commit -m "feat: ship modular wallpaper composition workflow"
```

## Execution Notes

- Keep `window.__app` intentionally rich during this refactor so browser checks stay reliable.
- Do not do aesthetic rewrites while refactoring module ownership. The first pass should preserve visual character as much as possible.
- If a module combination produces obviously broken output, disable that pairing in the compatibility matrix instead of trying to support everything at once.
