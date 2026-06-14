---
name: extended-agent-team-workflows
description: Run extended agent-team workflows in Codex using the bundled Extended Agent Teams command and agent specifications. Use when the user asks for team-spawn, team-review, team-debug, team-feature, team-refactor, team-techdebt, team-performance, team-status, team-delegate, or team-shutdown behavior from the extended-agent-teams plugin, or asks Codex to coordinate specialized team roles such as team-lead, team-implementer, team-reviewer, team-debugger, team-test-runner, team-doc-updater, architect-reviewer, legacy-modernizer, performance-engineer, c-pro, cpp-pro, golang-pro, or rust-pro.
---

# Extended Agent Team Workflows

Use the Extended Agent Teams plugin as workflow specifications for Codex-native execution. The source specs live in the same plugin directory:

- `../../README.md` for available commands, agents, presets, and reactive role semantics
- `../../commands/*.md` for command workflows and argument conventions
- `../../agents/*.md` for role behavior, constraints, model hints, and output expectations

## Operating Model

Codex does not have Claude Code's `Teammate`, `TaskCreate`, `TaskList`, or agent-team runtime. Treat those references as coordination intent, then execute with Codex-native capabilities:

1. Read the relevant command spec from `../../commands/`.
2. Read only the agent specs needed for the requested workflow from `../../agents/`.
3. Translate the command phases into an explicit Codex plan.
4. Use available Codex tools directly for file reads, edits, tests, reviews, and summaries.
5. If parallel subagents are available in the current environment, use them only for independent read/review/research tasks with clear file ownership or read-only scope.
6. If subagents are unavailable or the task requires sequential edits, perform the workflow in-process and preserve the same role boundaries in the report.

Never claim that a Claude teammate was spawned unless the current environment provides an actual compatible team runtime.

## Command Routing

Map user intent to command specs:

- `team-spawn`: read `team-spawn.md` and `team-lead.md`; create a Codex execution plan using the requested preset or custom roles.
- `team-review`: read `team-review.md`, `team-reviewer.md`, `code-reviewer.md`, and any specialized reviewer specs requested by dimensions.
- `team-debug`: read `team-debug.md`, `team-debugger.md`, and `team-test-runner.md`.
- `team-feature`: read `team-feature.md`, `team-lead.md`, `team-implementer.md`, `team-test-runner.md`, and `team-doc-updater.md`.
- `team-refactor`: read `team-refactor.md`, `legacy-modernizer.md`, `team-implementer.md`, `code-reviewer.md`, and `team-test-runner.md`.
- `team-techdebt`: read `team-techdebt.md`, `architect-reviewer.md`, `code-reviewer.md`, `legacy-modernizer.md`, and `team-doc-updater.md`.
- `team-performance`: read `team-performance.md`, `performance-engineer.md`, `architect-reviewer.md`, and `team-test-runner.md`; also read language expert specs when the target is C, C++, Go, or Rust.
- `team-status`, `team-delegate`, `team-shutdown`: read the matching command spec and summarize or emulate the lifecycle action against the current Codex task state.

## Execution Rules

- Preserve file ownership boundaries from the command specs when editing.
- Run tests or verification commands called for by `team-test-runner` whenever they are discoverable and safe in the current repo.
- For documentation updates, apply the `team-doc-updater` constraints: edit documentation files only and keep changes minimal.
- For review-style workflows, lead with findings sorted by severity and include file/line references.
- For implementation workflows, report role streams, files changed, verification results, and any skipped runtime features.

## Reporting

When completing a workflow, explicitly state:

- Which command spec and agent specs were used
- Whether execution was parallel, subagent-assisted, or in-process
- What files changed or were reviewed
- What verification ran
- Any Claude-only runtime behavior that was translated or skipped
