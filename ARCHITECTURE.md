# ARCHITECTURE.md — Prototype Architecture for M0–M2

**Status:** approved at Gate G1 by the project owner on 2026-10-27, together with `AGENTS.md`,
`TASK_OWNERSHIP.md`, `ROADMAP.md`, and `TESTING.md`. The approval covers this document as
written; it does not decide the open questions in §10.3, and the confirmations and owner
decisions named in §9.1 and §10.2 remain separate and are recorded where they are made.

This document defines the smallest architecture that can carry Project Xeno from an empty
repository through **M0–M2** of `ROADMAP.md`: a buildable skeleton, an arena with spawning and
respawn, and server-authoritative combat with overall health.

It is deliberately narrow. M3–M9 (scoring, body-part damage, abilities, ground mode, hub,
mobile pass) appear only as extension points, not as designs.

It does not replace:

| Document | What it still governs |
| --- | --- |
| `AGENTS.md` | Binding rules; §6 covers server authority, remotes, code organization, naming, mobile |
| `TASK_OWNERSHIP.md` | File ownership, task lifecycle, required task format |
| `ROADMAP.md` | Stages M0–M9, risks R-01…R-13, provisional decisions D-01…D-17 |
| `TESTING.md` | Mandatory checks T-M0/T-M1/T-M2, evidence rules, open questions Q-01…Q-09 (Q-09: owner confirmation pending) |
| `NETWORKING.md` (planned) | Final remote names, payload schema, numeric validation limits |
| `COMBAT_AND_DAMAGE.md` (planned) | Final overall-health and later part-health rules |

---

## 1. Scope, facts, and constraints

### 1.1 In scope

1. The client/server split and what the server alone may decide.
2. A proposed repository and Roblox layout for M0–M2.
3. Boot, player join, character spawn, death, and respawn flows.
4. Where attack logic, hit validation, health, and death live.
5. Remote boundaries and validation rules.
6. The extension path to modular body-part damage without physical detachment.
7. Minimum PC and mobile requirements for M1–M2.
8. Traceability from these decisions to `TESTING.md` checks and `ROADMAP.md` stages.

### 1.2 Out of scope

- Full game architecture; M3–M9 designs beyond their extension points.
- Hub, assembly, persistence, inventories, economy, matchmaking, rolling-ball form.
- Custom character controller, ragdoll, physical limb loss.
- Frameworks (Knit, Matter, Flamework), Wally packages, external services.
- Final balance numbers, final remote schema, final camera design.
- Anything that would require a tool beyond what Gate G2 approves.

### 1.3 Confirmed facts versus proposals

| Claim | Status | Source |
| --- | --- | --- |
| The repository contains documentation only: no `src/`, no project mapping file, no place file | Confirmed at time of writing | Repository state; `AGENTS.md` §2 |
| Roblox Studio and Luau are the only runtime | Confirmed | `AGENTS.md` §1 |
| The Luau/Roblox toolchain is not installed or verified | Confirmed at time of writing | `AGENTS.md` §2; `TESTING.md` §8.1 |
| The three container names `ReplicatedStorage.Shared`, `ServerScriptService.Server`, `StarterPlayerScripts.Client` | Required by M0 | `ROADMAP.md` §4, M0 |
| Rojo is a candidate build and sync pipeline, not a selected tool | **Proposal A-01** | Nothing installed or selected; `ROADMAP.md` R-01; `TESTING.md` Q-02 |
| Every module, folder, and file name in §3 | **Proposal** | This document |
| The M2 remote surface | **Proposal A-05** | `NETWORKING.md` not written |
| The M2 overall-health carrier | **Proposal A-04** | `COMBAT_AND_DAMAGE.md` not written |

### 1.4 Binding constraints on this design

| Constraint | Source |
| --- | --- |
| The server alone decides health, damage, eliminations, cooldowns, spawn, and match state | `AGENTS.md` §6.1 |
| Clients send intents, never outcomes | `AGENTS.md` §6.1 |
| Remotes are created by the server and referenced through one shared definition module | `AGENTS.md` §6.2 |
| No gameplay logic in `Script`/`LocalScript` bodies beyond bootstrap | `AGENTS.md` §6.3 |
| Tuning values live in shared configuration, never scattered literals | `AGENTS.md` §6.3 |
| Standard Roblox Humanoid rig in the prototype | `AGENTS.md` §5 |
| Character parts are disabled, never deleted, detached, or renamed | `AGENTS.md` §5, §6.4 |
| Smallest playable multiplayer prototype first; no dependency on unapproved tools | `AGENTS.md` §5; `README.md` §7 |
| Every interactive element must work on PC and touch | `AGENTS.md` §6.5 |
| Any new remote or service requires owner approval at Gate G3 | `AGENTS.md` §11 |

### 1.5 Assumptions that must be verified, not assumed

These are beliefs about Roblox behaviour, listed separately so they are not mistaken for facts:

- Roblox replicates `Instance` attributes from server to clients automatically in a form this
  design can use for part state (M4).
- `Players.CharacterAutoLoads = false` combined with explicit `LoadCharacter()` behaves
  predictably enough for controlled spawning (risk R-10; verify in M1).
- Humanoid characters tolerate visual changes to a limb (transparency, colour, attached
  effects) without entering the engine's death state (risk R-02; spike before M4).
- The installed Studio build exposes the safe-area and touch-button properties named in §8;
  their exact behaviour must be confirmed manually rather than assumed.
- Two Studio clients are available for L2 runs (risk R-13; owner confirms capacity).

Each assumption is a test item, not a design authority. If one is wrong, this document is
amended only by a change the owner has approved: the change is recorded in this document's
change log (§11), and the owner's decision is recorded in the decision log in `AGENTS.md` §14.
Gate G3 applies when the change meets the governance definition of an architecture change
(`AGENTS.md` §11); Gate G1 remains the approval gate for this document package as a whole. An
assumption is never silently reinterpreted. Gate G5 is the task-amendment gate and is not the
route for editing this document.

---

## 2. Client and server boundary

### 2.1 Plain-language model

Roblox runs one **server** (the authority) and one **client** per player. The server holds the
real game state and replicates it to clients. A `RemoteEvent` is a one-way message that can
cross that boundary; a `ModuleScript` is a reusable Luau file that returns a table of
functions.

One Roblox-specific fact drives this whole design: **the client simulates its own character's
physics**. The client therefore controls where its character appears to be, and no server-side
check can fully undo that. Everything else — health, damage, hits, deaths, scores — can and
must stay server-side.

Consequences:

1. The server never accepts a client statement of an **outcome** ("I hit", "I dealt 40",
   "he died").
2. The server accepts **intents** ("I aimed in this direction and want to attack").
3. The server recomputes everything that matters from its own state and shared configuration.
4. Movement remains client-controlled; the only defence is sanity checking plus documented
   residual risk (risk R-06).

### 2.2 Authority table

| Concern | Authority | Mechanism | Notes |
| --- | --- | --- | --- |
| Player alive/dead state | Server | Server-side player state plus Humanoid | Client view is a replica |
| Overall health value | Server | `HealthService` writes the replicated health value | Proposal A-04 |
| Damage amount | Server | Shared configuration value | Never read from a payload |
| Whether a shot hit | Server | Raycast performed on the server | Client may show a predicted effect only |
| Attack cooldown | Server | Server-side timestamps | Client display is cosmetic |
| Spawn position, respawn timing | Server | `SpawnService` plus arena configuration | Proposal A-03 |
| Part health and disabled flags (M4) | Server | Server-owned state, replicated read-only | Proposal A-06 |
| Score and elimination credit (M3) | Server | Server-side scoring module | `GAME_DESIGN.md` rules |
| Loadout validity (M7) | Server | Server validation before entry | `REASSEMBLY.md` |
| Character position | Client-simulated | Roblox replication | Server sanity checks only; R-06 |
| Camera | Client | Roblox camera | D-13 still open |
| Input mapping (key, mouse, touch) | Client | Input layer | Must produce the same intent |
| Visual effects and audio | Client | Cosmetic only | Never carries state |

The table describes the target boundary, not the current build. A concern exists only from the
stage named in its row or in §3.4; nothing listed here is implemented in M0.

### 2.3 What the client may send

- An attack intent: aim direction and a diagnostic sequence number.
- Later, ability activation intents (M5) and loadout selection (M7), each separately approved.
- Nothing else in M0–M2.

### 2.4 What the client must never send or decide

- Damage values, health values, or "target died".
- Hit confirmation or the identity of the player it hit, as an authoritative claim.
- Cooldown exemptions, ammo or energy balances.
- Score changes, elimination credit, or loadout ownership.
- Part health, disabled flags, or part replacement.
- An arbitrary spawn position or teleport request.
- A claim of being a different player; identity always comes from the engine-provided
  `player` argument of a remote handler, never from the payload.

### 2.5 Thin entry points and state ownership

- `Script` and `LocalScript` bodies contain **only** bootstrap: require modules, call an
  `init`/`start` function, stop. No gameplay branches, no loops, no combat logic.
- Every piece of mutable state has exactly one owning module on the server. Other modules read
  through that module; they do not keep a second copy.
- Clients keep display caches only. A cache may be stale or wrong; the server always wins.

### 2.6 Residual risk

Server authority reduces exploit surface; it does not eliminate it. A modified client can move
faster than intended, teleport within replication limits, and spam remotes. This architecture
keeps such abuse from becoming damage, death, or score, and records the rest as residual risk
(risk R-06). No claim of "exploit-proof" may be made from this document.

---

## 3. Proposed repository and Roblox layout

### 3.1 Build pipeline — proposal A-01

Rojo is a **candidate** build and sync pipeline, not a selected tool: a small JSON project
file maps disk folders into Roblox containers, a build command produces a place file, and a
Studio plugin keeps Studio in sync with disk. Rojo is **not installed, not pinned, and not
verified**, and this document does not select it.

The approved M0 task spec (Gate G4), together with Gate G2, must define the tool, version, pin
file, build command, artifact path, Studio connection method, and the separate handling of
**artifact build** versus **live sync**. Only after that selection and a passing M0 may the
chosen tool be treated as a fact. If the owner selects a different tool, §3.2 and §3.3 are
illustrative and must be replaced by the approved mapping; a replacement pipeline is an owner
decision (Gates G2/G3), not an agent decision.

### 3.2 Proposed mapping

| Disk path (repository-relative) | Roblox container | Instance type | Stage |
| --- | --- | --- | --- |
| Project mapping file (name and tool decided in the approved M0 task spec; Rojo example: `default.project.json`) | — (build definition) | File | M0 |
| `src/shared` | `ReplicatedStorage.Shared` | Folder | M0 |
| `src/server` | `ServerScriptService.Server` | Folder | M0 |
| `src/client` | `StarterPlayerScripts.Client` | Folder | M0 |
| `assets/` | — | Not mapped until policy D-16 | M1+ |
| build output (`build/`, `places/`) | — | Not committed until policy D-16 | M0+ |

The disk paths above are **illustrative of a Rojo-shaped mapping** and become authoritative
only if the owner selects that tool at Gate G2. The three Roblox containers are the required
M0 outcome; how disk files map into them is fixed by the approved M0 task spec.

The three containers are exactly the names `ROADMAP.md` M0 requires. They stay **Folders**:
entry-point scripts live *inside* them. In Rojo, a file named `init.server.luau` inside a mapped
folder would turn the folder itself into a `Script`; that convention must not be used at the
root of `src/server` or `src/client`, or T-M0-02 would fail against the agreed tree. (Rojo's
naming behaviour itself must be confirmed during M0.)

### 3.3 M0 skeleton — what exists after M0

M0 contains **no gameplay**. Proposed minimum:

```
project mapping file              # approved name and format; Rojo example: default.project.json
src/
  shared/
    Placeholder.luau              # ModuleScript with one marker value for the sync test
  server/
    ServerBootstrap.server.luau   # Script: prints one readiness line, nothing else
  client/
    ClientBootstrap.client.luau   # LocalScript: prints one readiness line, nothing else
```

The file names and the mapping file are illustrative; the approved M0 task spec fixes them. The
marker value exists so T-M0-03 can prove disk-to-Studio sync without adding gameplay. T-M0-04
requires that nothing beyond this skeleton exists.

### 3.4 M2 target shape — what M1 and M2 add

This tree is a **proposal** and is not created by M0. Names may change when
`NETWORKING.md`/`COMBAT_AND_DAMAGE.md` are approved; the responsibilities should not.

```
src/
  shared/
    Config/
      Arena.luau                  # M1: spawn points, respawn delay, kill-volume tuning
      Combat.luau                 # M2: max health, damage, range, cooldown
      Network.luau                # M2: rate-limit placeholders (replaced by NETWORKING.md)
    Net/
      RemoteDefinitions.luau      # M2: the single source of remote names and payload shape
    Types/
      BodySlot.luau               # M2: `Overall` only; widened to four slots in M4
  server/
    ServerBootstrap.server.luau   # requires and starts services in order
    Services/
      PlayerLifecycleService.luau # M1: join/leave state, character lifetime
      SpawnService.luau           # M1: spawn point choice, LoadCharacter, respawn scheduling
      RemoteGuard.luau            # M2: arity/type/rate validation shared by handlers
      CombatService.luau          # M2: attack intent, cooldown, ability gating
      HitValidation.luau          # M2: server raycast and target resolution
      HealthService.luau          # M2: authoritative damage application
      DeathService.luau           # M1: one elimination path, calls spawn on respawn
  client/
    ClientBootstrap.client.luau   # requires and starts controllers in order
    Controllers/
      InputController.luau        # M2: PC and touch input -> one intent
      CombatController.luau       # M2: sends intents, shows local prediction only
      HudController.luau          # M2: minimal debug/health display
```

### 3.5 Component responsibilities

| Component | Owns | Must not | Stage |
| --- | --- | --- | --- |
| `ServerBootstrap` | Startup order, one readiness log | Contain gameplay rules | M0/M1 |
| `ClientBootstrap` | Startup order on the client | Decide any outcome | M0/M1 |
| `PlayerLifecycleService` | Per-player state creation/cleanup | Own health or damage values | M1 |
| `SpawnService` | Spawn choice, character loading, respawn timing | Trust a client's requested position | M1 |
| `DeathService` | The single elimination path | Duplicate elimination, handle scoring | M1/M2 |
| `RemoteGuard` | Arity, type, finiteness, rate limits | Contain combat rules | M2 |
| `CombatService` | Attack state machine, cooldown, part gating | Compute geometry or apply health | M2/M4 |
| `HitValidation` | Raycast, range, line of sight, target resolution | Apply damage or trust client hits | M2 |
| `HealthService` | Authoritative health changes | Accept a client-supplied amount | M2 |
| `RemoteDefinitions` | Remote names and payload contract | Create instances | M2 |
| `Config/*` | Tuning values | Contain logic or per-player state | M1/M2 |
| `InputController` | Mapping keys/mouse/touch to intents | Send anything but an intent | M2 |
| `CombatController` | Sending intents, local feedback | Treating local feedback as truth | M2 |
| `HudController` | Displaying replicated state | Writing gameplay state | M2 |

### 3.6 Growth rules

- Add a module only when the current stage needs it; no placeholders "for later" beyond the M0
  skeleton (`AGENTS.md` §6.3).
- `Shared` modules must run on both sides: no `game:GetService` calls that only exist on one
  side, no server-only or client-only API without an explicit comment.
- Configuration holds values, not behaviour. A config change must not require a code change to
  take effect (T-M1-06).
- One service per concern. If two services need the same rule, the rule moves into one of them
  and the other calls it.

### 3.7 Naming

- `PascalCase` ModuleScripts and services (`CombatService`), `*.server.luau` / `*.client.luau`
  entry points (`AGENTS.md` §6.4).
- Domain names: `PartState`, `BodySlot`, `AttackIntent` — not `Thing`, `Data`, `Manager`.
- Never rename, delete, or reparent Roblox rig parts: `HumanoidRootPart`, `Head`, `Torso` /
  `UpperTorso`, arms, legs.

### 3.8 What must never exist

- A remote created by a client that the server listens to.
- A damage value, health value, or kill decision accepted from a payload.
- A `while true do` polling loop where an event or attribute signal exists.
- Tuning literals duplicated across modules.
- Any call to `Destroy()`, `Parent = nil`, or a rename on a character part.
- A third-party framework or Wally dependency without Gate G3 approval.

---

## 4. Boot, join, spawn, and respawn

### 4.1 Server boot sequence

1. `ServerBootstrap` runs (a `Script` inside `ServerScriptService.Server`).
2. It requires shared configuration and fails loudly if a required value is missing.
3. It creates the remotes defined in `RemoteDefinitions` under a folder in
   `ReplicatedStorage`, before any player can join (M2).
4. It starts services in dependency order: lifecycle → spawn → combat → hit validation →
   health → death. Each `start` returns after wiring connections, not after blocking.
5. It applies the spawn policy from proposal A-03 (`Players.CharacterAutoLoads`).
6. It connects `PlayerAdded`, `PlayerRemoving`, and per-character events.
7. It writes exactly one readiness line to Output. No per-frame logging.

### 4.2 Client boot sequence

1. `ClientBootstrap` runs (a `LocalScript` inside `StarterPlayerScripts.Client`).
2. It waits for the shared remote container to replicate (M2); it does not create it.
3. It starts controllers: input → combat → HUD.
4. It performs no gameplay decision and never writes health.

### 4.3 Player join

1. `PlayerAdded` fires on the server.
2. `PlayerLifecycleService` creates one state record for the player, guarded against duplicate
   events.
3. With `CharacterAutoLoads = false` (A-03), the service asks `SpawnService` to load the
   character explicitly; otherwise Roblox loads it and the same `CharacterAdded` path applies.
4. `CharacterAdded` fires; `SpawnService` places the character at a configured spawn point.
5. Per-life state is initialised (health at maximum in M2; part states in M4).
6. The player is marked alive; the server is now the only source of that fact.

### 4.4 Spawn selection (M1)

- Spawn positions come from `Config/Arena.luau`, not from `SpawnLocation` instances scattered
  in the place (proposal; T-M1-01 and T-M1-06 verify behaviour and configurability).
- Selection must avoid identical and overlapping positions for simultaneous joins.
- Whether selection is round-robin, occupancy-checked, or delegated to Roblox's default
  `SpawnLocation` behaviour is an implementation choice inside `SpawnService`, proven by
  T-M1-01. It is not an architectural commitment here.

### 4.5 Death and respawn in M1

M1 has no combat but still needs one death path, so M2 and M4 do not have to rewrite it.

1. A kill volume (a part the server watches) or a fall below the arena reaches the server.
2. The server routes this through `DeathService.eliminate(player, cause)` — it does **not**
   destroy the character and does not rely on an unrelated engine path.
3. `DeathService` marks the player dead, records the cause, and asks `SpawnService` to schedule
   a respawn after the configured delay.
4. The scheduled respawn loads a fresh character and resets per-life state.
5. A player who leaves while dead cancels the scheduled respawn; no orphan task remains.

Ten consecutive cycles (T-M1-04) must not accumulate duplicate characters or connections.

### 4.6 Death and respawn in M2

`HealthService` reduces the authoritative health. At zero it calls the **same**
`DeathService.eliminate` path with cause "combat". Elimination is idempotent: two simultaneous
lethal events produce one death and one respawn schedule.

Combat elimination in M2 happens only when server-resolved overall health reaches zero. The M1
out-of-bounds/kill-volume cause is a non-combat death cause that also uses `eliminate`; it does
not have to reduce health first. Which non-combat causes are allowed and how engine damage is
handled is decision A-13 (§10.3), which has no recorded owner answer, so this paragraph states
the routing only and assumes no damage policy. `TESTING.md` T-M2-02 records the non-combat half
as PROVISIONAL until A-13 is recorded. This preserves the M1 rule without weakening the combat
health path.

### 4.7 Failure handling

- Missing character at respawn time: retry once, then log a single error; do not loop forever.
- Duplicate `CharacterAdded` for one life: ignore the second, keyed by a per-life generation
  counter.
- Remote handler raising an error: caught by `RemoteGuard`; the player is not disconnected and
  the server keeps running.
- Config value missing or invalid: fail at boot with one explicit message, rather than at the
  moment of combat.

### 4.8 Spawn control — proposal A-03

Two workable options exist; this document recommends the first, and the owner must choose
before M1 starts:

| Option | Behaviour | Cost | Benefit |
| --- | --- | --- | --- |
| **A (recommended)** | `Players.CharacterAutoLoads = false`; `SpawnService` calls `LoadCharacter()` explicitly | More code, more failure modes (risk R-10) | One controlled path for join, respawn, and later hub routing (M7) |
| B | Keep automatic loading; set `Players.RespawnTime` from config | Less code | Less control; respawn is engine-driven and harder to route through a hub later |

Neither option is approved. The M1 task spec must name the chosen one.

---

## 5. Combat: where attack, hit validation, health, and death live

M2 implements exactly one placeholder attack, associated with the right-arm slot. The logic is
split so that M4 can add per-part routing without rewriting the flow.

### 5.1 Request path

1. The player presses the attack input (mouse, key, or touch button).
2. `InputController` converts it into the same intent regardless of device.
3. `CombatController` sends an `AttackRequest` with the aim direction and a sequence number.
4. `RemoteGuard` validates identity, arity, types, finiteness, and rate.
5. `CombatService` checks that the player is alive, in the correct phase, and that the right-arm
   slot is enabled (M4).
6. `CombatService` checks the cooldown, then starts it **before** geometry work, so repeated
   forged requests cannot force repeated raycasts.
7. `HitValidation` performs a server raycast from a server-known origin along the validated
   direction, within the configured range.
8. `HitValidation` rejects cover, non-character hits, dead targets, and self-hits as configured.
9. `HealthService` applies the configured damage to the resolved target. No amount from the
   payload is used, because the payload has no such field.
10. If health reaches zero, `DeathService` runs the single elimination path from §4.5.
11. The server sends a cosmetic feedback event; clients may show a hit marker.
12. Replication updates the target's health for all clients.

### 5.2 Concern-to-module map

| Concern | Module | Stage | Authority source |
| --- | --- | --- | --- |
| Input mapping, PC and touch | `InputController` (client) | M2 | Client, cosmetic |
| Intent transmission | `CombatController` (client) | M2 | Client, advisory |
| Rate and shape validation | `RemoteGuard` (server) | M2 | Server |
| Alive/phase/cooldown/part gating | `CombatService` (server) | M2/M4 | Server |
| Raycast, range, line of sight, target | `HitValidation` (server) | M2/M4 | Server |
| Damage amount | `Config/Combat.luau` | M2 | Server-owned data |
| Health mutation, elimination threshold | `HealthService` (server) | M2 | Server |
| Elimination, respawn scheduling | `DeathService` (server) | M1/M2 | Server |
| Part health and disabled flags | Part-state module (M4; proposed, not in the M0–M2 tree; see §7.4) | M4 | Server |
| Score and attribution | `ScoringService` (M3) | M3 | Server |

### 5.3 Validation order

Cheap checks run before expensive ones, so abuse cannot force raycasts:

1. Caller identity (engine-provided, not payload).
2. Argument count and types.
3. Number finiteness and magnitude bounds.
4. Rate limit.
5. Player state: alive, correct phase, not already eliminated.
6. Cooldown.
7. Part gating (M4).
8. Geometry: raycast, range, line of sight.
9. Target validity: character, alive, not self, within rules.
10. Damage application.

A failure at any step ends the request without an error. The order is itself a security
property: rate limiting before raycasting is what keeps T-M2-09 from degrading the server.

### 5.4 Health model — proposal A-04

- The server keeps the authoritative health for each life.
- The replicated carrier proposed is `Humanoid.Health`, written **only** by `HealthService`.
  Maximum health and all damage values come from `Config/Combat.luau`.
- The client never writes health; a local write is either overwritten by replication or has no
  effect on the server's decision (T-M2-10).
- Default Roblox health regeneration must be disabled; fall damage, drowning, and other engine
  damage sources must be disabled or routed through `DeathService`, otherwise a character can die
  for reasons the game did not decide (risk R-03, open question A-13).
- The M2 overall-health section of `COMBAT_AND_DAMAGE.md` decides the **carrier** of this pool
  (Humanoid-based or a separate replicated value) before M2. D-01 — how the overall pool
  relates to modular part health — is an M4 decision and is not required for M2. This document
  fixes only that there is **one** authoritative pool and one place that writes it.

### 5.5 Death handling

- One function, `DeathService.eliminate`, is the only way a player becomes eliminated.
- It is idempotent per life; repeated calls in the same life are ignored.
- It does not award score in M2 (scoring is M3 and needs an attribution rule, D-04).
- It does not detach, delete, or rename any character part.
- It always ends in the M1 respawn flow, so M2 does not introduce a second respawn mechanism.

### 5.6 Explicitly not in M2

Per-part health, four abilities, ground mode, scoring, leaderboard, spawn protection, hub,
lag compensation, custom camera, final VFX and audio. These are deferred by
`ROADMAP.md` §1.3 and M2 scope, not forgotten.

---

## 6. Remote contract and rules

### 6.1 Principles

1. The server creates every remote, in `ReplicatedStorage`, from one shared definition module.
2. Clients may only reference remotes that already exist; they never create or replace them.
3. Payloads carry intents only. Outcome fields are not merely ignored — they are absent from
   the schema.
4. Every handler validates and fails closed.
5. The surface stays as small as the current stage allows.

### 6.2 Creation and access

- `Shared/Net/RemoteDefinitions.luau` is a pure data module: names, instance classes, and the
  documented payload shape. It creates nothing.
- The server instantiates the remotes under one folder (for example
  `ReplicatedStorage.Net`) during boot.
- The client uses `WaitForChild` with a timeout and reports a clear error if the container never
  arrives, instead of silently doing nothing.

### 6.3 Proposed M2 surface — proposal A-05

Names are **provisional** until `NETWORKING.md` is approved.

| Remote (proposed) | Type | Direction | Conceptual payload | Purpose |
| --- | --- | --- | --- | --- |
| `AttackRequest` | `RemoteEvent` | client → server | aim direction (unit `Vector3`), sequence number | Ask to attack |
| `CombatFeedback` | `RemoteEvent` | server → client | feedback kind, position | Cosmetic hit/blocked feedback |

No `RemoteFunction` is used in M0–M2. Reasons in §6.8.

### 6.4 Payload rules

- Fixed arity: one message shape per remote, no "optional third argument" ambiguity.
- No damage, health, kill, cooldown, score, loadout, or part-state fields exist in any payload.
- The server derives the attack origin from its own view of the character; the client does not
  supply an origin. This removes one forgery vector and accepts the residual movement risk
  (R-06) explicitly.
- The sequence number is for diagnostics and duplicate suppression only; it grants no
  authority.
- Direction is normalised and bounded; `NaN` and infinity are rejected before use.

### 6.5 Validation checklist

| Check | Rule | On failure |
| --- | --- | --- |
| Caller identity | Use the engine-provided player argument | Ignore; log once per interval |
| Arity | Exactly the documented count | Reject |
| Types | `Vector3`, `number` as documented | Reject |
| Finiteness | No `NaN`, no infinity | Reject |
| Magnitude | Direction length and values within documented bounds | Reject |
| Rate | Within the per-player allowance | Drop silently; count |
| State | Alive, correct phase | Reject |
| Cooldown | Server timestamp at or past the allowed time | Reject |
| Part state (M4) | Required slot enabled | Reject; no partial effect |
| Geometry | Within range, line of sight clear | No damage; cosmetic feedback only |
| Target | Valid character, alive, allowed target | No damage |

The table is the contract that T-M2-04 through T-M2-10 exercise. Exact numeric limits belong to
`NETWORKING.md`; until then, `Config/Network.luau` holds clearly labelled placeholders that are
not presented as balanced values.

### 6.6 Frequency control

- One token-bucket-style allowance per player per remote, refilled over time.
- Excess requests are dropped, not queued; queuing would convert spam into latency.
- Rejections are counted and logged at a throttled rate so a burst cannot itself flood Output.
- The limiter must be cheap: constant work per call, no table growth per request, no per-frame
  sweep.
- T-M2-03 and T-M2-09 measure whether the limiter works and whether it costs more than it
  saves (risk R-07).

### 6.7 Rejection and error handling

- Every handler body is wrapped so a raised error cannot propagate into the server loop.
- Rejections are silent to the sender or answered with a generic feedback kind; no stack
  traces, internal paths, or state details go to the client.
- Server-side logging is rate-limited per player and per remote.
- A malformed payload never disconnects an innocent player and never mutates state
  (T-M2-08, T-M2-11).

### 6.8 `RemoteFunction` policy

`RemoteFunction` blocks the caller and, on the server, can hold a thread until the client
answers. That is a denial-of-service surface and complicates rate limiting.

- Not used in M0–M2.
- A future need must justify itself in `NETWORKING.md` and pass Gate G3 before use.
- Any future `RemoteFunction` must still validate all arguments and be rate-limited.

### 6.9 Forbidden patterns

- A remote whose payload includes a damage or health value.
- A server handler that trusts `args[1]` as the acting player.
- A client-created remote the server connects to.
- A handler that calls `wait()`/`task.wait()` before validating rate.
- Sending full server state to clients "for convenience".
- Using feedback remotes to drive authoritative client state.

### 6.10 Deferred to `NETWORKING.md`

Final names, final schema, numeric limits, cooldown representation, feedback vocabulary, and
the adversarial test payload set. `TESTING.md` marks dependent M2 checks **PROVISIONAL** until
that document is approved; this architecture does not pre-empt it.

---

## 7. Expansion path to modular body-part damage

The goal is that M4 adds a damage model without rewriting the M2 request path and without ever
physically removing a limb.

### 7.1 Invariants

1. The Humanoid rig stays intact: no part is deleted, detached, reparented, or renamed.
2. "Destroyed" means **disabled**: the part's ability is gated off and its state and visuals
   change.
3. Part state is server-owned and replicated read-only.
4. There is still exactly one death path and one authoritative overall-health pool.
5. A disabled part cannot be re-enabled except by an approved rule (D-02, D-06) or on respawn.

### 7.2 Slot model

| Slot | M2 meaning | M4 meaning | Rig mapping |
| --- | --- | --- | --- |
| `Overall` | The single health pool | Derived or separate per D-01 | Humanoid |
| `RightArm` | — | Attack ability | Right arm parts |
| `LeftArm` | — | Defensive ability | Left arm parts |
| `Torso` | — | Special ability | Torso parts |
| `Legs` | — | Movement ability | Leg parts |

The exact rig mapping depends on R15 versus R6, which is not decided (open question A-09).
Part names are read, never changed.

### 7.3 Stable damage interface — proposal A-07

All damage enters through one conceptual signature — a design sketch, not code:

`applyDamage(target, slot, amount, context)`

- M2 calls it with the single `Overall` slot.
- M4 calls it with the slot resolved from the hit part.
- `context` carries the cause and attacker for later scoring (M3) and logging.
- Only `HealthService` (or the part-state module it delegates to) mutates health.

Because the call site already exists in M2, M4 changes what happens inside the function, not
the flow around it.

### 7.4 State representation and replication — proposal A-06

- Server: one record per player per life, holding per-slot health and a disabled flag, derived
  from `Config/Combat.luau`.
- Replication: attributes on the character model (for example a numeric health attribute and a
  boolean disabled attribute per slot), because attributes are expected to replicate
  automatically (§1.5) and this avoids a custom remote.
- Client: reads attributes for HUD and visuals only; never writes them.
- Replication cost is unverified (risk R-05); M4 measures it before the pattern spreads to more
  state.

If attributes prove unsuitable in Studio, the fallback is a single server→client state remote
with a compact payload, which is a Gate G3 change, not an agent decision.

### 7.5 Disable semantics

- Disabling changes gameplay availability first: `CombatService` checks the slot before
  resolving an attack or ability.
- Visual change is allowed and expected, but limited to safe properties (transparency, colour,
  material, attached effects). D-05 still decides how far this goes.
- If a part is disabled while an ability is cooling down, the ability becomes unavailable
  immediately; the cooldown does not keep it alive (M5 requirement, prepared here).
- The Humanoid is never put into a ragdoll or death state by a part disable (risk R-02).

### 7.6 Reset on respawn

A new life starts from a clean record built from configuration. No part health, disabled flag,
or cooldown carries across respawn unless an approved rule says otherwise (D-02). Reset happens
in the same place that initialises per-life state (§4.3), so there is one reset point.

### 7.7 M2 to M4 migration steps

1. Add `Types/BodySlot` values for the four slots (M2 already has `Overall`).
2. Add the part-state module and its per-life record.
3. Make `HitValidation` return the hit part alongside the target.
4. Map the hit part to a slot; unknown parts are rejected or attributed by an approved rule.
5. Route `applyDamage` by slot and update the overall pool per D-01.
6. Replicate attributes and gate abilities in `CombatService`.
7. Reset on respawn and add the debug overlay required by M4 acceptance.

No step requires changing the remote schema, the request path, or the single death path.

### 7.8 Risks specific to this expansion

| Risk | Effect | Mitigation |
| --- | --- | --- |
| R-02 | Humanoid reacts badly to a visually altered limb | Spike on a test rig before M4 |
| R-04 | Limb hitboxes misattribute hits | Scripted raycast tests from fixed positions |
| R-05 | Four slots × N players costs too much to replicate | Measure before widening |
| D-05 | Visual rules undefined | Keep changes reversible and cosmetic until decided |

---

## 8. Minimum PC and mobile requirements

### 8.1 One input model

Both devices produce the same `AttackIntent`. Combat logic exists once, on the server; the
client only maps inputs. A second, touch-only combat path is forbidden.

### 8.2 PC minimum

- Default Roblox movement, jump, and camera remain in place for M1–M2 (camera design is D-13).
- Attack is reachable with mouse and with a keyboard binding; it must not require a right-click
  or hover.
- No custom character controller (prototype constraint).

### 8.3 Mobile minimum

- Roblox's default thumbstick, jump button, and camera drag area stay functional and
  unobstructed (T-M1-07).
- One temporary attack control is reachable with a thumb and does not overlap default controls
  (T-M2-12). A `ContextActionService` action with a touch button is the proposed mechanism;
  its exact behaviour must be confirmed in the installed Studio build.
- The control respects device safe areas and remains visible on a notched phone profile.
  Property names and their behaviour must be verified in the installed Studio build, not
  assumed.
- No critical state is hidden behind hover; no interaction requires a right-click.

### 8.4 Layout rules

- Use scale-aware UI sizing rather than fixed pixel offsets for anything that must survive a
  phone and a tablet.
- Keep the temporary HUD minimal in M2: current health is enough to prove replication; a full
  HUD belongs to `UI_AND_CONTROLS.md` (M8).
- Test at phone and tablet profiles, not only at desktop resolution.

### 8.5 Performance rules

- Event-driven only: attribute changes, `Touched` where appropriate, remote events. No
  per-frame loops, no polling of state that has a signal.
- Effects in M2 are minimal; every added effect is a mobile cost.
- Rate limiting costs must be constant per request (risk R-07).
- Measurements, when taken, are reported with tool, scene, and player count
  (`TESTING.md` §2.1). No performance claim may be made without recorded measurement at the
  stage where it is relevant (M4, M5, M9).

### 8.6 What L3 does and does not prove

Device Emulator (L3) is required for the M1 movement smoke check and the M2 attack-control
check. It does not prove real-device behaviour; that is L4, which is not required for M1–M2 and
is not claimed (risk R-12).

---

## 9. Traceability to `ROADMAP.md` and `TESTING.md`

### 9.1 Stage prerequisites

| Stage | Architecture prerequisites | Other prerequisites (`ROADMAP.md` §2.1) |
| --- | --- | --- |
| M0 | A-01/A-02 named in the M0 task spec; place/asset policy A-11 decided | G1 package (`AGENTS.md`, `TASK_OWNERSHIP.md`, `ROADMAP.md`, `TESTING.md`, `ARCHITECTURE.md`), approved 2026-10-27; Gate G2 |
| M1 | **This document approved**; A-03 and A-13 chosen (A-13 has no recorded answer yet, §10.3) | M0 accepted at G6; `TESTING.md` |
| M2 | A-04 confirmed by the M2 overall-health section of `COMBAT_AND_DAMAGE.md`; A-05 by `NETWORKING.md`; A-07 fixed here (§7.3), with its M4 widening gated by D-01; A-08 by the owner's D-13 camera decision | M1 accepted; `NETWORKING.md`; overall-health section and partial-hit rules of `COMBAT_AND_DAMAGE.md`; D-13 recorded in `AGENTS.md` §14; Q-06/A-12 only before T-M2-07/08/09 and the dependent T-M2-11 |

`ARCHITECTURE.md` was approved with the G1 package on 2026-10-27 and is named again as the M1
prerequisite in `ROADMAP.md` §2.1. Its M0-relevant proposals (A-01/A-02) must still be named in
the approved M0 task spec. This document does not satisfy the M2 prerequisites, and M2 must not start on its
strength alone.

### 9.2 Which check proves which decision

| Architectural decision | Proof | Level |
| --- | --- | --- |
| A build command produces an openable place (A-01) | T-M0-01 | L0 |
| Three containers exist as folders with the agreed contents | T-M0-02 | L0/L1 |
| Disk edits reach Studio without copying | T-M0-03 | L1 |
| No gameplay exists in M0 | T-M0-04 | L0/L1 |
| Build is reproducible on another checkout (A-01) | T-M0-05 | L0 |
| Toolchain versions are pinned and documented (A-01) | T-M0-06 | L0 |
| Sync failure is visible rather than silent | T-M0-07 | L1 |
| Build artifacts follow the place/asset policy (A-11) | T-M0-08 | L0 |
| Spawn selection produces distinct valid positions | T-M1-01 | L2 |
| Default movement and camera remain usable | T-M1-02 | L2 |
| One death path handles out-of-bounds and respawn timing | T-M1-03, T-M1-04, T-M1-05 | L2 |
| Spawn and respawn values are configuration-driven | T-M1-06 | L1 |
| Default touch controls stay unobstructed | T-M1-07 | L3 |
| Join/leave does not corrupt per-life state | T-M1-08 | L2 |
| M1 contains no combat | T-M1-09 | L2 |
| Server resolves hits and damage | T-M2-01, T-M2-02 | L2 |
| Cooldown is server-enforced | T-M2-03 | L2 |
| Geometry and range checks run on the server | T-M2-04, T-M2-05, T-M2-06 | L2 |
| Payloads carry no usable outcome | T-M2-07, T-M2-08 | L2 |
| Rate limiting protects the server | T-M2-09 | L2 |
| Health cannot be authored by a client | T-M2-10 | L2 |
| Abuse does not leak to other players | T-M2-11 | L2 |
| Attack input works on touch | T-M2-12 | L3 |

The artifact build (T-M0-01) and live sync (T-M0-03) are deliberately separate checks. A
passing build does not prove live sync and a passing live sync does not prove a reproducible
build. The exact commands and tool names are known only after the M0 task spec is approved and
Gate G2 is granted.

### 9.3 Provisional checks

T-M2 tests that name remote fields or numeric limits depend on `NETWORKING.md` and are marked
**PROVISIONAL** there. The non-combat half of T-M2-02 is PROVISIONAL until the treatment of
non-combat damage (A-13) is recorded, because it compares behaviour against that rule (§4.6).
The partial-hit half of T-M2-04 is PROVISIONAL until `COMBAT_AND_DAMAGE.md` is approved with
those rules; no document currently defines how a partially exposed target resolves. T-M0-08 is
**PROVISIONAL** until the place and asset policy (Q-03 / A-11) is decided; while it is
PROVISIONAL it is recorded `BLOCKED`. The `AGENTS.md` §14 decision log is the single
authoritative record of that decision: T-M0-08 becomes **FINAL** only when the owner has
recorded D-16 there, and the same change then updates the test's status line, preconditions, and
expected result, shows D-16 as decided in `ROADMAP.md` §6, and adds the matching change-log rows
in `ROADMAP.md` and `TESTING.md`. The test rewrite and the roadmap row are prerequisites for
executing it as FINAL; the change-log rows are a documentation duty of the same change, not a
separate gate. None of these checks can be counted as passed until the dependency is approved
or the decision recorded and the test is updated. This document does not make them pass by
describing them.

### 9.4 What passing every check still does not prove

The limits listed in `TESTING.md` §6.3 apply unchanged: no proof against a modified client, no
production-network behaviour, no mobile performance, no anti-cheat completeness. Architecture
reduces risk; tests measure specific cases; neither eliminates the rest.

---

## 10. Risks, proposals, and open questions

### 10.1 Inherited risks that constrain this design

| Risk | Constraint it places here |
| --- | --- |
| R-01 Candidate build/sync reliability | M0 must prove sync before M1 relies on it; no M1 work starts on an unverified pipeline |
| R-02 Humanoid behaviour when a limb is disabled | No visual limb change before a Studio spike; M4 design stays reversible |
| R-03 Engine death rules versus custom health | One death path, explicitly disabled stray damage sources (A-13) |
| R-04 Limb hitbox accuracy | Slot resolution is isolated in one module so M4 can correct it in one place |
| R-05 Part-state replication cost | Attributes are a proposal, not a commitment; measure before spreading |
| R-06 Client-authoritative movement | The server never accepts an origin; residual risk documented, not hidden |
| R-07 Rate-limit overhead | The limiter must be constant-cost and measured in M2/M9 |
| R-10 `CharacterAutoLoads` interaction with respawn | A-03 must be chosen and verified in M1, not assumed |
| R-13 Number of local test clients | L2 claims need two clients; capacity is an owner-confirmed precondition |

### 10.2 Architecture proposals awaiting approval

| ID | Proposal | Related | Decide before |
| --- | --- | --- | --- |
| A-01 | Rojo is a **candidate** for the M0 build and sync pipeline; the tool, version, pin file, build command, artifact path, Studio connection, and sync method are selected in the M0 task spec and Gate G2 | Q-02, R-01 | M0 |
| A-02 | Mapping `src/shared` → `ReplicatedStorage.Shared`, `src/server` → `ServerScriptService.Server`, `src/client` → `StarterPlayerScripts.Client`; containers remain Folders | M0 criteria, T-M0-02 | M0 |
| A-03 | `CharacterAutoLoads = false` with explicit `LoadCharacter()` through `SpawnService` (option A in §4.8) | R-10, M7 hub flow | M1 |
| A-04 | Overall health carried by `Humanoid.Health`, written only by `HealthService`, values from config | `COMBAT_AND_DAMAGE.md` M2 section, R-03 | M2 |
| A-05 | M2 surface is one attack `RemoteEvent` plus one cosmetic feedback `RemoteEvent`; no `RemoteFunction` | `NETWORKING.md` | M2 |
| A-06 | Part state is a server-owned record replicated as character attributes | R-05 | M4 |
| A-07 | Single damage interface with an `Overall` slot in M2 and four slots in M4 | D-01 | M2 |
| A-08 | PC and touch produce the same intent; default movement, jump, and camera retained | D-13 | M2 |

### 10.3 Open questions

| ID | Question | Blocks | Notes |
| --- | --- | --- | --- |
| A-09 | R15 or R6 rig for the prototype, and therefore the part-to-slot map | M4 | Affects hitboxes and visual rules; record before M4 planning |
| A-10 | Target player count per server (D-12) | M9 load planning (deferred); not M0 | Two clients are the M0–M2 test-session size and minimum network check; this architecture is count-agnostic and two clients are not a server cap |
| A-11 | Place and asset policy (D-16) | M0 | Proposed policy in `ROADMAP.md` §6; owner confirmation pending. Determines whether `build/` and `places/` are ever committed |
| A-12 | Where temporary test scaffolding may live (Q-06) | M2 negative checks | Needed before T-M2-07/08/09, and T-M2-11 which runs them, can be executed. Covers every mechanism: Studio Command Bar commands, temporary scripts, and repository test scaffolding (`TESTING.md` §7) |
| A-13 | Treatment of non-combat damage: fall damage, drowning, explosions | M1 | Needs an explicit rule (disabled or routed through `DeathService`); currently unspecified |

### 10.4 What still blocks the first stage

M0 remains blocked by owner decisions outside this document: tooling selection and install
(Q-02) and confirmation of the place and asset policy (Q-03 / D-16). The G1 package was approved
on 2026-10-27 (Q-01), so that prerequisite is satisfied. Target player count (D-12 / Q-04) is
deferred and is **not** an M0 blocker. The
rule that L4 does not apply to M0 (Q-09) is a derived rule recorded as a proposal, not an
approved decision: the owner has not confirmed it, so it is tracked as pending in
`AGENTS.md` §14. It is not a blocker for M0 work or testing, because M0 produces no playable or
published build and therefore no L4 check; it does constrain how M0 acceptance may be recorded.
`ROADMAP.md` §4 M0 Device checks states that condition, and `TESTING.md` §13 Q-09 lists every
location the owner's answer would change. This document was approved with the G1 package on
2026-10-27; its M0-relevant proposals (A-01, A-02) must still be named in the approved M0 task
spec at Gate G4, and A-11 is the same place/asset question as Q-03 rather than a new blocker.

M1 additionally requires this document to be approved, with A-03 (spawn control) and A-13
(non-combat damage) chosen; A-13 has no recorded answer yet (§10.3), so the M1 kill-volume rule
may not be implemented under an assumed policy. M2 requires `NETWORKING.md`, the overall-health
section and the partial-hit rules of `COMBAT_AND_DAMAGE.md`, and D-13; the Q-06/A-12 scaffolding
decision is needed only before T-M2-07 through T-M2-09, and the dependent T-M2-11, are
executed.

---

## 11. Change log

| Date | Change | Approved by |
| --- | --- | --- |
| 2026-10-27 | Initial draft created under Task 004: M0–M2 boundaries, layout, spawn/respawn flow, combat placement, remote rules, part-damage extension path, PC/mobile minimums, traceability | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 006 reconciliation: Rojo demoted to a candidate with tool selection deferred to the M0 task spec and G2 (§1.3, §3.1–3.3, §10.2 A-01); separate artifact-build and live-sync checks (§3.1, §9.2); M2 health carrier separated from M4 D-01 relationship (§5.4); M1 kill-volume death retained as an approved cause in M2 (§4.6); nonexistent `PartStateService` replaced by a proposed M4 part-state module (§5.2); D-12 deferred and G1 package prerequisites updated (§9.1, §10.3, §10.4) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 008 reconciliation of review findings: Q-09 downgraded from "resolved" to a derived rule with owner confirmation pending (document-overview table in the preamble; §10.4); M2 prerequisites now name D-13 and scope Q-06/A-12 to the crafted-payload checks (§9.1, §10.4); T-M0-08 PROVISIONAL→FINAL transition recorded (§9.3) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 010 correction of Task 009 findings: the Task 008 change-log pointer now names the preamble document-overview table rather than §1 (this table); the kill-volume death is described as needing the unrecorded A-13 rule instead of an "explicitly approved" cause (§4.6, §10.4); M2 prerequisites name the owning document or owner decision for each of A-04/A-05/A-07/A-08 (§9.1); T-M0-08's transition names the `AGENTS.md` §14 log as the single record and separates prerequisites from change-log follow-ups (§9.3); Q-09 stated as not blocking M0 work but constraining acceptance (§10.4); A-12/Q-06 scope extended to Command Bar commands, temporary scripts, and repository scaffolding (§10.3) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 012 corrective pass: the §1.5 amendment path now names this change log, the `AGENTS.md` §14 decision log, and the G3 threshold instead of an unspecified change "recorded in §11 and in `ROADMAP.md`" (§1.5); the §9.3 line break that split the decision-log reference is repaired and the T-M2-04 partial-hit half is recorded as PROVISIONAL on the unapproved `COMBAT_AND_DAMAGE.md` rules (§9.3); M2 prerequisites in §9.1 and §10.4 name the partial-hit rules and the dependent T-M2-11; A-12's scope names T-M2-11 (§10.3, §10.4) | Approved at G1, 2026-10-27 |
| 2026-10-27 | Task 015: Gate G1 owner approval recorded — this document approved on 2026-10-27 with `AGENTS.md`, `TASK_OWNERSHIP.md`, `ROADMAP.md`, and `TESTING.md`, with `README.md` informational and outside the approval (status line, §9.1 M0 row, §10.4). No open question or proposal status changed: A-13, D-13, Q-06, D-16, and Q-02 remain open, and Task 013 Minor findings MN-1…MN-3 remain open with their recorded timings | Approved at G1, 2026-10-27 |
