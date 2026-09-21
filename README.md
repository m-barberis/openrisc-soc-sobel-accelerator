# Real-Time Motion Detection on an OpenRISC FPGA SoC

Sobel edge detection and frame-difference motion detection on a live 640×480 camera feed, running on an OR1420 (OpenRISC) soft core at 74.25 MHz on a Lattice ECP5 FPGA (GECKO5). Implemented in Verilog and C.

The initial software-only implementation achieved 0.16 fps. Three rounds of profiling and hardware/software optimization increased system throughput to 7.7 fps, at which point camera acquisition rather than computation became the bottleneck.

- Profiled the software baseline and identified Sobel filtering as 88% of the per-frame execution cost, with 81% of that time caused by CPU stalls rather than arithmetic.
- Implemented Sobel as a custom CPU instruction with a multiplier-free datapath, packing the 3×3 neighbourhood into two 32-bit operands while omitting the zero-weighted centre pixel. This accelerated the Sobel stage by 6.9×; subsequent profiling showed that data transfer to the unit had become the dominant cost.
- Designed two bus-master hardware accelerators with line buffering and 16-word burst transfers, removing the CPU from the inner processing loop. The resulting accelerators achieved 216× speedup for Sobel and 17.4× for motion detection, while reducing CPU stalls from approximately 326 million to ~100 cycles per frame.
- Overlapped camera acquisition and computation using a non-blocking capture loop and three pairs of double buffers, with frame buffers exchanged by pointer rather than copied.

Per-frame computation time decreased from 6.12 s to 66 ms, corresponding to a 93× compute speedup. End-to-end system throughput increased from 0.16 fps to 7.7 fps, a 48× improvement, with the remaining limit imposed by camera acquisition.

The project was built using the OSS CAD Suite (Yosys, nextpnr-ecp5, openFPGALoader) and an OpenRISC GCC toolchain and deployed on the physical GECKO5 board. Reproducing the exact setup requires access to the EPFL GECKO5 teaching platform or adaptation of the design to another compatible FPGA system.

---

EPFL CS-476 — Embedded Systems Design  
Two-person project. The course provided the baseline SoC infrastructure; the hardware accelerators and final capture/processing software were developed collaboratively by the two project members.
