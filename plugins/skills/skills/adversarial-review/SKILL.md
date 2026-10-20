---
name: adversarial-review
description: Run a competitive multi-reviewer critique of a document, design spec, plan, or piece of code by spawning two or more independent fresh subagents in parallel, told they're competing to find the most real, significant problems. Use this whenever the user asks for a "harsh review," "adversarial review," "red team this," "poke holes in this," wants a second (and third) opinion on a design doc or plan before committing to it, or explicitly asks to turn up flaws/oversights/gaps in something they wrote or decided. Especially good for high-stakes or irreversible designs (data deletion, auth, money, migrations) where a single reviewer's blind spots are costly. Don't use this for routine code review of a diff — that's a different, narrower task; this is for standing back and stress-testing a whole document or design.
---

# Adversarial Review

Single-reviewer critique tends to converge on the same handful of issues a careful author already half-suspects, because one reviewer shares context, framing, and blind spots with whoever wrote the thing. Competitive framing across *independent* reviewers fixes this differently than just asking twice: telling each reviewer they're being scored against another, unseen reviewer changes what they optimize for — they stop settling for "a reasonable pass" and start hunting for what the other one might catch that they wouldn't.

The trick has three load-bearing parts. Get any of them wrong and you get noise, not signal:

1. **Independence.** Each reviewer must be a fresh subagent with no view into the other's findings and, ideally, no shared context bias from the conversation that produced the artifact — a `fork` inherits your context and your blind spots along with it, so prefer a fresh general-purpose agent unless the review genuinely needs your conversation's context to be legible at all.
2. **Double-blind.** Neither reviewer sees the other's findings (blind #1, covered by independence above) — and neither reviewer is told anything about *this artifact's own review history*: no prior findings, no "already fixed, don't re-report" list, no framing that this is a second or third pass (blind #2). A reviewer told what's already been checked stops forming its own view of the whole artifact and starts verifying a checklist instead — which is exactly backwards, since a previous fix can be wrong, incomplete, or have quietly reintroduced the problem it claimed to solve, and a reviewer primed to treat it as settled will walk right past that.
3. **A penalty for padding.** Competition alone incentivizes quantity. Without an explicit "fabricated or trivial findings cost you points" rule, reviewers under competitive pressure pad their list with nitpicks and hypotheticals to inflate their count, which drowns the real findings in noise. The penalty is what keeps the competition honest.

## When to reach for this

- The user asks for a harsh/adversarial/red-team review, or to "poke holes in" something.
- A design is about to be locked in and is high-stakes or hard to reverse (deletion logic, auth, billing, data migrations, anything where a missed flaw is expensive to discover later).
- The user wants a second and third opinion on a doc they (or you) just wrote, distinct from routine proofreading.

Not a fit for reviewing a small diff or a single function — that's ordinary code review, and the parallel-adversarial-review machinery is overhead for something a single careful pass handles fine.

## Running it

**1. Pick the reviewer count.** Two is the default and usually enough. Add a third only if the artifact is unusually large or the stakes justify the extra cost — more reviewers means more redundant findings to de-duplicate, not proportionally more real ones found.

**2. Spawn all reviewers in parallel, in a single message**, each as a genuinely independent subagent (not a fork of yourself, and not sequential — if reviewer B starts after reviewer A finishes, you'll be tempted to feed it A's results, which destroys the independence this whole approach depends on).

**3. Give every reviewer the same prompt shape:**

- **What they're reviewing** and a **file path**, not pasted content, so they read it fresh rather than working from your summary of it.
- **The competitive frame, explicitly**: they're one of N reviewers working independently and in parallel; whoever finds the most *real, significant* problems is ahead — but state the padding penalty in the same breath, not as an afterthought. Something like: *"if you report something trivial, made-up, or padded just to inflate your count, you lose points for it — quality and validity matter more than quantity, but a real finding is worth claiming."*
- **What counts as a real finding for this artifact.** Generic advice ("look for bugs") underperforms a list scoped to the artifact's actual risk surface — for a design doc that might be race conditions, contradictions between sections, unstated assumptions, security gaps, scaling claims that don't hold up; for code it's a different list. Write the list for what you're actually reviewing.
- **What NOT to do**: don't invent problems that aren't in the text, don't nitpick pure style, and — if the artifact already has a self-critique section (many good design docs do) — don't count restating an already-acknowledged risk as a new finding. Reviewers should go deeper than that section or find something it missed, not credit-farm off it.
- **Output shape**: a ranked list, each finding with a one-line summary, its location in the artifact, and *why* it's real (not hypothetical) — this is what makes findings checkable rather than assertions to take on faith.

**4. Tell the user you don't need to declare a winner** unless they asked for one — the scoring is a device to sharpen the reviewers' effort, not a deliverable. Say this to the reviewers too, so neither wastes output arguing its own case.

**Idempotent reruns.** If this is a second (or third) pass on the same artifact after fixes, resist the temptation to hand the new reviewers a list of what a previous round found so they can "skip those and hunt for what's new" — that's the double-blind property from point 2 above, and it holds on every rerun, not just the first pass. Every run gets the exact same prompt shape: point the reviewer at the artifact, give it the competitive frame and the risk-scoping for what it's reviewing, and let it form its own opinion cold. This is also simpler to operate — the prompt doesn't grow with each rerun, and nothing needs to track what's "already handled." A real fix should survive a reviewer who doesn't know it happened; if it only survives because the reviewer was told not to look, that's not a fix that's been verified.

**5. When both come back:** read both reports before saying anything. De-duplicate overlapping findings (agreement between independent reviewers is itself a signal — an issue both caught unprompted is probably real). For findings only one reviewer raised, spot-check the reasoning yourself before passing it to the user — competitive pressure occasionally produces a plausible-sounding but wrong claim, and your independent judgment is the last filter before the user sees it. Report the surviving findings ranked by severity, not grouped by which reviewer found what — the user cares about the problems, not the scoreboard.

## Example prompt skeleton (adapt per artifact)

```
You are reviewer <A/B> in an adversarial two-reviewer competition. Reviewer <other>
is doing the same task independently and in parallel — no visibility into your
findings or you into theirs. Whichever of you finds the most REAL, SIGNIFICANT
problems gets [N] points. But: if you report something trivial, made-up, or padded
just to inflate your count, you LOSE points for it. Quality and validity matter
more than quantity, but a real finding is worth claiming.

Review: <file path>
Context: <one paragraph on what this is and why it matters>

Look for: <artifact-specific risk list>

Do NOT invent problems that aren't in the text, nitpick style, or flag something
the document's own [self-critique section name] already acknowledges as an
accepted tradeoff — go deeper than that section or find what it missed.

Report a ranked list: one-line summary, location, and why it's real (not
hypothetical) for each finding.
```
