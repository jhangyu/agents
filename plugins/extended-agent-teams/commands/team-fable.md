---
description: "Fable-led hierarchical squad orchestration: the main agent commands 2-3 squads (opus squad-lead + 2-3 implementers + haiku test-runner each) through round-based milestones with handoff docs and strict chain-of-command reporting"
argument-hint: "<task description> [--squads N] [--name team-name] [--logs-dir path]"
---

# team-fable — Hierarchical Squad Orchestration

Round-based multi-squad orchestration for large tasks. The **main agent** (the agent invoking
this command — the strongest available model) is the **Orchestrator**: it owns task direction,
round decisions, and user reporting. Never spawn a separate agent to fill the orchestrator
role — only the main agent can reach the user.

## Hierarchy

```
User
 └─ Orchestrator (main agent) — direction, round decisions, reviewer dispatch, user reporting
     ├─ Squad A
     │   ├─ squadA-lead        team-lead         model: opus
     │   ├─ squadA-impl-1..N   team-implementer  model: sonnet (opus for judgment-dense work)
     │   └─ squadA-test        team-test-runner  model: haiku
     └─ Squad B (same shape)   [Squad C only if task width truly requires it]
```

Teams are **flat** — squads are a logical grouping enforced by the communication protocol
embedded in every member's task prompt, not by the harness. Name members with a squad prefix
(`squadA-lead`, `squadA-impl-1`, `squadA-test`) so routing rules are unambiguous.

## Pre-flight

1. Verify `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is set. If not, halt and instruct the user.
2. Parse arguments:
   - Positional: the overall task description
   - `--squads N`: squad count (default **2**, hard max **3**)
   - `--name <team-name>`: override default team name (`fable-team`)
   - `--logs-dir <path>`: handoff docs root (default `./docs/logs`)
3. Confirm the working directory is a git repository (worktree isolation requires it).

## Language Policy

All inter-agent communication — task descriptions, SendMessage between agents, TaskUpdate
notes, and all coordination — MUST use **English only**. The orchestrator uses the user's
preferred language (per CLAUDE.md) **only** when reporting directly to the user.

## Round Lifecycle

A **round** = one milestone. Round-end condition (all three, mechanically checked):
1. The milestone's acceptance criteria pass.
2. Every task assigned this round is `completed`.
3. Review passed (no open issues from the reviewer loop).

### Phase 1 — Decompose (orchestrator, before any spawn)

1. Define this round's milestone with **mechanically checkable acceptance criteria**
   (named tests pass, file exists with section X — never "works well").
2. Split into squad-scoped work packages with **exclusive file ownership** per squad.
3. Create **one git worktree per squad**. Squads never share a working tree.
4. Assign models: squad lead = `opus`; implementers = `sonnet` by default, `opus` for
   judgment-dense subtasks (architecture, tricky algorithms, security-sensitive paths);
   test-runner = `haiku`. Work packages touching authn/authz, secrets, crypto, or input
   validation get `opus` implementers, and that squad's Phase 3 review is a
   maximum-thoroughness security pass (probe abuse cases and trust-boundary bypasses,
   not just functional edges).
5. Output the allocation plan (squads, members, models, file ownership, worktree paths)
   before spawning.

### Phase 2 — Spawn

1. `TeamCreate` with `displayMode: "tmux"`.
2. Spawn **all** members from the orchestrator (squad leads never spawn agents).
3. `TaskCreate` per squad member. Every squad member prompt MUST embed the
   **Communication Protocol** and **Git Red Lines** blocks below, the member's file
   ownership list, worktree path, acceptance criteria, and paths to prior-round handoff
   docs (if any). Every member prompt must require that progress claims are audited
   against actual tool output before being reported — claims not backed by tool evidence
   are invalid.

#### Communication Protocol (embed verbatim in every member prompt)

- **Implementers / test-runners**: report ONLY to your own squad lead via SendMessage.
  Never message the orchestrator, other squads, or other members directly. If your task
  is blocked or needs delegation, stop and report to your squad lead.
- **Squad leads**: you are the only member who messages the orchestrator. Before reporting
  your squad's conclusion, you MUST wait for your test-runner's PASS/FAIL result — a squad
  report without test evidence is invalid. Summarize member results; do not forward raw logs.
- **Cross-squad dependency**: if your squad depends on another squad's output, WAIT for that
  squad lead's explicit SendMessage handing over the interface. Polling is forbidden — do not
  send repeated status queries; work on non-blocked items or go idle until the handoff arrives.
- **Escalation**: if the same subtask fails twice (same root cause), STOP retrying. The squad
  lead escalates to the orchestrator with the full failure trace (attempts, error output,
  current state). The orchestrator decides: re-scope, upgrade model, or ask the user.
- **Long-running processes**: never babysit. Launch detached (nohup + log file), sanity-check
  once, then report PID + log path to your squad lead and yield. Repeated polling of a
  still-running process is the signal to stop and report instead. A detached launch is a
  handoff, not a completed verification.
- A precise "blocked because X" report is a successful outcome; a guessed implementation is not.

#### Git Red Lines (embed verbatim in every member prompt)

- NEVER run `git stash`, `git reset`, `git checkout --`, or `git clean` — teammates'
  uncommitted work being present in the tree is normal.
- Commit ONLY with explicit `git add <your-own-files>`. Never `git add -A` / `git add .`.
- Never touch files outside your ownership list. Never force-push.

#### Verification Protocol (embed verbatim in reviewer prompts)

- **Adversarial framing**: assume the work is broken until evidence says otherwise. Your job
  is to actively REFUTE the squad's claim — not confirm it.
- **Self-produced evidence**: reproduce test runs and exercise the changed flow yourself.
  The implementer's or test-runner's own output is a claim, not evidence.
- **Negative-space question** (mandatory): "what existing behavior does this diff remove or
  stop handling, and who depends on it?" Read the diff for what it *doesn't* handle, not
  just what it does.
- **Verdict format**: **CONFIRMED** — every claim checked against evidence you produced
  yourself; list what you ran and observed. **REFUTED** — one concrete reproducible
  counterexample: exact inputs/state, expected vs actual, where it breaks.
- **Independence**: never fix anything — not even one line. Fixes route back to the
  originating squad lead. Independence is the reviewer's entire value.

### Phase 3 — Execute & Review

1. The orchestrator receives squad-lead summaries only; it does not micro-manage members.
2. When a squad reports its work package complete (with test-runner PASS evidence), the
   orchestrator dispatches an **opus reviewer** (`team-reviewer`, one per squad under
   review) scoped to that squad's diff. The reviewer works **read-only in the squad's
   existing worktree** — no new worktree, no file ownership list. Its task prompt embeds
   the **Verification Protocol** and **Git Red Lines** blocks verbatim (not the
   Communication Protocol). The reviewer reports its verdict ONLY to the orchestrator.
3. Reviewer issues go back to the originating squad lead for fixes, then re-review.
   **Maximum 2 review→fix cycles per squad per round.** Still failing after 2 cycles →
   stop, report the failure trace to the user, and wait for direction.
4. Spot-check: the orchestrator personally verifies 1-2 key claims per squad against
   evidence (`git show <hash> --stat`, grep for a content marker) — task status is not
   proof the work is in the tree.

### Phase 4 — Close the Round

1. Verify the round-end condition (all three items above).
2. Each squad lead confirms next-round direction with the orchestrator, THEN writes a
   handoff doc: `{logs-dir}/{YYYY-MM-DD}/round-{N}-{squad}-handoff.md` containing:
   - Completed work + evidence (commit hashes, passing test names)
   - Known limitations (explicitly listed — hiding them voids the round)
   - Interface contracts other squads or the next round depend on
   - Next-round notes for the incoming squad lead
3. The orchestrator merges squad worktrees into the integration branch (or designates
   exactly one squad lead to do it), verifying content markers before and after merge.
4. Report the round's results to the user: what each squad shipped, review outcome,
   open questions. **If any ambiguity requires a user decision, stop here and wait.**
5. Otherwise, shut down ALL round members following the team-shutdown flow (members →
   test-runners → squad leads, then `TeamDelete`), and verify no residue remains.
6. Next round: spawn fresh squads (fresh context by design). Their prompts MUST include
   the handoff doc paths from step 2 — that is how knowledge crosses the round boundary.

## Round Report Format (to the user)

```
Round {N} — {milestone}
  Squad A: {shipped} | tests: {PASS/FAIL} | review: {clean / N issues fixed / escalated}
  Squad B: ...
  Merged: {branch @ hash}
  Handoffs: {paths}
  Open questions: {none | list — awaiting your decision}
Next round plan: {milestone or "task complete"}
```

## Hard Rules

- Default 2 squads; open a third only when the task genuinely has three independent streams.
- Squad leads and members NEVER spawn subagents, teams, or workflows. Needing delegation
  means: stop and report up the chain.
- The final task in the task list is closed only by the user, never by an agent.
- Review loops, retries, and rounds are bounded (2 cycles / 2 attempts); when a bound is
  hit, escalate — never silently keep burning.
