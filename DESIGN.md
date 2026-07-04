# TacticsForge — Design & Architecture Doc

A Fire Emblem–style tactical RPG. Single self-contained `index.html`. No build step, no
network, no external assets. Double-click to run. All art is generated programmatically
(Canvas sprites + CSS). All sound is generated with the Web Audio API.

## 1. Design pillars
1. **Authentic FE tactics.** Real GBA-style combat math (2RN hit, doubling at AS+4,
   weapon triangle, effective damage, crits, skills).
2. **Coherence over breadth.** Every listed core system fully implemented, not stubbed.
3. **Data-driven content.** Classes, weapons, items, skills, units, and chapters are
   plain data objects so the campaign is easy to author and extend.
4. **Testable engine.** Pure logic (combat, pathfinding, AI) is separated from rendering
   and exposed on `window.TF` so playtests can be scripted headlessly.

## 2. Architecture (modules, all in one file)
- **DB** — static data tables: `TERRAIN`, `CLASSES`, `WEAPONS`, `ITEMS`, `SKILLS`,
  `UNITS` (recruit templates + growths), `CHAPTERS` (maps, objectives, spawns, scripts).
- **rng** — seedable PRNG (Mulberry32) so playtests are reproducible; `roll2RN` for hit.
- **Unit** — runtime instance: stats, level/exp, class, inventory, equipped weapon,
  position, hp, statuses, skills, faction, AI profile, support partners.
- **Stats module** — derived stats: atkSpeed, avoid, effective stats with class caps,
  support bonuses, terrain bonuses, level-up rolls, promotion.
- **Combat module** (pure) — `forecast(attacker, defender, map)` returns
  `{dmg, hit, crit, doubles, ...}` for both sides; `resolveCombat()` runs the exchange
  with skill procs and returns an event log for the animator.
- **Map/Grid** — tiles `{terrain, unit}`; `movementCost(class,terrain)`,
  `dijkstraReachable(unit)` (move range honoring movement type), `attackTilesFrom(set)`,
  `aStarPath(a,b)` for move animation.
- **AI module** — per enemy: enumerate (reachable tile × target) actions, score by
  expected damage / kill / triangle / terrain / target value / self-preservation, pick
  best; behaviors: `aggressive`, `guard` (springs once you enter its reach), `boss` (never
  leaves its tile), goal-seeking `flee` (via `ai.goal`). Healers auto-heal wounded allies.
- **Scene system** — finite state machine: `title → worldmap → dialogue → battle →
  results → worldmap …`; plus `gameover`. Dialogue scripts are arrays of commands
  (`say`, `enter`, `exit`, `recruit`, `bg`, `wait`).
- **Battle controller** — turn loop (player phase / enemy phase / other(green) phase),
  cursor + selection state machine, menus (Move→Attack/Item/Talk/Trade/Wait/Seize),
  objective checking, reinforcements, win/lose.
- **Render** — Canvas layer for terrain/units/overlays/effects (60fps rAF loop, error-guarded
  since v1.6.1 so one bad frame can't kill the animation chain; tween/particle queue); DOM
  layer for HUD, info panels, forecast, roster column, menus, dialogue, world map, title.
  Internal resolution fixed; CSS-scaled to fit.
- **BA (battle scene)** — FE-style side-view combat arena: terrain platforms, strike
  choreography, hit-stop, crit close-ups, per-element spell fx. Per-class fighter models
  register on the `BA.classBody` dispatcher (sprite key → body-renderer function). Purely
  visual — `resolveCombat` applies game state first, so the scene can be skipped or fail safely.
- **MUSIC** — procedural chiptune player (square lead / triangle bass / noise percussion)
  with five looping tracks (title / map / enemy phase / battle / boss), a lookahead
  scheduler, crossfades, autoplay-policy unlock, and an Options toggle persisted to localStorage.
- **Stat screens & combat log** — Class Compendium and per-unit Report panels; a scrollable
  color-coded combat log (strikes, procs, deaths, EXP) toggled with `L` / the ≡ Log button.
- **Save** — full game state serialize/deserialize to JSON; export to `.tfsave` file
  download and import via file picker (no reliance on localStorage; localStorage used as
  an opportunistic autosave only).

## 3. Combat formulas (GBA-authentic)
```
Atk      = (phys) Str + Mt(+triangle±1)(+eff ×3 on Mt) | (mag) Mag + Mt
Defense  = (phys) Def | (mag) Res ; +terrain def
Damage   = max(0, Atk - Defense)            (×3 crit)
AS       = Spd - max(0, Weight - Con)
Hit      = Mt.Hit + Skl*2 + Luck/2 + triangle(±15) + support
Avoid    = AS*2 + Luck + terrainAvo + support
HitRate  = clamp(Hit - Avoid, 0, 100), rolled with 2RN (avg of two d100)
Crit     = Mt.Crit + Skl/2 + classCrit + support - target.Luck
Double   = attacker.AS - defender.AS >= 4  (and can counter only within range)
```
Weapon triangle: Sword > Axe > Lance > Sword; Anima > Light > Dark > Anima.
Advantage = +1 Mt, +15 Hit; disadvantage = −1 Mt, −15 Hit.

## 4. Skills (proc/passive)
Sol (Skl% heal ½ dmg dealt), Luna (Skl% halve foe Def/Res), Vantage (strike first when
HP<50% & attacked in range), Pavise (Skl% halve incoming dmg), Wrath (+crit when HP<50%),
Renewal (heal 20% max HP at turn start), Adept (Skl% bonus attack), Charm (allies within
2 tiles +10 Hit/Avoid), Pursuit (follow-up on any AS lead), Lethality (Skl/2% instant
kill), Discipline (double weapon-rank exp). Bosses/lords have signature skills.

## 5. Classes & promotions (caps + movement type)
Lord→Great Lord, Mercenary→Hero, Myrmidon→Swordmaster, Soldier→Halberdier,
Knight→General(armored), Cavalier→Paladin(cav), Archer→Sniper, Mage→Sage,
Cleric→Bishop, Pegasus Knight→Falcon Knight(flier), Fighter→Warrior, Thief→Assassin.
Movement types: infantry, armored, cavalry, flier. Fliers ignore terrain move cost but
take effective dmg from bows. Armored take effective dmg from armorslayer/hammer; cavalry
from horseslayer.

## 6. Maps & objectives (campaign, 6 connected chapters)
1. **Prologue — Rout** (bandit raid; tutorial framing).
2. **Ch.2 — Seize gate** (chokepoint + forts; talk-recruit a defector).
3. **Ch.3 — Defend 6 turns** (hold the village line vs waves + reinforcements).
4. **Ch.4 — Escort/Escape** (move the lord party to the escape tiles; protect a green NPC).
5. **Ch.5 — Survive 7 turns** (ambush; reinforcements every turn; recruit a pegasus knight).
6. **Ch.6 — Seize throne / Boss kill** (villain on a throne with guards; finale).

## 7. Data formats
- **Map**: width/height + `tiles` (rows of terrain-id chars) + named regions
  (seize/escape tiles, fort lists).
- **Chapter**: `{id,name,objective,map,player starts,enemies[],greens[],reinforcements[],
   recruits, preDialogue, postDialogue, victory, defeat}`.
- **Unit template**: base class, level, personal growths, starting inventory, skills,
  portrait palette, AI profile (for enemies).

## 8. Milestones (each ends with a real playtest)
0. Design doc (this).  1. Vertical slice: one chapter end-to-end + AI + win/lose + juice.
2. Classes/weapons/skills/promotion/inventory/convoy/level-ups.  3. Full campaign +
dialogue/portraits + world map + supports + talk-recruits + all objective types.
4. Polish: title, settings, save/load files, controls help, Web Audio, accessibility.
5. QA pass (edge cases) + balance. See TESTING.md / CHANGELOG.md.
All of the above shipped; post-plan passes followed (see CHANGELOG.md): M5.x FE-style battle
scene & cinematics, M6.x mouse-first UI/HUD, M7.x aggressive AI + character models, M8/v1.5
stat screens + cleanup + test loadouts, M9/v1.6 procedural music + in-combat stats/log +
per-class models, v1.6.1 animation crash-proofing.

## 9. Accessibility & anti-softlock
Colorblind-safe faction colors (blue/teal allies, red/orange enemies, green NPCs) plus
shape/letter badges; every objective has a guaranteed path to resolution; "End Turn" and
"Restart Chapter" always reachable; casual-mode toggle disables permadeath.

## 10. Implemented vs intentionally deferred (v1.6.1)
**Implemented:** 6-chapter campaign covering every objective type; GBA-authentic combat
(2RN, triangle, effective ×3, doubling, crits); 28 classes with seal promotions; 11 skills;
supports (C/B/A + conversations); talk-recruits, story joins, and a mid-battle defection;
inventory + shared convoy + stat boosters; enemy AI (aggressive/guard/boss/flee, auto-healers);
FE-style battle scene with per-class models and cinematics; procedural chiptune music +
synthesized SFX; Class Compendium + Unit Report; combat log; danger zone; mouse-first and
full-keyboard UI; colorblind-safe badges; Normal/Hard + Casual; `.tfsave` export/import,
autosave, and mid-battle resume; `window.TF` headless QA harness.
**Deferred by design:** gold/shops (vestigial gold system removed in v1.5), fog of war,
rescue/canto/pair-up, arena & random skirmishes, reclassing/second seals, multiple manual
save slots (one autosave + unlimited file exports), localization, touch/mobile layout.
