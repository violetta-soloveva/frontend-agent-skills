---
name: fix-unit-test-frontend
description: Fix failing unit tests (Testing Library, on Vitest or Jest) in React frontend projects. Use whenever unit tests fail, whether the user asks to fix them, CI reports failures, or you see failures yourself after changing code (for example while implementing a feature). Decides first why each test fails (intended behavior change, regression in the code, or a broken or flaky test) and only then fixes the right side.
---

# Fixing frontend unit tests

A failing test means one of three things, and each needs a different fix:

| Cause | What it means | What to fix |
|---|---|---|
| **Intended change** | Someone changed the behavior on purpose. The test describes the old behavior. | The test |
| **Regression** | The behavior changed by accident. The test is right. | The code (after asking) |
| **Broken test** | The behavior is fine. The test fails for its own reasons: flaky timing, order dependence, real dates or time zone, a dependency upgrade, a change in the shared test setup. | The test |

The danger is the shortcut: changing the expected value to whatever the code now returns. That turns every regression into an "intended change" and makes the test useless. So decide the cause first, with evidence, and only then edit.

When the failures come from your own changes in this session (you implemented a feature and tests broke), you already know the intent: use it in step 3. A test that breaks because of behavior you changed on purpose for the task is an intended change; one that breaks in code you didn't mean to affect is a regression in your own work, so fix your code rather than the test.

## 1. Reproduce

1. Find the runner and scripts in `package.json` (Vitest or Jest). Run the failing tests: `npx vitest run <path>` or `npx jest <path>` (`-t "<test name>"` for one test). Save the error, the expected vs received values, and the stack.
2. If the user gave no target ("tests fail in CI", "fix the tests"), run the project's single-run test script (for example `npm run test:ci`, or `npm test -- --run`) and list every failing file first.
3. Run the failing file 3–5 times. If it passes sometimes, treat it as flaky and go to "Broken test" below. Also run it in random order to catch tests that depend on order (Vitest: `--sequence.shuffle`; Jest: `--randomize`).
4. Check whether it fails on the base branch (usually `develop` or `main`) too. If it does, it isn't caused by this branch. Fix it anyway unless the project's rules say otherwise (a failing suite blocks everyone), but say so in the report.

## 2. Find what changed

For each failing test, look at what changed in the code it covers and in the test itself since it last passed:

- `git diff <base>...HEAD -- <source file> <test file>` and uncommitted changes.
- `git log -p -n 5 -- <source file>` when the change came earlier (for example, a test that started failing on the base branch).
- Changes in the shared test folder (setup, global mocks, factories, render helpers) and in `package.json` / the lockfile (dependency upgrades, especially the UI library and Testing Library).
- The intent behind the change: commit messages, the task or ticket number in the branch name, the PR description if there is one.

## 3. Decide the cause

**Intended change** when all of these hold:

- the source change that breaks the test is deliberate and consistent (the new behavior is coded everywhere, not in one branch only);
- the task, commit message or PR describes the new behavior, or it plainly follows from them;
- the new behavior is something a user or product owner would want.

**Regression** when any of these hold:

- the change that breaks the test was a side effect of something else (a refactor, a shared helper, a dependency bump);
- the task says nothing about this behavior;
- the old behavior is still what the UI text, validation messages or other code expect.

**Broken test** when the behavior is unchanged and the test fails because of:

- timing: `getBy*` for async content, missing `await` on `user.*`, real timers;
- dates: real "now", the machine's time zone, a date that has since passed;
- order: state left by another test (global stores, localStorage, network mock handlers not reset);
- environment: a dependency upgrade that changed markup or class names, a jsdom API missing, a change in the shared test setup;
- the test asserting implementation details (props passed to mocks, call counts) that changed without any user-visible change.

**Unclear**: if the evidence doesn't settle it, stop and ask. Show the user the change, the failing assertion, and the two readings ("the new value is intended because…" / "this looks like a regression because…").

## 4. Fix the right side

- **Intended change**: update the test to describe the new behavior. Rename it if the old name now lies. If the change removed a behavior, delete the test that covered it, and say so. Add a test for any new branch the change introduced, using the project's testing conventions (the `add-unit-test-frontend` skill and the project's own test docs).
- **Regression**: don't touch the test. Report the regression with the change that caused it (`file:line`, commit) and a proposed fix in the code. Ask before editing source code: the fix may belong to another task or another person.
- **Broken test**: fix the cause, not the symptom. Use `findBy*` for async, `vi.useFakeTimers()` and `vi.setSystemTime()` for dates, explicit time zones, reset shared state in `beforeEach`. Replace implementation-detail assertions with what the user sees. If the cause is in the shared test setup, fix it there once rather than patching every test, and tell the user since it affects every test.

Never, without asking:

- change an expected value just to match the received one;
- loosen an assertion (`toEqual` → `toBeDefined`, exact text → regex) to get green;
- add `.skip`, `.todo` or delete a failing test;
- add a mock whose only purpose is to silence the failure;
- raise timeouts or add retries to hide flakiness.

## 5. Verify

1. Run the fixed files; all must pass. For flaky tests, run them 5 times, plus once in random order.
2. For an intended change: check the updated test fails against the old behavior (temporarily revert the source change, run, restore exactly). This proves the test now pins the new behavior.
3. Run `npx eslint <path>` on touched files.
4. If you changed anything in the shared test setup, run the full suite.

## 6. Report

One block per failing test:

- **Test**: `file` › test name
- **Cause**: intended change / regression / broken test / unclear, with the evidence (the change as `file:line` or commit, the task text)
- **What I did**: the fix, or "nothing, waiting for your answer"

Then, if any:

- **Regressions to fix in code**, with the proposed change, as questions.
- **Unclear cases**, with both readings, as questions.
- **Pre-existing failures** (also failing on the base branch).
- **Shared setup issues** found while fixing (duplicated helpers, mocks that could be global), as suggestions.
