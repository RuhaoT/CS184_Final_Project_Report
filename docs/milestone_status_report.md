![Example flamegraph](./images/milestone_status_report/flamegraph.svg)

# Milestone Status Report: The Story So Far

By Ruhao Tian

## Introduction: Our original plan

In the very early stage of our project, we decided to implement a CUDA-accelerated puzzle generator based on the SIGGRAPH 2022 paper "Computational Design of High-level Interlocking Puzzles". The paper introduces a novel method to generate high-level interlocking puzzles with a variety of constraints. However, the method is time-costly and, in our early experiments, it takes several minutes to generate a single puzzle. We believe that the method can be significantly accelerated by parallelizing the computation on GPU.

The method proposed by the paper is based on 'trials and errors'. It generates a puzzle in an approach that may encounter deadlocks, and if it does, it will start another trial with some random settings. It may take thousands of independent trials to generate a disassemblable puzzle. Our key idea is to move the trials to GPU and run them in parallel, which can significantly reduce the time cost.

## Full CUDA Migration: A dead end

No group member of our project has experience in CUDA programming. Based on the description of CUDA, it is claimed to fully support C++ syntax and we misunderstood its meaning. We thought that we could directly migrate the original C++ code to CUDA code without significant changes. However, after several days of exploration, we discovered a serious problem with CUDA is that it does not support dynamic data structures.

The key algorithm of the proposed method heavily involves graph traversal. The original C++ code uses a lot of dynamic data structures like `std::vector` to store the intermediate results. After extensive testing, we found that it is impossible to directly migrate the original C++ code to CUDA code. Even if we brutally replace all the `std::vector` with fixed-size arrays, the code would become inefficient because all expansions to arrays would require copying the entire array to a new location.

This understanding is a significant setback for our project, but we believe GPU acceleration is still possible. Instead of directly migrating all the original C++ code to CUDA code, we now need to carefully select parts of the algorithm we want to accelerate and redesign the algorithm to fit the CUDA programming model.

## Selective Acceleration: Identifying the bottleneck

As rewriting the entire algorithm in CUDA is not feasible, we need to identify the bottleneck of the original algorithm and accelerate it. The implementation of the paper executes on Linux platform and our group members either use macOS or virtualized WSL on Windows, so we profile both platforms to minimize the difference. We use `perf` on WSL and Xcode Instruments on macOS to profile the original C++ code.

![Profile result](./images/milestone_status_report/flamegraph3.svg)

Profiling results from both platforms identically show that the bottleneck of the original algorithm is computing the main path when creating a new piece, which consumes around 80% of execution time in some cases. To elaborate, the algorithm needs (1) to find a small set of seed voxels `SeedPath` that is movable in the target direction, (2) add a set of block voxels `BlockPath` to prevent movement in other directions, and (3) add voxels with small reachability to extend the piece. When computing the start and end of the paths, BFS is used, we believe that this is the root cause of the bottleneck.

## Further Steps

We are currently discussing optimizing the bottleneck as well as other parts of the algorithm. One possible way is to replace the path generation part with a neural network. No further conclusion has been made yet, but we are actively working on it.

## Additional Resources

- [Milestone Report Slides](https://docs.google.com/presentation/d/15-hStC1AsBgCzySKr5KrJB9gDAvTDwbA/edit?usp=drive_link&ouid=104238444029078377788&rtpof=true&sd=true)
- [Milestone Report Video](https://drive.google.com/file/d/13GXqBGRbrep4ciOEqJZoV9xwbnP8DNP1/view?usp=drive_link)