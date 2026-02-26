---
name: Panther-Serena for Ivy
description: This skill should be used when the user asks about "using serena for Ivy", "Ivy tool guidance", "panther-serena tools", "how to check Ivy files", "how to compile Ivy", "formal verification tools", "ivy_check alternative", "MCP tools for Ivy", or mentions any Ivy toolchain operation in the PANTHER framework. Provides tool mapping from CLI to panther-serena MCP equivalents and correct usage patterns.
---

# Panther-Serena for Ivy

## Two-MCP Architecture

The panther-ivy-plugin exposes **two** MCP servers with distinct responsibilities:

| Server | Role | Prefix | Tools |
|--------|------|--------|-------|
| **panther-serena** | Code manipulation and navigation | `mcp__plugin_panther-ivy-plugin_panther-serena__` | find_symbol, replace_symbol_body, create_text_file, etc. |
| **ivy-tools** | Read-only diagnostics and analysis | `mcp__plugin_panther-ivy-plugin_ivy-tools__` | ivy_verify, ivy_lint, ivy_traceability_matrix, etc. |

**This skill covers panther-serena** (code manipulation).
For ivy-tools diagnostics, see the `ivy-tools-reference` skill.

## Overview

All Ivy code manipulation within the PANTHER framework MUST use panther-serena MCP tools instead of direct CLI commands. panther-serena provides semantic code navigation, editing, and file operations through a consistent MCP interface that integrates with Claude Code's tool system.

For verification and diagnostics (ivy_check, ivy_lint, coverage), use ivy-tools instead.

## Why panther-serena Instead of Direct CLI

- **Consistency** — All operations tracked through MCP tool calls with structured JSON output
- **Semantic navigation** — Symbol-level understanding of Ivy code (not just text search)
- **Integration** — Verification results consumed by agents and commands
- **Safety** — Operations run within project context, preventing accidental execution outside the project
- **Tool tracking** — All tool calls are logged and auditable

## Tool Mapping Table

### panther-serena (code manipulation)

| Operation | MCP Tool | Usage |
|---|---|---|
| `cat file.ivy` | `mcp__plugin_panther-ivy-plugin_panther-serena__read_file` | Read file contents |
| `grep pattern *.ivy` | `mcp__plugin_panther-ivy-plugin_panther-serena__search_for_pattern` | Search across files |
| Manual symbol lookup | `mcp__plugin_panther-ivy-plugin_panther-serena__find_symbol` | Find specific symbols |
| Manual file overview | `mcp__plugin_panther-ivy-plugin_panther-serena__get_symbols_overview` | List top-level symbols |
| Manual dependency trace | `mcp__plugin_panther-ivy-plugin_panther-serena__find_referencing_symbols` | Find references |
| Create new .ivy file | `mcp__plugin_panther-ivy-plugin_panther-serena__create_text_file` | Write new files |
| Edit .ivy file | `mcp__plugin_panther-ivy-plugin_panther-serena__replace_symbol_body` | Edit symbol bodies |
| Edit by regex | `mcp__plugin_panther-ivy-plugin_panther-serena__replace_content` | Fine-grained edits |

### ivy-tools (diagnostics) -- see ivy-tools-reference skill

| Direct CLI Command | MCP Tool | Usage |
|---|---|---|
| `ivy_check file.ivy` | `mcp__plugin_panther-ivy-plugin_ivy-tools__ivy_verify` | Formal verification |
| `ivyc target=test file.ivy` | `mcp__plugin_panther-ivy-plugin_ivy-tools__ivy_compile` | Test compilation |
| `ivy_show file.ivy` | `mcp__plugin_panther-ivy-plugin_ivy-tools__ivy_model_info` | Model introspection |
| Fast structural lint | `mcp__plugin_panther-ivy-plugin_ivy-tools__ivy_lint` | No subprocess, ms |

## Ivy Diagnostic Tools (via ivy-tools MCP)

Verification, compilation, and model inspection have moved to the **ivy-tools** MCP server. See the `ivy-tools-reference` skill for full parameter documentation.

Quick reference:
- `ivy_verify` — formal verification (`ivy_check`)
- `ivy_compile` — test compilation (`ivyc`)
- `ivy_model_info` — model introspection (`ivy_show`)
- `ivy_lint` — fast structural lint (no subprocess)
- `ivy_include_graph` — dependency graph
- `ivy_capabilities` — check tool availability
- `ivy_traceability_matrix` — RFC coverage matrix
- `ivy_requirement_coverage` — coverage stats by level
- `ivy_impact_analysis` — symbol edge analysis
- `ivy_extract_requirements` — parse RFC normative statements
- `ivy_cross_references` — graph neighborhood
- `ivy_query_symbol` — rich symbol info

## Standard Serena Navigation Tools

Use panther-serena tools for code navigation and manipulation:

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

The panther-ivy-plugin plugin includes a PreToolUse hook that blocks direct Ivy CLI calls in Bash. If a Bash command containing `ivy_check`, `ivyc`, `ivy_show`, or `ivy_to_cpp` is attempted, the hook rejects it with a message directing to the serena equivalents.

Use `/nct-check`, `/nct-compile`, and `/nct-model-info` commands as convenient shortcuts for the most common operations.
