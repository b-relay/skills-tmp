---
name: function-design
description: Improve function boundaries while implementing, reviewing, or refactoring code. Use when functions hide state or effects, resist isolated tests, take broad or unclear parameters, return the wrong result shape, mix conceptual levels, repeat policy, or need extraction. Skip formatting-only edits and architecture work with no function-level decision.
---

# Function design

Improve the functions in scope without changing unrelated behavior. Treat a function as a contract boundary for limited reasoners: callers should understand the operation without reading its body, and implementers should understand the body without searching the whole codebase.

Follow repository guidance and the user's requested scope. Preserve public behavior unless the task authorizes a behavior change. A review request authorizes findings, not edits.

## Review a function in eight steps

### 1. Inspect the real use

Read the function, its direct callers, its tests, and the contracts of direct callees. State the caller's job in one sentence. Sketch the call using values already available at a real call site and a result the caller can use next.

**Complete when:** a reviewer can explain the desired call without opening the body, and the sentence contains no accidental implementation mechanism.

### 2. Build a contract ledger

Record:

| Field | Include |
| --- | --- |
| Inputs | Parameters, receiver state, caller-supplied state, and capabilities read |
| Outputs | Return values, errors, changed arguments or receiver, callback calls, and external effects |
| Ambient access | Globals, clocks, environment, files, network, devices, caches, and hidden manager state |
| Prerequisites | Ordering, locking, normalization, ownership, lifetime, and required capabilities |
| Failure | Invalid input, expected absence, partial progress, and the component that decides the response |

For each body operation and direct call, record every behavior-affecting read, write, prerequisite, effect, and failure. Trace deeper only when a callee has no usable contract or its implementation contradicts that contract.

**Complete when:** every behavior-affecting fact appears in the ledger, and no access or effect hides behind a name such as `helper`, `manager`, or `accessor`.

### 3. Make the boundary honest

Expose stable values and deterministic mutable state through parameters or receiver state. Pass a capability when the caller must choose the provider, then retain that provider's reads, writes, failures, and effects in the composed contract. Put acquisition of time, random seeds, files, network data, and other external inputs in a named integration function when the calculation should not own it.

**Complete when:** every ambient access is caller-controlled or deliberately owned by a named integration boundary, and every passed capability's effects remain in the ledger.

### 4. Ask for the exact contract

Check names, positional meaning, authority, ownership, lifetime, container requirements, and value invariants. Replace mystery booleans with named fields or meaningful types. Pass a purpose-built request instead of an unrelated domain object. Require only the capabilities the body uses.

**Complete when:** a reviewer can name each argument's role from the call expression, each restriction maps to a body operation, and the callee receives no unrelated authority.

### 5. Choose result and failure channels

Choose one or more channels that preserve what callers need: a returned value or error, mutation of caller-owned state, a visitor or iterator, and any deliberate outside effect. Account for output size, ownership, borrowed lifetime, early termination, partial progress, and distinctions callers need between negative outcomes. Put failure policy where enough domain context exists to choose it.

**Complete when:** the contract states memory and lifetime behavior, preserves every negative outcome callers use, and names one owner for each failure decision.

### 6. Align conceptual depth

Replace every executable block with a short action phrase. Indent mechanics beneath the action they implement. Keep peer actions together. Move a child procedure into a named function, a trusted algorithm, or a type that owns the operation.

**Complete when:** every executable block maps to one action phrase and all top-level phrases are peer steps in the function's stated job.

### 7. Give policy one owner

Inspect sibling functions in the same owner and direct call sites that construct or consume the same values. Find repeated normalization, ordering, validation, seeding, locking, key construction, or storage policy. Move a shared invariant to one owner. Remove transitional helpers that become pass-through calls after the deeper abstraction exists.

**Complete when:** each shared policy changes in one implementation and every remaining wrapper owns vocabulary, policy, compatibility, or a distinct contract.

### 8. Implement and verify the boundary

Make the smallest coherent change. Test ordinary values, boundaries, rejected inputs, changed state, every result channel, errors, lifetime-sensitive behavior, and partial progress. Inspect at least one real call site after the edit. Use broader test selection unless tooling proves a narrower dependency set.

**Complete when:** tests control every behavior-affecting input, assert every output channel callers use, and establish the lower-level contracts marked untrusted in step 2.

## Read the matching references

Read every reference whose trigger matches the function. A function may match more than one.

- For globals, clocks, I/O, mutable receivers, callbacks, framework hooks, hidden manager access, transactions, or deterministic state, read [contracts and effects](references/contracts-and-effects.md).
- For unclear positional arguments, broad objects, container constraints, large results, visitors, iterators, borrowed values, errors, proof tokens, locks, or invariant types, read [interfaces and results](references/interfaces-and-results.md).
- For missing boundaries, long bodies, section comments, raw loops, mixed conceptual levels, wrapper chains, or repeated policy, read [abstraction and policy](references/abstraction-and-policy.md).

## Report the outcome

For implementation work, state which contracts changed, which behavior stayed stable, and what verification passed. For review-only work, report prioritized findings with file and line evidence, the consequence, and a concrete correction. Separate required correctness fixes from worthwhile design improvements.

