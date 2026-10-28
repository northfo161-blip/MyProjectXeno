# TESTING.md — Test Strategy and Evidence

**Status:** approved at Gate G1 by the project owner on 2026-10-27, together with `AGENTS.md`,
`TASK_OWNERSHIP.md`, `ROADMAP.md`, and `ARCHITECTURE.md`. `README.md` is informational and
outside the approval.

This document defines how Project Xeno is tested in Roblox Studio, what evidence is required,
and how results are reported. It covers stages **M0–M2** in enforceable detail and gives
short, expandable directions for later stages.

It does not replace `AGENTS.md` §9 (test levels and evidence rules) or
`TASK_OWNERSHIP.md` (task specs and reports); it operationalises them.

---

## 1. Principles and scope

### 1.1 What this document is

- A manual-first test plan for a project with **no test framework installed** and no CI.
- A definition of the exact checks that must pass before M0, M1, and M2 are accepted.
- A reporting format that fits `TASK_OWNERSHIP.md` §6 and `AGENTS.md` §12.

### 1.2 What this document is not

- It is not proof that anything works. Nothing in this repository has been runtime-tested at
  the time of writing.
- It does not assume the existence of Jest, TestEZ, Lune, Selene, CI pipelines, or any
  automated runner. See §8.
- It does not define gameplay values. Damage, cooldowns, ranges, and similar numbers come from
  approved design documents and shared configuration modules.

### 1.3 Test levels (authoritative definition in `AGENTS.md` §9.1)

| Level | Meaning | Operational form in this project | Required when |
| --- | --- | --- | --- |
| **L0** | Static | Documentation tasks: diff, links, headings, formatting, and cross-document consistency. Code tasks: files parse, the build command succeeds, obvious syntax/type problems absent | Every task, in the form appropriate to its type |
| **L1** | Studio Play, solo | Single-player Play mode in Studio; Output panel checked | Every runtime change |
| **L2** | Studio local server, 2+ clients | "Test → Clients and Servers" with two or more players | Networking, combat, scoring, respawn |
| **L3** | Device Emulation | Studio Device Emulator using a phone and/or tablet profile | Any UI or control change |
| **L4** | Published playtest | Private or published server on real hardware | Only a stage that names L4 in `ROADMAP.md`, and any release-readiness claim. M0–M2 do not require publication |

Rules that follow from `AGENTS.md`:

- A check is only "passed" at the level actually exercised. A successful L1 run says nothing
  about L2 behaviour.
- L4 cannot be claimed without a real device and a real server; if either is unavailable, the
  result is **NOT RUN** and the limitation is recorded.
- A passing build (L0) is never evidence that gameplay works.
- An L4 check that does not apply to a stage is recorded as **NOT APPLICABLE** with a reason
  and is never counted as **PASS**. M0 has no playable build, so its L4 is recorded
  NOT APPLICABLE and contributes nothing to acceptance. That rule is **derived, not approved**:
  it is Q-09, whose owner confirmation is still pending (`AGENTS.md` §14). Pending Q-09 does
  not stop M0 work or testing, but M0 may not be accepted as compliant with an approved L4
  rule; see §4 "M0 exit" and `ROADMAP.md` §4 M0 Device checks.

### 1.4 Environments

| Environment | How it is produced | Used for |
| --- | --- | --- |
| E1 Studio edit mode | The built place opened in Studio | L0 inspection of the Explorer tree and properties |
| E2 Studio Play solo | Studio **Play** (F5) | L1 |
| E3 Studio local server | **Test → Clients and Servers**, 2 clients (3 for multi-attacker cases) | L2 |
| E4 Device Emulator | Studio **Test → Device** with a phone and a tablet profile | L3 |
| E5 Live test place | Private/published server, real PC/phone | L4 |

Every reported result must name the environment, the client count, and the Studio build if it
is relevant to a defect.

### 1.5 Result vocabulary

| Code | Meaning | Consequence |
| --- | --- | --- |
| **PASS** | Executed as written; observed result matched expectation; evidence attached | Counts toward acceptance |
| **FAIL** | Executed; result differed | Blocks the task's completion; see §11 |
| **BLOCKED** | Could not execute because a prerequisite is missing | Task cannot be accepted until resolved |
| **NOT RUN** | Deliberately not executed | Allowed only for optional checks, or as a disclosed limitation; required checks that are NOT RUN block acceptance |
| **NOT APPLICABLE** | The check has no surface at this stage (for example L4 on M0, which has no playable build) | Recorded with a reason; never counted as PASS and never used to hide a required check |
| **PARTIAL** | Some steps executed, others not | Reported explicitly; required steps that did not run block acceptance |

Where a `NOT APPLICABLE` result rests on a derived rule rather than an approved one, the report
must say so. The M0 L4 case is the current example: Q-09 still awaits owner confirmation, so an
M0 acceptance record must name the derived rule instead of presenting it as approved (§1.3,
§13, and `ROADMAP.md` §4 M0 Device checks).

`PASS` may never be recorded for a check that was not executed. This is a hard rule from
`AGENTS.md` §9.2 and §12.

---

## 2. Evidence rules

### 2.1 Acceptable evidence

- Exact command output (build logs, static-analysis output), pasted verbatim.
- Studio **Output** excerpts containing the relevant errors, warnings, or absence of errors,
  with a timestamp or sequence marker when it matters.
- A short screen recording or screenshot for visual/UI claims (leaderboard, HUD, touch
  layout). Describe what the image shows; do not rely on the image alone.
- Recorded measurements (frame time, server heartbeat, network stats) with the tool name, the
  scene, and the number of players.
- For multiplayer or input claims, the exact reproduction procedure and the number of clients.

### 2.2 Unacceptable evidence

- "It works", "should be fine", "tested locally" without an attached observation.
- Invented or reconstructed logs.
- Screenshots of an unrelated scene.
- A green build used as proof of runtime behaviour.
- Results copied from a previous run without re-executing, unless the document explicitly
  allows a regression waiver (none exists yet).

### 2.3 Environment record

Every report states:

- commit hash tested;
- Studio version or build identifier (from Help → About, if available);
- environment (E1–E5) and client count;
- date and who executed the test;
- any test scaffolding used, and whether it is still present in the build.

### 2.4 Where evidence lives

- Test evidence is written into the task file `docs/tasks/C-###.md` (Evidence section and
  Status log) for task-level work.
- For milestone acceptance, evidence is summarised in the milestone acceptance report (§10.2)
  and referenced by task ID.
- The commit body names the test levels actually run.
- Evidence must not include secrets, personal data, or unrelated player information.

---

## 3. Test case format

Every mandatory check is written in this shape:

```
ID            T-<stage>-<nn>
Title         Short imperative description
Level         L0 | L1 | L2 | L3 | L4
Requirement   Which acceptance criterion or ROADMAP statement it proves
Preconditions Concrete state, build, client count, config values
Steps         Numbered, repeatable actions
Expected      Observable result, specific enough to judge pass/fail
Evidence      What must be attached to prove the result
Negative?     Whether the check is an adversarial/negative case
```

If a test depends on an unapproved document (for example the exact remote name from
`NETWORKING.md`) **or on an unrecorded owner decision** (for example D-16, A-13, or Q-06), the
test is marked **PROVISIONAL** and cannot be counted as passed until the dependency is approved
or the decision is recorded and the test is updated (`AGENTS.md` §7.2). While it is PROVISIONAL
it is recorded `BLOCKED`, not `PASS`.

---

## 4. M0 — Toolchain and project skeleton

M0 has no gameplay. Its tests prove that the file-to-Studio pipeline is trustworthy; if this
is wrong, every later result is suspect. Two capabilities are checked separately: an
**artifact build** that produces an openable place (T-M0-01), and **live sync** from disk to
Studio (T-M0-03). Success in one does not imply success in the other.

The exact tool, version, pin file, build command, artifact path, and Studio connection method
must be defined in the approved M0 task spec and at Gate G2 before any of these tests are
executed. This document does not name a command or pin file in advance. **Rojo is a candidate
only**; no test may assume it has been selected.

### T-M0-01 — Static build succeeds

- **Level:** L0
- **Requirement:** ROADMAP M0 — the approved artifact build produces an openable place.
- **Preconditions:** G1 package approved; M0 task spec approved at Gate G4 and naming the exact
  tool, version, pin file, build command, and artifact path; Gate G2 install/authorization
  complete; clean checkout at a recorded commit.
- **Steps:**
  1. Run the exact build command defined in the approved M0 task spec.
  2. Record the exact command and its output.
  3. Confirm the output artifact exists at the path defined in that spec.
- **Expected:** command exits successfully; artifact is produced at the documented path; no
  error lines in the output.
- **Evidence:** verbatim command plus complete output; artifact path and size.
- **Negative?** No.
- **Note:** passing this test does not satisfy T-M0-03. The build and live sync are separate
  operations.

### T-M0-02 — Place opens with the agreed tree

- **Level:** L0 (inspection), plus L1 once opened
- **Requirement:** ROADMAP M0 — agreed folder tree exists.
- **Preconditions:** artifact from T-M0-01.
- **Steps:**
  1. Open the artifact in Studio (E1).
  2. Inspect `ReplicatedStorage.Shared`, `ServerScriptService.Server`,
     `StarterPlayerScripts.Client`.
  3. Confirm no placeholder script errors in Output on open.
- **Expected:** all three containers exist with the agreed names; their contents are the
  skeleton placeholders described in the approved M0 task spec; no errors. The final internal
  layout is fixed by `ARCHITECTURE.md` before M1 starts.
- **Evidence:** description of the Explorer tree (or screenshot) and Output state.
- **Negative?** No.

### T-M0-03 — Live sync from disk to Studio

- **Level:** L1
- **Requirement:** ROADMAP M0 — the approved live-sync mechanism reflects filesystem edits
  without manual copying.
- **Preconditions:** the live-sync mechanism named in the approved M0 task spec is running and
  connected; a disposable marker value exists in a placeholder module.
- **Steps:**
  1. Note the marker value in Studio.
  2. Change the marker in the file on disk.
  3. Observe Studio without restarting it.
  4. Revert the marker and observe again.
- **Expected:** the change appears in Studio in both directions of the revert, without manual
  copy/paste or place re-open.
- **Evidence:** before/after observations with timestamps; note if a Studio restart or manual
  step was required (that is a failure of this check).
- **Negative?** No.
- **Note:** a successful artifact build (T-M0-01) does not prove live sync, and a successful
  live sync does not prove a reproducible build.

### T-M0-04 — No gameplay code is present

- **Level:** L0 (file review) + L1 (runtime)
- **Requirement:** ROADMAP M0 — scope discipline; no arena, combat, or character logic yet.
- **Preconditions:** M0 build.
- **Steps:**
  1. Review the source tree for gameplay logic, remotes, or data systems.
  2. Play solo (E2) and observe Output.
- **Expected:** only skeleton modules and bootstrap placeholders; no gameplay remotes, no
  arena logic, no combat; Output free of project errors.
- **Evidence:** file list with a one-line description of each; Output excerpt.
- **Negative?** No.

### T-M0-05 — A fresh checkout reproduces the build

- **Level:** L0
- **Requirement:** reproducibility across machines, per `AGENTS.md` §1.1.
- **Preconditions:** a second checkout location or a clean clone; toolchain installed per the
  approved M0 task spec.
- **Steps:**
  1. Clone/copy the repository to a path different from the original.
  2. Run the exact build command defined in the approved M0 task spec, without editing files.
- **Expected:** same successful build; no absolute paths or machine-specific settings needed.
- **Evidence:** command output from the second location; note any manual step required.
- **Negative?** No.

### T-M0-06 — Toolchain versions are pinned and documented

- **Level:** L0
- **Requirement:** reproducibility.
- **Preconditions:** the approved M0 task spec names the pin file(s) and the versions expected.
- **Steps:** inspect the version pin file(s) named in the approved M0 task spec and the
  documented setup steps; compare with the versions actually installed.
- **Expected:** pinned versions exist, match what was used, and setup steps are sufficient for
  a beginner to repeat them.
- **Evidence:** pin file contents and installed version output.
- **Negative?** No.
- **Note:** no pin file or version is assumed before the M0 task spec is approved.

### T-M0-07 — Sync failure is visible, not silent

- **Level:** L1 (diagnostic)
- **Requirement:** predictable workflow.
- **Preconditions:** the approved live-sync mechanism is connected.
- **Steps:**
  1. Stop the sync process (or disconnect the mechanism).
  2. Edit a file on disk.
  3. Observe Studio and the sync mechanism's user interface.
- **Expected:** the disconnected state is clearly indicated; Studio does not silently show
  stale content as if it were current; reconnecting restores sync.
- **Evidence:** description of the indicator; confirmation that no silent stale edit occurred.
- **Negative?** Yes.

### T-M0-08 — Place and asset policy is honoured

- **Level:** L0
- **Requirement:** ROADMAP M0 / decision D-16.
- **Status:** **PROVISIONAL** while the place and asset policy (D-16 / Q-03) is unconfirmed.
- **Authoritative record:** the decision log in `AGENTS.md` §14 is the single authoritative
  record of the owner's D-16 decision. The `ROADMAP.md` §6 row and the change logs follow it;
  none of them can substitute for it, and a note in this test is not a decision.
- **Status transition:** this test becomes **FINAL** only when the owner has recorded the D-16
  policy decision in the `AGENTS.md` §14 decision log. One single documentation change performs
  the transition:
  1. this status line becomes **FINAL**, and the preconditions, steps, and expected result are
     rewritten to the approved policy;
  2. the `ROADMAP.md` §6 table shows D-16 as decided (and §7's nearest-action step for D-16
     is updated);
  3. `ROADMAP.md` and this document each receive a change-log row recording the decision and
     the PROVISIONAL→FINAL transition.
  **Prerequisites for executing the test as FINAL:** items 1 and 2. The test's steps and the
  approved policy must exist before the test can be run as a decided check. Item 3 is a
  documentation duty of the same change, not a separate gate or approval, but it may not be
  deferred in order to run the test earlier. No other event — an agent's summary, a passing
  build, or silence — moves the test out of PROVISIONAL.
- **Until that transition:** the test may not be recorded as `PASS`. Its result is recorded as
  `BLOCKED` (prerequisite missing), never `PASS` and never `NOT APPLICABLE`, and M0 cannot be
  accepted at Gate G6.
- **Preconditions:** owner-approved policy for built places and assets. The proposed policy in
  `ROADMAP.md` §6 is the expected baseline, not an approved rule: no publication, no
  third-party assets with unclear rights, source and configuration committed, generated builds
  and local data excluded.
- **Steps:** inspect the working tree and Git status after a build; confirm generated artifacts
  are excluded or committed exactly as the policy states.
- **Expected:** no unintended binary artifacts in Git; policy documented and followed.
- **Evidence:** `git status` output and the policy statement it is compared against.
- **Negative?** No.

### M0 exit

M0 may be accepted at Gate G6 only when T-M0-01 through T-M0-08 are PASS with evidence
attached. Any FAIL or required NOT RUN blocks acceptance.

T-M0-08 follows the transition rule in its own section: while it is **PROVISIONAL**, it is
recorded `BLOCKED`, and that blocked result also blocks M0 acceptance. M0 can therefore be
accepted only after the place and asset policy decision is recorded in the `AGENTS.md` §14
decision log, T-M0-08 has moved to **FINAL** with the `ROADMAP.md` §6 row and both change logs
updated, and it has then been executed as PASS. `PROVISIONAL` is never converted into `PASS`
retroactively.

If a test's expectation is wrong, the task specification is amended at Gate G5 and the test is
re-run; a failed or skipped required test is never waived or carried into acceptance
(`AGENTS.md` §9.4, §12).

M0 has no playable or published build, so **L4 is NOT APPLICABLE** and is recorded that way. It
is not a passed check and does not contribute to M0 acceptance. This is the derived rule Q-09,
whose owner confirmation is pending (`AGENTS.md` §14), so the Gate G6 acceptance record must say
either that Q-09 was still pending and L4 was recorded under the derived rule, or that the owner
confirmed the rule. If the owner decides L4 applies to M0, this exit rule, the Q-09 row and note
in §13, the M0 device-check paragraph in `ROADMAP.md` §4, and the Q-09 entry in `AGENTS.md` §14
are amended first, and the added L4 check must be run before M0 is accepted. Accepting M0 on an
unconfirmed rule is not permitted.

---

## 5. M1 — Arena, spawning, and respawn

### T-M1-01 — Two clients spawn at valid, distinct positions

- **Level:** L2
- **Requirement:** ROADMAP M1 — two clients in a Studio local server spawn at valid,
  non-overlapping positions.
- **Preconditions:** M0 accepted; arena and spawn configuration present; two clients (E3).
- **Steps:** start a local server with two clients; observe both characters at spawn; repeat
  three times with fresh sessions.
- **Expected:** both players spawn inside the arena on valid pads; positions are not identical
  and not overlapping; no player spawns outside the playable area.
- **Evidence:** observed positions (or screenshots) for each run; Output state.
- **Negative?** No.

### T-M1-02 — Movement and camera work for both clients

- **Level:** L2
- **Requirement:** ROADMAP M1 — movement and camera behave normally for both players (L2
  half of the movement criterion; T-M1-07 covers the touch half at L3).
- **Steps:** with two clients, move, jump, rotate the camera, and walk into geometry.
- **Expected:** movement is responsive and identical in behaviour for both clients; no
  rubber-banding beyond normal Roblox behaviour; camera does not clip permanently.
- **Evidence:** described observations, plus a short recording if feasible.
- **Negative?** No.

### T-M1-03 — Out-of-bounds elimination and timed respawn

- **Level:** L2
- **Requirement:** ROADMAP M1 — an out-of-bounds or kill-volume death respawns the player
  after a fixed, configurable delay.
- **Status:** **PROVISIONAL** while the A-13 non-combat damage rule is unrecorded. The intended
  elimination mechanism (kill volume or fall boundary) is a non-combat death cause, and
  `ROADMAP.md` §4 M1 states that criterion cannot be implemented or tested until a recorded rule
  says how non-combat damage is handled. Until then this check is reported `BLOCKED`, never
  `PASS`, and no assumed damage policy may be used to make it pass.
- **Preconditions:** kill volume or fall boundary configured; respawn delay configured.
- **Steps:** walk one player out of bounds; measure the time from elimination to respawn.
- **Expected:** the player is eliminated by the intended mechanism, respawns after the
  configured delay at a valid spawn, and the other player is unaffected.
- **Evidence:** measured delay, spawn location, Output excerpt.
- **Negative?** No.

### T-M1-04 — Ten consecutive respawn cycles

- **Level:** L2
- **Requirement:** ROADMAP M1 — ten consecutive respawn cycles complete without script
  errors or stuck characters.
- **Status:** **PROVISIONAL** for the same A-13 reason: every elimination available in M1 is a
  non-combat death, so the cycle cannot be driven before the non-combat rule is recorded.
  Reported `BLOCKED`, never `PASS`, until then.
- **Steps:** eliminate one player ten times in sequence; after each cycle, move and jump.
- **Expected:** all ten cycles complete; no stuck character, no duplicate character, no script
  errors, no accumulating warnings; the second player remains connected throughout.
- **Evidence:** per-cycle notes and the final Output state.
- **Negative?** No.

### T-M1-05 — Only intended volumes eliminate players

- **Level:** L2
- **Requirement:** ROADMAP M1 — elimination happens only through the intended kill volume
  or fall boundary (negative half of the out-of-bounds criterion).
- **Status:** **PROVISIONAL** for the kill-volume step, which depends on the unrecorded A-13
  rule. The decorative-geometry half can be observed while A-13 is pending, but it does not make
  the check pass on its own; the test is reported `BLOCKED` until the rule is recorded.
- **Steps:** walk through the arena, touch decorative geometry, stand on spawn pads, then
  deliberately enter the kill volume.
- **Expected:** only the intended kill volume or fall boundary eliminates; normal geometry
  does not.
- **Evidence:** list of surfaces tested with outcomes.
- **Negative?** Yes.

### T-M1-06 — Spawn and respawn values come from configuration

- **Level:** L1
- **Requirement:** ROADMAP M1 — spawn positions and respawn delay come from shared
  configuration, not hard-coded values.
- **Preconditions:** one configuration value (spawn set or respawn delay) is safe to change.
- **Steps:** change the value; rebuild with the approved M0 build command; observe behaviour;
  restore the value.
- **Expected:** behaviour changes accordingly; no source edit is needed to retune; restoring
  the value restores behaviour.
- **Evidence:** before/after observations and the configuration diff.
- **Negative?** No.

### T-M1-07 — Touch movement smoke test

- **Level:** L3
- **Requirement:** ROADMAP M1 — default mobile touch controls remain usable (L3 half of
  the movement criterion; T-M1-02 covers the L2 half).
- **Preconditions:** Device Emulator, phone profile.
- **Steps:** play solo on the emulator; use the default thumbstick, jump button, and camera.
- **Expected:** default touch controls move the character and rotate the camera; no custom UI
  blocks them; the character can leave the spawn area and return.
- **Evidence:** emulator profile used, observations, screenshot of the layout.
- **Negative?** No.

### T-M1-08 — Join and leave do not corrupt the session

- **Level:** L2
- **Requirement:** ROADMAP M1 goal — multiple players reliably enter and stay in the
  shared arena; a join or leave must not corrupt the session for the others.
- **Steps:** with one client running, join with a second, leave, rejoin; repeat twice.
- **Expected:** remaining player is unaffected; rejoining player spawns correctly; no errors or
  duplicate characters.
- **Evidence:** observations and Output excerpt.
- **Negative?** Yes.

### T-M1-09 — No combat exists yet

- **Level:** L2 (scope check)
- **Requirement:** ROADMAP M1 — the out-of-scope list excludes any attack, so M1 must
  contain no combat (M0/M1 stage boundary).
- **Steps:** attempt to damage another player by any available input.
- **Expected:** no damage is possible; no combat remotes are present.
- **Evidence:** statement of attempts made and absence of combat code/remotes.
- **Negative?** Yes.

### M1 exit

Every M1 check, T-M1-01 through T-M1-09, must PASS at its stated level before M1 is accepted.
T-M1-01, T-M1-03, T-M1-04, and T-M1-07 are named here only because they carry the stage's core
evidence; naming them does not make any other M1 check optional, skippable, or downgradable.
M1 cannot be accepted on L1 evidence alone. The checks marked **PROVISIONAL** on the unrecorded
A-13 non-combat rule (T-M1-03, T-M1-04, T-M1-05) are recorded `BLOCKED` until that rule is
recorded; while any of them is BLOCKED, M1 cannot be accepted.

---

## 6. M2 — Server-authoritative combat and overall health

M2 is where authority is proven or lost. The exact remote names, payload schema, validation
limits, and cooldowns are defined in `NETWORKING.md`; until that document is approved, M2
tests that reference them are **PROVISIONAL** and must be updated to match it. Rules owned by
`COMBAT_AND_DAMAGE.md` work the same way: where a check compares behaviour against them, that
part is **PROVISIONAL** until the rules are approved (the non-combat half of T-M2-02 and the
partial-hit half of T-M2-04).

### 6.1 Positive-path checks

#### T-M2-01 — Two clients damage each other through server resolution

- **Level:** L2
- **Requirement:** ROADMAP M2 — two clients can damage each other, and health decreases
  only through server-side resolution.
- **Preconditions:** M1 accepted; combat enabled; two clients; documented damage value.
- **Steps:** each player fires at the other from a clear line of sight; record health before
  and after each shot.
- **Expected:** health decreases by the configured amount per valid hit; both directions work;
  no duplicate damage per shot.
- **Evidence:** health values per shot, number of shots, Output state.
- **Negative?** No.

#### T-M2-02 — Elimination and respawn path

- **Level:** L2
- **Requirement:** ROADMAP M2 — combat elimination occurs only at zero server-resolved
  health and routes through the M1 respawn path; the non-combat cause runs only under the
  recorded A-13 rule.
- **Status:** **PROVISIONAL** for the non-combat half while the treatment of non-combat damage
  (A-13, `ARCHITECTURE.md` §10.3) is unrecorded; A-13 is the M1 prerequisite for the
  kill-volume rule. Until it is recorded, the non-combat case is reported `BLOCKED`, never
  `PASS`, and no assumed damage policy may be implemented to make it pass.
- **Steps:** reduce one player to zero health through combat; observe death, respawn timing,
  and state. Separately, trigger the non-combat death cause defined by the recorded A-13 rule
  (the M1 kill volume or fall boundary).
- **Expected:** combat elimination occurs only when server-resolved overall health reaches
  zero; it is resolved by the server and routes through the M1 respawn flow. The non-combat
  cause defined by the A-13 rule also eliminates and routes through the same flow without
  requiring zero combat health. Health resets correctly; the surviving player is unaffected.
- **Evidence:** health trace, timing, Output excerpt, and the death cause observed in each case,
  together with the recorded A-13 rule it is compared against.
- **Negative?** No.

#### T-M2-03 — Cooldown is enforced

- **Level:** L2
- **Requirement:** ROADMAP M2 — firing faster than the configured cooldown is throttled
  server-side.
- **Steps:** fire as fast as input allows for a fixed period; count accepted hits versus
  attempted shots.
- **Expected:** accepted shots never exceed the configured rate; excess attempts are ignored
  or rejected without errors.
- **Evidence:** attempted versus accepted counts and the observed interval between accepted
  shots.
- **Negative?** Yes (rate abuse).

### 6.2 Authority and negative checks

These checks exist to prove the client cannot decide outcomes. They must be executed only on a
local test server (E3), never on a live place.

#### T-M2-04 — Line-of-sight and cover rejection

- **Level:** L2
- **Requirement:** ROADMAP M2 — a hit behind cover deals no damage; the partially exposed
  case is judged only against the approved partial-hit rules in `COMBAT_AND_DAMAGE.md`
  (PROVISIONAL half).
- **Status:** **PROVISIONAL** for the partial-hit half only. `COMBAT_AND_DAMAGE.md` does not
  exist and is not approved, and no document defines how a partially exposed target resolves.
  Until those rules are approved, the partial-hit half is recorded `BLOCKED`, never `PASS`, and
  M2 cannot be accepted while it is unresolved. The fully blocked half is independent of that
  document and may be executed and recorded on its own.
- **Steps:** fire at a target fully behind solid geometry; then fire at a partially exposed
  target.
- **Expected:** fully blocked shots deal no damage. For the partially exposed target, the
  outcome must match the approved partial-hit rules in `COMBAT_AND_DAMAGE.md`; until those
  rules are approved that half has no expected result to judge and is `BLOCKED`.
- **Evidence:** positions used, outcomes, and the geometry involved.
- **Negative?** Yes.

#### T-M2-05 — Range rejection

- **Level:** L2
- **Requirement:** ROADMAP M2 — a hit beyond maximum range deals no damage; a hit inside
  range resolves normally.
- **Steps:** fire from beyond the configured maximum range; then from just inside it.
- **Expected:** beyond range deals no damage; inside range deals normal damage.
- **Evidence:** measured distances and outcomes.
- **Negative?** Yes.

#### T-M2-06 — Server-derived attack origin cannot be forged

- **Level:** L2
- **Requirement:** ROADMAP M2 — the server resolves the attack from its own view of the
  shooter; a client-supplied origin is not part of the protocol.
- **Status:** **PROVISIONAL** until `NETWORKING.md` is approved and confirms the payload
  schema. It must not be marked PASS while the protocol is undefined.
- **Steps:** attempt to influence the shot origin from the client, for example by moving the
  character client-side or by adding an origin-like value to the attack request. Then fire
  normally.
- **Expected:** the server resolves the attack from its own view of the shooter; the client
  cannot force damage from an impossible or unrelated origin. If the request shape is
  invalid, it is rejected without an error.
- **Evidence:** described manipulation method, outcome, and Output state.
- **Negative?** Yes.
- **Note:** the approved protocol does not require a client-supplied origin field. This test
  verifies the trust boundary, not a nonexistent payload field.

#### T-M2-07 — Outcome claims in the payload are rejected

- **Level:** L2
- **Requirement:** ROADMAP M2 — the attack payload carries no damage or outcome field;
  extra or unknown fields cannot change damage, health, or elimination.
- **Status:** **PROVISIONAL** until `NETWORKING.md` is approved. The test must be updated to
  the real payload shape; no damage, health, or kill field is assumed to exist.
- **Preconditions:** a documented way to send a hand-crafted request on the local test server
  (see §7 on test scaffolding).
- **Steps:** send the attack request with an extra or unknown field claiming damage, a health
  value, or an instant kill; observe target health and server state.
- **Expected:** the extra field grants no authority: target health is unchanged except through
  the normal validated attack path; no elimination occurs from the crafted request; the server
  rejects or ignores it without erroring.
- **Evidence:** payload shape (redacted of anything sensitive), observed health, server output.
- **Negative?** Yes.

#### T-M2-08 — Malformed payloads are rejected safely

- **Level:** L2
- **Requirement:** ROADMAP M2 — a burst of malformed remote calls produces no server
  errors.
- **Status:** **PROVISIONAL** until `NETWORKING.md` is approved and the exact payload shape is
  known. It must be updated to the real schema and re-run.
- **Steps:** send, one at a time: wrong argument count; `nil`; wrong type (string instead of
  number/vector); `NaN` and infinity; negative or absurdly large numbers; a table or instance
  where a vector is expected; an extra argument claiming a target or another player.
- **Expected:** every case is rejected or ignored; no damage occurs; no server error, no
  disconnect of innocent players, no state corruption. Identity and target selection remain
  server-derived; a client-supplied player reference grants no authority.
- **Evidence:** table of payload → observed outcome → Output state.
- **Negative?** Yes.

#### T-M2-09 — Frequency abuse does not degrade the server

- **Level:** L2
- **Requirement:** ROADMAP M2 — a burst of high-frequency remote calls does not degrade
  the server or other players.
- **Steps:** send a burst of remote calls (start at 100, then 1,000) at the highest permitted
  rate for a fixed period; monitor the second client's experience and Studio performance
  stats.
- **Expected:** excess calls are throttled or dropped; no script errors; the second player's
  movement and combat remain responsive; the server does not stall.
- **Evidence:** burst size, duration, observed behaviour of both clients, and any available
  server stats.
- **Negative?** Yes.

#### T-M2-10 — Client cannot authoritatively change health

- **Level:** L2
- **Requirement:** ROADMAP M2 — health decreases only through server-side resolution; a
  client cannot author it.
- **Steps:** from a client, attempt to set the local Humanoid health or a replicated health
  value directly; observe both the local view and the server's view.
- **Expected:** the server remains authoritative; any local change is corrected or has no
  effect on damage, elimination, or other players.
- **Evidence:** before/after values on both clients and the server-observed outcome.
- **Negative?** Yes.

#### T-M2-11 — Abuse does not leak into other sessions or players

- **Level:** L2
- **Requirement:** ROADMAP M2 — abuse does not degrade other players (the malformed and
  high-frequency criterion), checked with an uninvolved third client; blocked by Q-06/A-12
  because it runs T-M2-07 through T-M2-09.
- **Status:** **PROVISIONAL** while the Q-06 / A-12 scaffolding decision is undecided. Its steps
  run T-M2-07 through T-M2-09, which require an authorized test-scaffolding mechanism (§7), so
  this check cannot be executed before those three can. Until the decision is recorded it is
  reported `BLOCKED`, never `PASS`, and it never counts toward M2 acceptance on the strength of
  the other negative checks.
- **Steps:** run T-M2-07 through T-M2-09 while a third client plays normally.
- **Expected:** the third client sees no false damage, no score changes, no disconnects, and no
  visible lag beyond normal.
- **Evidence:** observations from the uninvolved client.
- **Negative?** Yes.

#### T-M2-12 — Attack input is reachable on touch

- **Level:** L3
- **Requirement:** ROADMAP M2 Device checks — one L3 check that the temporary attack input
  is reachable on touch; it is not a camera or aiming test (D-13/A-08 remain open).
- **Steps:** play solo on a phone profile; locate and use the temporary attack control.
- **Expected:** the control is visible, reachable with one thumb, and triggers the attack; it
  does not overlap the default movement controls.
- **Evidence:** profile used, screenshot, described interaction.
- **Negative?** No.

### 6.3 What M2 does not prove

Even with every M2 check passing:

- It does not prove resistance to a modified Roblox client, only that server-side validation
  rejects the documented abuse cases in Studio.
- It does not prove production-network behaviour (latency, jitter, region).
- It does not prove mobile performance; L2 runs on the development machine.
- It does not prove anti-cheat completeness; residual risk is documented, not eliminated.

These limits are stated in the milestone report, not hidden.

### M2 exit

M2 may be accepted at Gate G6 only when T-M2-01 through T-M2-12 are PASS at their stated
levels, with every **PROVISIONAL** test resolved and re-run: after `NETWORKING.md` and the
relevant `COMBAT_AND_DAMAGE.md` sections (including the partial-hit rules behind T-M2-04) are
approved, after the A-13 non-combat damage rule is recorded (the non-combat half of T-M2-02),
and after the Q-06 / A-12 scaffolding decision allows T-M2-07 through T-M2-09 and their
dependent check T-M2-11 to be executed. Any FAIL or required NOT RUN blocks acceptance, and no
required test may be dropped or redefined to reach the milestone. Negative tests (T-M2-04
through T-M2-11) may not be skipped or downgraded to L1.

---

## 7. Test scaffolding policy

Some checks (crafted payloads, forced teleports, health manipulation) require a mechanism that
sends or runs something outside the normal game flow. **Every such mechanism falls under the
Q-06 / A-12 decision**, including:

- commands typed into the Studio **Command Bar**;
- temporary `Script`, `LocalScript`, or `ModuleScript` files added for a test, whether they live
  in the built place, in a Studio-only test place, or in the repository source tree;
- repository test scaffolding of any kind — helper modules, test-only remotes, fixture files,
  or scripts committed to, or left in, the working tree.

None of these mechanisms is approved today. Until the owner records the Q-06 decision (the
`AGENTS.md` §14 decision log is where owner decisions are recorded; the §13 row here is updated
to match), the checks that need scaffolding are recorded as **BLOCKED**; they are never marked
PASS, never recorded as NOT APPLICABLE, and never quietly replaced by a weaker check. The
blocked set is T-M2-07, T-M2-08, T-M2-09, and T-M2-11, whose steps run those three checks.
While any of them is BLOCKED, M2 cannot be accepted or treated as complete.

Rules:

1. Scaffolding is allowed only on a local test server (E3), never in a published place.
2. Every scaffolding item must be listed in the task file with its file path and purpose.
3. Scaffolding must be disabled or removed before the task is submitted for review; if it
   remains, the task cannot be accepted until the owner explicitly approves its retention.
4. A test hook must never be reachable from a normal client in a live game.
5. If no safe way exists to send crafted payloads yet, the check is recorded as **BLOCKED**
   with the reason, not as PASS.

---

## 8. Tooling reality and unknowns

### 8.1 Confirmed

- No test framework, package manager, or CI is installed in this repository.
- Testing is manual in Roblox Studio unless the owner approves tooling at **Gate G2**.

### 8.2 Candidate tools requiring approval and verification

| Tool | Potential use | Status |
| --- | --- | --- |
| Build/sync tool (Rojo is one candidate, not a selection) | Mapping repository files into Roblox and syncing Studio with disk | Not selected and not installed; the tool, version, pin file, build command, artifact path, and sync method are chosen in the M0 task spec and approved at Gate G2 |
| Selene / StyLua | L0 static checks and formatting | Not installed; requires G2 approval and a check that it supports the chosen Luau syntax |
| Lune | Running pure Luau module tests outside Roblox | Not installed; viability for this project unverified |
| TestEZ or similar in-engine runner | Automated unit tests inside Studio | Not installed; whether it fits the project structure is unverified |
| Studio Command Bar / temporary scripts | Manual crafted-payload tests | Available in principle, but every such mechanism needs the Q-06 / A-12 decision first (§7); exact usage must be documented per task |
| Performance Stats / MicroProfiler | Measuring server and client cost | Must be verified as available and readable in the installed Studio build |

No tool in this table may be assumed present, and no test result may depend on one that is not
installed.

---

## 9. Regression policy

- **Per task:** run the smoke set for the affected area plus every mandatory check whose
  behaviour the change could affect.
- **M0:** T-M0-01 through T-M0-08 whenever the project layout, tooling, or build changes.
- **M1:** T-M1-01, T-M1-03, T-M1-04, T-M1-08 for any spawn, respawn, or arena change.
- **M2:** T-M2-01, T-M2-02, T-M2-04, T-M2-07, T-M2-08, T-M2-09 for any remote, validation,
  damage, or health change.
- **Before a milestone is submitted:** the full mandatory set for that stage.
- **After any change to `NETWORKING.md`:** re-run the M2 negative checks, because the expected
  rejections may have changed.
- A skipped regression check is recorded as NOT RUN with a reason; it may not be silently
  omitted.

---

## 10. Reporting format

### 10.1 Per-task report (`C-###`)

Aligned with `TASK_OWNERSHIP.md` §6 ("Evidence required") and `AGENTS.md` §12. The report is
written into the task file and repeated in the completion message.

```markdown
## Test report — C-###

- Commit: <full hash>
- Studio: <version/build, if known>
- Environment(s): E1/E2/E3/E4 (list), clients: <n>
- Date / executor: <date> / <agent or owner>
- Test scaffolding present: <none | list, with paths and removal status>

| Test ID | Level | Result | Evidence (short) |
| --- | --- | --- | --- |
| T-M#-## | L# | PASS/FAIL/BLOCKED/NOT RUN/NOT APPLICABLE/PARTIAL | log excerpt, measurement, observation |

### Failures and blockers
- <test ID>: observed vs expected, exact error text, suspected cause, status

### Limitations
- Levels not run and why (e.g. no real device, no second client available)

### Residual risk
- <what remains unproven after these tests>
```

Rules:

- Every mandatory test for the stage appears in the table, including those NOT RUN or NOT
  APPLICABLE.
- The summary line in the task file's status log records PASS/FAIL counts and the highest
  level reached.
- A FAIL or required NOT RUN keeps the task `in-progress` or `blocked`; a `wip` commit is
  permitted only under `AGENTS.md` §9.4.

### 10.2 Milestone acceptance report (Gate G6)

```markdown
## Milestone report — M#

- Stage: M#
- Tasks included: C-###, C-###
- Commit range: <first>..<last>
- Checklist: <link to ROADMAP M# completion criteria>
- Mandatory tests: <passed>/<total>; failures: <n>
- Levels exercised: L0, L1, L2, L3, L4 (state which were NOT RUN or NOT APPLICABLE and why)
- Owner-visible demo: <what the owner can personally verify>
- Open defects: <ID, severity, impact>
- Recommendation: accept / reject / accept with recorded limitations (never with a failing
  required test)
- Owner decision (Gate G6): <date, accepted/rejected>
```

A milestone report may not claim a level it did not exercise. An L4 claim requires a published
or private server and a real device. "Recorded limitations" may describe levels that are not
applicable or optional work deferred; it may never mask a failing or skipped required test.

### 10.3 Status transitions

Testing results feed the lifecycle in `TASK_OWNERSHIP.md` §4:

- all required tests PASS + non-`wip` commit → task may move to `review`;
- any FAIL → task stays `in-progress` or becomes `blocked`;
- owner accepts at **G8** (task) or **G6** (milestone); they are not interchangeable.

---

## 11. Defect handling

| Severity | Definition | Effect on acceptance |
| --- | --- | --- |
| **S1 — Critical** | Server authority broken (damage, death, or state decided by a client); server crash; data corruption | Blocks task and milestone; no workaround accepted |
| **S2 — Major** | Core loop broken (cannot spawn, respawn, damage, or re-enter); mandatory test fails | Blocks the affected task |
| **S3 — Minor** | Incorrect behaviour with a safe workaround; cosmetic or edge-case defect | May be accepted with a recorded known issue and owner approval, **only if no mandatory check fails** |
| **S4 — Trivial** | Cosmetic or documentation nit | Recorded, does not block |

Rules:

- S1 and S2 always block; they cannot be waived by an agent and are not converted into S3 to
  pass a stage.
- Known issues are listed in the milestone report with severity and reproduction steps.
- Fixing a defect requires a task; unauthorised drive-by fixes remain prohibited by
  `AGENTS.md` §4.

---

## 12. Directions for future stages (brief)

No detailed cases are written for mechanics that do not exist yet. These are directions only.

- **M3 (scoring):** attribution rules, exactly-one-award per elimination, client score
  injection attempts, leaderboard legibility at phone scale.
- **M4 (parts):** per-limb hit attribution, part HP isolation, disable replication, "part never
  deleted or detached" check, state reset on respawn.
- **M5 (abilities):** gating by part state, server cooldown enforcement, duplicate-activation
  and spam attempts, immediate lockout when a part is disabled mid-cooldown.
- **M6 (ground mode):** movement change on leg loss, survival of remaining abilities, reset on
  respawn, no client-side escape from the grounded state.
- **M7 (hub/assembly):** no direct arena respawn, server-side loadout validation, repeated
  full-loop cycles, disconnect/rejoin consistency.
- **M8 (mobile):** all abilities reachable by touch, no overlap with default controls, safe
  areas, phone and tablet profiles, full loop with touch only.
- **M9 (integration):** full regression, adversarial pass, performance measurement, L4
  private-server playtest, known-issues list.

Each of these will be expanded into mandatory checks when its stage's design documents are
approved.

---

## 13. Open questions and decisions

| ID | Question | Blocks | Notes |
| --- | --- | --- | --- |
| Q-01 | Owner approval of the G1 package: `AGENTS.md`, `TASK_OWNERSHIP.md`, `ROADMAP.md`, `TESTING.md`, `ARCHITECTURE.md` (Gate G1) | M0 — satisfied 2026-10-27 | **Decided: approved.** The owner approved the five listed documents at Gate G1 on 2026-10-27; the decision is recorded in `AGENTS.md` §14. `README.md` is informational and not part of the package. The approval decides no other row here: Q-09, Q-02, D-16, A-13, D-13, and Q-06 remain as recorded |
| Q-02 | Tooling selection, installation, and version pinning (Gate G2) | M0 | The M0 task spec must name the candidate tool, version, pin file, commands, and artifact path; Rojo is a candidate only |
| Q-03 | Place and asset policy (ROADMAP D-16) | M0 | Proposed policy in `ROADMAP.md` §6; owner confirmation pending. Needed for T-M0-08 only. T-M0-01 does not depend on it: that check only requires the build artifact to appear at the documented path |
| Q-04 | Target player count for v1 (ROADMAP D-12) | Later stages (M9), not M0 | Two clients are the M0–M2 test-session size and the minimum network check; not a server cap or architectural limit, and not an M0 blocker |
| Q-05 | Availability of a second machine or real mobile device for L4 | M9, but plan early | If unavailable, L4 is recorded as NOT RUN |
| Q-06 | Whether temporary test scaffolding may exist in the repository (and where) | T-M2-07, T-M2-08, T-M2-09, and the dependent T-M2-11, which runs those three checks | Covers every mechanism for crafted-payload testing: Studio Command Bar commands, temporary scripts, and repository test scaffolding (§7). Needed before those checks can be executed; it does not block the rest of M2 implementation, but M2 cannot be accepted while they are BLOCKED |
| Q-07 | Whether any automated test tool will be approved | Later stages | No tool may be presumed available |
| Q-08 | Studio build identification method (Help → About vs other) | Every report that records a Studio build | Needed only when a report records the Studio build. §1.4 makes the build conditional ("if it is relevant to a defect") and the §10.1 template allows "if known", so this row adds no requirement beyond them. Method still undecided |
| Q-09 | Does L4 apply to M0? | M0 device checks, and how M0 acceptance may be recorded | **Not an approved owner decision.** The rule currently applied is derived from the drafts: see the note below and the pending row in `AGENTS.md` §14. Does not block M0 work; constrains what the M0 acceptance record may claim |

**Q-09 — status: derived rule, owner confirmation pending.** The rule applied throughout this
document is derived, not approved. Two drafts support it: `AGENTS.md` §9.1 requires L4 only for
stages where `ROADMAP.md` names it, and `ROADMAP.md` §4 M0 records **L4 NOT APPLICABLE**
because M0 has no playable or published build. On that basis L4 on M0 is recorded as
`NOT APPLICABLE` with a reason and is never marked `PASS`.

The owner has not recorded a decision on Q-09 in the decision log (`AGENTS.md` §14); that log
lists it as pending. Gate G1 approval of this document does not decide it, so the M0
device-check wording stays a derived proposal within the approved package until the owner
confirms it.

**Effect on M0 acceptance.** Q-09 does not block M0 from starting, being implemented, or being
tested: M0 produces no playable or published build, so no L4 check exists to run either way. It
does constrain acceptance. The Gate G6 record for M0 must state either that Q-09 was still
pending and L4 was recorded `NOT APPLICABLE` under the derived rule, or that the owner has
confirmed the rule. M0 may not be accepted as if it satisfied an L4 requirement that the owner
has not decided.

**If the owner decides that L4 does apply to M0,** these locations are amended together through
the stage-criteria procedure in `ROADMAP.md` §2.2 before M0 is accepted:

- this section (the Q-09 row and this note);
- §1.3 and §1.5, which define how an L4 result is recorded;
- §4, both the M0 device-check paragraph, where L4 is recorded `NOT APPLICABLE`, and the M0
  exit rule that states what the acceptance record must claim;
- `ROADMAP.md` §2 (the L4 rule) and §4 M0 Device checks;
- `AGENTS.md` §14 Q-09 row and §9.1 L4 row;
- `ARCHITECTURE.md` §10.4.

T-M0-08 is not part of this list. It is the D-16 place/asset policy check and says nothing
about L4; the L4 rule lives only in the M0 device-check paragraph and the M0 exit rule.

The added L4 check must then be run, and it cannot be waived on the ground that M0 work had
already finished. Until that amendment, no section may describe the derived rule as an approved
owner decision.

---

## 14. Change log

| Date | Change | Approved by |
| --- | --- | --- |
| 2026-10-27 | Initial draft created under Task 003: M0–M2 mandatory checks, evidence rules, report formats | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 006 reconciliation: documentation-form L0 and stage-scoped L4 with NOT APPLICABLE (§1.3, §1.5); separate build and live-sync checks with no assumed command or pin file (T-M0-01/03/05/06/07); D-16 proposed policy (T-M0-08); M0 L4 recorded NOT APPLICABLE; M2 death causes aligned with M1 kill volume (T-M2-02); M2 origin/outcome tests aligned with the real protocol and marked PROVISIONAL (T-M2-06/07/08); G1 package, D-12 deferral, and Q-09 resolution (§13) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 008 reconciliation of review findings: Q-09 reclassified from "resolved" to a derived rule with owner confirmation pending, and added as a table row (§13); explicit PROVISIONAL→FINAL transition for T-M0-08 and the matching M0 exit rule (T-M0-08, M0 exit); Q-03 dependency narrowed to T-M0-08 so T-M0-01 is not blocked by it (§13); Q-06 recorded as blocking T-M2-07/08/09 only, with the M2 exit rule and scaffolding preconditions updated (§7, §13, M2 exit) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 010 correction of Task 009 findings: PROVISIONAL extended to an unrecorded owner decision (§3); T-M0-08 names the `AGENTS.md` §14 decision log as the single authoritative record, lists the roadmap-table and change-log updates, and states which are prerequisites for execution (T-M0-08, M0 exit); Q-09 states that it does not block M0 work but constrains acceptance, and lists every location the owner's answer would change (§1.3, §1.5, §4, §13); T-M2-02 marked PROVISIONAL on the unrecorded A-13 non-combat rule with no assumed policy allowed (T-M2-02, M2 exit); Q-06 confirmed as covering Command Bar commands, temporary scripts, and repository scaffolding (§7, §8.2, §13, M2 exit) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 012 corrective pass: `Requirement` field added to all nine M1 and all twelve M2 test cases, each citing a `ROADMAP.md` §4 criterion or a defined stage statement (T-M1-01…T-M1-09, T-M2-01…T-M2-12); T-M2-04 marked PROVISIONAL for its partial-hit half with `COMBAT_AND_DAMAGE.md` named as the dependency, and the M2 exit rule now requires that section (§6, M2 exit); T-M2-11 marked PROVISIONAL on Q-06/A-12 with the dependency added to §7, the §13 Q-06 row, and M2 exit; T-M1-03, T-M1-04, and T-M1-05 marked PROVISIONAL on the unrecorded A-13 rule so M1 cannot record kill-volume evidence under an assumed policy (T-M1-03…T-M1-05, M1 exit); M1 exit no longer reads as permission to skip the other M1 checks (§5); the M0/Q-09 rule is separated from the generic NOT APPLICABLE vocabulary row (§1.5); Q-09 amendment scope corrected to the M0 device-check paragraph and M0 exit rule with T-M0-08 excluded (§13); Q-08 scope reconciled with §1.4 and the §10.1 template (§13) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 015: Gate G1 owner approval recorded — `TESTING.md` approved on 2026-10-27 with `AGENTS.md`, `TASK_OWNERSHIP.md`, `ROADMAP.md`, and `ARCHITECTURE.md`, with `README.md` informational and outside the approval (status line, §13 Q-01). No test level, status, or requirement changed: Q-09, Q-02, D-16, A-13, D-13, and Q-06 remain pending, T-M0-08 remains PROVISIONAL and recorded `BLOCKED`, and Task 013 Minor findings MN-1…MN-3 remain open with their recorded timings | Approved at G1, 2026-10-27 |
