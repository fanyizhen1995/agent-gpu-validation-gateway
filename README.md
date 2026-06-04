# Agent GPU Validation Gateway

Agent GPU Validation Gateway is a prototype infrastructure project for giving AI coding agents, CI systems, and cloud workspaces safe, on-demand access to real GPU validation environments.

The goal is **not** to expose raw SSH access to GPU machines. The goal is to turn GPU testing, benchmarking, and profiling into a standardized tool that agents can call.

## Problem

Cloud coding agents such as Codex, Copilot coding agent, Cursor cloud agents, Jules, Claude Code, and Devin can clone repositories, edit files, run CPU tests, and open pull requests. But GPU software development still requires access to real hardware:

- CUDA / Triton kernel correctness
- PyTorch GPU smoke tests
- GPU-only runtime errors
- performance regression tests
- Nsight Compute / Nsight Systems profiling
- multi-GPU or hardware matrix validation

Without a standardized backend, the workflow often falls back to manual SSH, manual benchmark runs, and copy-pasting logs back into the agent.

## Vision

```text
Cloud Agent / CI / Developer
        |
        | REST API / CLI / MCP Tool / GitHub App
        v
Agent GPU Validation Gateway
        |
        | queue / quota / auth / baseline comparison
        v
GPU Validation Scheduler
        |
        | Kubernetes / Volcano / Kueue / DRA / custom GPU lease
        v
Isolated GPU Runner
        |
        | CUDA / PyTorch / Triton / ncu / nsys
        v
Structured Result
        |
        | JSON / Markdown / PR comment / agent-readable feedback
        v
Agent continues fixing or optimizing code
```

## MVP Scope

The MVP focuses on **GPU Test Runner** and **GPU Benchmark Runner**:

1. Submit a GPU validation job through CLI or REST API.
2. Run a command in a GPU-enabled container.
3. Capture stdout, stderr, exit code, metrics, and artifacts.
4. Return structured JSON result.
5. Support GitHub PR / CI integration later.

## Non-goals for MVP

- Transparent CUDA API remoting.
- Running untrusted multi-tenant workloads without sandbox hardening.
- NCCL-heavy distributed training.
- Long-running training platform.
- Replacing vLLM/SGLang inference services.

## Repository Layout

```text
docs/
  idea.md                    Product narrative and positioning
  architecture.md            System architecture
  mvp-roadmap.md             Implementation roadmap
  security-model.md          Security requirements

src/gpuval/
  cli.py                     Minimal CLI prototype

src/gateway/
  main.py                    Minimal HTTP gateway prototype

src/runner/
  job_runner.py              Local runner abstraction

k8s/
  gpu-validation-job.yaml    Kubernetes Job template

examples/
  gpuval.yaml                Example validation spec
  result.schema.json         Example result schema
  mcp-tool-definition.json   Example MCP-style tool definition
```

## Quick Start

This is a scaffold. The first runnable version can start with local subprocess execution, then switch to Kubernetes Jobs.

```bash
python -m src.gpuval.cli run --spec examples/gpuval.yaml
python -m src.gateway.main
```

## Core Concept

A validation request describes:

```yaml
target:
  gpu_type: H100
  gpu_count: 1
  image: cuda12.4-pytorch2.5-triton3.1

source:
  repo: https://github.com/example/project
  ref: main

commands:
  test:
    - pytest tests/test_cuda.py
  benchmark:
    - python benchmarks/bench_attention.py

validation:
  timeout_seconds: 600
  baseline: main
  max_perf_regression_percent: 5
```

The backend returns:

```json
{
  "status": "passed",
  "correctness": "passed",
  "latency_ms": 0.83,
  "speedup_vs_baseline": 1.18,
  "max_memory_mb": 642,
  "environment": {
    "gpu": "H100",
    "cuda": "12.4",
    "pytorch": "2.5"
  }
}
```

## Product Positioning

A good one-line description:

> GPU CI and validation backend for AI coding agents.

A more detailed description:

> A standardized, secure, automated GPU validation backend that allows AI coding agents and CI systems to run CUDA/PyTorch/Triton tests, benchmarks, and profiling jobs on shared or BYO GPU pools.
