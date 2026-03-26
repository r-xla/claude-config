# r-xla Ecosystem

## Overview

r-xla brings XLA-based machine learning compilation to R. All packages are developed together as a cohesive ecosystem -- when implementing a feature that spans multiple packages, make changes across all of them in a single effort. For example, adding a new operation may require changes in stablehlo (IR), pjrt (execution), anvil (user API), and tengen (generics).

The packages form a layered stack:

```
anvil          (user-facing: JIT compilation + autodiff, like JAX for R)
  |
stablehlo      (IR layer: create and manipulate StableHLO programs)
  |
pjrt           (runtime: compile and execute on CPU/CUDA/Metal/TPU)
  |
tengen         (tensor generics: shape(), dtype(), device(), as_array())
xlamisc        (shared utilities: LRU cache, formatting helpers)
```

Supporting repos:

- **docker** -- Daily Docker images (CPU + CUDA) with anvil pre-installed.
- **benchmarks** -- Performance comparisons of anvil vs PyTorch/torch.

## Packages

### anvil

Code transformation framework for R (like JAX). Provides `nv_*` API functions and `nvl_*` primitives for JIT compilation (`nv_jit()`) and automatic differentiation. Uses 1-based indexing; delegates to stablehlo (0-based) for IR generation.

### stablehlo

Creates and transforms StableHLO programs (a portable ML computation representation). Operations follow the StableHLO spec (see SPEC.md). Uses 0-based indexing. The `Func` object uses reference semantics; other objects use value semantics.

### pjrt

R interface to PJRT (Pretty much Just another RunTime). Compiles stableHLO programs to hardware-specific executables and runs them. Manages devices, buffers, and async execution. Supports CPU, CUDA, Metal, and TPU backends.

### tengen

Defines S3 generics for tensor operations: `shape()`, `dtype()`, `device()`, `as_array()`, `ndims()`, `nelts()`. Also provides the data type hierarchy (`BooleanType`, `IntegerType`, `FloatType`, etc.). This is the common interface that pjrt and anvil implement.

### xlamisc

Shared utility library: `LRUCache`, `seq0()`, `list_of()`, `shapevec_repr()`, `vec_repr()`, `format_bib()`. Depended on by the other packages for common operations.

### docker

Dockerfiles and CI for building daily images: `anvil-cpu`, `anvil-cuda`, and benchmark variants. Published to Docker Hub and GHCR.

### benchmarks

MLP training benchmarks comparing anvil, R torch, and PyTorch on CPU (single/multi-threaded). Uses Docker images for reproducible environments. Results published via Quarto website.

## Navigating Between Packages

All repos are assumed to be sibling directories under a common parent:

```
<parent>/
├── anvil/
├── stablehlo/
├── pjrt/
├── tengen/
├── xlamisc/
├── docker/
├── benchmarks/
└── claude-config/
```

To access a sibling package from any repo, use `../<package>/`. For example, from within `anvil/`, the stablehlo source is at `../stablehlo/`.

## Common Development Commands

All R packages (anvil, stablehlo, pjrt, tengen, xlamisc) use standard R tooling:

```r
devtools::load_all()    # Load for development
devtools::test()        # Run tests
devtools::document()    # Generate roxygen2 docs
devtools::check()       # CRAN compliance checks
```
