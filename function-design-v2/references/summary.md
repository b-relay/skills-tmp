# Function design at a glance

Use this page when you need every rule on one screen before or after the steps. Each leaf is stated in full in SKILL.md or a reference.

**Write functions that support local reasoning, abstraction, and testing. Reuse is secondary.**

- Make the contract honest
  - Show every behavior-affecting input.
  - Show every result and mutation.
  - Treat receiver state as an input.
  - Treat changed state as an output.
  - Keep clocks, globals, managers, and input or output out of the body unless the signature shows them.
  - Treat a callee's ambient access as this function's ambient access. Dishonesty spreads up the call tree, and a lambda around the read does not contain it.
  - Treat a passed clock, client, or callback as honest only while its effects, blocking, and failures are part of the stated contract.
- Keep outside effects in the effect owner
  - Put file, network, console, and device access in the effect owner, such as `main`, a request handler, or an adapter whose signature names the effect.
  - Build the inner calculation from caller-controlled values and state.
  - Pass a provider's results, not the provider, below the layer that calls it.
  - Pass deterministic state, such as a seeded random generator, explicitly.
  - Keep a framework hook such as `update` thin, and put the rules it triggers in ordinary callable functions.
  - Mutation is honest when the caller supplies the state and authorizes the write.
- Design the function from its caller
  - Write the call you wish existed before solving its implementation.
  - Ask for the values and operations the body uses. Keep a concrete type when a real caller supplies it and contiguity, ownership, performance, or language ergonomics favor it.
  - Check both directions: a concrete container can reject a valid caller, and an unconstrained generic can admit an input that lacks an operation the body needs.
  - Replace unclear positional arguments with named fields or types.
  - Pass the values the call needs instead of an unrelated object that happens to hold them.
  - Return the distinctions the caller needs to make its next decision.
- Put prerequisites and invariants in the contract
  - Turn "A must happen before B" into a value returned by A and required by B.
  - Represent an invariant with a type when it is repeated, consequential, and stable.
  - Establish an invariant once instead of checking it in every function.
  - Encode only a guarantee that holds for every value the type admits.
  - Keep a rare or local prerequisite as a local check when a new type adds more cost than protection.
- Choose the right result channel
  - Return the complete result as data by default, and let the caller decide what to do with it.
  - Use a visitor or iterator when materializing everything is too expensive.
  - Account for ownership, borrowing, early termination, and partial progress.
  - Keep different failures distinct when callers respond to them differently.
  - Decide failure policy where the domain context exists. Reject invalid input there instead of letting infinities or NaNs escape, because the failure then surfaces far from its cause.
- Keep one conceptual level inside each body
  - Make each top-level statement a peer action in the job the function name states.
  - Move lower-level mechanics behind names that describe their purpose.
  - Treat raw loops and section comments as clues, not automatic failures.
  - Use a trusted algorithm instead of hand-written search or traversal, since the hand-written version hides bugs that survive rereading.
- Extract functions for understanding
  - Consider a function at the first coherent occurrence instead of waiting for repetition.
  - Extract when a name reduces the amount a reader must understand.
  - Extract when it creates a useful test boundary.
  - Leave obvious code inline when a helper would only rename syntax.
- Give each policy one owner
  - Look for repeated normalization, ordering, validation, and lookup rules.
  - Move shared policy into the type or collection that owns it.
  - Reconsider function parameters after moving the policy.
  - Remove pass-through wrappers that add no vocabulary, policy, or compatibility, and keep an adapter that turns ambient access into a parameter.
  - Test the lower-level pieces so confidence can compose upward.
