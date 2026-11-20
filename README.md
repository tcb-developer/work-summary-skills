# work-summary-skills

Two [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills that keep a
complete, daily record of the work an agent does for you, and turn it into a
summary you can share.

| Skill | What it does |
|---|---|
| [`work-summary`](skills/work-summary/SKILL.md) | Logs every task in `WORK_SUMMARY/WORK_SUMMARY-DD-MM-YYYY.md`. A section is opened the moment a task arrives, filled during the work, and closed with a plain-English Human Summary before the agent says "done". A session-end audit catches anything missed. |
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

See [`examples/WORK_SUMMARY-15-05-2026.md`](examples/WORK_SUMMARY-15-05-2026.md)
for a sample day: a bug fix, its testing section with a bug log, and a task
still in progress.

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
  here"). It creates the `WORK_SUMMARY/` folder and today's file, keeps the
  folder out of git, and adds a short "Work Summary Logging" block to the
  project's `CLAUDE.md` so every later session follows the rules.
- **Day to day:** nothing to do. Each task you give gets its own section, and
  the agent fills the Human Summary before reporting the task as done.
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
