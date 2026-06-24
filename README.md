# U.M.E.R Engine — Ray Tracing Edge Cases & BVH Destruction

![Domain](https://img.shields.io/badge/Domain-Hardware%20Ray%20Tracing-9c27b0?style=flat-square)
![Validation](https://img.shields.io/badge/Validation-Monte%20Carlo%20Stress%20Tested-blue?style=flat-square)
![Optimization](https://img.shields.io/badge/Optimization-Software%20SER-00E676?style=flat-square)

> **Part of the U.M.E.R (Uniform Memory Encoded Representation) GPU Compute Core**

---

## Overview: The Industry Gauntlet

In the computer graphics industry, proving a ray tracing architecture requires surviving the "Researcher's Gauntlet." While coherent primary rays are easy to process, production renderers break under chaotic secondary bounces, massive spatial voids, and execution divergence. 

Traditionally, the industry relies on software Bounding Volume Hierarchies (BVHs) or hardware RT cores. BVHs excel at skipping empty space but choke completely on dynamic geometry rebuilds ($O(N \log N)$) and pointer-chasing memory stalls.

This repository subjects the **U.M.E.R. Sort-Free Pipeline** to extreme edge-case stress testing. The results empirically prove that U.M.E.R. trades static memory compression for instantaneous dynamic rebuilds ($O(N)$), absolute resilience to Monte Carlo chaos, and native execution sorting—while ultimately patching its only weakness (empty space) via $O(1)$ global teleportation.

---

## Benchmark 1: Incoherent Secondary Rays (Monte Carlo Resilience)

When a ray hits a rough surface, it scatters a secondary "bounce" ray in a random direction. This destroys cache coherency, normally imposing a 60% to 80% throughput penalty on BVH pipelines.

* **The Test:** Saturated Monte Carlo path tracing forcing worst-case VRAM access.
* **The Result:** * Coherent Latency: `4.52 ns/ray`
  * Incoherent Latency: `5.19 ns/ray`
  * **Throughput Penalty: Only 14.7%**
* **Architectural Reality (Latency Hiding):** Stripped to raw VRAM, random Monte Carlo hash reads degrade by 95.5%. However, the U.M.E.R. two-phase pipeline masks this entirely. Phase 1 (Macro-DDA traversal) is heavily ALU-bound. The massive arithmetic workload completely hides the memory latency of Phase 2, allowing the engine to sustain **~190 Million incoherent rays/second** on standard compute hardware.

---

## Benchmark 2: "Teapot in a Stadium" (Spatial Bounds & Memory)

What happens when you place a dense, 1-million particle teapot inside a massive 1,000,000-unit spatial void? Traditional 3D grids attempt to allocate Petabytes of memory and instantly crash.

* **The Test:** Massive spatial domain expansion with static particle counts.
* **The Result:** * VRAM Footprint: **264.00 MB**
  * Topology Build Time: **0.53 ms**
* **Architectural Reality (The Infinite Hash):** U.M.E.R. utilizes a flat 1D Spatial Hash. The spatial bounding box is functionally infinite, decoupling memory allocation from physical volume. While a BVH might "shrink-wrap" geometry to use only 64 MB, U.M.E.R. happily trades 200 MB of static VRAM to guarantee a flat array architecture that builds 19x faster than a BVH sorting algorithm.

---

## Benchmark 3: Material Divergence (Software-SER)

When perfectly mixed warps of rays hit different materials (Diffuse, Mirror, Glass), traditional "Uber Shaders" force GPU threads to serialize, crippling execution speed.

* **The Test:** Mixed-material rendering execution.
* **The Result:** * Divergent Uber Shader: `35.81 ms`
  * U.M.E.R Multi-Queue: `12.35 ms`
  * **Performance Gain: 2.90x Speedup** (Near the 3.0x theoretical maximum).
* **Architectural Reality (Software-SER):** Modern GPUs (like Ada Lovelace) require proprietary silicon (Shader Execution Reordering) to fix this. U.M.E.R. acts as a native Software-SER. By paying a microscopic ~3% latency toll to route rays into distinct atomic queues (`atomicAdd`) during intersection, the GPU launches perfectly cohesive kernels.

---

## Benchmark 4: Empty Space Teleportation (Beating the BVH)

The final BVH advantage is its ability to instantly skip massive voids. Standard grids must sequentially step through empty space (DDA stepping).

* **The Test:** Crossing ~499,990 units of empty space.
* **The Result:** * Vanilla DDA Stepping: `116.21 ms` (62,487 ALU steps)
  * U.M.E.R. Global AABB Teleportation: `0.0697 ms` (1 ALU step)
  * Standard BVH Baseline: `0.0711 ms`
* **Architectural Reality:** By computing global geometric bounds during the $O(N)$ topology build, U.M.E.R. executes a single Ray-AABB intersection test. It teleports rays across macroscopic voids fractionally faster than a BVH root-node check because it completely avoids the memory overhead of managing a traversal stack.

---

## The Final Verdict

You are not looking at a standard ray tracer; you are looking at a **Compute-Bound, Dynamic Topology Engine**. U.M.E.R. eliminates the $O(N \log N)$ sorting bottlenecks of hierarchical traversing, replacing them with flat memory arrays and ALU-bound math. It is specifically designed to bridge the gap between static architectural renders and fully destructible, real-time interactive physics at 60 FPS.

## Execution

All edge-case validations, telemetry logs, and benchmark executions are encapsulated in the Jupyter environment.

```bash
jupyter notebook ray-tracing-edgecases.ipynb
