# Identicon-style generative art from a parameter vector

Each cell visualizes a small set of continuous 0-1 parameters as a deterministic pixel grid with animated overlays.

## Core technique: seeded symmetric block grid

1. Quantize the parameter values (multiply by ~40, round) and pack them into a single integer seed. This ensures that similar param values produce the same visual -- adjacent entries in a gradient look like relatives, not strangers.

2. Feed the seed into a simple LCG PRNG (`s = (s * 16807) % 2147483647`) that returns the same sequence every frame for the same params.

3. Draw a 10x5 cell grid, but only generate the left 5 columns -- mirror each filled cell to `(cols-1-x)` on the right. This horizontal symmetry is what makes it read as a "face" or "icon" rather than noise (same trick as GitHub identicons).

4. For each grid cell, the PRNG decides fill/empty (threshold controlled by one param, like "density") and picks from a 3-color palette. The palette is derived from params: one controls base hue, another shifts a secondary hue, a third controls saturation. Active/highlighted state bumps luminance.

## Animated overlays (non-deterministic, time-driven)

On top of the static grid, layer:

- A **radial gradient glow** that slowly drifts position using `sin(time)`, scaled by a "breath" param. Gives a living, pulsing feel without redrawing the grid.
- **Shimmer pixels**: small bright dots whose visibility oscillates via `sin(time * 2 + i * offset)`. Only drawn above a param threshold.
- **Sparkle diamonds**: rotated 45-degree squares that phase in/out, driven by a different param (in our case "chimes"). Uses `ctx.rotate(PI/4)` around each point.

## Key design properties

- **Deterministic structure, animated surface** -- the grid never changes for the same params, only the overlays move
- **Quantized seed** means nearby param values (within 1/40th) produce identical grids -- critical for making a gradient of settings look like a family
- **Symmetry does most of the heavy lifting** for "looking designed" -- random asymmetric blocks just look like noise
- The **palette math is simple** (`baseHue + param * range`) but reads as intentional because the symmetric structure gives it context
- Keep the **grid small** (5x5 generating half = 25 decisions) -- too many cells and the symmetry stops registering
- **Canvas aspect ratio of 2:1** works well for the mirrored layout
