Fix the flaky tests in the checkout service.

## Context

Some tests in the checkout service pass and fail intermittently with no code change, which makes CI results untrustworthy. The goal is a suite that passes reliably because the source of nondeterminism is gone, not one where the flakiness is hidden.

[If known, list here: which tests flake, how often, whether it happens only in CI or also locally, and the command that runs the suite.]

## What a good fix looks like

Reproduce each flaky test before changing it: run it repeatedly, in isolation and together with the rest of the suite, in random order if the runner supports it. A fix for a failure you never observed is a guess.

Fix the cause of the nondeterminism: shared state between tests, order dependence, real clocks or timers, unmocked network calls, random data, race conditions in async code, and so on. Retries, skips, longer sleeps or timeouts, and loosened assertions hide the problem instead of fixing it, so avoid them unless one is genuinely the correct fix, and say so in the report when it is.

If a test is flaky because the checkout code itself has a real bug (for example a race condition), fix the code when the change is small and contained. If it needs a larger change, leave it and describe it in the report.

Verify before reporting: run each fixed test at least 20 times in a row, then run the full checkout test suite once.

## Constraints

Do not delete existing files in the repository, including test files, fixtures and snapshots. [State the reason here, e.g. "other teams depend on these fixtures".] If you conclude that deleting a file is the right fix, leave the file in place and recommend the deletion in the report. Cleaning up temporary files you created yourself is fine.

Keep changes inside the checkout service and its tests. If the cause lives elsewhere (shared test utilities, CI configuration, another service), report it instead of changing it.

## Working style

Do small or sequential work yourself. Use subagents only where the work is independent and can run in parallel, such as investigating several unrelated flaky tests at once.

Take the time the investigation needs. If the job turns out to be much bigger than expected, fix what you can fix properly and report the rest, rather than applying shallow fixes everywhere.

## Final report

When you finish, write a short report in plain prose covering:

- for each flaky test: the root cause, what you changed, and how you verified it (including how many runs passed)
- anything you could not fix or chose not to fix, and why
- the list of files you changed
