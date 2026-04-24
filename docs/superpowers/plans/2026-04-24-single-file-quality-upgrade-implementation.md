# Single File Quality Upgrade Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Improve the single-file wallpaper maker's artwork quality and selection workflow without adding any runtime dependency.

**Architecture:** All app behavior remains in `index.html`. The change adds candidate-state generation, thumbnail rendering, lighter UI styling, and tuned Canvas drawing helpers inside the existing script.

**Tech Stack:** Static HTML, CSS, JavaScript, Canvas 2D, existing optional WebGL pass, Playwright-based local verification.

---

### Task 1: Add Candidate Selection Workflow

**Files:**
- Modify: `index.html`

- [ ] Add candidate-strip markup below `#presetGrid` with `#candidateStrip` and `#refreshCandidatesButton`.
- [ ] Add `elements.candidateStrip` and `elements.refreshCandidatesButton`.
- [ ] Add `createCandidateState(index)`, `refreshCandidates()`, `applyCandidate(candidateState)`, and `renderCandidateTile(candidateState)`.
- [ ] Wire candidate refresh on preset, random, mix, reset, and refresh button actions.
- [ ] Expose `refreshCandidates` and `applyCandidate` through `window.__app` for browser checks.

### Task 2: Upgrade Artwork Rendering

**Files:**
- Modify: `index.html`

- [ ] Add reusable helpers for diagonal light fields, inner phone-safe shading, fine grain, and highlight strokes.
- [ ] Tune `drawMinimalGeometry`, `drawBlockGeometry`, `drawOrganicGeometry`, `drawFluidBlobGeometry`, and `drawTextureLayer`.
- [ ] Keep `renderArtwork(ctx, width, height, state)` as the single rendering entry.
- [ ] Keep export and preview on the same render path.

### Task 3: Reduce Interface Weight

**Files:**
- Modify: `index.html`

- [ ] Reduce title scale and remove the heavy two-line visual dominance.
- [ ] Make preset and candidate controls easier to scan.
- [ ] Move preview metadata into a less intrusive compact strip.
- [ ] Keep mobile order as preview first, controls second.

### Task 4: Documentation and Verification

**Files:**
- Modify: `README.md`

- [ ] Mention candidate selection in the feature list.
- [ ] Run a local static server and Playwright checks for desktop, mobile, candidates, and preset image metrics.
- [ ] Save temporary screenshots outside the repo for visual review.
