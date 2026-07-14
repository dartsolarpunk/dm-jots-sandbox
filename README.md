# WebGPU Marching Cubes — proof-of-concept prototype

## Run it
Serve this folder over HTTP (must be http/https, not file://) and open
`index.html` in a browser with real WebGPU support (current Chrome/Edge
on real hardware — check chrome://gpu shows "WebGPU: Hardware accelerated").

    python3 -m http.server 8080
    # then open http://localhost:8080/

## What it does
- `mc_tables.js` — the standard marching-cubes lookup tables (edge masks +
  triangle table), sourced/cited from the public reference implementation
  at https://gist.github.com/dwilliamson/c041e3454a713e58baf6e4f8e5fffecd
  (derived from Paul Bourke's tables, which trace to Lorensen & Cline's
  original 1987 paper).
- `index.html` — two GPU compute passes using Three.js's TSL (Three
  Shading Language) system, which compiles to real WGSL compute shaders:
  1. **Density pass**: evaluates a signed-distance-style field (sphere +
     layered fractal noise) at every grid point of a chunk.
  2. **Marching cubes pass**: one GPU thread per grid cell — computes the
     8-corner case index, looks up the triangle table, interpolates edge
     intersections, and writes triangles directly into a GPU storage
     buffer that's bound straight to the render mesh's position/normal
     attributes. No CPU readback anywhere in the pipeline.

## Status
Both compute passes run with zero shader errors — verified directly
against a live WebGPU device. The final render step hit a texture-view
error that traces to Three.js's internal render-target setup, reproducing
even with antialiasing off and across different GPU backend flags. That
points to a software-WebGPU (SwiftShader) implementation gap in the
specific testing environment used, not the shader code itself — needs
verification on a real GPU-backed browser to confirm.

## Next steps once render is confirmed
- Extend to a real cube-sphere (6 warped faces) instead of one chunk.
- Distance-based chunk LOD (subdivide near camera, coarsen far).
- Swap the flat fixed-slot-per-cell output for indirect/compacted draws
  once correctness is confirmed (more efficient, avoids the empty
  degenerate triangles the current fixed-slot approach wastes GPU on).
- Hook the density function up to per-planet parameters (seed, radius,
  water level, biome noise layers) instead of the placeholder sphere+noise.
