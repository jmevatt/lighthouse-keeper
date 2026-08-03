# Lighthouse Keeper — Design Document

*Draft v0.1 — August 2026*

A 2D mining/survival/base-defense game. You are a keeper stationed at a remote
lighthouse on an unknown cape. Beneath the basement lies a tunnel into a vast
underground world. Dig deep to feed the light; keep the light to survive the dark.

**Genre reference points:** Terraria (side-view digging/building), Core Keeper
(skill-by-use, food-as-buffs), Fallout Shelter (assignable settlers — borrowed
selectively).

**Engine:** Godot 4 (MIT-licensed). Side-view 2D for both surface and underground.

---

## 1. The Identity Mechanic

**Depth vs. duty.** The player is always being pulled in two directions: riches
are down, responsibility is up. An approaching ship can interrupt a mining run at
any moment, forcing a real decision — abandon the vein and sprint for the lamp
room, or keep digging and risk the wreck. Every other system exists to sharpen
this tension. If a proposed feature doesn't feed it, cut the feature.

## 2. Core Loop

```
DAY   — descend: mine, forage, fight, push one biome past comfort
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
  High fear: shrinking vision radius, aim sway, audio distortion, hallucinated
  enemies at the extreme. Fear acts as a *soft depth timer* — deeper biomes
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
| Deftness | Move speed, crafting speed, ranged aim |
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

## 8. Underground Biomes (sketch)

Increasing depth = ambient fear, darkness, enemy tier, and resource tier.

1. **Sea Caves** — damp stone, copper/tin, crabs and bats. Tutorial pressure.
2. **Root Hollows** — cave flora (oil source), spiders, first goblin scouts.
3. **Crystal Gallery** — light-refracting crystal (tier-3 material), fear spikes,
   enemies that are drawn to your lantern.
4. **The Drowned Dark** — flooded passages, high fear, pre-endgame materials.
5. **???** — endgame biome + whatever is at the bottom. (What is the tunnel *for*?
   Why is there a lighthouse *here*? Hold this mystery for the story hook.)

## 9. Vertical Slice — build this first

If this loop is fun at this size, the game works; everything else is content.
If it isn't, more biomes won't fix it.

**In:**
- One surface screen (lighthouse, shoreline, tunnel entrance), side-view
- One biome (Sea Caves), tile-based digging, one ore, one enemy type (bat or crab)
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

## 10. Open Questions

- Combat feel: Terraria-style loose platformer combat, or tighter/deliberate?
- Death penalty: drop resources at death site (Terraria), or softer?
- Story: what is at the bottom of the tunnel, and does the game "end"?
- Multiplayer: co-op keeper + digger role-split is tempting — decide *before*
  building netcode-hostile systems, even if the answer is "no for v1."
