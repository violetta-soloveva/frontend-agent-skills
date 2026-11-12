---
name: pr-description
description: Fill in this repo's PR template for the current branch and output it ready to paste into the GitHub PR description. Use when the user asks for a PR description, PR body, or to fill the PR template.
---

Generate the PR description for the current branch, following `.github/pull_request_template.md`.

## Gather

The description covers only what this PR adds, so first pick the base branch:

1. The user named a base: use it.
2. Otherwise, if the branch has an open PR, use the PR's base. PRs are often stacked on another feature branch, not `develop`:

   ```bash
   gh pr view --json number,baseRefName,url
   ```

3. Otherwise (no PR, or `gh` fails): use `develop`.

Compare against the remote copy so a stale local branch does not pull in extra commits. A branch passed as an argument is the head branch, not the base:

```bash
git fetch origin <base>
git branch --show-current
git log origin/<base>..HEAD --oneline
git diff origin/<base>...HEAD --stat
git diff origin/<base>...HEAD
```

Describe only those commits. If the base is not `develop`, do not describe work from the base branch, even when it is the same task.

Take the task number from the branch name or commit messages (`US-<number>`). If none is found, ask for it.

## Output

Print the filled template inside one fenced `markdown` block so the user can copy it as-is. Print nothing else except a one-line note if something needs the user's input (for example, why tests were skipped).

```markdown
## Task Number

US-<number>

## Description

<only if needed, see below>

## Tests Status

<one status line, see below>
```

### Task Number

Just `US-<number>`. Drop the example and helper text.

### Description

- Small fixes (typo, one-line fix, version bump, obvious change from the title): drop the whole `## Description` section.
- Otherwise, write 1–5 short bullets on what changed and anything reviewers should know: non-obvious decisions, behavior changes, follow-ups, manual testing notes. Use the domain language of the code. Do not list every file or restate commit messages.

### Tests Status

Keep exactly one line, drop the rest and the helper text:

- `:white_check_mark: Tests added` — only new tests: new `*.test.ts(x)` files, or new cases added to existing files without changing existing cases.
- `:white_check_mark: Tests updated` — existing tests changed, moved or removed, with or without new tests.
- `:x: Tests are not required` — no test changes and none are needed (config, styles, deps, docs). Add one line explaining why.
- `:warning: Tests skipped` — logic changed but no tests touched. Add one line explaining why. If the reason is not clear from the diff, write `<reason>` and ask the user.

The statuses are exclusive: never output more than one. `Tests updated` already includes any new tests.

### PR Creator Checklist

Always drop this section. It is a reminder only.
