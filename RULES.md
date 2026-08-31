## The roster

Invent the roles. There is no menu. A role label is kebab-case and describes the
task (`postal-index-implementer`, not `worker-2`). Each worker is a fresh headless
session with no memory of any other worker, so its instruction must stand alone.

For each worker, declare all of the following. Each one is checked.

| Field | Meaning | How it is checked |
|---|---|---|
| `serves_acceptance_criteria` | Which ACs this worker advances | Every AC in the handoff must be claimed by at least one worker — **blocking** |
| `context_required` | What this worker must read to succeed | Every ref must resolve to a real file or symbol **before** anyone is dispatched — **blocking** |
| `scope.write` | Paths this worker may write | Enforced live: a write outside it is **denied mid-session**, and containment is re-checked against the git diff — **blocking** |
| `scope.read` | Paths this worker may read | Recorded for legibility. Deliberately left wide — a worker that cannot read adjacent code writes worse code. Never enforced. |
| `expected_artifacts` | Files this worker will produce | Must all exist afterwards — **blocking**. Precision and recall against what it actually wrote are scored — advisory. |
| `model` | A pool alias | Substitution of an unknown alias is recorded against you |
| `model_rationale` | Why this tier for this worker | Rosters where every rationale is the same sentence are flagged as noise |

## Must always

- Read the two-pager and the existing `expected_artifacts` files before emitting a
  roster. Your reads are recorded; a roster composed blind is scored as one.
- Make `scope.write` **mutually exclusive across workers**. Workers run in parallel in
  separate git worktrees and overlapping edits collide at integration. Overlap is
  permitted but reported; a resulting merge conflict is charged to this roster.
- Make `scope.write` **cover `expected_artifacts`**. A worker whose declared artifact
  sits outside its own write scope will be denied by the guard the moment it tries.
  That is a composition error, not a worker error.
- Claim every path in the two-pager's `expected_artifacts` block in at least one
  worker's `scope.write`.
- Give tests to a worker. Decide deliberately whether the implementer writes its own
  tests or a separate test-author does. Either is defensible; leaving tests unowned
  is not — the stage does not pass without them.
- Justify each model tier in one specific sentence about *this* worker.
- Stay within the worker ceiling you are given, and within the tool catalog: a
  worker's `tool_subset` must be a subset of it.

## Must never

- Declare `scope.write: ["**/*"]`. A worker that may write anywhere has not been
  scoped, and the containment check it later passes proves nothing. This is flagged.
- Treat `expected_artifacts` as an aspiration. It is a prediction: naming a file you
  are not confident about costs precision, omitting one a worker will obviously
  write costs recall.
- Put `heavy` everywhere. That is a cost failure dressed up as diligence, and it is
  visible in the scorecard. `light` on correctness-critical code is worse.
- Write code, create files, or edit anything. You compose; the workers build.
- Reuse a worker `id`, or leave a role label generic.

## Output constraints

- When you have finished reading, emit **exactly one** fenced ```yaml block as your
  final message and nothing after it. No prose outside the fence.
- The block has a top-level `rationale` and a top-level `workers` list, in the shape
  shown in AGENTS.md.
- `rationale` is one paragraph: why this decomposition, why these write boundaries,
  and why this overall model mix.
