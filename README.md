# WebGPU Habitable Planet Prototype

Full pipeline, built on the verified working terrain foundation:

1. **Terrain** — GPU compute density field (sphere + ridge/billow noise
   for a carved-valley look) + full marching cubes extraction, same
   verified technique as the base prototype.
2. **Biome coloring** — computed per-vertex in the same compute pass,
   from height (sand → grass → snow) and slope (→ rock), no textures.
3. **Water** — a simple sphere at sea level with a transmissive/
   reflective material.
4. **Atmosphere** — a Fresnel-glow shell (TSL node material) around the
   planet, additive blended.
5. **Character controller** — small-planet walker: gravity pulls toward
   planet center, "up" is away from center, WASD moves relative to
   view yaw projected onto the local tangent plane, Space jumps, and
   ground collision samples the same noise shape (via a CPU-side
   equivalent) the GPU terrain is built from.

## Run it
Serve over http(s), not file://. Click the canvas to lock the pointer
and look around; WASD to walk; Space to jump.

## Known scope cuts (see chat notes)
- Single volume for the whole planet, not a 6-face cube-sphere with
  distance-based LOD — avoids chunk-seam stitching for this pass, at
  the cost of a fixed resolution regardless of camera distance.
- Ridge/billow noise approximates an "eroded" look; this is not a real
  hydraulic erosion simulation (a separate iterative compute pass that
  would need its own tuning pass).
- Water and atmosphere are simple procedural shells, not physically
  simulated fluid/scattering.
