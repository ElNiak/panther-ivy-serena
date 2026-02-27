---
name: nct-compile
description: Compile an Ivy model to a test binary via panther-serena
arguments:
  - name: file
    description: Path to the .ivy file to compile (relative to project root)
    required: true
  - name: target
    description: Compilation target (default "test")
    required: false
  - name: isolate
    description: Optional isolate name to compile specifically
    required: false
---

Compile the specified Ivy model to a test executable using panther-serena.

## Instructions

1. Accept the file path argument. If no file is provided, ask the user which .ivy file to compile.

2. Determine the target:
   - If `target` argument provided, use it
   - Otherwise default to `"test"`

3. Call `mcp__plugin_panther-ivy-plugin_panther-serena__ivy_compile` with:
   - `relative_path`: the provided file path
   - `target`: the compilation target
   - `isolate`: the isolate argument if provided, otherwise omit

4. Parse the JSON result containing `stdout`, `stderr`, and `return_code`.

5. Present results in this structured format:

### If return_code is 0 (SUCCESS):
```
## Compilation Result: SUCCESS

**File:** {file_path}
**Target:** {target}
**Isolate:** {isolate or "all"}

Test binary compiled successfully.
The executable can be found in the build/ directory.

### Next Steps
- Run the test binary against an IUT via PANTHER experiment framework
- Use `/nct-check` to verify formal properties before running
```

### If return_code is non-zero (FAILURE):
```
## Compilation Result: FAILURE

**File:** {file_path}
**Target:** {target}

### Errors
{Parse stderr for specific error messages}

### Suggested Actions
- Run `/nct-check {file}` first to verify formal properties
- Use `mcp__plugin_panther-ivy-plugin_panther-serena__get_symbols_overview` to check file structure
- Check for missing includes or undefined symbols
```

**IMPORTANT**: Do NOT run `ivyc` directly via Bash. Always use `mcp__plugin_panther-ivy-plugin_panther-serena__ivy_compile`.
