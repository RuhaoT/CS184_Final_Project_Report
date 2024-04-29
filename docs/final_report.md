![final project showcase](images/final_report/finalprojectshowcase.gif)

# Machine Learning Enhanced Computational Design of High-level Interlocking Puzzles

By Ruhao Tian, Yunshen Song, Zixuan Wan, Zineng Tang

## Abstract

## Technical Approach

### Early Attempts of CUDA Migration

In the very early stage of our project, we decided to implement a CUDA-accelerated puzzle generator based on the SIGGRAPH 2022 paper "Computational Design of High-level Interlocking Puzzles". The paper introduces a novel method to generate high-level interlocking puzzles with a variety of constraints. However, the method is time-costly and we believe that it can be significantly accelerated by parallelizing the computation on GPU.

The method proposed by the paper is based on 'trials and errors'. It may take thousands of independent trials to generate a disassemblable puzzle. Our original idea is to move the trials to GPU and run them in parallel, which can significantly reduce the time cost.

We believed that we could directly migrate the original C++ code to CUDA code without significant changes. However, after extensive exploration, the difficulty of CUDA programming became apparent. CUDA does not support dynamic data structures, which forced us to redirect our approach.

The key algorithm of the proposed method heavily involves graph traversal. The original C++ code implementation consists of dynamic data structures, e.g. `std::vector` to store the intermediate results. After extensive testing, we believe that it is impractical to directly migrate the original C++ code to CUDA code given the time and resources available to us. Even if we brutally replace all the `std::vector` with fixed-size arrays, the code would become inefficient because all expansions to arrays would require copying the entire array to a new location.

This understanding is a significant setback, but we believe GPU acceleration is still possible. Instead of directly migrating all the original C++ code to CUDA code, we now need to carefully select parts of the algorithm we want to accelerate and redesign the algorithm to fit the CUDA programming model.

### Profiling and Bottleneck Identification

To identify our selective acceleration target, we profile the implementation of the paper in search of a bottleneck. The implementation of the paper executes on the Linux platform and our group members either use macOS or virtualized WSL on Windows, consequently, we profile both platforms to minimize the difference. We use `perf` on WSL and Xcode Instruments on macOS to profile the original C++ code.

![Profile result](./images/milestone_status_report/flamegraph3.svg)

Profiling results from both platforms identically show that the bottleneck of the original algorithm is computing the main path when creating a new piece, which consumes around 80% of execution time in some cases.

![bottleneck](./images/final_report/bottleneck.png)

To elaborate, within each trial of the algorithm, every piece of the puzzle is created by (1) finding a small set of seed voxels `SeedPath` that is movable in the target direction, (2) adding a set of block voxels `BlockPath` to prevent movement in other directions, and (3) adding voxels with small reachability to extend the piece. According to the profiling results, (1) and (2) are the most time-consuming parts of the algorithm.

### Selective Acceleration by Machine Learning

#### Dataset Generation

To gather data for training, we implemented a dataset kernel to record the generation process of `SeedPath` and `BlockPath`.

![dataset kernel](./images/final_report/dataset_kernel.png)

When the `SeedPath` generation starts, a `seedPathCreationSequence` is created to record the intermediate states of the `SeedPath`. Every time the `SeedPath` changes, the updated `SeedPath` is appended to the `seedPathCreationSequence`. The final `seedPathCreationSequence` is saved to a JSON file. The same process is applied to `BlockPath`.

### Benchmarking

## Results

## References

**[1]** SIGGRAPH 2022 paper "Computational Design of High-level Interlocking Puzzles" [ACM Digital Library](https://dl.acm.org/doi/abs/10.1145/3528223.3530071)

**[2]** Recursive interlocking puzzles [ACM Digital Library](https://dl.acm.org/doi/10.1145/2366145.2366147)

## Contributions

不要忘了写各自的contribution！