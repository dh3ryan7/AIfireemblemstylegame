# TacticsForge — *Embers of Valden*

A complete, offline, single-file **Fire Emblem–style tactical RPG** that runs in any modern
browser with **zero install**. All art is drawn programmatically (Canvas + CSS) and all sound
is synthesized with the Web Audio API — including a procedural chiptune soundtrack (title, march,
enemy-phase, battle, and boss themes) — there are **no external assets and no network calls**.

> Double-click **`index.html`** and play. That's it.

---

## How to run
1. Open `index.html` in Chrome, Edge, or Firefox (double-click it, or drag it onto the browser).
2. Click **New Game**, pick **Normal** or **Hard** (and optionally **Casual** = no permadeath).
3. Play through a 6‑chapter campaign.

Nothing to build, no server, no dependencies. Saves are kept in‑memory with an opportunistic
browser autosave, **and** you can save to a real file (`.tfsave`) from the world map, results
screen, or in‑battle menu, and load one at the title via **Load from File** — so progress is never trapped in browser storage.

---

## Controls
| Action | Keyboard | Mouse |
|---|---|---|
| Move cursor | Arrow keys | Move mouse |
| Confirm / select / attack | `Space` or `Enter` | Left‑click |
| Cancel / back / deselect | `Esc` or `Backspace` | Right‑click |
| Open field menu (End Turn, etc.) | confirm on an empty tile | click empty tile |
| End turn | `E` | field menu → End Turn |
| Toggle enemy **danger zone** | `D` | field menu |
| Show a unit's range | `T` (cursor on unit) | click a unit |
| Switch attack/heal target | `◀ ▶` | click target |
| Advance dialogue | `Space` / `Enter` | click |
| Inspect unit / Class Compendium | `C` (in battle or world map) | right-click roster chip · "Class Guide" buttons |
| Toggle combat log | `L` | ≡ Log button (top bar) |
| Action-menu hotkeys | `A` Attack · `S` Staff · `I` Item · `T` Talk · `W` Wait · `Z` Seize | click a row |

**Mouse-only is fully supported:** click a unit → click a blue tile to move → pick an action, or just
click a red foe to attack (hover to preview the forecast). The forecast has **Attack / Back** buttons,
and the **End Turn**, **☰ Menu**, and **⚠ Danger** buttons live in the top-right HUD. Menus are also fully
navigable by keyboard (arrows + Space).

---

## How to play (the short version)
- **Goal:** complete each chapter's **objective** (shown top‑left). You lose if **Lyra** falls
  or your whole army is wiped (in **Classic** mode, dead units are gone for good).
- **Blue tiles** = where the selected unit can move. **Red tiles** = where it can attack.
  Select a unit → move → choose **Attack** → pick a target. A **forecast** shows damage, hit %,
  crit %, and **×2** when you double (Attack Speed 4+ higher than the foe).
- When combat starts, the game cuts to a **side-view battle scene** (Fire Emblem style): your unit on the
  left, the foe on the right — they dash in, swing, fire arrows or cast spells, dodge, and crit with the HP
  bars ticking down. Prefer speed? Toggle **Battle animations** off in **Options** for quick on-map effects.
- **Weapon triangle:** Sword › Axe › Lance › Sword, and Anima › Light › Dark. Advantage gives
  **+damage and +hit**. **Effective** weapons triple their might: bows vs fliers, armorslayers
  vs armored, horseslayers vs cavalry. **Magic** hits **Res** instead of **Def** — your answer
  to heavy armor.
- **Terrain matters:** forests/hills/mountains grant Def & Avoid; forts and thrones also **heal**.
  Fliers ignore movement cost but are weak to bows. **Hold chokepoints** and let foes break on you.
- **Grow:** win combat for EXP (100 = level up, with a per‑stat roll). At **Lv 10+** use a matching
  **Seal** to **promote** into a stronger class. Manage inventories and the shared **convoy** in the
  prep screen, and **recruit enemies** by moving an ally adjacent and choosing **Talk**.
- **Supports:** certain pairs build bonds by ending player phases within 3 tiles of each other,
  granting Hit/Avoid/Crit bonuses while within 2 tiles and unlocking short conversations.
- **Read the battle:** the arena panels show live **HIT / DMG / CRT** for both sides, `L` opens a
  color-coded **combat log**, and `C` opens a full **Unit Report** (stats vs caps, growths, ranks,
  supports) or the 28-class **Compendium**. Music, sound, and battle animations toggle in **Options**.

Stuck on a wall of Generals? Bring a **mage** (hits Res), an **armorslayer** (effective), or a
**Hammer**. Getting picked off at range? Press **`D`** to see exactly which tiles are dangerous.

---

1. **The Raid on Brookhollow** — *Rout* (tutorial pacing; a safe setup turn, then a bridge fight).



---

## How to extend it (data‑driven)
Everything is plain data near the top of the `<script>` in `index.html`.

**Add a weapon** — add an entry to `WEAPONS`:
```js
my_blade:{name:'My Blade',type:'sword',rank:'C',mt:9,hit:80,crit:10,wt:6,range:[1,1],uses:30,eff:['flier']},
```
`type` ∈ sword/lance/axe/bow/anima/light/dark/staff. `eff` tags (`armor`,`cavalry`,`flier`,`infantry`)
make it triple‑might effective. Staves use `healBase`/`staff:true`.

**Add a class** — add to `CLASSES` (with `caps`, `mov`, `move:'foot|horse|armor|fly'`, `weapons`,
`tags`, `sprite`, and optional `promo`/`promoTag`). Give it a `CBASE`/`CGROW` line for stat scaling.

**Add a recruitable unit** — add to `ROSTER` (`cls`, `level`, `growths`, `items`, `ranks`, `skills`,
and a `port` palette for the portrait). Reference it from a chapter as a starting member, a `joins`
arrival, or a talk‑recruit.

**Add a chapter** — copy any `CH#` object and edit:
```js
const CH7={ id:'ch7', name:'…', worldPos:{x:0.5,y:0.5},
  objective:{type:'rout'|'seize'|'boss'|'defend'|'survive'|'escape', turns?, tile?, tiles?, escortId?},
  map:{w,h,tiles:[ "################", … ]},   // each row exactly w chars; see TERRAIN keys below
  deploy:{tiles:[{x,y},…]}, forced:['lyra'], joins:['roland'],
  enemies:[ E('brigand',6,x,y,{ai:{behavior:'guard'},items:['iron_axe'],isBoss?,drop?}),
            {rosterId:'finn',x,y,recruitBy:'lyra',recruitScript:[…]} ],   // talk-recruit
  reinforcements:[{turn,side:'enemy',units:[E(…)]}],
  villages:[{x,y,reward:'speedwing',text:'…'}],
  pre:[{sp:'lyra',side:'left',text:'…'}], post:[…] };
CHAPTERS.push(CH7);
```
**Terrain keys** (in `map.tiles`): `.`plain `_`road `g`grass `f`forest `h`hill `m`mountain
`F`fort `G`gate `T`throne `v`village `p`pillar `b`bridge `~`water `#`wall.
**AI behaviors:** `aggressive`, `guard` (holds until you close in), `boss` (never leaves its tile),
`flee` (with `goal:{x,y}`); healers auto‑heal. Bosses can carry skills (e.g. `skills:['vantage']`).

---

## Files
- **`index.html`** — the entire game (engine, content, art, audio). This is the deliverable.
- `DESIGN.md` — architecture & combat‑formula reference.
- `CHANGELOG.md`, `TESTING.md` — what was built and how it was verified.

## Developer / QA console (`window.TF`)
Open DevTools and run, e.g.:
```js
TF.simMany(0, 30)              // headless AI-vs-AI sim of chapter 1, 30 runs (stability/balance)
TF.simChapter(5, {levelTo:16}) // simulate the finale with a level-appropriate roster
R.instant = true               // make animations resolve instantly (useful for fast testing)
```

Enjoy retaking Valden. **The swan flies again.**
