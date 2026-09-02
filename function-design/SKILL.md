---
name: function-design
description: Function design with an honest, exact, testable contract. Use when writing a new function, for all greenfield code, when making a substantial change to a function's body, signature, effects, results, or failure handling, when reviewing code at the function level, or when refactoring a function that hides state or effects, resists an isolated test, takes unclear parameters, returns the wrong result shape, mixes conceptual levels, or repeats policy. Skip formatting-only edits and architecture work with no function-level decision.
---

# Function design

Functions exist to manage complexity for limited reasoners, human or agent. A well-designed function supports local reasoning, so a caller understands the operation without reading the body. It supports abstraction, so an implementer understands the body without searching the codebase. It supports testing, so a test controls the operation through its signature. Reuse is secondary, and one occurrence can justify a function. Three rules settle most decisions:

1. Build the system from honest functions and put contact with the outside world in an effect owner. A function is **honest** when its signature shows every behavior-affecting dependency and every observable channel, including authorized mutation, results, effects, and failures. A caller can then predict the call from the signature, and a test can substitute or control each one. Honesty is not purity. `sort(range)` is honest and `get_time()` is not. A passed clock or client keeps a function honest only while the provider's effects, blocking, and failures are part of the stated contract. A dishonest function still abstracts an implementation and can be reused, but a caller cannot predict it from the call alone, and an isolated test cannot control it through the signature. Effect owners such as `main` or a request handler are dishonest by design, so everything below them can stay honest.
2. Make the contract ask for what the body needs and a real caller can supply, and return every distinction the caller acts on.
3. Keep the top-level statements of a body at one conceptual level. Mechanics belong behind a named function, a trusted algorithm, or a type that owns the policy.

## Scope

Apply the procedure to every function you create, to every existing function whose body, signature, effects, results, or failure handling you change substantially, and to the direct callers of any contract you change. On a review task, every function the user pointed at is in scope, changed or not. Other existing functions are out of scope unless the task, repository guidance, or the user says to refactor them. If new code must call an existing function that fails a step, adapt around it within scope and name the inherited debt in the report. Ask the user, once per task, only when fixing the callee would materially change the requested outcome or its cost.

Preserve public behavior unless the task authorizes a change. A review request authorizes findings, not edits.

## Set the depth

Scan each function in scope once for boundary signals before the steps. Look at the body and at what each direct callee reads and writes beyond its parameters. The signals are:

- ambient access such as globals, clocks, environment, files, network, or hidden manager state
- mutation of caller-owned or receiver state
- a borrowed parameter or result
- a callback, strategy, or client parameter
- locking or concurrent use
- an outside effect
- a change to failure handling that a caller can observe
- a signature that other modules or external callers use
- a direct callee that reads ambient state, or that is neither tested nor documented

- **Plain path.** The scan finds no signal. Do steps 1, 4, 5, 6, 7, and 8, writing one sentence for each. Step 8 still adds the tests its sentence names. The scan already answers steps 2 and 3: there is no ambient entry and no effect to place. Step 5 stays, because a plain predicate can still merge two negative outcomes that a caller separates. A pure mapper, a small predicate, a test helper, or a pure format conversion is usually plain.
- **Full path.** The scan finds any signal. Do all eight steps and write the ledger.

Move a function to the full path the moment a step finds a signal the scan missed. On either path, a step produces a finding only with evidence: a real or likely caller the contract excludes, a mechanism the caller must learn, a distinction a caller acts on, a dependency the signature hides, or a policy a sibling repeats. A raw loop, a concrete container, a boolean, or a long body is a reason to look, not a finding by itself.

## Follow the eight steps

Writing and review use the same order. Each step names the reference to read when it finds or introduces a problem.

### 1. State the caller's job and the call

Read the call site, or the call site you are about to write, and state the caller's job in one sentence. Write the call using values present at that site and a result the caller uses next. When writing, keep the sketch even if the implementation is unclear. When reviewing or changing an existing function, write today's call next to the call that should exist, and read the tests that govern this function's contract.

When the implementation is unclear and you need to sketch the call first, read [abstraction and policy](references/abstraction-and-policy.md).

**Done when** the call names its inputs and its result in domain words, the sentence names no mechanism, and you have read the tests that govern this function's contract.

### 2. Build the ledger

Keep one ledger per full-path function in working notes and report only entries that yield a finding.

| Field | Include |
| --- | --- |
| Inputs | Parameters, receiver state, caller-supplied state, and providers such as clocks or clients |
| Outputs | Return values, errors, changed arguments or receiver, callback calls, and external effects |
| Ambient access | Globals, clocks, environment, files, network, devices, caches, hidden manager state, and any shared state the receiver reaches through a reference |
| Prerequisites | Ordering, locking, normalization, ownership, and lifetime |
| Failure | Invalid input, expected absence, partial progress, and the component that decides the response |
| Callees | Each direct callee, what it reads and writes beyond its parameters, and its trust mark: trusted when tested or documented, otherwise untrusted |

When writing, fill it as a plan and correct it after the body exists. On a review, build it from each body operation and direct call. When a callee's result depends on ambient state, read the callee and add what it touches. A parameterless getter whose filter asks a global manager what is loaded is the typical case. A lambda or local wrapper around the same read does not contain it. A callee's ambient access is this function's ambient access.

For a callee that reads ambient state, read [contracts and effects](references/contracts-and-effects.md).

**Done when** every ledger field has an entry, every ambient entry names the state it reaches, and every callee row records its reads, writes, and trust mark.

### 3. Make the boundary honest

For each ambient entry, move its acquisition to an effect owner, then pass what the owner acquired down as a parameter or receiver state: a value such as a timestamp, or deterministic state such as a seeded generator. Pass a provider such as a clock or client instead only when the caller must choose it at call time, and keep the provider's effects in the ledger.

The effect owner is the outermost function that has the context to own the effect and whose contract shows it: the process entry, request handler, command, or framework hook `update` that this function serves. A value or deterministic state may pass down through every layer between the owner and this function, since each signature on the way then shows a real dependency. A provider is different. When a client or other mechanism would pass through layers that never call it, stop at the nearest layer that has the context, such as an adapter or repository whose signature names the effect, and pass its results below that point. Threading a low-level client through layers that only forward it exposes a mechanism to callers that never use it.

For any ambient access, passed callable, framework hook, or seeded generator, read [contracts and effects](references/contracts-and-effects.md).

**Done when** every ambient entry is caller-controlled or lives in an effect owner whose contract shows it, every passed provider's effects remain in the ledger, no provider passes through a layer that never calls it, and swapping the outside channel, such as console for file, changes only the owner.

### 4. Ask for the exact contract

Check names, positional meaning, what the callee can reach, ownership, lifetime, container requirements, and value invariants. Name positional booleans through fields or types. Pass a request object instead of an unrelated domain object. Require the operations the body performs, and pick the parameter form from the table.

| Pass | When |
| --- | --- |
| A value | The owner acquired it and the body only reads it, such as a timestamp or a rate |
| Deterministic state | The body advances it and a caller must reproduce the sequence, such as a seeded generator |
| A provider | The caller must choose the source at call time, such as a clock or client, and its effects stay in the ledger |
| A request object | Several related values form one call and positional meaning would otherwise be lost |
| A borrowed view | The body reads or iterates and retains nothing, and the owner's lifetime is stated |
| An owned value | The function retains or transfers the data |

A concrete container is a finding only when it excludes a real or likely caller, exposes storage policy the operation does not own, or makes substitution in a test materially harder. Contiguity, ownership, performance, and language ergonomics are legitimate reasons to keep it. A method on a cohesive receiver passes even when the receiver holds state the method does not read. The finding is a parameter that grants unrelated data or couples the callee to one representation. Carry an invariant in a strong type when it is repeated, consequential, and stable. Keep a rare local check local, since encoding every precondition in types is neither wise nor feasible.

For an unclear argument, a broad object, a container type stronger than the body needs, a borrowed value, a value invariant or strong type, a lock, or a prerequisite that repeats across callers, read [interfaces and results](references/interfaces-and-results.md).

**Done when** a reader can name each argument's role from the call without opening the declaration, each restriction maps to a body operation or a stated cost, and no parameter grants unrelated data or a representation the callee does not need.

### 5. Choose result and failure channels

Return the complete result as data by default and let the caller decide what to do with it. Mutate caller-owned state when the caller supplied it for that purpose. Switch to a visitor or iterator only when the complete result is too large to hold, and state its lifetime, early-stop, and partial-progress rules. Perform an outside effect only in an effect owner. Preserve every distinction between negative outcomes that a caller acts on, and keep a boolean when every caller treats the outcomes the same way. Put failure policy where enough domain context exists to choose it.

For a large result, a borrowed value, a boolean that collapses outcomes, or an unclear failure owner, read [interfaces and results](references/interfaces-and-results.md).

**Done when** the contract says whether the result is materialized or streamed, every borrowed result has a stated lifetime, every outcome the caller uses is preserved, and each failure decision has one owner.

### 6. Keep one conceptual level

Describe the body as short action phrases, each a peer step in the job the function name states. Indent mechanics beneath the action they implement. Move a child procedure into a named function, a trusted algorithm, or a type that owns the operation when the move creates vocabulary, isolates a contract, or gives tests a direct entry point. When writing, outline first and then code. When reviewing, map every executable block to a phrase and look for blocks that do not fit.

For a raw loop, a hand-written algorithm, a nested block, or a section comment, read [abstraction and policy](references/abstraction-and-policy.md).

**Done when** every executable block maps to one action phrase and no top-level phrase describes how a sibling phrase is done.

### 7. Give policy one owner

Inspect sibling functions and direct call sites that construct or consume the same values. Find repeated normalization, ordering, validation, seeding, locking, key construction, or storage policy, and move each shared invariant to one owner. If that owner would live in code outside the task's scope, keep the new function consistent with the existing policy and name the duplication in the report instead of editing the sibling. Remove pass-through wrappers, meaning helpers that forward a call and add no vocabulary, policy, or compatibility. An adapter that turns ambient access into a parameter owns a distinct contract, so keep it. Then return to steps 4 and 5 for every signature the new owner touched, since a parameter that served the old mechanics can now weaken.

For repeated policy or a pass-through wrapper, read [abstraction and policy](references/abstraction-and-policy.md).

**Done when** each shared policy changes in one implementation and every remaining wrapper owns vocabulary, policy, compatibility, or a distinct contract.

### 8. Verify the boundary

Test an ordinary value, then each case the function can have: an empty input, a single element, an input whose answer equals the input, rejected inputs, changed state, every result channel, errors, and partial progress. Name each skipped case with the reason it cannot occur, such as a total function having no rejected input. Give each untrusted callee one test at its own boundary, or one integration test through this function when this function relies on its behavior. After the function exists or changes, read one real call site.

When writing, add tests as each channel takes shape.

When changing an existing function, first list the callers that depend on its mutation, ordering, error distinctions, allocation, timing, or borrowed lifetime. If existing tests do not protect a boundary you are moving, add tests that pin today's behavior before you move it. Make the smallest coherent change. If behavior must change, say so in the task, update the callers, and test the new contract.

Run the tests that cover this function and its callers. Run the full suite when repository policy asks for it or the change touches a contract that many callers share.

When the task is review only, list the tests you would add and the test set the implementer should run, in place of writing or running them.

**Done when** tests control every behavior-affecting input, assert every output channel the caller uses, and establish each untrusted callee contract the function relies on. For review only, the report lists each missing test with the input it controls and the channel it asserts.

## Report the outcome

For implementation work, state which contracts you chose or changed, which behavior stayed stable, and what verification passed. Add each ledger entry that produced a finding and any inherited debt, out-of-scope duplication, or missing test the steps told you to name. For review-only work, report prioritized findings with file and line evidence, the consequence, and a concrete correction. Mark each finding by consequence: correctness, local reasoning or testability, caller usability, or optional design improvement.
