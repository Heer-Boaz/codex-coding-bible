---
name: format-fix
description: "Use when fixing indentation, whitespace, formatter drift, formatting checks, Prettier/Biome/clang-format/rustfmt/go fmt/stylua output, or other purely mechanical style failures in any project. Use only project-local formatting tools and do not mix formatting with semantic edits unless the user asks for both."
---

# Format Fix

Use this for mechanical formatting and indentation repair.

## Rules

- Use the project-local formatter, config, and package manager.
- Prefer documented scripts such as `format`, `fix:format`, `fix:indent`,
  `check:format`, `check:indent`, or language-native formatter commands.
- Do not introduce a formatter, config, or style policy unless asked.
- Do not rewrite code by hand when a formatter already owns the syntax.
- Do not mix semantic cleanup with formatting-only work.
- If formatting exposes syntax or parser errors, fix only what is required for
  the formatter to run unless the user asked for broader repair.

## Workflow

1. Inspect package scripts, formatter configs, task files, and existing docs.
2. Run the narrowest check command when available.
3. Run the matching fix command on the intended scope.
4. Re-run the check command.
5. Run `git diff --check`.

If no local formatter exists, report that directly. Do not invent one in a repo
that has no formatter contract.
