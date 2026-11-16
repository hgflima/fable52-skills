# Worked examples

Three requests and the prompts they become. Read them for the reasoning, not as templates: each prompt has the shape its task needs, and the three shapes differ on purpose.

## Contents

1. A long unattended coding task
2. A research task with a report
3. An existing prompt to improve

## 1. A long unattended coding task

Request (in Portuguese): "preciso migrar nossos testes de jest pra vitest no monorepo inteiro. o repo é nosso, pode arrumar o que achar ruim no caminho. vou deixar rodando de noite."

What the request already answers: the outcome (Jest to Vitest, whole monorepo), ownership (theirs, restructuring welcome), and that nobody will watch. What a quick look at the repo answers: the package manager, the workspace layout, the test command. Nothing left to ask.

```text
Migrate this monorepo's tests from Jest to Vitest, in every package under `packages/`.

We are moving because Jest's startup time dominates our CI runs, so the result should run on Vitest alone. Leaving Jest in place for some packages would keep both toolchains alive and defeat the purpose.

You are the owner and long-term maintainer of this codebase, and its design is up to you. If you meet duplicated test helpers or setup files while you work, consolidate them.

The work is done when:
- `pnpm -r test` passes and runs on Vitest in every package.
- No package depends on `jest`, `ts-jest`, or `@types/jest`, and no Jest config file remains.
- The CI workflow in `.github/workflows/test.yml` calls the new command.
- No test was deleted or skipped to get a pass. If a test cannot be migrated, leave it failing and explain why in the report.

I will not be available while you work. Commit to a new branch named `vitest-migration`, one commit per package, so that I can review and revert package by package. Do not push the branch, open a pull request, or change anything outside this repository. Leave those for me.

# Finishing work
Please avoid stopping while you still owe work the user asked for. Be persistent and do not stop unless needed.

While status notes are welcome, as is clarifying with the user on open decisions, carry on with work that does not depend on the user's answer without unnecessary stops.

If you notice yourself inviting the user to redirect you or offering to wait, instead proceed on the next part of the task. You may meet ordinary obstacles like errors, timeouts, locked files, empty results or failing tools. Diagnose them first, and when they are not real blockers, work through them with the access you have (wait and retry, fix the request, use another tool or source) rather than stopping unnecessarily.

If someone clearly decided something is a hard blocker, such as a file marked not to be touched, access that was intentionally withheld, or a safety guardrail, leave it alone, say plainly what you found, and look for another safe way to finish the task. Stopping before the task is complete is very rarely merited, as in when the task can't move forward without user input, or where the blocker is deliberate and should not be worked past. This does not override the need for confirmation on risky or destructive actions.

When you finish, write a report I can read in the morning without having watched the run. Start with whether every package passes. Then list, per package, what changed beyond a mechanical migration: helpers you consolidated, tests you rewrote, and any change to non-test code. End with the tests you left failing and what each one needs. Write in plain full sentences, and do not use a name you made up during the work without saying what it is.
```

Notes given to the user:

- Effort: `high`. It is a long unattended run on hard coding work.
- Included the ownership sentence, because they asked for cleanup on the way. Anthropic measured about 70 to 80 percent more output tokens with it, and changes nobody asked for. Review the full diff of each commit, not only the report.
- Included the finishing-work text, because nobody can say "continue" overnight. About 30 percent more output tokens.
- Left out progress updates, because nobody is watching.
- The final report is asked for in three sentences and not with the full "Communicating clearly" text. The reader is one person and the prompt already says what the report must contain.

## 2. A research task with a report

Request: "compare the three main open-source vector databases for our RAG product, I need to recommend one to the CTO"

Missing, and worth asking because each answer changes the prompt: which three (or should the model pick), the scale and the constraints that decide the choice, and what the CTO wants to read. Three questions, asked once. Answers: Qdrant, Milvus, and pgvector; about 50 million vectors, self-hosted on Kubernetes, the team already runs Postgres; a one-page memo.

```text
Compare Qdrant, Milvus, and pgvector as the vector store for our retrieval-augmented generation product, and recommend one.

I will hand your memo to our CTO, who decides this quarter. She knows databases well and has not followed vector search closely. She needs a recommendation she can defend, so every claim that drives the recommendation must have a source she can check.

Our situation: about 50 million vectors of 1,536 dimensions today, expected to triple in two years. We self-host on Kubernetes. The team already operates Postgres in production and has nobody with experience running Qdrant or Milvus. Queries combine vector similarity with metadata filters on tenant and date.

Judge the three on what matters for that situation: query latency and recall at our scale with filtering, the operational work of running each one on Kubernetes, the cost of the hardware each needs, and how mature and actively maintained each project is. If you find another criterion that would change the decision, add it and say why.

Use primary sources where they exist: official documentation, release notes, and benchmarks whose method is published. Vendor benchmarks favour the vendor, so check each performance claim against a second, independent source, and say so when you could not. Prefer material from the last 12 months, because these projects change fast. Where the sources disagree or the evidence is thin, say that plainly. I would rather read "unknown at this scale" than a confident guess.

Write a memo of about one page. Open with the recommendation and the main reason in two or three sentences. Follow with a comparison table of the three on each criterion, then one short paragraph per option on its main risk for us. End with what would change your recommendation. Put the sources in a numbered list after the memo, and cite them by number.

Before you finish, check each number in the table against the source you cite for it.
```

Notes given to the user:

- Effort: `high`. Research with verification across sources.
- No Fable 5.2 blocks added. The task is one deliverable, so there is nothing to stop early on, and the memo format is specified directly.
- The prompt asks for sources and for "unknown" where evidence is thin. It does not ask the model to show its reasoning, which Fable 5.2 can decline.

## 3. An existing prompt to improve

Request: "melhora esse prompt pra mim" followed by:

```text
You are a senior engineer. IMPORTANT: think step by step and explain your full reasoning before answering. NEVER touch the database. You have 150,000 tokens, use them wisely. Delegate to subagents whenever possible. Add rate limiting to our public API. Keep every sentence under 12 words.
```

The task itself is clear. One thing is not: "never touch the database" could be a real ban or a wish to be asked first. That changes the prompt, so it is worth one question. Answer: schema changes need their approval, reads are fine.

```text
Add rate limiting to our public API.

The API has no limits today, and one client's retry loop took the service down last week. The goal is that no single API key can degrade the service for the others. Limit by API key, return HTTP 429 with a `Retry-After` header when a key is over its limit, and make the limits configurable per key without a deploy.

Start by reading how requests are authenticated and where middleware is registered, then choose the approach that fits what is already there. Tell me the approach in two or three sentences before you write code.

You may read from the database freely. Ask for my approval before any schema change or migration, and say what the change is and why you need it.

The work is done when the limit is enforced on every public route, there are tests for a key under its limit, a key over it, and the reset after the window, and the existing test suite passes. Run the tests before you finish.

Close with a short summary: the approach you took, the files you changed, and anything I need to configure.
```

Notes given to the user, on what was removed and why:

- "Think step by step and explain your full reasoning": thinking is always on in Fable 5.2, and a request to reproduce reasoning in the response can be declined. The prompt asks for the approach in a few sentences, which is a result and not the reasoning.
- "You have 150,000 tokens": a budget shown in the prompt changes how much the model spends. Anthropic found lower cost and equal results without it.
- "Delegate to subagents whenever possible": raises cost with no gain on a task that is one line of work.
- "Keep every sentence under 12 words": Fable 5.2 obeys this closely and the prose turns choppy. Replaced with a request for a short summary.
- "NEVER touch the database": the model treats "never" as absolute, which would also block reads. Rewritten as approval before schema changes.
- "IMPORTANT" in capitals: stated plainly, with the reason.
- Added what the original lacked: why the work matters, what done means, and a closing check.
- Effort: `medium` if they work on it interactively, `high` if they leave it running.
