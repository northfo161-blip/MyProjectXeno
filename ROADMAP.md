# ROADMAP.md — Project Xeno Development Roadmap

**Status:** approved at Gate G1 by the project owner on 2026-10-27, together with `AGENTS.md`,
`TASK_OWNERSHIP.md`, `TESTING.md`, and `ARCHITECTURE.md`. The stages below are the approved plan;
`README.md` is informational and outside the approval.

This roadmap defines the staged path from an empty repository to a first playable version of
Project Xeno on Roblox. It follows the rules in `AGENTS.md` and the task process in
`TASK_OWNERSHIP.md`; it does not replace them.

---

## 1. Goal and boundaries of the first version

### 1.1 Goal

Produce a small, server-authoritative multiplayer arena prototype in which players can:

1. spawn into a blockout arena;
2. fight with one attack each;
3. take locational damage to four body parts;
4. lose the function of a destroyed part;
5. keep fighting from the ground after losing their legs;
6. be eliminated, return to a hub, assemble a new fighter, and re-enter combat;
7. earn points and see a leaderboard;
8. do all of the above on PC, Android, and iOS.

The first version is a **vertical slice**: the complete loop, thin in content. It is not a
content-complete game.

### 1.2 Boundaries and standing constraints

| Constraint | Source |
| --- | --- |
| Roblox Studio only; Luau only; no Unity, Unreal, or C# | `AGENTS.md` §1, `README.md` §2 |
| Devices: PC, Android, iOS | `AGENTS.md` §1, `README.md` §2 |
| Standard Roblox Humanoid rig in the early prototype | `AGENTS.md` §5 |
| Body parts are **disabled**, never physically deleted or detached | `AGENTS.md` §5 |
| Server is the sole authority for health, damage, points, and match state | `AGENTS.md` §6.1 |
| One implementation task in flight; explicit handoff; no shared file writes | `TASK_OWNERSHIP.md` §1, §3 |
| No feature is complete without recorded test evidence | `AGENTS.md` §9, §12 |
| Smallest playable prototype before content expansion | `README.md` §7, `AGENTS.md` §5 |

### 1.3 Explicitly outside the first version

These are deferred, not cancelled. None of them may leak into the early stages:

- rolling-ball starting form;
- part collection, drops, or acquisition loops;
- persistence (DataStore), inventories, unlocks, cross-session progression;
- shops, economy, monetization;
- custom non-Humanoid character controller;
- physical limb detachment or ragdoll limb loss;
- teams, matchmaking, ranked play;
- multiple maps, cosmetics, battle passes, quests;
- vehicles, AI bots (unless an owner-approved test aid).

---

## 2. How stages work

A **stage** and a **milestone** are the same thing in this roadmap: a numbered delivery phase
`M0`–`M9`. A stage is closed at **Gate G6** (`AGENTS.md` §11); a single task is accepted or
rejected at **Gate G8**. Terms and identifier namespaces are defined in `AGENTS.md` §7.2.

- Every stage is a **milestone** closed at **Gate G6** (`AGENTS.md` §11); a stage is only
  complete when the owner accepts its evidence.
- Work inside a stage is split into numbered tasks (`C-###`) defined by OpenCode and approved
  at **Gate G4** using the format in `TASK_OWNERSHIP.md` §6. Individual tasks are accepted or
  rejected at **Gate G8**; G8 never covers a milestone, and G6 never covers a single task.
- Each stage ends with a checkpoint commit and an annotated tag (for example `m0-scaffold`),
  per `AGENTS.md` §10 and `TASK_OWNERSHIP.md` §7.
- Test evidence follows the levels in `AGENTS.md` §9.1: L0 static, L1 Studio Play solo,
  L2 Studio local server with 2+ clients, L3 Device Emulator, L4 published playtest. **L4 is
  required only for a stage that names it below and before any release-readiness claim.**
  M0–M2 do not require publication. Whether that rule excludes M0 is the pending owner
  question Q-09; the M0 wording below records the derived rule and is not an approved owner
  decision (see §4 M0 Device checks and `AGENTS.md` §14).
- A failed required test blocks stage acceptance; partial work is preserved only as a
  documented `wip` commit (`AGENTS.md` §9.4).
- A design or architecture document that a stage depends on must be approved at Gate G1
  **before** any implementation task for that stage starts. If a document does not exist
  yet, that stage cannot start.
- Stages are sequential. Parallel work is allowed only in disjoint files and only with
  owner approval (`TASK_OWNERSHIP.md` §3).

### 2.1 Document prerequisites by stage

| Stage | Documents that must be approved first |
| --- | --- |
| M0 | The G1 package: `AGENTS.md`, `TASK_OWNERSHIP.md`, this roadmap, `TESTING.md`, `ARCHITECTURE.md` |
| M1 | `ARCHITECTURE.md`, `TESTING.md` |
| M2 | `NETWORKING.md`, `COMBAT_AND_DAMAGE.md` (overall-health subsection and partial-hit rules) |
| M3 | `GAME_DESIGN.md` v0.1 (scoring and attribution rules) |
| M4 | `COMBAT_AND_DAMAGE.md`, `NETWORKING.md` |
| M5 | `GAME_DESIGN.md` (ability set), `COMBAT_AND_DAMAGE.md` |
| M6 | `COMBAT_AND_DAMAGE.md` (grounded mode), `GAME_DESIGN.md` |
| M7 | `REASSEMBLY.md`, `GAME_DESIGN.md` (hub and assembly rules) |
| M8 | `UI_AND_CONTROLS.md` |
| M9 | `TESTING.md` (complete) |

`ROADMAP.md`, `TESTING.md`, and `ARCHITECTURE.md` were prepared in Tasks 002–004 and were
approved at Gate G1 on 2026-10-27 together with `AGENTS.md` and `TASK_OWNERSHIP.md`; the
approval is recorded in `AGENTS.md` §14. `GAME_DESIGN.md`,
`NETWORKING.md`, `COMBAT_AND_DAMAGE.md`, `REASSEMBLY.md`, and `UI_AND_CONTROLS.md` do not
exist yet. Creating any document requires explicit owner authorization; approval after
drafting is Gate G1, and creation is not.

### 2.2 Changing approved stage criteria

This procedure was approved with the G1 package on 2026-10-27. It concerns the stage
criteria in this roadmap and the tests and dependencies that follow from them; it does not
redefine Gate G5 and is not a route for adding files to a task. G5 remains the task-amendment
gate: a change that affects an already-approved task spec still requires G5 in addition, and a
task that needs files outside its allowed list still stops and requests an amendment, whether
or not any stage criterion changes.

1. An implementer or other agent may **not** change an approved stage criterion, test, or
   dependency on its own, even to make work pass.
2. The proposed change is written down: the exact current wording, the proposed wording, the
   reason, and the affected tasks, tests, and dependencies.
3. The project owner explicitly approves the change. Silence is not approval.
4. The approval is recorded in the decision log (`AGENTS.md` §14) and in this roadmap's change
   log; related tests and dependencies are updated in the same coherent change.
5. Two separate triggers exist for Gate G5, and this procedure removes neither. G5 is required
   when (a) the change actually alters an approved task spec — the amended spec is submitted
   for owner approval at G5 — or (b) a task needs files outside its allowed list, which needs
   an amended spec and owner approval under `AGENTS.md` §11 even when no requirement, criterion,
   or test changes. A task whose specification wording is unchanged and whose allowed files are
   sufficient needs no G5 approval and no new G5 record. Either way, every affected test is
   re-run; re-running a test is a testing action, not a gate.
6. A criterion change never converts a previously failed or skipped test into a pass.

---

## 3. Stage overview

| Stage | Name | Player-visible outcome | Depends on | Checkpoint tag |
| --- | --- | --- | --- | --- |
| M0 | Toolchain and project skeleton | Nothing playable; files build into a Studio place | — | `m0-scaffold` |
| M1 | Arena, spawning, respawn | Players appear, move, and come back after dying | M0 | `m1-arena` |
| M2 | Server-authoritative combat and health | Players can damage and eliminate each other | M1 | `m2-combat` |
| M3 | Scoring and session leaderboard | Eliminations award visible points | M2 | `m3-scoring` |
| M4 | Body-part damage model | Limbs take damage independently; destroyed parts stop working | M3 | `m4-parts` |
| M5 | Minimal four-slot ability set | Each body slot provides one usable ability | M4 | `m5-abilities` |
| M6 | Legs-lost ground mode | A legless fighter can still fight, degraded | M5 | `m6-ground` |
| M7 | Hub, assembly, re-entry | Full defeat → rebuild → re-enter loop | M3, M6 | `m7-hub` |
| M8 | Mobile controls and UI adaptation | The full loop is playable on phone/tablet | M5–M7 | `m8-mobile` |
| M9 | Integration, hardening, release prep | Stable first version with measured evidence | M8 | `m9-rc` |

### 3.1 The minimum viable prototype

**The nearest stage is M0; the minimum viable prototype is M0–M2.**

M0–M2 is the smallest build that proves the core technical idea: two players in one Roblox
server, server-validated combat, death, and respawn. M3–M9 extend that proven base.

The minimum viable prototype is **not** expected to include body parts, abilities, hub,
assembly, mobile-specific UI, or scoring. Those are deliberately absent so that the
networking and authority model is validated before content is layered on it.

---

## 4. Stages in detail

### M0 — Toolchain and project skeleton

**Goal.** Establish a reproducible, reviewable path from repository files into Roblox Studio
so that all later work is diffable and agent-safe.

**Concrete verifiable result.** The approved M0 task spec must define a toolchain and prove two
**separate** capabilities:

1. a reproducible **artifact build** that produces an openable place file containing the
   agreed folder tree (`ReplicatedStorage.Shared`, `ServerScriptService.Server`,
   `StarterPlayerScripts.Client`);
2. a **live-sync** path that makes a file edited on disk appear in Studio without manual
   copying.

No specific tool is selected by this roadmap. **Rojo is a candidate only**, not a selected,
installed, pinned, or verified tool. The tool, version, pin file, build command, artifact
path, Studio connection method, and the distinction between the artifact build and live sync
must be defined and approved in the M0 task spec and at Gate G2 before installation or use.
No gameplay logic exists yet.

**Completion criteria.**
- Toolchain versions are pinned in the repository exactly as the approved M0 task spec
  defines; the owner has performed or authorized the install at Gate G2.
- The artifact build command defined in the approved M0 task spec completes without error, and
  the output place opens in Studio from the documented artifact path.
- Live sync, using the mechanism defined in the approved M0 task spec, makes a disk edit
  appear in Studio without manual copying. This is checked separately: a successful build does
  not prove live sync, and a successful live sync does not prove a reproducible build.
- Studio Output shows no errors from project scripts on Play.
- Beginner-level setup steps for the build and the live-sync procedure are written into the
  approved M0 task spec itself and confirmed by the owner as repeatable by a beginner. The
  separate `docs/setup.md` guide is a later approved task and is **not** an M0 completion
  criterion.
- No gameplay code, no gameplay remotes, no arena.

**Dependencies.** G1 package approval, recorded 2026-10-27; confirmation of the D-16 place and
asset policy is a separate decision, still pending and recorded in `AGENTS.md` §14 when made;
Gate G2 for tooling; an approved M0 task spec (Gate G4) that names the tool, version, pin file,
commands, artifact path, and the beginner setup steps.
The separate `docs/setup.md` guide is not an M0 dependency and is not created by the M0 task.
The pending Q-09 question does not block M0 work; see Device checks below for what M0
acceptance must record.

**Device checks.** PC/Studio only. Mobile checks are not applicable. **L4 is NOT APPLICABLE to
M0** because M0 has no playable build and is not published; this is not a passed check and
never counts toward acceptance. The rule is **derived, not approved**: it follows from
`AGENTS.md` §9.1 and is tracked as the pending owner decision Q-09 (`AGENTS.md` §14,
`TESTING.md` §13).

Pending Q-09 does not block M0 from starting, being implemented, or being tested, because M0
produces no playable or published build and therefore has no L4 check to run either way. It
does mean M0 may not be accepted as compliant with an *approved* L4 rule: the Gate G6
acceptance record must state that Q-09 was still pending and that L4 was recorded
`NOT APPLICABLE` under the derived rule, or it must record the owner's confirmation of that
rule. If the owner decides that L4 does apply to M0, this paragraph, the M0 exit rule in
`TESTING.md` §4, and the Q-09 row and note in `TESTING.md` §13 are amended first through the
stage-criteria procedure in §2.2, and the added L4 check must then be run before M0 is
accepted. Accepting M0 without that amendment would treat an unconfirmed rule as an approved
requirement.

**Deliberately out of scope.** Arena geometry, characters, spawning, combat, UI, assets
beyond the default place.

---

### M1 — Arena, spawning, and respawn

**Goal.** Prove that multiple players reliably enter a shared blockout arena, move, die, and
return — with no custom combat yet.

**Concrete verifiable result.** A single blockout arena with spawn pads, out-of-bounds
handling, controlled player spawning, and automatic respawn after death.

**Completion criteria.**
- Two clients in Studio local server (L2) spawn at valid, non-overlapping positions.
- Falling out of the arena or touching a kill volume eliminates the player and respawns them
  after a fixed, configurable delay.
- Ten consecutive respawn cycles complete without script errors or stuck characters.
- Player movement and camera behave normally for both PC clients at L2 (T-M1-02), and the
  default mobile touch controls remain usable at L3 in the Device Emulator (T-M1-07).
- Spawn positions and respawn delay come from shared configuration, not hard-coded values.

**Dependencies.** M0 accepted at Gate G6. `ARCHITECTURE.md` approved, including the
spawn-control proposal (A-03) and the non-combat damage rule (A-13). A-13 is a real
dependency, not a formality: the out-of-bounds and kill-volume criterion above cannot be
implemented or tested until a recorded rule says how non-combat damage is handled. No owner
decision on A-13 exists yet — it is an open question in `ARCHITECTURE.md` §10.3, not an
approved rule — so M1 cannot rely on an assumed damage policy and must not be accepted on one.

**Device checks.** L2 on PC; L3 smoke check in Device Emulator to confirm default touch
movement still works.

**Deliberately out of scope.** Any attack, health beyond Roblox defaults, scoring, body parts,
hub, custom UI.

---

### M2 — Server-authoritative combat and overall health

**Goal.** Prove the networking model: clients send intents, the server decides hits, damage,
death, and respawn.

**Concrete verifiable result.** One placeholder attack (right-arm slot) that can damage another
player only when the server validates the shot, one authoritative overall health pool,
combat elimination when that pool reaches zero, and respawn through the M1 flow. M2 decides
only the **carrier** of overall health; the relationship between overall and modular health is
D-01 and is not decided until M4.

**Completion criteria.**
- Two clients (L2) can damage each other; health decreases only through server-side
  resolution.
- A hit behind cover or beyond maximum range deals no damage. The server resolves the attack
  from its own view of the shooter's origin; a client-supplied origin is not part of the
  protocol.
- The attack payload carries no damage or outcome field. Extra or unknown fields cannot change
  damage, health, or elimination.
- Firing faster than the configured cooldown is throttled server-side.
- A burst of malformed or high-frequency remote calls produces no server errors and does not
  degrade other players.
- Combat elimination occurs only when the server-resolved overall health reaches zero and
  routes through the M1 respawn path. The M1 out-of-bounds/kill-volume death is a non-combat
  cause that uses the same path; it may be implemented, tested, or accepted only under the
  non-combat damage rule recorded for A-13 (an M1 prerequisite), never under an assumed one.

**Dependencies.** M1 accepted at Gate G6. Before any M2 implementation work starts:

- `ARCHITECTURE.md` approved at Gate G1. That approval covers the architecture text itself; it
  does **not** approve a decision that another document or the owner owns. Confirmations are
  separate:
  - A-04 (overall-health carrier) is confirmed by the M2 overall-health section of
    `COMBAT_AND_DAMAGE.md` (`ARCHITECTURE.md` §5.4, §10.2).
  - A-05 (M2 remote surface) is confirmed by `NETWORKING.md` (`ARCHITECTURE.md` §6.3, §6.10).
  - A-07 (single damage interface) is defined in `ARCHITECTURE.md` §7.3 and needs no other
    document for M2; its M4 widening depends on the owner decision D-01, which M2 does not
    require.
  - A-08 (PC and touch produce the same intent) is confirmed by the owner's D-13 camera
    decision; no other document can confirm it.
- `NETWORKING.md` and the overall-health section of `COMBAT_AND_DAMAGE.md` approved,
  including the M2 health-carrier decision (A-04).
- The partial-hit rules that the second half of T-M2-04 compares against, in
  `COMBAT_AND_DAMAGE.md`, approved. That document does not exist yet and no partial-hit rule is
  recorded anywhere, so that half of T-M2-04 stays PROVISIONAL until the rules are approved.
  This adds no new M2 scope: T-M2-04 already requires cover rejection.
- D-13, the camera model, decided by the owner and recorded in the decision log
  (`AGENTS.md` §14; §6 below is updated in the same change). D-13 is a prerequisite because
  A-08 cannot be confirmed while the camera model is open. T-M2-12 checks only that the
  temporary attack control is reachable on touch; it does not test camera or aiming behaviour.
  Verifying camera or aiming in M2 would be a new requirement added through the stage-criteria
  procedure (§2.2) and the owner's Gate G5 amendment, not a redefinition of T-M2-12.

One further decision blocks only part of the stage: Q-06 / A-12 (where temporary test
scaffolding may live) must be decided before the crafted-payload checks T-M2-07, T-M2-08, and
T-M2-09 are executed, and before T-M2-11, whose steps run those three checks, is executed. The
rest of M2 may proceed while it is pending, but M2 cannot be accepted until those three checks
and T-M2-11 have run.

D-01 (the relationship between overall and modular health) is not required until M4.

**Device checks.** L2 on PC with two clients. One L3 smoke check that the temporary attack
input is reachable on touch. L4 is not required for M2.

**Deliberately out of scope.** Per-part health, multiple abilities, scoring, leaderboard,
final VFX/audio, lag compensation.

---

### M3 — Elimination scoring and session leaderboard

**Goal.** Make eliminations meaningful: points are awarded by the server and displayed.

**Concrete verifiable result.** An elimination awards points to the correct player according
to an approved attribution rule, and a session leaderboard displays the current ranking. The
persistence scope of points and the leaderboard is an open decision (D-15) and is not assumed
here.

**Completion criteria.**
- Three clients (L2): an elimination increments exactly one player's score by the configured
  amount.
- Attribution matches the approved rule (see D-04) in last-hit and multi-attacker cases.
- The leaderboard reflects server state within a short, documented delay.
- A client cannot increase its own score by calling remotes directly.
- The leaderboard and score display remain legible at phone scale (L3).

**Dependencies.** M2. `GAME_DESIGN.md` v0.1 approved, specifically scoring and attribution.

**Device checks.** L2 on PC; L3 for leaderboard legibility.

**Deliberately out of scope.** Persistence, rewards, unlocks, match rounds, team scoring,
anti-boosting rules beyond basic validation.

---

### M4 — Body-part damage model and function gating

**Goal.** Replace the single health pool's damage routing with four independently damageable
body parts, and make a destroyed part stop providing its function.

**Concrete verifiable result.** Each hit is attributed to a body part; that part's health
drops; at zero the part is marked disabled, replicated to clients, and its ability is blocked.
The Humanoid rig remains intact — nothing is deleted or detached.

**Completion criteria.**
- The four slots (right arm, left arm, torso, legs) each carry independent health and a
  disabled flag, configured centrally.
- A hit on a limb reduces only that limb's health; hitboxes map to the intended slot.
- When the right arm is disabled, the M2 attack is unavailable; re-enabling is not possible
  without an approved repair rule (see D-06).
- Disabling one part does not disable the others.
- State is replicated to all clients and resets correctly on respawn.
- No character part is destroyed, detached, or renamed; the Humanoid continues to function.
- A debug overlay shows part health and disabled state for testing.

**Dependencies.** M3. `COMBAT_AND_DAMAGE.md` and `NETWORKING.md` approved; owner answers D-01
(the relationship between overall and modular health), D-02, D-05.

**Device checks.** L2 on PC. Replication volume is observed in the Studio performance stats
as a first mobile-risk signal; no L3 requirement yet.

**Deliberately out of scope.** The three remaining abilities, ground mode, part visuals
polish, repair, part swapping.

---

### M5 — Minimal four-slot ability set

**Goal.** Give every body slot a usable ability so that disablement is meaningful for all
four, not just the attack arm.

**Concrete verifiable result.** Left arm provides a defensive ability, torso provides a
special ability, legs provide a movement ability, and each is gated by its part's state and by
a server-enforced cooldown.

**Completion criteria.**
- All four abilities activate from client intent and resolve on the server.
- Each ability is unavailable while its part is disabled, and becomes unavailable immediately
  if the part is disabled mid-cooldown.
- Cooldowns and resource costs are enforced server-side and displayed in the HUD.
- Repeated or malformed activation requests cannot bypass cooldowns or trigger duplicate
  effects.
- Ability parameters live in shared configuration, not in scattered literals.

**Dependencies.** M4. `GAME_DESIGN.md` ability definitions approved (D-03); relevant sections
of `COMBAT_AND_DAMAGE.md` approved.

**Device checks.** L2 on PC. HUD must remain usable at phone scale (L3 check), even though the
full mobile pass comes later.

**Deliberately out of scope.** Ability variety, loadout choice, upgrades, final art and audio.

---

### M6 — Legs-lost ground mode

**Goal.** Losing the legs degrades the fighter without removing them from the fight.

**Concrete verifiable result.** When the legs are disabled, the player switches to a grounded
movement state and can still use the surviving abilities.

**Completion criteria.**
- Legs disabled → movement changes immediately to the approved grounded behaviour (D-07).
- The player can still attack, defend, and use the torso ability while grounded.
- The camera remains controllable and the HUD clearly indicates the grounded state.
- Movement cannot be restored by client input, respawn tricks, or reconnecting.
- Respawn restores the normal state.
- The rig never enters Roblox's default death state because of the leg disable.

**Dependencies.** M5. Grounded-mode rules in `COMBAT_AND_DAMAGE.md` approved; D-07 answered.

**Device checks.** L2 on PC. One L3 check that grounded movement is controllable with touch.

**Deliberately out of scope.** Crawling animations, physical leg removal, ball form, custom
character controller.

---

### M7 — Hub, assembly, and re-entry

**Goal.** Close the loop: eliminated players return to a hub, assemble a valid fighter, and
re-enter the arena.

**Concrete verifiable result.** Elimination sends the player to the hub rather than directly
back to the arena. A valid loadout must be confirmed before entering the arena.

**Completion criteria.**
- An eliminated player returns to the hub, not the arena.
- Assembly presents the currently available part pool (starter pool only, per D-08) and
  validates the selection on the server.
- Entering the arena without a valid, complete loadout is impossible, including via direct
  remote calls.
- Re-entry works for at least five consecutive defeat cycles without duplication, stuck
  states, or score corruption.
- Spawn protection on entry follows the approved rule (D-09).
- Disconnecting mid-assembly and rejoining leaves the player in a consistent state.

**Dependencies.** M3 and M6. `REASSEMBLY.md` approved; D-08, D-09, D-10 answered.

**Device checks.** L2 on PC with two clients. Assembly UI is usable on touch at a basic level;
the full mobile pass remains M8.

**Deliberately out of scope.** Part collection, ball form, inventories, persistence, economy,
hub social features.

---

### M8 — Mobile controls and UI adaptation

**Goal.** Make the complete loop genuinely playable on Android and iOS, not merely loadable.

**Concrete verifiable result.** A touch layout that exposes all four abilities, the assembly
actions, and the leaderboard without fighting Roblox's default controls.

**Completion criteria.**
- Dedicated, documented touch controls exist for all four abilities and for assembly
  confirmation.
- Controls do not overlap the default thumbstick, jump, or camera area; safe areas are
  respected on notched devices.
- The HUD scales correctly across phone and tablet profiles in the Device Emulator.
- The full loop (fight → defeat → hub → assemble → re-enter) is completable using touch only.
- Keyboard and mouse remain fully functional.
- No UI element requires hover or a right-click.

**Dependencies.** M5, M6, M7. `UI_AND_CONTROLS.md` approved.

**Device checks.** L3 on at least one phone and one tablet profile; owner performs or
authorizes an L4 device check before the stage is accepted.

**Deliberately out of scope.** Gamepad support, localization, accessibility options beyond
legibility, cosmetic UI polish.

---

### M9 — Integration testing, hardening, release preparation

**Goal.** Turn the prototype into a stable first version with evidence instead of claims.

**Concrete verifiable result.** A documented test run of the full loop with multiple players,
an exploit checklist, measured performance, and a release checklist.

**Completion criteria.**
- The complete `TESTING.md` regression list passes at L0–L3, with results recorded.
- An adversarial pass covers: invalid remote payloads, rate-limit abuse, damage injection,
  score injection, loadout bypass, and movement sanity checks.
- Performance is measured and recorded: server heartbeat under load, client frame time in
  Studio and on an emulated device, and effect and part counts. Numeric performance budgets
  are defined and agreed with the owner at M9 planning, before results are judged against them.
- A multi-player session at the approved player count runs for a documented duration without
  critical errors (L4 private-server playtest).
- Known issues are listed with severity; no known critical exploit remains open.
- The owner accepts the milestone at Gate G6; publication itself requires Gate G7.

**Dependencies.** M8 and a complete `TESTING.md`.

**Device checks.** L3 plus L4 on real devices.

**Deliberately out of scope.** Launch marketing, monetization, live operations, content
updates beyond the first version.

---

## 5. Risks and technical unknowns to verify in Roblox Studio

Each item below is an assumption until it is tested in Studio. None may be treated as fact in
a task spec.

| ID | Risk or unknown | Why it matters | How to verify | Earliest stage |
| --- | --- | --- | --- | --- |
| R-01 | Candidate build/sync tool ↔ Studio sync behaviour with this Studio version and Windows setup (Rojo is one candidate) | If sync is unreliable, every later stage inherits a broken workflow | M0 acceptance test: disk edit appears in Studio; build opens cleanly; tool selected only at G2 | M0 |
| R-02 | Humanoid behaviour when a limb is *disabled* rather than deleted | Roblox may react to limb state changes in ways that affect death or replication | Deliberate Studio experiments on a test rig before M4 | M0–M1 (spike), M4 |
| R-03 | Roblox's own death rules conflicting with custom overall health | Character may die from engine causes unrelated to the game's health model | Test `Humanoid.Health` overrides and death handling in M2 | M2 |
| R-04 | Hit detection accuracy against Humanoid limb hitboxes | Locational damage depends on reliable per-limb hit attribution | Scripted raycast tests from fixed positions in M4 | M4 |
| R-05 | Replication cost of per-player part state | Four parts × N players may be expensive on mobile | Measure network and CPU stats with several simulated players | M4 |
| R-06 | Client-authoritative movement in Roblox | Speed and teleport exploits remain possible even with validated hits | Adversarial movement tests in M9; document residual risk | M2, M9 |
| R-07 | Remote spam resistance and rate-limiting overhead | Naive throttling can itself become a server cost | Structured burst tests in M2 and M9 | M2 |
| R-08 | Mobile performance with modular characters and effects | The game may be unplayable on lower-end phones | Device Emulator measurements; L4 real-device check | M5, M8, M9 |
| R-09 | Touch control layout colliding with default Roblox controls | Abilities may be unreachable or accidentally triggered | Layout tests on phone/tablet profiles | M8 |
| R-10 | `CharacterAutoLoads` / respawn control interacting with hub flow | Players may respawn into the arena or get stuck in the hub | Repetition test of the full loop in M7 | M7 |
| R-11 | Assembly state across disconnect/rejoin | Partial state may duplicate fighters or lose loadouts | Rejoin tests in M7 | M7 |
| R-12 | Studio Device Emulator fidelity vs real hardware | Emulator success may not reflect real devices | Owner-run L4 playtest before release claims | M9 |
| R-13 | Number of test clients available locally | Multiplayer behaviour may be under-tested | Plan L2 tests with at least two clients; owner confirms capacity | M1 |

---

## 6. Provisional decisions requiring owner approval

These are **not** decided. Each blocks the stage listed; none may be resolved by an agent
alone (`AGENTS.md` §4, §7). The exception is D-16, where a proposed policy is written out
below for the owner to confirm; it is still not approved until the owner records it.

**Recording rule.** The decision log in `AGENTS.md` §14 is the single authoritative record of
an owner decision. When the owner decides any item below, a row is added or updated there
first, and the table row in this section plus this document's change log follow in the same
change. This table is a working list, not the record; a tick or a note here that the decision
log does not contain is not a decision.

**Proposed place and asset policy for M0–M2 (D-16) — awaiting owner confirmation; not part of the
2026-10-27 G1 approval:**

- M0–M2 are developed and tested locally; the game is not published.
- Third-party assets with unclear rights are not used.
- Project source text files and necessary configuration may be committed to Git.
- Generated builds and local data are excluded from Git by default unless a separate
  justification is approved.
- No place file, build artifact, or third-party resource is added to the repository as part of
  this policy proposal.

This paragraph records a proposal, not an approved policy. It does not mark D-16 as decided and
does not permit publication (Gate G7 remains separate). Subject to that record, the
PROVISIONAL→FINAL rule for T-M0-08 in `TESTING.md` §4 governs when the test may be executed as
a decided check.

| ID | Question | Blocks | Notes |
| --- | --- | --- | --- |
| D-01 | Is overall health separate from part health, or derived from it? | M4 | Relationship only. M2 decides the carrier of the single overall pool; D-01 decides how modular health relates to it later |
| D-02 | Do disabled parts repair during a life, only on respawn, or never? | M4 | Affects pacing and the value of disengaging |
| D-03 | What are the concrete v1 abilities for left arm, torso, and legs? | M5 | Right arm attack is the only ability confirmed by concept |
| D-04 | Elimination attribution: last hit, damage contribution, or other? | M3 | Required before scoring can be implemented |
| D-05 | Are destroyed parts merely disabled, or do they also change hitbox/mass visually? | M4 | Prototype constraint forbids deletion; visual rules still open |
| D-06 | Can a player replace or repair a lost part mid-match? | M4 | Interacts with part acquisition (deferred) |
| D-07 | What movement, if any, does a legless fighter retain? | M6 | Core to the grounded-mode feel |
| D-08 | Which starter part pool is available in the hub for v1? | M7 | Acquisition is deferred, so the pool must be fixed for now |
| D-09 | Spawn protection: duration, behaviour, and exceptions | M7 | Prevents instant re-elimination |
| D-10 | Hub model: shared social hub or per-player instance? | M7 | Affects teleport, replication, and performance |
| D-11 | Match structure: continuous free-for-all or rounds? | M3 | Affects scoring, leaderboard reset, and respawn design |
| D-12 | Target player count per server for v1 | M9 (deferred) | Two clients are the M0–M2 test-session size and the minimum network check; this is not a server cap or architectural limit. Not an M0 blocker |
| D-13 | Camera model: third-person over-shoulder or other? | M2, M8 | Affects aiming and mobile layout. Required for A-08; recorded in `AGENTS.md` §14 when decided |
| D-14 | Is the rolling-ball form part of v1 at all, or a later milestone? | — | Explicitly deferred by current constraints |
| D-15 | Does v1 include persistence of points, or session-only leaderboard? | M3, M9 | Persistence is currently out of scope; confirm |
| D-16 | Place and asset policy: single place, and whether built places are committed | M0 | Proposed policy above; confirmation pending and recorded in `AGENTS.md` §14 when decided |
| D-17 | Publication timing and audience for the first test build | M9, G7 | Gate G7 requires owner action |

---

## 7. Nearest actions

1. **Done 2026-10-27:** the owner approved the G1 package — `AGENTS.md`, `TASK_OWNERSHIP.md`,
   this roadmap, `TESTING.md`, and `ARCHITECTURE.md` — recorded in `AGENTS.md` §14. `README.md`
   is informational and is not part of the package.
2. OpenCode writes the M0 task spec, naming the candidate tool, version, pin file, build
   command, artifact path, Studio connection, and the separate build and live-sync checks.
3. Owner approves the M0 task spec (Gate G4) and then the tooling at Gate G2.
4. Cline executes M0 only. No arena, combat, or character work happens in M0.
5. Owner confirms the D-16 place and asset policy; it was not part of the G1 approval, so the
   policy above remains a proposal until that decision is recorded.

M1 does not start until M0 is accepted at Gate G6.

---

## 8. Success criteria for the first version

The first version is successful when, with recorded evidence:

- two or more players complete the full loop repeatedly on PC and on at least one emulated
  phone profile;
- combat, damage, disabling, scoring, and re-entry are server-authoritative;
- no known critical exploit allows damage, score, or loadout manipulation;
- performance is measured and within limits agreed with the owner;
- the owner accepts the milestone at Gate G6 and, separately, authorizes publication at G7.

---

## 9. Change log

| Date | Change | Approved by |
| --- | --- | --- |
| 2026-10-27 | Initial draft created under Task 002 | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 006 reconciliation: G1 package prerequisites and non-circular G1 wording (§2, §2.1, §7); stage-criteria change procedure (§2.2); tool-neutral M0 result with separate build and live-sync checks (§4 M0); M2/M4 health split and approved death causes (§4 M2, §6 D-01); L4 limited to named stages and M0 NOT APPLICABLE; D-12 deferred to M9; proposed D-16 policy; protocol-aligned M2 criteria; numeric budgets deferred to M9 | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 008 reconciliation of review findings: M2 dependencies now list A-04/A-05/A-07/A-08, D-13, and the Q-06/A-12 payload-check condition (§4 M2); M1 dependencies name A-03 and A-13 (§4 M1); M0 setup evidence moved into the approved task spec, `docs/setup.md` removed from M0 criteria and dependencies (§4 M0); G5 limited to actual task-spec changes (§2.2) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 010 correction of Task 009 findings: M2 dependencies now separate G1 approval of `ARCHITECTURE.md` from confirmation of A-04/A-05/A-07/A-08 by their owning documents or the D-13 owner decision, and state that T-M2-12 does not test camera or aiming (§4 M2); "explicitly approved non-combat cause" replaced with wording that the kill-volume death needs the recorded A-13 rule (§4 M1, §4 M2); pending Q-09 stated as not blocking M0 work but as a condition on how M0 acceptance may be recorded, with the amendment path if L4 is found to apply (§2, §4 M0); decision-log recording rule added to §6; G5 wording separates task-spec amendments from the files-outside-allowed-list trigger (§2.2) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 012 corrective pass: the M1 movement/camera criterion now requires L2 with two clients (T-M1-02) plus an L3 touch-control check (T-M1-07), matching M1 Device checks and the architecture mapping (§4 M1); the partial-hit rules of `COMBAT_AND_DAMAGE.md` are named as an M2 document prerequisite so T-M2-04's PROVISIONAL half cannot be bypassed (§2.1, §4 M2); the Q-06/A-12 blocking list now includes the dependent T-M2-11 (§4 M2) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 015: Gate G1 owner approval recorded — this roadmap approved on 2026-10-27 with `AGENTS.md`, `TASK_OWNERSHIP.md`, `TESTING.md`, and `ARCHITECTURE.md`, with `README.md` informational and outside the approval (status line, §2.1, §2.2, §4 M0 dependencies, §6 D-16 notes, §7). No other decision recorded: D-16, Q-09, A-13, D-13, and Q-06 remain pending, and Task 013 Minor findings MN-1…MN-3 remain open | Approved at G1, 2026-10-27 |
