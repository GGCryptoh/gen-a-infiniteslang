# Infinite Slang — Design Spec

## Overview

A single-page HTML game where players merge Gen Z / Gen Alpha slang terms to discover new words. Inspired by Infinite Craft. Each word has an emoji. Lines connect merged words visually. All combos are hardcoded. LocalStorage persists multiple game saves.

**Stack:** Single `index.html` — no build tools, no frameworks, no dependencies. Pure HTML + CSS + vanilla JS.

**Target:** Desktop + mobile browsers. Responsive. Works offline after first load.

---

## Core Game Loop

1. Player sees 20 base words (Tier 1) as draggable/tappable tiles on a freeform board
2. Merge two words → if valid combo exists, new word + emoji appears with animation
3. New word is added to the board and can be merged with other discovered words
4. Invalid combo → shake animation (toggleable) + occasional roast message (1-in-4 chance, toggleable)
5. Goal: discover all 63 words, especially the 5 Tier-5 mythicals

---

## Merge Tree

**Merge rule:** Any discovered word can merge with any other discovered word. Tier labels indicate when a word *first becomes available* in a typical playthrough, not a restriction on what can combine. Combos are order-independent (A+B and B+A both work).

**Self-merge:** The combo `W + W = Dub` is a special case. Each base word exists as a single tile on the board. To self-merge, the player double-clicks (desktop) or double-taps (mobile) the tile. If the word has a self-merge combo defined, it triggers. The drop zone highlight also appears when dragging a tile back onto itself.

### Tier 1 — Base Words (20, always available)

| Word | Emoji | Meaning |
|------|-------|---------|
| Rizz | 😏 | Charm/charisma |
| Skibidi | 🚽 | Chaotic energy/nonsense |
| Sigma | 🐺 | Lone wolf mentality |
| Cap | 🧢 | Lies |
| Bussin | 🔥 | Really good |
| Slay | 💅 | Killing it |
| Sus | 👀 | Suspicious |
| Bet | 🤝 | Agreement/for sure |
| Gyatt | 😳 | Exclamation of shock |
| Mewing | 🤫 | Jawline thing/staying quiet |
| W | ✅ | Win |
| L | ❌ | Loss |
| Fanum Tax | 🍔 | Taking someone's food/stuff |
| Delulu | 🦄 | Delusional |
| Vibe | 🌊 | Energy/mood |
| NPC | 🤖 | Acting like a robot |
| Ick | 🤢 | Turnoff |
| Ratio | 📉 | Getting more likes than OP |
| Brainrot | 🧠 | Too much internet |
| Aura | ✨ | Reputation points |

### Tier 2 — Two base words combine (20 combos)

| Input A | Input B | Result | Emoji |
|---------|---------|--------|-------|
| Rizz | Sigma | Alpha | 🦁 |
| Rizz | Cap | Catfish | 🐟 |
| Rizz | Slay | Main Character | 🎬 |
| Skibidi | NPC | Bot | 💻 |
| Skibidi | Brainrot | Toilet | 🪠 |
| Bussin | Vibe | Immaculate | 👨‍🍳 |
| Sus | Cap | Snake | 🐍 |
| Sus | Ick | Red Flag | 🚩 |
| W | Rizz | Unspoken Rizz | 🤐 |
| L | Ratio | Cooked | 🍳 |
| L | Delulu | Copium | 💊 |
| Mewing | Sigma | Looksmaxxing | 💪 |
| Gyatt | Aura | Aura Points | 💎 |
| Bet | Slay | Ate | 🍽️ |
| Fanum Tax | W | Finesse | 🎩 |
| Vibe | Brainrot | Chronically Online | 📱 |
| NPC | Cap | Glitch | ⚡ |
| Ick | Delulu | Beige Flag | 🏳️ |
| Aura | Ratio | Clout | 👑 |
| W | W | Dub | 🏆 |

### Tier 3 — Higher-tier merges (12 combos, may include cross-tier inputs)

| Input A | Input B | Result | Emoji |
|---------|---------|--------|-------|
| Alpha | Unspoken Rizz | Giga Chad | 🗿 |
| Main Character | Chronically Online | Content Creator | 📸 |
| Cooked | Snake | Exposed | 📢 |
| Looksmaxxing | Ate | Glow Up | 🦋 |
| Red Flag | Copium | Situationship | 💔 |
| Finesse | Clout | Influencer | 🌟 |
| Bot | Toilet | Skibidi Toilet | 🚽✨ |
| Immaculate | Ate | No Notes | 💯 |
| Chronically Online | Brainrot | iPad Kid | 🧒 |
| Beige Flag | Dub | It's Giving | 💁 |
| Glitch | Red Flag | Gaslighting | 🔥🪞 |
| Aura Points | Finesse | Drip | 💧 |

### Tier 4 — Rare discoveries (6 combos)

| Input A | Input B | Result | Emoji |
|---------|---------|--------|-------|
| Giga Chad | Glow Up | Final Form | ⚔️ |
| Content Creator | Influencer | Mr. Beast | 🦁👑 |
| Exposed | Gaslighting | Cancelled | 🚫 |
| Situationship | It's Giving | Toxic | ☠️ |
| Skibidi Toilet | iPad Kid | Gen Alpha | 👶📱 |
| No Notes | Drip | Iconic | 🏛️ |

### Tier 5 — Mythical (5 combos)

| Input A | Input B | Result | Emoji |
|---------|---------|--------|-------|
| Final Form | Iconic | GOAT | 🐐 |
| Cancelled | Toxic | Redemption Arc | 🌅 |
| Gen Alpha | Mr. Beast | Brainrot King | 👑🧠 |
| GOAT | Redemption Arc | Legend | ⭐ |
| Legend | Brainrot King | Touch Grass | 🌱 |

**Total: 63 words (20 + 20 + 12 + 6 + 5)**

---

## Board & Tiles

- Full viewport canvas area where word tiles float freely
- Each tile displays: **emoji + word** (e.g. "😏 Rizz")
- Tiles are color-coded by tier:
  - **T1:** Neutral/white border
  - **T2:** Blue glow
  - **T3:** Purple glow
  - **T4:** Gold glow
  - **T5:** Rainbow/animated border
- **Connection lines:** SVG lines connect parent words to their child result. When "Alpha" is discovered from "Rizz + Sigma", thin lines draw from both parents to the new word.
- Tiles are freely draggable to organize the board
- **Initial layout:** On new game, 20 base words are arranged in a centered grid (4 columns × 5 rows) with ~120px spacing. Board is centered in viewport.
- **New discovery spawn:** Appears at the midpoint between the two parent tiles, offset slightly downward. If the midpoint is off-screen, clamp to visible viewport with 60px padding from edges.
- T4+ discoveries get confetti/particle effect

---

## Merge Interaction

### Desktop (drag-and-drop)
1. Drag a tile toward another tile
2. Drop zone highlights when hovering over any other tile (highlight is always shown — we do NOT reveal whether the combo is valid until drop, to preserve discovery surprise)
3. On release:
   - **Valid:** New word animates in from merge point, lines draw from parents, discovery celebration (scaled by tier)
   - **Invalid:** Tile returns to original position, shake animation, 1-in-4 chance of roast toast message
   - **Already discovered:** If the combo result already exists on the board, show a gentle "Already discovered ✨" toast. No new tile created, dragged tile returns to position.

### Mobile (tap-tap)
1. Tap first tile — highlights with selection ring
2. Tap second tile — triggers merge attempt
3. Same valid/invalid/already-discovered outcomes as desktop
4. Auto-detected based on touch capability
5. **Deselection:** Tapping the selected tile again, or tapping empty board space, cancels the selection

### Roast Messages (on invalid merge, ~25% chance)
- "skill issue 💀"
- "that ain't it fam"
- "down bad combo"
- "not even close bestie"
- "L + ratio + no merge"
- "bro thought 😭"
- "the math ain't mathing"
- "that combo is mid"

---

## Game Saves (localStorage)

- Up to 20 save slots (sufficient for casual play without hitting localStorage limits)
- Auto-named with timestamp, renamable by user
- Each save stores:
  - Discovered words (array of word IDs)
  - Tile positions (x, y per tile)
  - Discovery order (for replay/timeline)
  - Timestamp (created + last modified)
  - Note: connection lines are NOT persisted — they are re-derived from the combo tree + discovered words on load
- UI: sidebar or modal with save list
  - **New Game** — clears board, starts with 20 base words
  - **Load** — restore a saved game
  - **Delete** — remove a save (with confirmation)
  - **Rename** — edit save name
- localStorage key: `infinite-slang-saves`
- **localStorage unavailable/full:** If localStorage throws (quota exceeded, private browsing), show a warning toast "Can't save — storage full or unavailable" and continue gameplay in memory-only mode. Game still works, just doesn't persist.

---

## Settings

Accessed via gear icon (top-right corner). Stored in localStorage (`infinite-slang-settings`).

| Setting | Type | Default |
|---------|------|---------|
| Theme | Select (4 options) | Dark Gradient |
| Shake on miss | Toggle | On |
| Roast messages | Toggle | On |

---

## Themes (4 options, CSS custom properties)

Switching themes is instant — just swaps a `data-theme` attribute on `<html>`.

### Dark Gradient (default)
- **Background:** zinc-950 (#09090b)
- **Tiles:** zinc-800, subtle border, tier-colored glow
- **Text:** White
- **Accents:** Blue → purple → gold gradient per tier

### Pastel Chaos
- **Background:** Soft pink/warm white
- **Tiles:** Rounded, pastel per tier (mint, lavender, peach, butter)
- **Text:** Dark text
- **Accents:** Pink/purple/mint, soft shadows

### Terminal
- **Background:** Pure black (#000)
- **Tiles:** Monospace text, bordered green-on-black
- **Text:** Green (#00ff00) / amber
- **Accents:** Neon green glow, scanline overlay effect

### Meme-core
- **Background:** Bright yellow
- **Tiles:** Bold borders, Comic Sans font
- **Text:** Black
- **Accents:** Rainbow rotating hues, chaotic energy

---

## UI Layout

```
┌─────────────────────────────────────────────┐
│ 🌱 INFINITE SLANG          ⚙️  💾  23/63   │  ← Header
├─────────────────────────────────────────────┤
│                                             │
│     😏 Rizz ──────── 🦁 Alpha              │
│                       /                     │
│     🐺 Sigma ────────                       │
│                                             │
│          🧢 Cap    🔥 Bussin                │
│                                             │
│     💅 Slay    👀 Sus    🤝 Bet             │
│                                             │
│              (freeform tile board)           │
│                                             │
├─────────────────────────────────────────────┤
│ 23/63 ██████░░░░  T1:20 T2:3 T3:0 T4:0 T5:0│  ← Progress bar
└─────────────────────────────────────────────┘
```

- **Header:** Logo/title left, settings gear + save icon + progress counter right
- **Board:** Full remaining viewport, infinite canvas — pan by click-dragging the background (desktop) or two-finger drag (mobile). No zoom in v1. Tile dragging takes priority over panning (if pointer is on a tile, it drags the tile; if on empty space, it pans).
- **Progress bar:** Bottom strip showing discovery count and tier breakdown

---

## Visual Assets (generate during implementation)

- **Favicon:** 32x32 + 64x64, stylized merge/slang icon
- **Logo:** "Infinite Slang" wordmark for header
- **OG Image:** Social sharing preview card (1200x630)
- **Generation tool:** Nano Banana (Gemini image gen)
- **Fallbacks until generated:** Emoji favicon via SVG data URI (🌱), plain text "INFINITE SLANG" header with CSS styling, no OG image (browser default)

---

## Animation Inventory

| Event | Animation |
|-------|-----------|
| Discovery (T1-T3) | Pop-in scale from 0 → 1 with slight bounce |
| Discovery (T4) | Pop-in + gold confetti burst |
| Discovery (T5) | Pop-in + rainbow particle explosion + screen flash |
| Connection lines | Animated draw from parents to child (SVG stroke-dashoffset) |
| Invalid merge | Shake keyframe (translateX ±4px, 300ms) |
| Roast message | Toast slides up from bottom, fades out after 2s |
| Theme switch | Smooth 300ms CSS transition on all custom properties |
| Tile drag | Slight scale-up (1.05) + shadow elevation |
| Tile hover | Subtle glow intensify |

---

## Data Structure (JS)

```js
// Merge tree definition
const WORDS = {
  'rizz': { emoji: '😏', tier: 1, name: 'Rizz' },
  'alpha': { emoji: '🦁', tier: 2, name: 'Alpha' },
  // ...
};

const COMBOS = [
  { a: 'rizz', b: 'sigma', result: 'alpha' },
  // ... order-independent: a+b and b+a both work
];

// Persisted state (saved to localStorage)
const savedState = {
  discovered: ['rizz', 'skibidi', ...],  // word IDs
  positions: { 'rizz': { x: 100, y: 200 }, ... },
  discoveryOrder: ['rizz', 'skibidi', ..., 'alpha'],
  createdAt: 1711000000000,
  updatedAt: 1711000000000,
  name: 'Game 1',
};

// Runtime state (derived on load, NOT persisted)
// connections are re-derived from COMBOS + discovered words
const connections = [{ from: 'rizz', to: 'alpha' }, { from: 'sigma', to: 'alpha' }];
```

---

## Non-Goals (v1)

- No AI-generated combos (all hardcoded)
- No multiplayer
- No server/backend
- No leaderboard
- No sound effects (maybe v2)
- No Remotion video export (maybe v2)
