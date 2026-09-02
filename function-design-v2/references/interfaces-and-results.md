# Interfaces and results

Use this reference when a function's call shape, parameter reach, result representation, failure model, or proof obligations are unclear.

## Design from a real call site

Design the parameter list from the values the call site already holds, so the caller builds nothing unrelated and learns no storage mechanism that only the callee cares about.

### Replace mystery arguments

```text
set_timer(null, "refresh", 5, true, false, true)
```

The booleans' roles are invisible, and swapping two of them still compiles.

```text
set_timer({
    context: null,
    id: "refresh",
    interval_seconds: 5,
    delay_first_run: true,
    repeat: false,
    restart_if_active: true
})
```

Use named fields when names are enough. Use strong types when swapping two values must fail to compile. Both forms cost something. Wrappers add boilerplate, and either form can copy values into and out of the call, so check that cost at the intended call sites.

A request object is not the same as passing an existing broad object because it happens to contain the needed fields. The broad object grants access to unrelated fields, couples the callee to one representation, and rejects callers that hold the same values elsewhere.

## Require exactly the operations the body performs

If a body only iterates, a concrete resizable array grants and demands more than needed. If it needs contiguity, an arbitrary iterable is too weak. If it retains data, a borrowed view may be unsafe.

Choose the contract that proves every body operation valid and admits every real caller:

- Iterable when the body only iterates.
- Contiguous view when the body needs pointer-length access or contiguous storage.
- Mutable view when the body changes caller-owned elements.
- Owned value when the function retains or transfers data.
- Request object when several related values form one call contract.

The rule runs in both directions. A concrete container can reject valid callers when the body only iterates, and an unconstrained generic can admit inputs that lack an operation the body needs. Keep a concrete type when a real caller supplies it and contiguity, ownership, performance, or language ergonomics favor it. Every borrowed view or returned reference needs a stated owner, lifetime, and invalidation rule.

Revisit the parameter after a deeper owner takes over policy. A lookup that took an owned string because it lowercased a copy can take a read-only view once a case-insensitive map owns normalization. The old restriction was implementation-driven, and it disappears with the implementation.

## Choose value, visitor, or iterator from the output bound

Returning a collection gives callers a value they can inspect, reuse, and test. It also materializes the whole result. A visitor or iterator supports large or incremental output. It also adds contract questions: how long the visited value lives, which effects the callback brings, whether the caller can stop early, and what remains visible after a failure.

```text
function all_permutations(text):
    results = []
    for each permutation of text:
        results.append(copy(permutation))
    return results
```

This is right only when the output bound is acceptable. Permutations of distinct characters number `n!`.

```text
function for_each_permutation(text, consume):
    for each permutation of text:
        consume(read_only_borrow(permutation))
```

The producer owns its working state. The callback reads the visited value and writes only what it owns. A consumer that keeps a borrowed value must copy it. The composed call inherits the callback's effects and failures.

Layer the simple API over the general traversal when both are useful. Keep one traversal implementation.

## Preserve failure information callers need

A boolean may collapse distinct states such as "missing" and "present with the wrong kind." Keep the boolean only when every caller treats those states the same way. Otherwise return a result, an option plus reason, a tagged union, an exception, or the project's error type, whichever preserves the distinction.

Put failure policy where domain context exists. A low-level normalization helper should not decide whether a zero vector means skip, substitute, log, assert, or fail the request. Never let an invalid input flow through as infinities or NaNs, because the failure then surfaces far from its cause. Use an assertion when invalid input is a programmer error and the project accepts assertions. Return an explicit error when invalid data can arrive in normal operation. Carry the invariant in a type when one check establishes the fact and several boundaries rely on it.

## Carry reusable prerequisites in proof values

When operation B is valid only after A succeeds, A can return a proof value and B can require it.

```text
proof = initialize_resource()
use_resource(proof)
```

The proof works only when callers cannot forge it and the owner creates it after success.

A lock guard proves only what its type and value encode. Requiring the guard as a parameter catches one real mistake. In C++ an unnamed guard is a temporary, and a temporary dies at the end of its full expression, so a guard created unnamed on its own line releases the lock before the next statement runs. Naming the guard and passing it, or constructing it inside the call expression, keeps it alive through the call. A generic `Lock<Mutex>` still proves presence and lifetime without proving that it protects the exact resource. Encode resource identity with an owner-created guard, a distinct guard type, or an API reachable only through the correct protected object.

Use proof types for repeated, meaningful prerequisites. Straight-line control flow is clearer for a rare local ordering rule.

## Carry true invariants in strong types

A strong type can establish a fact once and let many consumers rely on it.

```text
class NormalizedVector:
    private value

    static create(vector):
        magnitude = length(vector)
        if magnitude is non_finite or near_zero:
            return error
        return NormalizedVector(vector / magnitude)

    implicit conversion to vector:
        return read_only value
```

The type needs controlled construction, explicit failure, invariant-preserving operations, and free weakening to a read-only plain value. Let the strong value pass wherever the plain value is accepted, with no checked step, since forgetting the fact is safe. The reverse direction always goes through `create`. Public mutation or inheritance from a mutable base can break the guarantee. Test construction rejection and every operation that produces or changes the strong type. A consumer such as `reflect(ray, NormalizedVector normal)` can then skip its own check, and an overload such as `normalize(NormalizedVector)` can return its input without recomputing.

Encode only guarantees that are true for every value the type admits. A tempting claim is that the cross product of two normalized vectors returns a normalized vector. That is false. Two parallel unit vectors have a zero cross product, and any pair not at right angles gives a non-unit result, so the honest return type is a plain vector or a fallible normalized one. Before adding a postcondition to a type, state the invariant exactly, prove it over the full input set, and test the boundary counterexamples. An unproved postcondition makes the API more dangerous, not safer.

Keep rare local checks local when a new type would create more code than protection.
