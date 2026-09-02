# Function design at a glance

Use this page when you need every rule on one screen before or after the steps. Each leaf is stated in full in SKILL.md or a reference.

**Write functions that support local reasoning, abstraction, and testing. Reuse is secondary.**

- Make the contract honest
  - Show every behavior-affecting input.
  - Show every result and mutation.
  - Treat receiver state as an input.
  - Treat changed state as an output.
  - Keep clocks, globals, managers, and input or output out of the body unless the signature shows them.
  - Treat a callee's hidden access as this function's hidden access. Dishonesty spreads up the call tree, and a lambda around the read does not contain it.
- Keep outside effects in the effect owner
  - Put file, network, console, and device access in the effect owner, such as `main`, a request handler, or an adapter whose signature names the effect.
  - Keep the inner calculation deterministic when practical.
  - Pass changing state, such as a seeded random generator, explicitly.
  - Keep a framework hook such as `update` thin, and put the rules it triggers in ordinary callable functions.
  - Remember that mutation is fine when the caller supplies and controls it.
- Design the function from its caller
  - Write the call you wish existed before solving its implementation.
  - Ask for the values and operations the body uses, and keep a stronger type when a real caller supplies it and contiguity, ownership, or performance favors it.
  - Replace unclear positional arguments with named fields or types.
  - Pass the values the call needs instead of an unrelated object that happens to hold them.
  - Return the distinctions the caller needs to make its next decision.
- Put requirements in the contract
  - Turn "A must happen before B" into a value returned by A and required by B.
  - Represent an invariant with a type when it is repeated, consequential, and stable.
  - Establish an invariant once instead of checking it in every function.
  - Keep rare or local requirements as local checks when a new type adds more cost than value.
- Choose the right result channel
  - Return a value when the result has a reasonable size.
  - Use a visitor or iterator when materializing everything is too expensive.
  - Account for ownership, borrowing, early termination, and partial progress.
  - Keep different failures distinct when callers respond to them differently.
  - Decide failure policy where the domain context exists, and never let invalid input flow through as infinities or NaNs.
- Keep one conceptual level inside each body
  - The top-level statements should describe peer actions.
  - Move lower-level mechanics behind names that describe their purpose.
  - Treat raw loops and section comments as clues, not automatic failures.
  - Use a trusted algorithm instead of hand-written search or traversal, since the hand-written version hides bugs that survive rereading.
- Extract functions for understanding
  - Extract at the first coherent occurrence instead of waiting for repetition.
  - Extract when a name reduces the amount a reader must understand.
  - Extract when it creates a useful test boundary.
  - Leave obvious code inline when a helper would only rename syntax.
- Give each policy one owner
  - Look for repeated normalization, ordering, validation, and lookup rules.
  - Move shared policy into the type or collection that owns it.
  - Reconsider function parameters after moving the policy.
  - Remove forwarding helpers that no longer own a useful decision.
  - Test the lower-level pieces so confidence can compose upward.
