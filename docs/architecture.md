# Architecture

## High-level architecture

```text
[Agent / CI / Developer]
        |
        | CLI / REST / MCP / GitHub App
        v
[Gateway]
        |
        | Auth, validation, queue, quota
        v
[Scheduler]
        |
        | Select GPU pool / GPU type / priority
        v
[Runner]
        |
        | Kubernetes Job / container / microVM
        v
[GPU Node]
        |
        | CUDA / PyTorch / Triton / profiler
        v
[Result Store]
        |
        | JSON / Markdown / artifacts
        v
[Agent Feedback]
```

## Components

### 1. Gateway

Responsibilities:

- Accept validation requests.
- Validate schema.
- Authenticate clients.
- Enforce project-level quota.
- Submit jobs to scheduler.
- Expose job status and result endpoints.

### 2. Scheduler

Responsibilities:

- Select GPU type and pool.
- Apply queueing and priority.
- Enforce timeout and budget.
- Integrate with Kubernetes, Volcano, Kueue, DRA, HAMi, or custom GPU lease.

### 3. Runner

Responsibilities:

- Prepare workspace.
- Pull repo or apply patch.
- Start container.
- Run test/benchmark/profile command.
- Collect stdout/stderr/artifacts.
- Capture environment metadata.
- Clean up process, workspace, and GPU context.

### 4. Validation Harness

Responsibilities:

- Correctness checks.
- Shape/dtype sweeps.
- Determinism checks.
- Latency and throughput measurements.
- Baseline comparison.
- Performance regression classification.

### 5. Profiler Adapter

Responsibilities:

- ncu integration.
- nsys integration.
- torch profiler integration.
- DCGM / nvidia-smi metrics collection.
- Agent-readable summary extraction.

### 6. Result Store

Responsibilities:

- Store result JSON.
- Store logs.
- Store profile artifacts.
- Generate Markdown report.
- Generate PR comments.
- Expose result API.

## Deployment modes

### Hosted GPU Pool

The platform provides GPU capacity.

```text
User agent -> gateway -> hosted K8s GPU cluster
```

### BYO GPU

The customer brings its own GPU cluster.

```text
User agent -> gateway -> customer GPU pool
```

This is important for teams that already own GPUs but need standardized validation automation.

## Security model

The secure baseline is:

- no direct SSH to GPU nodes;
- no long-lived credentials exposed to agents;
- job-level isolation;
- timeout and quota;
- limited network egress;
- ephemeral workspace;
- artifact scanning;
- GPU memory/process cleanup.
