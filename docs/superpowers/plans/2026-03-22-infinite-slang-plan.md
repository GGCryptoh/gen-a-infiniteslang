# Infinite Slang Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-page HTML merge game where players combine Gen Z/Gen Alpha slang terms to discover 63 words across 5 tiers.

**Architecture:** Single `index.html` file containing all CSS (with 4 theme variants via CSS custom properties), HTML markup, and vanilla JS. No dependencies, no build tools. Game state managed in-memory with localStorage persistence. SVG overlay for connection lines. Touch/pointer event unification for cross-device support.

**Tech Stack:** HTML5, CSS3 (custom properties, keyframes, transitions), vanilla JavaScript (ES2020+), SVG, localStorage API, pointer events API.

**Spec:** `docs/superpowers/specs/2026-03-22-infinite-slang-design.md`

---

## File Structure

Single file: `index.html`

Since this is one file, tasks are organized by **logical section** within that file. Each task builds a self-contained layer that the next task builds on. Tasks 1-4 can be built by separate sub-agents and stitched together; Tasks 5+ integrate them.

---

### Task 1: HTML Shell + CSS Themes + Layout

**Files:**
- Create: `index.html`

This task creates the complete HTML structure, all CSS (including all 4 themes), and the static layout. No JS yet — just the visual skeleton.

- [ ] **Step 1: Create `index.html` with doctype, meta tags, emoji favicon fallback**

```html
<!DOCTYPE html>
<html lang="en" data-theme="dark-gradient">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Infinite Slang</title>
  <link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🌱</text></svg>">
</head>
```

- [ ] **Step 2: Add the complete CSS with all 4 theme variants**

CSS custom properties on `[data-theme]` selectors:
- `dark-gradient`: bg #09090b, tiles zinc-800, white text, tier glows (blue/purple/gold/rainbow)
- `pastel`: bg soft pink (#fff5f5), rounded tiles in pastels, dark text
- `terminal`: bg #000, green monospace, neon glow, scanline overlay
- `memecore`: bg bright yellow (#ffe600), Comic Sans, bold borders, rainbow accents

Include all animation keyframes:
- `@keyframes pop-in` (scale 0→1.1→1, 300ms)
- `@keyframes shake` (translateX ±4px, 300ms)
- `@keyframes rainbow-border` (hue-rotate for T5 tiles)
- `@keyframes confetti-burst` (particle animation)
- `@keyframes line-draw` (stroke-dashoffset for SVG)
- `@keyframes toast-slide` (translateY 100%→0, opacity)
- `@keyframes scanline` (for terminal theme)

Tile styles per tier:
- `.tile` base styles (position absolute, cursor grab, user-select none, transition)
- `.tile[data-tier="1"]` through `.tile[data-tier="5"]` with tier-specific glow/border
- `.tile.dragging` (scale 1.05, elevated shadow, z-index 1000)
- `.tile.selected` (selection ring for mobile tap-tap)
- `.tile.drop-target` (highlight glow when another tile hovers over)
- `.tile.shake` (shake animation class)

Layout styles:
- Header bar (fixed top, flex between logo and controls)
- Board container (fill viewport minus header/footer, overflow hidden, position relative)
- SVG overlay (absolute, full board size, pointer-events none)
- Progress bar (fixed bottom)
- Toast container (fixed bottom-center, above progress bar)
- Settings modal (centered overlay)
- Saves modal (centered overlay)

- [ ] **Step 3: Add HTML structure for all UI regions**

```html
<body>
  <header class="header">
    <div class="logo">🌱 INFINITE SLANG</div>
    <div class="controls">
      <button id="settings-btn" class="icon-btn" title="Settings">⚙️</button>
      <button id="saves-btn" class="icon-btn" title="Saves">💾</button>
      <span id="progress-count" class="progress-count">0/63</span>
    </div>
  </header>

  <main class="board" id="board">
    <svg class="connections-svg" id="connections-svg"></svg>
    <!-- Tiles are dynamically inserted here -->
  </main>

  <footer class="progress-bar" id="progress-bar">
    <div class="progress-fill" id="progress-fill"></div>
    <span class="progress-text" id="progress-text">0/63  T1:0 T2:0 T3:0 T4:0 T5:0</span>
  </footer>

  <div class="toast-container" id="toast-container"></div>

  <!-- Settings Modal -->
  <div class="modal-overlay" id="settings-modal">
    <div class="modal">
      <h2>Settings</h2>
      <div class="setting">
        <label>Theme</label>
        <select id="theme-select">
          <option value="dark-gradient">Dark Gradient</option>
          <option value="pastel">Pastel Chaos</option>
          <option value="terminal">Terminal</option>
          <option value="memecore">Meme-core</option>
        </select>
      </div>
      <div class="setting">
        <label>Shake on miss</label>
        <input type="checkbox" id="shake-toggle" checked>
      </div>
      <div class="setting">
        <label>Roast messages</label>
        <input type="checkbox" id="roast-toggle" checked>
      </div>
      <button class="modal-close" id="settings-close">Done</button>
    </div>
  </div>

  <!-- Saves Modal -->
  <div class="modal-overlay" id="saves-modal">
    <div class="modal">
      <h2>Game Saves</h2>
      <div id="saves-list" class="saves-list"></div>
      <button id="new-game-btn" class="btn">+ New Game</button>
      <button class="modal-close" id="saves-close">Done</button>
    </div>
  </div>

  <script>
    // JS goes here in subsequent tasks
  </script>
</body>
```

- [ ] **Step 4: Verify — open in browser, confirm layout renders**

Open `index.html` in browser. Verify:
- Header with logo, gear, save, counter visible
- Empty board area fills viewport
- Progress bar at bottom
- Settings modal opens/closes (wire click handlers minimally)
- All 4 themes apply correctly when `data-theme` attribute is changed manually in devtools

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: HTML shell with CSS themes, layout, and modals"
```

---

### Task 2: Game Data — Complete Merge Tree

**Files:**
- Modify: `index.html` (inside `<script>` tag)

This task adds all 63 words and 43 combos as hardcoded JS data. No game logic yet — just the data layer.

- [ ] **Step 1: Add WORDS object with all 63 entries**

```js
const WORDS = {
  // Tier 1 (20)
  'rizz': { emoji: '😏', tier: 1, name: 'Rizz' },
  'skibidi': { emoji: '🚽', tier: 1, name: 'Skibidi' },
  'sigma': { emoji: '🐺', tier: 1, name: 'Sigma' },
  'cap': { emoji: '🧢', tier: 1, name: 'Cap' },
  'bussin': { emoji: '🔥', tier: 1, name: 'Bussin' },
  'slay': { emoji: '💅', tier: 1, name: 'Slay' },
  'sus': { emoji: '👀', tier: 1, name: 'Sus' },
  'bet': { emoji: '🤝', tier: 1, name: 'Bet' },
  'gyatt': { emoji: '😳', tier: 1, name: 'Gyatt' },
  'mewing': { emoji: '🤫', tier: 1, name: 'Mewing' },
  'w': { emoji: '✅', tier: 1, name: 'W' },
  'l': { emoji: '❌', tier: 1, name: 'L' },
  'fanum-tax': { emoji: '🍔', tier: 1, name: 'Fanum Tax' },
  'delulu': { emoji: '🦄', tier: 1, name: 'Delulu' },
  'vibe': { emoji: '🌊', tier: 1, name: 'Vibe' },
  'npc': { emoji: '🤖', tier: 1, name: 'NPC' },
  'ick': { emoji: '🤢', tier: 1, name: 'Ick' },
  'ratio': { emoji: '📉', tier: 1, name: 'Ratio' },
  'brainrot': { emoji: '🧠', tier: 1, name: 'Brainrot' },
  'aura': { emoji: '✨', tier: 1, name: 'Aura' },

  // Tier 2 (20)
  'alpha': { emoji: '🦁', tier: 2, name: 'Alpha' },
  'catfish': { emoji: '🐟', tier: 2, name: 'Catfish' },
  'main-character': { emoji: '🎬', tier: 2, name: 'Main Character' },
  'bot': { emoji: '💻', tier: 2, name: 'Bot' },
  'toilet': { emoji: '🪠', tier: 2, name: 'Toilet' },
  'immaculate': { emoji: '👨‍🍳', tier: 2, name: 'Immaculate' },
  'snake': { emoji: '🐍', tier: 2, name: 'Snake' },
  'red-flag': { emoji: '🚩', tier: 2, name: 'Red Flag' },
  'unspoken-rizz': { emoji: '🤐', tier: 2, name: 'Unspoken Rizz' },
  'cooked': { emoji: '🍳', tier: 2, name: 'Cooked' },
  'copium': { emoji: '💊', tier: 2, name: 'Copium' },
  'looksmaxxing': { emoji: '💪', tier: 2, name: 'Looksmaxxing' },
  'aura-points': { emoji: '💎', tier: 2, name: 'Aura Points' },
  'ate': { emoji: '🍽️', tier: 2, name: 'Ate' },
  'finesse': { emoji: '🎩', tier: 2, name: 'Finesse' },
  'chronically-online': { emoji: '📱', tier: 2, name: 'Chronically Online' },
  'glitch': { emoji: '⚡', tier: 2, name: 'Glitch' },
  'beige-flag': { emoji: '🏳️', tier: 2, name: 'Beige Flag' },
  'clout': { emoji: '👑', tier: 2, name: 'Clout' },
  'dub': { emoji: '🏆', tier: 2, name: 'Dub' },

  // Tier 3 (12)
  'giga-chad': { emoji: '🗿', tier: 3, name: 'Giga Chad' },
  'content-creator': { emoji: '📸', tier: 3, name: 'Content Creator' },
  'exposed': { emoji: '📢', tier: 3, name: 'Exposed' },
  'glow-up': { emoji: '🦋', tier: 3, name: 'Glow Up' },
  'situationship': { emoji: '💔', tier: 3, name: 'Situationship' },
  'influencer': { emoji: '🌟', tier: 3, name: 'Influencer' },
  'skibidi-toilet': { emoji: '🚽✨', tier: 3, name: 'Skibidi Toilet' },
  'no-notes': { emoji: '💯', tier: 3, name: 'No Notes' },
  'ipad-kid': { emoji: '🧒', tier: 3, name: 'iPad Kid' },
  'its-giving': { emoji: '💁', tier: 3, name: "It's Giving" },
  'gaslighting': { emoji: '🔥🪞', tier: 3, name: 'Gaslighting' },
  'drip': { emoji: '💧', tier: 3, name: 'Drip' },

  // Tier 4 (6)
  'final-form': { emoji: '⚔️', tier: 4, name: 'Final Form' },
  'mr-beast': { emoji: '🦁👑', tier: 4, name: 'Mr. Beast' },
  'cancelled': { emoji: '🚫', tier: 4, name: 'Cancelled' },
  'toxic': { emoji: '☠️', tier: 4, name: 'Toxic' },
  'gen-alpha': { emoji: '👶📱', tier: 4, name: 'Gen Alpha' },
  'iconic': { emoji: '🏛️', tier: 4, name: 'Iconic' },

  // Tier 5 (5)
  'goat': { emoji: '🐐', tier: 5, name: 'GOAT' },
  'redemption-arc': { emoji: '🌅', tier: 5, name: 'Redemption Arc' },
  'brainrot-king': { emoji: '👑🧠', tier: 5, name: 'Brainrot King' },
  'legend': { emoji: '⭐', tier: 5, name: 'Legend' },
  'touch-grass': { emoji: '🌱', tier: 5, name: 'Touch Grass' },
};
```

- [ ] **Step 2: Add COMBOS array with all 43 merge recipes**

```js
const COMBOS = [
  // Tier 2 (20)
  { a: 'rizz', b: 'sigma', result: 'alpha' },
  { a: 'rizz', b: 'cap', result: 'catfish' },
  { a: 'rizz', b: 'slay', result: 'main-character' },
  { a: 'skibidi', b: 'npc', result: 'bot' },
  { a: 'skibidi', b: 'brainrot', result: 'toilet' },
  { a: 'bussin', b: 'vibe', result: 'immaculate' },
  { a: 'sus', b: 'cap', result: 'snake' },
  { a: 'sus', b: 'ick', result: 'red-flag' },
  { a: 'w', b: 'rizz', result: 'unspoken-rizz' },
  { a: 'l', b: 'ratio', result: 'cooked' },
  { a: 'l', b: 'delulu', result: 'copium' },
  { a: 'mewing', b: 'sigma', result: 'looksmaxxing' },
  { a: 'gyatt', b: 'aura', result: 'aura-points' },
  { a: 'bet', b: 'slay', result: 'ate' },
  { a: 'fanum-tax', b: 'w', result: 'finesse' },
  { a: 'vibe', b: 'brainrot', result: 'chronically-online' },
  { a: 'npc', b: 'cap', result: 'glitch' },
  { a: 'ick', b: 'delulu', result: 'beige-flag' },
  { a: 'aura', b: 'ratio', result: 'clout' },
  { a: 'w', b: 'w', result: 'dub' },  // self-merge

  // Tier 3 (12)
  { a: 'alpha', b: 'unspoken-rizz', result: 'giga-chad' },
  { a: 'main-character', b: 'chronically-online', result: 'content-creator' },
  { a: 'cooked', b: 'snake', result: 'exposed' },
  { a: 'looksmaxxing', b: 'ate', result: 'glow-up' },
  { a: 'red-flag', b: 'copium', result: 'situationship' },
  { a: 'finesse', b: 'clout', result: 'influencer' },
  { a: 'bot', b: 'toilet', result: 'skibidi-toilet' },
  { a: 'immaculate', b: 'ate', result: 'no-notes' },
  { a: 'chronically-online', b: 'brainrot', result: 'ipad-kid' },
  { a: 'beige-flag', b: 'dub', result: 'its-giving' },
  { a: 'glitch', b: 'red-flag', result: 'gaslighting' },
  { a: 'aura-points', b: 'finesse', result: 'drip' },

  // Tier 4 (6)
  { a: 'giga-chad', b: 'glow-up', result: 'final-form' },
  { a: 'content-creator', b: 'influencer', result: 'mr-beast' },
  { a: 'exposed', b: 'gaslighting', result: 'cancelled' },
  { a: 'situationship', b: 'its-giving', result: 'toxic' },
  { a: 'skibidi-toilet', b: 'ipad-kid', result: 'gen-alpha' },
  { a: 'no-notes', b: 'drip', result: 'iconic' },

  // Tier 5 (5)
  { a: 'final-form', b: 'iconic', result: 'goat' },
  { a: 'cancelled', b: 'toxic', result: 'redemption-arc' },
  { a: 'gen-alpha', b: 'mr-beast', result: 'brainrot-king' },
  { a: 'goat', b: 'redemption-arc', result: 'legend' },
  { a: 'legend', b: 'brainrot-king', result: 'touch-grass' },
];

const ROAST_MESSAGES = [
  "skill issue 💀",
  "that ain't it fam",
  "down bad combo",
  "not even close bestie",
  "L + ratio + no merge",
  "bro thought 😭",
  "the math ain't mathing",
  "that combo is mid",
];
```

- [ ] **Step 3: Add combo lookup helper**

```js
// Build lookup map for O(1) combo checks
// Key format: "word1|word2" (alphabetically sorted)
const COMBO_MAP = new Map();
for (const combo of COMBOS) {
  const key = [combo.a, combo.b].sort().join('|');
  COMBO_MAP.set(key, combo.result);
}

function lookupCombo(wordA, wordB) {
  const key = [wordA, wordB].sort().join('|');
  return COMBO_MAP.get(key) || null;
}
```

- [ ] **Step 4: Add BASE_WORDS constant**

```js
const BASE_WORDS = Object.keys(WORDS).filter(id => WORDS[id].tier === 1);
```

- [ ] **Step 5: Verify — open console, confirm `Object.keys(WORDS).length === 63` and `COMBOS.length === 43`**

Open browser console. Run:
- `Object.keys(WORDS).length` → should be 63
- `COMBOS.length` → should be 43
- `lookupCombo('rizz', 'sigma')` → should be 'alpha'
- `lookupCombo('sigma', 'rizz')` → should be 'alpha' (order-independent)
- `lookupCombo('w', 'w')` → should be 'dub' (self-merge)
- `lookupCombo('rizz', 'rizz')` → should be null

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add complete merge tree data (63 words, 43 combos)"
```

---

### Task 3: Game State + localStorage

**Files:**
- Modify: `index.html` (inside `<script>` tag, after data)

State management, save/load, settings persistence.

- [ ] **Step 1: Add state object and initialization**

```js
let gameState = {
  discovered: [],
  positions: {},
  discoveryOrder: [],
  createdAt: Date.now(),
  updatedAt: Date.now(),
  name: '',
};

let settings = {
  theme: 'dark-gradient',
  shakeOnMiss: true,
  roastMessages: true,
};

let currentSaveIndex = -1; // -1 = unsaved new game

function newGame() {
  const now = Date.now();
  gameState = {
    discovered: [...BASE_WORDS],
    positions: generateInitialPositions(),
    discoveryOrder: [...BASE_WORDS],
    createdAt: now,
    updatedAt: now,
    name: `Game ${new Date(now).toLocaleString()}`,
  };
  currentSaveIndex = -1;
}
```

- [ ] **Step 2: Add initial position generator (4×5 centered grid)**

```js
function generateInitialPositions() {
  const positions = {};
  const cols = 4;
  const spacingX = 160;
  const spacingY = 80;
  const startX = (window.innerWidth - (cols - 1) * spacingX) / 2;
  const startY = 80;

  BASE_WORDS.forEach((id, i) => {
    const col = i % cols;
    const row = Math.floor(i / cols);
    positions[id] = {
      x: startX + col * spacingX,
      y: startY + row * spacingY,
    };
  });
  return positions;
}
```

- [ ] **Step 3: Add localStorage helpers with error handling**

```js
function safeGetStorage(key) {
  try {
    const raw = localStorage.getItem(key);
    return raw ? JSON.parse(raw) : null;
  } catch { return null; }
}

function safeSetStorage(key, value) {
  try {
    localStorage.setItem(key, JSON.stringify(value));
    return true;
  } catch {
    showToast("Can't save — storage full or unavailable");
    return false;
  }
}

function loadSettings() {
  const saved = safeGetStorage('infinite-slang-settings');
  if (saved) Object.assign(settings, saved);
  applySettings();
}

function saveSettings() {
  safeSetStorage('infinite-slang-settings', settings);
}

function getSaves() {
  return safeGetStorage('infinite-slang-saves') || [];
}

function saveGame() {
  gameState.updatedAt = Date.now();
  const saves = getSaves();
  if (currentSaveIndex >= 0) {
    saves[currentSaveIndex] = { ...gameState };
  } else {
    saves.push({ ...gameState });
    currentSaveIndex = saves.length - 1;
  }
  if (saves.length > 20) saves.shift(); // cap at 20
  safeSetStorage('infinite-slang-saves', saves);
}

function loadGame(index) {
  const saves = getSaves();
  if (saves[index]) {
    gameState = { ...saves[index] };
    currentSaveIndex = index;
    renderBoard();
  }
}

function deleteGame(index) {
  const saves = getSaves();
  saves.splice(index, 1);
  safeSetStorage('infinite-slang-saves', saves);
  if (currentSaveIndex === index) currentSaveIndex = -1;
  else if (currentSaveIndex > index) currentSaveIndex--;
}
```

- [ ] **Step 4: Add settings application**

```js
function applySettings() {
  document.documentElement.setAttribute('data-theme', settings.theme);
  document.getElementById('theme-select').value = settings.theme;
  document.getElementById('shake-toggle').checked = settings.shakeOnMiss;
  document.getElementById('roast-toggle').checked = settings.roastMessages;
}
```

- [ ] **Step 5: Verify — in console, run `newGame(); console.log(gameState.discovered.length)` → should be 20**

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: game state management with localStorage saves"
```

---

### Task 4: Board Rendering + Tiles + SVG Connection Lines

**Files:**
- Modify: `index.html` (inside `<script>`)

Renders tiles on the board and SVG lines between parents and children.

- [ ] **Step 1: Add tile creation and rendering**

```js
function createTileElement(wordId) {
  const word = WORDS[wordId];
  const pos = gameState.positions[wordId];
  const el = document.createElement('div');
  el.className = 'tile';
  el.dataset.word = wordId;
  el.dataset.tier = word.tier;
  el.textContent = `${word.emoji} ${word.name}`;
  el.style.left = `${pos.x}px`;
  el.style.top = `${pos.y}px`;
  return el;
}

function renderBoard() {
  const board = document.getElementById('board');
  // Remove old tiles (keep SVG)
  board.querySelectorAll('.tile').forEach(el => el.remove());

  // Add tiles for all discovered words
  for (const wordId of gameState.discovered) {
    const el = createTileElement(wordId);
    board.appendChild(el);
  }

  renderConnections();
  updateProgress();
}
```

- [ ] **Step 2: Add connection line rendering (SVG)**

```js
function deriveConnections() {
  const connections = [];
  const discovered = new Set(gameState.discovered);
  for (const combo of COMBOS) {
    if (discovered.has(combo.result) && discovered.has(combo.a) && discovered.has(combo.b)) {
      connections.push({ from: combo.a, to: combo.result });
      if (combo.a !== combo.b) {
        connections.push({ from: combo.b, to: combo.result });
      }
    }
  }
  return connections;
}

function renderConnections() {
  const svg = document.getElementById('connections-svg');
  svg.innerHTML = '';
  const connections = deriveConnections();

  for (const conn of connections) {
    const fromPos = gameState.positions[conn.from];
    const toPos = gameState.positions[conn.to];
    if (!fromPos || !toPos) continue;

    const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
    // Offset to center of tile (approximate)
    line.setAttribute('x1', fromPos.x + 60);
    line.setAttribute('y1', fromPos.y + 20);
    line.setAttribute('x2', toPos.x + 60);
    line.setAttribute('y2', toPos.y + 20);
    line.classList.add('connection-line');
    svg.appendChild(line);
  }
}
```

- [ ] **Step 3: Add progress bar update**

```js
function updateProgress() {
  const total = Object.keys(WORDS).length;
  const found = gameState.discovered.length;
  const tiers = [0, 0, 0, 0, 0];
  for (const id of gameState.discovered) {
    tiers[WORDS[id].tier - 1]++;
  }

  document.getElementById('progress-count').textContent = `${found}/${total}`;
  document.getElementById('progress-fill').style.width = `${(found / total) * 100}%`;
  document.getElementById('progress-text').textContent =
    `${found}/${total}  T1:${tiers[0]} T2:${tiers[1]} T3:${tiers[2]} T4:${tiers[3]} T5:${tiers[4]}`;
}
```

- [ ] **Step 4: Add toast system**

```js
function showToast(message, duration = 2000) {
  const container = document.getElementById('toast-container');
  const toast = document.createElement('div');
  toast.className = 'toast';
  toast.textContent = message;
  container.appendChild(toast);
  requestAnimationFrame(() => toast.classList.add('show'));
  setTimeout(() => {
    toast.classList.remove('show');
    setTimeout(() => toast.remove(), 300);
  }, duration);
}
```

- [ ] **Step 5: Verify — call `newGame(); renderBoard();` in console. 20 tiles should appear in 4×5 grid. Progress shows 20/63.**

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: board rendering with tiles, SVG lines, and progress bar"
```

---

### Task 5: Drag-and-Drop + Tap-Tap Merge Interaction

**Files:**
- Modify: `index.html` (inside `<script>`)

The core interaction layer. Handles pointer events for drag (desktop) and tap-tap (mobile), merge logic, discovery animations, and the self-merge double-click/tap.

- [ ] **Step 1: Add pointer event system for tile dragging**

Use pointer events (unified touch/mouse). Track drag state. On pointerdown on a tile, begin drag. On pointermove, update tile position. On pointerup, check if over another tile.

```js
let dragState = null; // { wordId, startX, startY, offsetX, offsetY, el }
let selectedTile = null; // for tap-tap on mobile
let isTouchDevice = false;
let lastTapTime = 0;
let lastTapTarget = null;

function initInteraction() {
  const board = document.getElementById('board');
  isTouchDevice = 'ontouchstart' in window;

  board.addEventListener('pointerdown', onPointerDown);
  board.addEventListener('pointermove', onPointerMove);
  board.addEventListener('pointerup', onPointerUp);
  board.addEventListener('pointercancel', onPointerCancel);
}
```

- [ ] **Step 2: Implement pointer handlers with drag vs tap detection**

```js
function onPointerDown(e) {
  const tile = e.target.closest('.tile');
  if (!tile) {
    // Clicked empty space — deselect if tap-tap mode
    if (selectedTile) {
      selectedTile.classList.remove('selected');
      selectedTile = null;
    }
    return;
  }

  const wordId = tile.dataset.word;
  const rect = tile.getBoundingClientRect();

  // Double-tap/click detection for self-merge
  const now = Date.now();
  if (lastTapTarget === wordId && now - lastTapTime < 400) {
    attemptMerge(wordId, wordId);
    lastTapTarget = null;
    lastTapTime = 0;
    return;
  }
  lastTapTarget = wordId;
  lastTapTime = now;

  if (isTouchDevice) {
    // Tap-tap mode
    if (!selectedTile) {
      selectedTile = tile;
      tile.classList.add('selected');
    } else if (selectedTile === tile) {
      tile.classList.remove('selected');
      selectedTile = null;
    } else {
      const wordA = selectedTile.dataset.word;
      selectedTile.classList.remove('selected');
      selectedTile = null;
      attemptMerge(wordA, wordId);
    }
  } else {
    // Drag mode
    tile.setPointerCapture(e.pointerId);
    dragState = {
      wordId,
      el: tile,
      startX: gameState.positions[wordId].x,
      startY: gameState.positions[wordId].y,
      offsetX: e.clientX - rect.left,
      offsetY: e.clientY - rect.top,
    };
    tile.classList.add('dragging');
  }
}

function onPointerMove(e) {
  if (!dragState) return;
  const x = e.clientX - dragState.offsetX;
  const y = e.clientY - dragState.offsetY;
  dragState.el.style.left = `${x}px`;
  dragState.el.style.top = `${y}px`;
  gameState.positions[dragState.wordId] = { x, y };

  // Highlight drop targets
  document.querySelectorAll('.tile.drop-target').forEach(t => t.classList.remove('drop-target'));
  const target = getDropTarget(e.clientX, e.clientY, dragState.el);
  if (target) target.classList.add('drop-target');

  renderConnections(); // Update lines while dragging
}

function onPointerUp(e) {
  if (!dragState) return;
  dragState.el.classList.remove('dragging');
  document.querySelectorAll('.tile.drop-target').forEach(t => t.classList.remove('drop-target'));

  const target = getDropTarget(e.clientX, e.clientY, dragState.el);
  if (target) {
    const wordB = target.dataset.word;
    // Return tile to start position before merge attempt
    dragState.el.style.left = `${dragState.startX}px`;
    dragState.el.style.top = `${dragState.startY}px`;
    gameState.positions[dragState.wordId] = { x: dragState.startX, y: dragState.startY };
    attemptMerge(dragState.wordId, wordB);
  }
  // If no target, tile stays where dropped (free positioning)
  renderConnections();
  saveGame();
  dragState = null;
}

function onPointerCancel() {
  if (dragState) {
    dragState.el.classList.remove('dragging');
    dragState.el.style.left = `${dragState.startX}px`;
    dragState.el.style.top = `${dragState.startY}px`;
    gameState.positions[dragState.wordId] = { x: dragState.startX, y: dragState.startY };
    dragState = null;
  }
}

function getDropTarget(clientX, clientY, excludeEl) {
  const tiles = document.querySelectorAll('.tile');
  for (const tile of tiles) {
    if (tile === excludeEl) continue;
    const rect = tile.getBoundingClientRect();
    if (clientX >= rect.left && clientX <= rect.right &&
        clientY >= rect.top && clientY <= rect.bottom) {
      return tile;
    }
  }
  return null;
}
```

- [ ] **Step 3: Implement merge attempt logic**

```js
function attemptMerge(wordA, wordB) {
  const result = lookupCombo(wordA, wordB);

  if (!result) {
    // Invalid merge
    if (settings.shakeOnMiss) {
      const tileA = document.querySelector(`[data-word="${wordA}"]`);
      const tileB = document.querySelector(`[data-word="${wordB}"]`);
      if (tileA) { tileA.classList.add('shake'); setTimeout(() => tileA.classList.remove('shake'), 300); }
      if (tileB && wordA !== wordB) { tileB.classList.add('shake'); setTimeout(() => tileB.classList.remove('shake'), 300); }
    }
    if (settings.roastMessages && Math.random() < 0.25) {
      showToast(ROAST_MESSAGES[Math.floor(Math.random() * ROAST_MESSAGES.length)]);
    }
    return;
  }

  if (gameState.discovered.includes(result)) {
    showToast('Already discovered ✨');
    return;
  }

  // Valid new discovery!
  discoverWord(result, wordA, wordB);
}
```

- [ ] **Step 4: Implement discovery with spawn position and animation**

```js
function discoverWord(resultId, parentA, parentB) {
  const word = WORDS[resultId];

  // Calculate spawn position (midpoint of parents, offset down)
  const posA = gameState.positions[parentA];
  const posB = gameState.positions[parentB];
  let spawnX = (posA.x + posB.x) / 2;
  let spawnY = (posA.y + posB.y) / 2 + 40;

  // Clamp to visible viewport with 60px padding
  const vw = window.innerWidth;
  const vh = window.innerHeight;
  spawnX = Math.max(60, Math.min(vw - 180, spawnX));
  spawnY = Math.max(60, Math.min(vh - 120, spawnY));

  gameState.discovered.push(resultId);
  gameState.discoveryOrder.push(resultId);
  gameState.positions[resultId] = { x: spawnX, y: spawnY };
  gameState.updatedAt = Date.now();

  // Create and animate the tile
  const el = createTileElement(resultId);
  el.classList.add('pop-in');
  document.getElementById('board').appendChild(el);

  // Tier-based celebration
  if (word.tier >= 4) {
    spawnConfetti(spawnX + 60, spawnY + 20, word.tier);
  }
  if (word.tier === 5) {
    document.body.classList.add('screen-flash');
    setTimeout(() => document.body.classList.remove('screen-flash'), 500);
  }

  renderConnections();
  updateProgress();
  saveGame();
}
```

- [ ] **Step 5: Add confetti burst for T4/T5**

```js
function spawnConfetti(x, y, tier) {
  const colors = tier === 5
    ? ['#ff0000', '#ff8800', '#ffff00', '#00ff00', '#0088ff', '#8800ff']
    : ['#ffd700', '#ffaa00', '#ff8800'];
  const count = tier === 5 ? 30 : 15;

  for (let i = 0; i < count; i++) {
    const particle = document.createElement('div');
    particle.className = 'confetti';
    particle.style.left = `${x}px`;
    particle.style.top = `${y}px`;
    particle.style.backgroundColor = colors[i % colors.length];
    particle.style.setProperty('--angle', `${Math.random() * 360}deg`);
    particle.style.setProperty('--distance', `${60 + Math.random() * 80}px`);
    particle.style.setProperty('--rotation', `${Math.random() * 720}deg`);
    document.body.appendChild(particle);
    setTimeout(() => particle.remove(), 1000);
  }
}
```

- [ ] **Step 6: Verify — open in browser, drag "Rizz" onto "Sigma". "Alpha" should pop in with 🦁, lines should draw. Try an invalid merge — tile should shake. Double-click "W" — "Dub" should appear.**

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: drag-drop and tap-tap merge interaction with animations"
```

---

### Task 6: Settings Modal + Saves Modal + Event Wiring

**Files:**
- Modify: `index.html` (inside `<script>`)

Wire up the settings and saves modals, theme switching, and the init sequence.

- [ ] **Step 1: Wire settings modal**

```js
function initModals() {
  // Settings
  document.getElementById('settings-btn').addEventListener('click', () => {
    document.getElementById('settings-modal').classList.add('open');
  });
  document.getElementById('settings-close').addEventListener('click', () => {
    document.getElementById('settings-modal').classList.remove('open');
  });
  document.getElementById('theme-select').addEventListener('change', (e) => {
    settings.theme = e.target.value;
    applySettings();
    saveSettings();
  });
  document.getElementById('shake-toggle').addEventListener('change', (e) => {
    settings.shakeOnMiss = e.target.checked;
    saveSettings();
  });
  document.getElementById('roast-toggle').addEventListener('change', (e) => {
    settings.roastMessages = e.target.checked;
    saveSettings();
  });

  // Saves
  document.getElementById('saves-btn').addEventListener('click', () => {
    renderSavesList();
    document.getElementById('saves-modal').classList.add('open');
  });
  document.getElementById('saves-close').addEventListener('click', () => {
    document.getElementById('saves-modal').classList.remove('open');
  });
  document.getElementById('new-game-btn').addEventListener('click', () => {
    newGame();
    renderBoard();
    document.getElementById('saves-modal').classList.remove('open');
  });

  // Close modals on overlay click
  document.querySelectorAll('.modal-overlay').forEach(overlay => {
    overlay.addEventListener('click', (e) => {
      if (e.target === overlay) overlay.classList.remove('open');
    });
  });
}
```

- [ ] **Step 2: Add saves list rendering with load/delete/rename**

```js
function renderSavesList() {
  const saves = getSaves();
  const list = document.getElementById('saves-list');
  list.innerHTML = '';

  if (saves.length === 0) {
    list.innerHTML = '<p class="empty-saves">No saves yet. Start playing!</p>';
    return;
  }

  saves.forEach((save, i) => {
    const item = document.createElement('div');
    item.className = 'save-item';
    if (i === currentSaveIndex) item.classList.add('active');

    const discovered = save.discovered ? save.discovered.length : 0;
    const date = new Date(save.updatedAt).toLocaleString();

    item.innerHTML = `
      <div class="save-info">
        <span class="save-name" contenteditable="true" data-index="${i}">${save.name || 'Unnamed'}</span>
        <span class="save-meta">${discovered}/63 — ${date}</span>
      </div>
      <div class="save-actions">
        <button class="btn-sm load-btn" data-index="${i}">Load</button>
        <button class="btn-sm delete-btn" data-index="${i}">🗑️</button>
      </div>
    `;
    list.appendChild(item);
  });

  // Wire load buttons
  list.querySelectorAll('.load-btn').forEach(btn => {
    btn.addEventListener('click', () => {
      loadGame(parseInt(btn.dataset.index));
      document.getElementById('saves-modal').classList.remove('open');
    });
  });

  // Wire delete buttons
  list.querySelectorAll('.delete-btn').forEach(btn => {
    btn.addEventListener('click', () => {
      if (confirm('Delete this save?')) {
        deleteGame(parseInt(btn.dataset.index));
        renderSavesList();
      }
    });
  });

  // Wire rename (contenteditable blur)
  list.querySelectorAll('.save-name').forEach(el => {
    el.addEventListener('blur', () => {
      const idx = parseInt(el.dataset.index);
      const saves = getSaves();
      if (saves[idx]) {
        saves[idx].name = el.textContent.trim();
        safeSetStorage('infinite-slang-saves', saves);
        if (idx === currentSaveIndex) gameState.name = el.textContent.trim();
      }
    });
    el.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') { e.preventDefault(); el.blur(); }
    });
  });
}
```

- [ ] **Step 3: Add game initialization / boot sequence**

```js
function init() {
  loadSettings();
  initModals();
  initInteraction();

  // Try to load most recent save, otherwise new game
  const saves = getSaves();
  if (saves.length > 0) {
    loadGame(saves.length - 1);
  } else {
    newGame();
    renderBoard();
  }
}

// Boot
document.addEventListener('DOMContentLoaded', init);
```

- [ ] **Step 4: Verify full flow**

Open `index.html` fresh:
1. Should show 20 base words in grid
2. Drag Rizz onto Sigma → Alpha appears with line
3. Open settings → switch theme to Terminal → board goes green/black
4. Open saves → save shows current game with progress
5. Click New Game → fresh 20 words
6. Load previous save → Alpha is back
7. On mobile/touch: tap Rizz, tap Sigma → Alpha
8. Tap empty space → deselects
9. Double-click W → Dub appears

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: settings, saves modals, and full game boot sequence"
```

---

### Task 7: Polish — Animations, Confetti CSS, Theme Refinement

**Files:**
- Modify: `index.html` (CSS section)

Final CSS polish pass — ensure all animations are smooth, confetti particles work, theme colors are dialed in.

- [ ] **Step 1: Add confetti particle CSS**

```css
.confetti {
  position: fixed;
  width: 8px;
  height: 8px;
  border-radius: 2px;
  pointer-events: none;
  z-index: 9999;
  animation: confetti-fly 1s ease-out forwards;
}

@keyframes confetti-fly {
  0% { transform: translate(0, 0) rotate(0deg); opacity: 1; }
  100% {
    transform: translate(
      calc(cos(var(--angle)) * var(--distance)),
      calc(sin(var(--angle)) * var(--distance) - 40px)
    ) rotate(var(--rotation));
    opacity: 0;
  }
}
```

- [ ] **Step 2: Add screen flash for T5 discoveries**

```css
.screen-flash {
  animation: flash 0.5s ease-out;
}
@keyframes flash {
  0% { filter: brightness(2); }
  100% { filter: brightness(1); }
}
```

- [ ] **Step 3: Add pop-in animation class**

```css
.tile.pop-in {
  animation: pop-in 0.4s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}
@keyframes pop-in {
  0% { transform: scale(0); opacity: 0; }
  70% { transform: scale(1.1); }
  100% { transform: scale(1); opacity: 1; }
}
```

- [ ] **Step 4: Ensure connection line animation (stroke-dashoffset)**

```css
.connection-line {
  stroke: var(--line-color);
  stroke-width: 1.5;
  opacity: 0.3;
  stroke-dasharray: 1000;
  stroke-dashoffset: 1000;
  animation: line-draw 0.6s ease-out forwards;
}
@keyframes line-draw {
  to { stroke-dashoffset: 0; }
}
```

- [ ] **Step 5: Verify all animations play correctly across all 4 themes**

Switch through each theme and verify:
- Tile glows match spec colors per tier
- Connection lines visible
- Confetti spawns on T4 discovery
- Screen flash on T5 discovery
- Shake animation on invalid merge
- Toast slides up and fades
- Terminal theme has scanline overlay

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: polish animations, confetti, and theme refinement"
```

---

### Task 8: Board Panning (Desktop + Mobile)

**Files:**
- Modify: `index.html` (inside `<script>`)

Add infinite canvas panning. Background click-drag pans on desktop. Two-finger drag on mobile.

- [ ] **Step 1: Add pan state and handlers**

```js
let panState = null; // { startX, startY, scrollX, scrollY }
let boardOffset = { x: 0, y: 0 };

function initPanning() {
  const board = document.getElementById('board');

  board.addEventListener('pointerdown', (e) => {
    // Only pan if clicking empty board space (not a tile)
    if (e.target.closest('.tile')) return;
    if (selectedTile) {
      selectedTile.classList.remove('selected');
      selectedTile = null;
      return;
    }
    panState = { startX: e.clientX, startY: e.clientY, offsetX: boardOffset.x, offsetY: boardOffset.y };
    board.style.cursor = 'grabbing';
  }, true); // capture phase so it fires before tile handlers

  board.addEventListener('pointermove', (e) => {
    if (!panState || dragState) return;
    const dx = e.clientX - panState.startX;
    const dy = e.clientY - panState.startY;
    boardOffset.x = panState.offsetX + dx;
    boardOffset.y = panState.offsetY + dy;
    applyBoardOffset();
  });

  board.addEventListener('pointerup', () => {
    if (panState) {
      panState = null;
      board.style.cursor = '';
    }
  });
}

function applyBoardOffset() {
  const board = document.getElementById('board');
  board.style.transform = `translate(${boardOffset.x}px, ${boardOffset.y}px)`;
}
```

Note: The panning approach uses CSS transform on the board container. Tile positions remain in absolute coordinates relative to the board origin. This keeps tile dragging and panning independent.

- [ ] **Step 2: Verify — click-drag on empty board space pans the canvas. Tiles stay in place relative to each other. Tile dragging still works.**

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: infinite canvas panning on desktop and mobile"
```

---

## Summary

| Task | Description | Depends On |
|------|-------------|------------|
| 1 | HTML shell + CSS themes + layout | — |
| 2 | Game data (63 words, 43 combos) | — |
| 3 | Game state + localStorage | Task 2 |
| 4 | Board rendering + tiles + SVG lines | Tasks 1, 2, 3 |
| 5 | Drag-drop + tap-tap merge interaction | Tasks 1-4 |
| 6 | Settings + saves modals + event wiring | Tasks 1-5 |
| 7 | Polish — animations, confetti, themes | Tasks 1-6 |
| 8 | Board panning | Tasks 1-6 |

**Tasks 1 and 2 can run in parallel.** Task 3 depends on 2. Tasks 4+ are sequential.

**Total: 8 tasks, ~35 steps, single file output.**
