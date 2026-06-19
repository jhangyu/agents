---
description: "Spawn an agent team using presets (review, debug, feature, fullstack, refactor, techdebt, performance, systems, research, security, migration) or custom composition"
argument-hint: "<preset|custom> [--name team-name] [--members N] [--delegate] [--lang c|cpp|go|rust]"
---

# team-spawn

Create & spawn a coordinated agent team using a preset configuration or custom composition.

## Pre-flight Checks

1. Verify the `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` environment variable is set. If not, halt and instruct the user to set it before proceeding.
2. Parse arguments:
   - First positional arg: preset name or `custom`
   - `--name <team-name>`: override default team name
   - `--members N`: override default member count (custom mode)
   - `--delegate`: pass task delegation hints to agents
   - `--lang c|cpp|go|rust`: language specialization for `performance` and `systems` presets

## Preset Configurations

All agents use subagent_type prefix `extended-agent-teams:`.

### `review`
Default name: `review-team`
Members:
- `extended-agent-teams:team-reviewer` — focus: security
- `extended-agent-teams:team-reviewer` — focus: performance
- `extended-agent-teams:team-reviewer` — focus: architecture
- `extended-agent-teams:team-doc-updater` — documents review findings

### `debug`
Default name: `debug-team`
Members:
- `extended-agent-teams:team-debugger` — hypothesis 1
- `extended-agent-teams:team-debugger` — hypothesis 2
- `extended-agent-teams:team-debugger` — hypothesis 3
- `extended-agent-teams:team-doc-updater` — documents debug runbook

### `feature`
Default name: `feature-team`
Members:
- `extended-agent-teams:team-lead` — coordinates streams, owns integration
- `extended-agent-teams:team-implementer` — stream 1
- `extended-agent-teams:team-implementer` — stream 2
- `extended-agent-teams:team-test-runner` — runs tests after each stream completes, reports PASS/FAIL
- `extended-agent-teams:team-doc-updater` — updates docs at leader checkpoints

### `fullstack`
Default name: `fullstack-team`
Members:
- `extended-agent-teams:team-lead` — coordinates all layers
- `extended-agent-teams:team-implementer` — frontend
- `extended-agent-teams:team-implementer` — backend
- `extended-agent-teams:team-implementer` — tests
- `extended-agent-teams:team-test-runner` — runs test/build commands, reports PASS/FAIL
- `extended-agent-teams:team-doc-updater` — updates docs at leader checkpoints

### `research`
Default name: `research-team`
Members:
- `extended-agent-teams:general-purpose` — research area 1
- `extended-agent-teams:general-purpose` — research area 2
- `extended-agent-teams:general-purpose` — research area 3

### `security`
Default name: `security-team`
Members:
- `extended-agent-teams:team-reviewer` — OWASP top 10 review
- `extended-agent-teams:team-reviewer` — auth & access control
- `extended-agent-teams:team-reviewer` — dependency vulnerabilities
- `extended-agent-teams:team-reviewer` — secrets & config exposure

### `migration`
Default name: `migration-team`
Members:
- `extended-agent-teams:team-lead` — coordinates migration plan
- `extended-agent-teams:team-implementer` — migration stream 1
- `extended-agent-teams:team-implementer` — migration stream 2
- `extended-agent-teams:team-reviewer` — verifies migration correctness
- `extended-agent-teams:team-test-runner` — runs test/build commands after each stream, reports PASS/FAIL
- `extended-agent-teams:team-doc-updater` — updates migration docs at leader checkpoints

### `refactor`
Default name: `refactor-team`
Members:
- `extended-agent-teams:team-lead` — owns refactor plan and integration
- `extended-agent-teams:legacy-modernizer` — identifies and rewrites legacy patterns
- `extended-agent-teams:team-implementer` — refactor stream 1
- `extended-agent-teams:team-implementer` — refactor stream 2
- `extended-agent-teams:team-test-runner` — runs tests after each stream completes, reports PASS/FAIL
- `extended-agent-teams:team-reviewer` — verifies refactor correctness
- `extended-agent-teams:team-doc-updater` — updates docs at leader checkpoints

### `techdebt`
Default name: `techdebt-team`
Members:
- `extended-agent-teams:architect-reviewer` — evaluates structural and design debt
- `extended-agent-teams:code-reviewer` — evaluates code-level debt
- `extended-agent-teams:legacy-modernizer` — proposes and applies modernization
- `extended-agent-teams:team-test-runner` — runs tests after each stream completes, reports PASS/FAIL
- `extended-agent-teams:team-doc-updater` — documents findings and recommended roadmap

### `performance`
Default name: `performance-team`
Members:
- `extended-agent-teams:performance-engineer` — profiling, bottleneck identification, optimization
- `extended-agent-teams:architect-reviewer` — architectural performance concerns
- `extended-agent-teams:team-test-runner` — runs benchmarks and performance tests, reports PASS/FAIL
- `extended-agent-teams:team-doc-updater` — documents findings and improvements

If `--lang` is specified, also spawn the corresponding language-specialist agent:
- `--lang c` → `extended-agent-teams:c-pro`
- `--lang cpp` → `extended-agent-teams:cpp-pro`
- `--lang go` → `extended-agent-teams:golang-pro`
- `--lang rust` → `extended-agent-teams:rust-pro`

### `systems`
Default name: `systems-team`
Members depend on `--lang` flag. If `--lang` is not provided, use AskUserQuestion to ask which language: C, C++, Go, or Rust.

Base members (always included):
- `extended-agent-teams:team-lead` — owns system design and integration
- `extended-agent-teams:team-test-runner` — runs system tests, reports PASS/FAIL

Language-specific additions:
- `--lang c` → `extended-agent-teams:c-pro`
- `--lang cpp` → `extended-agent-teams:cpp-pro`
- `--lang go` → `extended-agent-teams:golang-pro`
- `--lang rust` → `extended-agent-teams:rust-pro`

## Custom Composition

If the first argument is `custom`:
1. Use AskUserQuestion to present the full agent role list and ask the user to select members:
   - `team-lead`
   - `team-reviewer`
   - `team-debugger`
   - `team-implementer`
   - `team-test-runner`
   - `team-doc-updater`
   - `architect-reviewer`
   - `legacy-modernizer`
   - `performance-engineer`
   - `c-pro`
   - `cpp-pro`
   - `golang-pro`
   - `rust-pro`
2. Ask for team name if `--name` was not provided.
3. Use `--members N` to override the number of any repeatable role (e.g., multiple team-implementers).
4. Compose the final member list from user selections.

## Language Policy

All inter-agent communication — task descriptions, SendMessage between agents, TaskUpdate notes, broadcast messages, and all coordination between the main agent, team-lead, and team members — MUST use **English only**. This applies to every phase of team operation.

The main agent (the agent that invoked this command) uses the user's preferred language (as configured in CLAUDE.md) **only** when presenting the final consolidated summary or report directly to the user.

## Team Creation

1. Use the `TeamCreate` tool with `displayMode: "tmux"` to create the team. The tmux display mode enables real-time inter-agent communication and visibility across all team members.
2. Call TaskCreate once per member to assign their initial role context and any relevant instructions. All task prompts MUST be written in English.
3. If `--delegate` is set, include delegation hints in each task prompt: owned files, blockedBy relationships, and acceptance criteria placeholders.

## Output

Present a formatted summary after spawn completes:

```
Team: {team-name}
Preset: {preset or "custom"}

Members:
  [1] {subagent_type} — {role/focus}
  [2] {subagent_type} — {role/focus}
  ...

Status: All agents spawned. Tasks assigned.
```
