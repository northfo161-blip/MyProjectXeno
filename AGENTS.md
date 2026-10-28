# AGENTS.md — Project Xeno Global Agent Rules

This document is binding for every AI agent working in this repository, regardless of
model, client, or session. It overrides default agent behavior. If a request conflicts
with this document, stop and ask the human project owner for a decision.

**Status:** exists; approved at Gate G1 by the project owner on 2026-10-27, together with
`TASK_OWNERSHIP.md`, `ROADMAP.md`, `TESTING.md`, and `ARCHITECTURE.md`. `README.md` is
informational and not part of that approval.

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

## 2. Repository state — dated snapshot

This section is a **historical snapshot**, not a live status. Agents must verify current
state rather than assume it.

At the Task 001 snapshot (2026-10-27):

- The repository contained `README.md`, `.gitignore`, and this documentation set.
- There was **no** source tree, no project mapping file, no place file, and no gameplay code.
- The Roblox/Luau toolchain was **not installed** and was **not** on PATH. Rojo is only a
  candidate build/sync tool; no tool is selected until Gate G2 and an approved M0 task spec.
- Roblox Studio was installed on the development machine.
- No Git remote was configured. Local commits are sufficient for now; a private remote may
  be added later.
- Architectural decisions were **provisional** until written in an approved document.

At the Task 006 snapshot (2026-10-27), `ROADMAP.md`, `TESTING.md`, and `ARCHITECTURE.md` also
exist as unapproved drafts; see §7.1. Any statement above may be stale. If reality differs,
record the difference in your task report instead of silently adapting.

## 3. Agent roles and authority

| Role | Owns | May write code? | May edit docs? |
| --- | --- | --- | --- |
| **OpenCode** (architect/planner) | Architecture, specifications, risk analysis, task definitions, review; documentation paths assigned in `TASK_OWNERSHIP.md` §2.1 | **No**, unless the project owner explicitly authorizes a specific implementation task | Yes, only within its assigned paths |
| **Cline** (implementer) | Implementation of approved tasks only: `src/**`, project mapping files, `assets/**` | Yes, only inside the allowed files of an approved task | **No**, with one narrow exception: it may update only the status log and evidence section of its own assigned `docs/tasks/C-###.md` (see `TASK_OWNERSHIP.md` §2.2). It never edits any other documentation |

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

### 7.1 Project document set

The project documentation set comprises the documents below (ten project documents plus
`README.md`). Path ownership is defined in `TASK_OWNERSHIP.md` §2.1, which is the authoritative
list; this table records the intended purpose and current state.

| Document | Purpose | State |
| --- | --- | --- |
| `AGENTS.md` | Global rules for all agents (this document) | Exists — approved at Gate G1, 2026-10-27 |
| `TASK_OWNERSHIP.md` | File ownership, handoffs, task lifecycle, task format | Exists — approved at Gate G1, 2026-10-27 |
| `README.md` | Public project overview | Exists — informational; not part of the G1 package |
| `ROADMAP.md` | Staged milestones, prerequisites, exit criteria | Exists; prepared in Task 002 — approved at Gate G1, 2026-10-27 |
| `GAME_DESIGN.md` | Game rules, open design questions, out-of-scope list | Planned — not yet created |
| `ARCHITECTURE.md` | Roblox services, module boundaries, code organization | Exists; prepared in Task 004 — approved at Gate G1, 2026-10-27 |
| `NETWORKING.md` | Server authority, remote contract, validation, exploit prevention | Planned — not yet created |
| `COMBAT_AND_DAMAGE.md` | Body-part HP, damage, disablement, combat flow | Planned — not yet created |
| `REASSEMBLY.md` | Defeat, hub, assembly, re-entry | Planned — not yet created |
| `UI_AND_CONTROLS.md` | HUD, camera, PC/mobile/gamepad controls, accessibility | Planned — not yet created |
| `TESTING.md` | Test levels, evidence requirements, regression list, and where numeric budgets are to be defined | Exists; prepared in Task 003 — approved at Gate G1, 2026-10-27 |

Gate G1 approves a document **after** it has been drafted; it is never permission to create
one. Creating, renaming, or deleting a document requires explicit owner authorization and
follows the order agreed with the owner. **No agent may create, rename, or delete a document
outside its assigned paths**, and no agent may create one of the planned documents early to
"help".

Beginner setup instructions are planned for `docs/setup.md`, which is owned by OpenCode. That
file is not created in this documentation set; it is a separate approved task after M0
tooling is decided (see `TASK_OWNERSHIP.md` §2.1).

### 7.2 Terms and identifier namespaces

These definitions exist so that a document cannot be read as granting an approval that did
not happen:

| Term | Meaning in this project |
| --- | --- |
| **Stage** / **milestone** | The same thing: a numbered delivery phase `M0`–`M9` in `ROADMAP.md`. A stage is closed at Gate G6; a single task is accepted or rejected at Gate G8. |
| **Approved** | The project owner explicitly accepted the document, task spec, or milestone at the named gate. Drafting by an agent, a passing test, or silence is **not** approval. |
| **Blocked** | A task or stage cannot proceed because a prerequisite, decision, access, or passing required test is missing. `blocked` is not acceptance and cannot be reported as success. |
| **Provisional** | A value, test, or design that depends on an unapproved document **or an unrecorded owner decision**, and cannot be counted as passed until that document is approved or that decision is recorded and the item is updated. |

Identifier namespaces used across the document set:

| Prefix | Meaning | Defined in |
| --- | --- | --- |
| `G1`–`G8` | Owner approval gates | `AGENTS.md` §11 |
| `C-###` | Implementation task files | `docs/tasks/C-###.md`, format in `TASK_OWNERSHIP.md` §6 |
| `M0`–`M9` | Stages/milestones | `ROADMAP.md` |
| `D-nn` | Provisional decisions requiring owner approval | `ROADMAP.md` §6 |
| `R-nn` | Technical risks to verify | `ROADMAP.md` §5 |
| `T-<stage>-<nn>` | Mandatory test cases | `TESTING.md` |
| `Q-nn` | Open questions | `TESTING.md` §13 |
| `A-nn` | Architecture proposals and open questions | `ARCHITECTURE.md` §10 |

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
| L0 Static | Documentation tasks: diff, links, headings, formatting, and cross-document consistency. Code tasks: files parse, the project builds, and no obvious syntax/type errors are present | Every task, in the form appropriate to its type |
| L1 Studio Play (solo) | Manual run in Studio Play mode, Output checked | Every task that changes runtime behavior |
| L2 Local server (2+ clients) | Studio "Test → Clients and Servers" with 2+ players | Any networking, combat, scoring, or respawn change |
| L3 Device emulation | Studio Device Emulator for phone/tablet | Any UI or control change |
| L4 Published playtest | Private/published server on real devices | Only stages where `ROADMAP.md` names L4, and before any release-readiness claim. M0–M2 do not require publication. Whether this rule excludes M0 is the pending owner question Q-09; see §14 |

A check that does not apply to a stage is recorded as `NOT APPLICABLE` with a reason, never as
`PASS`. Result vocabulary is defined in `TESTING.md` §1.5.

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
- Milestone checkpoints use annotated tags: `m0-scaffold`, `m1-arena`, etc. Tag names follow
  the checkpoint column in `ROADMAP.md` §3.
- A normal task commit is allowed only after **every required test has passed**. Partial
  verification — a failing or skipped required test — may be preserved only as a `wip`
  commit under §9.4. A normal commit must never carry a failing required test, and partial
  verification never satisfies the task checkpoint.
- No remote is required for now. Do not attempt to add or change remotes without approval.

## 11. Approval gates

Work stops and waits for the project owner at each gate:

| Gate | Trigger | Who may proceed |
| --- | --- | --- |
| G1 — Document approval | A drafted document set is submitted for owner approval | Owner approves the listed documents. G1 is approval **after** drafting, never permission to create a document. The G1 package is recorded in §14; `README.md` is informational and not a G1 blocker |
| G2 — Tooling | Installing or upgrading any tool (a build/sync tool, version manager, formatter, linter, Studio plugin) | Owner installs or explicitly authorizes; the specific tool, version, and pin file are named in the approved task spec first |
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
5. No new errors or warnings appear in Studio Output that were not already documented. For a
   documentation-only task, the equivalent check is the L0 documentation check in §9.1.
6. Documentation impact is reported (which document is now out of date, if any) — but not
   edited by the implementer, except the status log and evidence section of its own task file
   (`TASK_OWNERSHIP.md` §2.2).
7. The commit exists with the required message format and the task's checkpoint tag, if any.
   A `wip` commit does not satisfy this item.
8. A short report is written: what changed, why, how it was tested, what remains open.

## 13. Amending this document

- Only OpenCode may propose changes; only the project owner may approve them.
- Amendments are dated and summarized at the bottom of this file.
- Conflicting rules are resolved by the project owner, not by agent preference.

## 14. Decision log (owner approvals)

This is the single authoritative record of owner decisions: gate outcomes (G1–G8) and every
owner decision that a governance document assigns to the owner, including the provisional
decisions `D-nn` in `ROADMAP.md` §6 and the open questions `Q-nn` in `TESTING.md` §13, with
their current status and any constraint each decision imposes. Those tables are working lists
and derived records; where either disagrees with this log, this log governs. A row is only
updated after the owner explicitly states the decision; silence, an agent's summary, or a
passing test never changes it. Gate definitions are in §11.

| Decision | Scope | Status | Where the detail lives |
| --- | --- | --- | --- |
| G1 — Document approval | `AGENTS.md`, `TASK_OWNERSHIP.md`, `ROADMAP.md`, `TESTING.md`, `ARCHITECTURE.md` (the G1 package). `README.md` is informational and not part of the package | **Approved** — the project owner approved these five documents at Gate G1 on 2026-10-27. Provenance: drafts prepared in Tasks 001–004, reconciled in Task 006, amended in Tasks 008, 010, and 012 (Amendments 4–6), audited in Task 011, and reviewed in Tasks 013–014. The approval covers these five documents only; `README.md` is excluded. It decides no other item: Q-09, Q-02/G2, D-16, A-13, D-13/A-08, Q-06/A-12, and every other `D-nn`/`A-nn` remain as recorded below or in their own documents, and the three Minor findings disclosed by Task 013 (MN-1, MN-2, MN-3) remain open with the timings recorded in the Task 014 review brief | Change logs of the listed documents; `ROADMAP.md` §6 for decisions still open; Task 013 audit report and Task 014 review brief for the open Minor findings |
| Q-09 — Does L4 apply to M0? | Every normative location that states or relies on the derived L4 rule: `TESTING.md` §1.3, §1.5, §4 (the M0 device-check paragraph and the M0 exit rule) and §13; `ROADMAP.md` §2 and §4 M0; `ARCHITECTURE.md` §10.4; and this document's §9.1 L4 row | **Pending owner decision.** No owner statement is recorded. The documents apply a derived rule (L4 not applicable to M0, which has no playable build) and mark it as a proposal, not an approval. M0 work and testing may proceed, but M0 may not be accepted as compliant with an approved L4 rule until the owner records an answer; see `ROADMAP.md` §4 M0 Device checks. T-M0-08 is the D-16 place/asset policy check and is not part of this scope | `TESTING.md` §13 Q-09 note; `ROADMAP.md` §4 M0 Device checks. If the owner decides L4 applies, the locations in the Scope column are amended together through the stage-criteria procedure in `ROADMAP.md` §2.2 before M0 is accepted |

The G1 approval recorded above was entered in the same change that updated the status line of
each listed document. Later amendment of an approved document follows that document's own
amendment path and is recorded here. A pending row is never upgraded by an agent, and G1 being
approved decides nothing else: recording the Q-09 confirmation still requires its own entry here.

---

## Change log

| Date | Change | Approved by |
| --- | --- | --- |
| 2026-10-27 | Initial version created under Task 001 | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 1: portable repository-relative paths (§1.1); reconciled planned document set with ownership matrix (§7.1); added failed-test/WIP/acceptance rules (§9.2, §9.4, §10, §11 G6/G8, §12) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 2: normal commits require all required tests to pass, with partial verification limited to `wip` (§10); absolute checkout path removed (§1.1); G5 broadened to task amendment and G8 added as the task-acceptance gate (§11); §12 aligned with §9.4 | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 3 (Task 006): dated repository snapshot (§2); Cline status-log/evidence exception (§3); accurate document registry and non-circular G1 definition (§7.1); terms and identifier namespaces (§7.2); documentation-form L0 and L4 limited to stages naming it (§9.1); `m1-arena` tag example (§10); G1/G2 gate wording (§11); documentation-task DoD clause (§12); decision log with the pending G1 package (§14) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 4 (Task 008): decision log widened to cover gate outcomes and any question a governance document treats as an owner decision; Q-09 recorded as **pending owner decision** rather than resolved (§14) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 5 (Task 010): `Provisional` extended to cover a dependency on an unrecorded owner decision, not only an unapproved document (§7.2); the L4 test-level row names Q-09 as pending instead of settled (§9.1); Q-09's scope now lists every normative location that the owner's answer would change, and its status states how M0 acceptance may be recorded while it is pending (§14) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 6 (Task 012): decision-log preamble now states that §14 is the single authoritative record for gate outcomes and for every `D-nn`/`Q-nn` decision a governance document assigns to the owner, with `ROADMAP.md` §6 and `TESTING.md` §13 as derived lists (§14); Q-09 scope corrected to the M0 device-check paragraph and M0 exit rule, with T-M0-08 explicitly excluded (§14); G1 provenance extended to Tasks 008, 010, and 012 (Amendments 4–6) and the Task 011 audit, still pending (§14) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Amendment 7 (Task 015): Gate G1 owner approval recorded — the project owner approved `AGENTS.md`, `TASK_OWNERSHIP.md`, `ROADMAP.md`, `TESTING.md`, and `ARCHITECTURE.md` on 2026-10-27, with `README.md` excluded as informational (§14 G1 row, §7.1 document states, status line). No other decision is recorded: Q-09, Q-02/G2, D-16, A-13, D-13/A-08, and Q-06/A-12 remain pending, and the Task 013 Minor findings MN-1, MN-2, and MN-3 remain open with their recorded timings | Approved at G1, 2026-10-27 |
