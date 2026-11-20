# Bootstrap (first use in a project; idempotent)

Read this only when `<WS>` or one of its files is missing.

1. Create `<WS>` if missing. Inside a git repository, add `WORK_SUMMARY/` to
   `.git/info/exclude` (a local-only ignore list that is never committed). Do
   not edit the shared `.gitignore` unless the user asks. Warn the user that
   `git clean -x` would still delete the folder; for full safety they can keep
   it outside the repository and pin that path in `CLAUDE.md`.
2. Create any missing file from the templates below. Never overwrite an
   existing one.
3. Seed `CONTEXT.md` lightly: fill only what you already know (the test and
   run commands from the README or package files). It grows as tasks teach
   you more. Do not scan the whole codebase to fill it.
4. If the project's `CLAUDE.md` (or `AGENTS.md`) has no
   `## Work Summary Logging` heading, append the block below. If neither file
   exists, create `CLAUDE.md` with just the block. Never rewrite the user's
   existing content.
5. Tell the user the folder path, which files were created, and whether
   `CLAUDE.md` was changed.

On later days, only step 2 applies (the new day file).

## Day file: `WORK_SUMMARY-DD-MM-YYYY.md`

```markdown
# Work Summary - DD-MM-YYYY

## Session Info
- **Date:** <e.g. Friday, 15 May 2026>
- **Carried Over From:** <latest earlier file date, or ->

## Today's Work

<!-- One ### section per task: opened on receive, closed when done. -->

---

## End of Day Summary
- **Completed:**
- **In Progress:**
- **Blocked:**
- **Tomorrow:**
```

## `INDEX.md`

```markdown
# Work Index
<!-- Newest first. One line per task: date time | Open/Done/Blocked | area | title | files | commit -->
<!-- Agents: read the first 25 lines, then grep by file, area or keyword. -->
```

## `CONTEXT.md`

```markdown
# Project Context (for agents)
Updated: YYYY-MM-DD
<!-- Durable, verified facts only. Under about 120 lines. Edit lines, do not append duplicates. -->

## Run / test
- 

## Map (area -> path, entry points)
- 

## Conventions and gotchas
- 
```

## `CLAUDE.md` block (append once)

Kept short on purpose, because `CLAUDE.md` is loaded into every session.

```markdown
## Work Summary Logging

Use the `work-summary` skill. At session start, resume from
`WORK_SUMMARY/CONTEXT.md` and the top of `WORK_SUMMARY/INDEX.md` before
exploring code. Every task gets its own section in
`WORK_SUMMARY/WORK_SUMMARY-DD-MM-YYYY.md` and a row in `INDEX.md`. Open it
when the task arrives, and close it with a filled Human Summary before
saying "done". Never commit `WORK_SUMMARY/`.
```
