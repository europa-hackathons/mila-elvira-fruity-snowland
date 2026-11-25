# Fruity Snowland

**Fruity Snowland** (*"The Sweetest Snowball Arcade!"*) is a single-file, zero-dependency HTML5 Canvas arcade catching game built for and inspired by **Mila & Elvira** (authored in this repository by Dmitry - `Maxim's Dad`).

Set in front of a moonlit, snowy castle, players spend a two-minute round catching falling fruit, grabbing temporary Extended Arms power-ups, and dodging brown and yellow snowballs. Desktop browsers open in two-player competition mode; mobile devices open in the original single-player mode with character color customization and the local Top 5 leaderboard. Desktop users can switch modes from the main screen. Two-player teams have distinctive berry/gold star and blue/mint snowflake outfits.

## Run it

No build step, server, or package installation is required:

Open [index.html](index.html) directly in any modern desktop or mobile web browser (double-click the file or drag it into a browser tab).

## Files

| File | Description |
| --- | --- |
| [index.html](index.html) | Self-contained game application: responsive arcade-cabinet CSS, semantic HTML panels, inline SVG character avatars, HTML5 Canvas renderer, game loop, Web Audio / Speech Synthesis engine, and `localStorage` leaderboard. |
| [README.md](README.md) | Project overview and documentation. |

## Controls

### Desktop
- **Two-player (default)**: Player 1 uses **`A` / `D`**; Player 2 uses **`←` / `→`**.
- **Single-player mode**: Use **`←` / `→`** to move.
- **`🔊` / `🔇` button** (top-right HUD): Toggle background music, sound effects, and spoken phrases.

### Mobile
- Single-player mode is always loaded and cannot be switched.
- **`◀` / `▶` on-screen buttons**: Press and hold to move left or right.

### High-Score Entry (Game Over)
- **`A`–`Z` keys** or **on-screen letter pad**: Enter your 3-letter initials.
- **`Backspace`** or **`⌫ DELETE`**: Remove the last letter.
- **`Enter`** or **`SAVE MY SCORE! 🏆`**: Save your score to the Top 5 board.

## How the game is organised

Everything lives inside [index.html](index.html) in three sections:

1. **Styling (`<style>`)** — Retro arcade cabinet frame, subtle CRT scanline overlay, HUD chips, start/game-over modal cards, character & color swatch pickers, letter pad, shake/pulse animations, and responsive breakpoints (`≤ 620px` for mobile touch controls; `max-height: 700px` for compact desktop viewports).
2. **Markup (`<main class="game-shell">`)** — Brand header, cabinet wrapper, full-bleed `<canvas id="gameCanvas">`, HUD (`SCORE`, `TIME LEFT`, `BEST`, sound toggle), active status badges (lucky fruit & extended arms timer), start panel, game-over/leaderboard panel, and mobile controls.
3. **Game Engine (`<script>` IIFE)**:
   - **Data & State**: Constants (`ROUND_SECONDS`, `STORAGE_KEY`), character/color/fruit tables, and a single `state` object managing phase (`"start"`, `"playing"`, `"over"`), player position, falling objects, particles, popups, combo state, freeze/upgrade timers, and audio handles.
   - **Character Picker & Avatars**: `avatarSvg`, `renderCharacterChoices`, and `renderColorChoices` dynamically generate SVG portraits reflecting each character's traits and selected coat/hat color.
   - **Canvas Rendering**: `drawBackground` (starry sky, crescent moon, castle silhouette, snowy ground), `drawPlayer` (coats, scarves, hair styles, glasses, scar, freckles, extended arms, frozen ice block), `drawObject`, `drawParticles`, and `render` (with screen-shake support).
   - **Game Loop & Physics**: `tick` runs via `requestAnimationFrame` (clamping frame delta to `0.05s`), updating the countdown timer, speed stage, object spawning (`spawnObject`), and circle-distance collision checks (`catchItem`).
   - **Audio & Speech**: Synthesized tones and looping chiptune melody via the Web Audio API (`playTone`, `soundCatch`, `playMusicStep`) plus spoken callouts via the Web Speech API (`speakPhrase`, `playPhraseTune`).
   - **Persistence & Events**: `safeScores` / `saveScores` validate and persist the top 5 scores in `localStorage`; `wireEvents` binds keyboard, pointer, window `blur`, and `resize` listeners.

## Specifications

### Round & Difficulty Progression
- **Round length**: 2 minutes (`120` seconds). The timer switches to an urgent pulsing style when `≤ 30s` remain.
- **4 Speed Stages**:
  | Stage | Time Elapsed | Speed Multiplier | Banner |
  | --- | --- | --- | --- |
  | Level 1 | `0–30s` | `1.0×` | — |
  | Level 2 | `30–60s` | `1.35×` | `SPEED UP!` |
  | Level 3 | `60–90s` | `1.8×` | `SPEED UP!` |
  | Level 4 | `90–120s` | `2.4×` | `FINAL BLIZZARD!` |
- **Spawn interval**: `max(0.25s, 0.78s / multiplier)`.

### Playable Characters
Each character has a **Lucky Fruit** worth `+10` bonus points and can wear any of **7 coat/hat colors** (*Berry pink, Grape purple, Ice blue, Mint green, Tangerine, Cherry red, Snow white*):

| Character | Role | House | Lucky Fruit |
| --- | --- | --- | --- |
| Luna Lovegood | Dreamy believer | Ravenclaw | 🍊 Orange |
| Ginny Weasley | Brave seeker | Gryffindor | 🍓 Strawberry |
| Ron Weasley | Loyal jokester | Gryffindor | 🍎 Apple |
| Hermione Granger | Clever scholar | Gryffindor | 🫐 Blueberry |
| Harry Potter | Courageous hero | Gryffindor | 🥝 Kiwi |
| Neville Longbottom | Kind-hearted hero | Gryffindor | 🍍 Pineapple |

### Falling Objects & Scoring
- **Fruits** (~58.6% of spawns):
  - 🍓 Strawberry: `10` pts
  - 🍊 Orange: `10` pts
  - 🫐 Blueberry: `10` pts
  - 🍎 Apple: `12` pts
  - 🥝 Kiwi: `15` pts
  - 🍍 Pineapple: `20` pts
- **Lucky Fruit Bonus**: `+10` extra points on top of the fruit's base value.
- **Combo Bonus**: Catching fruits less than `2.2s` apart increments a combo; streaks of `3+` add `+10` bonus points (`COMBO!`, or `SUPER!` at `5+`).
- **Extended Arms Upgrade (`↔` orb, 7% of spawns)**: Active for `10s`, expanding fruit catch reach from `26px` to `70px` (hazards keep the standard `26px` reach).
- **Hazards — Brown & Yellow Snowballs (~34.4% of spawns, 50/50 split)**: `-15` points (score floors at `0`), resets combo, freezes the player for `3s` (no movement or catches), and triggers screen shake.

### Leaderboard Storage
- Saved locally in browser `localStorage` under the key `milaElviraFruitySnowlandScores`.
- Keeps the top 5 entries validated as 3 uppercase ASCII letters (`^[A-Z]{3}$`) and non-negative integer scores.

## Others

- **Audio & Spoken Callouts**: Uses the browser's Web Audio API for music/SFX and the Web Speech API (`speechSynthesis`) for playful callouts (*"Lucky!"*, *"Extended arms"*, *"Emotional Damage"*). Two-player callouts use distinct voices, with pitch/rate fallbacks when the browser only provides one English voice. Both degrade gracefully if unsupported or muted.
- **Display & Accessibility**: Scales canvas rendering for HiDPI screens (capped at `2×` `devicePixelRatio`), includes ARIA labels and `aria-live` regions for status updates, and clears held movement keys if the browser window loses focus.
- **Offline-Ready**: Uses native emoji and system fonts with zero external network requests.
