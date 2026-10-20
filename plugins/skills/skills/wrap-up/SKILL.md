---
name: wrap-up
description: "After a task finishes, review the session for avoidable mistakes (user corrections, rejected or blocked tool calls, retries, wrong-tool picks, stale facts, repeated questions) and propose durable fixes to the environment so they do not recur. Fixes target skills, AGENTS.md/CLAUDE.md, repo docs, hooks, settings, and personal config. Use when a task is done, when the user says \"wrap up\", \"what went wrong\", \"how do we stop this happening again\", or after a session that hit roadblocks or drew corrections."
disable-model-invocation: true
argument-hint: [focus]
---

# Wrap-up (session retrospective to environment fixes)

Turn this session's mistakes into changes that stop them recurring. The unit of output is a fix to the environment, not a summary of the session.

An optional focus (`[focus]`) narrows the review to one theme. Without it, review the whole session.

## 1. Collect evidence

Read back through the session and list each avoidable gaffe with the evidence that proves it. Evidence is something that happened, quoted or pointed to: a user correction, a rejected or blocked tool call, a repeated failing command, a tool that could not do the job, a wrong assumption later contradicted, a question the user had to answer twice, a fact that was stale.

Count only gaffes with evidence. Do not infer gaffes from how the session felt. If the session had none, say so and stop: that is a valid result.

## 2. Find the cause and the home for each fix

For each gaffe, name the cause, then pick the home that would have prevented it. Check whether guidance for it already exists before proposing anything.

| Cause | Home for the fix |
|---|---|
| A skill's steps were wrong, missing, or ambiguous | Edit that skill (if it is generated from a source file, edit the source and regenerate) |
| A repo or workspace convention was unknown | `AGENTS.md`, `CLAUDE.md`, or the repo's contributor docs |
| Domain knowledge was missing | The project's shared context or docs |
| A rule existed and was ignored, and the skill step or file where the agent acted does not state it | Add the rule there, in the action's own words. Placement comes before enforcement |
| A rule existed and was ignored where it is already stated, and every call a block would stop is a violation | Enforce it: a hook, a `settings.json` permission, or a script. Another sentence of prose will not hold |
| A rule existed and was ignored where it is already stated, and a block could stop a legitimate call | Keep the rule as prose at the point of use and write the legitimate exception into it as a condition |
| A tool lacked a capability or was the wrong one | Name the right tool in the skill or `AGENTS.md`, or note the missing capability |
| The point is personal (habits, accounts, local paths) | The user's own `~/.claude` config or memory. Propose only; never write it into a committed file |

If the guidance already exists, the fix is placement, enforcement, or a clearer wording of the existing line. Edit that line. Do not add a duplicate.

## 3. Shape each fix

- Write what the output must be, or what to do. Reserve prohibitions for rules an agent knew and skipped.
- Keep a fix to the smallest edit that closes the gap. One cause, one change.
- State rules as rules. Do not narrate the session ("in the last run we found...").
- Keep counts, dates tied to a moment, and `file:line` references out of written guidance. They go stale.
- Committed files carry no one person's setup: no home paths, personal accounts, or machine-specific values.

## 4. Present and ask

Show the findings in chat as a ranked list. For each: the evidence, the cause, the proposed fix, and its home file. Put the most repeatable gaffes first.

Then ask which fixes to apply with `AskUserQuestion`, one option per fix, batched up to four per call. End your turn there. Do not apply anything the user has not chosen.

## 5. Apply what was chosen

1. Enter a worktree before editing a committed repo file.
2. Make each approved edit in its home. For a generated file, edit the source and regenerate. Do not hand-edit generated output.
3. Verify: re-read each edit, run the repo's lint and any generate-drift checks, and for a hook or permission change run it once.
4. Commit on the branch and push it. Never push to the default branch.
5. Personal-config fixes: show the exact text and where it goes, and let the user place it.
6. Report what changed, where it lives, and what was left undone.

## Hard limits

- Never post, comment, message, or open a PR on the user's behalf from this skill. Write files and push a branch only.
- Never edit a skill, hook, or setting the user did not approve in step 4.
- Do not write secrets, tokens, or PII into any file.
- This skill reviews the session that just ran. It does not audit the whole environment unprompted.

> Claude Code: this skill is manual (`/wrap-up`). Run it at the end of a task, in the same session, so the evidence is still in context.
