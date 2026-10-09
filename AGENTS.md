# AGENTS.md — Guide for AI Agents Working on This Project

## Dependency Management
- Use `uv` for all package operations.
- Install dependencies: `uv add <package>==<version>`
- Run the app: `uv run python calculator/main.py`
- Run tests: `uv run pytest tests/`

## File Operations
- Use `pycharm-mcp` tools for all file operations. Do NOT use bash-based file manipulation (no `cat`, `echo`, `sed`, etc. for creating/editing files).
- **Read a file:** `mcp_pycharm-mcp_read_file` — pass `file_path` (project-relative), optional `offset` (1-based line) and `limit` (max 5000 lines).
- **Create a new file:** `mcp_pycharm-mcp_create_new_file` — pass `pathInProject` (project-relative) and `text` content.
- **Search for files:** `mcp_pycharm-mcp_search_file` — glob pattern search (e.g., `"**/*.py"`), optional `paths` filter and `limit`.
- **Search text:** `mcp_pycharm-mcp_search_text` — fast substring search across project files, optional `paths` filter.
- **Search regex:** `mcp_pycharm-mcp_search_regex` — PCRE regex search with match coordinates, optional `paths` filter.
- **Rename symbol:** `mcp_pycharm-mcp_rename_refactoring` — rename a symbol across all references; pass `pathInProject`, `symbolName`, and `newName`.

## Code Quality
- **Reformat code:** `mcp_pycharm-mcp_reformat_file` — pass a list of project-relative `files` to reformat after edits.
- **Check a single file for problems:** `mcp_pycharm-mcp_get_file_problems` — analyze a file for IDE-reported errors and warnings; use after edits to catch syntax or type issues before running tests.
- **Lint multiple files:** `mcp_pycharm-mcp_lint_files` — lint a list of project-relative files in one call; use when batch-checking edited files for style or correctness issues.
- **Analyze call hierarchy:** `mcp_pycharm-mcp_analyze_calls` — build a call tree (incoming/outgoing) for a symbol; use before refactoring to identify all callers and callees that may be affected.
- **Build project:** `mcp_pycharm-mcp_build_project` — trigger project build and return compilation errors; use after edits to validate code compiles correctly.
- **Run tests/app:** `mcp_pycharm-mcp_execute_run_configuration` — execute a run configuration by name or code location; use for running tests or the application.
- **List run configurations:** `mcp_pycharm-mcp_get_run_configurations` — discover available run configurations or executable entry points in a file.
- **Lint files:** Prefer `mcp_pycharm-mcp_lint_files` or `mcp_pycharm-mcp_get_file_problems` over bash-based linting commands.

## Context Budget Awareness
- Keep edits atomic and small to fit within worker subagent context budgets (128k context window).
- One task per worker delegation; do not bundle multiple changes.
