---
description: Check the milestone and close the approved phase
---

Load the `foreman:foreman` skill with the Skill tool and follow it. It holds the file formats and the rules.

When a step below proposes a task, stop after it: on the user's yes, add the task to the phase and tell them to run `/foreman:next`.

1. Find the phase with status `approved` in `docs/plan/phases/`. If there is none, say so and stop. If it already has a Recap, the milestone has passed and this run is the user's yes: go to the yes case of step 7.
2. If `git status` shows uncommitted changes outside `docs/plan/`, ask the user whether any are follow-up tweaks not yet logged. Log those as `Amended:` under their task, and commit them as the skill's Commits section says.
3. If tasks are still unticked, list them and ask whether to close anyway. On yes, they move to `BACKLOG.md` when the phase closes. Find the `Next needs (task N)` lines with `grep -n 'Next needs (task' <phase file>`. One left for a ticked task is work that may have been dropped: show it and ask whether it was done. If not, propose a task for it.
4. Run the milestone command with a 10-minute timeout, as `<command> > "${TMPDIR:-/tmp}/milestone.log" 2>&1; echo "exit $?"; tail -n 20 "${TMPDIR:-/tmp}/milestone.log"`. The exit code must come from the command itself, never from a pipe.
5. If it fails because of the environment, such as a sandbox or a missing service, say so. Ask the user to run the command and report its exit code, and record under Decisions that they ran it. If it fails because of the code, show the exit code and the output, and propose a task that fixes it.
6. Move each `Next needs (later tasks)` line that still holds into Decisions, and delete each decision a later one replaced, since later phases read only Goal, Decisions and Recap. Write the Recap: three to six plain sentences on what the phase delivered, for a teammate who was not there. Name any task or check moved to `BACKLOG.md`. Each claim the Recap makes about the code needs a `retriever` that checked it in the code, as in `/foreman:plan`, not only in the plan. If a claim does not hold, show it and propose a task that fixes it.
7. Ask the user to confirm, by saying yes or running `/foreman:close` again. On yes, set the status to `closed`, check the phase in `PLAN.md`, move unticked tasks to `BACKLOG.md` as `- Task (phase NN): <task>` and unticked `After deploying` checks as `- Check (phase NN): <check>`. If every phase in `PLAN.md` is now closed, ask whether the plan's goal is met. On yes, move `PLAN.md` and `phases/` to `docs/plan/archive/<YYYY-MM-DD>-<slug>/`, with today's date and a slug from the goal, and leave `BACKLOG.md` where it is. Commit the plan files, including any archive move, as the skill's Commits section says. Then suggest a fresh session, such as with `/clear`, before the next `/foreman:plan`.
