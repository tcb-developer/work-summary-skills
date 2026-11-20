# work-summary-skills

Two [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills that keep a
complete, daily record of the work an agent does for you, turn it into a
summary you can share, and give the next agent session a cheap way to pick up
where the last one stopped.

| Skill | What it does |
|---|---|
| [`work-summary`](skills/work-summary/SKILL.md) | Logs every task in `WORK_SUMMARY/WORK_SUMMARY-DD-MM-YYYY.md`. A section is opened the moment a task arrives, filled during the work, and closed with a plain-English Human Summary before the agent says "done". It also keeps a one-line-per-task `INDEX.md` and a small `CONTEXT.md` project map, so later sessions resume from a few hundred tokens instead of re-reading code. |
| [`end-of-day-summary`](skills/end-of-day-summary/SKILL.md) | Reads the day's file and prints a copy-ready "Task N:" summary in your project's own terminology. Read-only. |

## Why

Without rules, long agent sessions tend to:

- bundle many tasks into a few sections written at the end of the day,
- lose tasks that were interrupted halfway,
- describe work in file paths and function names that managers, HR and
  accountants cannot read.

These skills fix that with a fixed task lifecycle (open on receive, fill during
work, close before "done", audit at session end) and a strict two-part format
for every task:

1. **Technical Details** for the next engineer: request, files, root cause,
   commits, verification.
2. **Human Summary** for everyone else: an 8-15 word outcome-led title and one
   80-150 word plain-English paragraph, with no code, paths, hashes or
   engineering acronyms.

The Human Summary uses fixed markers (`> **Human Summary:**`, `Task:`,
`Task summary:`, `Date:`, `Duration:`), so a script can lift it straight into a
timesheet or report.

## Fewer tokens for the next agent

A new agent session normally spends a large part of its budget re-reading
files and re-exploring code just to learn what happened last time. The log is
built to stop that:

- **`INDEX.md`**: one line per task (date, status, area, title, files,
  commit), newest first. An agent reads the top 25 lines or greps for a file
  name to find past work on it.
- **`CONTEXT.md`**: a capped project map (run and test commands, where things
  live, gotchas), holding only facts that took real effort to discover. It is
  edited in place, never appended to like a diary.
- **Compact task sections**: fixed fields (`Touched`, `Cause`, `Change`,
  `Decisions`, `Verify`, `Commit`, `Next`). `Touched` names path, line range
  and symbol, so the agent can jump straight to the code. No pasted code or
  logs; the commit holds the diff.
- **A resume routine**: read `CONTEXT.md`, then the top of `INDEX.md`, then
  only the one section needed (by heading line and offset), then check whether
  the code changed since the recorded commit. Day files are never read end to
  end.
- **A light skill**: the setup templates live in `reference/bootstrap.md` and
  are read only on first use. The `CLAUDE.md` block, which loads in every
  session, is six lines.

## Examples

- [`examples/WORK_SUMMARY-15-05-2026.md`](examples/WORK_SUMMARY-15-05-2026.md):
  a bug fix, its testing section with a bug log, and a task still in progress.
- [`examples/INDEX.md`](examples/INDEX.md) and
  [`examples/CONTEXT.md`](examples/CONTEXT.md): the matching index and
  project map.

## Install

Personal (all your projects):

```bash
git clone https://github.com/tcb-developer/work-summary-skills.git
mkdir -p ~/.claude/skills
cp -R work-summary-skills/skills/* ~/.claude/skills/
```

One project only (shared with everyone who works on the repository):

```bash
mkdir -p .claude/skills
cp -R /path/to/work-summary-skills/skills/* .claude/skills/
```

Start a new Claude Code session afterwards so the skills are picked up.

## Use

- **Set up a project:** run `/work-summary` (or say "set up work summaries
  here"). It creates the `WORK_SUMMARY/` folder with today's file, `INDEX.md`
  and `CONTEXT.md`, keeps the folder out of git, and adds a short "Work
  Summary Logging" block to the project's `CLAUDE.md` so every later session
  follows the rules.
- **Day to day:** nothing to do. Each session starts by resuming from the
  index and context file. Each task you give gets its own section and index
  row, and the agent fills the Human Summary before reporting the task as
  done.
- **Ask about the past:** "what did we do on the invoice PDF?" is answered from
  `INDEX.md` and one section, not from a fresh search of the code.
- **End of the day:** run `/end-of-day-summary` (or say "give me today's work
  summary") and copy the block it prints. Name a date to summarise another day.

## Where the files go

By default `WORK_SUMMARY/` sits at the root of the current git repository and is
added to `.git/info/exclude`, so it is never committed and survives branch
switches. To keep it somewhere else (for example outside the repository, which
also protects it from `git clean -x`), name the folder in your project's
`CLAUDE.md`; both skills use a pinned path first.

Work summaries are local records. Do not commit them to a shared repository.

## Licence

[MIT](LICENSE)
