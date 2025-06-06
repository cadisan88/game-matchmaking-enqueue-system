# Distributed FPS Game Matchmaking & Enqueue System

## Project Overview

This project implements a distributed, asynchronous matchmaking and game enqueue system tailored for a First-Person Shooter (FPS) game. It leverages **RabbitMQ** as the core messaging backbone to handle player requests, route them to appropriate game queues based on specific criteria (game mode, skill level, and region), and facilitate the start of game sessions by simulated game servers.

The primary goal is to demonstrate a robust, scalable, and fault-tolerant matchmaking solution that can handle high volumes of concurrent player requests in a microservices-like architecture.

---

## Core Functionality

### Player Game Request Submission

Players submit requests to join a game via an API endpoint. Each request includes:
* `player_id`: A unique string identifier for the player.
* `game_mode`: The desired game mode.
* `player_skill_level`: An integer from 1 to 100, indicating the player's skill.
* `timestamp_requested`: The time the request was made (Unix timestamp).
* `preferred_region`: The geographic region the player prefers for matchmaking.

### Skill-Based Matchmaking

The system prioritizes matching players with similar `player_skill_level` within their defined skill tier. For the initial design (V1), **players are only matched within their exact skill tier** (e.g., Novice with Novice). Cross-tier matching for graceful degradation (e.g., if a player waits too long) is out of scope for V1 but noted for future expansion.

### Game Mode and Skill Tier Specific Queues

Player requests are routed to distinct RabbitMQ queues based on a combination of their requested `game_mode`, their derived `player_skill_level` tier, and their `preferred_region`. Each queue holds players exclusively for a specific combination.

### Game Server Consumption

Simulated "Game Servers" act as consumers, pulling player requests from specific queues. Each Game Server instance is configured to handle a particular `game_mode`, `skill_level` tier, and `region`. Game Servers **must manually acknowledge** player messages only after successfully adding the player to an in-memory game lobby.

### Game Session Creation

Each Game Server has a predefined `max_players_per_session` for each `game_mode`:
* **TeamDeathmatch (TDM):** 12 players (6v6)
* **FreeForAll (FFA):** 8 players
* **CaptureTheFlag (CTF):** 16 players (8v8)

When a Game Server accumulates the required players for a session, it simulates "starting" the game by logging a message (e.g., "Game [Session ID] starting with [X] players in [Game Mode] in [Region] (Skill Tier: [Tier Name])") and clearing those players from its internal lobby.

### Player Notification (Simulated)

Once a game session is ready and players are matched, the system simulates notifying those players (e.g., by logging "Player [player_id] matched into Game [Session ID] on Server [IP:Port]"). In a production environment, this would typically involve real-time communication like WebSockets.

---

### 1. Game Modes

The system supports the following FPS game modes:
- **TeamDeathmatch:** Classic team-vs-team elimination. Code `TDM`.
- **FreeForAll:** Every player competes individually. Code **`FFA`.**
- **CaptureTheFlag:** Team-based objective where players capture an opponent's flag. Code **`CTF`.**

### 2. Player Skill Levels

Player skill is represented by a numerical value from **1 to 100**, mapped to these military-themed tiers for matchmaking:
- **Novice:** Skill Level 1 - 25. Code `N`.
- **Soldier:** Skill Level 26 - 50. Code `S`.
- **Special Forces:** Skill Level 51 - 75. Code `F`.
- **Elite:** Skill Level 76 - 100. Code `E`.

### 3. Regions

Players select a preferred region for matchmaking, corresponding to major geographical areas. Matchmaking occurs exclusively within the chosen region to ensure low latency.
* **Americas:** North and South American continents. Code `AM`.
* **Europe:** European continent. Code `EU`.
* **Asia:** Asian continent. Code `AS`.
* **Africa:** African continent. Code `AF`.
* **Oceania:** Australia and surrounding islands. Code `OC`.
