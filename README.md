# Team Claude skills

Claude Code skills for frontend work, shared as one plugin, `team-skills`. Each skill is a folder with a `SKILL.md` inside.

## Skills

| Skill | What it does |
| --- | --- |
| `commit-message` | Suggests 3+ commit message variants with the `US-<number>` task from the branch (skipped on `develop`, `staging`, `main`, `prod`). |
| `pr-description` | Fills the repo's PR template for the current branch. |
| `resolve-vulnerabilities` | Fixes High/Critical npm vulnerabilities reported by osv-scanner or `npm audit`; lower ones only when easy. Asks before overrides, major bumps, or ignores. |
| `add-unit-test-frontend` | Writes and reviews unit tests (Testing Library on Vitest or Jest) in any React frontend; it maps the project's own test setup first: what to test, smallest test level, mocking at boundaries, flags components that need too many mocks. Also starts on its own after feature work to propose (not write) missing tests. |
| `fix-unit-test-frontend` | Fixes failing frontend unit tests: first decides if the failure is an intended behavior change (fix the test), a regression (report, fix the code after asking) or a broken/flaky test (fix the test), then fixes the right side. Also starts on its own when tests fail after a change. |

## Install (Claude Code, all repos)

Run once in Claude Code:

```bash
/plugin marketplace add violetta-soloveva/claude-custom-skills
/plugin install team-skills@team-skills
```

The plugin installs at user level, so the skills work in every repo. They show up as `/team-skills:<name>`. New skills appear after starting a new session.

To get updates, run:

```bash
/plugin marketplace update team-skills
```

## Changing the skills

- Add or remove a skill: add or delete its folder, then edit the `skills` list in `.claude-plugin/marketplace.json`. Only listed skills are part of the plugin.
- Bump `version` in `marketplace.json` whenever a skill changes, or installed copies won't update.
- Check the manifest with `claude plugin validate .` before pushing.

## Layout

```
<skill-name>/
  SKILL.md          # frontmatter: name (matches folder) + description
.claude-plugin/
  marketplace.json  # publishes the listed skills as the team-skills plugin
```
