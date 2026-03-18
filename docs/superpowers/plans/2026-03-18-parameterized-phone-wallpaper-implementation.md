# Parameterized Phone Wallpaper Generator Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file, static, browser-only phone wallpaper generator with four geometric presets, layered controls, live preview, and PNG export.

**Architecture:** Keep the shipped app inside one `index.html` file with inline CSS and JavaScript, but structure the code into clear in-file modules: state, preset definitions, render engine, control rendering, and export. Use a shared Canvas drawing pipeline for both preview and export so every visual change is deterministic and consistent across viewport rendering and final output.

**Tech Stack:** HTML, CSS, vanilla JavaScript, Canvas 2D API, browser download APIs, Playwright MCP for browser verification

---

## File Structure

- Create: `index.html`
  - Single shipped artifact containing layout, styling, state model, preset definitions, render engine, controls, and export logic.
- Modify: `docs/superpowers/specs/2026-03-18-parameterized-phone-wallpaper-design.md`
  - Only if implementation uncovers a real mismatch that must be reflected back into the design spec.
- Reference: `PRINCIPLES.md`
  - Keep implementation decisions aligned with single-file delivery, static hosting, and result-first UI.

## Notes Before Execution

- The workspace is currently not a Git repository, so commit steps are intentionally omitted from this plan.
- There is no local package or test runner. Verification will use browser-based checks through Playwright MCP plus targeted console assertions inside the app while keeping the shipped artifact dependency-free.
- TDD still applies at the behavior level: for each major feature slice, write the browser verification first, watch it fail, then implement the minimum code to make it pass.

## Chunk 1: Single-File App Shell

### Task 1: Create the static page skeleton and empty state contract

**Files:**
- Create: `index.html`
- Reference: `docs/superpowers/specs/2026-03-18-parameterized-phone-wallpaper-design.md`

- [ ] **Step 1: Write the failing browser check**

Check that the app shell does not exist yet, so the first UI contract is still missing.

Run with Playwright MCP against `file:///Users/zhaolixing/Documents/GitHub/wallpaper-maker/index.html` after file creation attempt.
Expected initial failure:
- Page is missing or does not contain:
  - a left control panel
  - a right preview stage
  - a canvas element
  - primary actions for regenerate, randomize, reset, and export

- [ ] **Step 2: Create the minimal page shell**

Add `index.html` with:

```html
<main class="app-shell">
  <aside class="control-panel"></aside>
  <section class="preview-panel">
    <canvas id="wallpaperCanvas"></canvas>
  </section>
</main>
```

Include:
- responsive CSS for desktop left-controls/right-preview and mobile top-preview/bottom-controls
- a visible title and short helper line
- semantic button and field containers for future controls

- [ ] **Step 3: Verify the shell check passes**

Run the same Playwright shell check again.
Expected:
- `index.html` loads from a local file URL
- the page shows the control panel, preview panel, and canvas
- layout changes correctly when viewport switches between desktop and mobile widths

### Task 2: Add the base state model and first paint path

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Define the first render contract:
- on page load, the canvas should not remain blank
- a default preset should be selected
- default size inputs should be populated

Expected initial failure:
- the page shows a canvas, but no rendered wallpaper or no default state-driven UI

- [ ] **Step 2: Add minimal state and first render**

Implement inline JavaScript that:

```js
const state = {
  preset: 'minimal-grid',
  width: 1290,
  height: 2796,
  seed: 314159,
  palette: 'dawn',
  density: 0.45,
  complexity: 0.5,
};
```

Add:
- startup initialization
- deterministic seeded random helper
- a first simple render path that paints background plus a few visible guide elements

- [ ] **Step 3: Verify the page paints a non-empty wallpaper**

Run a Playwright check that samples the canvas and confirms pixel variance is greater than zero.
Expected:
- the canvas bitmap is not blank
- a preset control reflects the default preset
- width and height controls reflect the default export size

## Chunk 2: Preset Engine And Shared Parameters

### Task 3: Build preset definitions and common parameter schema

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Add a failing check that expects:
- four preset options
- common controls for width, height, preset size, palette, density, complexity, and seed
- a hidden or collapsed advanced control area

Expected initial failure:
- fewer than four presets exist, or controls are missing

- [ ] **Step 2: Implement preset registry and shared control schema**

Create an in-file preset map like:

```js
const presets = {
  'minimal-grid': { /* defaults, visible controls, render weights */ },
  'bold-blocks': { /* ... */ },
  'tech-texture': { /* ... */ },
  'organic-flow': { /* ... */ },
};
```

Also define:
- common size presets
- palette presets
- common parameter metadata
- advanced parameter metadata per preset

- [ ] **Step 3: Verify control coverage**

Run the same browser check and confirm:
- all four preset options render
- shared controls are visible by default
- advanced controls are collapsed until opened

### Task 4: Implement preset switching and live control updates

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Define interactive expectations:
- switching presets changes visible advanced controls
- changing density or complexity triggers a visible redraw
- editing the seed and regenerating changes the image deterministically

Expected initial failure:
- controls update text only, or redraw does not happen, or preset-specific controls stay static

- [ ] **Step 2: Add event wiring and state updates**

Implement:
- control-to-state bindings
- re-render on input changes
- preset change handlers that merge preset defaults into current state
- advanced control visibility based on preset configuration

- [ ] **Step 3: Verify redraw behavior**

Run browser checks that:
- capture the canvas before and after density changes and confirm pixel data changes
- switch among the four presets and confirm control groups update
- enter the same seed twice and confirm the generated image is reproducible

## Chunk 3: Full Render Pipeline

### Task 5: Implement the shared layered Canvas renderer

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Define rendering expectations for each preset:
- minimal preset shows structured lines or grid rhythm
- bold preset shows large blocks or circles
- tech preset shows texture or radiating/scanning structure
- organic preset shows flowing or curved composition

Expected initial failure:
- presets produce nearly identical visuals or only placeholder geometry

- [ ] **Step 2: Implement the renderer in minimal slices**

Add helper functions for:

```js
function drawBackground(ctx, state, rng) {}
function drawBaseGeometry(ctx, presetConfig, state, rng) {}
function drawAccentLayer(ctx, presetConfig, state, rng) {}
function drawTextureLayer(ctx, presetConfig, state, rng) {}
```

Keep all helpers inline in `index.html`, but organized by section comments.

Implement just enough preset-specific behavior so each preset is visually distinct while still sharing the same pipeline order.

- [ ] **Step 3: Verify preset distinction**

Run browser checks that:
- export canvas thumbnails or screenshots for all four presets
- confirm each preset produces non-empty, visibly different output
- confirm no console errors occur during repeated preset switching

### Task 6: Add advanced controls for style depth without turning the default UI into a parameter wall

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Expect:
- an advanced controls toggle
- preset-specific sliders or selects inside the advanced section
- advanced controls hidden on load and visible after expand

Expected initial failure:
- advanced controls are always visible, absent, or not preset-specific

- [ ] **Step 2: Implement advanced controls**

Add grouped controls for parameters such as:
- geometry weights
- alignment and grid subdivision
- margin or safe-area ratio
- gradient strength
- texture intensity
- disturbance amount

Render only the groups relevant to the current preset.

- [ ] **Step 3: Verify advanced control behavior**

Run browser checks that:
- expand the advanced section
- adjust one preset-specific control
- confirm the canvas redraws
- switch presets and confirm advanced controls change accordingly

## Chunk 4: Export, Validation, And Polish

### Task 7: Add size presets, custom size validation, and PNG export

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Expect:
- size preset selection updates width and height
- invalid width or height is rejected or corrected
- export triggers a PNG download

Expected initial failure:
- size presets do nothing, invalid sizes are accepted silently, or export is missing

- [ ] **Step 2: Implement size and export flows**

Add:
- common mobile size preset list
- width and height inputs with validation and last-valid fallback
- offscreen canvas export using the same renderer
- download filename generation based on preset and timestamp/seed

- [ ] **Step 3: Verify export flow**

Run browser checks that:
- select a preset size and confirm width and height update
- type an invalid size and confirm the UI recovers gracefully
- trigger export and confirm a PNG download event fires

### Task 8: Add user feedback, empty-error safeguards, and responsive polish

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Expect:
- export failures surface readable feedback
- invalid input shows a local message
- mobile layout stacks preview before controls
- the preview remains the visual focus

Expected initial failure:
- no feedback is shown, or the mobile order is wrong, or the layout feels like a raw form dump

- [ ] **Step 2: Implement the minimal polish**

Add:
- lightweight inline feedback region
- input helper text only where needed
- restrained panel styling so the tool does not visually overpower the wallpaper
- responsive spacing and type adjustments for small screens

- [ ] **Step 3: Verify final UX contract**

Run browser checks that:
- resize to mobile viewport and confirm preview comes first
- trigger validation feedback and confirm message text is visible
- confirm buttons remain usable and the preview canvas remains large and central

## Final Verification Checklist

- [ ] Open `index.html` directly in a browser and confirm the default wallpaper renders immediately.
- [ ] Verify all four presets render distinct results.
- [ ] Verify common controls update the preview live.
- [ ] Verify advanced controls stay collapsed by default and expand on demand.
- [ ] Verify size presets, custom dimensions, regenerate, randomize, reset, and export all work.
- [ ] Verify desktop uses left controls and right preview.
- [ ] Verify mobile uses top preview and bottom controls.
- [ ] Verify no backend, no build step, and no runtime dependency is required to use the shipped app.

Plan complete and saved to `docs/superpowers/plans/2026-03-18-parameterized-phone-wallpaper-implementation.md`. Ready to execute?
