# AGENTS.md — Project Xeno Global Agent Rules

This document is binding for every AI agent working in this repository, regardless of
model, client, or session. It overrides default agent behavior. If a request conflicts
with this document, stop and ask the human project owner for a decision.

**Status:** exists; this revision awaits project-owner approval.

Companion document: `TASK_OWNERSHIP.md` (file ownership, handoffs, task format).

---

## 1. Project identity

Project Xeno is a multiplayer arena shooter with modular combat bodies, built
**exclusively for Roblox**.

| Item | Value |
| --- | --- |
| Engine | Roblox |
| Editor | Roblox Studio |
| Language | Luau (strict typing preferred where practical) |
| Target devices | PC, Android, iOS |
| Repository | Single Git repository; all documented paths are repository-relative |
| Default branch | `master` |

### 1.1 Repository paths are portable

These rules must work on any machine, editor, or agent runtime — not just the owner's
current computer.

- **Use repository-relative paths in all procedures, documents, scripts, and configs**
  (for example `src/server/`, `docs/tasks/C-001.md`, `default.project.json`).
- **Never hard-code an absolute checkout path** in any committed file. Absolute paths break
  on other machines and leak local environment details.
- Document paths only in repository-relative form. A machine-specific checkout location is an
  environment detail that must not appear in committed documentation; each agent resolves it
  at runtime instead of recording it.
- Before acting on files, resolve the root yourself with `git rev-parse --show-toplevel`
  and work from the returned directory. Do not assume the working directory is the root.
- When reporting a change, give the repository-relative path.

### Non-negotiable platform rules

- **No Unity, Unreal Engine, C#, or any non-Roblox runtime.** Never add engine, package,
  or build tooling for another engine.
- **All gameplay code is Luau** running in Roblox (server scripts, client scripts,
  ModuleScripts).
- Anything that cannot run inside Roblox must not become a dependency of the game.
- Do not introduce a web backend, external database, or third-party service without
  explicit written approval from the project owner.

## 2. Current confirmed state of the repository

Agents must verify rather than assume. As of Task 001:

- The repository contains `README.md`, `.gitignore`, and this documentation set.
- There is **no** source tree, no Rojo project file, no place file, and no gameplay code.
- The Roblox/Luau toolchain (Rojo, Aftman/Rokit, Selene, StyLua, Lune) is **not installed**
  and is **not** on PATH.
- Roblox Studio is installed on the development machine.
- No Git remote is configured. Local commits are sufficient for now; a private remote may
  be added later.
- Architectural decisions are **provisional** until they are written in an approved
  document.

If reality differs from the above, record the difference in your task report instead of
silently adapting.

## 3. Agent roles and authority

| Role | Owns | May write code? | May edit docs? |
| --- | --- | --- | --- |
| **OpenCode** (architect/planner) | Architecture, specifications, risk analysis, task definitions, review; documentation paths assigned in `TASK_OWNERSHIP.md` §2.1 | **No**, unless the project owner explicitly authorizes a specific implementation task | Yes, only within its assigned paths |
| **Cline** (implementer) | Implementation of approved tasks only: `src/**`, Rojo project files, `assets/**` | Yes, only inside the allowed files of an approved task | **No** — never edits project documentation |

The authoritative path-level ownership matrix, including the full list of project
documents, is `TASK_OWNERSHIP.md` §2. This section only summarizes roles; if the two
documents differ, `TASK_OWNERSHIP.md` §2 governs the paths and `AGENTS.md` governs the
rules of conduct.

Additional rules:

- No agent may approve its own work. OpenCode defines and reviews; Cline implements and
  reports. The project owner is the final authority.
- An agent may not expand its own scope. If a task requires files outside its allowed
  list, it stops and requests an amendment.
- Agents must never work on the same files at the same time. See `TASK_OWNERSHIP.md`.

## 4. Hard prohibitions

Agents must not, without explicit written approval from the project owner:

1. Install, upgrade, or remove tools, SDKs, plugins, or packages.
2. Change project configuration, CI, or repository settings.
3. Add third-party libraries or Roblox frameworks (Knit, Matter, Flamework, Wally
   packages, etc.).
4. Rewrite, delete, or "tidy" files outside their owned paths.
5. Force-push, rewrite history, amend published commits, or delete branches.
6. Commit secrets, API keys, `.env` files, or personal data.
7. Disable, bypass, or weaken server-side validation to make a test pass.
8. Report unverified behavior as working.
9. Invent final game-design rules. Unresolved questions must be recorded as open
   questions, not silently decided.
10. Change the target platform, engine, or language.

## 5. Prototype constraints (currently binding)

These constraints come from the project owner and apply until explicitly revised:

- **Keep the standard Roblox Humanoid rig** for the initial prototype. Do not replace it
  with a custom character controller yet.
- **Never physically delete or detach character parts** in the first prototype. Destroyed
  parts are **disabled**: their ability is gated off and their state/visuals change.
  Removing a limb from a Humanoid rig breaks Roblox's own death and replication rules.
- **Prioritize the smallest playable multiplayer prototype.** Deferred until basic arena
  combat works:
  - rolling-ball form
  - part collection / acquisition loop
  - persistence (DataStore, inventories, unlocks)
  - advanced progression, shops, economy, monetization
- **All architectural choices are provisional** until documented and approved.
- Prefer the smallest change that satisfies the acceptance criteria of the current task.

## 6. Roblox and Luau engineering rules

### 6.1 Server authority

- The server is the only authority for: health, part HP, ability cooldowns, ammo/energy,
  hit validation, eliminations, points, scores, loadout validity, respawn and match state.
- Clients send **intents** (e.g. "I want to fire in this direction"), never outcomes
  (e.g. "I dealt 40 damage").
- Never trust client-supplied values for damage, position teleports, cooldown timers, or
  currency.
- Roblox character physics is client-controlled. Do not pretend otherwise; add server-side
  sanity checks and document residual risk.

### 6.2 Remotes

- All `RemoteEvent`/`RemoteFunction` instances are created **by the server** (in
  `ReplicatedStorage`) and referenced through one shared definition module. Clients must
  never create remotes the server listens to.
- Every remote handler validates: argument count, argument types, value ranges, caller
  identity, cooldown, and current state (alive, correct zone, correct phase).
- Wrap remote handlers so malformed input cannot error out the server. Rate-limit
  high-frequency remotes.
- Only introduce a remote when a documented need exists; keep the remote surface small.

### 6.3 Code organization

- No gameplay logic in `Script`/`LocalScript` bodies beyond bootstrap. Logic lives in
  ModuleScripts under the agreed service/controller split.
- Balance numbers, cooldowns, and tuning values live in one shared configuration module —
  never hard-coded in multiple places.
- Shared modules must not depend on server-only or client-only APIs unless clearly marked.
- Prefer small, named functions with Luau type annotations at module boundaries.
- No per-frame polling when an event or attribute-change signal is available.
- Do not add abstractions "for later" without an approved requirement.

### 6.4 Naming

- Files and services: `PascalCase` for ModuleScripts (`CombatService`), `*.server.luau`
  and `*.client.luau` for entry points.
- Never rename Roblox rig parts (`HumanoidRootPart`, `Head`, `Torso`/`UpperTorso`,
  `RightArm`, `LeftArm`, `RightLeg`, `LeftLeg`) or other engine-significant names.
- Use explicit, domain-accurate names (`PartState`, `AbilityId`), not `Thing`, `Data`,
  `Manager` used loosely.

### 6.5 Client and mobile

- Every interactive element must be usable with touch, keyboard/mouse, and be verifiable
  in Roblox Studio's Device Emulator.
- Respect mobile screen space and safe areas; do not hide critical state behind hover.
- Keep per-frame client work and effect counts budgeted; mobile is a first-class target,
  not a later port.

## 7. Documentation is authoritative

- Approved documents in `docs/` and the repository root define the intended design.
- If code and documentation disagree, do not "fix" the documentation to match the code.
  Stop, report the conflict, and request a decision.
- If a task is ambiguous or contradicts a document, stop and ask. Guessing is a defect.
- Open design questions are recorded in a dedicated questions section of the relevant
  document and are never resolved by an agent acting alone.

### 7.1 Planned document set

The project documentation set comprises the existing repository documents plus the planned
design documents listed below (ten project documents plus `README.md`). Path ownership is
defined in `TASK_OWNERSHIP.md` §2.1, which is the authoritative list; this table records the
intended purpose and current state.

| Document | Purpose | State |
| --- | --- | --- |
| `AGENTS.md` | Global rules for all agents (this document) | Exists — this revision awaits owner approval |
| `TASK_OWNERSHIP.md` | File ownership, handoffs, task lifecycle, task format | Exists — this revision awaits owner approval |
| `README.md` | Public project overview | Exists — owner-visible summary |
| `ROADMAP.md` | Staged milestones, prerequisites, exit criteria | Planned — not yet created |
| `GAME_DESIGN.md` | Game rules, open design questions, out-of-scope list | Planned — not yet created |
| `ARCHITECTURE.md` | Roblox services, module boundaries, code organization | Planned — not yet created |
| `NETWORKING.md` | Server authority, remote contract, validation, exploit prevention | Planned — not yet created |
| `COMBAT_AND_DAMAGE.md` | Body-part HP, damage, disablement, combat flow | Planned — not yet created |
| `REASSEMBLY.md` | Defeat, hub, assembly, re-entry | Planned — not yet created |
| `UI_AND_CONTROLS.md` | HUD, camera, PC/mobile/gamepad controls, accessibility | Planned — not yet created |
| `TESTING.md` | Test levels, evidence requirements, budgets, regression list | Planned — not yet created |

Document creation requires owner approval (Gate G1) and follows the order agreed with the
owner. **No agent may create, rename, or delete a document outside its assigned paths**, and
no agent may create one of the planned documents early to "help".

## 8. Beginner-friendly communication

The project owner is learning Roblox development. All agent output must therefore:

- Explain unfamiliar Roblox/Luau concepts in plain language at first use (e.g. what a
  RemoteEvent, `ServerScriptService`, or a ModuleScript is, and why it is used).
- Give concrete, ordered steps — including where in Roblox Studio to click — for any
  manual action the owner must take.
- State assumptions explicitly and separately from confirmed facts.
- Avoid unexplained jargon, and avoid large unexplained code dumps.
- Never say "just" or "simply" about a step that involves setup, Studio, or configuration.
- Keep instructions copy-pasteable where possible (exact commands, exact paths).

## 9. Testing and evidence

No feature is "done" until it has been tested and the evidence is recorded.

### 9.1 Test levels

| Level | Meaning | Required when |
| --- | --- | --- |
| L0 Static | File parses, project builds, no obvious syntax/type errors | Every task |
| L1 Studio Play (solo) | Manual run in Studio Play mode, Output checked | Every task that changes runtime behavior |
| L2 Local server (2+ clients) | Studio "Test → Clients and Servers" with 2+ players | Any networking, combat, scoring, or respawn change |
| L3 Device emulation | Studio Device Emulator for phone/tablet | Any UI or control change |
| L4 Published playtest | Private/published server on real devices | Milestones, before release claims |

### 9.2 Evidence rules

- Report exactly which levels were run and which were **not** run.
- Paste real command output, error text, or a described observation. Never invent logs.
- If a test cannot be run (no tool installed, no second device, no account), say so
  explicitly and mark the task partially verified.
- A passing automated build is **not** proof that gameplay works.
- Do not mark acceptance criteria as met if the corresponding test was skipped.
- **A failed required test is a failure, not a partial success.** It blocks task acceptance
  and cannot be waived at acceptance time. See §9.4 and §12.

### 9.3 What cannot be claimed

- Claims of multiplayer correctness without an L2 run.
- Claims of mobile support without an L3 run.
- Claims of exploit resistance without a documented adversarial test.
- Claims of performance without measured numbers.

### 9.4 Failed tests, WIP work, and acceptance

Three states must never be conflated: *implemented*, *committed*, and *accepted*.

- **A failed required test always blocks acceptance.** No task may move to `accepted` while
  any test the task marks as required is failing, regardless of how small the failure looks
  or how much time was invested.
- The correct responses to a failed required test are:
  1. fix the cause and re-run the test, or
  2. report `blocked` and ask the owner to amend the requirement through Gate G5 (task
     amendment). Amending the task specification changes the requirement; it does not
     retroactively pass the old test.
- **A WIP commit may preserve incomplete work, but only under these conditions:**
  - it is labeled with the `wip:` commit type,
  - the commit body states which required test failed or could not be run, the observed error
    or symptom (or why it could not be run), and which test levels were actually run,
  - the task file's status is `in-progress` (never `review` or `accepted`) with the same
    failure recorded in its status log,
  - it carries **no milestone tag**,
  - it does not claim completion, and the completion report says "partial verification".
- A WIP commit is a checkpoint for preservation, not evidence of a finished feature. It
  must not be presented to the reviewer as ready for acceptance.

## 10. Git rules

- One task = one focused commit (or a small, clearly labeled series). Commit message
  format: `type(scope): summary (TASK-ID)`, e.g.
  `feat(combat): add server-validated hitscan (C-004)`.
- Allowed types: `feat`, `fix`, `docs`, `chore`, `test`, `refactor`, `perf`, `wip`.
- `wip` is reserved for preserving incomplete work whose required test is failing, and is
  only valid under §9.4. A `wip` commit is not a completion checkpoint and never receives a
  milestone tag.
- Do not commit generated build output, Studio autosaves, or local settings unless a
  document explicitly allows it.
- Never commit `.rbxl`/`.rbxlx` artifacts without an approved asset policy.
- Milestone checkpoints use annotated tags: `m0-scaffold`, `m1-playable`, etc.
- A normal task commit is allowed only after **every required test has passed**. Partial
  verification — a failing or skipped required test — may be preserved only as a `wip`
  commit under §9.4. A normal commit must never carry a failing required test, and partial
  verification never satisfies the task checkpoint.
- No remote is required for now. Do not attempt to add or change remotes without approval.

## 11. Approval gates

Work stops and waits for the project owner at each gate:

| Gate | Trigger | Who may proceed |
| --- | --- | --- |
| G1 — Document approval | A design or architecture document is drafted | Owner approves; then agents rely on it |
| G2 — Tooling | Installing or upgrading any tool (Rojo, Aftman/Rokit, Selene, StyLua, Studio plugin) | Owner installs or explicitly authorizes |
| G3 — Architecture change | New service, new remote, new framework, changed character model, changed replication strategy | Owner approves first; document updated first |
| G4 — Task authorization | A Cline task may begin | Owner approves the task spec; OpenCode has reviewed it |
| G5 — Task amendment | The task needs files outside its allowed list, or a requirement, acceptance criterion, or test in the task spec must change | Owner approves the amended task spec; the affected work is then re-tested against it |
| G6 — Milestone acceptance | Milestone exit criteria claimed met | Owner reviews evidence and accepts the milestone; **no required test may be failing** (§9.4) |
| G7 — Publication | Publishing a place, enabling DataStore, or exposing the game publicly | Owner performs or authorizes |
| G8 — Task acceptance | A single task is claimed complete | Owner accepts or rejects that task; **no required test may be failing** (§9.4). G8 never covers a milestone |

Silence is never approval. If an approval is missing, stop and ask.

## 12. Definition of done

A task is complete only when all of the following are true:

1. Changes are confined to the task's allowed files.
2. Acceptance criteria are met and each one is explicitly checked off.
3. The stated tests were run, with real evidence, and skipped tests are disclosed.
4. **Every required test passed.** A task with a failing required test is **never accepted**.
   Its permitted interim outcomes are: fix the failure and re-run the test; `blocked` with a
   documented reason; or `in-progress` with an allowed `wip` commit (§9.4). `rejected` is a
   separate owner decision (Gate G8) and is not successful completion.
5. No new errors or warnings appear in Studio Output that were not already documented.
6. Documentation impact is reported (which document is now out of date, if any) — but not
   edited by the implementer.
7. The commit exists with the required message format and the task's checkpoint tag, if any.
   A `wip` commit does not satisfy this item.
8. A short report is written: what changed, why, how it was tested, what remains open.

## 13. Amending this document

- Only OpenCode may propose changes; only the project owner may approve them.
- Amendments are dated and summarized at the bottom of this file.
- Conflicting rules are resolved by the project owner, not by agent preference.

---

## Change log

| Date | Change | Approved by |
| --- | --- | --- |
| 2026-10-27 | Initial version created under Task 001 | Awaiting project-owner approval |
| 2026-10-27 | Amendment 1: portable repository-relative paths (§1.1); reconciled planned document set with ownership matrix (§7.1); added failed-test/WIP/acceptance rules (§9.2, §9.4, §10, §11 G6/G8, §12) | Awaiting project-owner approval |
| 2026-10-27 | Amendment 2: normal commits require all required tests to pass, with partial verification limited to `wip` (§10); absolute checkout path removed (§1.1); G5 broadened to task amendment and G8 added as the task-acceptance gate (§11); §12 aligned with §9.4 | Awaiting project-owner approval |
