# Changelog

## [M0] Design & scaffold
- Wrote DESIGN.md (architecture, formulas, milestones).
- Single-file `index.html` shell: CSS UI, canvas, HUD, dialogue/title/world/modal scaffolds.

## [M1] Vertical slice — one chapter end to end (verified)
- Core engine: seedable RNG + 2RN hit, Web Audio SFX.
- Data tables: terrain, weapon & magic triangle, 35 weapons/staves/items, 11 skills, 28 classes with caps/promotions/movement types.
- Unit model: per-class bases & growths, derived stats, weapon ranks, equip, EXP, level-up, promotion.
- Combat module: GBA-authentic damage/hit/avoid/crit/AS-doubling, weapon triangle, effective damage, skill procs (Sol/Luna/Pavise/Vantage/Wrath/Adept/Renewal/Charm/Lethality), supports, terrain, EXP/staff.
- Pathfinding (Dijkstra by movement type), move/attack/threat ranges, fliers ignore terrain.
- Enemy AI: action scoring (kill potential, triangle, terrain, target value, self-preservation); behaviors aggressive/guard/boss/healer; pathfinding approach.
- Renderer: camera, programmatic terrain & unit tokens, HP bars, overlays, tween/particle/damage-number system, screen shake, crit flash, portrait generator.
- Battle controller: player/enemy/ally phases, cursor + selection state machine, action menu (Attack/Staff/Talk/Trade/Item/Seize/Wait), forecast panel, target select, undo move, objectives, win/lose, reinforcements.
- Scenes: dialogue runner w/ portraits & typewriter, title, world map, prep/deploy, inventory & convoy management, results, game-over/retry, options, how-to-play.
- Save/load: file export/import (.tfsave), localStorage autosave/continue, mid-battle resume.
- QA harness `window.TF`: headless AI-vs-AI battle simulation; `R.instant` fast-forward.
- Chapter 1 authored (Rout) with story dialogue; map redesigned for a turn-1 safe buffer & deep bridge chokepoint.

### Verified in M1
See TESTING.md. Built & ran in a real browser (no console errors); validated pathfinding,
forecast, combat, promotion, weapon triangle, effective damage, supports, talk-recruit,
full enemy-phase AI, real select→move→attack flow, and 200-turn stability with no soft-lock.

## [M2] Progression systems (folded into the engine, verified)
- Promotion via Seals (Lv10+) from prep, convoy, and in-battle Item menu.
- Inventory management + shared convoy + stat boosters + weapon equip/trade.
- Weapon-rank growth with use; per-unit growth-rate level-ups with stat-roll display.

## [M3] Full campaign
- Six connected chapters covering every objective type: Rout, Seize, Defend, Escape/Escort,
  Survive, and Seize-throne/Boss finale; rising difficulty; reinforcement waves; village rewards.
- Narrative: lord (Lyra), 12 recruitable units, villain arc (Varkas + Morwen), opening & ending,
  pre/post-battle dialogue with generated portraits & typewriter text.
- World map / chapter select with progress, army management, and replay.
- Recruitment: story `joins` (Roland), talk-recruits (Bryce, Mira, Dorian, Finn), and a
  mid-battle defecting flier (Aria).
- Support pairs (auto-linked by template) with bonuses + support conversations at C/B/A.
- Escort/escape logic: escortee death = defeat; lord on escape tile wins.

## [M4] Polish & accessibility
- Colorblind-safe faction badges (triangle = ally, square = enemy, diamond = NPC) in addition to colour.
- Title screen, how-to-play panel, options (sound/danger-zone), pause/field menu, restart-chapter.
- Screen shake + crit flash + floating damage + death fade + level-up panel juice.
- Danger-zone overlay default option; immediate info-card on battle start.

## [M5] FE-style battle scene (combat animations)
- Combat now cuts to a side-view **battle arena** (like the GBA Fire Emblem games), then returns to the map.
- Terrain-themed backdrops (meadow/forest/fort/water/mountain) with parallax hills, ground texture, and motifs.
- Programmatic animated **chibi fighters** built from shapes, colored by faction + each character's portrait
  palette, with class-specific weapons (sword/lance/axe/bow/tome/staff) and helmet/hood/circlet/wing headgear.
- Full strike choreography: wind-up → dash-in melee swing (or nocked **arrow** / cast **spell orb** for ranged/magic)
  → impact. Misses trigger a **dodge hop**; hits flash the target, drain the HP bar, and spawn floating damage.
- **Critical hits**: zoom-in, white flash, red vignette, heavy screen shake, big "pop" damage number.
- EFFECTIVE / skill-proc (Sol, Luna…) / heal labels, death **collapse-and-fade**, and intro/outro fades.
- FE-style HP panels (name, class, Lv, weapon, ticking HP gauge with segment marks) for both combatants.
- Default **on**, with an Options toggle ("Battle animations") that falls back to the quick on-map effects.

## [M5.1] Battle-scene refinement pass
- **Hit-stop**: the action freezes for ~70ms on a hit / ~130ms on a crit (with the frame still rumbling) — the single biggest game-feel upgrade.
- **Per-class attack motions**, dispatched by weapon: swords/axes do windup→swing (axes heavier, with extra shake); **lances thrust** straight forward; **cavalry charge** further; **bows** draw the string back then loose; **mages** raise their hands as a glow builds, then cast.
- **Per-element spells**: Fire = flying fireball → ground flames; Thunder = a lightning bolt struck from above; Light = a radiant pillar; Dark = a swirling purple vortex.
- **Weapon trails**: a fading motion-blur ribbon follows the blade through melee swings.
- **Knockback** on the struck unit, scaled up on crits.
- **HP "damage ghost"**: the lost HP chunk shows as a lagging white trail before settling (FE-style).
- **Crit close-up**: stronger zoom toward the attacker, white flash, red vignette, and radial speed-lines.
- **Multi-slash flourish** for myrmidon/swordmaster-type units and on crits; **dodge-hop** on a miss.
- **Dramatic death**: white flash + particle burst + knockback collapse-and-fade.
- Fighters got stance/stride shifts, cape sway, a polished bow with a nocked arrow, and clearer weapon art.
- New synthesized SFX: weapon whoosh, bow draw/release, spell charge, and per-element cast sounds.

## [M5.2] Battle-scene cinematics
- **Terrain platforms**: each combatant now stands on a platform matching *their own* map tile — stone parapet on a fort, grassy mound + brush in forest, rocky crag on a mountain, wooden planks on a bridge, grassy field by default.
- **Camera push-in on engage**: the scene cuts in slightly zoomed and pulls back to frame both fighters as they take stance.
- **Boss intro flourish**: the first time you fight a named commander, a red banner sweeps across with their name and an ominous stinger (shown once per boss).
- **Low-HP tremble**: badly wounded fighters breathe heavier and shake while they can still stand.

## [M6] Mouse usability & UI refinement
- **Fixed the "clicks don't work" bug**: the unit info card overlapped the bottom-left tiles and was
  swallowing clicks meant for units beneath it. It is now `pointer-events:none` (click-through).
- **Persistent HUD buttons** (top-right): **End Turn**, **☰ Menu**, **⚠ Danger** — the game is now fully
  playable with the mouse alone, no keyboard required. Buttons grey out on the enemy phase.
- **One-click attacking**: in target mode, click a foe to attack, hover to preview its forecast, or click
  away to go back. The forecast panel also has explicit **Attack / Back** buttons (and **Heal / Back** for staves).
- **Click off an open menu** now cancels it (like right-click/Esc).
- Attacking a foe directly now pre-selects *that* foe in the forecast (not just the first in range).
- **Redesigned stats card**: faction-colored class chip, name/level/EXP, big HP, an 8-tile stat grid,
  a weapon line with a weapon-type color dot, and skill tags — clearer and more readable.

## [M6.1] More UI polish
- **Roster column** (right edge): a clickable chip per deployed unit (class glyph, name, HP bar) — click to
  jump the camera and select that unit, hover to inspect it, with acted/selected states shown. Auto-hides
  during the battle scene, dialogue, and cutscenes. (Every map is 16 wide, so it sits in the margin and
  never covers the board.)
- **Passive hover forecast**: with a unit selected, hovering a reachable foe shows the full combat preview
  (damage, hit%, crit%, doubling, both directions) without committing — just like FE — and hides when you move off.
- Bigger action-menu / list targets for easier mouse clicking.

## [M6.2] Menu click reliability
- **Fixed a real overlap bug**: opening **Item**/**Trade** then **Back** left the item panel (higher z-index)
  sitting on top of the action menu, so clicks landed on the invisible panel instead of menu rows like **Wait**.
  Menus now hide any other open panel before showing, and entering Attack/Staff targeting clears them too.
- Action menu no longer rebuilds its DOM on hover (which could drop an in-flight click); it just re-highlights.
- Bigger action-menu targets; click isolation on menu rows.
- Added a **build version stamp** ("build v1.3") on the title screen so it's obvious whether a copy is current.

## [M7] Aggressive AI + cooler characters
- **AI attacks far more reliably.** The old decision gate refused any attack scoring below a threshold,
  so enemies often declined winnable trades and looked passive. Now an enemy attacks whenever it can deal
  real damage (or score a kill) and the move isn't pure suicide-for-nothing — and "guard" units spring as
  soon as you come within their reach. In a clustered test, 6 of 7 enemies chose to attack (the 7th was
  repositioning to reach). It still focuses the best target (wounded/squishy/kills) and avoids throwing
  its life away for a scratch.
- **Cooler battle-scene characters (class-distinct bodies):**
  - **Cavaliers/Paladins** ride an animated **warhorse** (galloping legs, mane, tail, faction barding).
  - **Pegasus/Falcon Knights** ride a **winged pegasus** that hovers with flapping wings.
  - **Mages/Sages/Clerics/Bishops** wear flowing **robes** instead of legs, with a hood/hat and tome glow.
  - **Archers/Snipers** carry a **quiver of arrows** on the back.
  - **Knights/Generals** are bulkier with bigger pauldrons; all classes got gradient-shaded armor, face
    shading, and a two-tone cape for depth.

## [M7.1] Individual characters & boss designs
- **Bosses now look the part.** Commanders use their portrait palette in the battle scene (Varkas in his
  blood-red crown, Morwen in her dark hood, Garrok with his orange trim) and radiate a pulsing **red aura**.
- **Per-enemy variety**: generic foes get varied hair/skin (by unit id) and class-appropriate headgear, so a
  squad of bandits isn't a row of clones — while named recruits/lords keep their authored colors.
- **Map tokens read by movement type**: cavalry tokens get a little **mount base + legs**, fliers get
  **wings**, so you can spot horses and pegasi at a glance on the board.
- Added a portrait for Aldric (the chancellor) and Captain Vorn.

## [M7.2] Critical AI turn bug
- **Enemies now act every turn, not just turn 1.** `runFactionAI` skips units whose `acted` flag is set and
  set it on all enemies at the end of their phase, but only *player* units were being reset at the start of a
  turn — so from turn 2 on, every enemy was still flagged "acted" and got skipped. The phase runner now clears
  `acted` for its faction at the start of every enemy/ally phase. (The headless sim had masked this because it
  used its own loop that reset the flag.) Verified: enemies pre-flagged `acted=true` still attack after the fix.

## [M8 / v1.5] Multi-agent pass: cleanup, stat screens, test loadouts
Built with a 4-agent workflow (cleanup audit, item design, stat-screen build, improvement review);
all outputs integrated in a single verified pass (55/55 patches applied, zero failures).

- **Codebase cleanup (28 patches):** removed dead functions (`expToNext`, `isFlier`, `weaponRange`,
  `placedRosterEnemy`), no-op statements (`SFX.move&&0`, empty if-blocks), unused variables
  (`wasLast`, `wasScene`, `lv`), the vestigial gold system (`G.gold`, `#goldText`), orphaned/duplicate
  CSS rules, a dead `itemmenu` state test, and an always-true defend guard. Fixed two real bugs:
  the start-of-turn heal floating text could show the wrong amount (computed after clamping), and the
  escort defeat message typo ("you sworn"→"you swore"). `showLevelUp` now takes a named `gains` param.
- **Stat screens (new):**
  - **Class Compendium** — browse all 28 classes grouped base/promoted: movement type, MOV/CON, weapon
    types, class crit/avoid, promotion path (with the exact seal named, clickable to jump), traits, and
    base stats vs caps vs growth% as bars. Open via title menu, world-map "Class Guide", the battle menu,
    or the **C** key.
  - **Unit Report** — FE-style inspect for any live unit: HP/EXP bars, all 8 stats as bars against class
    caps (capped = gold, boosted = green), growths (player units only), weapon ranks with progress-to-next,
    inventory with EQ badge/uses, skills with descriptions, and support ranks with points-to-next.
    Open via **C** on a unit, right-click a roster chip, "Info" in the unit list, "Inspect" in army manage,
    or the battle menu.
- **Test loadouts:** all 12 characters now start with 4-5 items covering every system — weapon tiers,
  killer/crit weapons (Killing Edge, Wo Dao, Killer Lance/Bow), effective weapons (Armorslayer, Hammer,
  Horseslayer), 1-2 range (javelins/hand axe/tomes), 2-3 range (Longbow), full staff kit (Heal/Mend/Physic),
  consumables, and Finn's door key. The convoy starts with a 20-item kit: all 6 promotion seals, 6 stat
  boosters, elixirs, silver weapons, and light tomes for post-promotion testing.

## [M9 / v1.6] Music, in-combat stats, and class character models
Built by a 5-agent workflow (music engine, combat-stats/log, and three model artists each owning a
disjoint class set via the new `BA.classBody` per-class renderer dispatcher); integrated in one pass
(30 patches, 1 predicted anchor collision merged by hand).

- **Procedural chiptune music (Web Audio, no assets):** five looping tracks — noble *title*, marching
  *map/player phase*, tense *enemy phase*, driving *battle arena*, ominous *boss* — played by a
  lookahead scheduler over square lead / triangle bass / noise percussion, with ~0.25s crossfades,
  autoplay-policy unlock on first input, a Music toggle in Options, and on/volume persisted to
  localStorage. Victory/defeat hand off to the existing jingles.
- **Combat stats during combat (GBA-style):** the battle-scene panels grew to show **HIT / DMG / CRT**
  for both sides (×2 marker when doubling; em-dashes when a side can't counter), captured *before*
  combat resolution so broken weapons can't skew them. Player is always the left panel.
- **Combat log:** every battle records turn/phase headers, each strike ("Kael → Archer: CRIT 36!"),
  procs, deaths ("☠ Archer falls"), heals, EXP and level-ups, and reinforcement notices into a
  scrollable, color-coded panel — toggled with **L** or the new **≡ Log** HUD button.
- **Class character models:** all 11 battle-scene archetypes replaced with distinct designs —
  regal coat-and-cuirass **Lord**; bandolier duelist **Mercenary**; hulking bare-armed **Axe** fighter;
  masked **Thief/Assassin**; plumed kettle-helm **Pikeman** with mail rows; wall-like **General**
  (tower pauldrons, glowing visor slit); hooded **Ranger** with fletched quiver that leans into the
  draw; rune-robed **Mage** with orbiting cast-motes; stole-and-cowl **Cleric**; and proper stirrup-
  posture **Cavalry/Pegasus riders** with streaming scarves. Each reacts to swing/lunge/draw/cast
  animation state and keeps the shared shoulder/head anchors for the weapon arm and hit flash.

## [v1.6.1] Animation crash-proofing
Investigated a report of walking + battle animations disappearing. Frame-by-frame verification of
v1.6 (manual-pumped frames in a headless window) shows both work end-to-end: walk tween interpolates
smoothly, and a full arena combat runs fade-in → windup → dash → impact (with live HIT/DMG/CRT
panels) → HP drain → outro → clean handback with zero errors. However, the audit exposed a real
fragility matching the reported symptom class:
- **The rAF loop had no error guard** — a single exception in any draw call would permanently kill
  the animation chain (walking AND battle scenes) while the DOM UI kept half-working. `loop()` now
  catches per-frame errors, logs them, and keeps the chain alive.
- **`BA.play` is now wrapped** in performAttack — if the battle scene ever throws mid-fight, combat
  recovers (restores HUD, clears hit-stop, toasts a notice) instead of soft-locking on 'animating'.

## [M10 / v1.7] Ten-agent pass: range pins, anim speed, full supports, Hard mode+, docs
A 10-agent workflow (4 adversarial animation bug-hunters, 4 builders, balance analyst, docs auditor);
5 builders/auditors completed before the session's subagent quota capped the hunters/balance (they can
be resumed later — the workflow caches completed agents). Integrated in one pass (37 patches) plus one
integration-ordering fix (support-convo assignments must run after the base literal) and hand-built polish.

- **Enemy range pins (chosen update):** click any enemy (or press T) to pin its attack range in
  **orange**, persisting across your whole turn while you plan; pin multiple enemies; a marker shows on
  pinned foes; pins auto-clear when they die. The red danger-zone toggle (D) remains independent.
- **Animation speed & persistent options:** Options now has **Anim speed 1×/1.5×/2×**; holding
  **Shift/Space fast-forwards the enemy phase (3×)** without affecting your own inputs; sound/battle-anims/
  danger-zone/speed settings persist across sessions.
- **Support conversations complete:** all **8 pairs** now have full C/B/A arcs (15 new scenes from the
  content agent + the Lyra–Kael lord arc hand-written), each with a build from banter to bond.
- **Hard mode identity:** the dormant **Pursuit** skill now works (follow-up on any AS lead); on Hard,
  bosses gain a signature skill (Garrok gets Wrath, etc.), promoted generics get sharper, and every
  chapter fields **one extra aggressive enemy** via a data-driven `hardExtra` hook.
- **In-combat stat cues (hand-built):** the arena HIT/DMG/CRT strips now show the **weapon-triangle
  arrow** (▲ advantage / ▼ disadvantage), tint **DMG orange when effective**, **CRT red when ≥20%**, and
  **HIT green when ≥90%** — the whole matchup readable at a glance mid-fight.
- **Docs refreshed** (11 surgical patches): README controls/features, DESIGN architecture (battle scene,
  MUSIC, classBody dispatcher, stat screens), skills list, milestone history, and a new
  "Implemented vs intentionally deferred" section.

## [M11 / v1.8] Crit cinematics, branching promotions, promotion cutscene
One workflow agent delivered the killer-weapon loadouts (12 verified patches); the other three agents hit
the monthly subagent spend cap, so the crit animation, branching promotions, and promotion cutscene were
built inline by the lead.

- **Crit cinematics:** before a critical strike, a **cut-in interstitial** — letterbox bars snap in, a
  diagonal accent-colored banner sweeps across with the attacker's name and light streaks, and a rising
  stinger plays; the impact adds a **double shockwave ring**, **8 flying shards**, and **ground cracks**
  under the target, with longer hit-stop (190ms) and heavier shake — on top of the existing zoom, flash,
  vignette, and speed-lines.
- **Killer weapons everywhere (crit testing):** Lyra +Killing Edge, Garrett +Killer Lance, Roland/Bryce/
  Dorian/Finn killer swaps, and a new **Rune Edge** killer tome (30 crit anima) for Tobin; Garrok, Rhys,
  and CH1's aggressive brigands carry Killer Axes so incoming crits happen too.
- **Branching promotions:** every starter class now has **3 promotion options** (Mercenary→Hero/Vanguard/
  Duelist, Cavalier→Paladin/Dragoon/Bow Knight, Pegasus→Falcon/Storm Knight/Valkyrie, …), backed by
  **8 new classes**: Vanguard, Duelist, Marshal, Dragoon, Bow Knight, Valkyrie, Rogue, Storm Knight —
  full caps/bases, Compendium lists all three paths as clickable links (36 classes total).
- **Promotion cutscene:** using a seal opens a **"Choose a destiny"** picker (3 paths with movement,
  weapons, blurb, and key caps; Cancel keeps the seal), then plays a class-change scene — the unit's
  actual fighter model stands under a descending light pillar, whites out, and **emerges as the new
  class's model** with gold rings, rising sparkles, and the class-name reveal, before the stat-gains panel.
- **Seals in hand for testing:** every eligible character now carries their matching seal (Hero Crest,
  Knight Crest, Orion Bolt, Guiding Ring, Elysian Whip, Master Seal) alongside the convoy's full set.

## [M12 / v1.8.1] Three-artist pass: map tokens, Lord showcase, Knight showcase
Three dedicated art agents, each delivering a pure override (no anchor conflicts).

- **Chibi-bust map tokens** (replaces the flat capsules): faction-armored shoulders and pauldrons with
  highlight gradients + accent rivets, cloth tabard with accent stripe, layered two-tone hair with jaw
  shading, class headgear hints (helm for armored/mounted, hood for casters, wing clip for fliers), a
  **weapon silhouette over the shoulder** matching the equipped weapon (sword/lance/axe/bow/tome/staff),
  rim-light on the facing side, and a subtle idle bob for units that can still act. Every functional layer
  preserved: HP bars, colorblind shape badges, boss star, pin markers, recruit heart, mount/wing hints,
  acted-graying, death fade.
- **Lyra, showcase model** ("the last ember of Valden"): double-layer royal cape (badge outer, satin
  accent lining with sheen) that ripples in the wind and streams back on a charge; cinched engraved
  cuirass with gold filigree and a white swan crest; layered flowing hair + glinting circlet; a golden
  **ember aura with rising sparks that intensifies below half HP** and burns brighter as Great Lord.
- **The Knights, showcase model** ("the anvil"): widened segmented plate wall with rivets and a tabard
  panel, twin tower pauldrons (spiked for Generals, crowned + a **waving battle-standard pennant** for
  Marshals), piston legs with wedge sabatons that slam wide on the charge, and a **pulsing faction-colored
  visor slit** that flares in combat.
- **Bug fixed (latent since M7.1, exposed by the new art):** `enemyPort()` used a signed shift (`h>>4`)
  for the hair index — negative for large uid hashes → `hair: undefined`. The old flat token silently
  swallowed the invalid color; the new one shaded it and threw. Fixed with an unsigned shift + defensive
  fallbacks. (The error-guarded loop from v1.6.1 contained the damage exactly as designed.)

## [M13 / v1.9] Unique class icons + promotion-scene bug fixes
Both agents for this pass died at the monthly subagent spend cap mid-run, so the work was
built inline by the lead.

- **Every class icon is now unique on the map.** A per-archetype identity layer on the chibi-bust
  tokens: Lord (gold circlet with glinting gem, swan collar, cape edge) · Mercenary-line (accent
  headband with flying tails, bandolier) · Soldier (kettle helm + plume) · Fighter (wild spiked hair,
  bare shoulder, fur trim) · Archer (raised hood, quiver fletchings) · Mage (wide-brim pointed hat —
  dark hood for dark mages — with an orbiting rune) · Cleric (cowl + gold halo) · Cavalry (plumed
  half-helm, cheek guard, saddle blanket) · Knight (full-face helm with pulsing faction visor slit,
  oversized pauldrons, crest ridge — no visible face) · Pegasus (silver winged band, fluttering scarf)
  · Thief (face mask, spiky hair, dark scarf, dagger held low). All functional layers preserved.
- **Bug: world-map promotions played an invisible cutscene.** The `#world` DOM overlay covers the
  canvas, so promoting from Manage Army showed a frozen screen for ~3s. The chooser now hides the
  world screen during the cutscene and restores it before the stat-gains panel; in battle it also
  clears the HUD/roster/log/topbar so the scene plays clean.
- **Bug: input could leak over the cutscene** — pressing `C` (or other keys) during it opened modals
  on top. All keyboard input is now swallowed while the class-change scene plays.
- **Bug: anim-speed desync in the crit cut-in** — the banner's timer ran on real time while its
  motion was speed-divided, so at 2× it lingered into the strike. Its duration now scales too.

## [v1.9.1] Bug: equipped weapon was silently overridden on attack
Manually equipping a weapon in the Item menu worked, but starting an attack re-picked the
"best" weapon by raw might and equipped that instead — so a selected Killer Lance (Mt 10)
lost the tie to Steel Lance (Mt 10, earlier in inventory) and the crit build never fired.
Fix: the attack flow and the hover forecast now **respect the player's equipped weapon**
whenever it can reach the target, falling back to another usable weapon only when the
equipped one is out of range (e.g. a 1-range killer vs a target at range 2 still auto-uses
the Javelin). AI weapon optimization unchanged.

## [v1.9.2] Roster at Lv10 (promotion testing)
All 12 characters (and the talk-recruit spawns) now start at level 10 with stats scaled to
match, so every promotion seal is usable from the first turn of a new game.

### Fixes found via QA
- **Weapon-rank bug:** generic enemies/recruits given Steel/Killer/Silver weapons were never
  granted the rank to wield them, leaving many effectively unarmed. `makeUnit` now guarantees a
  unit can use every weapon it starts with. (Made enemies fight with intended kits → real difficulty.)
- AI "guard" units aggroed from too far; tightened their leash so chokepoints hold.
- Lord target-value lowered so enemies don't laser-focus Lyra unrealistically.
- Chapter 1 map redesigned for a guaranteed safe setup turn (no enemy reaches the deploy zone T1).
- Fixed authored unit placements that landed on walls / overlapped deploy tiles (CH4/5/6).
- Added an Armorslayer to the starting convoy so melee teams always have an anti-armor option.
