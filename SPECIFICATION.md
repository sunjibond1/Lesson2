# Tactical Sports Card Game — Game Specification

---

## 1. Game Concept

A **turn-based tactical PvP card game** set in a sports theme. Players compete in matches (Football, Basketball, Tennis) on a 5x5 grid, controlling their character avatar. The goal is to be the first to reach **20 points** by hitting a ball toward targets (goals, teammates, opponents). Winners earn collectible cards as prizes.

Players collect cards, complete themed collections, earn Gacha packs, and exchange cards for powerful single-use gear.

---

## 2. Core Gameplay

### 2.1 Match Overview
| Parameter | Value |
|-----------|-------|
| Players | 1v1 (or vs AI) |
| Field | 5x5 grid (square tiles) |
| Ball | Single ball, both players compete |
| Win Condition | First to **20 points** |
| Turn Timer | 15 seconds per turn |
| Timeout Behavior | Auto-skip turn (no action taken) |

### 2.2 Scoring Actions

When a player kicks/hits the ball, the result determines points:

| Outcome | Points |
|---------|--------|
| Ball enters the **goal** | **+5** |
| Ball hits a **teammate** | **+3** |
| Ball hits an **opponent** | **-1** |
| Ball goes **out of bounds** | **-3** |

### 2.3 Gear Effect
If a player equips gear before the match:
- **Effect:** All points scored are **multiplied by x2**
- **Duration:** Consumed after the match ends (single use)

### 2.4 Turn Flow
1. Both players see the current game state (ball position, scores, grid)
2. Each player selects an action (Move, Kick/Capture)
3. Both submit their actions within 15 seconds
4. Server resolves both actions simultaneously
5. Results are displayed, scores updated
6. Repeat until a player reaches 20 points

### 2.5 Action Types
| Action | Description |
|--------|-------------|
| **Move** | Move character to an adjacent tile (up, down, left, right, diagonals) |
| **Kick** | Hit the ball from current position toward a target tile |

### 2.6 Conflict Resolution (Simultaneous Actions)
When both players act on the same tile or ball:
1. **Ball possession** is determined by proximity before the action
2. If equidistant: **random** 50/50 chance
3. If one player moves toward ball while other doesn't: moving player gets priority
4. If both try to kick: ball owner kicks first, then secondary effects resolve

---

## 3. Characters

### 3.1 Overview
| Property | Value |
|----------|-------|
| Total Characters | 10 |
| Sport Types | Football, Basketball, Tennis |
| Function | Visual avatar (cosmetic: clothes, gear, background) |
| Unlock | All available from the start |
| Stats | No gameplay stats difference (purely cosmetic) |

### 3.2 Character Selection
- Players select their character **before** entering a match
- The sport type of the character determines the **visual theme** of the match
- Football character → Football field background, football visuals
- Tennis character → Tennis court background, tennis visuals

---

## 4. Card System

### 4.1 Card Types

| Rarity | Color | Drop Rate | Purpose |
|--------|-------|-----------|---------|
| **Common** | Gray | 80% | Exchange for gear (10 cards = 1 gear) |
| **Rare** | Blue | 15% | Collectible, prestige |
| **Super** | Gold | 5% | Ultra-rare collectible, special badge |

### 4.2 Card Collections

A **Collection** is a themed set of **exactly 5 cards**. Completing a collection unlocks a Gacha pack.

| Collection Name | Sport | Cards Required |
|-----------------|-------|----------------|
| Barcelona FC | Football | 5 players |
| Real Madrid | Football | 5 players |
| Manchester United | Football | 5 players |
| Wimbledon 2024 | Tennis | 5 players |
| Wimbledon 2025 | Tennis | 5 players |
| Wimbledon 2026 | Tennis | 5 players |
| NBA Champions | Basketball | 5 players |
| Lakers Legends | Basketball | 5 players |

*New collections can be added over time.*

### 4.3 Earning Cards

| Source | Description |
|--------|-------------|
| **Match Prize** | Winner receives the prize card wagered for the match |
| **Gacha Packs** | Earned by completing collections |
| **Daily Rewards** | (Future feature) |

### 4.4 Prize Card System

Before a match starts, a **prize card** is selected:
1. **System offers 3 random cards** from the card pool
2. Match creator can choose one of the 3
3. Match creator can **wager their own card** from their inventory instead
4. Winner of the match **receives the prize card**

The prize card is always visible at the **top of the game screen** during the match.

### 4.5 Gacha System

#### Earning Gacha Packs
- Complete any 5-card collection → **Unlock 1 Gacha Pack**

#### Opening a Gacha Pack
- Animated "rip open" experience
- Reveals 5 random cards

#### Gacha Pack Contents

| Outcome | Probability |
|---------|-------------|
| 1 Super Card + 4 Regular Cards | 10% |
| 5 Regular Cards (Common or Rare) | 90% |

*Players do not know which specific cards they will receive until the pack is opened.*

### 4.6 Regular Card Exchange

| Exchange Rate | Result |
|---------------|--------|
| 10 Regular Cards | 1 Gear item (sport-specific) |

- Players can exchange **any 10 regular cards** (Common or Rare) for 1 gear item
- Gear can be chosen for **any sport** (not limited to the cards' sport)
- Gear is stored in inventory until used

---

## 5. Gear System

### 5.1 Overview
| Property | Value |
|----------|-------|
| Types | Football Boots, Basketball Shoes, Tennis Racquet |
| Cost | 10 Regular Cards |
| Effect | **Points multiplier x2** for the entire match |
| Duration | Single use (consumed after match) |
| When Equipped | Before match starts |

### 5.2 Gear Selection
- Players select gear from their inventory **before joining a match**
- Only **one gear item** can be equipped per match
- Gear is **burned** (removed from inventory) after the match ends, regardless of outcome

---

## 6. Matchmaking

### 6.1 Public Matchmaking

| Step | Description |
|------|-------------|
| 1 | Player clicks **"Start Game"** |
| 2 | Select sport type (Football/Basketball/Tennis) |
| 3 | Select prize card (3 system options OR own card) |
| 4 | System searches for another player in the queue |
| 5 | If found → Match created, both players notified |
| 6 | If no one found → Player enters queue |
| 7 | If still no match after timeout → Offer **vs AI** option |

### 6.2 Private Matches (Invite Friends)

| Step | Description |
|------|-------------|
| 1 | Player clicks **"Create Private Game"** |
| 2 | Select sport type and prize card |
| 3 | System generates a **unique invite link** |
| 4 | Player shares link with friend |
| 5 | Friend opens link → Prompted to join the match |
| 6 | Friend accepts → Match created |
| 7 | Link expires after match starts or after **10 minutes** |

### 6.3 Join Notifications

When a player creates a public game, other online players see:

> **"[PlayerNickname] is inviting you to join the Game"**
> 
> [Join]  [Decline]

### 6.4 Match Restrictions
- **One active game at a time** per player
- Cannot join another match while already in one
- Cannot be in queue while in an active match

---

## 7. User Accounts

### 7.1 Registration
| Field | Requirement |
|-------|-------------|
| Email | Valid, unique |
| Nickname | 3-20 characters, unique, alphanumeric |
| Password | 8+ characters |

### 7.2 Login
- Email + Password
- JWT token (24-hour expiry)
- Token stored in browser localStorage

### 7.3 User Profile
| Property | Description |
|----------|-------------|
| Nickname | Customizable (unique) |
| Level | Earned through XP |
| Gems | In-game currency |
| Collections | Progress tracker |
| Win/Loss Record | Match history stats |

### 7.4 XP & Leveling

| Action | XP Earned |
|--------|-----------|
| Win a match | +50 XP |
| Lose a match | +20 XP |

| Level | Cumulative XP | Reward |
|-------|---------------|--------|
| 1 | 0 | Starting level |
| 2 | 100 | Feature unlock |
| 3 | 250 | Feature unlock |
| 4 | 500 | +50 Gems bonus |
| 5 | 1,000 | Feature unlock |
| 6 | 2,000 | Feature unlock |
| 7 | 3,500 | Feature unlock |
| 8 | 5,000 | Feature unlock |

*Feature unlocks and rewards will be defined as development progresses.*

### 7.5 Gems Currency
| Property | Value |
|----------|-------|
| Starting Amount | 100 Gems |
| Earning | Level-ups, events (TBD) |
| Spending | Future features (cosmetics, boosts) |

---

## 8. Game UI Specification

### 8.1 Main Menu / Dashboard
- **Play** button (opens matchmaking)
- **Collection** (view card collections)
- **Inventory** (view gear and cards)
- **Profile** (stats, level, gems)
- **Gacha** (open packs)

### 8.2 Matchmaking Screen
- Sport type selector (Football / Basketball / Tennis)
- Prize card selection:
  - 3 system-suggested cards (with "refresh" option)
  - "Use My Card" button (opens card inventory)
- **Start Game** button
- **Create Private Game** button (generates invite link)

### 8.3 Game Screen (Match)

#### Layout
```
┌──────────────────────────────────────────┐
│         🃏 Prize Card Display             │
│    [Card Image] | Card Name | Rarity      │
├──────────────┬─────────────────┬──────────┤
│  Player 1    │     Timer       │ Player 2 │
│  [Nickname]  │     0:15        │ [Nickname│
│  Score: 12   │                 │ Score: 8 │
├──────────────┴─────────────────┴──────────┤
│                                          │
│          ┌───┬───┬───┬───┬───┐           │
│          │   │   │   │   │   │           │
│          ├───┼───┼───┼───┼───┤           │
│          │   │   │ 🏐│   │   │           │
│          ├───┼───┼───┼───┼───┤           │
│          │   │   │   │   │   │  5x5 Grid │
│          ├───┼───┼───┼───┼───┤           │
│          │   │👤 │   │  👤│   │           │
│          ├───┼───┼───┼───┼───┤           │
│          │   │   │   │   │   │           │
│          └───┴───┴───┴───┴───┘           │
│                                          │
├──────────────────────┬───────────────────┤
│  [Move]  [Kick]      │  [Forfeit]        │
└──────────────────────┴───────────────────┘
```

#### Elements
| Element | Position | Description |
|---------|----------|-------------|
| Prize Card Banner | Top center | Always visible, shows wagered card |
| Player 1 Info | Top left | Nickname, score, avatar |
| Player 2 Info | Top right | Nickname, score, avatar |
| Timer | Top center (below prize) | 15-second countdown |
| Game Grid | Center | 5x5 interactive grid |
| Ball | On grid | Visual indicator of ball position |
| Action Buttons | Bottom | Move, Kick, Forfeit |

### 8.4 Collection Screen
- Grid of all collections
- Each collection shows progress (e.g., 3/5 cards)
- Clicking a collection shows individual cards
- Completed collections glow/glowing effect
- "Open Gacha Pack" button appears for completed collections

### 8.5 Inventory Screen
- Tabs: Cards / Gear
- **Cards tab:** All owned cards with quantities
- **Gear tab:** Owned gear items with "Exchange 10 Cards" button
- "Exchange" button: Select 10 regular cards → Choose gear → Confirm

### 8.6 Gacha Screen
- Stack of unopened Gacha packs
- Click to open → Animation plays
- Cards revealed one by one
- "Claim All" button after reveal

---

## 9. AI Opponent

### 9.1 When AI is Used
- No opponent found in matchmaking queue
- Player chooses "Play vs AI" option

### 9.2 AI Behavior
The AI plays with a simple strategy:

1. **If ball is within 2 tiles** → Kick toward the best target
2. **Otherwise** → Move toward the ball
3. **Target selection priority:**
   - Goal (+5 points): 50% chance
   - Teammate (+3 points): 30% chance
   - Opponent (-1 point): 10% chance
   - Random/out of bounds: 10% chance

### 9.3 AI Difficulty
- Single difficulty level (no Easy/Medium/Hard)
- AI always responds within 5-10 seconds

---

## 10. Technical Requirements

### 10.1 Frontend
| Technology | Purpose |
|------------|---------|
| TypeScript | Primary language |
| React | UI framework |
| WebSocket | Real-time game sync |
| CSS / Styled Components | Styling |
| Canvas or DOM Grid | Game board rendering |

### 10.2 Backend
| Technology | Purpose |
|------------|---------|
| Node.js | Runtime |
| Express | REST API |
| WebSocket (ws) | Real-time communication |
| PostgreSQL | Database |
| JWT | Authentication |
| bcrypt | Password hashing |

### 10.3 Database
| Entity | Description |
|--------|-------------|
| users | User accounts, stats |
| characters | Character definitions |
| cards | Card definitions |
| collections | Collection definitions |
| user_inventory | Cards owned by users |
| gear | Gear definitions |
| user_gear | Gear owned by users |
| matches | Match records |
| match_actions | Turn action history |
| matchmaking_queue | Active queue entries |

### 10.4 Real-Time Protocol (WebSocket)

#### Client → Server Events
| Event | Payload |
|-------|---------|
| `submit_action` | `{matchId, turn, action, data}` |
| `join_match` | `{matchId}` |
| `queue_join` | `{sportType, prizeCardId}` |
| `queue_leave` | `{}` |
| `invite_player` | `{sportType, prizeCardId}` |
| `open_gacha` | `{packId}` |

#### Server → Client Events
| Event | Payload |
|-------|---------|
| `match_found` | `{matchId, opponent, sportType, prizeCard}` |
| `invite_received` | `{playerNickname, matchId, sportType}` |
| `turn_start` | `{turnNumber, timer}` |
| `turn_result` | `{actions, scores, ballPosition}` |
| `game_over` | `{winner, finalScores, prizeCard}` |
| `error` | `{message}` |

---

## 11. Error Handling & Edge Cases

### 11.1 Disconnection
| Scenario | Behavior |
|----------|----------|
| Player disconnects mid-match | Auto-submits random valid action for disconnected player |
| Both disconnect | Match cancelled, no rewards |
| Reconnect within 30 seconds | Resume from last known state |

### 11.2 Simultaneous Win
If both players reach 20 points in the same turn resolution:
- Player with **higher score** wins
- If scores are **equal** → Tiebreaker: random winner

### 11.3 Prize Card Already Owned
- Winner can receive duplicate cards
- Duplicates stack in inventory (quantity +1)

---

## 12. Future Considerations (Not in MVP)

| Feature | Description |
|---------|-------------|
| Chat System | In-match text communication |
| Spectator Mode | Watch ongoing matches |
| Leaderboards | Global rankings |
| Seasons | Time-limited events |
| Achievements | Badges and milestones |
| Multiple Gear Slots | More than one gear per match |
| Character Abilities | Gameplay-affecting character stats |
| Mobile Responsive | Touch-friendly UI |

---

## 13. Glossary

| Term | Definition |
|------|------------|
| **Character** | Player avatar (cosmetic only) |
| **Card** | Collectible item with rarity, part of collections |
| **Collection** | Themed set of 5 cards |
| **Gacha Pack** | Random card pack earned by completing collections |
| **Gear** | Single-use item that doubles points (x2) for one match |
| **Prize Card** | Card wagered on a match, won by the winner |
| **Match** | A single game between two players |
| **Turn** | Simultaneous action phase (15 seconds) |
| **Grid** | 5x5 playing field |

---

*This specification is a living document and will be updated as development progresses.*
