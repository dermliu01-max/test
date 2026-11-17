---
name: fable-playbook
description: Working method for planning a task, breaking it into verifiable steps, and checking your own work before declaring it done. Use at the start of any non-trivial task (more than a couple of tool calls, anything touching multiple files, anything the user will ship or rely on), and again before reporting completion.
---

# Fable Playbook

A method for doing tasks well: understand first, plan briefly, work in small
verified steps, and check the result against the request rather than against
your own intentions. Scale it to the task: a one-line fix needs seconds of this,
a multi-file feature needs all of it.

---

## 1. Plan the task

### 1a. Restate the goal and "done"
Before touching anything, write (at least in your head) one sentence each:
- **Goal**: what the user actually wants, in their terms.
- **Done means**: the observable condition that proves it. "Tests pass",
  "file X exists with Y", "page renders on phone width", "branch pushed".
- **Constraints**: branch names, files not to touch, language/format asked for,
  things the user explicitly said or rejected earlier in the conversation.

If the request is ambiguous *and* the ambiguity changes what you'd build, ask
one focused question. If a sensible default exists, pick it, say so in one line,
and proceed.

### 1b. Orient before deciding
Look before you build. Cheap reads beat wrong assumptions.
- What exists already? (`ls`, `git status`, `git log`, read the relevant files,
  grep for the function/name you're about to add — it may already exist.)
- What conventions does the project use? (README, CLAUDE.md, neighbouring code,
  test layout, how existing similar things were done.)
- What does the environment allow? (Network, branch rules, ephemeral disk.)

Do not re-derive facts already established in the conversation. Do not
re-ask decisions the user already made.

### 1c. Choose an approach — one, with a reason
- Prefer the smallest change that fully meets the goal.
- Prefer reusing existing code/patterns over inventing new ones.
- Name the main **risk** (what is most likely to go wrong) and how you'll
  detect it. Example: "Risk: the skill only lives in an ephemeral directory →
  also commit it to the repo."
- If two approaches are close, pick one and move on. Don't write surveys.

---

## 2. Break it into steps

### 2a. Steps are small and each has a check
Write the plan as a short ordered list. Each step should be:
- **Small**: one logical change (one file, one function, one command).
- **Verifiable**: you know what output proves it worked.
- **Ordered by dependency**: things that later steps rely on come first;
  the riskiest unknown goes as early as possible so failure is cheap.

Template:
```
1. <action>  → check: <what proves it>
2. <action>  → check: <what proves it>
...
N. Final verification against "done" (section 3)
```

For multi-step work, track steps with the task list tool if available, marking
each in progress / done as you go — exactly one in progress at a time.

### 2b. Execute step by step
- Do the step, then run its check **before** starting the next one. Errors
  compound; catch them where they're cheap.
- Batch independent tool calls in parallel; serialize dependent ones.
- Read tool output fully. Don't assume a command succeeded — look at exit codes,
  warnings, and actual content.
- When something fails: diagnose first (read the error, check the assumption it
  breaks), then fix. Don't retry the same thing blindly. After two failed fixes
  of the same kind, step back and question the approach.
- If you discover the plan is wrong, update the plan explicitly rather than
  drifting.
- Don't stop mid-task to ask permission for things the user already asked for.
  Stop only for genuine blockers or risky/irreversible actions (deleting,
  force-pushing, publishing, sending externally) — those need confirmation.

### 2c. Make results durable early
In ephemeral environments, write files and commit/push as soon as a coherent
piece is done, not only at the very end. An unwritten result is lost if the
session dies.

---

## 3. Check your own work

Checking means comparing the result to the **request**, not to your plan.
Your plan may have drifted; the request didn't.

### 3a. Re-read the original request
Go back to the user's exact words. List every explicit ask (name, location,
format, language, scope). Tick each one off against what actually exists.
Common misses: wrong file name/path, a sub-request forgotten, wrong language,
a constraint violated (branch, "don't touch X").

### 3b. Verify with evidence, not belief
- Run it: tests, the build, the script, render the page, `cat` the file.
- A check must be able to fail. "It compiles" doesn't prove "it's correct";
  test the actual behaviour asked for, including one edge case.
- If you can't run something, say so plainly instead of implying it works.

### 3c. Review the diff as a skeptical reviewer
`git diff` (or re-read the file) and ask:
- Is anything here that wasn't asked for? Remove it.
- Debug leftovers, TODOs, placeholder text, secrets, personal data?
- Does it match the project's existing style and conventions?
- Would a new reader understand it without this conversation?

### 3d. Report honestly
- Lead with the outcome: what was done and where it is (exact paths, branch).
- State what you verified and how. State what you did *not* verify.
- If tests failed or a step was skipped, say so with the reason.
- Note any decision you made on the user's behalf in one line.
- No exaggeration, no hedging on things you did verify.

---

## Quick checklist

Before starting:
- [ ] Goal, "done", and constraints stated
- [ ] Looked at what already exists
- [ ] One approach chosen; main risk named

During:
- [ ] Each step small, with a check, run before moving on
- [ ] Failures diagnosed, not blindly retried
- [ ] Work saved/committed as it becomes coherent

Before saying "done":
- [ ] Re-read the user's exact request; every ask ticked off
- [ ] Verified by running/reading, with a check that could fail
- [ ] Diff reviewed; nothing extra, nothing leftover
- [ ] Report says what was verified and what wasn't
