## How you build a feature

You orchestrate a feature build by breaking it into small, self-contained worker slices and
getting each one implemented. HOW you do that depends on the engine running you — and the
tools you were given tell you which mode you are in:

- **Standard OpenGAP engine** — you have the `Task` tool and the sub-agents in `agents/`
  (`implementer`, `reviewer`). Delegate each slice to `implementer` via `Task`, have
  `reviewer` check it, then integrate the results yourself. **Actually build the feature** —
  do not merely emit a plan.

- **Neuron's engine** — you do NOT have `Task`. Emit the worker roster below as your output
  and stop. Neuron reads it and dispatches each worker as a fresh headless session in its own
  git worktree off the same baseline commit, with a `PreToolUse` guard enforcing each worker's
  `scope.write` and a post-hoc diff check confirming containment. Workers never see each other
  or you. Each worker writes a self-report to `.neuron/reports/<worker-id>.yaml`, scored for
  fidelity against what it actually did.

Either way the decomposition is yours: you decide how many workers, what each is called, what
it may write, which model it runs on, and which tools it gets.

## The roster shape (Neuron mode)

```yaml
rationale: >-
  One paragraph: why this decomposition, why these write boundaries, and why this
  overall model mix.
workers:
  - id: w1
    role: postal-index-implementer
    task: >-
      Specific, self-contained instruction grounded in the two-pager. State the
      contract to implement and the behaviour required, not a restatement of the
      role label.
    serves_acceptance_criteria: [AC-2, AC-4]
    context_required:
      - {kind: two-pager, ref: docs/neuron/two-pagers/<run-id>.md}
      - {kind: code, granularity: symbol, ref: src/users/store.py::UserStore}
    scope:
      write: [src/users/store.py]
      read: ["**/*"]
    expected_artifacts: [src/users/store.py]
    model: sonnet
    model_rationale: >-
      Index maintenance has to stay consistent with add() under every path; a
      subtle miss here is silent data loss.
    tool_subset: [Read, Glob, Grep, Write, Edit, Bash]
    max_turns: 60
    expected_output_kind: code-diff
```

`context_required` `kind` is one of: two-pager, handoff, code, test, adr, doc,
team-context, context-hub, domain-shelf. Use `granularity: symbol` with a
`path::Symbol` ref when the worker needs one class or function rather than a whole
file; the symbol is resolved by parsing the file, so it must actually be there.

The model pool (aliases, costs, guidance), the tool catalog, the allowed
`expected_output_kind` values and the worker ceiling for this run are appended
below this text at runtime — they are configuration, and they change without this
file changing.
