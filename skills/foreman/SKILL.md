---
name: foreman
description: Plan-driven workflow in phases. Use when the user wants to plan work in phases, run the next planned task, or close a phase.
user-invocable: false
---

# Foreman

Work happens in phases. Each phase has a goal, a short list of tasks, and a milestone: one command that proves the goal is met. The plan is kept in `docs/plan/` and committed with the work, so every session picks up from the same files.

## Files

- `docs/plan/PLAN.md`: the plan's goal, then `Commits: me` or `Commits: foreman`, then one line per phase: `- [ ] [01 <title>](phases/01-<slug>.md): <goal>`, checked when the phase closes.
- `docs/plan/BACKLOG.md`: ideas, annoyances, bugs and open work no phase has taken yet, one line each, kept from plan to plan. It always exists: a command that finds it missing creates it with a `# Backlog` heading. The user writes there in their own words. Claude writes `- Bug: <what happens>, <where the cause likely lives>`, `- Idea: <what>`, `- Task (phase NN): <task>` for a task left open when its phase closed, or `- Check (phase NN): <check>` for an after-deploy check still open.
- `docs/plan/archive/<YYYY-MM-DD>-<slug>/`: a finished plan, its `PLAN.md` and `phases/`. No command reads it. A question about an old decision goes to the `retriever`.
- `docs/plan/phases/NN-<slug>.md`: one file per phase, in this shape:

```markdown
# Phase NN: <title>

Status: draft

## Goal

<One or two sentences: what is true when this phase is done.>

## Milestone

`<command that exits 0 when the goal is met>`

After deploying:

- [ ] <a check the command cannot run, such as one on a live host or one only a person can judge>

## Tasks

- [ ] 1. <task>. Done when <observable result>. Files: <paths it likely touches>.

## Log

## Decisions

## Recap
```

Leave out the `After deploying` list when the command covers every check. Status moves `draft` → `approved` → `closed`. Decisions go in as `- YYYY-MM-DD <decision>: <why>`. A decision that replaces an earlier one replaces its line. A decision that every X must do Y also needs a check the milestone runs, preferably one the repository enforces, such as an assertion or a lint. Log entries go under `### Task N`, at most five lines, leaving out the lines that don't apply:

```markdown
### Task N

Done: <one line on what changed>.
Next needs (task M | later tasks): <what that task, or every later task, must know>.
Assumed: <a choice the plan left open, and the reading the coder took>.
Unverified: <checks that did not run, and why>.
Failed: <a check the user ran that failed, and its key error line>.
Review failed: <each must-fix finding: what is wrong at path:line, fixed when check>.
Amended: <follow-up tweaks the user asked for after the task>.
User did: <a manual step the user reports done, such as re-encrypting a secret>.
Blocked: <what is in the way>.
```

Merge lines of one kind into one. Tables, measurements, the coder's Verified lines and its full summary, and the reviewer's reply beyond its must-fix findings go to the user or the commit message, not the log.

## Rules

1. Nothing is built for a phase until its status is `approved`, and only the user approves.
2. One task at a time, each in a fresh context through the `coder` agent, then reviewed in another fresh context by the `reviewer` agent. The main session holds the plan, not the code. The exception is a follow-up tweak the user asks for after a task, such as a visual change from a screenshot, that stays in the files the task touched and adds or changes no test: make it in the main session, log it as `Amended:` under that task, and rewrite the task if its outcome changed. Any other follow-up, such as one that changes behaviour, needs a test or reaches other files, is a new task: propose it as rule 9 says instead of making it.
3. Send long reading (big files, many files, "where is X handled") to the `retriever` agent and keep its answer, not the files.
4. After each task, add its log entry. Tick it only when every "Done when" check has passed, whether the coder ran it or the user did, and the reviewer passed it. The log is the memory for later tasks and sessions: short and factual. When the user reports a manual step done that a task or log line asked of them, log it as `User did:` under that task, and never ask for it again. When the user reports an `After deploying` check passed, tick it.
5. A task that needs something outside the phase goal stops and asks the user. The answer goes under Decisions.
6. A phase closes only when its milestone command passes and the user confirms.
7. The plan files are the only state. Never rely on what an earlier session remembers.
8. Never push, or change settings or permissions on the user's behalf. Commit only as the Commits section says.
9. An approved phase changes only on the user's yes. A new task goes after the open ones, work moved from one task to another is rewritten in both, and the change goes under Decisions. When a failure prompts the change, rather than a new wish from the user, first ask the user whether it shows that the phase's goal or approach is wrong. When the check itself is the suspect, because it passes on a rerun or fails differently each time, also offer a task that makes it reliable, run before the failing task's next coder. Offer closing the phase early with `/foreman:close` alongside any new tasks.
10. When the user mentions a bug, an idea or an annoyance that the current task will not handle, add it to `BACKLOG.md` and say so.

## Commits

`PLAN.md` says who commits. The user chooses when the plan is created, and a plan without the line means `Commits: me`. A phase gets one commit for its approved plan files, one per finished task with the plan files it changed, and one when it closes.

With `Commits: me`, give the `git add -- <paths>` command and suggest a commit message in its own block.

With `Commits: foreman`, commit yourself on the phase's branch, with a message in the repository's style, from its CLAUDE.md and recent `git log`:

- Run `git status --short` first. Stage only this commit's paths and leave other changes alone:

  ```
  git add -- <paths> && git commit -F - > "${TMPDIR:-/tmp}/commit.log" 2>&1 <<'EOF'; echo "exit $?"; tail -n 30 "${TMPDIR:-/tmp}/commit.log"
  <message>
  EOF
  ```

- A follow-up tweak to the task whose commit is HEAD is amended into that commit, with the phase file that logs it: the same command with `git commit --amend -F -` and the updated message. Amend only when `git branch -r --contains HEAD` prints nothing. Otherwise the tweak gets its own commit.
- If a hook fails the commit and its only new changes, compared with the earlier `git status --short`, are in this commit's paths, run the same command once more.
- If the commit still fails, show the key error lines and hand the commit to the user as with `Commits: me`. If the environment caused it, such as signing the sandbox cannot reach or a hook that needs the network, ask whether to switch to `Commits: me`. On yes, change the line in `PLAN.md` and record it under Decisions.
- Never push, rebase, reset or force, or skip hooks or signing.

## Commands

- `/foreman:plan <what>`: draft the next phase with the user.
- `/foreman:next [task]`: run one task of the approved phase.
- `/foreman:close`: check the milestone and close the phase.
