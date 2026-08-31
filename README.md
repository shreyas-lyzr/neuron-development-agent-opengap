# neuron-development-agent — OpenGAP agent

The **Neuron Development-stage orchestrator** as a GAP (GitAgentProtocol) agent: an
agent-as-git-repo. Identity is the four root files, read at runtime by
`computeragent`'s `GapIdentityLoader`:

| File | Role |
|---|---|
| `agent.yaml` | manifest — name, model alias (`heavy`), `tools: [Read, Glob, Grep]`, `runtime.max_turns` |
| `SOUL.md` | who the orchestrator is, what it receives, what to read first |
| `RULES.md` | the seven roster declarations + must-always / must-never |
| `AGENTS.md` | how the crew (workers) is formed at runtime |
| `prompts/` | `worker_system.md` (worker contract) + `orchestrator_reemit.md` (plan-time re-emit) |
| `config/` | `model_pool.yaml`, `assessors.yaml`, `tool_catalog.yaml` |

> The full measurement pipeline (compose → dispatch workers in isolated worktrees →
> integrate → evals → scorecard) lives in the `computer-agent-py`-based project
> `APP10460-neuron-development-agent`. This repo is the **portable GAP identity** of the
> orchestrator — the part that is "an agent = a repo".

## Run it against an AgentOS control plane

Connecting the agent to the console needs **no code changes — two env vars**:

```bash
export AGENTOS_DISCOVERY_URL="http://<agentos-host>/agentos/api/discovery"
export COMPUTERAGENT_HARNESS_TOKEN="cak_..."     # an AgentOS API key
export ANTHROPIC_API_KEY="sk-ant-..."            # host model creds (or Bedrock)
```

The orchestrator + every worker then appear in the console as separate agents
(`neuron-development-agent/orchestrator`, `neuron-development-agent/worker:<id>`), each
with its own cost / tokens / transcript.

## Guardrails (per-agent)

Attach an org guardrail as a **PreToolUse hook** (not `can_use_tool` — an entry in
`allowed_tools` or `permission_mode: bypassPermissions` auto-approves the tool before
`can_use_tool` is consulted). See `src/development_agent/guardrail.py` in the pipeline
project: opt-in via `NEURON_GUARDRAIL=1`, blocks a configured tool/bash set, and every
blocked call is denied per agent and visible in the console.

Neuron also ships a **native per-worker scope guard** (PreToolUse) that denies
out-of-scope writes — surfaced as `containment` (A20) in the scorecard.
