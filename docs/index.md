---
title: HetLitmus
hide:
  - navigation
  - toc
---

<div class="hl-hero" markdown>

# Generate and run heterogeneous CPU&#8209;GPU litmus tests on real hardware.

Multi-vendor support with CUDA for NVIDIA and HIP for AMD.

[Get started](getting-started/index.md){ .md-button .md-button--primary }
[View on GitHub](https://github.com/TUM-DSE/hetlitmus){ .md-button }

</div>

## Features

<div class="grid cards" markdown>

-   :lucide-layers:{ .lg .middle } __Ready-made tests__

    ---

    10,979 generated tests across 29 shapes, covering CPU-GPU placements, scopes and memory orders.

    [:octicons-arrow-right-24: Test corpora](guide/corpora.md)

-   :lucide-git-branch:{ .lg .middle } __Generate__

    ---

    Describe a new test as a [relaxation cycle](guide/generating-tests.md#cycles) and HetLitmus generates it.

    [:octicons-arrow-right-24: Generating tests](guide/generating-tests.md)

-   :lucide-cpu:{ .lg .middle } __Run__

    ---

    Each test compiles to one program that runs CPU and GPU threads on shared memory.

    [:octicons-arrow-right-24: Compiling tests](guide/compiling-tests.md)

-   :lucide-activity:{ .lg .middle } __Stress__

    ---

    Stress and noise on the CPU and the GPU make weak outcomes more likely to appear.

    [:octicons-arrow-right-24: Running on hardware](guide/running-on-hardware.md)

</div>

## Why HetLitmus?

Hardware-coherent CPU-GPU systems such as NVIDIA GH200 and AMD MI300A let CPU and GPU threads
share memory, yet the memory-consistency guarantees across the two devices remain unclear.
Litmus testing is an established approach to observing weak memory behaviour empirically on real
hardware.

[Walk through an example :octicons-arrow-right-24:](getting-started/quick-start.md){ .md-button .hl-walk }
