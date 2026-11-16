---
name: prompt
description: Turn a rough request into a ready-to-paste task prompt written for Claude Fable 5.2 (and Claude Mythos 5.2), following Anthropic's Fable 5.2 prompting guide. Use this whenever the user wants a prompt written, created, improved, rewritten, optimized, or adapted for Fable 5.2, wants to hand a task to Claude, Fable, or Claude Code and needs the message to send, or pastes an old prompt and asks why it behaves badly on the new model. Typical phrasings include "cria um prompt para…", "transforma esse pedido em um prompt", "melhora esse prompt", "como eu peço isso pro Claude/Fable", "write me a prompt for…", and "prompt engineering", even when Fable 5.2 is not named. Produces a task prompt (a user message, not a system prompt), always in English, shown in chat and saved to a file, with a recommended effort level.
---

# Fable 5.2 task-prompt writer

Turn what the user asked for into one task prompt that Claude Fable 5.2 can act on well. The user will send that prompt in another session, so your output is the prompt and a few notes about it. Do not carry out the task the prompt describes. If the user asks for a prompt to refactor a module, they want the prompt, not the refactor.

Fable 5.2 follows instructions closely and literally. That is why this skill exists: a prompt written for an earlier model often carries lines that Fable 5.2 obeys too well (a stray "never", a sentence-length rule, a standing order to delegate), and it often lacks the few things Fable 5.2 needs to be told (who owns the code, whether anyone is watching, what "done" means).

## What you produce

- **A task prompt.** A user message to paste into Claude Code, a chat, or the `messages` array of an API call. System prompts are out of scope. If the user clearly wants a system prompt for an agent or product, say that this skill writes task prompts, and point them to `references/fable-5-2-patterns.md`, whose texts Anthropic wrote for system prompts in the first place.
- **In English, whatever language the user writes in.** The texts in the reference file were tested in English, and a prompt in one language reads better than a mixed one. Quoted material that the model must process (the user's data, their document, their customers' messages) stays in its original language.
- **Shown in chat and saved to a file.** See "Deliver".
- **Notes in the user's language.** The effort level to run it at, the assumptions you made, and what the added texts cost.

## Workflow

### 1. Understand the request

Read everything the user gave you, including pasted files and any earlier prompt they want improved.

If the task is about the repository or folder you are in, look up the real names before you write: the paths, the test command, the package manager, the module the user means. Three or four lookups are enough. A prompt that says "run `pnpm -r test`" is worth more than one that says "run the tests", and looking it up costs the user nothing. Stop there. Reading the code to find the bug is doing the task.

If there is nothing to look up (no repository, or the task is about something elsewhere), do not invent names. Tell the model how to find the fact, for example "use the test command that CI runs", and say in your notes that the prompt gets better if the user adds the real name.

### 2. Decide whether to ask

A task prompt needs four things. Check which ones you have:

1. **The outcome.** What should exist or be true when the work is finished, and why the user wants it.
2. **The definition of done.** How anyone would check that it worked.
3. **The boundaries.** What must not change, which actions need the user's approval, and for code, whether the user wants a minimal diff or welcomes restructuring.
4. **The reader and the form.** Who reads the result, in what format, and whether a person will be watching the run or it runs unattended.

For each one that is missing, ask yourself whether a wrong guess would change the prompt in a way the user would care about. If yes, ask. If you can infer it safely, or look it up, do that and record it as an assumption.

When you ask, ask once: at most three short questions, in one message, each with two to four likely answers to pick from. Use the question tool if you have one. People answer quick multiple-choice questions and abandon open interviews, and three good answers are enough to write a strong prompt. Do not ask a second round. Whatever is still open after the answers becomes a stated assumption.

A specific request needs no questions. Do not ask for the sake of asking.

If nobody can answer (a non-interactive run), write the prompt with your best assumptions, and put the questions you would have asked at the top of your notes so the user can correct them.

Two defaults for what stays unknown:

- **Whether someone watches the run.** If you cannot tell, add neither the progress text nor the "Finishing work" text, and say in one line of the notes which to add in which case. Both cost tokens, and guessing wrong adds a cost with no benefit.
- **A fact that nobody can assume and the model cannot discover**, such as where the user's data is. Leave a slot written as `[FILL IN: what goes here]` and name every slot in the first line of your notes. A slot is a last resort. When you can ask, this kind of fact is the first thing to ask about.

### 3. Write the core of the prompt

See "Writing the core" below.

### 4. Add what Fable 5.2 needs

See "What Fable 5.2 needs to be told". Read `references/fable-5-2-patterns.md` for the exact texts before you add any of them.

### 5. Remove what works against Fable 5.2

See "What to take out". This step matters most when the user gave you an existing prompt.

### 6. Choose an effort level

See "Effort".

### 7. Check and deliver

Read the prompt as a capable colleague who knows nothing about the project. Would they know what to do, when they are done, and what to leave alone? Would anything in it surprise the user if the model obeyed it to the letter? Fix what fails, then deliver.

## Writing the core

Write the prompt the way you would brief a brilliant new colleague who has no context on the project. Each point below comes with its reason, because the reason tells you when to bend it.

**Open with the task as an instruction.** "Change the retry logic so that…", not "Can you look at the retry logic?". Fable 5.2 takes a question literally and may answer it without doing the work.

**Say why.** One or two sentences on what the work is for and who uses the result. The model generalizes from a reason far better than from a rule: knowing that the output is read aloud tells it more than "no ellipses" does. The reason is also what lets it make good calls on cases you did not foresee.

**State what done means, in terms someone can check.** Passing tests, a file that exists, a question answered with sources. Fable 5.2 works toward the stated finish line, so a vague one gives vague work. If there is a way to verify, ask for it at the end: "Before you finish, run X and confirm Y."

**Ask for everything you want.** The model does not add extras on its own. If the user wants tests, a summary, or edge cases handled, the prompt has to say so.

**Give boundaries with their reasons, and say what to do at the boundary.** "Ask for my approval before any schema change" is clear. "Be careful with the database" is not. A boundary without a reason gets applied too broadly or too narrowly.

**Describe the result: reader, format, length.** Effort settings barely change how long Fable 5.2's replies are, so length and tone have to come from the prompt. Say who will read the result and what they need from it. That works better than rules about sentence length. Describe the format you want, not the one you do not want. When people will read the result, also say which language to write it in. The prompt is in English, so the model will answer in English unless told otherwise. Default to the language the user wrote to you in, and record that as an assumption.

**Prefer a goal to a procedure.** Give numbered steps only when the order truly matters. A capable model with a clear goal plans better than a script you wrote without seeing the code.

**Separate materials from instructions.** Wrap each pasted item in a descriptive XML tag (`<error_log>`, `<draft>`, `<document>`). When the materials are long, put them first and the instructions after them, because the model answers better with the question at the end.

**Use examples only when there is a pattern to match.** Give three to five varied ones inside `<example>` tags. The model copies examples closely, so a single example gets cloned, accidents included.

**Write in prose.** A prompt made of terse bullets gets terse bullets back. Keep lists for things that are lists, such as the criteria for done.

**Keep it proportional.** A one-line question deserves a prompt of a few sentences. A night-long migration deserves several paragraphs. Every sentence should change what the model does. If removing a sentence would change nothing, remove it.

## What Fable 5.2 needs to be told

Each row is a behaviour Anthropic documented for this model. Add the text only when the condition holds, because each one costs tokens and some have side effects. The exact texts, the measured effects, and the cases for leaving them out are in `references/fable-5-2-patterns.md`.

| When the task… | Add | Cost and side effects |
|---|---|---|
| is long and multistep, and above all runs unattended or at `low` or `medium` effort | The "Finishing work" text (section 2). Without it the model may stop to list what is left or to offer to continue. | About 30% more output tokens. The model asks less, so name the risky actions that still need approval. |
| changes code, and the user wants the structure improved along the way | The ownership sentence (section 4). | About 70 to 80% more output tokens, and changes nobody asked for. Tell the user to review the full diff. |
| changes code, and the user wants the smallest change | No ownership sentence. Say whose code it is, ask for a minimal diff, and ask for nearby problems to be listed and not fixed. | None. |
| has a person watching it run | The progress text (section 5). | About 20% more output tokens. |
| ends in a report or handoff read by someone who did not watch | Say who reads the report, what it must contain, and how long it is. When the prompt says all three, that is enough. Add the full "Communicating clearly" text (section 7) when the readers are several or unknown, or when the user says the model's reports are dense or hard to follow. | Small. |
| includes actions that are destructive, hard to undo, or visible to other people | The exact actions that need approval, and the routine ones that do not (section 3). If nobody will be there to approve, say what to do in place of asking. | None. |
| would benefit from parallel work, or the user complains about too many subagents | A sentence on when delegation is worth it (section 6). Otherwise say nothing about subagents. | Subagents can be a large share of the cost of a session. |

Two rules that are easy to get wrong:

- **Never pair the ownership sentence with a scope limit.** A line such as "no drive-by refactors" does not cancel it. The model still restructured code in more than half of Anthropic's test runs. Choose one stance. If you cannot tell which the user wants and the task touches code they care about, it is one of your three questions. A scope limit here means a limit on how much of the code may change. A requirement about behaviour, such as "the amounts charged must not change", is a correctness criterion, and it goes well with the ownership sentence.
- **Write "ask for approval before X" unless the user means a true ban.** Fable 5.2 treats "never" as absolute. "Never touch the database" also forbids the read that the task needs.

## What to take out

Remove these from any prompt you write or improve. The reference file (section 8) has the full table with reasons.

- **Requests to show reasoning** ("think step by step", "explain your reasoning", "use `<thinking>` tags"). Thinking is always on, and a request to reproduce it in the response can be declined as `reasoning_extraction`. If the user wants to be able to check the work, ask for the evidence or the sources behind each conclusion. That is a result, not the reasoning.
- **Requests to think less** ("answer directly", "don't overthink"). They do not work at `medium` effort and above. Recommend a lower effort level.
- **A token budget shown to the model.** It changes how much the model spends. Anthropic saw lower cost and equal results without it.
- **Standing orders to delegate** ("use subagents whenever possible").
- **Hard style limits** ("sentences under 15 words"). The model obeys and the prose turns choppy. Describe the reader in their place.
- **Emphasis in capitals** ("CRITICAL", "YOU MUST"). State the instruction plainly and give the reason.
- **Broad or stale caution** ("be very careful", "stop at the first error", "never retry"). Narrow it to the actions that matter, or drop it.
- **Prefill-style lines** ("start your answer with `{`"). Not supported. Describe the format.
- **A generic role line** ("You are an expert engineer"). It tells the model nothing that the task does not. Keep a role only when it is specific and changes what the model should do, and say so in a plain sentence.

When you improve an existing prompt, keep the user's task and their real constraints. Tell them what you removed and why, in a line each, so that they can put something back if you misread its purpose.

If the task is in security or life sciences, write the prompt plainly and mention in the notes that Fable 5.2's safety classifiers may decline it (reference, section 9). Do not reword a prompt to get past a classifier.

## Effort

Effort is a setting on the request, not text in the prompt. Recommend one level with every prompt, with a one-line reason.

| Level | Recommend it when the task is |
|---|---|
| `low` | quick, well-specified, or mechanical |
| `medium` | interactive coding, chat, or everyday knowledge work |
| `high` | hard coding, research, agentic work, or a long unattended run. This is the default. Recommend it when unsure. |
| `xhigh` | among the hardest long-running coding or agentic work, and the user can afford many more tokens for a slight gain |
| `max` | an open-ended problem, such as research mathematics, where time and tokens do not matter |

If you recommend `low` or `medium` for agentic work, the "Finishing work" text becomes more important, because the model stops early more often at those levels.

## Deliver

Save the prompt to `prompts/<short-kebab-slug>.md` under the current working directory, and create the folder if it does not exist. The file holds the prompt and nothing else, so that the user can pipe it straight into a model. If a file with that name exists, add a numeric suffix. Do not overwrite it.

Then reply in the user's language, in this order:

1. The prompt in a fenced `text` block, ready to copy. Use a four-backtick fence if the prompt itself contains triple backticks.
2. The path of the saved file, relative to the working directory.
3. **Effort**: the level and a one-line reason.
4. **Assumptions**: what you assumed or inferred, one line each. If you could not ask your questions, list them here first.
5. **Added for Fable 5.2**: each text you included, why, and its cost. If you left one out that the user might expect, say so in one line with the case in which to add it. Skip this section if there is nothing to say.
6. **Removed**: only when improving an existing prompt. What you took out and why.

Keep the notes short. The user wants the prompt, and the notes exist so they can correct you quickly.

If the prompt includes approval steps, add one line to the notes: when the model asks to proceed, a specific reply ("Yes, run the migration on staging") works better than "continue".

## Example

Request, in Portuguese, made inside a repository: "faz um prompt pro fable arrumar o bug do 401 depois que o token expira".

The outcome is clear. A look at the repo shows the auth code lives in `src/auth/` and tests run with `npm test`. What is not clear, and changes the prompt, is whether the user wants a minimal fix or a cleanup of the auth code, so that is the one question to ask. Answer: minimal, because the change will be reviewed by another team.

```text
Fix the bug where a request fails with a 401 after the access token expires, when it should refresh the token and retry.

This is the client auth code in `src/auth/`. When the access token expires, the client is supposed to use the refresh token to get a new one and repeat the original request. That retry is not happening, so users are logged out in the middle of a session and lose what they were typing.

The fix is done when:
- A request made with an expired access token and a valid refresh token succeeds, and the user sees no error.
- A request made with an expired refresh token still fails with a 401 and sends the user to the login page.
- There is a test for each of those two cases, and `npm test` passes.

Another team owns this code and will review the change, so keep the diff as small as the fix allows. If you find other problems in the auth code, list them at the end and leave them as they are.

Find the cause before you change anything. Before you finish, run `npm test` and confirm both cases.

Close with a short summary that someone who did not watch you work can follow: what the cause was, which files you changed, and what you noticed and left alone.
```

Notes for the user: effort `medium` for an interactive fix, or `high` if they leave it running. No ownership sentence, because they want a minimal diff. No "Finishing work" text, because the task is short.

`references/examples.md` has three longer examples: an unattended overnight coding task, a research memo, and an existing prompt to improve. Read it when the request resembles one of those.
