# Elemental Wars: Handoff (v0.41, October 2026)

> **For the next Claude session.** This replaces the earlier v0.40 handoff.
> Read this first, then work from `index.html`.
> The file is about 17 MB because of embedded art, animations and music, so grep for names instead of opening it whole.

---

## 1. What to give the new chat

| File | Why |
|---|---|
| **`index.html`** | **The game** (v0.41): one self-contained file with all code, art, animations and music. This is the file to edit. |
| **`HANDOFF.md`** (this file) | Context, rules, code map, decisions and the to-do list. |
| `elemental-wars-rulebook.pdf` | Comic-style 10-page rulebook. *Out of date* on the Time Hero and the element wheel (see §8). It's generated from the game, so it can be rebuilt. |
| `electro-hero-cards-print.pdf` | Printable Electro deck at Magic sleeve size (63×88 mm), with discard values. Same generator works for other heroes. |
| `element-tracker-*.pdf/png` | Printable 3.80×2.75 in resource tracker boards, one per hero plus a generic one. |
| `beta.html` | Identical to `index.html` except the version label says `0.41-beta`. Not needed unless you want a separate test build. |

- **Repo:** `rojastories-lgtm/Elemental-Wars`.
- **Default (live) branch:** `claude/focused-feynman-a7mtbt`. There is no `main`. GitHub Pages serves `index.html` from it.
- **Work branch:** `claude/compassionate-bohr-55ldst`. The newest work (Time Hero rebalance, dungeon-win upgrades, new battle music, tracker boards) is pushed here **but not merged into the default branch yet**. Merge it to make it live.

---

## 2. The game in one paragraph
A browser card + dice battler in one HTML file.
- **Six heroes:** Electro (Lightning), Ice Queen (Ice), Dark Hero (Darkness), Steel Hero (Steel, 4 core slots), Volcanic Lord (Lava), Time Hero (Time).
- **Each hero has** 8 abilities, a 30-card deck (15 hero + 15 generic) and four resources: Fire, Water, Nature and their own element.
- **Turn:** Income (roll 6 dice, collect resources) → Main 1 (cards) → Combat (abilities, once each) → Main 2 → End.
- **Win:** bring the rival from 30 to 0.
- **Modes:**
  - **Versus:** pass & play, vs AI, or online through Firebase.
  - **PvE dungeon:** solo or co-op, local or online. Two acts of branching maps. Act 1 ends with a Corrupted Hero (an unpicked hero turned boss). Act 2 ends with Malkorr, the Elemental Overlord.

---

## 3. Everything that changed since the first handoff (v0.40 → v0.41)

### Heroes and cards
- **Decks** are now exactly 15 generic + 15 per hero.
  - Removed from decks (kept in `CARDLIST` so saved games still load): Drain Resource, Flip Side, Loaded Dice, Piercing Core, Null Ward, Rock Slide.
  - Second Chance rerolls up to 4 dice. Blaze Core costs 3 Fire.
- **Electro Hero**
  - **Passive "Charged":** after 2 lightning hits, all his lightning deals +1 until he takes damage. Shown by a big ⚡+1 token on his banner.
  - Fire Blast 1 unblockable + 3, Static Shock 2 + 2, Lightning Strike 3 unblockable + 2.
  - Fire Sync 1 unblockable + 2, counted as lightning damage.
  - Grounding Wave (1 Lightning, 2 Nature): 1 unblockable, charges him right away, refreshes another used ability.
  - Chain Lightning is an instant. Mirror Veil reflects half. Tesla Coil counts as an instant.
- **Ice Queen**
  - Glacial Armor: 2 shield, +1 if she unfreezes a die.
  - Frozen Fortress: 2 shield, +1 per die she unfreezes.
  - Meltwater: heal 2, freeze 1.
  - Glacier Core: in the Defend window, unfreeze frozen dice to block 1 each. No free blocking.
  - Cold Storage is free. Whiteout costs 3 Ice.
- **Dark Hero:** Twist Fate cancels 2 of the enemy's next dice. Shadow Step only prevents damage.
- **Steel Hero**
  - Forge button fixed.
  - Iron Wall and Quench now upgrade a core: +1 max health and +1 current health.
- **Volcanic Lord**
  - Pyroclast and Magma Fist (2 damage) place a **burn token** on an enemy ability. Using it costs the owner 2 health. 1 Water quenches it in versus. In the dungeon it goes on the monster's move, and a boss with a token takes 2 each turn.
  - Ash Cloud is a core. Volcano Core costs 4 Lava. Burn caps at 10 in the dungeon.
- **Time Hero** (new, fully playable)
  - Every ability runs on a **timer**: switch it on, it ticks down 1 each time you enter combat, and it fires at 0 on the target chosen at activation.
  - Current numbers: Sand Shield 3 shield (t1), Delayed Blast 4 (t2), Rewind heal 3 + 1 resource (t1), Time Bomb 6 (t3), Erosion 2 dmg + enemy loses 2 resources (t2).
  - Instant timer tricks: Accelerate (tick −1, +1 dmg), Borrowed Time (+1 wait, +3 dmg), Temporal Collapse (all tick −1, anything that fires gets +2).
  - **His cards feed timers** instead of acting directly:
    - Time Bolt +2, Chrono Strike +4 (waits +1), Echo Strike +5 (waits +1), Age to Dust / Time Warp +3 and tick.
    - Sands of Fate +2 dmg +2 shield. Pendulum and Chrono Storm affect all timers.
    - Rewind Wounds: +1 wait, +3 heal. Hourglass Shield: +1 wait, +3 shield.
    - Temporal Rift fires every timer now. Tick Tock / Haste / Eternity tick timers.
  - Damage cards need a running damage timer.
  - **Time is weak to every element** and beats nothing.
  - Art: hero card from the "TIME HERO" title card; portrait/banner from "Time's Bifurcation".

### Combat and UI
- Enemy hits on core stones fly into that core, which shakes. Damaged cores show corner cracks.
- **Combo ultimate needs both players to agree:** the proposer picks a target, the partner gets Agree / Decline, and nobody pays until they agree.
- **Tooltips** on nearly every button, shop item, reward, mystery-room option and map node. Mouse only.
- **🐛 Bug button** (bottom right) opens a pre-filled GitHub issue with the player's note and a game-state snapshot. The player must submit it on GitHub.
- **Debug mode:** type `5684` (keyboard) to toggle a 🐞 panel. It edits resources, health, gold, dice, hand, cores and items, and jumps to any fight, boss, mystery room or act. It's remembered per device.

### PvE dungeon
- **Rewards:** after a fight, a card **or** 1 of every resource, plus gold. Elites give a powerful core but **no longer an extra die**. Extra dice come only from ley lines and the rare **Wishing Well** event.
- **Depth scaling:** +5% monster health per floor, harder hits deep down. Act 2 is tougher.
- **New monsters**
  - Act 2: Hollowed Knight, Ashen Cultist, Plague Priest, Fallen Warlord (elite), Storm Wraith, Magma Elemental, Void Rift.
  - Extra: Obsidian Titan, Starlight Wyvern, Thunderhoof Kirin, Gale-Wing Griffin, Tidal Angler, Magma-Crust Behemoth, Overgrowth Spider, Iron-Bark Warden (elite), Bloom Vanguard, Ice-Forged Paladin, Frost-Bound Weaver.
  - **Ruin Hydra** segmented elite: 3 heads, and killing one makes the others stronger.
- **Monster mechanics**
  - **Curses** deal damage each round until reflected with the element shown.
  - **Element-immune** monsters can only be hurt by certain elements or Burn. If a hero has no ability of a valid element, the monster becomes vulnerable to that hero's main element (Fire) for the fight.
  - Guard, immunity and burn tokens are shown as badges.
- **Corrupted Steel Hero** forges up to 3 targetable corrupted cores (Anvil, Bastion, Furnace). Each gives his attacks +1 and powers his core abilities.
- **Items** (shown under the abilities): Gambler's Charm, Ember Censer, Healing Potion, Frost Bomb, Mirror Shard.
- **Shop:** buy as many things as you can afford. Each item is one of a kind, so once bought it's SOLD for the partner.
- **Mystery rooms:** Spring, Shrine, Library, Cache, Merchant, Gambler's Table, Altar of Tribute, Cursed Chest, Echo of a Fallen Hero (learn another hero's attack), Mirror of Excess (2× resources, 2× damage), Blood Altar (trade a die for +3 damage), Dice Forge (gild, wild-cut or build a die), Wishing Well.
- **Map rules**
  - The unclaimed core stone can't appear on the last 2 floors before Malkorr.
  - **Discards don't auto-reshuffle** in the dungeon. Used cards stay out until the pre-boss deck refresh.
- **Dungeon wins and permanent upgrades** (stored in the browser's `localStorage` under `ewLegacy`)
  - Each win lets you pick one face of your next die to give +1 resource forever, starting at die 1, then die 2, and so on.
  - The dungeon menu shows wins and upgrades and has a Reset option with an "Are you sure?" step.
  - Applies to every hero in local games, and to the host only online.
- **Visuals**
  - Dark dungeon backdrop, and an arena for boss fights.
  - "Living" monster art: breathing, sway and element particles.
  - Animated hydra heads.
  - **Real animations** (from the artist's idle videos, cropped only, no edits) for the Sand Titan, Ember Stalker, Tempest Skylord and Abyssal Devourer.
- **Music**
  - Map: "Lanterns in the Maw".
  - Battles: random each fight from Stone Vault Pulse, Dungeon Dance, Subterranean Pulse, Dungeon Pulse and Minor Key Dungeon. The new four are mono at about 48 kbps to keep size down.

### Art pipeline used
- **New hero art:** cards with the old name plates pasted under the new art (Dark Hero keeps its own title). Banners and portraits are cropped from the new art. Steel still uses old art.
- **Enemy art:** cropped from the artist's sheets, picture only, with no frame or name plate.

---

## 4. Decisions and preferences (follow these)
- **Show previews before committing visual or animation changes** to the artist's material. Never "fix" or blend their animations: crop and convert only.
- Keep it **one self-contained HTML file** with no build step. Art is base64 WebP in `IMG`, and music is base64 MP3.
- Keep the player-facing changelog: add notes to `VERSION_HISTORY` for every change.
- `index.html` and `beta.html` are kept in sync. The only difference is the version label.
- Balance numbers are first guesses. Expect the user to retune after playtesting, especially Act 2 difficulty, the Corrupted Hero bosses and the Time Hero.
- The user rejected sprite-sheet animation (Sand Titan sheet). Use their idle videos instead.

---

## 5. How the code is organised (important)
- The original v0.40 code makes up the first ~5,100 lines of the main `<script>`. All v0.41 work is **appended after it** in clearly marked blocks:

| Marker (grep for it) | Contents |
|---|---|
| `v0.41 BETA ADDITIONS` | Deck rules, Time Hero, new cards, items, events, Act 2 monsters, curses, immunity, Electro passive, backgrounds |
| `enemy art (from the artist` | Enemy art and 11 new monsters |
| `core stone damage: cracks` | Core cracks |
| `batch 3: Electro rework` | Electro abilities, burn tokens, Time timers (engine) |
| `combo ultimate: the partner` | Ultimate agreement |
| `living monster art` | Breathing / particles |
| `Ice Queen: less free armor` | Ice changes (+ Glacier Core edits inside `defendDialog`) |
| `shop: buy as much` / `tooltips:` | Shop and tooltips |
| `Corrupted Steel Hero forges` | Boss cores |
| `DEBUG MODE` | Debug panel |
| `element-immune monsters must stay beatable` | Immunity safety net |
| `animated monster art` | Animated WebP monsters |
| `bug reports:` | Bug button |
| `Time Hero rebalance` | Current Time numbers + timer-feeding cards |
| `Dungeon wins & permanent dice upgrades` | Legacy upgrades + dungeon menu |
| `more battle music` | Random battle tracks |
| `/* ---------- version ---------- */` | `GAME_VERSION` + `VERSION_HISTORY` (stays last) |

- **Wrapper pattern:** most features **re-wrap existing functions**, for example `const _x0 = runOp; runOp = function(...){ ...; return _x0(...); }`.
  - Many functions are wrapped several times: `runOp`, `expand`, `runPacket`, `heroDamage`, `damageCore`, `coopStep`, `setupFight`, `gather`, `coopPrompt`, `coopEnemyHTML`, `abilitiesHTML`, `playerPanel`, `mitBadges`, `intentText`, `doCollect`.
  - **The last definition in the file wins.** To change a behaviour, find the latest wrapper, or add a new one at the end before the version block.
- **Card ids must never shift**, because saved games store them. New cards are only ever appended (`NEW_CARDS`, minion cards, etc.).
- Core engine facts:
  - Every action clones the state (`T=clone(S)`), changes it, then calls `commitA`.
  - Effects are data: `ABIL`, `CFX` → `expand` → queue `S.q` → `advance`.
  - Seats: 0/1 are heroes, and seat 2 is the enemy in the dungeon. Use `FOE` / `TURN` / `STEP`.
  - Online merges co-op moves via `coopMerge`.

---

## 6. Testing approach used
- There are no unit tests. Headless Chromium (Playwright) scripts drove the real page:
  - **Engine-level dungeon simulation:** an auto-played run through both acts, for all heroes, solo and co-op, checking for crashes and soft-locks.
  - **Targeted engine checks** per feature.
  - **Screenshots and GIFs** of the UI.
- Every change was syntax-checked by parsing each `<script>` with `new Function`.

---

## 7. Known issues and caveats
- **File size is about 17 MB.** The first load is slower. Music is the biggest part, then art and animations.
- **Tooltips** don't show on touch devices. **Debug mode** needs a keyboard.
- **Permanent upgrades** are stored per device, not per account. Online they apply to the host only.
- **Bug button:** needs GitHub Issues enabled on the repo, and the player must be logged into GitHub.
- **Combo ultimate agreement** is only tested in local co-op, not online.
- **Element-immune monsters** made fights slow in simulations. Watch Act 2 balance.
- **The versus AI** doesn't use Glacier Core thawing, the refresh from Grounding Wave, or burn-token targeting choices (it auto-picks).
- **The corrupted Time Hero boss** ignores timers: its abilities fire immediately.

---

## 8. Next steps / open items
1. **Merge the work branch into the default branch** so the latest changes go live.
2. **Regenerate the rulebook PDF:**
   - The Time Hero section is outdated: old numbers, and it says "Time beats Darkness" (Time is now weak to everything).
   - Add the newer systems: burn tokens, ultimate agreement, dungeon-win upgrades.
3. **Playtest and rebalance:** Act 2 difficulty, the Corrupted Hero bosses, the Time Hero after its rebalance, and element-immune monsters.
4. **More animated monsters:** the user may send more idle videos. Crop the frame and name plate only, then convert to animated WebP (12 fps, ~400 px wide). Set `IMG.m_<key>` and `MINIONS.<key>.animated = 1`. Show a preview before committing.
5. **Printables for other heroes:** card sheets like the Electro one.
6. **Possible:** a touch-friendly way to unlock debug mode, and account-based (Firebase) storage for upgrades so they work for both players online.
7. The user's own list: ______________________
