---
description: "Run a consolidated command from disabled plugins: /cmd <name> [args] — scaffolds, security scans, migrations."
argument-hint: "<command-name> [arguments...]"
---

# /cmd — Consolidated Command Dispatcher

Parse `$ARGUMENTS`: the **first token** is the command name; everything after it is the **payload**.

Look up the command name in the index below. If found, Read the referenced file (path is relative to this plugin's root directory — go up one level from this `commands/` folder) and execute its full instructions, treating the payload as that command's `$ARGUMENTS`.

If the name is missing or not in the index, print the available commands table below and stop. Do not guess or suggest alternatives.

## Command Index

| Name | Reference path | Source plugin | Summary |
|---|---|---|---|
| python-scaffold | skills/python-dev-router/references/python-scaffold.md | python-development | Scaffold a new Python project with best-practice layout |
| typescript-scaffold | skills/js-ts-dev-router/references/typescript-scaffold.md | javascript-typescript | Scaffold a new TypeScript/Node.js project |
| component-scaffold | skills/js-ts-dev-router/references/component-scaffold.md | frontend-mobile-development | Scaffold frontend UI component with tests and styles |
| feature-development | skills/backend-dev-router/references/feature-development.md | backend-development | End-to-end feature development workflow |
| full-stack-feature | skills/workflow-extras-router/references/full-stack-feature.md | full-stack-orchestration | Multi-agent full-stack feature orchestrator (db to deploy) |
| multi-platform | skills/workflow-extras-router/references/multi-platform.md | multi-platform-apps | Orchestrate feature build across web/iOS/Android/desktop |
| data-driven-feature | skills/workflow-extras-router/references/data-driven-feature.md | data-engineering | 16-step orchestrator: data-driven feature dev with A/B testing |
| tdd-red | skills/workflow-extras-router/references/tdd-red.md | tdd-workflows | Write failing tests for TDD red phase |
| tdd-green | skills/workflow-extras-router/references/tdd-green.md | tdd-workflows | Write minimal code to pass failing tests |
| tdd-refactor | skills/workflow-extras-router/references/tdd-refactor.md | tdd-workflows | Refactor code safely while keeping tests green |
| test-generate | skills/workflow-extras-router/references/test-generate.md | unit-testing | Generate comprehensive test suites for code |
| api-mock | skills/workflow-extras-router/references/api-mock.md | api-testing-observability | Generate mock API servers/responses for testing |
| error-analysis | skills/workflow-extras-router/references/error-analysis.md | error-diagnostics | Analyze error messages and stack traces |
| error-trace | skills/workflow-extras-router/references/error-trace.md | error-diagnostics | Trace error origins through call stack |
| smart-debug | skills/workflow-extras-router/references/smart-debug.md | error-diagnostics | AI-assisted debugging from error to root cause |
| debug-trace | skills/workflow-extras-router/references/debug-trace.md | distributed-debugging | Trace distributed system requests to find bugs |
| smart-fix | skills/workflow-extras-router/references/smart-fix.md | incident-response | Multi-agent issue resolution: diagnose, fix, verify |
| full-review | skills/workflow-extras-router/references/full-review.md | comprehensive-review | Multi-phase orchestrated comprehensive code review |
| tech-debt | skills/workflow-extras-router/references/tech-debt.md | codebase-cleanup | Inventory and prioritize technical debt |
| deps-audit | skills/workflow-extras-router/references/deps-audit.md | codebase-cleanup | Audit dependencies for vulnerabilities/staleness |
| performance-optimization | skills/workflow-extras-router/references/performance-optimization.md | application-performance | End-to-end app performance profiling and optimization |
| doc-generate | skills/workflow-extras-router/references/doc-generate.md | code-documentation | Auto-generate API/architecture/code/user docs from codebase |
| code-explain | skills/workflow-extras-router/references/code-explain.md | code-documentation | Explain complex code via diagrams, step-by-step breakdowns |
| monitor-setup | skills/workflow-extras-router/references/monitor-setup.md | observability-monitoring | Set up observability monitoring stack and dashboards |
| sql-migrations | skills/workflow-extras-router/references/sql-migrations.md | database-migrations | Plan and generate safe SQL schema migrations |
| migration-observability | skills/workflow-extras-router/references/migration-observability.md | database-migrations | Instrument and monitor database migrations in flight |
| config-validate | skills/workflow-extras-router/references/config-validate.md | deployment-validation | Validate deployment configs before rollout |
| workflow-automate | skills/workflow-extras-router/references/workflow-automate.md | cicd-automation | Automate CI/CD workflow pipelines |
| data-pipeline | skills/workflow-extras-router/references/data-pipeline.md | data-engineering | Design and build data engineering pipelines |
| security-sast | skills/security-router/references/security-sast.md | security-scanning | Static application security testing |
| security-hardening | skills/security-router/references/security-hardening.md | security-scanning | Defense-in-depth security hardening |
| security-dependencies | skills/security-router/references/security-dependencies.md | security-scanning | Dependency vulnerability scanning |
| xss-scan | skills/security-router/references/xss-scan.md | frontend-mobile-security | Scan frontend/mobile code for XSS vulnerabilities |
| compliance-check | skills/security-router/references/compliance-check.md | security-compliance | Check codebase against compliance frameworks (GDPR/SOC2 etc) |
| code-migrate | skills/workflow-extras-router/references/code-migrate.md | framework-migration | Framework/language migration plans and scripts |
| deps-upgrade | skills/workflow-extras-router/references/deps-upgrade.md | framework-migration | Safe incremental dependency upgrades |
| legacy-modernize | skills/workflow-extras-router/references/legacy-modernize.md | framework-migration | Strangler-fig legacy system modernization |

## Argument Mapping

The payload (all tokens after the command name) becomes the target command's `$ARGUMENTS` verbatim. Examples:

- `/cmd python-scaffold my-api --framework fastapi` reads `python-scaffold.md` and executes with `$ARGUMENTS = "my-api --framework fastapi"`
- `/cmd security-sast ./src` reads `security-sast.md` and executes with `$ARGUMENTS = "./src"`
- `/cmd` (empty) prints the command index above
