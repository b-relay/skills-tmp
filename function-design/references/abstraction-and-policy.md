# Abstraction and policy

Use this reference when the code lacks a useful boundary, a function mixes conceptual levels, or several functions repeat one policy.

## Write the call you want to exist

When the caller's next operation is clear but the implementation is not, sketch the desired call first.

```text
config = load_config(config_path())
```

The missing functions reveal missing boundaries. The sketch is not a finished contract. It still needs types, ownership, failure, and effect decisions.

A function is worth considering on its first coherent occurrence. Extraction does not need to wait for duplication. Extract when the name creates useful vocabulary, isolates a contract, or creates a test seam. Keep code inline when a helper only renames obvious syntax.

Pair extraction with pruning. A helper that was useful during discovery may become a pass-through wrapper after a standard algorithm or a better data abstraction owns the work.

**Check:** Does the boundary reduce what the caller or implementer must hold in mind?

## Keep peer operations at one conceptual depth

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

- arithmetic belongs in a low-level numeric helper but interrupts a high-level orchestration function;
- a loop is appropriate inside the operation whose job is iteration;
- a section comment may reveal a missing boundary, or it may explain a necessary invariant; and
- calling a trusted algorithm and interpreting its result can remain one coherent operation.

Loops, long bodies, and section comments are review signals, not automatic violations.

**Check:** Can every top-level statement be described as a peer step in the function's name?

## Prefer a trusted algorithm to hand-written machinery

After extraction reveals a standard operation, use the project's trusted library implementation when its contract fits. This removes loop state, boundary conditions, and maintenance burden from the local function.

Keep result interpretation beside the algorithm when both belong to the same conceptual operation. Do not create one custom helper per line merely to make the body look uniform.

**Check:** Does custom control flow still encode domain policy, or is it recreating a tested general algorithm?

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

Then inspect sibling operations. If insertion and lookup both repeat case normalization and ordering, the missing abstraction is not another helper. It is a case-insensitive index that owns normalization, storage, insertion, and lookup.

```text
function is_record_of_kind(name, kind):
    record = records_by_name.find(name)
    return record exists and record.kind == kind
```

Once the index owns the policy, a transitional `find_record` wrapper may add no vocabulary or contract and should be deleted. The index needs its own contract tests for normalization, insertion, misses, text policy, and lifetime. Those tests should remain valid if its storage changes from sorted data to hashing.

This refactor sequence matters:

1. Outline behavior to expose mixed levels and hidden outcomes.
2. Extract a coherent operation so callers stop carrying its mechanics.
3. Replace hand-written general machinery with a trusted algorithm.
4. Search sibling producers and consumers for repeated policy.
5. Move the policy to one owner.
6. Remove wrappers made redundant by the deeper owner.

**Check:** Can normalization, ordering, lookup, or validation policy change in exactly one implementation without coordinated edits?

## Preserve caller-visible behavior during refactoring

Before changing a boundary, identify callers that depend on mutation, ordering, error distinctions, allocation, timing, or borrowed lifetime. A cleaner local function is not an improvement if it silently changes a relied-on contract.

When behavior must change, make that change explicit in the task, update callers, and test the new contract. When behavior must stay stable, add characterization tests before moving the boundary if existing tests do not protect it.

**Check:** Do tests prove the behavior the task intends to preserve, including non-return-value channels?

