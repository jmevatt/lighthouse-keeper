# Lighthouse Keeper — Design Document

*Draft v0.4 — September 2026*

A 2D mining/survival/base-defense game. You are a keeper stationed at a remote
lighthouse on an unknown cape. Beneath the basement lies a tunnel into a vast
underground world. Dig deep to feed the light; keep the light to survive the dark.

**Genre reference points:** Terraria (digging/building depth), Core Keeper
(view/art direction, skill-by-use, food-as-buffs), Dreadmyst (iso pixel art,
tab-target combat), Fallout Shelter (assignable settlers — borrowed
selectively).

**Engine:** Godot 4 (MIT-licensed) — confirmed. Isometric-style top-down 2D:
square grid with iso-styled pixel-art tiles (the Core Keeper approach — the
iso look without diamond-grid pathfinding/sorting costs; revisit before tile
art starts, painful to swap later). Underground biomes are stacked levels
linked by stairs and shafts.

---

## 1. The Identity Mechanic

**Distance vs. duty.** The player is always being pulled away from the lamp:
riches are down the tunnel or out past the treeline; responsibility stays at
the lighthouse. Both axes share one risk currency — **time-to-return**. An
approaching ship can interrupt a mining run or a far expedition at any moment,
forcing a real decision — abandon the vein and sprint for the lamp room, or
keep going and risk the wreck. Every other system exists to sharpen this
tension. If a proposed feature doesn't feed it, cut the feature.

## 2. Core Loop

```
DAY   — venture: descend or range out; mine, forage, fight, push one biome past comfort
      — interrupted (sometimes): surface emergency → beam/signal minigame
      — return: haul resources up, craft, cook, upgrade, assign settlers
NIGHT — defend: raids from land/sea/below; the beam is a weapon
      — quiet nights: safe crafting time, repairs, planning
```

The day/night cycle is the game's only clock. No idle timers, no wait-to-build.

## 3. Light — the Unifying Resource

One resource touches every system: **oil** (later: refined fuels, crystal light).

| System | How light touches it |
|---|---|
| Lighthouse lamp | Burns oil constantly; a dark lighthouse = failed rescues, bolder night raids |
| Mining lantern | Same fuel; radius = safe zone underground |
| Fear | Rises in darkness, falls in light |
| Enemies | Most underground enemies avoid lit areas; some rare ones are drawn to light |
| Night defense | The beam can be aimed — blind pirate ships, burn/panic ground waves |
| Failure spiral | Low oil degrades *everything* at once — the survival pressure is one number |

Oil sources scale with progression: rendered from fish/creatures (early), pressed
from cave flora (mid), refined from deep deposits (late).

## 4. Survival Model

- **Fear is the primary meter.** Ambient fear per biome + darkness modifier.
  High fear: shrinking vision radius, cooldown jitter, target-lock slip,
  audio distortion — and at the extreme, hallucinated enemies that enter the
  tab-target rotation, baiting real cooldowns on ghosts. Fear acts as a *soft depth timer* — deeper biomes
  overwhelm you faster until gear/levels extend your stay. No hard walls.
- **Food is buffs, not starvation** (Core Keeper model). Well-fed grants bonuses
  (speed, fear resistance, mining power); unfed just means no bonuses. Cooking
  quality scales with the kitchen bench and the cook settler.
- **No thirst meter.** Cut for busywork. Water matters as fishing/environment instead.

## 5. Progression — Three Tracks, One Lock Each

**Rule: everything is gated exactly once.**
Lighthouse tiers gate *what you can craft*. Keeper skills gate *how well you
perform*. Materials gate *when*. Never double-gate.

### 5.1 Keeper Levels (skill-by-use)

Skills level by doing: Mining, Combat, Running, Signaling (beam work),
Fishing, Cooking, Crafting. Small trees — milestone perk choice roughly every
5–10 levels. Slice cap: level 25, one perk choice per skill.

Examples: Mining 25 — chance to not consume lantern oil. Signaling 15 — beam
sweeps 50% faster. Combat 20 — fear gain from taking hits halved.

### 5.2 Lighthouse Tiers (resource investment)

Each tier physically grows the tower: a new floor (bench slot), stronger beam,
and one new emergency capability. Tier-up materials always come from the next
biome down — the lighthouse is what pulls you deeper.

| Tier | Lamp | Unlocks (sketch) |
|---|---|---|
| 1 | Hand-cranked lamp | Workbench; basic horn signal |
| 2 | Oil lamp, fixed beam | Kitchen; aimable beam |
| 3 | Rotating Fresnel lens | Research bench; beam tracking (storm rescues) |
| 4 | Dual-wick + reflectors | Alchemy bench; colored signal flares |
| 5 | Crystal-cored lamp | Enchanting bench; beam as true night-defense weapon |

### 5.3 The Cape (surface growth)

Saved ships and lighthouse tiers wake the cape up: rescued sailors settle,
a dock gets built, supply ships visit on schedule, a hamlet forms behind the
lighthouse. Visible "look what I've built" payoff; also creates stakes — night
defense now protects people and buildings, not just yourself.

## 5.4 Character Creation — Attributes & Templates

Chosen at creation; orthogonal to skills. **Attributes are broad flat modifiers
(who you are); skills grow by use (what you've practiced).** One lock each —
never double-gate.

| Attribute | Modifies |
|---|---|
| Brawn | Melee damage, carry capacity, dig speed |
| Grit | Fear resistance, health, stamina |
| Deftness | Move speed, crafting speed, cooldown recovery |
| Insight | Research speed, alchemy/cooking yields, ore-spotting radius |
| Sea-sense | Fishing, beam-minigame forgiveness, weather prediction |

**Templates are presets, not classes** — a pre-filled attribute spread + one
starting perk + a starting tool/recipe feeding their station. Templates change
your start, never your ceiling; every keeper can eventually learn everything.
"Custom" allows free allocation.

| Template | Spread favors | Starting hook (early-game expression required) |
|---|---|---|
| Miner | Brawn | Better pick + smelting recipe (workbench) |
| Engineer | Deftness | Trap blueprint (night defense) |
| Harvester | Grit | Farming/kitchen starter (food buffs) |
| Forager | Insight | Cave-flora knowledge (oil economy) |
| Alchemist | Insight | Mortar + oil-pressing recipe (bridges to tier-4 bench) |
| Signaler | Sea-sense | Beam perk (lamp room / emergencies) |

### Identity & inclusivity (day-one architecture)

No creator choice gates any other (BG3 standard): body type, build, skin tone,
hair, face, scars, and voice set are all independent and none are labeled by
gender. **Pronouns are their own selection** — he/him, she/her, they/them, or
custom-entered — separate from body and voice. No gendered clothing or stats;
all outfits fit all bodies. Settlers reflect the same diversity; any future
relationship mechanics are never orientation-gated.

*Implementation rule:* ALL dialogue/UI text goes through templated strings with
a pronoun-set resource on the player (`{they} {were} out at the lamp room`) —
trivial in Godot as a dictionary + format function; miserable to retrofit.
Same day-one rule for accessibility basics: remappable keys, colorblind-safe
palettes, screen-shake toggle.

## 6. Settlers (Fallout Shelter, borrowed selectively)

Every rescue is a recruitment roll; every failed rescue is a person who never
arrived. Rescued sailors take **role slots**:

- Cook → kitchen: stronger food buffs
- Navigator → lamp room: easier beam minigame
- Merchant → dock: trade rotation, restocks each morning
- Carpenter → repairs overnight damage
- Specialists — rare survivors from hard rescues (storm-lost ships) with unique bonuses

**Deliberately NOT borrowed:** idle wait-timers (settlers buff active play,
never run clocks) and per-settler happiness micromanagement (settler morale is
just the fear system: a lit, defended cape = calm settlers).

## 7. Emergencies (day) and Raids (night)

### Emergencies — vary by severity, not just type
- Passing ship: one horn blast, small reward
- Fog approach: sustained signal, moderate
- Storm-lost ship: manually track the beam on the ship for a full minute — hard,
  best rewards, chance of specialist survivor
- Shipwreck (failed or scripted): survivors wash ashore → settlers

### Night raids — two fronts, asymmetric
- **Below/land:** goblin/creature waves from the tunnel mouth and the treeline;
  threaten the lighthouse itself. Tower-defense-lite: barricades, lit zones, traps.
- **Sea:** pirate ships; threaten dock and hamlet (where the settlers are).
  Beam can blind/burn; shore defenses craftable at higher tiers.

Quiet nights are deliberate — pacing valleys make raid nights land harder.

## 8. Biomes — Two Axes

The world is expansive both vertically and horizontally. Both axes obey the
same laws: farther = higher hazard tier, higher resource tier, longer
time-to-return; and every biome must feed the light economy.

### 8.1 Underground (vertical)

Increasing depth = ambient fear, darkness, enemy tier, and resource tier.

1. **Sea Caves** — damp stone, copper/tin, crabs and bats. Tutorial pressure.
2. **Root Hollows** — cave flora (oil source), spiders, first goblin scouts.
3. **Crystal Gallery** — light-refracting crystal (tier-3 material), fear spikes,
   enemies that are drawn to your lantern.
4. **The Drowned Dark** — flooded passages, high fear, pre-endgame materials.
5. **The Mirror Cape** — the tunnel was never a tunnel; it's a crossing. A
   dark parallel of the surface: inverted lighthouse, dark sea, your own
   shoreline made wrong — and the wrecks of every ship you failed to save.
   Entry is gated by the fear economy alone (ambient fear overwhelms until
   gear/skills/food extend your stay — §4's soft depth timer, no hard wall).
   Endgame: relight the mirror lamp.

### 8.2 Surface (horizontal)

Ranging out from the cape, roughly by hazard tier. Distance works like depth:
the farther out, the longer the sprint home when the horn sounds — and
emergencies never pause. The beam is visible from everywhere on the surface;
when it goes dark on the horizon, you know exactly what your absence costs.
Hazard gear (cold, heat) follows the single-gate rule — gated by materials
*or* bench tier, never both.

1. **Forest** — timber, game, herbs. First expansion; building materials for
   the cape and hamlet.
2. **Dark Forest** — the treeline, deep. The one place the boundary is thin
   *above* ground: ambient surface fear, night-tier resources by day, and the
   mustering ground for land raids. Surface bleed-through foreshadows the
   Mirror Cape.
3. **Snowfields** — the cold coast. Ice fishing, food preservation, travel
   slowed (time-to-return stakes sharpen). Clearest skies — best long-range
   beam visibility.
4. **Desert** — salt flats and glass sand. Lens-grade glass: tier-3+ lamps
   (Fresnel and up) want desert glass. Salt for preservation and alchemy.
5. **Firelands** — the volcanic far coast. Forge-tier smelting and late-game
   fuel refining — §3's refined fuels are processed here from deep deposits
   (cross-axis dependency: dig it below, refine it out here).

## 9. Vertical Slice — build this first

If this loop is fun at this size, the game works; everything else is content.
If it isn't, more biomes won't fix it.

**In:**
- One surface screen (lighthouse, shoreline, tunnel entrance), top-down iso-style
- One underground level (Sea Caves), tile-based digging, one ore, one enemy
  type (bat or crab)
- Tab-target combat core: target cycling, 2–3 hotbar abilities, cooldowns
- Day/night cycle with 2D lighting (dark caves, lantern radius, lamp beam)
- Oil as fuel for lamp + lantern; fear meter driven by darkness
- Workbench only; 3 lighthouse tiers (visual growth + beam improvement)
- Skill-by-use for Mining, Combat, Signaling (cap 25, one perk each)
- Character creation: template picker, pronoun selection, a few body/hair
  options — full wardrobe/slider creator is v2, but the decoupled identity
  data + templated-dialogue architecture is v1
- One emergency type (passing ship, horn signal) + one night raid type (land wave)
- No settlers, no dock, no food system yet

**Out (v2+):** settlers/cape growth, sea raids, remaining benches, biomes 2–5,
food buffs, specialists, story.

**Godot implementation notes:**
- `TileMapLayer` + autotiling for destructible terrain
- `PointLight2D` + occluders + `CanvasModulate` for day/night and cave darkness
- Skill system: dictionary of counters with threshold checks
- Lighthouse tiers: swapped scenes
- Dev on Windows-native Godot (WSL2 host — keep the project on the Windows
  filesystem for GPU/editor performance)

## 10. Decisions (resolved v0.2)

- **Combat: tab-target hotbar rotations (Dreadmyst model), deliberate by
  construction.** Ability rotations, cooldown management, positioning — no
  manual aiming. Few enemies on screen; encounters are puzzles of rotation
  and placement. Fear taxes the combat verbs directly: vision shrink,
  cooldown jitter, target-lock slip, hallucinated tab targets at the
  extreme (§4).
- **View & art: isometric-style top-down pixel art** (Core Keeper / Dreadmyst
  reference). Square grid, iso-styled tiles, stacked underground levels. The
  beam becomes a rotating light cone seen from above — the lamp-room minigame
  is aiming the sweep across a dark sea.
- **Death penalty: drop the haul, keep the gear, pay in time.** Carried
  resources drop at the death site (the corpse-run pulls you back down —
  feeds depth-vs-duty). Tools and gear are never lost. Respawn at the
  lighthouse with time elapsed: oil burned, duty missed. Death routes through
  the oil failure spiral rather than adding a second punishment economy.
- **Multiplayer: no.** Single-player, full stop. Architecture carries no
  netcode constraints.
- **World generation: seed-based procedural (Terraria/Valheim model), set
  v0.4.** Underground levels generate from a world seed — same seed, same
  world. Generators must guarantee invariants (sealed borders, valid spawn,
  spawn-reachable caverns) and those invariants are unit-tested per seed.
  Hand-authored content overlays procgen later (set-piece rooms, the
  lighthouse basement entrance), not the other way around.
- **Story: the mirror world.** Beneath the cape lies its dark parallel — the
  final biome is a mirrored surface world, and descending was always a
  crossing. Fear is bleed-through: high-fear hallucinations are glimpses of
  the other side, and deeper biomes sit closer to the boundary. The lamp
  keeps the boundary thick — a dark lighthouse lets things cross, which is
  why night raids embolden in darkness. Failed rescues wreck on the mirror
  shore. Endgame: reach the Mirror Cape and relight the inverted lamp.

## 11. Open Questions

- Ending structure: after the mirror lamp is relit, does the game roll
  credits, continue as sandbox, or both?
- Reveal pacing: when does the player learn fear is bleed-through (keeper
  logs, environmental tells, a mid-game event)?
- Surface map geometry: biome adjacency and compass layout, world size, and
  travel mechanics (on foot only, or later mounts/boats/waypoints?).
