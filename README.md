# Project Xeno

## 1. Project Overview

Project Xeno is a multiplayer arena shooter built for Roblox.

The game focuses on modular character construction, tactical combat, destructible body parts, and rebuilding a fighter after defeat.

**Current MVP scope.** The near-term minimum viable prototype is limited to stages M0–M2:
project skeleton, an arena with spawning and respawn, and server-authoritative combat with
overall health. The hub, part assembly, the full defeat → rebuild → re-enter loop, body-part
damage, the four abilities, scoring, and mobile-specific UI belong to later stages. The
sections below describe the intended game, not the current prototype.

## 2. Target Platform

- Game engine: Roblox
- Development environment: Roblox Studio
- Programming language: Luau
- Target devices: PC, Android, iOS
- Multiplayer: online arena combat

Do not use Unity, Unreal Engine, or C# for the game implementation.

## 3. Core Gameplay

Players collect and combine modular body parts to create a combat unit.

The main body modules are:

- Right arm: attack ability
- Left arm: defensive ability
- Torso: special ability, such as a shield or teleport
- Legs: movement ability

Each module provides its own gameplay function. The character has four active abilities associated with these modules.

## 4. Destructible Body Parts

Body parts can receive damage and lose functionality when destroyed.

Examples:

- Destroyed right arm: the associated attack ability becomes unavailable.
- Destroyed left arm: the associated defensive ability becomes unavailable.
- Destroyed legs: the character loses normal mobility and must fight from the ground using available abilities.

The implementation must define clear rules for damage, module health, and disabled abilities.

## 5. Defeat and Reassembly

When the character's overall health reaches zero, the player returns to the hub and can assemble a new fighter from available parts before re-entering combat.

The initial concept also includes a rolling-ball starting form. The exact relationship between this form, collecting parts, and hub-based assembly must be clarified during design.

## 6. Progression

- Players earn points for eliminations.
- The game includes a leaderboard.
- Further progression and reward systems are to be designed.

## 7. Development Principles

- Build in small, testable stages.
- Prioritize a playable prototype before expanding content.
- Keep multiplayer authority and damage validation on the server.
- Design controls and interfaces for PC and mobile.
- Avoid unnecessary complexity and oversized systems.
- Document decisions before implementing major systems.
- Never claim that a feature works without testing it.

## 8. AI Agent Workflow

OpenCode owns planning, architecture, technical specifications, and review of implementation proposals.

Cline owns implementation tasks explicitly assigned to it.

The agents must not edit the same files simultaneously. Every implementation task must have defined scope, acceptance criteria, and a test procedure.

All game implementation must target Roblox Studio and Luau.