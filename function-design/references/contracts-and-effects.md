# Contracts and effects

Use this reference when a function depends on state or behavior not fully represented by its call boundary.

## Honest boundaries

Call a function **honest** when its contract accounts for every meaningful input, authorized mutation, result, effect, and failure. This is a reasoning property, not a moral label and not the same as purity.

- Mutation can be honest when the caller supplies or owns the changed state.
- A receiver is an implicit input and may also be an output channel.
- A `void` return does not mean no output. Receiver mutation, argument mutation, callbacks, logs, files, and network calls are outputs.
- A function may have several result channels, such as mutating a collection and returning an iterator.
- An honest function can still be slow, incorrect, or badly named. The boundary makes its behavior inspectable; it does not guarantee quality.

### Hidden state versus explicit state

```text
tax_rate = load_global_rate()

function total(items):
    return subtotal(items) * (1 + tax_rate)
```

The same explicit input can produce a different result after unrelated code changes `tax_rate`. The name and parameter list omit a real input.

```text
function total(items, tax_rate):
    return subtotal(items) * (1 + tax_rate)
```

The caller can now choose, reproduce, and test the effective input. If fetching the current rate is the operation's purpose, keep that fetch in a named integration function and pass the fetched value into the calculation.

**Check:** Can a test control every value that changes the result without mutating process-wide state?

## Higher-order functions inherit behavior

Passing a callback, strategy, clock, printer, or client makes the dependency visible and replaceable. It does not erase what the dependency does. The composed call inherits the supplied dependency's hidden reads, writes, failures, blocking behavior, and external effects.

```text
function remove_matching(items, predicate):
    mutate items by removing values where predicate(value) is true
```

The collection mutation is caller-authorized. The full call is locally controlled only if `predicate` has the required contract. A predicate that reads the clock or writes a global adds those behaviors to the composed call.

For callbacks, state:

- whether the value is read-only, mutable, owned, or borrowed;
- whether retaining it requires a copy;
- whether callback failure stops the producer;
- what partial progress remains visible; and
- which callback effects the producer inherits.

**Check:** Does the contract still describe the call after substituting an effectful or failing callback?

## Keep effects near integration boundaries

Build calculations from caller-controlled values and state. Keep file access, UI drawing, network delivery, environment reads, and other outside effects near the outermost practical owner, such as a command handler, request handler, controller, or framework hook.

```text
function framework_update(delta):
    step_physics(delta)
    manage_lifetime(delta)
```

The hook may translate framework values, preserve required ordering, and coordinate effects. Application rules still get ordinary callable homes that tests can invoke directly.

Do not split calculation from effect when one transaction must make them atomic. Let the boundary own the transaction and expose success, failure, rollback, and partial-progress behavior.

Use a practical observation boundary. Routine allocator bookkeeping rarely belongs in an application contract. Time, caller-visible mutation, file output, shared cache state, and network activity usually do.

**Check:** Is the effect located at the highest practical owner without splitting an invariant or transaction?

## Pass deterministic state explicitly

A pseudorandom generator is deterministic mutable state after it is seeded. A hidden global generator makes sequence position, reseeding, and concurrency invisible.

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

The explicit generator supports replay, debugging, tests, and caller-selected behavior. Passing a global generator explicitly exposes the choice but does not make shared mutation thread-safe. Fixed-seed replay also depends on generator algorithm and call order.

Treat clocks differently. Passing a clock capability makes acquisition replaceable, but reading it is still an effect of the composed call. Pass a timestamp value when the calculation needs only one observation.

**Check:** Is changing state explicit, and does the contract distinguish deterministic state from an effectful provider?

