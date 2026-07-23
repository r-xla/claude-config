# r-xla Ecosystem

## Overview

r-xla brings XLA-based machine learning compilation to R.
All packages are developed together as a cohesive ecosystem -- when implementing a feature that spans multiple packages, make changes across all of them in a single effort.
For example, adding a new operation may require changes in stablehlo (IR), pjrt (execution), anvl (user API), and tengen (generics).
They all share a common CLAUDE.md document (this file), but each have their own CLAUDE.md as well.
If you make changes to a repository, always ensure that you have read it's specific CLAUDE.md as well.

The packages form a layered stack:

```
anvl          (user-facing: JIT compilation + autodiff, like JAX for R)
  |
stablehlo      (IR layer: create and manipulate StableHLO programs)
  |
pjrt           (runtime: compile and execute on CPU/CUDA/Metal/TPU)
  |
tengen         (tensor generics: shape(), dtype(), device(), as_array())
xlamisc        (shared utilities: LRU cache, formatting helpers)
```

Supporting repos:

- **docker** -- Daily Docker images (CPU + CUDA) with anvl pre-installed.
- **benchmarks** -- Performance comparisons of anvl vs PyTorch/torch.

## Packages

### anvl

Code transformation framework for R (like JAX). Provides `nv_*` API functions and `nvl_*` primitives for JIT compilation (`nv_jit()`) and automatic differentiation. Uses 1-based indexing; delegates to stablehlo (0-based) for IR generation.

### stablehlo

Creates and transforms StableHLO programs (a portable ML computation representation). Operations follow the StableHLO spec (see SPEC.md). Uses 0-based indexing. The `Func` object uses reference semantics; other objects use value semantics.

### pjrt

R interface to PJRT (Pluggable Jit RunTime). Compiles stableHLO programs to hardware-specific executables and runs them. Manages devices, buffers, and async execution. Supports CPU, CUDA, and Metal backends.

### tengen

Defines S3 generics for tensor operations: `shape()`, `dtype()`, `device()`, `as_array()`, `ndims()`, `nelts()`. Also provides the `DataType` dtype enum (`as_dtype()`, `dtype_width()`, `is_dtype_*()`).

### xlamisc

Shared utility library: `LRUCache`, `seq0()`, `list_of()`, `shapevec_repr()`, `vec_repr()`, `format_bib()`. Depended on by the other packages for common operations.

### docker

Dockerfiles and CI for building daily images: `anvl-cpu`, `anvl-cuda`, and benchmark variants. Published to Docker Hub and GHCR.

### benchmarks

MLP training benchmarks comparing anvl, R torch, and PyTorch on CPU (single/multi-threaded). Uses Docker images for reproducible environments. Results published via Quarto website.

## Navigating Between Packages

All repos are assumed to be sibling directories under a common parent:

```
<parent>/
├── anvl/
├── stablehlo/
├── pjrt/
├── tengen/
├── xlamisc/
├── docker/
├── benchmarks/
└── claude-config/
```

To access a sibling package from any repo, use `../<package>/`. For example, from within `anvl/`, the stablehlo source is at `../stablehlo/`.

## Keeping Documentation in Sync

When making important structural design changes to a package, ensure that the "Design" section in the package's AGENTS.md is updated to reflect the new structure.

## Common Development Commands

All R packages use standard R tooling:

```r
devtools::load_all()    # Load for development
devtools::test()        # Run all tests
devtools::document()    # Generate roxygen2 docs
devtools::check()       # CRAN compliance checks
devtools::install()     # Install the package
devtools::build()       # Build tar.gz

# Run a specific test file
testthat::test_active_file("tests/testthat/test-foo.R")
```

## General

* Do not carry around unneeded baggage. Do not keep "legacy" interfaces around. When making a change, commit to it, remove old functions if you just wrote a new one that supersedes it etc. Our project currently lives in a vacuum, there is no outside code that depends on it.
* Our code does not explain how things used to be unless it's a clear bugfix / regression. Otherwise, no-one will see the old version of our code. No deprecate, only delete. 


### Formatting and Linting

Packages with a Makefile (anvl, stablehlo, pjrt) support `make format` to format code (uses `air` for R, `clang-format` for C++).

To check for linter errors, run `jarl check .` from the package root.

### Documentation Rules

* Never edit `.Rd` files manually. Instead, edit the corresponding roxygen2 comments (lines starting with `#'`) and run `devtools::document()` to re-generate the `.Rd` files.
* Never edit `README.md` directly -- it is generated from `README.Rmd`. Always edit `README.Rmd` and then run `devtools::build_readme()` to regenerate `README.md`.
* When adding a new S3 methods (such as `print.<Class-Name>`), always run `devtools::document()` afterwards to re-generate the NAMESPACE.
* Environment variables and options are documented in package-level documentation (typically `R/package.R`).
* Check the `man-roxygen/` directory for existing templates when writing documentation.  Use `@template` to avoid duplicating common parameter descriptions.
  Only create new templates for sections that will likely be re-used.
* Exported functions and objects should have the appropriate roxygen documentation.
* For functions, always document the return value (section `#' @return'`).
* Internal functions that are not very short or not very obvious should have a list a short documenting comment
* Do not write comments / documentation about how things used to be before a change. This is not helpful for readers, who will usually only see the present state of code and now know about the past state.

## Test Rules

* never call `set.seed()` in tests, use `withr::local_seed()` instead

## Style

* For length-1 vectors, don't use `c()`. For example, use `1L` instead of `c(1L)`.
* Only add comments for complex code.
* Never use `:::` to access a package's own internal objects. Within a package (including tests), non-exported functions and objects are already available directly -- just call them by name.
* Vectorize where possible. Prefer `vapply()` / `lapply()` / `Map()` / `mapply()` over `for` loops that build up a result, and avoid repeated `c(x, new)` / `list(..., new)` inside loops (quadratic growth). Loops are fine for side effects or when each iteration depends on the previous one.

## Pkgdown

When adding a new exported function, ensure it's in `_pkgdown.yml` file.

## Updating CLAUDE.md/.claude

When asked to make changes to CLAUDE.md (or create a new skill), there are two cases:

1. The rule is project-specific (e.g. only applies to the `pjrt` package).
   In this case, edit `pjrt/CLAUDE.md` or `pjrt/.claude/`
2. It's a general development guideline, then add it to `claude-config/CLAUDE.md` or `/claude-config/.claude`
