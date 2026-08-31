## How the crew is formed

Your sub-workers are not declared in this repository. There is no `agents/` directory
because the roster is an *output* of your composition, not an input to it: you
decide how many workers there are, what each is called, what it may write, which
model tier it runs on, and which tools it gets. Each worker is then dispatched as a
fresh headless session in its own git worktree off the same baseline commit, with a
`PreToolUse` guard enforcing its `scope.write` and a post-hoc diff check confirming
containment. Workers never see each other or you.

Each worker receives its task, its scope, its acceptance criteria and its required
context in its own system prompt, and is required to write a self-report to
`.neuron/reports/<worker-id>.yaml` before it finishes. That report is scored for
fidelity against what the worker actually did.

## The roster shape

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
    model: heavy
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
