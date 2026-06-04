# DexGate Academy v9 — Maximum Drama

Built from `vibe-coding-kit-dexgate-uiux-v8-verified.html`. Single HTML file, no external dependencies, no build step. All images remain base64-embedded.

File: `vibe-coding-kit-dexgate-uiux-v9-drama.html`

## Added

**Visual system (Phase 1)**
- Neon glow system: multi-ring glow tokens (`--glow-cyan/pink/violet`), `.glow-text`/`.glow-edge` utilities, amplified glowing headings and gate names.
- Card edge-glow, hover lift, button hover/active/disabled states, and a clear locked (dimmed) vs unlocked (bright, glowing) distinction.
- Subtle animated background: drifting particle layer + slowly breathing gradient (CSS-only, `pointer-events:none`).
- Toggleable CRT scanline overlay (off by default, persisted in state, header `▤` button).

**Dexter's expanded role (Phase 2)**
- Central expression engine (`dexExpress`) driving the sidebar and header brand images together, with a pop animation on change.
- Per-page resting expressions; Dexter reacts to mistakes (Floppy Dex crash) and goes bored when idle.
- Dry, deadpan rewrite of the welcome greeting ("You're here. I noticed.") and a rotating pool of smug victory lines.

**Tool Recommendation System (Phase 3) — the functional core**
- Five questions, one per gated page (Spark Fetch → Crostini Bridge), presented in a forced, Dexter-fronted modal (safe DOM, no `innerHTML`; first option auto-focused for keyboard users).
- Deterministic scoring engine (`computeRecommendation`) over Project IDX / Claude Code / OpenAI Codex with an idx>claude>codex tie-break.
- Three-tool breakdown section on the Pro Vault page (capabilities, strengths, weaknesses, pricing) plus Dexter's single personalized pick (why, what to do next, why not the others), with the chosen card highlighted.
- Answers and the computed recommendation persist in state and re-reveal on reload.

**Game elevation (Phase 4)**
- Shared particle-burst FX and a full-screen win flash.
- Bursts on spark collect and bug hit; flash on every gate clear.
- Pulsing neon glow on sparks, hazards, Bug Sweep bugs, and Bonewall bugs (blur capped ≤14 for performance; never on full-canvas fills).
- Cyan/yellow/red danger stages for the Bug Sweep timer and Bonewall leak counter.

**Interaction polish (Phase 5)**
- Slide/fade page transitions; smooth scroll-to-top on navigation.
- Unlock sequence: gate overlay fades out, content panel glows in (works around `.hidden{display:none}` by fading via inline opacity before hiding).
- Instant tap ripple on every game canvas.
- Subtle idle "breathing" on the side panels.

**Infrastructure**
- State migrated `v5` → `v6`, preserving existing gate/secret progress and adding `answers`, `recommendation`, and `scanlines`.
- Honors `prefers-reduced-motion` (all animations/transitions collapse to near-zero duration).

## Verified
- `node --check` passes on the full inline script; DOM structure balanced (6 pages, 6 canvases, 2 style blocks, 1 script).
- Recommendation engine determinism confirmed against fixed vectors (e.g. all-A → IDX, all-B → Claude Code, empty → IDX tie-break).
- Full funnel verified live in-browser: all five questions captured via the real modal, reveal fires, pick text + highlight correct, and it persists across reload. Engine drives the UI (different answer sets yield different winners).
- Mobile (375px) question modal fits and is fully usable.
- Zero console errors on fresh load and throughout the funnel.

## Residual manual check
- Each of the six canvas games still needs a human to actually *play to a win* once, to eyeball the in-game FX (bursts/flash) and the gate-clear → question hand-off. The underlying game win/fail logic is unchanged from the verified v8 build; only additive FX calls were inserted.
