# Frontend agent skills

Agent skills for frontend work, for Claude Code and Codex. Each skill is a folder with a `SKILL.md` inside and is its own plugin; the `team-skills` plugin installs all of them at once.

## Skills

| Skill | What it does |
| --- | --- |
| `commit-message` | Suggests 3+ commit message variants with the `US-<number>` task from the branch (skipped on `develop`, `staging`, `main`, `prod`). |
| `pr-description` | Fills the repo's PR template for the current branch. |
| `resolve-vulnerabilities` | Fixes High/Critical npm vulnerabilities reported by osv-scanner or `npm audit`; lower ones only when easy. Asks before overrides, major bumps, or ignores. |
| `add-unit-test-frontend` | Writes and reviews unit tests (Testing Library on Vitest or Jest) in any React frontend; it maps the project's own test setup first: what to test, smallest test level, mocking at boundaries, flags components that need too many mocks. Also starts on its own after feature work to propose (not write) missing tests. |
| `fix-unit-test-frontend` | Fixes failing frontend unit tests: first decides if the failure is an intended behavior change (fix the test), a regression (report, fix the code after asking) or a broken/flaky test (fix the test), then fixes the right side. Also starts on its own when tests fail after a change. |

## Install (Claude Code, all repos)

Add the marketplace once in Claude Code:

```bash
/plugin marketplace add violetta-soloveva/frontend-agent-skills
```

Then install everything:

```bash
/plugin install team-skills@team-skills
```

Or only the skills you want, one at a time:

```bash
/plugin install commit-message@team-skills
```

Each skill is installed once either way, so installing a single skill and later the whole set doesn't duplicate anything. Plugins install at user level, so the skills work in every repo. They show up as `/<skill>:<skill>`, for example `/commit-message:commit-message`. New skills appear after starting a new session. You can also run `/plugin` and pick from the list.

The `team-skills` bundle needs a recent Claude Code; run `claude update` if the install fails.

To get updates, run:

```bash
/plugin marketplace update team-skills
```

## Using with Codex

The skills use the open Agent Skills format, so Codex can run them too. The `/plugin` install above is Claude Code only; in Codex, copy the skill folders instead.

1. Clone the repo:

   ```bash
   git clone https://github.com/violetta-soloveva/frontend-agent-skills.git
   ```

2. Copy the skill folders you want (each folder with its `SKILL.md`) into one of:
   - `~/.agents/skills/<skill-name>/` for all projects (Windows: `C:\Users\<name>\.agents\skills\<skill-name>\`)
   - `<repo>/.agents/skills/<skill-name>/` for one repo only

   For example, on macOS or Linux:

   ```bash
   mkdir -p ~/.agents/skills && cp -r frontend-agent-skills/commit-message ~/.agents/skills/
   ```

3. Restart Codex.

Copied skills don't update on their own: pull the repo and copy them again to get changes. Skip the `bundle/` and `.claude-plugin/` folders; they are for Claude Code only.

## Changing the skills

- Add a skill: add its folder, add an entry for it to `plugins` in `.claude-plugin/marketplace.json` (`"source": "./<skill-name>"`), and add its name to `dependencies` in `bundle/.claude-plugin/plugin.json`. Bump the bundle's `version` in both files.
- Remove a skill: do the reverse.
- Change a skill: bump that skill's `version` in `marketplace.json`, or installed copies won't update.
- Check with `claude plugin validate .` and `claude plugin validate ./bundle` before pushing.

## Layout

```
<skill-name>/
  SKILL.md          # frontmatter: name (matches folder) + description
bundle/
  .claude-plugin/plugin.json  # team-skills: depends on every skill, installs them all
.claude-plugin/
  marketplace.json  # lists each skill as a plugin, plus the team-skills bundle
```
