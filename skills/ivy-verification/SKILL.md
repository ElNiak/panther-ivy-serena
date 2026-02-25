---
name: ivy-verification
description: Use when running formal verification on Ivy models, checking for errors, or verifying protocol specifications. Covers ivy_check workflows, interpreting verification output, and debugging model errors.
---

# Ivy Verification Workflow

## Overview

Ivy is a formal specification language used for modeling and verifying protocol implementations.
The primary verification tool is `ivy_check`, which statically checks Ivy models for type safety,
invariant preservation, and protocol correctness.

## Running Verification

### Using Serena (preferred)

When Serena is available with the Ivy LSP backend, use the `execute_shell_command` tool:

```
ivy_check <file.ivy>
```

For specific isolates:

```
ivy_check isolate=<isolate_name> <file.ivy>
```

### Common Verification Patterns

**Full model check** -- verifies all isolates and exported actions:
```
ivy_check protocol_model.ivy
```

**Single isolate check** -- faster, targets one component:
```
ivy_check isolate=protocol_spec protocol_model.ivy
```

**Check with trace output** -- produces counterexample traces on failure:
```
ivy_check trace=true protocol_model.ivy
```

## Interpreting Results

### Successful Verification

A successful check produces output like:
```
OK
```
or lists each checked isolate with `OK` status. This means all proof obligations were discharged.

### Verification Failures

Failures include:
- **Line numbers** pointing to the failing assertion or action
- **Counterexample traces** showing a sequence of actions leading to the failure
- **Error type** indicating what went wrong

Common failure patterns:

1. **Invariant not preserved** -- an action modifies state in a way that violates a declared invariant.
   The output shows which action and which invariant clause failed.
   ```
   error: failed to verify invariant preservation in action client.send
   ```
   Fix: strengthen the invariant, add preconditions to the action, or fix the action logic.

2. **Type safety error** -- a value is used with an incompatible type.
   ```
   error: type mismatch at line 42: expected packet_type, got nat
   ```
   Fix: ensure all variables and expressions have consistent types.

3. **Ungrounded relation** -- a relation is used in a way that leaves variables unbound.
   ```
   error: ungrounded variable X in relation recv(X,Y)
   ```
   Fix: ensure all variables in relation expressions are bound by quantifiers or appear in the head.

4. **Safety property violation** -- an exported action can reach an unsafe state.
   ```
   error: safety property violated at line 85
   ```
   Fix: add missing invariants, strengthen preconditions, or fix the protocol logic.

5. **Liveness/progress failure** -- the model cannot guarantee progress.
   Fix: check for deadlock scenarios and add fairness assumptions if appropriate.

## Debugging Workflow

Follow this cycle when verification fails:

1. **Check**: Run `ivy_check` on the model.
2. **Read the error**: Note the line number, error type, and any counterexample trace.
3. **Locate the issue**: Open the file at the indicated line. Understand the failing obligation.
4. **Diagnose**: Determine if the issue is:
   - A missing invariant (the model under-specifies expected behavior)
   - A bug in the action logic (the model is incorrect)
   - A missing precondition (the action is called in unexpected contexts)
5. **Fix**: Apply the minimal fix. Prefer adding invariants over weakening specifications.
6. **Re-check**: Run `ivy_check` again. Repeat until all checks pass.

## Common Ivy Verification Errors and Fixes

### "failed to verify" on an action body
The action's postcondition or an invariant is not maintained. Check:
- Are all modified relations updated consistently?
- Does the action's `ensure` clause match what the body actually does?

### "cannot find isolate X"
The isolate name is misspelled or not declared. Check:
- Spelling of the isolate name in the command and in the `.ivy` file.
- That the isolate is declared with `isolate X = { ... }` or `object X = { ... }` with `specification` or `implementation` keywords.

### "circular dependency"
Two or more modules or objects depend on each other. Fix:
- Refactor to break the cycle, typically by introducing an abstract interface.

### "uninterpreted sort has no instances"
A type was declared but never given concrete values. Fix:
- Add at least one constructor or axiom that provides instances of the sort.

### Z3 timeout or "unknown" result
The SMT solver could not decide within the time limit. Fix:
- Simplify the proof obligation by breaking it into smaller lemmas.
- Add ghost state or auxiliary invariants to guide the prover.
- Use `isolate` boundaries to limit what the solver must reason about.
