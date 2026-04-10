# Tactical Sports Card Game - Architecture

## 🎮 Game Overview

**Turn-based tactical PvP strategy game** where players compete in sports-themed matches (Football, Basketball, Tennis) on a 5x5 grid. Players simultaneously submit moves to compete for a ball, score points, and win collectible cards as prizes.

---

## 📋 Game Rules Summary

| Rule | Value |
|------|-------|
| Grid Size | 5x5 tiles |
| Win Condition | First to **20 points** |
| Turn Timer | 15 seconds (auto-skip if timeout) |
| Players | 1v1 (or vs AI) |
| Ball | Single ball, both players compete |
| Simultaneous Turns | Both players submit, then results calculated |

### Scoring Actions
| Action | Points |
|--------|--------|
| Ball hits goal | +5 |
| Ball hits teammate | +3 |
| Ball hits opponent | -1 |
| Ball goes out of bounds | -3 |

### Gear System
| Rule | Value |
|------|-------|
| Cost | 10 regular cards = 1 gear item |
| Effect | **Points multiplier x2** |
| Duration | Single use (consumed after match) |
| Types | Football Boots, Basketball Shoes, Tennis Racquet |

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENT (Browser)                      │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │  Game UI    │  │  Card Deck   │  │  User Profile │  │
│  │  (Canvas)   │  │  Manager     │  │  Dashboard    │  │
│  └──────┬──────┘  └──────┬───────┘  └───────┬───────┘  │
│         │                │                   │          │
│         └────────────────┼───────────────────┘          │
│                          │                              │
│                   ┌──────▼──────┐                       │
│                   │   API Client │                      │
│                   │  (Fetch/WS)  │                      │
│                   └──────┬───────┘                       │
└──────────────────────────┼──────────────────────────────┘
                           │ HTTP + WebSocket
┌──────────────────────────┼──────────────────────────────┐
│                    SERVER (Node.js)                      │
│                          │                              │
│     ┌────────────────────┼────────────────────┐         │
│     │                    │                    │         │
│ ┌───▼────┐      ┌───────▼──────┐     ┌───────▼──────┐ │
│ │ Auth    │      │ Matchmaking  │     │  Game Engine │ │
│ │ Service │      │ Service      │     │  (Core Logic)│ │
│ └────────┘      └──────────────┘     └──────┬───────┘ │
│                                             │         │
│     ┌────────────────────┬──────────────────┤         │
│     │                    │                  │         │
│ ┌───▼────┐      ┌───────▼──────┐     ┌─────▼──────┐  │
│ │ Card    │      │  Collection  │     │  AI Engine │  │
│ │ Service │      │  & Gacha     │     │  (Fallback)│  │
│ └────────┘      └──────────────┘     └────────────┘  │
└───────────────────────────────────────────────────────┘
                           │
┌──────────────────────────┼──────────────────────────────┐
│                    DATABASE (PostgreSQL)                 │
│                          │                              │
│     ┌────────────────────┼────────────────────┐         │
│     │                    │                    │         │
│ ┌───▼────┐      ┌───────▼──────┐     ┌───────▼──────┐ │
│ │ users  │      │  matches     │     │  cards       │ │
│ │ table  │      │  table       │     │  table       │ │
│ └────────┘      └──────────────┘     └──────────────┘ │
│                                                        │
│ ┌───────────┐     ┌────────────────┐  ┌──────────────┐│
│ │inventory  │     │  game_actions  │  │  collections ││
│ │table      │     │  table         │  │  table       ││
│ └───────────┘     └────────────────┘  └──────────────┘│
└────────────────────────────────────────────────────────┘
```

---

## 🗄️ Database Schema

### Tables

```sql
-- Users
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nickname VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    gems INT DEFAULT 100,
    level INT DEFAULT 1,
    xp INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Characters (pre-populated with 10 characters)
CREATE TABLE characters (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    sport_type VARCHAR(20) NOT NULL,  -- 'football', 'basketball', 'tennis'
    description TEXT,
    image_url VARCHAR(255),
    is_active BOOLEAN DEFAULT true
);

-- Cards
CREATE TABLE cards (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    sport_type VARCHAR(20) NOT NULL,
    rarity VARCHAR(20) NOT NULL,  -- 'common', 'rare', 'super'
    collection_name VARCHAR(100),  -- e.g., 'Barcelona', 'Wimbledon 2024'
    image_url VARCHAR(255),
    is_super_card BOOLEAN DEFAULT false
);

-- Card Collections (5-card sets)
CREATE TABLE collections (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    sport_type VARCHAR(20) NOT NULL,
    card_ids UUID[] NOT NULL,  -- Array of 5 card IDs
    is_complete BOOLEAN DEFAULT false
);

-- User Inventory (cards owned by user)
CREATE TABLE user_inventory (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    card_id UUID REFERENCES cards(id),
    quantity INT DEFAULT 1,
    acquired_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(user_id, card_id)
);

-- Gear
CREATE TABLE gear (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    sport_type VARCHAR(20) NOT NULL,
    effect VARCHAR(50) DEFAULT 'points_x2',
    regular_cards_cost INT DEFAULT 10,
    description TEXT
);

-- User Gear Inventory
CREATE TABLE user_gear (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    gear_id UUID REFERENCES gear(id),
    quantity INT DEFAULT 1,
    UNIQUE(user_id, gear_id)
);

-- Matches
CREATE TABLE matches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    player1_id UUID REFERENCES users(id),
    player2_id UUID REFERENCES users(id),  -- NULL if vs AI
    sport_type VARCHAR(20) NOT NULL,
    prize_card_id UUID REFERENCES cards(id),
    status VARCHAR(20) DEFAULT 'waiting',  -- 'waiting', 'active', 'finished', 'cancelled'
    winner_id UUID REFERENCES users(id),
    player1_score INT DEFAULT 0,
    player2_score INT DEFAULT 0,
    player1_gear_id UUID REFERENCES gear(id),
    player2_gear_id UUID REFERENCES gear(id),
    created_at TIMESTAMP DEFAULT NOW(),
    started_at TIMESTAMP,
    finished_at TIMESTAMP
);

-- Match Actions (turn submissions)
CREATE TABLE match_actions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    match_id UUID REFERENCES matches(id),
    player_id UUID REFERENCES users(id),
    turn_number INT NOT NULL,
    action_type VARCHAR(50) NOT NULL,  -- 'move', 'kick', 'capture'
    action_data JSONB NOT NULL,  -- {from_tile, to_tile, target_tile}
    submitted_at TIMESTAMP DEFAULT NOW()
);

-- Matchmaking Queue
CREATE TABLE matchmaking_queue (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) UNIQUE,
    sport_type VARCHAR(20),
    prize_card_id UUID REFERENCES cards(id),
    created_at TIMESTAMP DEFAULT NOW(),
    invite_link VARCHAR(255) UNIQUE
);
```

---

## 📁 Project Structure

```
tactical-sports-game/
├── backend/
│   ├── src/
│   │   ├── config/
│   │   │   └── database.ts          # PostgreSQL connection
│   │   ├── services/
│   │   │   ├── auth.service.ts      # Registration, login, JWT
│   │   │   ├── matchmaking.service.ts
│   │   │   ├── game.service.ts      # Core game logic
│   │   │   ├── card.service.ts      # Card operations
│   │   │   ├── gacha.service.ts     # Gacha pack opening
│   │   │   ├── gear.service.ts      # Gear management
│   │   │   └── ai.service.ts        # AI opponent logic
│   │   ├── controllers/
│   │   │   ├── auth.controller.ts
│   │   │   ├── matchmaking.controller.ts
│   │   │   ├── game.controller.ts
│   │   │   ├── card.controller.ts
│   │   │   └── user.controller.ts
│   │   ├── middleware/
│   │   │   ├── auth.middleware.ts   # JWT verification
│   │   │   └── error.middleware.ts
│   │   ├── routes/
│   │   │   ├── auth.routes.ts
│   │   │   ├── matchmaking.routes.ts
│   │   │   ├── game.routes.ts
│   │   │   ├── card.routes.ts
│   │   │   └── user.routes.ts
│   │   ├── websocket/
│   │   │   └── game.handler.ts      # Real-time game sync
│   │   ├── utils/
│   │   │   ├── gameEngine.ts        # Turn resolution
│   │   │   └── scoring.ts           # Point calculations
│   │   └── index.ts                 # Entry point
│   ├── package.json
│   ├── tsconfig.json
│   └── .env
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── GameBoard.tsx        # 5x5 grid rendering
│   │   │   ├── Character.tsx        # Player avatar
│   │   │   ├── Ball.tsx             # Ball component
│   │   │   ├── CardDisplay.tsx      # Prize card display
│   │   │   ├── DeckBuilder.tsx      # Pre-match setup
│   │   │   ├── GearSelector.tsx     # Gear selection
│   │   │   ├── Timer.tsx            # 15s countdown
│   │   │   └── Scoreboard.tsx       # Score display
│   │   ├── screens/
│   │   │   ├── LoginScreen.tsx
│   │   │   ├── Dashboard.tsx        # Main menu
│   │   │   ├── MatchmakingScreen.tsx
│   │   │   ├── GameScreen.tsx       # Active game
│   │   │   ├── CollectionScreen.tsx # Card collection
│   │   │   ├── InventoryScreen.tsx  # Gear & cards
│   │   │   └── GachaScreen.tsx      # Pack opening
│   │   ├── services/
│   │   │   ├── api.ts               # HTTP client
│   │   │   ├── websocket.ts         # WS connection
│   │   │   └── auth.ts
│   │   ├── hooks/
│   │   │   ├── useGame.ts
│   │   │   └── useTimer.ts
│   │   ├── types/
│   │   │   ├── game.types.ts
│   │   │   ├── card.types.ts
│   │   │   └── user.types.ts
│   │   ├── utils/
│   │   │   └── gameLogic.ts
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── package.json
│   ├── tsconfig.json
│   └── index.html
│
└── README.md
```

---

## 🎯 Core Game Flow

### 1. Matchmaking
```
User clicks "Start Game"
    → Select sport type (Football/Basketball/Tennis)
    → Select prize card (3 system options OR own card)
    → System checks queue for matching player
    → If found: Create match, notify both players
    → If not found: Enter queue, play vs AI if timeout
    → Private link: Generate unique invite URL
```

### 2. Match Start
```
Both players join
    → Select character (avatar)
    → Optional: Select gear (x2 multiplier, consumed)
    → Kick-off: Ball placed at center
    → Game begins
```

### 3. Turn Resolution (Simultaneous)
```
Both players submit actions (15s timer)
    → Server receives both actions
    → Resolve conflicts (priority system)
    → Calculate ball trajectory
    → Update scores
    → Check win condition (20 points)
    → If not won: Next turn
    → If won: Award prize card, update user stats
```

### 4. Scoring Resolution
```
Ball result → Determine outcome
    → Goal: +5 points
    → Teammate: +3 points
    → Opponent: -1 point
    → Out of bounds: -3 points
Apply gear multiplier (if active): score *= 2
```

---

## 🔄 WebSocket Events

### Client → Server
| Event | Payload | Description |
|-------|---------|-------------|
| `submit_action` | `{matchId, turn, action, data}` | Submit turn action |
| `join_match` | `{matchId}` | Join a specific match |
| `queue_join` | `{sportType, prizeCardId}` | Enter matchmaking queue |
| `queue_leave` | `{}` | Leave matchmaking queue |
| `invite_player` | `{sportType, prizeCardId}` | Generate invite link |

### Server → Client
| Event | Payload | Description |
|-------|---------|-------------|
| `match_found` | `{matchId, opponent, sportType}` | Match created |
| `invite_received` | `{playerNickname, matchId}` | Invite banner popup |
| `turn_start` | `{turnNumber, timer}` | New turn started |
| `turn_result` | `{actions, scores, ballPosition}` | Results of turn |
| `game_over` | `{winner, finalScores, prizeCard}` | Match ended |
| `error` | `{message}` | Error notification |

---

## 🃏 Card System

### Card Rarities
| Rarity | Drop Rate | Use |
|--------|-----------|-----|
| Common | 80% | Exchange for gear (10 = 1 gear) |
| Rare | 15% | Collectible, higher prestige |
| Super | 5% | Rare collectible, special badge |

### Gacha System

#### How to Earn Gacha Packs
1. **Complete a Collection** — Collect all 5 cards of a specific collection (e.g., Barcelona FC)
2. **Reward** — Unlock 1 Gacha Pack per completed collection
3. **Open Pack** — Rip open the pack to reveal 5 random cards

#### Gacha Pack Contents
| Outcome | Probability |
|---------|-------------|
| 1 Super Card + 4 Regular Cards | 10% |
| 5 Regular Cards (any rarity) | 90% |

*Note: You don't know which cards you'll get until you open the pack!*

### Collection Types (5-card sets)
| Collection | Cards | Sport |
|------------|-------|-------|
| Barcelona FC | 5 players | Football |
| Real Madrid | 5 players | Football |
| Wimbledon 2024 | 5 players | Tennis |
| Wimbledon 2025 | 5 players | Tennis |
| Wimbledon 2026 | 5 players | Tennis |
| NBA Champions | 5 players | Basketball |
| *(expandable)* | 5 players | *any* |

### Regular Card Exchange
| Exchange | Result |
|----------|--------|
| 10 Regular Cards | 1 Gear item (sport-specific) |
| Gear Effect | Points multiplier x2 for one match |
| After Match | Gear is consumed (burned) |

---

## 🔐 Authentication Flow

```
Registration:
  → User provides: email, nickname, password
  → Server validates uniqueness
  → Password hashed (bcrypt)
  → Account created with default gems (100), level 1

Login:
  → Email + password
  → Returns JWT token (24h expiry)
  → Token stored in localStorage

Protected Routes:
  → All API calls require Authorization header
  → WebSocket auth via token on connect
```

---

## 🤖 AI Opponent Logic

```typescript
// Simple AI decision making
function aiTurn(ballPosition: Tile, aiPosition: Tile, opponentPosition: Tile): Action {
    // Priority 1: If ball is close, move to ball
    if (distance(aiPosition, ball_position) <= 2) {
        return { type: 'kick', target: calculateBestTarget() };
    }
    
    // Priority 2: Move towards ball
    return { type: 'move', target: pathTowards(ball_position) };
    
    // calculateBestTarget():
    //   - If has teammate: 70% chance target teammate (+3), 30% goal (+5)
    //   - If no teammate: 80% goal, 20% random opponent (-1)
}
```

---

## 📊 XP & Level Progression

| Level | XP Required | Reward |
|-------|-------------|--------|
| 1 | 0 | Starting level |
| 2 | 100 | Unlock feature |
| 3 | 250 | Unlock feature |
| 4 | 500 | +50 gems |
| 5 | 1000 | Unlock feature |

**XP per match:**
- Win: +50 XP
- Loss: +20 XP
- Draw: +30 XP

---

## 🎨 UI Components Overview

### Main Screens
1. **Login/Register** - Email, nickname, password
2. **Dashboard** - Play, Collection, Inventory, Profile
3. **Matchmaking** - Sport selection, prize card selection
4. **Game Screen** - 5x5 grid, timer, scores, prize card display
5. **Collection** - Card gallery, collection progress
6. **Gacha** - Pack opening animation
7. **Inventory** - Gear management

### In-Game UI
- Prize card banner (top center)
- Score display (top left/right)
- 15s timer (top center)
- Action buttons (bottom)
- Ball position (on grid)
- Character avatars (on grid)

---

## 🚀 Technology Stack Summary

| Layer | Technology |
|-------|------------|
| Frontend | TypeScript + React |
| Backend | Node.js + Express |
| Database | PostgreSQL |
| Real-time | WebSocket (ws library) |
| Auth | JWT + bcrypt |
| Deployment | TBD (Vercel/Render/Heroku) |

---

## 📝 Next Steps

1. ✅ Architecture design
2. ⬜ Set up project structure
3. ⬜ Initialize database schema
4. ⬜ Implement auth system
5. ⬜ Implement matchmaking
6. ⬜ Implement core game engine
7. ⬜ Implement card collection
8. ⬜ Implement gacha system
9. ⬜ Implement gear system
10. ⬜ Build frontend UI components
11. ⬜ Integrate frontend with backend
12. ⬜ Testing & polish
