# Extended Agent Teams

Extended multi-agent orchestration for Claude Code. Builds on the `agent-teams` plugin with specialized workflows for refactoring, technical debt analysis, and performance optimization. Adds language-specific expert agents and introduces two new support roles: automated test runner and documentation updater.

## Setup

### Prerequisites

Enable the experimental Agent Teams feature:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Configure teammate display mode in `~/.claude/settings.json`:

```json
{
  "teammateMode": "tmux"
}
```

## Commands

| Command | Description |
|---|---|
| `/team-spawn` | Spawn a team using presets or custom composition |
| `/team-status` | Display team members, tasks, and reactive agent states |
| `/team-shutdown` | Gracefully flush docs, shut down team, clean up resources |
| `/team-review` | Multi-reviewer parallel code review with auto doc update |
| `/team-debug` | Competing hypotheses debugging with repro verification and runbook update |
| `/team-feature` | Parallel feature development with test verification and doc tracking |
| `/team-delegate` | Task delegation dashboard with doc flush support |
| `/team-refactor` | Parallel refactoring with code-reviewer analysis and test gate |
| `/team-techdebt` | Technical debt inventory, scoring, and remediation roadmap |
| `/team-performance` | Performance profiling with language-specific expert analysis |

## Agents

### Core Orchestration

| Agent | Role | Model | Color |
|---|---|---|---|
| `team-lead` | Orchestrates team lifecycle, decomposes work, synthesizes results | opus | blue |
| `team-implementer` | Implements within file ownership boundaries | opus | yellow |
| `team-reviewer` | Single-dimension code reviewer | opus | green |
| `team-debugger` | Hypothesis-driven investigator | opus | red |

### New Support Roles

| Agent | Role | Model | Color |
|---|---|---|---|
| `team-test-runner` | Executes test/build scripts, reports PASS/FAIL/PARTIAL | haiku | cyan |
| `team-doc-updater` | Updates documentation at team-lead checkpoints | sonnet | magenta |

### Specialized Experts

| Agent | Role | Model |
|---|---|---|
| `architect-reviewer` | Architecture review, SOLID, DDD, scalability | opus |
| `legacy-modernizer` | Refactoring, framework migration, tech debt reduction | sonnet |
| `performance-engineer` | Profiling, observability, caching, optimization | opus |
| `c-pro` | C systems programming, memory management, POSIX | opus |
| `cpp-pro` | Modern C++, RAII, templates, STL | opus |
| `golang-pro` | Go 1.21+, concurrency, microservices | opus |
| `rust-pro` | Rust 1.75+, async, ownership, Tokio | opus |

## Presets

| Preset | Composition | Use Case |
|---|---|---|
| `review` | 3× reviewer + doc-updater | Multi-dimensional code review |
| `debug` | 3× debugger + doc-updater | Root cause analysis |
| `feature` | lead + 2× implementer + test-runner + doc-updater | New feature development |
| `fullstack` | lead + 3× implementer + test-runner + doc-updater | Full-stack feature |
| `refactor` | lead + legacy-modernizer + 2× implementer + test-runner + doc-updater | Code refactoring |
| `techdebt` | architect-reviewer + code-reviewer + legacy-modernizer + doc-updater | Tech debt analysis |
| `performance` | performance-engineer + architect-reviewer + test-runner + doc-updater | Performance profiling |
| `systems` | lead + language-pro (c/cpp/go/rust) + test-runner | Systems programming |
| `research` | 3× general-purpose | Parallel research |
| `security` | 4× reviewer (OWASP/auth/deps/secrets) | Security audit |
| `migration` | lead + 2× implementer + reviewer + test-runner + doc-updater | Codebase migration |

## How team-test-runner Works

`team-test-runner` is a **reactive** agent — it does not pick up tasks from the queue. Instead:
1. An implementer or team-lead **signals** it with specific commands to run
2. It executes the commands verbatim, captures output, and reports PASS/FAIL/PARTIAL
3. It never modifies code — only run and report

Supported ecosystems: npm/yarn/pnpm, pytest/unittest, cargo, go test, make/cmake, gradle/maven

## How team-doc-updater Works

`team-doc-updater` is also a **reactive** agent:
1. `team-lead` broadcasts a checkpoint summary describing what changed
2. It identifies which doc files need updating and makes minimal edits
3. It only touches `.md`, `.rst`, `.adoc`, `.txt` files — never source code
4. After completing, it messages team-lead with a list of changed files
