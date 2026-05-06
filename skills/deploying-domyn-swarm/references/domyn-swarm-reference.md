# Domyn Swarm Reference

## CLI Location

- Internal repo: `/Users/federico.dambrosio@igenius.ai/Repos/domyn-swarm`
- Local dev command: `/Users/federico.dambrosio@igenius.ai/Repos/domyn-swarm/.venv/bin/domyn-swarm`
- Package entrypoint: `domyn-swarm`

Running most CLI commands triggers the root callback, which may initialize or migrate the local state DB unless `DOMYN_SWARM_SKIP_DB_UPGRADE` is set.

## Main Commands

```bash
domyn-swarm version
domyn-swarm up -c config.yaml
domyn-swarm up -c config.yaml --replicas 3 --reverse-proxy
domyn-swarm status <swarm-name>
domyn-swarm swarm list
domyn-swarm swarm list --no-probe
domyn-swarm swarm describe <swarm-name> -o yaml
domyn-swarm down <swarm-name> -y
domyn-swarm down -c config.yaml --select
domyn-swarm down -c config.yaml --all -y
domyn-swarm init defaults
```

`domyn-swarm up` submits an allocation. For Slurm this includes replica jobs and a load balancer job, and records state in the local SQLite DB.

## Slurm Config Example

```yaml
model: Qwen/Qwen3-30B-A3B-Thinking-2507
name: qwen3-30b
revision: main
gpus_per_replica: 8
gpus_per_node: 4
replicas: 1
image: /shared/images/vllm.sif
args: >-
  --dtype bfloat16
  --gpu-memory-utilization 0.90
  --max-model-len 32768
  --max-num-seqs 64
env:
  HF_HOME: /shared/hf_home
backend:
  type: slurm
  partition: partition_name
  account: account_name
  qos: qos_name
  modules:
    - cuda/12.2
  endpoint:
    nginx_image: /shared/images/nginx.sif
    port: 9001
    wall_time: "36:00:00"
    nginx_timeout: "8h"
```

## Lepton Config Example

```yaml
model: deepseek-ai/DeepSeek-R1-Distill-Qwen-32B
name: deepseek-r1
revision: main
replicas: 1
wait_endpoint_s: 1800
image: vllm/vllm-openai
args: "--reasoning-parser deepseek_r1 --gpu-memory-utilization 0.95"
port: 8080
backend:
  type: lepton
  workspace_id: workspace-xxxxxx
  endpoint:
    image: vllm/vllm-openai
    allowed_dedicated_node_groups:
      - nodegroup-xxxxxx
    resource_shape: gpu.4xh200
    env:
      HF_HOME: /mnt/lepton-shared-fs/hf_home/
    image_pull_secrets:
      - my-lepton-secret
  job:
    allowed_dedicated_node_groups:
      - nodegroup-xxxxxx
    image: igeniusai/domyn-swarm:dev-main
    image_pull_secrets:
      - my-lepton-secret
```

## Core Config Fields

| Field | Notes |
| --- | --- |
| `model` | Required HF model ID or local path passed to vLLM. |
| `name` | Required; lowercased, stripped, max 38 chars. |
| `revision` | Optional HF revision. |
| `replicas` | Independent vLLM clusters. |
| `gpus_per_replica` | GPUs used by each vLLM replica; sets tensor parallel size. |
| `gpus_per_node` | GPUs available per node, currently validated `1..4`. |
| `nodes` | Auto-computed unless explicitly set. |
| `replicas_per_node` | Auto-computed when a node can host multiple replicas. |
| `cpus_per_task` | Auto-computed if omitted. |
| `image` | Slurm Singularity image or Lepton container image. |
| `args` | Extra flags passed to vLLM. |
| `port` | vLLM server port, default `8000`. |
| `env` | Environment variables applied to the deployment. |
| `wait_endpoint_s` | Endpoint readiness wait, default `1200`. |

## Slurm Backend Fields

| Field | Notes |
| --- | --- |
| `backend.type` | Must be `slurm`. |
| `backend.partition` | Slurm partition; required unless defaults provide it. |
| `backend.account` | Slurm account; required unless defaults provide it. |
| `backend.qos` | Slurm QoS; required unless defaults provide it. |
| `backend.time_limit` | Allocation wall clock, default `36:00:00`. |
| `backend.modules` | Modules loaded by Slurm scripts. |
| `backend.exclude_nodes` | Slurm exclude list, for example `node[001-004]`. |
| `backend.node_list` | Explicit node list. |
| `backend.endpoint.nginx_image` | Required load balancer image unless defaults provide it. |
| `backend.endpoint.port` | Public load balancer port, default `9000`. |
| `backend.endpoint.wall_time` | LB/driver wall time, default `24:00:00`. |
| `backend.endpoint.nginx_timeout` | NGINX request timeout, default `60s`. |

## Job Commands

```bash
domyn-swarm job submit \
  --name my-swarm \
  --input examples/data/chat_completion.parquet \
  --output results.parquet \
  --job-kwargs '{"temperature":0.3}' \
  --checkpoint-interval 16 \
  --max-concurrency 8 \
  --retries 2

domyn-swarm job submit-script \
  --name my-swarm \
  path/to/script.py -- --foo 1 --bar 2
```

For `job submit`, use exactly one of `--config` or `--name`. Ray jobs require `--data-backend ray`, an `--id-column`, native backend execution, and `--ray-address` or `DOMYN_SWARM_RAY_ADDRESS` / `RAY_ADDRESS`.
