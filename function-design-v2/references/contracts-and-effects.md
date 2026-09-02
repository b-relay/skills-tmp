# Contracts and effects

Use this reference when a function depends on state or behavior that its call boundary does not show.

## Honest boundaries

`sort(range)` mutates the caller's range, yet the caller supplies the state, authorizes the write, and can inspect the result. `get_time()` mutates nothing, yet its empty parameter list hides the system clock, so the caller cannot choose or reproduce the effective input. Mutation alone decides nothing. The question is whether the signature shows every behavior-affecting dependency and observable channel, so that a test can substitute or control each one. Concentrate dishonest functions in a few named effect owners so the tree below them can stay honest.

- A receiver is an implicit input and may also be an output channel.
- A `void` return does not mean no output. Receiver mutation, argument mutation, callbacks, logs, files, and network calls are outputs.
- A function may use several result channels at once, such as mutating a collection and returning an iterator.
- A dishonest callee makes its caller dishonest unless the caller turns the ambient access into an explicit input. Dishonesty spreads up the call tree.

### Ambient state versus explicit state

```text
tax_rate = load_global_rate()

function total(items):
    return subtotal(items) * (1 + tax_rate)
```

The same explicit input produces a different result after unrelated code changes `tax_rate`. The name and parameter list omit a real input.

```text
function total(items, tax_rate):
    return subtotal(items) * (1 + tax_rate)
```

The caller can now choose, reproduce, and test the effective input. If fetching the current rate is the operation's purpose, let the effect owner fetch it and pass the fetched value into the calculation.

### Accessors that hide history

```text
function required_assets():
    assets = all required assets
    remove each asset the asset manager reports as already loaded
    return assets
```

The name promises every required asset. The body returns the ones not yet loaded, and "loaded" depends on what the process did earlier. One caller sees the expected result. A later caller with a different loading history sees a different one, and a focused test passes for the wrong reason.

## Higher-order functions inherit behavior

Passing a callback, strategy, clock, printer, or client makes the dependency visible and replaceable. It does not erase what the dependency does. The composed call inherits the supplied dependency's ambient reads, writes, failures, blocking, and outside effects. A callback that reads the clock or writes a global adds those behaviors to the composed call, so the contract must still describe them.

## Keep effects in the effect owner

Build calculations from caller-controlled values and state. Put ambient reads and outside effects in the effect owner, such as `main`, a command handler, a request handler, a controller, a framework hook, or an adapter whose signature names the effect. Honest functions can call other honest functions and form whole subtrees. A small effectful region near the roots leaves a large locally testable region below it. The boundary is right when switching console input and output to file input and output changes only the owner.

Framework hooks invert ownership. The application writes `update`, and the framework decides when to call it. Keep the hook thin:

```text
function framework_update(delta):
    step_physics(delta)
    manage_lifetime(delta)
```

The hook may translate framework values, preserve required ordering, and coordinate effects. It need not be one line. Application rules still get ordinary callable functions that tests invoke directly.

In a layered service the effect owner is often not the process root:

```text
function handle_order_request(request, repository, clock):
    order = repository.load(request.order_id)
    priced = price_order(order, clock.now())
    repository.save(priced)
    return response_for(priced)
```

The handler and the repository own the effects, and their contracts show them. `price_order` takes the loaded order and the time, so it stays honest and testable with plain values. Passing `repository` or `clock` into `price_order` would thread a mechanism through a layer that only needs their results.

Use a practical observation boundary. Routine allocator bookkeeping rarely belongs in an application contract. Time, caller-visible mutation, file output, shared cache state, and network activity usually do. Apply the cutoff consistently.

## Pass deterministic state explicitly

A pseudorandom generator is deterministic mutable state once seeded. The volatile step is acquiring the seed from time or the operating system. A hidden global generator makes sequence position, reseeding, and concurrent use invisible.

```text
function populate(count):
    repeat count times:
        add random_particle(global_rng())
```

```text
function populate(count, rng):
    repeat count times:
        add random_particle(rng)
```

The explicit generator lets a product share a seed, lets QA reproduce an odd run, and lets a developer replay suspected random behavior. Wrapping the global read in a lambda does not contain it. The lambda still reads and mutates the global generator. If a global generator must stay, pass it at the call site, `populate(count, global_rng())`, so the dependency is visible there.
