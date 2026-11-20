---
name: work-summary
description: Zero-miss daily work logging for any project. Every task the user gives (feature, bug fix, error resolution, change request) gets its own `###` section in WORK_SUMMARY/WORK_SUMMARY-DD-MM-YYYY.md, opened the moment the task arrives, filled during the work, and closed with a two-part Technical Details + plain-English Human Summary before the agent says "done". Use PROACTIVELY on every code-touching task without being asked, at task completion, and at session wrap-up to audit that nothing was missed. Also triggers on explicit requests such as "/work-summary", "set up work summaries here", "initialize work logging", "log this task".
---

# work-summary - zero-miss daily work logging

This skill keeps a daily, per-task record of everything the agent does for the
user. Each record has two parts:

1. **Technical Details** for the next engineer or agent: request, files, root
   cause, decisions, commit IDs, verification.
2. **Human Summary** for non-technical readers (managers, HR, accountants,
   timesheets): one plain-English paragraph that a downstream script can extract
   verbatim.

The method exists because, left alone, long agent sessions bundle ten tasks into
three sections at the end of the day, forget tasks that were interrupted, and
write summaries full of file paths that nobody outside engineering can read. The
rules below close each of those gaps.

---

## 1. Where the files live

Resolve the summary folder in this order; the first match wins:

1. **A path pinned in the project's `CLAUDE.md` / `AGENTS.md`.** If the project
   names a canonical work-summary folder, use exactly that folder and nothing
   else.
2. **`<ROOT>/WORK_SUMMARY/`**, where `<ROOT>` is:
   - `git rev-parse --show-toplevel` run from the primary working directory, or
   - the primary working directory, if it is not a git repository.
3. If several folders are plausible (for example one at a workspace root and a
   stale copy inside a sub-repository), prefer the one `CLAUDE.md` names; if
   still unclear, ask the user once.

File name: `WORK_SUMMARY-DD-MM-YYYY.md`, one file per day.

### Preservation rules (why the location matters)

- **Never commit work summaries.** They are local records, not source code.
  When the folder is inside a git repository, add `WORK_SUMMARY/` to
  `.git/info/exclude` (a local-only ignore list that is itself never committed)
  so it cannot be staged by accident. Do not edit the shared `.gitignore` unless
  the user asks.
- **Survive branch switches.** An untracked, excluded folder is untouched by
  `git checkout`, `switch`, `reset` and `stash`. Warn the user that
  `git clean -x` would still delete it. For maximum safety, recommend keeping
  the folder outside the repository (for example in the parent workspace folder)
  and pinning that path in `CLAUDE.md`.
- **One canonical folder.** Never write a day's sections into two different
  folders. A split day means a timesheet extractor sees only half the work.
- **Parallel sessions append, never overwrite.** Another agent may already have
  written today's file. Always read it first, append your sections in
  chronological order, and extend (do not rewrite) the End of Day block.

### Today's date

Take today's date from the harness context (the current-date line), not only
the machine clock; cross-check them if they disagree. Use `date "+%H:%M"` for
the section time.

---

## 2. First use in a project (bootstrap, idempotent)

1. Create the summary folder if it is missing, and add it to
   `.git/info/exclude` when inside a git repository.
2. Create today's file from the **daily-file template** (section 7) if it does
   not exist. If it exists, leave it as is.
3. Add the **CLAUDE.md block** (section 8) to the project's `CLAUDE.md` (or
   `AGENTS.md` if that is the repository's convention) only if a
   `## Work Summary Logging` heading is not already there. Append; never rewrite
   the user's existing content. If no such file exists, create one containing
   just the block.
4. Tell the user the folder path, today's file, and whether `CLAUDE.md` was
   updated.

At the start of every later session, repeat step 2 for the new date.

---

## 3. The task lifecycle (mandatory for every task)

### Step 1 - On task receive, BEFORE any other work

The moment the user gives a task, bug fix, error resolution or change request:

1. Open today's file (create it from the template if missing).
2. Append a new section:

   ```markdown
   ### HH:MM - <Task title>

   <!-- Part 1: Technical Details -->
   **Request:** <the user's ask, restated as a clear engineering statement>

   **Status:** In Progress

   <!-- Part 2: Human Summary (MANDATORY) -->
   > **Human Summary:**
   > Task:
   > Task summary:
   > Date: <DD Mon YYYY>
   > Duration:

   ---
   ```

3. Save the file.
4. Only then start the implementation.

Opening first is what makes the log robust: if the session is interrupted, the
file still shows that the task was received and is unfinished.

### Step 2 - During work

Fill Part 1 progressively: files modified (`- path: what changed`), root cause
for bugs, function names, commit IDs, verification commands and results,
decisions and open questions. Markdown is free here. Leave the Human Summary
empty.

### Step 3 - On completion, BEFORE telling the user "done"

1. Change `**Status:**` to `Completed`.
2. Fill the Human Summary (rules in section 5).
3. Save. Only then report completion.

If the file is not updated, the task is not done.

### Step 4 - End of day

Update the `## End of Day Summary` block: Completed, In Progress, Blocked
(with the reason), Tomorrow (carry-over notes).

### Save points

The file is written at two checkpoints per task: after Step 1 (section present,
In Progress, empty Human Summary) and after Step 3 (Completed, Human Summary
filled). A reader checking mid-session sees an honest In Progress, never a
missing task.

---

## 4. Edge-case rules (fixed decisions, do not reinterpret)

- **Granularity: every message that asks for a change gets its own section**,
  even a one-line copy fix. A message with three tasks gets three sections.
  Section spam is a smaller problem than missing work.
- **Mid-task pivots:** if the user starts task B while task A is In Progress,
  open a new section for B at once. A keeps its In Progress status; return to it
  after B. Never bury B inside A.
- **Iterations on the same task** ("bigger", "now thinner", "move it to the
  corner") stay in ONE section; append each refinement to its Technical
  Details. The Human Summary describes the final state, not the journey.
- **Pure conversation gets no section.** If answering writes nothing to disk in
  the project (a question, a diagnosis, an explanation), do not open a section.
  When in doubt, write the section.
- **Testing sections (recommended).** When a newly built feature is tested in
  the running app, record the testing as its own section titled
  `### HH:MM - [AREA] Testing: <feature>` with the test-case list and results,
  a bug log (ID, steps, expected, actual, severity, status, fix), the number of
  test cycles and the final result. Its Human Summary says what was tested, how
  many issues were found and fixed, and that the feature works.

---

## 5. Human Summary rules (strict)

The Human Summary is pasted verbatim into timesheets and reports read by people
who do not read code. Exact shape, at the end of each section:

```markdown
> **Human Summary:**
> Task: <plain-English, business-outcome title, 8-15 words>
> Task summary: <one flowing paragraph, 80-150 words>
> Date: <DD Mon YYYY, for example 15 May 2026>
> Duration:
```

- **Plain English only.** No other languages or transliterations, no smart
  quotes.
- **No code-shaped tokens:** no file paths, function, class or variable names,
  database table names, commit hashes, branch names, version numbers, build
  hashes, error or HTTP status codes, build or shell commands, URLs.
- **No engineering acronyms** (API, SDK, OCR, LLM, JWT, HTTP, SSO, BE, FE and
  similar). Business acronyms the readers use daily (GST, VAT, UAT, PR) are
  fine.
- **One flowing paragraph:** no bullets, tables, headings, inline code or bold
  inside the text.
- **Outcome-led, not implementation-led.**
  - Engineer phrasing: "Added a non-null owner column with a migration on three
    client tables."
  - Human Summary phrasing: "Added a field on every client record to show which
    staff member owns it."
- **Precise verbs:** implemented, built, added, fixed, patched, debugged,
  refactored, validated, reviewed, deployed, reproduced, root-caused, migrated.
  Avoid filler: "successfully", "seamlessly", "leveraged", "robust solution",
  "best practices", "state-of-the-art".
- **Unique per task.** Two sections must not share a title or a near-identical
  summary; if they truly describe the same work, merge them.
- **Lengths:** title 8-15 words (a sentence, not a bare noun phrase or a
  question); summary 80-150 words, aiming for the middle.
- **Date** is required and matches the file's date. **Duration** stays blank
  unless the user gives a figure to forward.

Downstream tools find the block by the literal line `> **Human Summary:**` and
the four markers `Task:`, `Task summary:`, `Date:`, `Duration:`. Never rename
or reorder them.

---

## 6. Session-end self-audit

Before the session wraps up ("done", "thanks", "that's all", a push request):

- Every task message in the session has a `###` section.
- Every section is Completed, or In Progress / Blocked with a stated reason.
- Every Completed section has a filled Human Summary with no placeholder text.

Backfill anything missing from the conversation and the git history
(`git status --short`, `git diff`, `git log`), marking it
`Backfill note (DD-MM-YYYY): ...`. A backfilled section still gets a full
Human Summary.

---

## 7. Daily-file template

```markdown
# Work Summary - DD-MM-YYYY

## Session Info
- **Date:** <full date, e.g. Friday, 15 May 2026>
- **Carried Over From:** <date of the latest earlier file, or - if none>
- **Agents / areas active today:** <list>

## Today's Work

<!-- One ### section per task: opened on receive, closed when done. -->

---

## End of Day Summary
- **Completed:**
- **In Progress:**
- **Blocked:**
- **Tomorrow:**
```

---

## 8. CLAUDE.md block (append once, during bootstrap)

```markdown
## Work Summary Logging

Work is logged in `WORK_SUMMARY/WORK_SUMMARY-DD-MM-YYYY.md` (one file per day,
never committed). Every task the user gives gets its OWN `###` section: opened
the moment the task is received (Status: In Progress), filled during the work,
and closed (Status: Completed) with a filled Human Summary BEFORE telling the
user it is done.

Each section has two parts:
1. Technical Details: request, files, root cause, decisions, commit IDs,
   verification. Free-form markdown.
2. Human Summary, in exactly this shape:
   > **Human Summary:**
   > Task: <8-15 word outcome-led title>
   > Task summary: <one 80-150 word plain-English paragraph; no file paths,
   > function names, commit hashes, engineering acronyms or error codes>
   > Date: <DD Mon YYYY>
   > Duration:

One section per task; refinements of an open task append to its section;
questions that change nothing on disk get no section. Parallel sessions append
to today's file and never overwrite it. At session end, audit that every task
has a Completed section with a filled Human Summary.

The `work-summary` skill enforces this lifecycle.
```

---

## Related skill

`end-of-day-summary` reads the day's file and prints a shareable "Task N:"
summary. It only reads; recording work is this skill's job.
