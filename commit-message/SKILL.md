---
name: commit-message
description: Suggest commit message variants for the current changes, tagged with the task number from the branch. Use when the user asks for a commit message or what to write when committing.
---

# Commit message

The format and rules live in [`.cursor/rules/16-version-control.mdc`](../../../.cursor/rules/16-version-control.mdc). Read it first; every variant must pass it.

## Steps

1. **Find the task number.** Run `git branch --show-current`.
   - On `develop`, `staging`, `main`, `master`, `prod` or `production`: there is no task number. Variants end with the description alone.
   - Otherwise take the `US-<digits>` part of the branch name (`refactor/US-23900-tech-debt` → `US-23900`). If the branch has none, ask the user for the task number.
2. **Read the changes.** Use the staged diff (`git diff --cached`). If nothing is staged, use the unstaged diff and say so. Leave out files the user did not change in this task (ask when unsure). Done when you can name the one main change in a phrase.
3. **Write at least 3 variants**, each a different angle on the same change:
   - the plain *what* (`Replace table column types with ExtendedGridColDef US-23900`),
   - the *why* or effect (`Unify table column definitions US-23900`),
   - a shorter or more specific wording (`Use ExtendedGridColDef in all tables US-23900`).
4. **Check each variant**: imperative present tense, starts with a capital letter, no trailing period, no quote characters, task number last with one space before it.

## Output

Each variant in its own fenced block so it copies cleanly, with the recommended one first and marked. After them, list the files the commit should include if that differs from what is staged.
