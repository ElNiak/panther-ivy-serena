---
name: Panther-Serena for Ivy
description: This skill should be used when the user asks about "using serena for Ivy", "Ivy tool guidance", "panther-serena tools", "how to check Ivy files", "how to compile Ivy", "formal verification tools", "ivy_check alternative", "MCP tools for Ivy", or mentions any Ivy toolchain operation in the PANTHER framework. Provides tool mapping from CLI to panther-serena MCP equivalents and correct usage patterns.
---

# Panther-Serena for Ivy

## Overview

All Ivy operations within the PANTHER framework MUST use panther-serena MCP tools instead of direct CLI commands. panther-serena provides semantic code navigation, formal verification, and compilation through a consistent MCP interface that integrates with Claude Code's tool system.

## Why panther-serena Instead of Direct CLI

- **Consistency** — All operations tracked through MCP tool calls with structured JSON output
- **Semantic navigation** — Symbol-level understanding of Ivy code (not just text search)
- **Integration** — Verification results consumed by agents and commands
- **Safety** — Operations run within project context, preventing accidental execution outside the project
- **Tool tracking** — All tool calls are logged and auditable

## Tool Mapping Table

| Direct CLI Command | panther-serena MCP Tool | Usage |
|---|---|---|
| `ivy_check file.ivy` | `mcp__plugin_serena_serena__ivy_check` | Formal verification |
| `ivy_check isolate=X file.ivy` | `mcp__plugin_serena_serena__ivy_check` with `isolate` param | Isolate-specific checking |
| `ivyc target=test file.ivy` | `mcp__plugin_serena_serena__ivy_compile` | Test compilation |
| `ivyc target=test isolate=X file.ivy` | `mcp__plugin_serena_serena__ivy_compile` with `isolate` param | Isolate-specific compilation |
| `ivy_show file.ivy` | `mcp__plugin_serena_serena__ivy_model_info` | Model introspection |
| `ivy_show isolate=X file.ivy` | `mcp__plugin_serena_serena__ivy_model_info` with `isolate` param | Isolate-specific info |
| `cat file.ivy` | `mcp__plugin_serena_serena__read_file` | Read file contents |
| `grep pattern *.ivy` | `mcp__plugin_serena_serena__search_for_pattern` | Search across files |
| Manual symbol lookup | `mcp__plugin_serena_serena__find_symbol` | Find specific symbols |
| Manual file overview | `mcp__plugin_serena_serena__get_symbols_overview` | List top-level symbols |
| Manual dependency trace | `mcp__plugin_serena_serena__find_referencing_symbols` | Find references |
| Create new .ivy file | `mcp__plugin_serena_serena__create_text_file` | Write new files |
| Edit .ivy file | `mcp__plugin_serena_serena__replace_symbol_body` | Edit symbol bodies |
| Edit by regex | `mcp__plugin_serena_serena__replace_content` | Fine-grained edits |

## Ivy-Specific Tool Parameters

### ivy_check — Formal Verification
```
Parameters:
  relative_path: str          # Path to .ivy file (relative to project root)
  isolate: str | None = None  # Optional isolate name to check specifically
  max_answer_chars: int = -1  # Output limit (-1 for default)

Returns: JSON { stdout, stderr, return_code }

Underlying command: ivy_check [isolate=X] path.ivy
```

Checks isolate assumptions, invariants, and safety properties. Return code 0 means all checks pass.

**Common output patterns:**
- `OK` — All checks passed
- `FAIL` — One or more checks failed
- `error: assumption failed` — An isolate assumption was violated
- `error: invariant ... failed` — An invariant does not hold
- `error: safety property ... violated` — A safety property was violated

### ivy_compile — Test Compilation
```
Parameters:
  relative_path: str          # Path to .ivy file
  target: str = "test"        # Compilation target (usually "test")
  isolate: str | None = None  # Optional isolate name
  max_answer_chars: int = -1  # Output limit

Returns: JSON { stdout, stderr, return_code }

Underlying command: ivyc target=X [isolate=Y] path.ivy
```

Compiles Ivy model to C++ test executable. The `target=test` option generates a randomized test binary that uses Z3/SMT for action generation.

**Common output patterns:**
- Successful compilation produces no stdout, return code 0
- Compilation errors appear in stderr with line numbers
- Large models may take significant time to compile

### ivy_model_info — Model Introspection
```
Parameters:
  relative_path: str          # Path to .ivy file
  isolate: str | None = None  # Optional isolate name
  max_answer_chars: int = -1  # Output limit

Returns: JSON { stdout, stderr, return_code }

Underlying command: ivy_show [isolate=X] path.ivy
```

Displays model structure: types, relations, actions, invariants, and isolates. Useful for understanding model architecture and debugging.

## Standard Serena Navigation Tools

Beyond Ivy-specific tools, use standard serena tools for code navigation:

### Symbol Navigation
- **`find_symbol`** — Find a symbol by name path. Use `name_path` with pattern like `module/function` and `include_body=True` to read implementation.
- **`get_symbols_overview`** — List all top-level symbols in a file. Use `relative_path` to specify the .ivy file.
- **`find_referencing_symbols`** — Find all references to a symbol. Trace dependencies between layers.

### File Operations
- **`read_file`** — Read file contents. Use `start_line`/`end_line` for partial reads.
- **`create_text_file`** — Create new .ivy files.
- **`replace_symbol_body`** — Replace an entire symbol's implementation.
- **`replace_content`** — Replace content by regex pattern for fine-grained edits.
- **`insert_after_symbol`** / **`insert_before_symbol`** — Add code relative to existing symbols.

### Search and Discovery
- **`search_for_pattern`** — Regex search across files. Use `relative_path` to restrict scope.
- **`list_dir`** — List directory contents for exploring protocol structure.
- **`find_file`** — Find file by name pattern.

## Recommended Workflow

### Navigate -> Understand -> Edit -> Verify

1. **Navigate** — Use `find_symbol` and `get_symbols_overview` to locate relevant code
2. **Understand** — Use `read_file` and `find_referencing_symbols` to understand context and dependencies
3. **Edit** — Use `replace_symbol_body` or `create_text_file` for modifications
4. **Verify** — Use `ivy_check` to confirm formal properties still hold

### Example: Adding a New Requirement

```
1. search_for_pattern("frame.stream.handle")    # Find where stream handling is defined
2. find_symbol("frame/stream/handle")            # Read the current implementation
3. find_referencing_symbols("frame.stream.handle") # Find all tests using it
4. replace_symbol_body(...)                       # Add new before/after monitor
5. ivy_check(relative_path="...")                 # Verify consistency
```

## Prerequisites

1. **panther-serena** installed with Ivy tools enabled:
   - `ivy` added to `.serena/project.yml` languages list
   - `ivy_check`, `ivy_compile`, `ivy_model_info` in `included_optional_tools`
2. **ivy_lsp** installed:
   - `pip install ivy-lsp` (from PyPI)
   - Or `pip install -e ".[lsp]"` from panther_ivy directory
3. **Ivy toolchain** available in PATH (ivy_check, ivyc, ivy_show)

## Enforcement

The panther-ivy-serena plugin includes a PreToolUse hook that blocks direct Ivy CLI calls in Bash. If a Bash command containing `ivy_check`, `ivyc`, `ivy_show`, or `ivy_to_cpp` is attempted, the hook rejects it with a message directing to the serena equivalents.

Use `/nct-check`, `/nct-compile`, and `/nct-model-info` commands as convenient shortcuts for the most common operations.
