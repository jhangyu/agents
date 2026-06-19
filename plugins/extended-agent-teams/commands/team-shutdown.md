---
description: "Gracefully shut down an agent team, flush documentation, collect final results, and clean up resources"
argument-hint: "[team-name] [--force] [--keep-tasks] [--skip-doc-flush]"
---

## Language Policy

All inter-agent communication — task descriptions, SendMessage between agents, TaskUpdate notes, broadcast messages, and all coordination between the main agent, team-lead, and team members — MUST use **English only**. This applies to every phase of team operation.

The main agent (the agent that invoked this command) uses the user's preferred language (as configured in CLAUDE.md) **only** when presenting the final consolidated summary or report directly to the user.

## Phase 1: Pre-Shutdown

1. Parse args: team name, `--force`, `--keep-tasks`, `--skip-doc-flush`
2. Read team config from `~/.claude/teams/{team-name}/config.json`
3. Call TaskList to check for in-progress tasks
4. If in-progress tasks exist and `--force` is **not** set:
   - Warn the user
   - List all in-progress tasks with their owners
   - Ask: "These tasks are still in progress. Proceed with shutdown anyway? (y/n)"
   - Abort if user declines

---

## Phase 2: Documentation Flush

Unless `--skip-doc-flush` is set:
1. Check if `team-doc-updater` is present in the team config
2. If yes: call SendMessage to `team-doc-updater` (subagent_type: `extended-agent-teams/team-doc-updater`) with:
   > "Final flush before shutdown. Please complete any pending documentation updates and confirm when done."
3. Wait for team-doc-updater to confirm (or timeout after 60 seconds)
4. Report: "Docs flushed: {list of files updated}"

If `--skip-doc-flush` is set or `team-doc-updater` is not in the team: skip this phase and note it in the summary.

---

## Phase 3: Graceful Shutdown

Send shutdown messages in the following order:
1. **Implementers, reviewers, debuggers** (in parallel)
2. **team-test-runner**
3. **team-doc-updater**
4. **team-lead** (last)

For each member:
1. Call SendMessage with `type: "shutdown_request"`:
   > "Team shutdown requested. Please finish your current work, save any state, and confirm shutdown."
2. If `--force` is not set: wait for shutdown confirmation response before proceeding to the next group
3. Mark the member as shut down

---

## Phase 4: Cleanup

Display the shutdown summary:

```
Team "{team-name}" shutdown complete.

Members shut down: {N}/{total}
Tasks completed: {completed}/{total}
Tasks remaining: {remaining}
Docs updated: {list from doc-updater flush}
```

Unless `--keep-tasks` is set: call `TeamDelete` to remove the team and task directories under `~/.claude/teams/{team-name}/`.
