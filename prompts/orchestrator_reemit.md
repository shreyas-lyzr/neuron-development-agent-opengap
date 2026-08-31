Your roster was **rejected** before any worker was dispatched. Nothing has run and
nothing has been written, so this costs only a re-composition.

Below is the roster you emitted and every finding against it. Fix the findings and
emit a corrected roster in the same format.

## Rejected roster (attempt {attempt})

```yaml
{rejected_roster}
```

## Findings

{findings}

## How to respond

Re-emit **one complete roster**, not a patch. The corrected roster replaces the
rejected one wholesale; the rejected version stays in the append-only composition
record for audit, so you do not need to preserve it.

Fix the specific findings. Do not paper over them:

- **Unclaimed acceptance criteria** — assign each to a worker that can genuinely
  advance it. Bolting an AC onto an unrelated worker to clear the check produces a
  worker that fails its own task.
- **Unresolvable `context_required`** — the ref does not exist. Use `Glob` and
  `Grep` to find what the real path or symbol is, rather than deleting the entry:
  if the worker needed that context, it still needs it. If a symbol genuinely does
  not exist yet because *this stage* will create it, the ref belongs to the file
  that will contain it or should be dropped, not fabricated.
- **Uncovered two-pager artifacts** — some path the design says will change is not
  inside any worker's `scope.write`. Either give it to a worker or explain in
  `rationale` why the design's expectation is wrong. An unowned path is unwritten.
- **Scope overlap** — narrow the boundary or merge the two workers. Two workers
  editing one file in parallel is a merge conflict waiting to happen.

You have one attempt. A second rejection stops the stage for a human.
