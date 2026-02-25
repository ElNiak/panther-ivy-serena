---
name: ivy-compile
description: Compile an Ivy model to a C++ test binary using ivyc
args:
  - name: file
    description: Path to the .ivy file to compile
    required: true
---

Compile the specified Ivy model to a C++ test binary using `ivyc`.

## Execution

1. Verify the target file exists and has a `.ivy` extension.
2. Run the compilation command:

```bash
ivyc target=test $ARGUMENTS
```

If additional arguments are provided (e.g., `isolate=<name>`), include them in the command.

3. Capture and report the full output.

## Reporting Results

### On success:
Report that compilation succeeded. Note the output binary location if shown in the output.
The compiled binary is typically placed in the current working directory with the same base name as the `.ivy` file.

### On failure:
- Quote the exact error message from `ivyc`.
- Identify the file and line number where the error occurred.
- Common compilation errors:
  - **Type errors**: Same as `ivy_check` type errors. The model must pass verification before it can compile.
  - **Extraction errors**: Some Ivy constructs cannot be compiled to C++. Identify which construct caused the issue.
  - **Missing implementations**: Abstract actions without implementation blocks cannot be extracted.
  - **C++ compilation errors**: The generated C++ code failed to compile. This may indicate an `ivyc` bug or an unsupported pattern.
- Suggest running `ivy_check` first if the model has not been verified.

### If `ivyc` is not installed:
Report that `ivyc` is not available in the current environment. Suggest:
- Checking that the Ivy tool is installed (`pip install ms-ivy` or built from the PANTHER ivy submodule).
- Verifying the virtual environment is activated.
- Checking the PATH includes the Ivy binary directory.

## Notes

- Compilation requires that the model passes `ivy_check` first. If verification has not been run, recommend running `/ivy-check` before compiling.
- The `target=test` flag generates a test binary that can be run to exercise the protocol model.
- Other target options include `target=class` (C++ class library) and `target=repl` (interactive REPL).
