# Abstraction and policy

Use this reference when you need to sketch a call before its implementation exists, a function mixes conceptual levels, or several functions repeat one policy.

## Write the call you want to exist

When the caller's next operation is clear but the implementation is not, sketch the desired call first.

```text
config = load_config(config_path())
```

The missing functions reveal missing boundaries. The same move unblocks a hard step. If a particle update needs a reflected velocity and the formula is not at hand, write `velocity = find_bounce_vector(velocity, surface_normal)` and finish the update. The math gets its own function, its own tests, and its own time.

Consider a function on its first coherent occurrence. Extract when the name creates useful vocabulary, isolates a contract, or gives tests a direct entry point. Keep code inline when a helper only renames obvious syntax.

Pair extraction with pruning. A helper that was useful during discovery can become a pass-through wrapper once a trusted algorithm or a better data abstraction owns the work.

## Keep peer operations at one conceptual level

Outline the body before moving code. Replace every executable block with an action phrase and indent mechanics beneath the action they implement.

```text
Find record
  Normalize key
    Normalize each character
  Search index
    Find lower bound
    Check match

Compare record kind
```

`Find record` and `Compare record kind` are peer operations. Character traversal and binary-search mechanics are children of lookup.

One conceptual level is contextual:

- Arithmetic belongs in a low-level numeric helper but interrupts a high-level orchestration function.
- A loop is appropriate inside the operation whose job is iteration.
- A section comment may reveal a missing boundary, or it may explain a necessary invariant.
- Calling a trusted algorithm and interpreting its result can stay one coherent operation.

Loops, long bodies, and section comments are reasons to look, not automatic violations, and none of them moves a function to the full path.

## Prefer a trusted algorithm to hand-written machinery

Once extraction reveals a standard operation, use the trusted algorithm when its contract fits. This removes loop state, boundary conditions, and maintenance burden from the local function. Hand-written search loops hide off-by-one and stale-variable bugs that survive repeated reading.

Keep result interpretation beside the algorithm when both belong to the same conceptual operation. One custom helper per line is not the goal.

## Move repeated policy to one owner

Suppose lookup code lowercases a key, performs a binary search, exits the process on a miss, and compares a record kind. Insertion code separately lowercases keys and maintains sort order.

The initial mixed function owns too many mechanisms:

```text
function is_record_of_kind(name, kind):
    lowercase name character by character
    perform hand-written binary search
    terminate process when absent
    return found_record.kind == kind
```

First separate lookup from the predicate and replace the manual search with a trusted algorithm. Preserve absence as data instead of hiding a process exit.

```text
function is_record_of_kind(name, kind):
    record = find_record(name)
    if record is absent:
        return false
    return record.kind == kind
```

Returning `false` merges "missing" with "wrong kind." That is correct only when every caller wants one negative answer.

Then inspect sibling operations. If insertion and lookup both repeat case normalization and ordering, the missing abstraction is not another helper. It is a case-insensitive index that owns normalization, storage, insertion, and lookup.

```text
function is_record_of_kind(name, kind):
    record = records_by_name.find(name)
    return record exists and record.kind == kind
```

Once the index owns the policy, the transitional `find_record` wrapper adds no vocabulary or contract, so delete it. The index needs its own contract tests for normalization, insertion, misses, text policy, and lifetime. Those tests should stay valid if its storage changes from sorted data to hashing. Encapsulation moves the proof obligation. It does not remove it.
