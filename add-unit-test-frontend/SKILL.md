---
name: add-unit-test-frontend
description: Add or review unit tests (logic and component tests with Testing Library, on Vitest or Jest) in React frontend projects. Use when the user asks to add, write or review tests for a file, function, component or hook, and also after you implement a feature or bug fix whose new or changed logic has no tests, to propose what is worth covering. With a named target it writes the tests; otherwise it proposes and asks before writing. Not for tests that already fail (use fix-unit-test-frontend).
---

# Writing frontend unit tests

Frontend "unit tests" here are two kinds:

- **Logic tests**: pure functions and hooks (data transformation, dates, validators, mappers).
- **Component tests**: one component or form rendered and used the way a user would (typing, clicking, selecting), with the API mocked.

Multi-page flows, real backend contracts and real-browser behavior (layout, scrolling, downloads) are out of scope: they belong to e2e tests or manual QA. If a request needs one of those, say so instead of forcing it into a unit test.

The goal is not maximum coverage. A test earns its place when it protects code that is likely to break and would otherwise cost QA manual checks.

This skill adds and reviews tests. Fixing tests that already fail is a different job (first deciding whether the code or the test is wrong), covered by the `fix-unit-test-frontend` skill. If existing tests fail while you work here, don't adjust them to pass: report them and suggest `/fix-unit-test-frontend`.

The skill works across projects, so it names concepts, not files. Step 1 maps each concept to the current project. The examples in parentheses come from one project and only show what to look for.

## 0. Pick the mode

How the skill was invoked decides whether you write tests right away or propose first.

**Target given**: the user names a file, folder, function, component or hook ("add tests for `bulkValidationErrors.ts`", "test `useDialog`"), or asks to review specific test files. They've already decided what to cover, so go ahead: steps 1–8 below. You still stop and ask when step 3's testability check says the component needs a split.

**No target**: the skill is invoked bare, with something like "check my changes" or "what should I test here", or you started it yourself after implementing a feature or fix (the user didn't ask for tests). Don't write tests yet. The user hasn't decided what's worth covering, and writing tests for every changed file produces exactly the low-value tests this skill is meant to prevent. Instead:

1. Find the base branch (usually `develop` or `main`; check the remote's default branch). Find the changed files: `git diff --name-only <base>...HEAD` plus uncommitted changes (`git status --porcelain`). On the base branch itself, use only the uncommitted changes. Ignore tests, styles, assets, generated files and config.
2. Read each changed source file and its existing test, if any, and look at what actually changed in it (`git diff <base>...HEAD -- <file>`), not just the whole file.
3. Classify each file with step 2's "what to test" rules:
   - **Should be covered**: the change adds or alters validation, brittle logic (dates, drag and drop, table editing) or branching logic. Name the specific behaviors (one line each) and the test level from step 3 (pure function, hook, component).
   - **Covered already**: an existing test exercises the changed behavior. Say which test.
   - **Not worth testing**: name the reason (trivial render, styling, library behavior, pass-through).
   - **Needs a split first**: the testability check (step 3) fails. Name the pure function or split you'd propose.
4. Reply with that list, grouped by category, plus any "Shared setup to move" or "Mocks that could be global" findings you noticed while reading. End by asking which ones to write. Write nothing until the user answers.

Keep the proposal short: one line per behavior, not a test plan. If nothing in the changes is worth testing, say so plainly.

When you started the skill yourself after feature work, limit the proposal to the files changed in that work, not the whole branch, and keep it to a few lines at the end of your reply. If those changes contain nothing worth testing, skip the proposal entirely instead of reporting "nothing to test".

## 1. Map the project's test setup

The project is the source of truth. This skill only says how to use it. Before writing anything, find:

1. **Test docs and rules**: a testing section in `README.md`, `AGENTS.md` / `CLAUDE.md`, `.cursor/rules`, team docs linked from them. They may define naming templates, what to test, and run commands. Where they disagree with this skill, the project wins.
2. **Runner and config**: Vitest or Jest (`package.json`, `vite.config.*`, `vitest.config.*`, `jest.config.*`), the `test` scripts, the environment (jsdom, happy-dom), whether globals are on, and the setup files. Examples below use Vitest (`vi.*`); use the `jest.*` equivalents in a Jest project.
3. **The shared test folder** (often `src/tests/`, `src/test-utils/`, `test/`). Look in it for:
   - the **custom render** that wraps components in the app's providers (router, query client, theme, modals), and any variant that also sets up `userEvent`;
   - a **hook render helper** with the same providers;
   - the **global setup file**: what it resets between tests (mocks, storage, network handlers);
   - **global module mocks** applied to every test, including hooks wrapped in a spy around the real implementation;
   - **factories** for test data (`createEvent(overrides)`), **form test values** and **form error helpers**;
   - **network mocks** (MSW handlers) and **browser API stubs** jsdom lacks.
4. **The closest existing test** to the code under test (same folder or feature), so the new test matches local style.
5. **Reference tests**, if the docs name any as the good examples. If the docs say parts of the suite were AI-generated or are known to be weaker, reuse their setup ideas but don't copy their patterns blindly.

If the project has no tests yet, say so, propose a minimal shared setup (custom render, setup file) and ask before creating it.

### Shared setup: reuse, don't repeat

The shared test folder usually exists because helpers kept being copied between test files. Don't let them drift back.

- Before writing any helper in a test file (a `createX`/`makeX` factory, a render wrapper, a query client, a storage or browser API stub, a network handler, form test values), check whether the shared test folder already has it. If it does, use it. If it almost does, extend it there (for example, add an override or a new factory next to the existing one).
- Don't repeat what the global setup and global mocks already do (clearing mocks, resetting storage, mocking ids, starting the network mock server).
- If the new test needs a helper that another test file already defines locally, don't copy it. Put a shared version in the shared test folder and use that in the new test.
- Leave other test files alone (unrelated edits mix the task with changes nobody asked for). If you notice they define their own copy of something that exists in the shared folder, or that the same local helper or the same `vi.mock` setup appears in several files, report it (see step 8). Don't fix it.

A quick way to find hotspots: list local factory names across test files (`grep -rhoE "^const (create|make)[A-Z]\w*" --include=*.test.*`) and `vi.mock` targets that appear in three or more files.

## 2. Decide whether and what to test

Test:

- validation (schemas, validators, error messages, server validation errors mapped to fields)
- brittle code (dates and time zones, drag and drop, table editing)
- complex logic (data transformation, conditional rendering, state flows)
- a bug fix: write the regression test first and see it fail, then fix

Skip:

- UI library behavior (MUI, Chakra, Ant, etc.) and other third-party code
- pure styling and layout (unless the style depends on data, then mock the data)
- trivial render-only components and pass-through utils
- implementation details (internal state, hook call counts, which props a child received)

If the user asks for tests on something in the skip list, say why it's low value and suggest what to test instead. Don't silently refuse.

## 3. Pick the smallest level that proves the behavior

From cheapest to most expensive:

1. **Pure function** test: no render, no mocks. Fastest and most stable.
2. **Hook** test: `renderHook` from Testing Library when the hook needs no providers, the project's hook render helper when it needs the app's providers. Don't build a local `Wrapper` for these. If the project has no hook render helper, ask the user before adding one to the custom render file, modeled on the component render (it changes a shared file). If a hook needs another provider (a form, a feature store), wrap it in the test and consider whether it belongs in the shared folder.
3. **Component** test with the custom render.
4. **Page** test with network mocks. Use this only where the project already does it or for a flow that can't be tested smaller.

If the behavior is a decision ("which actions are available for this row", "is this time valid"), test it at level 1. Render only to prove the user-visible wiring, and only once.

### Testability check

Before writing a component test, list what it needs to render. If it needs **more than ~3 `vi.mock` calls of the project's own modules** (hooks, contexts, child components, mappers), stop and treat it as a design signal, not a mocking task:

- The logic you want to test is probably mixed with wiring (routing, queries, context, grid or table APIs).
- Propose a split to the user, for example:
  - extract the decision into a pure function next to the component (`getRowActions(row, { editModeActive })`) and test it with `it.each`;
  - split into a container (hooks, context, navigation) and a presentational component that takes values and callbacks as props, then test the presentational one with plain props;
  - render real providers (with a small test wrapper) instead of mocking context hooks.
- Refactoring code the user didn't ask to change needs their OK, so describe the split and ask before doing it.
- Ask **before** writing the component test, not after. A large suite built on mocks or heavy provider setup is mostly thrown away if the user agrees to the split, so don't write it first. While waiting, do only what doesn't depend on the answer, for example tests for pure functions that already exist.
- If the user declines the split, write the test with mocks (or real providers) and mention the cost.

When designing a new component, run this check up front: write the list of test cases first and shape the component so each case needs as few mocks as possible.

### When a test is slow

Measure before changing anything. Vitest prints `transform`, `setup`, `import` and `tests` times per run (Jest: run with `--verbose` and compare per-test times to the file's total). Import time usually dominates: heavy UI libraries (data grids, charts, editors), feature barrels (`index.ts`) and providers can take seconds, while the tests themselves take under one. Removing mocks doesn't fix that, and neither does adding them.

What helps:

- Put extracted logic in a file that imports only models and utils, never UI libraries, providers or barrels. A pure function in the same file as the component still pays for the component's imports.
- Import from the specific file, not a barrel, both in the new logic file and in the test.
- Keep the component test small (only the wiring a render can prove), so the slow import is paid once for a few tests.

A first run after a cache clear can take much longer because of transforms. That is not specific to one file, so don't report it as the file being slow.

## 4. What every test file covers, in order

1. The main happy path.
2. Validation and error paths: invalid input, empty required fields, API error responses (400/500), server validation errors shown on fields.
3. Boundaries: empty lists, `null`/missing values, min/max dates, time zone edges.
4. Each branch of conditional rendering that a user can see (e.g. edit mode vs view mode, cancelled vs active).

Use `it.each` when cases differ only by input. Don't add cases that repeat what another test already proves.

## 5. Mocking policy

Mock at the boundaries, keep everything else real. Where the project has an established way of mocking something, follow it so new tests look like existing ones.

- **Server data in component and hook tests**: mock the data hook (React Query, SWR, RTK Query) the way existing tests do. In Vitest, `vi.mock(path, { spy: true })` + `vi.mocked(useXQuery).mockReturnValue(...)` keeps the real module shape and is a good default.
- **Network in page and flow tests**: network mocks (MSW handlers; shared ones in the shared folder, per-test overrides with `server.use(...)`). Don't mock API service modules directly.
- **Time**: `vi.useFakeTimers()` / `vi.setSystemTime()`. Never wait with real timeouts.
- **Dates**: use fixed dates, freeze "now" with `vi.setSystemTime`, and pass an explicit time zone. Unless the config pins `TZ`, anything that relies on the machine's zone can pass locally and fail in CI. When the logic crosses days (ranges, all-day items, "next day" rules), add a DST case, such as a range across the US or EU daylight saving switch.
- **Randomness and ids**: check whether they're already mocked globally; if not, mock them where the test needs stable output.
- **Browser APIs jsdom lacks**: add them to the shared browser stubs, not per file.
- **Any other project hook**, when you must: the same `{ spy: true }` pattern.

Don't mock:

- the module under test
- pure helpers and mappers it uses (they're fast and real behavior is the point)
- a child component just to assert which props it received; assert on what the user sees instead
- things the global mocks already cover

Don't repeat what the global setup already does (for example a global `vi.clearAllMocks()` before each test).

### Pre-spied globals

Some projects wrap commonly controlled hooks (modals, snackbars or toasts, the query client) in a spy around the real implementation in the global mocks. Those behave normally, and any test can control them without its own `vi.mock`:

```ts
const showSnackbar = vi.fn();
vi.mocked(useSnackbar).mockReturnValue({ showSnackbar, hideSnackbar: vi.fn() });
// ...act...
expect(showSnackbar).toHaveBeenCalledWith(/* ... */);
```

Use them to check "the dialog opened" or "the success toast showed". Don't `vi.mock` those modules again in a test file (including through a barrel that re-exports them).

### Repeated mocks that could be global

If the test you're writing needs a `vi.mock` that other test files also set up the same way (same module, same mock shape), don't copy it in silently. Write the test, then ask the user in the report whether it should move to the global mocks, naming the module and the files that repeat it. For example: "`src/hooks/useTypedParams` is mocked the same way in 4 files. Should it become a global spy?" Making it global changes every test, so it's the user's decision, not yours. Prefer the spy-around-the-real-implementation form, so tests that don't care still get real behavior.

## 6. Writing the test

- Render with the project's custom render; interact with `userEvent` (`const user = userEvent.setup()`, or the project's helper that returns `user`).
- Queries: `getByRole` → `getByLabelText` → `getByText` → `getByTestId` (last resort). `findBy*` for anything that appears async.
- Test data from the shared factories with overrides, not inline objects cast with `as unknown as`.
- Naming: use the project's template if it has one. Otherwise `should <verb> <observable outcome> when <trigger> [and <precondition>]`, in user language ("shows", "saves", "disables"), not implementation ("sets state", "calls hook").
- `describe` per component or function, nested `describe` for context (`when row is in edit mode`), one behavior per `it`.
- Imports: follow what existing test files do (explicit `import { describe, it, expect, vi } from 'vitest'` or globals).
- **Form validation**: use the project's form test values and error helpers if it has them (for example a `getHelpText(fieldName)` helper and an error class constant), and add new values next to the existing ones instead of inline strings. Cover both sides: the error shows for invalid input and clears for valid input.
- **Accessibility gaps**: if an element can only be found with `getByTestId` (for example an icon button with no `aria-label`), use the test id for now and report the missing accessible name. Don't edit the component to add it unless asked.
- **New shared helpers** follow the project's documentation rules (for example JSDoc on every export).
- **Domain naming**: if the project's UI text uses different words than the code (code says `event`, UI says "meeting"), queries match the UI text.
- Follow the project's lint rules.

## 7. Anti-patterns

Each of these looks plausible and proves nothing or breaks on refactors:

- Asserting a mocked child got a prop (`expect(StatusMenu).toHaveBeenCalledWith(expect.objectContaining({ onEdit: undefined }))`). Test the decision as a pure function, or open the menu and check the item.
- Mocking a mapper or util to return `{}` so the component renders.
- Snapshot tests instead of behavior checks.
- `getBy*` for async content (flaky), or `waitFor` around a `findBy*`.
- Tests that depend on order or on state left by a previous test.
- Re-testing the same branch at the component level that a pure function test already covers.
- Tests named "works correctly" or "renders" with no outcome.
- A local factory, wrapper or stub that duplicates one in the shared test folder, or the same helper copied between test files.

## 8. Before finishing

1. Run only the files you touched (`npx vitest run <path>` or `npx jest <path>`). All must pass. For a bug fix, confirm the new test failed before the fix.
2. Check that the key tests can fail. For the 2–3 most important behaviors, break the rule in the source on purpose (flip a condition, drop a guard), run the tests, confirm at least one fails, then restore the source exactly. A test that stays green when its rule is broken asserts nothing, so fix it. Mention in the report which rules you checked this way.
3. Run the linter on them (`npx eslint <path>`) and fix issues.
4. Report back briefly:
   - what the tests cover (one line per `describe`)
   - what you chose not to test and why
   - any testability issue from step 3 (mock count, suggested split) if you didn't fix it
   - missing accessible names you worked around with `getByTestId`
   - **Mocks that could be global**, as questions to the user (see "Repeated mocks that could be global").
   - **Shared setup to move**, if you found any: each duplicate as `file:line`, what it duplicates (an existing shared helper, or the same helper in other files), and where it should live in the shared folder. Leave this section out when there's nothing to report.

## 9. Reviewing tests

When the task is to review tests ("review the tests in X", "check this test file"), report only. Don't edit the test files, the shared test folder or the source, even when a fix looks obvious. The user asked for an opinion, and edits would mix the review with changes they haven't agreed to.

Check the files under review against everything in this skill (what's worth testing, mocking policy, naming, anti-patterns, weak assertions, missing cases), against the shared test folder and against each other. You may run the tests, and you may break a rule in the source temporarily to see whether a test catches it, as long as you restore it exactly.

Structure the review as:

- **Issues**: each as `file:line`, what's wrong and the suggested fix, most important first.
- **Missing cases**: behaviors worth covering that no test checks.
- **Mocks that could be global** and **Shared setup to move**, as in step 8.
- **Possible bugs in the source** noticed along the way, as questions.

End by asking which fixes to apply.
