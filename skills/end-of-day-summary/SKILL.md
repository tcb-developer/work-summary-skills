---
name: end-of-day-summary
description: Produce a shareable end-of-day work summary from a project's WORK_SUMMARY files. Project-agnostic, with no hardcoded paths; it resolves the current project's summary folder, reads the day's WORK_SUMMARY-DD-MM-YYYY.md, and prints a team-facing summary grouped as "Task N:" with numbered short sentences, written in the project's own terminology, free of implementation detail and em-dashes. Triggers on "/end-of-day-summary" or requests like "end of day summary", "give me today's work summary to share", "summary for 14 July".
---

# end-of-day-summary - shareable daily summary from WORK_SUMMARY files

This skill turns a day's `WORK_SUMMARY-DD-MM-YYYY.md` file into a shareable,
plain-English summary that the user copies into a chat message, an email or a
status update. It reads what is already recorded; it does not re-derive work
from git or the codebase.

It only reads and formats. It writes no file unless the user asks to save the
output.

---

## Step 1 - Resolve the summary folder

First match wins:

1. A work-summary path pinned in the project's `CLAUDE.md` / `AGENTS.md`.
2. `<git toplevel>/WORK_SUMMARY` (`git rev-parse --show-toplevel` from the
   primary working directory).
3. `<primary working directory>/WORK_SUMMARY`.
4. If several plausible folders exist (for example one at the workspace root and
   a stale one inside a sub-repository), prefer the one `CLAUDE.md` names; if
   still unclear, ask the user once.

Call the resolved folder `<WS>`.

## Step 2 - Pick the day

- Default to today, taken from the harness context (the current-date line).
  Cross-check against the machine clock, which can be wrong.
- If the user names a date, use that date.
- Target file: `<WS>/WORK_SUMMARY-DD-MM-YYYY.md`.
- If it does not exist, list the available files, say the date is missing and
  offer the most recent one. Never invent content for a day with no file.

## Step 3 - Read and select

Collect every `### ` task section. Use its `> **Human Summary:**` block as the
main source (it is already plain English) and the Technical Details only to
understand what to say.

Include only that day's work:

- Sections with `**Status:** Completed`. Mention In Progress or Blocked items
  only as a short note after the block, never as a Task.
- Skip carried-over context from earlier days.
- Omit a change that was added and reverted the same day (net zero), unless the
  user asks.

## Step 4 - Produce the summary (exact format)

Print it inside a plain code block so it can be copied as is:

```
Work Summary - DD Month YYYY

Task 1: <plain-English, outcome-led task title>

1. <plain-English point about the problem or the change>
2. <next point>
3. <next point>

Task 2: <plain-English, outcome-led task title>

1. <point>
2. <point>
```

Rules:

- **Header:** `Work Summary - DD Month YYYY` with the long month name, for
  example `Work Summary - 15 July 2026`.
- **One `Task N:` per distinct deliverable**, numbered in order. Same-day
  refinements of one feature fold into that task's points. Follow the reader's
  mental model, not the commit history.
- **Task title:** a business-outcome sentence, not a bare noun phrase and not a
  code identifier.
- **Body:** numbered points, each a short plain-English sentence. No sub-bullets,
  tables or nested numbering.
- **Use the project's own terminology (critical).** The readers are the team
  that built the product; they know its document, screen, field, role and
  feature names. Use those names exactly as the summary file and the product use
  them. Replacing a real project term with a generic euphemism makes the summary
  harder to read.
  - Right: "Made the Return Request SLA clock reliable at every status."
  - Wrong: "Made the returns ticket countdown timer reliable at every stage."
  - Acronyms and product nouns the team uses daily are fine. When unsure whether
    a word is project vocabulary, check how the day's sections and the codebase
    name it.
- **Avoid pure implementation detail** that means nothing to the reader: file
  paths, function, class or variable names, commit hashes, version numbers,
  error or HTTP codes, build or shell commands, URLs. Branch names are fine when
  the task is about moving work between branches.
- **No em-dashes anywhere, no smart quotes**, and no status or deployment footer
  line.
- **Faithful:** do not invent, inflate or drop real deliverables (apart from the
  net-zero rule above).

## Step 5 - Deliver

- Print the block in the chat. Save it to a file only if the user asks.
- If you know of work that has no section in the file, say so after the block
  instead of silently adding it. Recording work is the `work-summary` skill's
  job.

---

## Boundaries

- Read and reformat only. Never open, fill or audit task sections.
- Never inspect git or the codebase to discover work. A thin file gives a thin
  summary, and that is said openly.
- For several projects, run once per project folder and label each block with
  the project name.
