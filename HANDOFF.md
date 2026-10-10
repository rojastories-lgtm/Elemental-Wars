# Elemental Wars — Session Handoff (v0.40)

> Paste this into a new Claude Code session on the repo `rojastories-lgtm/Elemental-Wars`.
> The full code is NOT pasted here on purpose: `index.html` is ~3 MB (mostly embedded base64 art + an MP3).
> The new session should read the code directly from the repo, using the line map below to jump around.

---

## 1. Core Game Overview
A browser card + dice battler in **one self-contained file** (`index.html`), hosted on GitHub Pages (`.nojekyll` makes Pages serve it as is).

- **Heroes (5):** Electro Hero (L, Lightning), Ice Queen (I), Dark Hero (D), Steel Hero (S, 4 core slots instead of 3), Volcanic Lord (V). Each has 8 abilities and its own card group. A **Time Hero** shows up only as a "coming soon" placeholder.
- **Turn loop:** `income` (roll dice → turn faces into resources) → `main1` (play cards/abilities) → `combat` → `main2` → end turn.
  - Die faces: 1 = Fire, 2 = Water, 3 = Nature, 4 = Hero element, 5–6 = Any (wild).
  - Limits: 30 HP to start, 4-card starting hand, at most 10 of each resource.
- **Systems:** element wheel (`BEATS`), core stones (damage, heal, destroy), Burn (max 8, can be cooled with Water), frozen dice, drains, shields/reflect, defend/respond windows, negate cards, Dark Hero's Shadow bar, Steel Hero's forge (False Core).
- **Modes:**
  - **Versus:** pass & play, vs AI, or online.
  - **PvE dungeon** (solo or co-op, local or online): a branching map across 7 floors with minion lairs, elites, core-stone crystals, ley lines and `?` events (spring, shrine, library, cache, merchant). It ends at the boss **Malkorr** (100 HP), who uses his own dice faces.
  - **Rewards:** card drafts, relics (an extra die), gold, and pair-combo ultimates.

## 2. Current Working State

### Added in v0.39–v0.40
- **Two acts.** Act 1 ends with a **Corrupted Hero**: one of the heroes nobody picked, using its real abilities and resources.
  - It rolls 6 dice solo, 8 in co-op. Each turn it plays a corruption card, and corrupted dice are skipped on your next roll.
  - HP is 75 in co-op and 45 solo. It uses at most 3 abilities a turn (2 solo).
  - These are tuning guesses; balance is unverified.
- Beating it ends Act 1: heal half your missing health, pick a core, and get a new map. Act 2 has tougher monsters and Malkorr.
- **Music:** two embedded MP3s, each about 2 MB.
  - "Lanterns in the Maw" plays on the map, events and crystal screens.
  - "Stone Vault Pulse" plays in PvE battles.
  - They replace the old dungeon track.
- Map text is now readable in dark mode.
- Fixes: Cinder Curse no longer soft-locks waiting for the enemy; hitting an enemy that holds resources no longer waits for a block.
- The file is now ~7 MB, and the line numbers in the code map below have shifted. Grep for names instead.

### Complete and shipped (per the in-game changelog, through v0.40)
- All 5 heroes, versus modes, and the AI opponent (versus only).
- Online play through Firebase with "your move" alerts.
- The full PvE dungeon:
  - Map and events.
  - Gold and the merchant.
  - 8 neutral scroll cards.
  - The enemy intent display redesign.
  - Timed-effect expiry (fixed in v0.38).
- Undo, local save/resume, story intro, guide/rules pages, sound effects, procedural music plus a dungeon MP3, and the version-history badge.

### Partial or placeholder
- **Time Hero:** UI placeholder only (`TIME_TH`, `TIME_ABIL`, `timeGuideHTML`). It has no cards and isn't in `HEROES`.
- **AI bot:** plays versus only. It is heuristic: it scores cards and effects, and always plays seat 1.
- **Online co-op simultaneous moves:** these rely on `coopMerge`. A rare "the other player acted at the same moment" toast still cancels a move.
- **Tests:** there are none. Everything is verified by playing manually.

## 3. Key Constraints & Architecture Decisions
- **Single file, no build step, no framework.** Plain JS and template-string HTML; `render()` rebuilds the DOM from state. This is intentional so the game opens directly from disk or GitHub Pages. Keep it that way unless you decide otherwise.
- **Immutable-style state transitions.** Every action does `const T = clone(S)`, mutates `T`, then calls `commit(T)` / `commitA(T, seat, label)`.
  - `commit` bumps `ver`, re-renders, and in online play writes to Firebase.
  - Writes take a short lock (`acquire`), do a version check, and on a version conflict either merge (co-op) or reject (versus).
- **Undo:** stores the previous state, but is disabled after any action that rolled dice or drew cards (`rolledFlag`). This prevents rerolling for a better result.
- **Effect engine:** card and ability effects are data (`ABIL`, `CFX`). `expand()` turns them into a queue `S.q`, and `advance()` / `runStep()` / `runOp()` process it.
  - Steps in `INPUT` (defend, respond, hex, seal, burn window…) pause the queue for player input (`pendingInput`).
- **Seat model:** in versus, seats are 0 and 1. In co-op, the heroes are seats 0 and 1 and the enemy side is seat 2.
  - The helpers `FOE`, `TURN` and `STEP` hide this difference.
  - Always use them instead of writing `1 - seat`.
- **Bounded logs and effects:** the log keeps 80 entries, the sound queue 10 and the flash queue 6. This keeps Firebase documents small.
- **Firebase:** web SDK 12.19.0, loaded from gstatic. The config is public on purpose (Firebase web keys aren't secret; security rules do the protecting). Opening the game as a local `file://` page makes online play unavailable (`EW_onlineError = 'file'`).
- **Fonts:** Bangers (display) and Fredoka (body) from Google Fonts. Colors are CSS variables with light/dark themes.

## 4. Art & Visual Pipeline
- **All raster art is embedded** as base64 WebP in `const IMG = {...}` (line ~1107):
  - `<hero>_face`, `<hero>_card` and `<hero>_banner` for each of electro, ice, dark, steel and volcanic.
  - `boss`.
  - Minions: `m_abyss`, `m_ember`, `m_frost`, `m_root`, `m_sand`, `m_tempest`, `m_volt`, `m_wave`.
  - Story frames: `s_pano`, `s_squirrel`, `s_firesquirrel`, `s_mech`, `s_unleash`, `s_electro`.
- **To add art:** convert the image to WebP, base64-encode it, add a key to `IMG`, and reference it as `IMG.key`. Hero images are wired up automatically through `HEROES[k].art()` / `.card()`. Keep images small, because the file size grows quickly.
- **Vector icons:**
  - `ICONS`: inline SVG for each element. `pip()`, `inlineIcon()` and `richText()` turn `{F}`-style tokens in card text into icons.
  - `ARTICONS`, `place()`, `burst()` and `artSvg()` compose card art from icons.
  - `THEME` holds hero colors; `COL` holds element colors.
  - Map icons are inline SVG (`MAP_DEMON`, `MAP_CRYSTAL`).
- **Audio:**
  - All sound effects and the versus music are synthesized with WebAudio (`playSfx`, `tone`, `nburst`, `musicTick`).
  - The dungeon track is an embedded base64 MP3 (`DUNGEON_MP3`, near the end of the file).
- **Layout:** comic style (thick borders, yellow buttons), responsive up to `#app` max-width 1280px. Overlays: `#modal`, `#toast`, `#splash`, `#flashes`, `#guide`.

## 5. Code Map (`index.html`, approximate line numbers)
| Lines | What |
|---|---|
| 1–1010 | CSS |
| 1012–1105 | Firebase wrapper (`EW_FIREBASE`, `makeDb`, `makeUser`) |
| 1107 | `IMG` (huge base64 line — avoid printing it) |
| 1108–1175 | Icons, element tables, `THEME` |
| 1176–1345 | `CARDS`, `HEROES` |
| 1346–1460 | Rules constants, `ABIL`, `CARDLIST`, `CFX`, card sets |
| 1460–1930 | Rules engine: resources, damage, cores, burn, roll/collect, phases, effect queue |
| 1933–2050 | Global state, `commit` / `commitA` / undo, modals |
| 2050–2660 | UI: dice, actions, dialogs, panels, `render()` |
| 2658–2870 | Sound effects, animations |
| 2867–3275 | Rules/guide screens, lobby, online create/join, local start |
| 3280–3460 | Versus AI bot |
| 3466–3550 | Procedural music |
| 3553–3675 | Game list, notifications, local save, vitals/draw animations |
| 3676–4380 | PvE: minions, boss, relics, map, events, enemy AI, co-op UI |
| ~4583 | `GAME_VERSION` + `VERSION_HISTORY` (bump this with every change) |
| ~4660 | Story intro |
| ~4900 | `DUNGEON_MP3` |

Tip: use `grep -n` with `cut -c1-150`. Never `cat` the whole file, because it would flood the context.

## 6. Conventions
- For each change: bump `GAME_VERSION` and add player-facing notes to `VERSION_HISTORY`.
- Commit messages so far: "Update Elemental Wars game".
- Push to the session's designated branch.

## 7. Next Steps / Immediate Goals
_No explicit to-do list was recorded in the previous session. These are inferred from the code — edit before pasting:_
1. Playtest and balance the Corrupted Hero (HP, ability cap) and Act 2 difficulty.
2. Build the **Time Hero** (element `T` / hourglass already exists in `EL`; placeholder data in `TIME_ABIL`).
2. Playtest the v0.38 map, events, merchant and gold balance.
3. Reduce conflict toasts in online co-op.
4. Possibly extend the AI to the PvE dungeon, or make the versus bot smarter.
5. Your own list: ________________________________
