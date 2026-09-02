# Interfaces and results

Use this reference when a function's call shape, parameter authority, result representation, failure model, or proof obligations are unclear.

## Design from a real call site

Write the call the caller should be able to make. Use values already present at that site. Avoid forcing the caller to construct an unrelated domain object or know a storage mechanism only the callee cares about.

### Replace mystery arguments

```text
set_timer(null, "refresh", 5, true, false, true)
```

The booleans' roles are invisible and can be swapped without a type error.

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

Use named fields when names are enough. Use strong types when values can be confused, require validation, or carry behavior. Do not create wrappers whose only effect is more syntax.

**Check:** Can a reviewer name every argument's role from the call expression?

## Require exact capabilities

If a body only iterates, a concrete resizable array grants and demands more than needed. If it needs contiguity, an arbitrary iterable is too weak. If it retains data, a borrowed view may be unsafe.

Choose the narrowest contract that proves every body operation valid:

- iterable when the body only iterates;
- contiguous view when the body needs pointer-length access or contiguous storage;
- mutable view when the body changes caller-owned elements;
- owned value when the function retains or transfers data; and
- purpose-built request when several related values form one call contract.

The target is exact capability, not maximum genericity. Every non-owning view needs a stated lifetime.

**Check:** For each type restriction, point to the body operation that requires it.

## Choose value, visitor, or iterator from the output bound

Returning a collection gives callers a simple value they can inspect, reuse, and test. It also materializes the whole result. A visitor or iterator supports large or incremental output but adds a lifetime, effect, early-stop, failure, and partial-progress contract.

```text
function all_permutations(text):
    results = []
    for each permutation of text:
        results.append(copy(permutation))
    return results
```

This is appropriate only when the output bound is acceptable.

```text
function for_each_permutation(text, consume):
    for each permutation of text:
        consume(read_only_borrow(permutation))
```

The callback must not mutate the producer's working state. A consumer that retains a borrowed value must copy it. The composed call inherits the callback's effects and failures. An iterator may be better when the caller needs pull-based control or early termination.

Layer the simple API over the general traversal when both are useful. Keep one traversal implementation.

**Check:** Does the contract state output size, borrowing, retention, early stop, failure, and partial progress?

## Preserve failure information callers need

A boolean may collapse distinct states such as "missing" and "present with the wrong kind." Keep the boolean only when every caller treats those states identically. Otherwise return a result, option plus reason, tagged union, exception, or project-standard error type that preserves the distinction.

Put failure policy where domain context exists. A low-level normalization helper should not invent whether a zero vector means skip, substitute, log, assert, or fail the request.

**Check:** Can each caller distinguish every negative outcome that changes its next action?

## Carry reusable prerequisites in proof values

When operation B is valid only after A succeeds, A can return a proof value and B can require it.

```text
proof = initialize_resource()
use_resource(proof)
```

The proof works only when callers cannot forge it and the owner mints it after success. Construction proves history, not freshness. If later operations can invalidate the prerequisite, tie the proof to the resource lifetime or generation, or expose B only through an object that remains valid.

A lock guard proves only what its type and value encode. A generic `Lock<Mutex>` may prove presence and lifetime without proving that it protects the exact resource. Encode resource identity with an owner-minted guard, a distinct guard type, or an API available only through the correct protected object.

Use proof types for repeated, meaningful prerequisites. Straight-line control flow is clearer for a rare local ordering rule.

**Check:** Is construction controlled, is the proof fresh, and does it identify the exact resource or fact the consumer relies on?

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

    as_vector():
        return read_only value
```

The type needs controlled construction, explicit failure, invariant-preserving operations, and safe weakening to a less specific read-only value. Public mutation or inheritance can break the guarantee. Test construction rejection and every operation that can produce or change the strong type.

Encode only guarantees that are true and useful. An unproved postcondition in a return type makes the API more dangerous, not safer. Keep rare local checks local when a new type would create more ceremony than protection.

**Check:** Can any public operation create an invalid instance or expose mutable storage that breaks the invariant?

## State borrowed lifetime and invalidation

References, pointers, iterators, views, and callback values borrow storage. State:

- who owns the storage;
- how long the borrow remains valid;
- which insertions, removals, moves, or mutations invalidate it; and
- whether callers need a stable handle or owned copy instead.

`const` or a read-only method prevents some mutation through that path. It does not create ownership or guarantee stable lifetime.

**Check:** Can the caller tell exactly when the returned or passed borrow becomes invalid?

