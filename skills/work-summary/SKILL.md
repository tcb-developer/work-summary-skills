---
name: work-summary
description: Zero-miss daily work log that doubles as a low-token memory for coding agents. Every task gets its own section in WORK_SUMMARY/WORK_SUMMARY-DD-MM-YYYY.md (opened on receive, closed with a plain-English Human Summary before "done"), plus a one-line row in WORK_SUMMARY/INDEX.md and durable codebase facts in WORK_SUMMARY/CONTEXT.md, so the next session resumes from a few hundred tokens instead of re-reading files. Use PROACTIVELY at session start (resume), on every code-touching task, at completion, and at wrap-up. Also triggers on "/work-summary", "set up work summaries here", "what did we do last time", "log this task".
---

# work-summary - daily work log + low-token agent memory

The log serves two readers:

- **People** (managers, HR, timesheets) read the Human Summary of each task.
- **The next agent** reads the index, the context file and the compact
  Technical Details, so it can pick up the last work and find the right code
  without re-exploring the repository.

Writing a few precise lines now saves thousands of tokens of re-discovery later.
Every rule below serves one of those two readers.

## 1. Files

Folder `<WS>`, resolved in order, first match wins:
1. a path pinned in the project's `CLAUDE.md` / `AGENTS.md`;
2. `<git toplevel>/WORK_SUMMARY/`, or `<cwd>/WORK_SUMMARY/` outside git.
If two folders are plausible, use the one `CLAUDE.md` names, or ask once.

| File | Purpose | Size rule |
|---|---|---|
| `WORK_SUMMARY-DD-MM-YYYY.md` | one file per day, one `###` section per task | sections stay compact (section 4) |
| `INDEX.md` | one line per task, newest first | one line each, never paragraphs |
| `CONTEXT.md` | durable project facts for agents | stay under about 120 lines; replace, do not append |

- Never commit `<WS>`. Inside git, list it in `.git/info/exclude`.
- Parallel sessions: read before writing, append or insert, never overwrite
  another agent's lines.
- Today's date comes from the harness date line (cross-check the clock); the
  section time comes from `date "+%H:%M"`.
- First use in a project: follow `reference/bootstrap.md` (creates the folder,
  the templates and a short `CLAUDE.md` block). Read it only then.

## 2. Resume cheaply (session start, or before touching an area)

Do this before exploring the codebase. Stop as soon as you have enough.

1. Read `CONTEXT.md` in full (it is small by design).
2. Read the newest rows only: `head -n 25 <WS>/INDEX.md`.
3. Find past work on what you are about to touch:
   `grep -n -i "<file name|area|keyword>" <WS>/INDEX.md`
4. To open one past task, read only its section, never the whole day file:
   `grep -n "^### " <WS>/WORK_SUMMARY-DD-MM-YYYY.md`, then Read with
   `offset`/`limit` for that section.
5. Jump straight to the code named in `Touched:`. Before trusting line numbers,
   check whether the file changed since the recorded commit
   (`git log --oneline <commit>..HEAD -- <path>`). If it did, locate the
   recorded symbol by name instead.
6. Rows with status `Open` or `Blocked` are unfinished work. Mention them to
   the user if they are related to the new request.

Do not read old day files end to end, and do not re-map code that
`CONTEXT.md` already maps unless it looks stale.

## 3. Task lifecycle (every task, no exceptions)

1. **On receive, before any work:** append a section to today's file with
   `**Status:** In Progress`, the `**Request:**` line and an empty Human
   Summary block. Insert an `Open` row at the top of `INDEX.md`. Save both.
2. **During work:** fill the agent fields (section 4) as facts become known.
3. **On completion, before saying "done":** set `**Status:** Completed`, fill
   the Human Summary (section 5), and flip the index row to `Done` (adding
   files and commit). If you learned a durable fact, update `CONTEXT.md`.
4. **End of day:** update the `## End of Day Summary` block (Completed, In
   Progress, Blocked with reason, Tomorrow).

Edge cases (fixed decisions):
- Every message that asks for a change gets its own section and index row,
  even a one-line fix. Three tasks in one message means three sections.
- A new task while another is in progress gets a new section at once. The old
  one stays In Progress.
- Iterations on the same task ("bigger", "now thinner") stay in one section;
  add a line under `**Change:**`. The Human Summary describes the final state.
- Pure conversation that writes nothing to disk gets no section. When in
  doubt, write one.
- Testing of a new feature goes in its own section:
  `### HH:MM - [AREA] Testing: <feature>` with the cases, a bug log, the cycle
  count and the final result.

## 4. Section format (agent fields are terse facts)

```markdown
### HH:MM - [Area] <task title>

**Status:** In Progress | Completed | Blocked (<reason>)
**Request:** <the ask, as one clear engineering statement>
**Touched:** `path/file.py:120-168` (`function_name`), `path/view.js` (`onSave`)
**Cause:** <root cause in one or two sentences; bugs only>
**Change:** <what changed and why, one line per change>
**Decisions:** <choices made, rejected options, user confirmations>
**Verify:** `<command>` -> <result>
**Commit:** <short sha(s)> on <branch>, or "uncommitted"
**Next:** <open threads, follow-ups, or ->

> **Human Summary:**
> Task: ...
> Task summary: ...
> Date: DD Mon YYYY
> Duration:

---
```

Rules for the agent fields:
- Write facts, not a story. No "first I looked at...". Leave out fields that
  do not apply.
- `Touched` always names the path and the symbol, with line ranges when
  known. The symbol keeps the pointer valid after the line numbers drift.
- Never paste code, diffs or long logs. The commit holds the diff. Quote at
  most the one log line that proves a point.
- Aim for under 20 lines before the Human Summary. Add more only when the
  user asks for detail.
- Keep the `[Area]` tag consistent from day to day (same spelling), because
  agents grep for it.

### INDEX.md row

Newest first, directly under the header. One line per task:

```
2026-05-15 10:12 | Done | Invoicing | Invoice PDF totals use invoice currency | templates/invoice_pdf.html, tests/test_invoice_pdf.py | 3f9c2ab
```

Columns: date, time, `Open`/`Done`/`Blocked`, area, short title, main files
(paths only, at most four), commit or `-`. Keep each row under about 200
characters. The day file and time locate the full section.

### CONTEXT.md

This is a project map for agents, not a diary. Allowed content:
- how to run, test, build and lint (exact commands);
- where things live (`area -> path`, with entry points);
- conventions, gotchas and non-obvious constraints, each costly to rediscover.

Write a fact only when it is stable and you had to spend real effort to find
it. Edit the existing line rather than adding a near-duplicate. Delete facts
that turn out to be wrong. Do not repeat what `CLAUDE.md` already says. Keep
an `Updated: YYYY-MM-DD` line at the top.

## 5. Human Summary (for people; the format is a contract)

```
> **Human Summary:**
> Task: <8-15 word, outcome-led title>
> Task summary: <one plain-English paragraph, 80-150 words>
> Date: <DD Mon YYYY, the file's date>
> Duration:
```

- Plain English, one flowing paragraph, outcome-led (what the user or the
  business gets, not how the code is built).
- No code-shaped tokens: paths, symbols, table names, hashes, branches,
  versions, error or status codes, commands, URLs.
- No engineering acronyms (API, SDK, LLM, JWT, HTTP, SSO...). Business
  acronyms the readers use (GST, VAT, UAT, PR) are fine.
- Precise verbs (built, fixed, added, validated, migrated). No filler
  ("successfully", "seamlessly", "robust").
- Each task gets a unique title and summary. `Duration` stays blank unless the
  user gives a figure.
- Never rename or reorder the five marker lines; extractors grep for them.

## 6. Session-end audit

Before wrap-up ("done", "thanks", a push request), check:
- every task has a section and an index row;
- every section is Completed, or In Progress / Blocked with a reason, and its
  index status matches;
- every Completed section has a filled Human Summary with no placeholders;
- `CONTEXT.md` reflects anything durable learned today and is still under the
  size cap.

Backfill anything missing from the conversation and `git log` / `git status`,
marked `Backfill note (DD-MM-YYYY)`.

## Related skill

`end-of-day-summary` turns a day file into a shareable "Task N:" summary. It
only reads.
