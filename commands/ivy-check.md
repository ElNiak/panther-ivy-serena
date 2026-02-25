---
name: ivy-check
description: Run ivy_check on an Ivy model file to verify it for errors
args:
  - name: file
    description: Path to the .ivy file to check
    required: true
---

Run `ivy_check` on the specified Ivy model file to verify it for type safety, invariant preservation, and protocol correctness.

## Execution

1. Verify the target file exists and has a `.ivy` extension.
2. Run the verification command:

```bash
ivy_check $ARGUMENTS
```

If the file path contains the `$ARGUMENTS` substitution, use it directly. Otherwise construct the command as:

```bash
ivy_check <file>
```

To check a specific isolate, the user may pass additional arguments like `isolate=<name>`.

3. Capture and report the full output.

## Reporting Results

### On success (output contains "OK"):
Report that verification passed. List any warnings if present.

### On failure:
- Quote the exact error message.
- Identify the file and line number where the error occurred.
- Briefly explain what the error means:
  - "failed to verify" -- an invariant or postcondition could not be proved.
  - "type error" -- a type mismatch was found.
  - "ungrounded" -- a variable is not properly bound.
  - "not found" -- a referenced symbol does not exist.
- Suggest a fix if the cause is apparent from the error message.

### If `ivy_check` is not installed:
Report that `ivy_check` is not available in the current environment. Suggest:
- Checking that the Ivy tool is installed (`pip install ms-ivy` or built from the PANTHER ivy submodule).
- Verifying the virtual environment is activated.
- Checking the PATH includes the Ivy binary directory.
