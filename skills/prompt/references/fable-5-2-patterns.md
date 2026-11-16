# Fable 5.2 prompting patterns

Reference for the `fable52:prompt` skill. It holds the texts Anthropic published for Claude Fable 5.2 and Claude Mythos 5.2, the condition under which each one belongs in a prompt, and what each one costs.

Sources (read on 2026-11-16):

- Prompting Claude Fable 5.2: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-2
- Prompting best practices: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- What's new in Claude Fable 5.2: https://platform.claude.com/docs/en/models/fable-5-2/whats-new-fable-5-2
- Migration guide: https://platform.claude.com/docs/en/models/fable-5-2/migration-guide
- Effort: https://platform.claude.com/docs/en/build-with-claude/effort

One caveat applies to every number in this file. Anthropic measured these texts in system prompts, on its own tasks. This skill places them in a task prompt (a user message). The direction of each effect should hold there, but the size is not measured. Report the numbers to the user as Anthropic's figures, not as a promise.

Copy the quoted texts word for word unless a sentence clearly does not fit the task. Anthropic calls them starting points to adapt, so dropping a sentence that does not apply is fine. Rewording a sentence that does apply loses the tested wording for no gain. Some of the texts say "the user" where the rest of a task prompt says "I". Leave that as it is. The model understands that the user is the author of the prompt.

## Contents

1. Effort
2. Finishing work
3. Approvals and caution
4. Code ownership
5. Progress updates
6. Subagents
7. Final reports
8. Lines to remove
9. Refusals
10. API facts that limit what a prompt can ask for
11. General techniques and optional texts

## 1. Effort

Effort is an API setting (`output_config.effort`), not prompt text. It sets how much the model thinks, how many tool calls it makes, how long a turn takes, and how many tokens it uses. Recommend a level next to every prompt.

| Level | Use it for |
|---|---|
| `low` | Quick back-and-forth, well-specified tasks, high-volume work, fast subagents. |
| `medium` | Interactive coding, chat, everyday knowledge work. |
| `high` | Hard coding, research, agentic work, long unattended runs. This is the API default. Turns take noticeably longer than at `medium`. |
| `xhigh` | The hardest long-running coding and agentic work, where a gain has been measured. Uses many more tokens for a slight gain. |
| `max` | Measuring the model's ceiling, or open-ended problems such as scientific discovery and mathematics where time and tokens do not matter. |

Facts to rely on:

- Most of the capability gain is present at `high`. Above it, gains are uneven and depend on the task.
- At `high` and above, the model thinks before almost every reply, including a greeting. For easy requests, a lower level is the fix.
- A prompt line that asks for less thinking does not stop the model from thinking at `medium` and above. Lower the level instead.
- The model is more likely to stop early at `low` and `medium`. For agentic work at those levels, add the text in section 2.
- Effort has little effect on reply length. Use the prompt to set length and tone.
- A setting tuned on an earlier model does not carry over. The levels rank the same way, but the amount of thinking at each level differs.
- At `high` and above, the caller should set a large `max_tokens`, because thinking counts toward it.

## 2. Finishing work

Behaviour: on long, multistep tasks the model sometimes ends a turn before the work is done. It lists what is left, offers to continue, or stops at an ordinary obstacle such as a timeout. This happens more often at `low` and `medium`.

Add this text when the task is long and multistep, and above all when nobody will be there to say "continue":

```text
# Finishing work
Please avoid stopping while you still owe work the user asked for. Be persistent and do not stop unless needed.

While status notes are welcome, as is clarifying with the user on open decisions, carry on with work that does not depend on the user's answer without unnecessary stops.

If you notice yourself inviting the user to redirect you or offering to wait, instead proceed on the next part of the task. You may meet ordinary obstacles like errors, timeouts, locked files, empty results or failing tools. Diagnose them first, and when they are not real blockers, work through them with the access you have (wait and retry, fix the request, use another tool or source) rather than stopping unnecessarily.

If someone clearly decided something is a hard blocker, such as a file marked not to be touched, access that was intentionally withheld, or a safety guardrail, leave it alone, say plainly what you found, and look for another safe way to finish the task. Stopping before the task is complete is very rarely merited, as in when the task can't move forward without user input, or where the blocker is deliberate and should not be worked past. This does not override the need for confirmation on risky or destructive actions.
```

Measured effect: on recorded sessions from an agentic coding product at `medium`, about half as many sessions ended with unfinished work that could have been continued. It removed most endings that ask permission or offer to continue. It did not remove endings that list what remains and then stop.

Cost: output tokens rose by about 30 percent.

Consequence: the model asks less often with this text. So the prompt must name the risky or irreversible actions that still need confirmation. See section 3.

Leave it out for short tasks, single questions, and writing tasks with one deliverable. It adds cost and nothing to finish.

## 3. Approvals and caution

The model follows cautious instructions closely. Vague or broad caution makes it hold routine work.

- Name the exact actions that need approval. A broad category backfires. In Anthropic's testing with nobody there to approve, a rule that listed "sending messages to people outside this conversation" led the model to hold back about half of its routine requests to colleagues and other teams.
- Name the routine steps that need no approval. "Act more freely" is too vague to change anything.
- The model treats an instruction that contains "never" as absolute. If the user wants to keep the decision, write "ask for approval before deleting files" and not "never delete files". Keep "never" for rules that truly have no exception, and give the reason.
- A condition such as "stop and review at the first error" can block autonomous work. Narrow it or remove it.
- When nobody will be there to approve, say what to do in place of asking. For example: "Do not push or deploy. Leave the branch ready and list the commands I should run."
- When the user later tells the model to proceed, a specific reply works better than "continue" or "go". Example: "Yes, run the migration on the staging database."

An optional general text for confirmation rules, from the best practices page. Trim the examples to the actions that exist in the task:

```text
Consider the reversibility and potential impact of your actions. You are encouraged to take local, reversible actions like editing files or running tests, but for actions that are hard to reverse, affect shared systems, or could be destructive, ask the user before proceeding.

Examples of actions that warrant confirmation:
- Destructive operations: deleting files or branches, dropping database tables, rm -rf
- Hard to reverse operations: git push --force, git reset --hard, amending published commits
- Operations visible to others: pushing code, commenting on PRs/issues, sending messages, modifying shared infrastructure

When encountering obstacles, do not use destructive actions as a shortcut. For example, don't bypass safety checks (e.g. --no-verify) or discard unfamiliar files that may be in-progress work.
```

Note the last bullet group. "Sending messages" as a blanket category is the wording that held back routine work in testing. If the task includes routine messages, name which ones are fine.

## 4. Code ownership

Behaviour: the model pays close attention to who owns the code. When the prompt says nothing about ownership, or presents the code as someone else's, the model adds to the existing structure, even when that duplicates code, and reports nearby problems without fixing them. When the prompt says the code is the model's own, it often consolidates duplicated code and sometimes makes related fixes without being asked.

If the user wants the structure improved as the work goes, add:

```text
You are the owner and long-term maintainer of this codebase, and its design is up to you.
```

Measured effect: the model consolidated duplicated code in under 10 percent of runs without the sentence and in about 75 percent of runs with it.

Cost: output tokens rose by about 70 to 80 percent. The model made changes nobody had asked for, many of them new tests and documentation. In some runs it changed how an existing feature behaves. Its closing summary often left out small changes. Tell the user to review the full diff, not only the summary.

If the user wants the smallest possible change, leave the sentence out. Say whose code it is, ask for a minimal diff, and ask for nearby problems to be listed and not fixed.

Do not combine the two. A scope rule does not cancel the ownership sentence. With "No drive-by refactors, renames or cleanups" earlier in the same prompt, the model still consolidated duplicated code in more than half of runs.

## 5. Progress updates

Behaviour: the model sometimes writes little user-facing text during long runs of tool calls, and may make its first tool call without saying what it is about to do.

Use these only where a person is watching the run.

For an opening line, brief updates, and a closing recap:

```text
Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own (what you found, what you did, and what's next), so a reader who only sees the last message has the full picture.
```

Measured effect: the model wrote a message before its first tool call in every run of the coding tasks. Cost: output tokens per task rose by roughly 20 percent.

A lighter option, one sentence:

```text
Every few tool calls, give the user a one-line update on where things stand.
```

Measured effect: it cut most long silent stretches on workflow automation tasks. Runs of several tool calls between updates still happen.

For unattended runs, leave both out. Anthropic recommends the model for long asynchronous work where nobody needs frequent updates. A final report (section 7) serves that reader better.

If the user reports that a client "looks silent", the cause may be the client and not the prompt. The model's between-tool updates come back as `thinking` blocks whose text is empty at the default `thinking.display` of `"omitted"`. That is a harness setting, outside what a task prompt can fix. Mention it in the notes.

## 6. Subagents

Behaviour: how much the model delegates depends on the harness and the task. With a minimal subagent tool it rarely delegates. In a product whose prompts invite delegation it delegates readily. At `xhigh` and `max` it may start subagents to review its own work. Subagents can account for a large part of what a session costs.

Default: say nothing about subagents in the prompt.

Remove any standing line that tells the model to delegate, such as "use subagents for everything".

If the user wants parallel work, or wants to limit delegation, say when it is worth it. A general text from the best practices page:

```text
Use subagents when tasks can run in parallel, require isolated context, or involve independent workstreams that don't need to share state. For simple tasks, sequential operations, single-file edits, or tasks where you need to maintain context across steps, work directly rather than delegating.
```

If the model still delegates too much, say more about what a subagent costs. In Anthropic's testing at `high`, a longer tool description focused on cost roughly halved the number of subagents.

## 7. Final reports

Behaviour: after long agentic work, the person who reads the final report usually has not seen the tool calls or the files behind it. The report has to make sense on its own.

Add this when the task ends with a report or a handoff that someone will read later:

```text
# Communicating clearly
Write for an intelligent but low-context reader. The person reading has not seen your tool calls, your reasoning or the files you opened, and may be switching in from other work. Messages should generally be self-contained: the reader should understand what you mean without having to dig for other context. Put the key points in the first few sentences. Practice progressive disclosure in your sentence order, and do not use undefined references and ideas in line 1 that won't make sense until the reader reaches line 12.

Use plain words, and write in clear, understandable full sentences. Do not use a term or abbreviation that you made up during the work. Say what the thing is in a way any reader would understand. Write mostly in the style of ASD-STE100 Simplified Technical English: short sentences, one idea per sentence, active voice, and consistently using the same word for the same thing. Use a paragraph of flowing prose where ideas connect, and keep bullets for lists of separate items. Leave out unnecessary details unless the reader asks and practice concision, but do ensure that your messages have the key context. Aim to make your user-facing text as human-readable and clear as possible.
```

Measured effect: final reports were less dense and had fewer semicolons. Sentences were shorter on workflow automation tasks.

Also say how long the report should be and which specific details it must contain. The text above does not set either.

Before adding it, remove hard style rules from the prompt. See section 8.

For a short task, one sentence is enough: ask for a closing summary that a reader who did not watch the work can follow.

## 8. Lines to remove

These lines were written for earlier models. On Fable 5.2 they cost tokens, cause side effects, or do nothing. Remove them when you write a new prompt, and remove them from a prompt the user asks you to improve.

| Line in the prompt | Problem on Fable 5.2 | Replace with |
|---|---|---|
| "Think step by step", "show your reasoning", "put your reasoning in `<thinking>` tags" | Thinking is always on. A request to reproduce reasoning in the response can be declined with the `reasoning_extraction` refusal. | Nothing. If the user wants to see reasoning, that is `thinking.display: "summarized"` in the API. If they want justification in the answer, ask for "the evidence for each conclusion". |
| "Answer directly without deliberating", "don't overthink" | Does not stop thinking at `medium` and above. | A lower effort level. |
| A token budget shown to the model ("you have 200k tokens") | The size of the budget changes how much the model spends. Removing a large budget lowered cost with equivalent performance. | Nothing. This is about prompt text, not the API's task budgets feature. |
| "Use subagents for everything", "always delegate" | Delegation rises and so does cost. | Nothing, or the conditional text in section 6. |
| "Treat your earlier answer as done", "focus only on the new message" | The model becomes far less likely to point out mistakes in its earlier answers. | Nothing, where the user wants errors caught. |
| Hard style limits ("20 words or fewer per sentence", "one instruction per sentence") | The model follows them closely and the prose turns choppy. | The reader and the purpose: who reads this and what they need from it. |
| "Never X", where the user wants oversight and not a ban | Treated as absolute. | "Ask for approval before X." |
| A broad approval category ("anything external", "sending messages") | Routine work is held. | The exact actions that need approval. |
| "Stop and review at the first error" | Blocks autonomous work. | A narrower condition, or nothing. |
| Old cautions carried over from notes and memory ("never retry") | Still obeyed after the reason is gone. | Nothing, or the rule with its scope and reason. |
| "Start your answer with `{`" and other prefill-style lines | Prefill is not supported. | A format instruction, or structured outputs in the API. |

General cleanup from the best practices page. It applies to all current models:

- Emphasis in capitals ("CRITICAL", "YOU MUST"). Newer models are responsive to instructions, so shouting causes overuse of the thing shouted about. State the instruction plainly and give the reason.
- "Can you suggest changes…" when the user wants the changes made. The model may only suggest. Use the imperative: "Change this function…".
- Format rules written as prohibitions ("do not use markdown"). Say what to write: "Write flowing prose paragraphs."
- Blanket tool defaults ("if in doubt, use the tool"). Say when the tool helps.

## 9. Refusals

Fable 5.2 runs safety classifiers that can decline a request. The API returns `stop_reason: "refusal"` and a category in `stop_details.category`: `cyber`, `bio`, `frontier_llm`, `reasoning_extraction`, `general_harms`, or `null`.

What this means for prompt writing:

- Do not ask the model to write out its internal reasoning in the response. That is the `reasoning_extraction` category, and it is the one a prompt can cause by accident.
- Benign security and life sciences work can also trigger `cyber` and `bio`. Rewording a prompt to get past a classifier is not what this skill does. If the user's task is in one of these areas, write the prompt plainly and tell the user in the notes that the request may be declined, and that Anthropic's Cyber Verification Program and Life Sciences Verification Program exist for organizations with legitimate work there. Claude Mythos 5.2 is the same model, available to verified organizations.

## 10. API facts that limit what a prompt can ask for

- Thinking is adaptive and always on. It cannot be disabled or given a manual budget.
- Prefilling the assistant message returns an error. Put format requirements in the instructions.
- Forcing a specific tool call is not supported. If one tool must be used, name the tool in the instruction.
- `temperature`, `top_p`, and `top_k` are omitted for this model. Do not tell the user to tune them.
- Changing the system prompt, the tools, or an earlier message partway through a stored conversation can make the request fail. New instructions go in the newest turn, or in a new conversation. A task prompt is a new turn, so it is safe. Tell the user to start a new conversation if the prompt replaces instructions they gave earlier.
- Context window is 1M tokens. Maximum output is 128k tokens.

## 11. General techniques and optional texts

These come from the best practices page and apply to all current Claude models.

Principles:

- Be clear and direct. Treat the model as a brilliant new employee who has no context on your norms. Test: would a colleague with minimal context be able to follow the prompt?
- Give the reason behind an instruction. The model generalizes from the reason. "Your response will be read aloud by a text-to-speech engine, so never use ellipses" works better than "NEVER use ellipses".
- Ask explicitly for anything beyond the literal task. The model follows instructions precisely and does not add extras on its own.
- Use examples when the output has a pattern to match. Three to five, relevant and varied, each in `<example>` tags inside `<examples>`. The model copies examples closely, so varied examples prevent it from copying one accident of format.
- Wrap each kind of content in its own XML tag with a consistent, descriptive name: `<instructions>`, `<context>`, `<data>`.
- Put long material (about 20k tokens or more) at the top and the instructions and question at the end. For several documents, use `<documents>` with one `<document index="n">` each, holding `<source>` and `<document_content>`.
- For work grounded in long documents, ask the model to quote the relevant passages first and then answer from the quotes.
- Match the style of the prompt to the style you want back. A prompt written in prose gets prose more readily than a prompt made of bullets.
- Prefer a general instruction over a hand-written step-by-step plan. Give steps only when the order or completeness truly matters.
- Ask for a check at the end: "Before you finish, verify your answer against [criteria]." This catches errors reliably in coding and math.
- For research, state success criteria and ask for verification across several sources.
- For long work that spans context windows, have the model keep state in files: structured data for test status, free text for progress notes, git for checkpoints.

Optional texts. Use one only when the task shows the need, and trim it to the task. These are shortened from the longer versions on the best practices page, with the capitals removed, so unlike the texts in sections 2 to 7 they are not word-for-word copies.

To make the model act and not only advise:

```text
By default, implement changes rather than only suggesting them. If the user's intent is unclear, infer the most useful likely action and proceed, using tools to discover any missing details instead of guessing.
```

To make the model advise and not act:

```text
Do not jump into implementation or change files unless clearly instructed to make changes. When the user's intent is ambiguous, default to providing information, doing research, and providing recommendations rather than taking action.
```

To keep answers about code grounded:

```text
Never speculate about code you have not opened. If the user references a specific file, read the file before answering. Investigate and read relevant files before answering questions about the codebase.
```

For complex research:

```text
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

To keep a solution general and not fitted to the tests:

```text
Implement a solution that works correctly for all valid inputs, not just the test cases. Do not hard-code values or create solutions that only work for specific test inputs. Tests are there to verify correctness, not to define the solution. If the task is unreasonable or infeasible, or if any of the tests are incorrect, please inform me rather than working around them.
```

To leave no scratch files behind:

```text
If you create any temporary new files, scripts, or helper files for iteration, clean up these files by removing them at the end of the task.
```
