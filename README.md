# VoxelEGL

## Images of execution of the src files:

![Execution Screenshot 1](images/Screenshot_20260906_185523.png)
![Execution Screenshot 2](images/Screenshot_20260906_185540.png)
![Execution Screenshot 3](images/Screenshot_20260906_185553.png)

A voxel terrain renderer written from scratch with **C++23, OpenGL 4.6 and GLSL**.

VoxelEGL explores the architecture of a Minecraft-style voxel renderer: chunked world generation, CPU-side visible-face extraction, packed GPU vertex data, ambient occlusion, frustum culling and instanced OpenGL rendering.

> This is a graphics-engineering project and rendering experiment, not a complete voxel game.

## Features

- C++23
- OpenGL 4.6 core profile
- Chunk-based voxel world
- Procedural terrain generation
- Multi-octave 2D Perlin noise
- 16×16 voxel columns per chunk
- 256 vertical voxel levels
- Chunk border data for neighbor visibility checks
- CPU-side hidden-face culling
- Per-vertex ambient occlusion
- Packed 32-bit voxel face data
- Instanced rendering
- Frustum culling
- Asynchronous chunk generation
- Dynamic chunk regeneration around the player
- First-person camera
- Directional lighting
- HDR environment contribution
- Texture mipmaps
- Back-face and depth testing

## World representation

The terrain is organized into chunks.

The current renderer maintains a **32 × 32 active chunk region** around the player.

Each chunk contains:

- a `16 × 16` rendered horizontal area
- a one-voxel border used for neighborhood checks
- `256` vertical voxel levels

The internal chunk storage therefore uses an `18 × 18 × 256` working volume.

## Procedural terrain generation

Terrain height is generated with layered 2D Perlin noise.

The implementation combines multiple octaves with configurable persistence and lacunarity.

The resulting heightmap determines the solid voxel volume for each chunk.

The current terrain generator uses four noise octaves and converts the result into a height value up to the configured world height.

## Chunk generation

New chunks are generated asynchronously using `std::async`.

This moves the expensive world-generation work away from the OpenGL draw loop.

## Visible-face extraction

The CPU does not submit every voxel cube to OpenGL.

Instead, it checks neighboring blocks and emits only faces whose adjacent voxel is empty.

For each visible face, the renderer calculates:

- block position
- face direction
- four corner ambient-occlusion values

This significantly reduces the amount of geometry that needs to be drawn.

## Packed vertex representation

Voxel faces are stored as a single `uint32_t`.

The exact shader-side layout is decoded from the packed integer.

The vertex shader reconstructs the corresponding cube face procedurally rather than storing six complete vertices for every face.

This reduces CPU-generated vertex data and allows one compact integer attribute to describe a face.

## Ambient occlusion

Voxel corners receive local ambient-occlusion values based on neighboring occupied voxels.

The CPU computes the four corner AO values for each visible face and packs them into the face data.

The vertex shader expands these values into an AO factor that is interpolated across the face.

## GPU rendering

The renderer uploads packed face data into OpenGL vertex buffers.

Each face is represented by one instance, while the shader procedurally generates the six vertices required for that face.

Conceptually:

```cpp
glDrawArraysInstanced(
    GL_TRIANGLES,
    0,
    6,
    face_count
);
```

This allows the CPU to store compact per-face data while the GPU reconstructs the actual geometry.

## Frustum culling

Before a chunk is rendered, the camera performs a frustum test.

Chunks outside the camera frustum are skipped.

This prevents off-screen chunks from entering the OpenGL draw loop.

## Lighting

The fragment shader currently combines:

- directional diffuse lighting
- specular lighting
- ambient/environment contribution
- HDR environment sampling
- texture color
- voxel ambient occlusion
- exposure mapping
- gamma correction

The environment texture is sampled using a spherical mapping based on the surface normal.

## Textures

The current renderer uses separate textures for different grass surfaces, together with an HDR environment texture.

Texture filtering includes mipmapping and linear filtering for the HDR environment.


## Requirements

- CMake 3.25+
- C++23 compiler
- OpenGL 4.6 capable GPU/driver
- GLFW3
- GLM
- GLAD
- stb_image

## Build

Install all the Requirements before building this.

```bash
git clone https://github.com/Leonuraht/VoxelEGL.git
cd VoxelEGL

cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
```

Run:

```bash
./build/VoxelE
```

## Runtime pipeline

```text
Player movement
      ↓
Chunk boundary crossed?
      ↓
Generate missing chunks asynchronously
      ↓
Create voxel data
      ↓
Extract visible faces
      ↓
Compute AO
      ↓
Upload VBOs
      ↓
Frustum culling
      ↓
Instanced OpenGL rendering
      ↓
Lighting + textures + HDR
```

## Current limitations

VoxelEGL is still an experimental renderer.

Current limitations include:

- no block editing system
- no collision system
- no physics
- no block inventory
- no persistence
- no save/load system
- no greedy meshing
- no texture atlas/material system
- no GPU chunk meshing
- no occlusion culling
- no LOD
- no multiplayer
- terrain generation is currently heightmap-based rather than fully volumetric


## Technical focus

The project is mainly an exploration of how a voxel engine can map sparse block data into an efficient rendering representation.

This keeps the CPU representation compact while moving repetitive cube geometry reconstruction to the GPU.

## License

See the repository for license information.
