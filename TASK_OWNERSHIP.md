# TASK_OWNERSHIP.md — File Ownership, Handoffs, and Task Lifecycle

This document defines **who may change what**, **how work is handed off**, and the
**required format for every implementation task** in Project Xeno.

It is binding for all AI agents and complements `AGENTS.md`. If the two documents
conflict, `AGENTS.md` governs the rules of conduct and the conflict must be reported to the
project owner.

**Status:** exists; approved at Gate G1 by the project owner on 2026-10-27, together with
`AGENTS.md`, `ROADMAP.md`, `TESTING.md`, and `ARCHITECTURE.md`. `README.md` is informational and
outside the approval.

All paths in this document are **repository-relative** and must remain portable across
machines. Resolve the repository root with `git rev-parse --show-toplevel` before acting,
and never record a machine-specific checkout path in a committed file.

---

## 1. Core principle

> **One owner per file. One task in flight. No shared writes.**

Two agents must never edit the same file at the same time, and preferably never at all.
Ownership is by path, not by intent. "I only changed one line" is still a violation.

## 2. Exclusive ownership matrix

Paths are repository-relative. This section is the **authoritative list of project
documents and owned paths**; other documents describe purposes, not ownership.

### 2.1 OpenCode (architecture and planning)

| Path | Contents | Notes |
| --- | --- | --- |
| `AGENTS.md` | Global agent rules | Exists. Amendment requires owner approval; approved at Gate G1, 2026-10-27 |
| `TASK_OWNERSHIP.md` | This document | Exists. Approved at Gate G1, 2026-10-27 |
| `README.md` | Public project overview | Exists. Informational; outside the G1 package; not edited during implementation tasks |
| `ROADMAP.md` | Milestones, prerequisites, exit criteria | Exists; prepared in Task 002. Approved at Gate G1, 2026-10-27 |
| `GAME_DESIGN.md` | Rules and open questions | Planned — not yet created. Creation requires explicit owner authorization |
| `ARCHITECTURE.md` | Services, modules, code layout | Exists; prepared in Task 004. Approved at Gate G1, 2026-10-27 |
| `NETWORKING.md` | Remote contract and validation | Planned — not yet created. Creation requires explicit owner authorization |
| `COMBAT_AND_DAMAGE.md` | Part HP, damage, disablement | Planned — not yet created. Creation requires explicit owner authorization |
| `REASSEMBLY.md` | Defeat, hub, assembly, re-entry | Planned — not yet created. Creation requires explicit owner authorization |
| `UI_AND_CONTROLS.md` | HUD, camera, PC/mobile/gamepad controls, accessibility | Planned — not yet created. Creation requires explicit owner authorization |
| `TESTING.md` | Test strategy, evidence rules, regression list, and where numeric budgets are to be defined | Exists; prepared in Task 003. Approved at Gate G1, 2026-10-27 |
| `docs/**` | Design records, task specs, reviews, and the proposed `docs/setup.md` beginner guide | Includes `docs/tasks/`. `docs/setup.md` is not created yet; it is a separate approved task after M0 tooling is decided |

The ten project documents above (plus `README.md`) are the complete project document set.
Creating a document that is not in this table, or creating a document before its creation is
explicitly authorized, is a scope violation. Gate G1 approves a document **after** drafting;
it is never permission to create one. Adding or renaming a document updates this table in the
same commit.

OpenCode does **not** write gameplay code. Any exception requires the project owner to
authorize a specific, named implementation task.

### 2.2 Cline (implementation)

| Path | Contents | Notes |
| --- | --- | --- |
| `src/**` | All Luau source | Server, client, shared |
| Project mapping file (tool decided at Gate G2; Rojo example: `default.project.json`) | Maps disk folders into Roblox containers | Only when a task allows it |
| `assets/**` | Maps, models, UI art sources | Only when a task allows it |
| `.gitignore` | Ignore rules | Only when a task allows it, and only for Roblox/binary entries |
| Toolchain pin files (names depend on the approved tool; Aftman example: `aftman.toml`) | Tool versions | Only after tooling is approved (Gate G2) |

Cline never edits project documentation except for one narrow location: the **status log and
evidence section of its own assigned `docs/tasks/C-###.md`**. It may append dated state
changes and test evidence there. It may not change requirements, acceptance criteria, scope,
allowed files, test procedures, or any other section of that file, and it never edits any
other document, including `AGENTS.md`, `README.md`, `TASK_OWNERSHIP.md`, `ROADMAP.md`,
`TESTING.md`, or `ARCHITECTURE.md`. If a document is wrong, report it.

### 2.3 Shared or contested paths

Some paths are owned by neither agent by default:

| Path | Rule |
| --- | --- |
| `places/`, `build/` | Build output. Never committed unless the owner approves an asset policy. |
| `.vscode/`, `.git/config`, `.git/hooks` | Owner-only. Not agent-editable. |
| Roblox Studio place contents | Authored manually by the owner unless a task explicitly covers a path mapped by the approved build/sync tool. |

If a needed file is not listed in a task's allowed files, the agent stops and requests an
amendment. Guessing is a defect.

## 3. Concurrency rules

1. **Only one implementation task may be in progress at a time** until the owner says
   otherwise.
2. OpenCode may draft documents while Cline implements, but **only in disjoint paths**.
3. Before starting a task, the agent confirms the working tree is clean or that
   uncommitted changes belong to its own in-flight task.
4. If an unexpected uncommitted change appears in files the agent does not own, the agent
   **stops** and reports it. It must not revert, stash, or overwrite it.
5. No agent runs destructive Git operations (reset --hard, clean -fd, checkout -- .) while
   another agent's work may be uncommitted.
6. Long-running work is announced in the task file (see §4) so state is visible in the
   repository rather than only in chat.

## 4. Task lifecycle

The terms stage/milestone, approved, blocked, provisional, and the identifier namespaces are
defined in `AGENTS.md` §7.2. `approved` always means explicit owner approval at the named
gate; an agent cannot approve its own task or document.

States:

```
proposed --> approved --> in-progress --> review --> accepted
                 |              |             |
                 |              v             v
                 |           blocked       rejected
                 |              |
                 |              +--> in-progress   (block resolved; owner permits resume)
                 +--> blocked
                 +--> rejected

review --> in-progress   (review findings require changes)
```

Transition rules, in addition to the table below:

- `proposed → approved`: owner authorizes the task (Gate G4).
- `approved → in-progress`: Cline begins work.
- `in-progress → blocked`: either agent, as soon as a blocker is identified and recorded.
- `blocked → in-progress`: only after the blocker is resolved **and** the task owner
  explicitly permits resuming; both facts are recorded in the status log.
- `in-progress → review`: every required test passes, evidence is attached, and the final
  commit is not a `wip` commit.
- `review → in-progress`: review findings require changes.
- `review → accepted`: owner accepts the task (Gate G8), with no failing required test.
- `review → rejected`: owner rejects the task (Gate G8).
- `proposed → rejected` or `approved → rejected`: owner closes the task before or instead of
  implementation; the reason is recorded.
- After `rejected` the task is **closed**. Further work needs a new approved task (new ID)
  or an explicit owner reopening recorded in the status log. `rejected` is never a successful
  completion, and `blocked` is never an acceptance state.

| State | Who acts | What must be true |
| --- | --- | --- |
| **proposed** | OpenCode | Task file written in `docs/tasks/<ID>.md` using the format in §6 |
| **approved** | Project owner | Owner states approval; OpenCode has reviewed the spec (Gate G4) |
| **in-progress** | Cline | Spec is approved; allowed files listed; prerequisites met; status line updated in the task file |
| **review** | OpenCode | Implementation finished, every required test passing, evidence attached, commit exists and is not a `wip` commit |
| **accepted** | Project owner | Acceptance criteria verified, evidence reviewed, no required test failing (Gate G8) |
| **blocked** | Either | Reason recorded in the task file, plus what decision or access is needed. Not an acceptance state; resuming requires a resolved blocker and owner permission |
| **rejected** | Project owner | Reason recorded; work is not merged or is reverted at owner's instruction; the task is closed and requires a new approved task or explicit owner reopening |

Rules:

- An agent may not move its own task to **accepted**.
- A task in **review** must not be modified further by Cline except to fix review findings,
  which returns it to **in-progress**.
- Every state change is recorded in the task file's status log with a date.
- **A task with any failing required test cannot enter `review` or `accepted`.** It stays
  `in-progress` (optionally with a `wip` commit, see §7) or becomes `blocked`.
- `accepted` requires all of: every required test passing with evidence, no `wip` commit as
  the task's final checkpoint, and owner approval at Gate G8. Implemented is not committed;
  committed is not accepted.
- A single task is accepted or rejected at **Gate G8 (task acceptance)**. **Gate G6
  (milestone acceptance)** is used only when an entire milestone's exit criteria are
  claimed met; the two gates are never interchangeable.
- If a required test turns out to be wrong, the owner amends the task specification
  (Gate G5, task amendment). Only then is the requirement changed and the task re-tested
  against it.

## 5. Handoff protocol

**OpenCode → Cline (assignment)**

1. OpenCode writes `docs/tasks/<ID>.md`.
2. OpenCode states, in plain text to the owner: what will change, which files, why, and
   what is out of scope.
3. Owner approves (Gate G4).
4. Cline reads the task file, restates scope in its first message, and begins.

**Cline → OpenCode (completion)**

1. Cline updates only the task file's status log and evidence section; it does not change the
   requirements, acceptance criteria, or any other section (`§2.2`).
2. Cline commits only when the task's required tests pass. If a required test fails, Cline
   either fixes it or records a `wip` commit under §7, and the task stays `in-progress`.
3. Cline reports: files changed, commands run, test levels reached, what was not tested,
   which required tests failed (if any), open questions, and any documentation impact.
4. OpenCode reviews against the acceptance criteria and writes a short verdict into the
   task file. A failing required test stops the review.
5. Owner accepts or rejects (Gate G8, task acceptance).

**Escalation** — stop and ask the owner when:

- an allowed file must be added,
- documentation and code contradict each other,
- a design question blocks progress,
- an acceptance criterion cannot be tested,
- a tool or install is required.

## 6. Required format for future Cline tasks

Every implementation task is a Markdown file at `docs/tasks/C-###.md` containing these
sections. A task missing any mandatory section is not approved.

```markdown
# C-### — <short imperative title>

- Milestone: M#
- Status: proposed | approved | in-progress | review | accepted | blocked | rejected
- Owner approval: <date or "pending">
- Estimated size: small (<= ~150 changed lines) | medium | large (split before starting)

## Objective
One or two sentences. What becomes true for the player or the codebase.

## Context and references
Approved documents and sections this task implements. Link exact document names.

## In scope
Bullet list of concrete deliverables.

## Out of scope
Bullet list of tempting adjacent work that must NOT be done in this task.

## Allowed files
Exact paths or globs Cline may create or modify.

## Forbidden files
Paths Cline must not touch (at minimum: all documentation, including docs/**, AGENTS.md,
README.md, TASK_OWNERSHIP.md, and any config not listed above). The only permitted exception
is the status log and evidence section of this task file itself (TASK_OWNERSHIP.md §2.2).

## Prerequisites
Tools installed, prior tasks accepted, decisions made. State blockers explicitly.

## Gameplay and technical requirements
Numbered, testable statements. Include authoritative source of truth for any value
(e.g. "damage values come from Shared/Config/Combat.luau").

## Acceptance criteria
- [ ] AC1 ...
- [ ] AC2 ...
(each one objectively verifiable)

## Test procedure
Ordered manual/automated steps, with test level labels (L0/L1/L2/L3/L4).
Mark each step Required or Optional. A failing Required step blocks acceptance.
State the exact Studio mode, number of clients, and expected observation.

## Evidence required
What Cline must paste or describe in the completion report for each test level.

## Git checkpoint
- Commit message: `type(scope): summary (C-###)`
- Tag (if milestone): `m#-<slug>`
- Rollback: how to undo safely (which commit/files)

## Risks and notes
Known edge cases, performance concerns, and anything the reviewer must scrutinize.

## Status log
| Date | Actor | State | Note |
| --- | --- | --- | --- |
```

## 7. Git checkpoints

- **One commit per task** unless the task file explicitly allows a series.
- Message format: `type(scope): summary (C-###)`; body explains the approach and lists
  test levels actually run. `wip` is a permitted type only under the conditions below.
- Milestone completion tags are annotated and named `m#-<slug>` (e.g. `m0-scaffold`).
- Checkpoint required at: end of every accepted task, end of every milestone, and before
  any risky refactor.
- If a required test fails, the task does **not** commit as complete and can never be
  accepted in that state. Fix it and re-run, or preserve the work as a `wip` commit.
- A `wip` commit is allowed only when all of the following are true:
  - the required test failure or skip, the observed symptom (or why it could not be run),
    and the test levels actually run are written in the commit body,
  - the same details are recorded in the task file's status log, with status `in-progress`,
  - no milestone tag is applied,
  - the completion report states "partial verification" and does not claim success.
- A later passing commit (not `wip`) is required before the task can reach `review`. The
  `wip` commit is a preservation checkpoint only; it never counts as the task checkpoint.
- If a required test fails in a way that cannot be fixed within the task, mark the task
  `blocked`, record what is needed, and ask the owner. Do not weaken the test.
- Rollback is done with a **revert commit**, not history rewriting. History rewriting
  requires the project owner.
- No remote is required for now. Adding one is an owner action.

## 8. Review checklist (OpenCode)

- [ ] Changes are confined to the allowed files (verify with `git diff --stat`).
- [ ] Acceptance criteria are individually verifiable and were actually demonstrated.
- [ ] Evidence is real: command output or described observation, not a summary claim.
- [ ] **No required test is failing.** Any failure blocks review acceptance.
- [ ] Skipped test levels are disclosed and justified.
- [ ] No `wip` commit is the task's final checkpoint, and no `wip` commit claims completion.
- [ ] No secrets, no build artifacts, no generated files committed.
- [ ] Server authority preserved: no client-supplied outcomes.
- [ ] Remote surface unchanged unless the task allowed it.
- [ ] No Roblox rig part renamed, deleted, or detached (prototype constraint).
- [ ] Balance values live in configuration, not scattered literals.
- [ ] Naming and structure match `ARCHITECTURE.md` once it is approved.
- [ ] Documentation edits by Cline are confined to the permitted status-log/evidence section
      of its own task file.
- [ ] Commit message and tag match the required format.
- [ ] Documentation impact reported.

## 9. Conflict and recovery

- **Two agents touched the same file:** stop all work; owner decides which version
  stands; the losing change is re-applied through a new task.
- **Unexplained changes in the tree:** do not revert; report with `git status` output.
- **Task blocked by a design question:** create or update the open-questions entry in the
  relevant document; do not invent rules.
- **Required test failing near the end of a task:** do not lower the bar. Either fix and
  re-test, or record a documented `wip` commit and keep the task `in-progress` / `blocked`.
  Acceptance is only possible after a passing run.
- **Model or session changed mid-task:** the task file is the source of truth, not the
  chat history. Resume by reading the task file and the current diff.

## 10. Emergency stop

Any agent must stop immediately and report if it observes:

- gameplay code being written by OpenCode without authorization,
- documentation edited by Cline outside the permitted status-log/evidence section of its own task file,
- a Hard Prohibition from `AGENTS.md` §4 being violated,
- an attempt to weaken validation to make a test pass,
- a claim of testing that did not occur.

## 11. Amendments

Only the project owner approves changes to this document. Amendments are dated in the
change log. The fact and scope of a G1 approval are recorded in the decision log in
`AGENTS.md` §14. That log is the single authoritative record for owner decisions that the other
documents treat as decided, including the stage-blocking items tracked in `ROADMAP.md` §6 and
open questions in `TESTING.md` §13: the owning document's table row and change log follow the
log, never the other way round.

---

## Change log

| Date | Change | Approved by |
| --- | --- | --- |
| 2026-10-27 | Initial version created under Task 001 | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 1: repository-relative portable paths; ownership matrix reconciled with the full document set including `ROADMAP.md`, `TESTING.md`, `UI_AND_CONTROLS.md`; explicit failed-test / `wip` / acceptance rules (§2.1, §4, §5, §6, §7, §8) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 2: absolute checkout path removed; document statuses unified; lifecycle diagram and transitions completed (`in-progress → blocked`, `blocked → in-progress`, `review → rejected`, closed-task rules); G8 added for task acceptance and G6 restricted to milestones (§4, §5, §2.1) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 3 (Task 006): accurate document registry and G1 wording (§2.1); narrow Cline status-log/evidence exception reconciled across §2.2, §5, §6, §8, §10; tool-neutral project mapping and pin-file names; terms pointer (§4); decision-log pointer (§11) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 4 (Task 010): §11 states that the `AGENTS.md` §14 decision log is the single authoritative record for stage-blocking owner decisions, with the owning document's table row and change log following it | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 5 (Task 015): Gate G1 owner approval recorded — the five approved documents are marked approved in the §2.1 registry and the status line, with `README.md` still informational and outside the approval. Ownership paths, handoff rules, and task format are otherwise unchanged | Approved at G1, 2026-10-27 |
