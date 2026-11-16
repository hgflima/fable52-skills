Fix the flaky tests in the checkout service: the tests that pass on some runs and fail on others with no change to the code.

The goal is a test suite whose failures can be trusted, so that a failing test means something is broken. That goal rules out the shortcuts. Do not make a test pass by deleting it, skipping it, loosening what it asserts, wrapping it in automatic retries, or lengthening a sleep or timeout until the failure becomes rare. Each of those hides the cause and leaves it in place.

Start by finding out how this repository runs the checkout service's tests (the README, the package scripts, or the CI configuration will say) and which tests are flaky. Use the CI history if you can reach it, and run the suite repeatedly to see the failures for yourself. For each flaky test, reproduce the failure and find its cause before you change anything.

Fix the cause where it is. If it is in the test, fix the test. If it is in the service code, for example a real race condition that the test exposes only sometimes, fix it there, keep that change narrow, and put it first in your recap, because a change to checkout behaviour needs a closer review than a change to a test.

Treat this as code that other people maintain and will review. Keep the diff as small as the fixes allow. If you find other problems nearby, list them at the end and leave them as they are.

You may edit files and run tests as often as you need without asking. Ask for my approval before you delete any file, and tell me which file and why. The exception is temporary files that you created during this task: remove those when you are done.

The work is done when:
- Every test you identified as flaky passes on many consecutive runs. Choose the number of runs from the failure rate you measured before the fix, so that the old failure would almost certainly have appeared. For example, a test that failed about 1 run in 20 needs about 100 clean runs.
- The full test suite of the checkout service passes.
- No test was deleted, skipped, or weakened.

If you cannot reproduce a failure, or cannot find the cause of one, say so and leave that test as it is. Do not guess at a fix.

Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own (what you found, what you did, and what's next), so a reader who only sees the last message has the full picture.

In the recap, give for each flaky test: its name, the cause and the evidence that showed it, the change you made, and how many runs passed in a row afterwards. Then list the files you changed, and the problems you noticed and left alone.
