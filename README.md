# skills-tmp

Working repository for skills that change often. Each top-level directory is one skill and is symlinked into `~/.codex/skills/` and `~/.claude/skills/`.

- `function-design`: function design with an honest, exact, testable contract. The earlier v1 skill is in git history before this rename.

## Function design at a glance

A good function reduces what the reader must know,
hides mechanics behind a useful abstraction,
exposes every real dependency and result,
and creates a boundary that focused tests can trust.

The tree below is a human orientation to the rules in `function-design`. The skill files are the source of truth.

```text
Functions manage human complexity
│
├── Support local reasoning
│   └── Let the reader understand the call without searching the system.
│
├── Provide abstraction
│   └── Hide mechanics behind a name that states the caller's operation.
│
├── Create focused test boundaries
│   └── Let tests control the function's inputs and inspect its outputs directly.
│
├── Treat reuse as a benefit, not the extraction threshold
│   └── Consider extracting the first coherent operation when it improves reasoning or testing.
│
├── Make the contract honest
│   ├── Show every meaningful input.
│   ├── Show every result and authorized mutation.
│   ├── Treat receiver state as an input.
│   ├── Treat changed state as an output.
│   └── Avoid hidden clocks, globals, managers, and I/O.
│
├── Trace dishonesty through the call tree
│   ├── A hidden dependency in a callee also belongs to its caller.
│   ├── An accessor can hide process history despite its harmless name.
│   ├── A callback contributes its reads, writes, failures, and effects.
│   └── Wrapping hidden access in a lambda does not contain it.
│
├── Keep outside effects near orchestration boundaries
│   ├── Put file, network, console, and device access near the system edge.
│   ├── Build inner calculations from caller-controlled values and state.
│   ├── Pass deterministic state, such as a seeded random generator, explicitly.
│   └── Remember that explicit mutation can still form an honest contract.
│
├── Keep framework hooks thin
│   ├── The framework controls when the hook runs.
│   ├── Let the hook translate inputs, preserve ordering, and coordinate effects.
│   └── Move application rules into ordinary functions that code and tests can call.
│
├── Design the function from its caller
│   ├── Write the call you wish existed before solving its implementation.
│   ├── Use values already available at the call site.
│   ├── Replace unclear positional arguments with named fields or types.
│   └── Avoid passing an unrelated object merely because it contains the needed value.
│
├── Match parameters to the real operations
│   ├── Do not require storage, ownership, or authority that the body does not use.
│   ├── Do not weaken the contract until it stops guaranteeing what the body needs.
│   ├── Ask for iteration when the body only iterates.
│   ├── Ask for contiguous storage when the implementation requires it.
│   └── Prefer an exact contract over either concrete coupling or maximum genericity.
│
├── Put requirements in the contract
│   ├── Turn "A must happen before B" into a value returned by A and required by B.
│   ├── Represent important, repeated invariants with types.
│   ├── Establish an invariant once instead of checking it in every consumer.
│   └── Keep rare local requirements as local checks when a new type costs more than it saves.
│
├── Put failure policy where the needed context exists
│   ├── Do not make a low-level helper invent a domain response.
│   ├── Let the informed caller choose whether to skip, substitute, assert, or return an error.
│   ├── Keep expected failures visible instead of allowing invalid values or NaNs to escape.
│   └── Preserve different failures when they lead callers to different actions.
│
├── Choose the right result channel
│   ├── Return the complete result as data by default.
│   ├── Use a visitor or iterator when materializing everything is too expensive.
│   ├── Account for ownership, borrowing, early termination, and partial progress.
│   └── Account for every output channel, including mutation and callback calls.
│
├── Keep one conceptual level inside each body
│   ├── Make the top-level statements describe peer actions.
│   ├── Move lower-level mechanics behind names that describe their purpose.
│   ├── Treat raw loops and section comments as clues, not automatic failures.
│   └── Do not confuse one conceptual level with one function call per line.
│
├── Prefer trusted algorithms over hand-written machinery
│   ├── Replace custom searches and similar machinery when a library operation fits.
│   ├── Remove local loop state and boundary arithmetic that can hide bugs.
│   └── Keep interpretation of the algorithm's result beside the call when both form one action.
│
└── Give each policy one owner
    ├── Find repeated normalization, ordering, validation, and lookup rules.
    ├── Move shared policy into the type or collection that owns it.
    ├── Reconsider function parameters after moving the policy.
    ├── Remove pass-through wrappers, and keep an adapter that turns hidden access into a parameter.
    └── Test the lower-level contracts so confidence can compose upward.
```
