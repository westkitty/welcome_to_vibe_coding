# DexGate Academy: Maximum Drama Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transform the existing v8 single-file HTML coding academy into a premium neon-cyberpunk experience with amplified visuals, elevated games, a reactive Dexter character, and a 5-question tool-recommendation funnel — all in one HTML file with no external dependencies.

**Architecture:** Work on a *copy* (`v9`) of the verified v8 file so v8 stays as a known-good fallback. All new CSS goes into a single appended `<style id="drama-css">` block before `</head>` (the existing CSS lines 7–48 are minified one-liners — do **not** edit them in place; override via the new block using equal-or-higher specificity). All new JavaScript goes **inside the existing IIFE**, inserted immediately before the init line (`preloadDex(); populateProjects(); ...`), so it can access `state`, `save`, `speak`, `setDexImage`, `pageDefs`, and the game objects. Behavior is added by (a) new helper functions and (b) small, surgical edits to four named functions (`unlockGate`, `switchPage`, `render`, and each game's win path) using unique-substring anchors.

**Tech Stack:** Vanilla HTML + CSS (custom properties, keyframes, `@media`, `prefers-reduced-motion`) + vanilla JS (Canvas 2D, `requestAnimationFrame`, `localStorage`). No libraries, no build step, no external assets (all images already base64-embedded). New DOM is built with `createElement` + `textContent` (never `innerHTML`) to stay XSS-safe.

---

## Source-of-Truth Reference (the v8 file you are modifying)

These identifiers already exist in `vibe-coding-kit-dexgate-uiux-v8-verified.html` and are reused throughout this plan. Confirm them before editing.

**CSS custom properties (`:root`, line 8):** `--bg #070a11`, `--bg2 #0d1220`, `--panel #101724`, `--panel2 #141d2c`, `--line #273248`, `--line2 #374663`, `--text #eef4ff`, `--muted #a7b4ca`, `--cyan #00e5ff`, `--pink #ff007f`, `--violet #8759ff`, `--red #ff5567`, `--gold #f6c…` (plus `--touch`, `--radius`, `--shadow`).

**Dexter expressions (8, `DEXTER_IMAGES`, line 355):** `neutral`, `annoyed`, `smug`, `unimpressed`, `correct`, `happy`, `friendly`, `bored`. Set via `setDexImage(expression, elementId)`.

**State (`STORE='dexgate_academy_state_v5'`, line 368; `load()` line 428):** `{welcomeSeen, gates:{}, secrets:[], title, sound, longTrails, lastExpression}`.

**Pages (`pageDefs`, line 369):** `start`(Spark Fetch), `tooling`(Floppy Dex), `pricing`(Bonewall Defense), `beginner`(Dexter Hop), `linux`(Crostini Bridge, `elastic:true`), `pro`(Bug Sweep Boss).

**Core functions:** `speak(text,expr)` 432 · `toast(title,text,expr)` 433 · `modal(title,text,expr)` 434 · `unlockGate(page,gameName,fast)` 440 · `switchPage(id)` 444 · `render()` 445 · `setOverlayHidden(id,hidden)` 438 · `beep(freq,dur,type)` 446 · `roundRect` 448 · `canvasPoint` 449 · `drawDexSprite` 450 · `drawStar` 451 · `commonCanvas(id)` 455.

**Games (each returns `{start, stop}`, built in `ensureGames()` line 633):** `SparkFetch('start')` 457 · `FloppyDex('tooling')` 475 · `BonewallDefense('pricing')` 513 · `DexterHop('beginner')` 532 · `CrostiniBridge('linux')` 617 · `BugSweep('pro')` 625. Each calls `unlockGate(id, gameName)` on win.

**Per-page HTML:** `<section class="page" id="page-X">` → `.gameCard` containing `<canvas id="game-X">` + `.gateOverlay#overlay-X`, then `.panel.content.locked[data-content="X"]`. Right sidebar: `.dexPanel` (`#dexImage`, `#dexBubble`, status spans), `.trophies`, `.footerBox`. Welcome `#welcomeOverlay`, modal `#modal`, toast `#toast`.

**Insertion anchors (memorize these two):**
- **New CSS** → immediately before `</head>` (line 49).
- **New JS** → immediately before the init line that begins `preloadDex(); populateProjects(); ensureGames(); render();` (line 662, inside the IIFE).

---

## Verification Model (read before starting)

This is a single HTML file with Canvas games — there is no unit-test framework. Verification uses the **preview_* browser tools**:

- `preview_start` with the v9 file → loads it in a headless browser.
- `preview_console_logs` → must show **zero** errors after every change.
- `preview_eval` → call functions / read state to assert **pure-logic** behavior (this is our "unit test" for the recommendation engine and answer storage). Expected return values are given explicitly.
- `preview_snapshot` → assert DOM structure/visibility (e.g. a modal is open, content is unlocked).
- `preview_screenshot` → capture visual proof for glow/animation tasks.
- `preview_inspect` → read computed CSS values (e.g. confirm a glow `box-shadow` is applied).
- `preview_resize` → 390px (phone) and 1280px (desktop) to verify both layouts.

**Golden rule:** after every task, (1) `preview_console_logs` shows no errors, (2) all six gates remain completable, (3) the page still loads at 390px and 1280px. If any fails, fix before committing.

**Determinism note:** the recommendation engine (Phase 3) is pure logic and MUST be tested with `preview_eval` against the exact input→output pairs given. Do not "eyeball" it.

---

## File Structure

Single file only. Logical regions inside `vibe-coding-kit-dexgate-uiux-v9-drama.html`:

| Region | Location | Responsibility |
|---|---|---|
| Existing minified CSS | `<style>` 7–48 | **Untouched.** Base layout/colors. |
| **New** `<style id="drama-css">` | before `</head>` | All Phase 1/2/4/5 visual additions (glows, typography, particles, transitions, modal styles). Overrides via specificity. |
| Existing markup | `<body>` 50–350 | **Mostly untouched.** A few additive HTML hooks (background layer, question modal, recommendation section). |
| Existing IIFE JS | `<script>` 352–663 | Surgical edits to `unlockGate`, `switchPage`, `render`, game win paths. |
| **New** JS block | before init (line 662) | Dexter expression engine, question system, recommendation engine, particle/FX helpers, transition helpers. |

---

## Phase 0 — Setup & Safety Net

### Task 0: Create the v9 working copy and verify baseline

**Files:**
- Create: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (copy of v8)

- [ ] **Step 1: Copy the verified v8 file to v9**

```bash
cd "/Users/andrew/welcome to vibe coding"
cp vibe-coding-kit-dexgate-uiux-v8-verified.html vibe-coding-kit-dexgate-uiux-v9-drama.html
```

- [ ] **Step 2: Add the empty drama CSS block before `</head>`**

Edit `vibe-coding-kit-dexgate-uiux-v9-drama.html`. The existing file has `</style>` (line 48) then `</head>` (line 49). Insert a new block between them.

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
</style>
<style id="drama-css">
/* === DexGate Maximum Drama additions (Phases 1,2,4,5) === */
@media (prefers-reduced-motion: reduce){
  *{animation-duration:.001ms !important;animation-iteration-count:1 !important;transition-duration:.001ms !important;}
}
</style>
</head>
```

- [ ] **Step 3: Add the new-JS anchor comment inside the IIFE**

So later tasks have a stable, unique insertion point, add an empty marker right before the init line. Anchor (old_string, line 662):
`preloadDex(); populateProjects(); ensureGames(); render(); switchPage('start');`
Replace with:
```javascript
/* === DRAMA-JS-START === */
/* New drama functions are inserted between these markers. */
/* === DRAMA-JS-END === */
preloadDex(); populateProjects(); ensureGames(); render(); switchPage('start');
```
From now on, **all new JS** is inserted by replacing `/* === DRAMA-JS-END === */` with `<your new code>\n/* === DRAMA-JS-END === */` so the marker always stays last.

- [ ] **Step 4: Start the preview and verify baseline loads clean**

Run: `preview_start` on `vibe-coding-kit-dexgate-uiux-v9-drama.html`
Then `preview_console_logs`.
Expected: page renders the welcome overlay; **zero console errors**.

- [ ] **Step 5: Capture baseline screenshots (before-state for comparison)**

Run: `preview_screenshot` at default size, then `preview_resize` 390×800 and screenshot again.
Expected: two baseline images saved (desktop + phone). Keep for visual diffs.

- [ ] **Step 6: Commit**

```bash
cd "/Users/andrew/welcome to vibe coding"
git add vibe-coding-kit-dexgate-uiux-v9-drama.html docs/superpowers/plans/2026-06-04-dexgate-maximum-drama.md
git commit -m "chore: branch v9 drama from verified v8 + scaffold CSS/JS anchors"
```

---

### Task 1: Bump state version with non-destructive migration

The recommendation system adds `state.answers` and `state.recommendation`. Bump the store key and default the new fields so existing v5 saves migrate cleanly (old gate progress should NOT be wiped — copy it forward).

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (`STORE` line 368, `load()` line 428)

- [ ] **Step 1: Update the store key and load() defaults**

Anchor (old_string): `const STORE='dexgate_academy_state_v5';`
Replace with: `const STORE='dexgate_academy_state_v6';`

Anchor (old_string, the `load()` default object — match the first occurrence):
`Object.assign({welcomeSeen:false,gates:{},secrets:[],title:'Vibe Coding Kit',sound:false,longTrails:false,lastExpression:'neutral'}, JSON.parse(localStorage.getItem(STORE)||'{}'))`
Replace with:
```javascript
Object.assign({welcomeSeen:false,gates:{},secrets:[],title:'Vibe Coding Kit',sound:false,longTrails:false,lastExpression:'neutral',answers:{},recommendation:null}, (function(){try{var v6=JSON.parse(localStorage.getItem(STORE)||'null'); if(v6) return v6; var v5=JSON.parse(localStorage.getItem('dexgate_academy_state_v5')||'{}'); return v5;}catch(e){return {};}})())
```
This reads v6 if present, else migrates the old v5 blob forward (preserving `gates`/`secrets`), and guarantees `answers` and `recommendation` exist.

- [ ] **Step 2: Verify migration via preview_eval**

Run: `preview_eval` →
```javascript
localStorage.setItem('dexgate_academy_state_v5', JSON.stringify({welcomeSeen:true,gates:{start:true},secrets:['konami']}));
location.reload();
```
Then after reload, `preview_eval` →
```javascript
JSON.stringify({gates: JSON.parse(localStorage.getItem('dexgate_academy_state_v6')||'{}').gates, hasAnswers: typeof JSON.parse(localStorage.getItem('dexgate_academy_state_v6')||'{}').answers})
```
Expected output: `{"gates":{"start":true},"hasAnswers":"object"}` — old progress carried over, `answers` present.

- [ ] **Step 3: Reset to a clean slate for further work**

Run: `preview_eval` → `localStorage.clear(); location.reload();`
Then `preview_console_logs`. Expected: zero errors, welcome overlay shown.

- [ ] **Step 4: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: migrate state to v6 with answers/recommendation fields"
```

---

## Phase 1 — Visual System (the "wow" on open)

> All CSS in this phase goes inside `<style id="drama-css">`. Append rules by replacing the drama block's closing `</style>\n</head>` anchor — insert your rules *before* the closing tags each time.

### Task 2: Glow utility classes + amplified neon variables

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (`<style id="drama-css">`)

- [ ] **Step 1: Append glow tokens and utilities**

Anchor (old_string): `</style>\n</head>`
Replace with (keep the closing tags; insert rules before them):
```html
/* --- Phase 1: glow system --- */
:root{
  --glow-cyan:0 0 6px rgba(0,229,255,.55),0 0 18px rgba(0,229,255,.35),0 0 40px rgba(0,229,255,.18);
  --glow-pink:0 0 6px rgba(255,0,127,.55),0 0 18px rgba(255,0,127,.35),0 0 40px rgba(255,0,127,.18);
  --glow-violet:0 0 6px rgba(135,89,255,.55),0 0 18px rgba(135,89,255,.35),0 0 40px rgba(135,89,255,.18);
}
.glow-text{color:#eafcff;text-shadow:0 0 4px rgba(0,229,255,.9),0 0 12px rgba(0,229,255,.5),0 0 28px rgba(0,229,255,.3);}
.glow-text.pink{text-shadow:0 0 4px rgba(255,0,127,.9),0 0 12px rgba(255,0,127,.5),0 0 28px rgba(255,0,127,.3);}
.glow-edge{box-shadow:var(--glow-cyan);}
.glow-edge.pink{box-shadow:var(--glow-pink);}
/* lift the hero title + gate names without touching minified base CSS */
.main h1{font-size:clamp(1.6rem,4.5vw,2.4rem);font-weight:900;letter-spacing:.005em;text-shadow:0 0 5px rgba(0,229,255,.55),0 0 22px rgba(0,229,255,.25);}
.gameTop h2{text-shadow:0 0 6px rgba(255,0,127,.5);}
.welcomeCard h1{text-shadow:0 0 8px rgba(0,229,255,.6),0 0 30px rgba(135,89,255,.35);}
</style>
</head>
```

- [ ] **Step 2: Verify glow renders and no errors**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome: `document.getElementById('welcomeOverlay').classList.remove('show');`
Run: `preview_inspect` on selector `.main h1` → read `text-shadow`.
Expected: a non-`none` `text-shadow` containing `rgba(0, 229, 255`. `preview_console_logs`: zero errors.

- [ ] **Step 3: Screenshot for visual proof**

Run: `preview_screenshot`. Expected: title text visibly glows.

- [ ] **Step 4: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: add neon glow utility classes and amplified headings"
```

---

### Task 3: Card edge-glow, hover lift, and locked/unlocked distinction

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (`<style id="drama-css">`)

- [ ] **Step 1: Append card + button + locked-state rules**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 1: cards, buttons, locked states --- */
.panel{transition:box-shadow .25s ease, border-color .25s ease, transform .25s ease;}
.gameCard{box-shadow:var(--glow-cyan), var(--shadow);border-color:var(--line2);}
.gameCard:hover{box-shadow:0 0 8px rgba(0,229,255,.7),0 0 26px rgba(0,229,255,.4),var(--shadow);}
.card{transition:box-shadow .2s ease, transform .2s ease;}
.card:hover{box-shadow:var(--glow-cyan);transform:translateY(-2px);}
.content.locked{filter:saturate(.55) brightness(.78);} /* dimmed when locked */
.content:not(.locked){box-shadow:var(--glow-cyan), var(--shadow);} /* bright when unlocked */
.btn{transition:transform .12s ease, box-shadow .12s ease;}
.btn:hover{transform:scale(1.05);box-shadow:var(--glow-cyan);}
.btn:active{transform:scale(.97);}
.btn:disabled,.btn[disabled]{filter:grayscale(.7) brightness(.6);box-shadow:none;cursor:not-allowed;transform:none;}
.nav button{transition:box-shadow .2s ease, transform .15s ease;}
.nav button:hover{transform:translateX(2px);}
.nav button.done{box-shadow:inset 0 0 0 1px rgba(0,229,255,.4),0 0 12px rgba(0,229,255,.2);}
</style>
</head>
```

- [ ] **Step 2: Verify locked vs unlocked visual difference**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then `document.getElementById('welcomeOverlay').classList.remove('show');`
Run: `preview_inspect` on `.content.locked` → read `filter`. Expected: contains `saturate(0.55)` and `brightness(0.78)`.
Run: `preview_console_logs`. Expected: zero errors.

- [ ] **Step 3: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: card edge-glow, button states, locked/unlocked distinction"
```

---

### Task 4: Animated background (drifting particles + breathing gradient)

A lightweight CSS-only background layer (no JS RAF cost). Behind all content, very subtle, respects reduced-motion (handled by the Task 0 global guard).

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (markup near `<body>` line 50 + `<style id="drama-css">`)

- [ ] **Step 1: Add the background layer element right after `<body>`**

Anchor (old_string): `<body>\n<div class="silkDot" id="silkDot" aria-hidden="true"></div>`
Replace with:
```html
<body>
<div class="dramaBg" aria-hidden="true"><div class="dramaParticles"></div></div>
<div class="silkDot" id="silkDot" aria-hidden="true"></div>
```

- [ ] **Step 2: Append the background CSS**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 1: animated background --- */
.dramaBg{position:fixed;inset:0;z-index:0;pointer-events:none;overflow:hidden;
  background:
    radial-gradient(circle at 18% 12%, rgba(0,229,255,.10), transparent 30%),
    radial-gradient(circle at 84% 22%, rgba(255,0,127,.08), transparent 32%),
    radial-gradient(circle at 60% 90%, rgba(135,89,255,.08), transparent 34%);
  animation:dramaBreath 9s ease-in-out infinite;}
@keyframes dramaBreath{0%,100%{opacity:.7;}50%{opacity:1;}}
.dramaParticles{position:absolute;inset:-50% 0;background-image:
  radial-gradient(1.5px 1.5px at 20% 30%, rgba(0,229,255,.6), transparent),
  radial-gradient(1.5px 1.5px at 70% 60%, rgba(135,89,255,.5), transparent),
  radial-gradient(1px 1px at 40% 80%, rgba(255,0,127,.5), transparent),
  radial-gradient(1.5px 1.5px at 90% 15%, rgba(0,229,255,.5), transparent),
  radial-gradient(1px 1px at 55% 45%, rgba(255,255,255,.4), transparent);
  background-size:480px 480px;animation:dramaDrift 60s linear infinite;}
@keyframes dramaDrift{from{transform:translateY(0);}to{transform:translateY(25%);}}
/* keep content above the bg */
.topbar,.wrap{position:relative;z-index:1;}
</style>
</head>
```

- [ ] **Step 3: Verify background present and content still on top**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then `document.getElementById('welcomeOverlay').classList.remove('show');`
Run: `preview_snapshot`. Expected: `.dramaBg` exists; nav/header still interactive (z-index above bg).
Run: `preview_console_logs`. Expected: zero errors.
Run: `preview_screenshot`. Expected: faint particles/gradient visible behind content, not distracting.

- [ ] **Step 4: Verify it does not block clicks (pointer-events)**

Run: `preview_eval` → `getComputedStyle(document.querySelector('.dramaBg')).pointerEvents`
Expected: `"none"`.

- [ ] **Step 5: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: subtle animated neon background (particles + breathing gradient)"
```

---

### Task 5: Optional scanline overlay toggle

A subtle CRT scanline layer, **off by default**, toggled by a small header button (matches spec "can be toggled"). Persist in `state`.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (header markup line 115 area, `<style id="drama-css">`, new JS block, `render()` line 445)

- [ ] **Step 1: Append scanline CSS**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 1: scanlines (toggleable) --- */
body.scanlines::after{content:"";position:fixed;inset:0;z-index:40;pointer-events:none;
  background:repeating-linear-gradient(to bottom, rgba(0,0,0,0) 0, rgba(0,0,0,0) 2px, rgba(0,0,0,.10) 3px);
  mix-blend-mode:overlay;}
</style>
</head>
```

- [ ] **Step 2: Add the toggle button to the header actions**

Anchor (old_string): `<button class="btnIcon" id="glitchBtn">⚠</button>`
Replace with:
```html
<button class="btnIcon" id="scanBtn" title="Toggle scanlines">▤</button><button class="btnIcon" id="glitchBtn">⚠</button>
```

- [ ] **Step 3: Add the toggle JS (inside IIFE)**

Anchor (old_string): `/* === DRAMA-JS-END === */`
Replace with:
```javascript
function applyScanlines(){ document.body.classList.toggle('scanlines', !!state.scanlines); }
$('#scanBtn').addEventListener('click', ()=>{ state.scanlines=!state.scanlines; save(); applyScanlines(); });
/* === DRAMA-JS-END === */
```

- [ ] **Step 4: Call applyScanlines() on load — extend render()**

First read the tail of `render()` (line 445) to get the exact substring:
```bash
cd "/Users/andrew/welcome to vibe coding"; sed -n '445p' vibe-coding-kit-dexgate-uiux-v9-drama.html | cut -c1-400
```
Then anchor on a stable fragment near the end of `render()` (e.g. `setDexImage('smug','welcomeDex2');`) and prepend `applyScanlines();` before it:
Anchor (old_string): `setDexImage('smug','welcomeDex2');`
Replace with: `applyScanlines(); setDexImage('smug','welcomeDex2');`

- [ ] **Step 5: Verify toggle works and persists**

Run: `preview_eval` →
```javascript
document.getElementById('scanBtn').click();
JSON.stringify({on: document.body.classList.contains('scanlines'), saved: JSON.parse(localStorage.getItem('dexgate_academy_state_v6')).scanlines})
```
Expected: `{"on":true,"saved":true}`.
Run: `preview_console_logs`. Expected: zero errors.

- [ ] **Step 6: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: toggleable CRT scanline overlay, persisted in state"
```

---

## Phase 2 — Dexter's Expanded Role

### Task 6: Expression engine + contextual per-page expression

Centralize Dexter's expression so every reaction is consistent, and update it on page navigation.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (new JS block; `switchPage()` line 444)

- [ ] **Step 1: Add the expression engine (inside IIFE)**

Anchor (old_string): `/* === DRAMA-JS-END === */`
Replace with:
```javascript
/* Dexter expression engine: one place to drive every Dexter <img>. */
const DEX_TARGETS=['dexImage','logoDexImg']; // sidebar + header brand
function dexExpress(expression){
  const key=DEXTER_IMAGES[expression]?expression:'neutral';
  state.lastExpression=key; save();
  DEX_TARGETS.forEach(id=>setDexImage(key,id));
  const frame=document.querySelector('.dexFrame');
  if(frame){ frame.classList.remove('dex-pop'); void frame.offsetWidth; frame.classList.add('dex-pop'); }
}
// Per-page resting expression (neutral/focused vibe per page)
const PAGE_EXPRESSION={start:'neutral',tooling:'neutral',pricing:'unimpressed',beginner:'friendly',linux:'neutral',pro:'annoyed'};
/* === DRAMA-JS-END === */
```

- [ ] **Step 2: Add the dex-pop animation CSS**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 2: Dexter pop on expression change --- */
.dexFrame{transition:box-shadow .3s ease;box-shadow:var(--glow-cyan);}
.dex-pop{animation:dexPop .4s ease;}
@keyframes dexPop{0%{transform:scale(.92);}55%{transform:scale(1.05);}100%{transform:scale(1);}}
.logoDex{box-shadow:0 0 18px rgba(0,229,255,.35) !important;}
</style>
</head>
```

- [ ] **Step 3: Drive expression from switchPage()**

First read `switchPage` (line 444) to confirm the exact fragment:
```bash
cd "/Users/andrew/welcome to vibe coding"; sed -n '444p' vibe-coding-kit-dexgate-uiux-v9-drama.html | cut -c1-400
```
Anchor (old_string, a stable fragment): `$('#currentPageStatus').textContent=def?def.label:id;`
Replace with:
```javascript
$('#currentPageStatus').textContent=def?def.label:id; if(typeof dexExpress==='function') dexExpress(state.gates[id]?'smug':(PAGE_EXPRESSION[id]||'neutral'));
```

- [ ] **Step 4: Verify expression updates on navigation**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then `document.getElementById('welcomeOverlay').classList.remove('show');`
Run: `preview_eval` →
```javascript
document.querySelector('.nav button[data-page="beginner"]').click();
JSON.stringify({last: JSON.parse(localStorage.getItem('dexgate_academy_state_v6')).lastExpression})
```
Expected: `{"last":"friendly"}` (page `beginner` resting expression).
Run: `preview_console_logs`. Expected: zero errors.

- [ ] **Step 5: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: Dexter expression engine + per-page contextual expressions"
```

---

### Task 7: In-game reactions (good move / mistake) + idle boredom

Hook Dexter expression changes to gameplay events without rewriting game loops. We expose two tiny helpers and call them from a few existing game win/fail points already present in the code.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (new JS block; `FloppyDex.fail` line 481; `idleLoop` line 660)

- [ ] **Step 1: Add reaction helpers (inside IIFE)**

Anchor (old_string): `/* === DRAMA-JS-END === */`
Replace with:
```javascript
let _dexReactCooldown=0;
function dexGoodMove(){ const now=performance.now(); if(now<_dexReactCooldown) return; _dexReactCooldown=now+1200; dexExpress('smug'); }
function dexBadMove(){ const now=performance.now(); if(now<_dexReactCooldown) return; _dexReactCooldown=now+1200; dexExpress('unimpressed'); }
/* === DRAMA-JS-END === */
```

- [ ] **Step 2: React on Floppy Dex crash (existing fail path)**

First confirm the fail text:
```bash
cd "/Users/andrew/welcome to vibe coding"; sed -n '481p' vibe-coding-kit-dexgate-uiux-v9-drama.html | cut -c1-200
```
Anchor (old_string): `function fail(){ run=false; speak('Floppy Dex crashed.`
Replace with: `function fail(){ run=false; if(typeof dexBadMove==='function') dexBadMove(); speak('Floppy Dex crashed.`

- [ ] **Step 3: Make Dexter look bored when idle (extend existing idle hook)**

Anchor (old_string): `idle=true; unlockSecret('screenSaver', true); speak(lines.idle,'neutral');`
Replace with: `idle=true; unlockSecret('screenSaver', true); if(typeof dexExpress==='function') dexExpress('bored'); speak(lines.idle,'bored');`

- [ ] **Step 4: Verify no errors and helpers exist**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then `document.getElementById('welcomeOverlay').classList.remove('show');`
Run: `preview_console_logs`. Expected: zero errors.
(Helpers are IIFE-scoped; a syntax error would surface as a console error and the games would fail to start — so a clean load is the assertion.)

- [ ] **Step 5: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: Dexter reacts to mistakes and idle boredom"
```

---

### Task 8: Dry welcome greeting + smug victory lines

Replace warm copy with Dexter's deadpan voice and add a rotating dry victory line on gate clear.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (welcome `<h1>` line 57; new JS block)

- [ ] **Step 1: Make the welcome greeting dry**

Anchor (old_string): `<h1 style="margin:.55rem 0 .5rem">DexGate Academy: Learn coding tools and projects through games.</h1>`
Replace with:
```html
<h1 style="margin:.55rem 0 .5rem" class="glow-text">You're here. I noticed.</h1>
<p class="small" style="margin:0 0 .5rem;opacity:.85">DexGate Academy. Six gates, six lessons. Don't embarrass us.</p>
```

- [ ] **Step 2: Add a dry victory-line pool (inside IIFE)**

Anchor (old_string): `/* === DRAMA-JS-END === */`
Replace with:
```javascript
const DEX_WIN_LINES=['Adequate.','I tolerate this.','You earned that one.','Acceptable.','Fine. Moving on.','That was almost competent.'];
function dexWinLine(){ return DEX_WIN_LINES[Math.floor(Math.random()*DEX_WIN_LINES.length)]; }
/* === DRAMA-JS-END === */
```

> `dexWinLine()` is consumed by `unlockGate` in Task 11. Keep this task scoped to defining it.

- [ ] **Step 3: Verify**

Run: `preview_eval` → `localStorage.clear(); location.reload();`
Run: `preview_snapshot`. Expected: welcome heading reads "You're here. I noticed."
Run: `preview_console_logs`. Expected: zero errors (a syntax error in the win-line array would surface here).

- [ ] **Step 4: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: dry Dexter welcome greeting and victory-line pool"
```

---

## Phase 3 — Tool Recommendation System (functional core)

> This is the highest-value NEW feature. It is **pure logic + a modal UI**. The engine MUST be verified with the exact `preview_eval` assertions given. Build the engine first (testable), then the UI, then wire it to wins.

### Task 9: Question data + recommendation scoring engine

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (new JS block)

- [ ] **Step 1: Add question definitions + scoring engine (inside IIFE)**

Anchor (old_string): `/* === DRAMA-JS-END === */`
Replace with:
```javascript
/* --- Tool recommendation: 5 questions, mapped to pages 1-5 --- */
const REC_QUESTIONS=[
  {page:'start', q:"What's your actual constraint here?", opts:[
    {k:'A', t:'Budget is tight. Free or very cheap is essential.'},
    {k:'B', t:'I have money. Paying for quality is fine.'},
    {k:'C', t:'I want to see what is possible first before spending.'},
    {k:'D', t:'I need something predictable for a team (budgeting matters).'}]},
  {page:'tooling', q:"What are you building with this AI?", opts:[
    {k:'A', t:'Mostly talking to it, getting ideas and feedback.'},
    {k:'B', t:'Code and technical problem-solving.'},
    {k:'C', t:'Long documents, research, analysis.'},
    {k:'D', t:'Something collaborative — multiple people using it together.'}]},
  {page:'pricing', q:"How much do you care about privacy?", opts:[
    {k:'A', t:'Not at all. Use my data however.'},
    {k:'B', t:'Somewhat. I would prefer a company that is not Google or OpenAI.'},
    {k:'C', t:'Very much. I want to know what happens to my conversations.'},
    {k:'D', t:'Extremely. I need guarantees.'}]},
  {page:'beginner', q:"What's your technical comfort level?", opts:[
    {k:'A', t:'I am not technical. I just want something that works in a browser.'},
    {k:'B', t:'I can follow instructions but do not want to code.'},
    {k:'C', t:'I can write some code if I need to.'},
    {k:'D', t:'I am comfortable with APIs and integrations.'}]},
  {page:'linux', q:"What would actually make you use this?", opts:[
    {k:'A', t:'A good free option I can share with friends.'},
    {k:'B', t:'Something that works seamlessly on my phone and Chromebook.'},
    {k:'C', t:'The smartest AI available, even if it costs.'},
    {k:'D', t:'A tool designed for collaboration and teams.'}]}
];
const REC_PAGE_TO_Q={start:'q1',tooling:'q2',pricing:'q3',beginner:'q4',linux:'q5'};
/* Scoring: idx=Project IDX, claude=Claude Code, codex=OpenAI Codex. Tie-break idx>claude>codex. */
const REC_SCORE={
  q1:{A:{idx:2}, B:{claude:2}, C:{idx:1,claude:1}, D:{idx:2}},
  q2:{A:{claude:2}, B:{claude:2,codex:1}, C:{claude:2}, D:{idx:2}},
  q3:{A:{codex:1}, B:{claude:2}, C:{claude:2}, D:{claude:2}},
  q4:{A:{idx:2}, B:{idx:1,claude:1}, C:{claude:1,codex:1}, D:{claude:1,codex:1}},
  q5:{A:{idx:2}, B:{idx:1,claude:1}, C:{claude:2}, D:{idx:2}}
};
function computeRecommendation(answers){
  const score={idx:0,claude:0,codex:0};
  Object.keys(REC_SCORE).forEach(q=>{
    const pick=answers&&answers[q]; if(!pick) return;
    const add=REC_SCORE[q][pick]||{};
    Object.keys(add).forEach(tool=>{ score[tool]+=add[tool]; });
  });
  const order=['idx','claude','codex']; // tie-break priority
  let best=order[0];
  order.forEach(t=>{ if(score[t]>score[best]) best=t; });
  return {winner:best, score:score};
}
window.__computeRecommendation=computeRecommendation; // exposed for verification only
/* === DRAMA-JS-END === */
```

- [ ] **Step 2: Verify engine with deterministic input→output pairs**

Run: `preview_eval` →
```javascript
JSON.stringify([
  __computeRecommendation({q1:'A',q2:'A',q3:'A',q4:'A',q5:'A'}).winner,
  __computeRecommendation({q1:'B',q2:'B',q3:'C',q4:'C',q5:'C'}).winner,
  __computeRecommendation({q1:'A',q2:'D',q3:'A',q4:'A',q5:'A'}).winner,
  __computeRecommendation({}).winner
])
```
Expected output: `["idx","claude","idx","idx"]`
- All-A: idx 2+0+0+2+2=6 vs claude 0+2+0+0+0=2 → **idx**.
- B/B/C/C/C: claude 2+2+2+1+2=9 vs idx 0 → **claude**.
- A/D/A/A/A: idx 2+2+0+2+2=8 → **idx**.
- empty: all zero, tie-break → **idx**.

If output differs, the scoring tables are wrong — fix before proceeding.

- [ ] **Step 3: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: tool-recommendation question data + deterministic scoring engine"
```

---

### Task 10: Question modal UI + answer storage

A dedicated modal (separate from the existing `#modal`) that presents one question with four option buttons, stores the answer, then unlocks content. DOM is built with `createElement`/`textContent` (no `innerHTML`).

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (markup after `#toast` line 93; `<style id="drama-css">`; new JS block)

- [ ] **Step 1: Add the question modal markup**

Anchor (old_string): `<div class="toast" id="toast"><b id="toastTitle"></b><p id="toastText"></p></div>`
Replace with:
```html
<div class="toast" id="toast"><b id="toastTitle"></b><p id="toastText"></p></div>
<div class="modal qmodal" id="qmodal" aria-modal="true" role="dialog" aria-labelledby="qmodalQ">
  <div class="modalCard qmodalCard">
    <div class="modalBody">
      <div class="modalDex"><img id="qmodalDex" alt="Dexter" src=""></div>
      <div style="min-width:0">
        <div class="eyebrow">Dexter wants to know</div>
        <h2 id="qmodalQ" class="glow-text" style="margin:.2rem 0 .2rem"></h2>
      </div>
    </div>
    <div class="qmodalOpts" id="qmodalOpts"></div>
  </div>
</div>
```

- [ ] **Step 2: Add question-modal CSS**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 3: question modal --- */
.qmodalCard{max-width:560px;width:100%;box-shadow:var(--glow-violet),var(--shadow);}
.qmodalOpts{display:grid;gap:10px;margin-top:14px;}
.qmodalOpts button{text-align:left;min-height:var(--touch);border-radius:14px;border:1px solid var(--line2);
  background:rgba(8,11,18,.7);color:var(--text);padding:12px 14px;cursor:pointer;font:inherit;
  transition:box-shadow .15s ease, transform .12s ease, border-color .15s ease;}
.qmodalOpts button:hover{box-shadow:var(--glow-cyan);transform:translateX(3px);border-color:var(--cyan);}
.qmodalOpts button:active{transform:scale(.98);}
.qmodalOpts .qk{display:inline-block;min-width:1.4em;font:800 .9rem ui-monospace,monospace;color:var(--cyan);margin-right:.4em;}
</style>
</head>
```

- [ ] **Step 3: Add the question-flow JS (inside IIFE) — safe DOM, no innerHTML**

Anchor (old_string): `/* === DRAMA-JS-END === */`
Replace with:
```javascript
function storeAnswer(page, key){
  const qid=REC_PAGE_TO_Q[page]; if(!qid) return;
  state.answers=state.answers||{}; state.answers[qid]=key; save();
}
function askQuestion(page, onDone){
  const def=REC_QUESTIONS.find(q=>q.page===page);
  if(!def){ if(onDone) onDone(); return; }
  setDexImage('neutral','qmodalDex');
  $('#qmodalQ').textContent=def.q;
  const box=$('#qmodalOpts'); box.textContent='';
  def.opts.forEach(o=>{
    const b=document.createElement('button');
    const span=document.createElement('span'); span.className='qk'; span.textContent=o.k;
    b.appendChild(span); b.appendChild(document.createTextNode(o.t));
    b.addEventListener('click', ()=>{
      storeAnswer(page, o.k);
      $('#qmodal').classList.remove('show');
      if(onDone) onDone();
    });
    box.appendChild(b);
  });
  $('#qmodal').classList.add('show');
}
window.__askQuestion=askQuestion; // verification only
/* === DRAMA-JS-END === */
```

- [ ] **Step 4: Verify the modal opens, stores an answer, and closes**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then `document.getElementById('welcomeOverlay').classList.remove('show');`
Run: `preview_eval` →
```javascript
__askQuestion('pricing', ()=>{window.__qdone=true;});
document.getElementById('qmodal').classList.contains('show')
```
Expected: `true` (modal open).
Run: `preview_eval` →
```javascript
document.querySelectorAll('#qmodalOpts button')[1].click(); // choose option B
JSON.stringify({answer: JSON.parse(localStorage.getItem('dexgate_academy_state_v6')).answers, open: document.getElementById('qmodal').classList.contains('show'), done: window.__qdone})
```
Expected: `{"answer":{"q3":"B"},"open":false,"done":true}` (pricing → q3, option B stored, modal closed, callback fired).
Run: `preview_console_logs`. Expected: zero errors.

- [ ] **Step 5: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: question modal UI with four options and answer storage"
```

---

### Task 11: Wire questions into the gate-clear flow

On clearing a gate for pages 1–5, show the dry win line, then the question, then unlock. Page 6 (`pro`) clears normally (no question) and triggers the recommendation reveal (Task 13).

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (`unlockGate()` line 440)

- [ ] **Step 1: Inspect the exact unlockGate body to anchor on**

The existing `unlockGate(page,gameName,fast)` (line 440) sets `state.gates[page]=true`, re-renders, hides the overlay, then calls `speak(...)` and `modal('Gate cleared', ...)`. Read the full line first (it is one minified line) so your anchor substring is exact:
```bash
cd "/Users/andrew/welcome to vibe coding"; sed -n '440p' vibe-coding-kit-dexgate-uiux-v9-drama.html | cut -c1-500
```

- [ ] **Step 2: Insert the question hook after the gate is marked cleared**

Anchor (old_string): `speak(gameName+' cleared. '+lines.gate,'correct');`
Replace with:
```javascript
if(typeof dexExpress==='function') dexExpress('smug'); speak(gameName+' cleared. '+dexWinLine(),'smug'); if(typeof askQuestion==='function' && REC_PAGE_TO_Q[page]){ askQuestion(page, ()=>{ if(typeof maybeReveal==='function') maybeReveal(); }); }
```

> This keeps the original `modal('Gate cleared', ...)` call that follows it. The question modal (`#qmodal`) and status modal (`#modal`) are distinct elements; the question modal sits on top. If overlap feels heavy during verification, suppress the legacy `modal(...)` for gated pages by making that following call conditional — but only if verification shows a problem.

- [ ] **Step 3: Add a placeholder `maybeReveal` so the hook is safe until Task 13**

Anchor (old_string): `/* === DRAMA-JS-END === */`
Replace with:
```javascript
function maybeReveal(){ /* implemented in Task 13 (recommendation reveal) */ }
/* === DRAMA-JS-END === */
```

- [ ] **Step 4: Verify the question path is reachable and compiles**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Run: `preview_eval` →
```javascript
__askQuestion('start', ()=>{}); document.getElementById('qmodal').classList.contains('show')
```
Expected: `true`. (Full game-play verification happens in Task 23; here we confirm the wiring compiles and the question path is reachable.)
Run: `preview_console_logs`. Expected: zero errors.

- [ ] **Step 5: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: present recommendation question after clearing gates 1-5"
```

---

### Task 12: Final recommendation page content (three tools)

Add a dedicated, initially-hidden recommendation section on the **Pro Vault** page (`page-pro`) holding the full three-tool breakdown and a slot for Dexter's pick.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (inside `<section id="page-pro">`, before its closing `</section>` — around line 330; `<style id="drama-css">`)

- [ ] **Step 1: Read the end of the pro page to find the anchor**

```bash
cd "/Users/andrew/welcome to vibe coding"; sed -n '303,333p' vibe-coding-kit-dexgate-uiux-v9-drama.html | grep -n 'data-content="pro"\|</section>\|dexPanel'
```
Identify the `</section>` that closes `#page-pro` — it immediately precedes `<section class="panel dexPanel">` (line 332). Insert the recommendation block just before that closing `</section>`.

- [ ] **Step 2: Insert the recommendation markup**

Anchor (old_string), using the unique sibling that follows the pro page's close:
`</section>\n    <section class="panel dexPanel">`
Replace with:
```html
      <div class="panel content recPanel hidden" id="recPanel">
        <h2 class="glow-text pink" style="margin:.1rem 0 .3rem">Dexter's Tool Verdict</h2>
        <p class="small">You answered five questions. Here are the three real options, then my pick. I don't do "it depends."</p>
        <div class="recGrid">
          <div class="card recCard" data-tool="idx">
            <h4 class="glow-text">Project IDX <span class="small">(Google)</span></h4>
            <p class="small"><b>What:</b> Browser-based cloud IDE built on VS Code. Runs on any device.</p>
            <p class="small"><b>Excels:</b> Zero setup, Chromebook-native, phone-accessible, team collaboration, generous free tier.</p>
            <p class="small"><b>Weak:</b> Needs internet, slower than local for big projects, smaller ecosystem.</p>
            <p class="small"><b>Price:</b> Free tier; paid from ~$10/mo; Google Cloud + Gemini billed separately.</p>
          </div>
          <div class="card recCard" data-tool="claude">
            <h4 class="glow-text">Claude Code <span class="small">(Anthropic)</span></h4>
            <p class="small"><b>What:</b> AI-first coding environment with Claude integrated directly.</p>
            <p class="small"><b>Excels:</b> Seamless AI pair-programming, mobile-friendly, privacy-conscious, predictable per-token pricing.</p>
            <p class="small"><b>Weak:</b> Lighter on deploy/DevOps, no built-in hosting, limited multi-user collab.</p>
            <p class="small"><b>Price:</b> Limited free tier; ~$20/mo standard, ~$200/mo pro; per-token option.</p>
          </div>
          <div class="card recCard" data-tool="codex">
            <h4 class="glow-text">OpenAI Codex <span class="small">(ChatGPT)</span></h4>
            <p class="small"><b>What:</b> Code generation + assistant via ChatGPT/API. Not a full IDE.</p>
            <p class="small"><b>Excels:</b> Strong code reasoning, huge community, mature API, clear explanations.</p>
            <p class="small"><b>Weak:</b> No dedicated mobile IDE, needs a separate editor, unpredictable token cost, data-privacy concerns.</p>
            <p class="small"><b>Price:</b> Free tier limited; Plus $20/mo; API usage-based and can spike.</p>
          </div>
        </div>
        <div class="recPick glow-edge pink" id="recPick"></div>
      </div>
      </section>
    <section class="panel dexPanel">
```

> Structure: the new `<div id="recPanel">` is added *inside* the pro `<section>`, then the page's `</section>` and the dexPanel section that followed are re-emitted unchanged.

- [ ] **Step 3: Add recommendation CSS**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 3: recommendation page --- */
.recGrid{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:12px;margin-top:12px;}
.recCard{transition:box-shadow .2s ease, transform .2s ease;}
.recCard.is-pick{box-shadow:var(--glow-pink);transform:translateY(-3px);border-color:var(--pink);}
.recPick{margin-top:16px;padding:16px;border-radius:16px;border:1px solid var(--line2);background:rgba(10,14,22,.8);white-space:pre-line;line-height:1.55;}
.recPick:empty{display:none;}
@media (max-width:860px){.recGrid{grid-template-columns:1fr;}}
</style>
</head>
```

- [ ] **Step 4: Verify markup parses and panel is hidden by default**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Run: `preview_eval` →
```javascript
JSON.stringify({exists: !!document.getElementById('recPanel'), hidden: document.getElementById('recPanel').classList.contains('hidden'), cards: document.querySelectorAll('#recPanel .recCard').length})
```
Expected: `{"exists":true,"hidden":true,"cards":3}`.
Run: `preview_snapshot`. Expected: all 6 nav pages intact (a broken nested `</section>` would corrupt the DOM).
Run: `preview_console_logs`. Expected: zero errors.

- [ ] **Step 5: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: three-tool recommendation section on Pro Vault page (hidden)"
```

---

### Task 13: Reveal logic + Dexter's personalized pick

Implement `maybeReveal()` (placeholder added in Task 11) to compute the winner once all five answers exist, render Dexter's pick text, highlight the chosen card, and reveal the panel.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (`maybeReveal` placeholder in new JS block)

- [ ] **Step 1: Replace the maybeReveal placeholder with the full implementation**

Anchor (old_string): `function maybeReveal(){ /* implemented in Task 13 (recommendation reveal) */ }`
Replace with:
```javascript
const REC_TOOL_NAME={idx:'Project IDX', claude:'Claude Code', codex:'OpenAI Codex'};
const REC_PICK_COPY={
  idx:{ why:['You need something on a Chromebook that just works — IDX is built for exactly that.','You care about cost. The free tier covers what you need. No surprises.','Collaboration is baked in. No setup required.'],
        next:'Go to idx.google.com, sign in with Google, create a new project.',
        notClaude:'Claude Code: excellent for AI-assisted work, but get a dedicated IDE shipped first.',
        notCodex:'OpenAI Codex: too expensive for exploration and needs an external editor. Not a first move.'},
  claude:{ why:['You want the AI doing real work with you, not just autocomplete.','Predictable per-token pricing means no surprise bills.','Privacy matters to you, and you get control over your data.'],
        next:'Open Claude Code in your browser, start a project, and let it pair with you.',
        notIdx:'Project IDX: great Chromebook IDE, but lighter as an AI collaborator.',
        notCodex:'OpenAI Codex: strong reasoning, but no dedicated mobile IDE and unpredictable cost.'},
  codex:{ why:['You want the strongest raw code reasoning and a huge community.','You are comfortable wiring it into your own editor.','You are fine with usage-based pricing.'],
        next:'Use ChatGPT for code help, and add the Codex/API integration to your editor.',
        notIdx:'Project IDX: simpler to start, but weaker pure code reasoning.',
        notClaude:'Claude Code: better integrated IDE experience, but you wanted maximum reasoning + ecosystem.'}
};
function renderPick(winner){
  const c=REC_PICK_COPY[winner], name=REC_TOOL_NAME[winner];
  const others=Object.keys(REC_TOOL_NAME).filter(t=>t!==winner);
  const notLines=others.map(t=>{
    if(winner==='idx') return t==='claude'?c.notClaude:c.notCodex;
    if(winner==='claude') return t==='idx'?c.notIdx:c.notCodex;
    return t==='idx'?c.notIdx:c.notClaude;
  });
  const txt='🎯 DEXTER’S PICK: '+name+'\n\nWhy this is your best option:\n• '+c.why.join('\n• ')+'\n\nWhat to do next:\n'+c.next+'\n\nWhy not the others:\n• '+notLines.join('\n• ');
  $('#recPick').textContent=txt;
  $$('#recPanel .recCard').forEach(card=>card.classList.toggle('is-pick', card.dataset.tool===winner));
}
function allAnswered(){ return ['q1','q2','q3','q4','q5'].every(q=>state.answers&&state.answers[q]); }
function maybeReveal(){
  if(!allAnswered()) return;
  const res=computeRecommendation(state.answers);
  state.recommendation=res.winner; save();
  renderPick(res.winner);
  const panel=$('#recPanel'); if(panel){ panel.classList.remove('hidden'); panel.classList.add('rec-reveal'); }
  dexExpress('smug');
  speak('Five questions. One answer. I picked '+REC_TOOL_NAME[res.winner]+'. You are welcome.','smug');
}
window.__maybeReveal=maybeReveal; // verification only
```

> This replaces only the placeholder function, so the surrounding `/* === DRAMA-JS-END === */` marker stays in place.

- [ ] **Step 2: Add reveal animation CSS**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 3: reveal animation --- */
.rec-reveal{animation:recReveal .6s ease;}
@keyframes recReveal{0%{opacity:0;transform:translateY(16px);}100%{opacity:1;transform:translateY(0);}}
</style>
</head>
```

- [ ] **Step 3: Verify full reveal with a known answer set**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Run: `preview_eval` →
```javascript
var s=JSON.parse(localStorage.getItem('dexgate_academy_state_v6')||'{}');
s.answers={q1:'A',q2:'A',q3:'A',q4:'A',q5:'A'}; localStorage.setItem('dexgate_academy_state_v6', JSON.stringify(s));
location.reload();
```
Then `preview_eval` →
```javascript
__maybeReveal();
JSON.stringify({hidden: document.getElementById('recPanel').classList.contains('hidden'), pick: JSON.parse(localStorage.getItem('dexgate_academy_state_v6')).recommendation, pickHead: document.getElementById('recPick').textContent.split('\n')[0], highlighted: document.querySelector('.recCard.is-pick').dataset.tool})
```
Expected: `{"hidden":false,"pick":"idx","pickHead":"🎯 DEXTER'S PICK: Project IDX","highlighted":"idx"}`.
Run: `preview_console_logs`. Expected: zero errors.

- [ ] **Step 4: Navigate to Pro page and screenshot the verdict**

Run: `preview_eval` → `document.querySelector('.nav button[data-page="pro"]').click();`
Run: `preview_screenshot`. Expected: three tool cards, IDX card glowing pink, Dexter's pick text visible.

- [ ] **Step 5: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: reveal Dexter's personalized tool pick after all 5 answers"
```

---

## Phase 4 — Game Elevation (polish)

> Each game already runs its own RAF loop and draws via shared helpers (`drawScene`, `drawDexSprite`, `drawStar`, `roundRect`, `beep`). We add **shared FX helpers** once, then make small additive calls inside each game. Do not rewrite game loops; insert calls at existing event points (collect, hit, win). **Always `sed -n` the exact line before crafting an `old_string` — these are minified multi-statement lines.**

### Task 14: Shared particle-burst + screen-flash FX helpers

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (new JS block)

- [ ] **Step 1: Add a canvas-agnostic FX system (inside IIFE)**

Anchor (old_string): `/* === DRAMA-JS-END === */`
Replace with:
```javascript
/* Shared FX: short-lived particle bursts drawn on the active game canvas. */
const _fx=[]; // {canvasId,x,y,vx,vy,life,max,color}
function fxBurst(canvasId, x, y, color, count){
  count=count||14;
  for(let i=0;i<count;i++){
    const a=Math.random()*Math.PI*2, sp=1+Math.random()*3;
    _fx.push({canvasId,x,y,vx:Math.cos(a)*sp,vy:Math.sin(a)*sp,life:1,max:.018+Math.random()*.02,color:color||'#00e5ff'});
  }
}
function fxDraw(canvasId, ctx){
  for(let i=_fx.length-1;i>=0;i--){
    const p=_fx[i]; if(p.canvasId!==canvasId) continue;
    p.x+=p.vx; p.y+=p.vy; p.vy+=.04; p.life-=p.max;
    if(p.life<=0){ _fx.splice(i,1); continue; }
    ctx.save(); ctx.globalAlpha=Math.max(0,p.life); ctx.fillStyle=p.color;
    ctx.shadowColor=p.color; ctx.shadowBlur=10;
    ctx.beginPath(); ctx.arc(p.x,p.y,2.2*p.life+.6,0,Math.PI*2); ctx.fill(); ctx.restore();
  }
}
/* Screen flash: a brief white glow pulse over the whole app on a win. */
function screenFlash(){
  let el=document.getElementById('dramaFlash');
  if(!el){ el=document.createElement('div'); el.id='dramaFlash'; document.body.appendChild(el); }
  el.classList.remove('on'); void el.offsetWidth; el.classList.add('on');
}
window.__fxBurst=fxBurst;
/* === DRAMA-JS-END === */
```

- [ ] **Step 2: Add the flash element CSS**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 4: screen flash --- */
#dramaFlash{position:fixed;inset:0;z-index:45;pointer-events:none;background:radial-gradient(circle at 50% 50%, rgba(255,255,255,.55), rgba(0,229,255,.18) 40%, transparent 70%);opacity:0;}
#dramaFlash.on{animation:dramaFlash .5s ease;}
@keyframes dramaFlash{0%{opacity:0;}25%{opacity:1;}100%{opacity:0;}}
</style>
</head>
```

- [ ] **Step 3: Verify helpers exist and no errors**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Run: `preview_eval` → `typeof __fxBurst`
Expected: `"function"`.
Run: `preview_console_logs`. Expected: zero errors.

- [ ] **Step 4: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: shared particle-burst FX and screen-flash helpers"
```

---

### Task 15: Wire FX into game events (collect, hit, win)

Add `fxBurst`/`screenFlash` calls at existing game points and draw `fxDraw` inside each loop.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (`SparkFetch` loop ~463; `BugSweep` tap ~628 + loop ~629; `unlockGate` 440)

- [ ] **Step 1: Read each game's loop/tap to find stable anchors**

```bash
cd "/Users/andrew/welcome to vibe coding"
echo '--- SparkFetch loop ---'; sed -n '463,474p' vibe-coding-kit-dexgate-uiux-v9-drama.html | cut -c1-260
echo '--- BugSweep tap+loop ---'; sed -n '628,632p' vibe-coding-kit-dexgate-uiux-v9-drama.html | cut -c1-260
```
Note the exact `drawScene(g.ctx,w,h,'<LABEL>')` strings and the exact `collected++`/`score++` fragments. Use these verbatim as `old_string`s below.

- [ ] **Step 2: Add fxDraw to Spark Fetch loop + burst on spark collect**

Using the exact `drawScene(...)` call printed in Step 1 (e.g. `drawScene(g.ctx,w,h,'SPARK FETCH')`), append a draw call:
Anchor (old_string): `drawScene(g.ctx,w,h,'SPARK FETCH')`  ← replace with the exact string from Step 1
Replace with: `drawScene(g.ctx,w,h,'SPARK FETCH'); if(typeof fxDraw==='function') fxDraw('start', g.ctx)`

Then add a burst where a spark is collected. Using the exact collect fragment from Step 1 (e.g. `collected++; beep(`):
Anchor (old_string): `collected++; beep(`  ← replace with the exact fragment from Step 1
Replace with: `collected++; if(typeof fxBurst==='function'){fxBurst('start', s.x, s.y, '#00e5ff', 12);} beep(`

- [ ] **Step 3: Add burst to Bug Sweep on hit + fxDraw in its loop**

Using the exact hit fragment from Step 1 (e.g. `score++; beep(700,.03);`):
Anchor (old_string): `score++; beep(700,.03);`  ← exact from Step 1
Replace with: `score++; if(typeof fxBurst==='function'){fxBurst('pro', b.x, b.y, '#ff007f', 12);} beep(700,.03);`

Using the exact Bug Sweep `drawScene(...)` from Step 1:
Anchor (old_string): `drawScene(g.ctx,w,h,'BUG SWEEP')`  ← exact from Step 1
Replace with: `drawScene(g.ctx,w,h,'BUG SWEEP'); if(typeof fxDraw==='function') fxDraw('pro', g.ctx)`

- [ ] **Step 4: Trigger screenFlash on every gate win (central point)**

Anchor (old_string): `if(typeof dexExpress==='function') dexExpress('smug'); speak(gameName+' cleared. '+dexWinLine(),'smug');`
Replace with: `if(typeof screenFlash==='function') screenFlash(); if(typeof dexExpress==='function') dexExpress('smug'); speak(gameName+' cleared. '+dexWinLine(),'smug');`

- [ ] **Step 5: Verify FX run without errors**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Run: `preview_eval` → `__fxBurst('start', 100, 100, '#00e5ff', 10); 'ok'`
Expected: `"ok"` (no throw).
Run: `preview_console_logs`. Expected: zero errors.
Play Spark Fetch via preview interactions (start gate, drag to collect) and capture `preview_screenshot` — bursts should appear.

- [ ] **Step 6: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: particle bursts on collect/hit and screen flash on gate win"
```

---

### Task 16: Glow/pulse on game objects (sparks, hazards, bugs)

Add `ctx.shadowBlur`/`shadowColor` glow to the existing draw calls for game objects so they pulse and glow.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (`SparkFetch` spark/hazard draws; `BugSweep` bug draw)

- [ ] **Step 1: Read the spark/hazard/bug draw code**

```bash
cd "/Users/andrew/welcome to vibe coding"
echo '--- SparkFetch draws ---'; sed -n '463,474p' vibe-coding-kit-dexgate-uiux-v9-drama.html | cut -c1-300
echo '--- BugSweep draws ---'; sed -n '629,632p' vibe-coding-kit-dexgate-uiux-v9-drama.html | cut -c1-300
```
Locate the spark fill (`fillStyle=...; arc(...); fill()`), the hazard fill (red static), and the bug fill. Copy each exact fragment.

- [ ] **Step 2: Add a pulsing glow to sparks**

Wrap the spark-fill fragment (exact from Step 1) with a save/shadow/restore. Example pattern (substitute the real fragment as `old_string`):
If the spark draw is `g.ctx.fillStyle='#bff7ff'; g.ctx.beginPath(); g.ctx.arc(s.x,s.y,s.r,0,7); g.ctx.fill();`, replace with:
```javascript
g.ctx.save(); g.ctx.shadowColor='#00e5ff'; g.ctx.shadowBlur=12+6*Math.sin(performance.now()/200); g.ctx.fillStyle='#bff7ff'; g.ctx.beginPath(); g.ctx.arc(s.x,s.y,s.r,0,7); g.ctx.fill(); g.ctx.restore();
```

- [ ] **Step 3: Add a warning pulse to hazards (red glow)**

Wrap the hazard fill fragment (exact from Step 1) with `g.ctx.save(); g.ctx.shadowColor='#ff5567'; g.ctx.shadowBlur=8+6*Math.sin(performance.now()/160); ... g.ctx.restore();` around the fill.

- [ ] **Step 4: Add glow to bugs (Bug Sweep)**

Wrap the bug fill fragment (exact from Step 1) with `g.ctx.save(); g.ctx.shadowColor='#ff007f'; g.ctx.shadowBlur=12; ... g.ctx.restore();` around the fill.

- [ ] **Step 5: Verify visually + no perf regression**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Play Spark Fetch briefly via preview interactions. Run: `preview_console_logs`. Expected: zero errors.
Run: `preview_screenshot`. Expected: sparks/bugs visibly glow and pulse.

> Performance guard: `shadowBlur` is the most expensive Canvas op. Keep blur ≤ ~14 and only on small objects (sparks/bugs), never on full-canvas fills. If a low-end Chromebook stutters, drop the `Math.sin` pulse and use a constant blur.

- [ ] **Step 6: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: pulsing neon glow on sparks, hazards, and bugs"
```

---

### Task 17: Dramatic counter states (timer/leak color stages)

Bonewall's leak counter and Bug Sweep's timer should shift cyan → yellow → red as they approach failure.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (`<style id="drama-css">`; the counter update sites)

- [ ] **Step 1: Identify how each counter is rendered**

```bash
cd "/Users/andrew/welcome to vibe coding"
grep -n "meta-pricing\|meta-pro\|#meta\|textContent='Bugs\|textContent='Leak" vibe-coding-kit-dexgate-uiux-v9-drama.html | sed -n '1,20p'
```
Determine if the leak/timer is a DOM element (e.g. `#meta-pro`) updated via `textContent`, or drawn on canvas. Copy the exact update fragment.

- [ ] **Step 2: Add counter-stage CSS classes**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 4: counter danger stages --- */
.gameMeta.stage-ok{color:var(--cyan);text-shadow:0 0 6px rgba(0,229,255,.5);}
.gameMeta.stage-warn{color:#ffd23f;text-shadow:0 0 8px rgba(255,210,63,.6);animation:metaPulse 1s ease-in-out infinite;}
.gameMeta.stage-crit{color:var(--red);text-shadow:0 0 10px rgba(255,85,103,.8);animation:metaPulse .5s ease-in-out infinite;}
@keyframes metaPulse{0%,100%{opacity:.7;}50%{opacity:1;}}
</style>
</head>
```

- [ ] **Step 3: Apply stage classes when updating the counter (DOM case)**

Using the exact Bug Sweep timer update fragment from Step 1 (e.g. `$('#meta-pro').textContent='Bugs '+score`), replace with a version that also sets a stage by remaining `time`:
```javascript
(function(){var el=$('#meta-pro'); el.textContent='Bugs '+score; el.classList.remove('stage-ok','stage-warn','stage-crit'); el.classList.add(time>10?'stage-ok':time>5?'stage-warn':'stage-crit');})()
```
(Adapt variable names to the real code. If the counter is canvas-drawn instead, set `g.ctx.fillStyle` by the same `time>10 / time>5` thresholds and skip the CSS classes.)

- [ ] **Step 4: Verify the stage class flips**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Start Bug Sweep via preview interactions and let the timer run; Run: `preview_eval` →
```javascript
(document.getElementById('meta-pro')||{className:'n/a'}).className
```
Expected: contains `stage-ok` early, transitions to `stage-warn`/`stage-crit` as time drops. `preview_console_logs`: zero errors.

- [ ] **Step 5: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: cyan/yellow/red danger stages for timer and leak counters"
```

---

## Phase 5 — Animations, Interactions & Final Verification

### Task 18: Page transition animation

Slide+fade new pages in when `switchPage` activates them.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (`<style id="drama-css">`; `switchPage()` line 444)

- [ ] **Step 1: Add transition CSS**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 5: page transitions --- */
.page.active{animation:pageIn .34s ease;}
@keyframes pageIn{0%{opacity:0;transform:translateX(18px);}100%{opacity:1;transform:translateX(0);}}
</style>
</head>
```

- [ ] **Step 2: Scroll to top on page switch**

Anchor (old_string): `function switchPage(id){ activePage=id;`
Replace with: `function switchPage(id){ activePage=id; try{window.scrollTo({top:0,behavior:'smooth'});}catch(e){window.scrollTo(0,0);}`

- [ ] **Step 3: Verify transition fires**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Run: `preview_eval` →
```javascript
document.querySelector('.nav button[data-page="linux"]').click();
getComputedStyle(document.getElementById('page-linux')).animationName
```
Expected: `"pageIn"`. `preview_console_logs`: zero errors.

- [ ] **Step 4: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: slide/fade page transitions with smooth scroll-to-top"
```

---

### Task 19: Unlock sequence polish (overlay fade-away + content glow-in)

When a gate clears, the gate overlay should fade away and the content panel should glow-in. Because base CSS uses `.hidden{display:none !important}` (line 13), a transition won't show if we rely solely on `.hidden` — so we fade via inline opacity first, then add `.hidden` after 400ms.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (`<style id="drama-css">`; `unlockGate()` line 440)

- [ ] **Step 1: Add unlock CSS**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 5: unlock sequence --- */
.gateOverlay{transition:opacity .4s ease, transform .4s ease;}
.content.unlock-glow{animation:unlockGlow .8s ease;}
@keyframes unlockGlow{0%{box-shadow:0 0 0 rgba(0,229,255,0);}40%{box-shadow:0 0 10px rgba(0,229,255,.8),0 0 40px rgba(0,229,255,.4);}100%{box-shadow:var(--glow-cyan),var(--shadow);}}
</style>
</head>
```

- [ ] **Step 2: Add content glow-in + overlay fade-before-hide**

Anchor (old_string): `$('#overlay-'+page).classList.add('hidden');`
Replace with:
```javascript
(function(){var ov=$('#overlay-'+page); if(ov){ov.style.opacity='0'; ov.style.transform='translateY(-12px)'; setTimeout(function(){ov.classList.add('hidden');},400);} var ct=document.querySelector('.content[data-content="'+page+'"]'); if(ct){ct.classList.add('unlock-glow'); setTimeout(function(){ct.classList.remove('unlock-glow');},800);} })();
```

- [ ] **Step 3: Verify overlay fades and content glows**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Play Spark Fetch to a real win via preview interactions. After win (wait ≥1s), Run: `preview_eval` →
```javascript
JSON.stringify({overlayHidden: document.getElementById('overlay-start').classList.contains('hidden'), contentUnlocked: !document.querySelector('.content[data-content="start"]').classList.contains('locked')})
```
Expected: `{"overlayHidden":true,"contentUnlocked":true}`.
`preview_console_logs`: zero errors.

- [ ] **Step 4: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: unlock sequence — overlay fade-out + content glow-in"
```

---

### Task 20: Canvas tap ripple feedback

Every tap on a game canvas gets an immediate ripple, so no click feels dead.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (new JS block; `<style id="drama-css">`)

- [ ] **Step 1: Add a pointer ripple on all game canvases**

Anchor (old_string): `/* === DRAMA-JS-END === */`
Replace with:
```javascript
document.querySelectorAll('canvas[id^="game-"]').forEach(function(cv){
  cv.addEventListener('pointerdown', function(e){
    const wrap=cv.closest('.canvasWrap'); if(!wrap) return;
    const r=cv.getBoundingClientRect();
    const dot=document.createElement('span'); dot.className='tapRipple';
    dot.style.left=(e.clientX-r.left)+'px'; dot.style.top=(e.clientY-r.top)+'px';
    wrap.appendChild(dot); setTimeout(function(){dot.remove();},450);
  });
});
/* === DRAMA-JS-END === */
```

- [ ] **Step 2: Add ripple CSS**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 5: canvas tap ripple --- */
.canvasWrap{position:relative;}
.tapRipple{position:absolute;width:8px;height:8px;border-radius:50%;pointer-events:none;
  transform:translate(-50%,-50%);background:rgba(0,229,255,.6);box-shadow:0 0 12px rgba(0,229,255,.8);
  animation:tapRip .45s ease-out forwards;z-index:6;}
@keyframes tapRip{0%{opacity:.9;transform:translate(-50%,-50%) scale(.4);}100%{opacity:0;transform:translate(-50%,-50%) scale(4);}}
</style>
</head>
```

- [ ] **Step 3: Verify ripple element is created on pointerdown**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Run: `preview_eval` →
```javascript
var cv=document.getElementById('game-start'); var r=cv.getBoundingClientRect();
cv.dispatchEvent(new PointerEvent('pointerdown',{clientX:r.left+40,clientY:r.top+40,bubbles:true}));
cv.closest('.canvasWrap').querySelectorAll('.tapRipple').length
```
Expected: `1`. `preview_console_logs`: zero errors.

- [ ] **Step 4: Commit**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: instant canvas tap ripple feedback"
```

---

### Task 21: Idle card breathing

Subtle life on the side panels; reduced-motion already guarded by Task 0.

**Files:**
- Modify: `vibe-coding-kit-dexgate-uiux-v9-drama.html` (`<style id="drama-css">`)

- [ ] **Step 1: Add breathing CSS**

Anchor (old_string): `</style>\n</head>`
Replace with:
```html
/* --- Phase 5: idle breathing --- */
.dexPanel,.trophies{animation:cardBreath 4s ease-in-out infinite;}
@keyframes cardBreath{0%,100%{box-shadow:0 0 0 rgba(0,229,255,0);}50%{box-shadow:0 0 14px rgba(0,229,255,.12);}}
</style>
</head>
```

- [ ] **Step 2: Verify + commit**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Run: `preview_console_logs`. Expected: zero errors.
```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "feat: subtle idle breathing on side panels"
```

---

### Task 22: Mobile pass — 390px verification of every page and game

**Files:**
- Modify (only if issues found): `vibe-coding-kit-dexgate-uiux-v9-drama.html`

- [ ] **Step 1: Resize to phone width and walk all six pages**

Run: `preview_eval` → `localStorage.clear(); location.reload();` then dismiss welcome.
Run: `preview_resize` 390×800.
For each page id in `['start','tooling','pricing','beginner','linux','pro']`:
Run: `preview_eval` → `document.querySelector('.nav button[data-page="<id>"]').click();`
Run: `preview_snapshot` and `preview_screenshot`.
Expected: canvas not clipped, controls reachable, no horizontal scroll, recommendation grid collapses to one column.

- [ ] **Step 2: Verify the question modal fits on phone**

Run: `preview_eval` → `__askQuestion('start', ()=>{});`
Run: `preview_screenshot`. Expected: modal fully visible, four option buttons tappable, no overflow.
Run: `preview_eval` → `document.getElementById('qmodal').classList.remove('show');`

- [ ] **Step 3: Fix any overflow found**

If a panel overflows at 390px, add a targeted rule in `<style id="drama-css">` under a `@media (max-width:560px)` block (reduce padding / force single column). Re-verify.

- [ ] **Step 4: Commit (if changes were made)**

```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "fix: mobile (390px) layout pass for pages, games, modals"
```

---

### Task 23: Full end-to-end playthrough + final verification

The acceptance gate for the whole redesign.

**Files:**
- Modify (only if issues found): `vibe-coding-kit-dexgate-uiux-v9-drama.html`

- [ ] **Step 1: Clean-state run, dismiss welcome, confirm dry greeting**

Run: `preview_eval` → `localStorage.clear(); location.reload();`
Run: `preview_snapshot`. Expected: welcome heading "You're here. I noticed."; Dexter images visible in welcome, header brand, and sidebar.

- [ ] **Step 2: Play and clear all six gates in order, answering each question**

For each of the five gated pages, play to a win via preview interactions, then in the question modal click an option. Use a known set — click option **B every time** — to make the outcome deterministic.
After each clear, `preview_eval` →
```javascript
JSON.stringify({page:'<id>', cleared: !!JSON.parse(localStorage.getItem('dexgate_academy_state_v6')).gates['<id>'], answers: JSON.parse(localStorage.getItem('dexgate_academy_state_v6')).answers})
```
Expected: gate cleared true; the corresponding `qN` recorded as `"B"`.

- [ ] **Step 3: Clear the Bug Sweep boss and confirm the reveal**

After the boss (`pro`) clears, Run: `preview_eval` →
```javascript
JSON.stringify({rec: JSON.parse(localStorage.getItem('dexgate_academy_state_v6')).recommendation, revealed: !document.getElementById('recPanel').classList.contains('hidden')})
```
Expected: `{"rec":"claude","revealed":true}`.

> Determinism check for all-`'B'`: q1B→claude2, q2B→claude2+codex1, q3B→claude2, q4B→idx1+claude1, q5B→idx1+claude1 ⇒ claude=8, idx=2, codex=1 ⇒ **claude**. Confirm `rec === "claude"`.

- [ ] **Step 4: Confirm Dexter's pick text matches the winner**

Run: `preview_eval` → `document.querySelector('.nav button[data-page="pro"]').click();` then
`document.getElementById('recPick').textContent.split('\n')[0]`
Expected: `🎯 DEXTER'S PICK: Claude Code`, and `.recCard[data-tool="claude"]` has class `is-pick`.

- [ ] **Step 5: Zero-error sweep at both breakpoints**

Run: `preview_console_logs`. Expected: zero errors across the whole session.
Run: `preview_resize` 1280×900 and 390×800; `preview_screenshot` each. Expected: both polished, no clipping.

- [ ] **Step 6: Persistence check**

Run: `preview_eval` → `location.reload();` then dismiss any welcome.
Run: `preview_eval` →
```javascript
JSON.stringify({gates:Object.keys(JSON.parse(localStorage.getItem('dexgate_academy_state_v6')).gates).length, rec: JSON.parse(localStorage.getItem('dexgate_academy_state_v6')).recommendation})
```
Expected: `{"gates":6,"rec":"claude"}` — progress and recommendation persist across reload.

- [ ] **Step 7: Capture final hero screenshots and commit**

Run: `preview_screenshot` desktop + phone (the "wow on open" proof).
```bash
git add vibe-coding-kit-dexgate-uiux-v9-drama.html
git commit -m "test: full end-to-end playthrough verified — gates, questions, recommendation"
```

---

### Task 24: Promote v9 to canonical + changelog

**Files:**
- Create: `CHANGELOG_V9.md`
- (Optional) Update: `index.html` if it is meant to mirror the latest build

- [ ] **Step 1: Write the changelog**

Create `CHANGELOG_V9.md`:
```markdown
# DexGate Academy v9 — Maximum Drama

Built from v8-verified. Single HTML file, no external deps.

## Added
- Neon glow system: glowing headings, card edge-glow, button states, locked/unlocked distinction.
- Animated background (drifting particles + breathing gradient) and toggleable CRT scanlines.
- Dexter expression engine: per-page expressions, in-game reactions, idle boredom, dry welcome + victory lines.
- Tool Recommendation System: five questions (one per gated page), deterministic scoring engine, three-tool breakdown page, and Dexter's personalized pick.
- Game elevation: particle bursts on collect/hit, screen flash on win, pulsing glow on sparks/bugs/hazards, danger-stage timer/leak counters.
- Interaction polish: page transitions, unlock sequence (overlay fade + content glow-in), canvas tap ripples, idle card breathing.
- State migrated to v6 (preserves v5 progress; adds answers + recommendation). Honors prefers-reduced-motion.

## Verified
- All six gates completable; five questions captured; recommendation revealed after the boss.
- Zero console errors; layouts checked at 1280px and 390px.
```

- [ ] **Step 2: Decide canonical filename with the user**

Confirm whether to overwrite `index.html` (the repo's served page) with v9 or keep v9 as a named build. **Do not overwrite `index.html` without confirmation.**

- [ ] **Step 3: Commit**

```bash
git add CHANGELOG_V9.md
git commit -m "docs: add v9 Maximum Drama changelog"
```

---

## Challenges & Risk Notes

- **Editing minified one-liners:** the base CSS (7–48) and most JS are single long lines. **Always read the exact line with `sed -n 'Np'` before crafting an `old_string`.** Never guess a minified substring. New CSS is appended in the separate `<style id="drama-css">` block to avoid touching base CSS entirely.
- **IIFE scoping:** new logic that touches `state`/`speak`/`pageDefs` MUST live inside the IIFE (before the init line). Functions exposed for verification are attached to `window.__*` explicitly — those are the only globals.
- **`.hidden{display:none !important}` vs animation:** `display:none` kills transitions. Task 19 handles this by setting opacity/transform first, then adding `.hidden` after the 400ms fade.
- **Canvas `shadowBlur` cost:** the single biggest perf risk. Restrict glow to small objects, cap blur ≤ ~14, never apply to full-canvas fills, and drop the sine pulse if a low-end Chromebook stutters (Task 16 note).
- **Two modals stacking:** the legacy `#modal` "Gate cleared" and the new `#qmodal` can both open on a win. Verified order is question-on-top; only suppress the legacy modal if Task 11 verification shows it's intrusive.
- **Determinism of the recommendation:** the engine is the one piece with a single correct answer. It is verified against explicit input→output pairs (Tasks 9, 13, 23). If a table changes, re-run those assertions.
- **XSS hygiene:** all new dynamic DOM is built with `createElement` + `textContent`, never `innerHTML` (the question options and Dexter pick text are user-adjacent and must stay text-only).
- **Single-file size:** the file is ~195KB (mostly base64). Adding CSS/JS keeps it well under any practical limit; no new assets are introduced.

## Build Order (MVP → polish)

1. **Phase 0** (Tasks 0–1): safety copy + state migration. *Required first.*
2. **Phase 1** (Tasks 2–5): visual system — the immediate "wow." *MVP.*
3. **Phase 3** (Tasks 9–13): recommendation system — the functional core/point of the app. *MVP.* (Depends only on Phase 0; can precede Phase 2.)
4. **Phase 2** (Tasks 6–8): Dexter's expanded role. *MVP-adjacent; small.*
5. **Phase 4** (Tasks 14–17): game elevation. *Polish.*
6. **Phase 5** (Tasks 18–23): interaction polish + full verification. *Polish + acceptance.*
7. **Task 24:** promote + changelog.

> Minimum shippable slice = Phases 0 + 1 + 3 (impressive look + working recommendation funnel). Phases 2/4/5 layer on top without reworking earlier tasks.
