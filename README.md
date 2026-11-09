# <img src=".claude-plugin/icon.png" width="36" align="top" alt=""> Foreman

A Claude Code plugin that makes Claude work like a disciplined contractor.

On a long job Claude can lose track of what was decided, say something works without running it, or delete the test that was failing. With Foreman you approve the plan one phase at a time. A fresh agent builds each task and a second one tries to prove it wrong to ensure quality.

Use it for feature development or bigger work that spans multiple files.

## Why not just plan mode

Plan mode gets you an approved plan, and then that session starts editing. Foreman saves the plan in your repo and keeps you in the loop one task at a time. Giving you more control and a better understanding of what was done.

- Your session stays small, because the coding, reviewing and reading happen in separate agents on Sonnet and Haiku. That leaves your session to plan and direct the work, so give it your strongest model.
- The plan is saved in `docs/plan/` and committed with the work, so a new session or a teammate can pick up where you stopped.
- A task is only ticked off after its checks have run and the reviewer found nothing that must be fixed.
- Nothing is built until you approve it. Claude never pushes, and it only commits if you agree to it.
- If Claude commits, each phase is a branch and each task its own commit.

## Install

```
/plugin marketplace add myrheimb/foreman
/plugin install foreman@foreman
```

## Use

1. Run `/foreman:plan <what you want to build>`. Claude drafts a phase with a goal and the tasks to reach it. It also sets out how to prove the goal was met, with a command to run and anything you may need to check by hand. You edit it until you are happy and approve it.
2. Run `/foreman:next`. One agent builds the next task and another reviews it. Repeat until all tasks are done.
3. Run `/foreman:close`. Claude runs the command from step 1, writes a short summary and closes the phase once you confirm.

Foreman keeps the plan in your repo.

```
docs/plan/
├── archive/              finished plans
├── BACKLOG.md            ideas, bugs and open work
├── PLAN.md               the goal, who commits, the list of phases
└── phases/
    └── 01-csv-export.md  goal, milestone, tasks, log, recap
```

- Claude stops and asks you if a check fails or can't run, or if the reviewer finds something that must be fixed.
- Start a new session when a phase closes or after a longer break. Everything Claude needs is in the plan files.
- Write ideas and bugs in `docs/plan/BACKLOG.md` in your own words. Claude goes through them with you every time you run `/foreman:plan`.
- Agents run tests through your normal permissions, so a task can stop and wait for your approval. Auto mode or `/fewer-permission-prompts` cuts down on the prompts.
- Each task also runs a separate review agent. It adds to the cost, but no agent reviews its own work, so bugs are more likely to get caught.

## What's in the plugin

Only Markdown skills, commands and agents. There are no hooks or scripts, so the plugin itself never touches the network. Claude and its agents work under your normal permissions, and can use the web, MCP servers and your tools as Claude Code usually would. Uninstalling the plugin removes all of it, and your plan files stay.

The reviewer and retriever agents don't have edit privileges and are instructed never to use their bash access to change your files.

## License

MIT, see [LICENSE](LICENSE).
