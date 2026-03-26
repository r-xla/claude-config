# claude-config

Shared [Claude Code](https://docs.anthropic.com/en/docs/claude-code) configuration for the [r-xla](https://github.com/r-xla) organization.

r-xla brings XLA-based machine learning compilation to R. The packages form a layered stack:

```
anvil          User-facing: JIT compilation + autodiff (like JAX for R)
  |
stablehlo      IR layer: create and manipulate StableHLO programs
  |
pjrt           Runtime: compile and execute on CPU/CUDA/Metal/TPU
  |
tengen         Tensor generics: shape(), dtype(), device(), as_array()
xlamisc        Shared utilities: LRU cache, formatting helpers
```

This repo centralizes the Claude Code instructions and skills that are shared across all r-xla packages. Each package imports these shared instructions via its `CLAUDE.md` and adds package-specific context on top.

See [CONTRIBUTING.md](CONTRIBUTING.md) for setup instructions and how to structure per-repo configuration.
