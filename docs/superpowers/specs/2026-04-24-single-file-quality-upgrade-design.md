# Single File Quality Upgrade Design

## Goal

Keep the project as a single-file wallpaper tool while making the first-screen result feel more finished, easier to choose from, and less like a parameter demo.

## Non-Negotiable Constraint

The working product stays in `index.html`. No build step, no package install, no runtime dependency, no backend. Documentation can change, but the app must still run by double-clicking the HTML file or through GitHub Pages.

## Product Changes

- Add a compact candidate strip near the preset area. It shows several rendered variants of the current direction, and selecting one applies its seed, palette, and tuned values.
- Keep the existing preset, free-composition, size, parameter, and export model.
- Keep advanced controls collapsed by default.
- Make preview metadata less intrusive so it does not cover the wallpaper content.

## Visual Changes

- Reduce the interface's visual weight: smaller title, calmer control panel, lighter preview frame.
- Give preset buttons and candidates enough visual signal to support choice without adding explanatory copy.
- Improve default artwork quality by adding more composition structure, sharper focal layers, cleaner texture, and less flat blur.
- Make every default preset visibly distinct at a glance.

## Rendering Direction

- Preserve the current Canvas 2D plus existing WebGL pass.
- Add reusable helpers for gradient washes, edge shadows, highlights, and subtle paper/grain detail.
- Tune presets with stronger center/diagonal/foreground relationships instead of relying only on soft blobs.

## Verification

- Browser check confirms the page still loads with no console errors.
- DOM check confirms candidate strip exists and applying a candidate changes state.
- Pixel check confirms each preset has non-trivial contrast and visual information.
- Desktop and mobile screenshots confirm the preview remains first and controls remain usable.
