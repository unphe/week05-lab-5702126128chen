# Copilot instructions for this pendulum simulator repo

## Project snapshot

- This repository is a small static-browser visualization project, not a package-based application.
- The main deliverables are two self-contained HTML demos:
  - `index.html` for the single pendulum
  - `double_pendulum.html` for the double pendulum
- Each file embeds its own CSS and JavaScript in the same document, using a `<canvas>` for drawing and DOM controls for the parameter panel.
- The expected workflow is to edit the HTML/JS directly and then verify the behavior in a browser.

## Build, test, and lint commands

There is no build system, test runner, or lint configuration in this repository.

- No `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, or similar project manifest is present.
- No CI workflow or lint script is configured.
- Manual validation is the current verification path:
  1. Open `index.html` or `double_pendulum.html` in a browser.
  2. Confirm the animation renders and updates smoothly.
  3. Adjust the sliders and verify the pendulum response changes immediately.
  4. Check the browser console for JavaScript errors.

## High-level architecture

### 1) Self-contained browser app

Each HTML file is effectively a standalone app:

- The page structure defines the visual shell (canvas, HUD, slider controls, buttons).
- The script runs immediately with a single IIFE (`(function () { ... })();`).
- State and rendering logic remain in the same file instead of being split across modules.

This means future edits are usually best done in the same file that owns the behavior being changed.

### 2) Simulation state and parameter separation

The scripts intentionally separate live configuration from runtime state:

- `P` holds the current user-tunable parameters (length, gravity, damping, mass, etc.).
- `S` and `Y` hold the simulation state (angle, angular velocity, time, and the double-pendulum state vector).
- `cam` holds camera/world-view state used for pan/zoom behavior.
- `trail`, `dragMode`, and related UI state are kept near the interaction logic rather than in a global app state object.

### 3) Physics and rendering are colocated

For both pendulum variants, the same file contains:

- numerical integrators (`integrate` for simple pendulum, `rk4` for double pendulum)
- physical formulas and derivations
- drawing functions (`drawGrid`, `drawPendulum`, `drawTrail`, etc.)
- pointer/touch interactions and zoom logic
- UI event handlers for sliders, pause, reset, and camera controls

This repo favors a single-file, tightly coupled architecture; keep the logic near its related code when modifying it.

## Key conventions specific to this codebase

### Units are explicit and must stay consistent

The code intentionally uses pixels for the visible length values and then converts to meters before using real-world physics.

- `PPM = 100` converts pixels per meter.
- `Lpx`, `L1px`, and `L2px` are display-space lengths.
- The actual equations use meter-based values, such as `g = 9.8` and `L_m = L_px / PPM`.

Do not remove this conversion or substitute raw pixel values into the formulas. It is a core correctness rule for the simulation.

### Camera transforms follow a consistent world/screen model

The project uses world coordinates for the pendulum and a screen-space transform pipeline:

- `toWorld()` converts pointer/screen coordinates into world coordinates.
- `beginWorld()` and `endWorld()` wrap drawing in a transform to render in the world coordinate system.
- `cam.x`, `cam.y`, and `cam.zoom` are kept in sync with resize and zoom logic.

If you change camera code, keep pointer hit-testing and panning behavior aligned with the same transform model.

### Angle normalization is part of the numerical stability strategy

The double-pendulum script normalizes angles with `wrapPi()` to avoid drift from repeated large angle values.

- This is not cosmetic; it prevents long-running simulation drift and preserves clean angle values for sin/cos calculations.
- Keep this behavior when modifying the integrator or state updates.

### Interaction logic is tightly tied to screen-space hit detection

Pointer/touch behavior uses `toWorld()` and a drag mode state such as:

- `dragMode = 'bob'` for dragging the pendulum bob
- `dragMode = 'pan'` for camera panning
- `dragMode = 'pinch'` for multi-touch zooming

When modifying interaction code, preserve the same world-to-screen conversion and drag target logic; otherwise the user interaction will feel offset or broken.

### Preserve the physical comments and formulas

The scripts contain detailed comments explaining the derivations and numerical choices. These comments are part of the engineering intent of the project.

- If adjusting the model, keep the formulas and comments in sync.
- The repo is small enough that the physics derivations are a useful source of truth for future edits.

## Working style for this repo

- Prefer small, direct changes in the same HTML file instead of introducing new frameworks or module systems.
- Keep state and UI logic together in the file that owns the feature being changed.
- Verify visual behavior in the browser after each change rather than relying on broad tooling.
- When the code is physics-sensitive, favor a focused, local fix over refactoring unrelated sections.
