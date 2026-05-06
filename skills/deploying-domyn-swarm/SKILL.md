---
name: deploying-domyn-swarm
description: Use when deploying, validating, monitoring, or tearing down vLLM or LLM serving clusters with the internal domyn-swarm CLI and YAML configs for Slurm or Lepton backends
---

# Deploying Domyn Swarm

## Overview

Domyn Swarm deploys OpenAI-compatible vLLM endpoints from a YAML config, with Slurm and Lepton backends. Treat `domyn-swarm up` as a real deployment command: it can submit jobs and create or migrate local state.

## Required Context

Before writing commands or YAML:

1. Read the target config or ask for deployment constraints: backend, model path/HF ID, GPU topology, image paths, partition/account/QoS or Lepton workspace.
2. Validate config statically with Python; do not use `up` as validation.
3. Include explicit values for required site-specific fields. Do not invent partition, account, QoS, image, node group, or secret names.

## Static Validation

Use this from `/Users/federico.dambrosio@igenius.ai/Repos/domyn-swarm` or with that repo on `PYTHONPATH`:

```bash
PYTHONPATH=src .venv/bin/python - <<'PY' config.yaml
import sys
from domyn_swarm.config.swarm import DomynLLMSwarmConfig

cfg = DomynLLMSwarmConfig.read(sys.argv[1])
cfg.build_plan()
print(
    f"OK name={cfg.name} nodes={cfg.nodes} "
    f"replicas_per_node={cfg.replicas_per_node} ray={cfg.watchdog.ray.enabled}"
)
PY
```

## Common Workflow

| Task | Command |
| --- | --- |
| Create interactive defaults | `domyn-swarm init defaults` |
| Deploy | `domyn-swarm up -c config.yaml` |
| Override replicas | `domyn-swarm up -c config.yaml --replicas 3` |
| List swarms | `domyn-swarm swarm list` or `domyn-swarm swarm list --no-probe` |
| Runtime status | `domyn-swarm status <swarm-name>` |
| Static state | `domyn-swarm swarm describe <swarm-name> -o yaml` |
| Tear down | `domyn-swarm down <swarm-name> -y` |
| Tear down by config name | `domyn-swarm down -c config.yaml --select` or `--all` |

## YAML Minimums

Slurm configs need `model`, `name`, GPU sizing, `image`, and `backend.type: slurm` with partition/account/QoS plus `backend.endpoint.nginx_image`. Multi-node vLLM is inferred when `gpus_per_replica > gpus_per_node`; `gpus_per_replica` must then be a multiple of `gpus_per_node`.

Lepton configs need `model`, `name`, `image`, `backend.type: lepton`, `workspace_id`, and endpoint/job resource settings appropriate for the target workspace.

For command details, examples, and config fields, read [domyn-swarm-reference.md](references/domyn-swarm-reference.md).

## Common Mistakes

- Omitting `name`: current `DomynLLMSwarmConfig` requires it even if older quickstarts omit it.
- Running `domyn-swarm --help` casually in a dirty environment: the root callback may initialize or migrate the swarm DB.
- Treating `replicas` as tensor parallelism: `gpus_per_replica` drives vLLM `--tensor-parallel-size`; `replicas` launches independent clusters.
- Forgetting shared paths on Slurm: `.sif` images and `HF_HOME` must be readable by compute nodes.
- Passing both `--config` and `--name` to `job submit`: use exactly one.
