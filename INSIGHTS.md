# Insights — LILA Player Journey Tool

Three things learned about LILA BLACK using the visualization tool.

---

## Insight 1: Bots are doing almost all the killing

### What caught my eye
Switching to the Events view and enabling the kill layer, nearly every kill marker renders in bot-purple rather than human-blue. This is immediately striking.

### The numbers
Out of 2,418 total kill events across all maps and dates:
- **2,415 kills were by bots** (BotKill events)
- **3 kills were by human players** (Kill events)

That is a **99.9% bot kill rate**. Human players generated 63,526 events total (movement, loot, deaths) but registered almost zero kills.

### Why a level designer should care
This means the current data set reflects a game state where human players are not successfully engaging in combat — they are either playing passively, dying before they can kill, or the bot population is so aggressive it is shutting down PvP interactions entirely. If combat is supposed to be a core loop, this ratio signals something is breaking it: potentially bot density, spawning proximity, weapon availability, or map geometry creating choke points that bots exploit better than humans.

### Actionable items
- **Metric to track:** Human kill rate (human Kill events / total Kill events). Target should be much higher than 0.1%.
- **Action:** Cross-reference human kill locations with bot patrol paths to identify whether geometry is creating unfair bot advantages.
- **Action:** Review bot difficulty tuning — if bots are eliminating humans before they find each other, PvP zones become effectively dead space.

---

## Insight 2: AmbroseValley is doing nearly all the work — GrandRift is barely played

### What caught my eye
Switching between maps in the sidebar, the event density difference is enormous. AmbroseValley's heatmap is thick with activity across the canvas while GrandRift looks sparse even with opacity at maximum.

### The numbers
| Map | Total Events | Matches | % of all activity |
|---|---|---|---|
| AmbroseValley | 61,013 | most active | **68%** |
| Lockdown | 21,238 | mid | 24% |
| GrandRift | 6,853 | least active | **8%** |

GrandRift also has the worst map coverage: only **71% of its spatial grid cells** were visited, vs 77% for AmbroseValley. Players are not only spending less time there — they are also exploring less of it when they do play.

### Why a level designer should care
An 8% activity share means GrandRift is close to being ignored. In an extraction shooter, players self-select maps based on perceived reward density and risk. GrandRift likely has a loot or engagement problem: either rewards feel thin, the map is too open/exposed, or routing from spawn to loot zones is unintuitive. The low spatial coverage suggests players are sticking to a narrow path rather than exploring.

### Actionable items
- **Metric to track:** Map selection rate per session and average dwell time per map.
- **Action:** Use the loot heatmap on GrandRift to identify whether loot is concentrated in one corner — if it is, redistribute to pull players across more of the map.
- **Action:** Audit GrandRift's cover and routing: the 71% cell coverage means roughly 30% of the map is dead space from a player perspective. Flag those cells for geometry review.

---

## Insight 3: Loot is heavily concentrated in a few hotspots, creating predictable and exploitable rotations

### What caught my eye
On the Heatmap view with only the Loot layer enabled, bright density spikes appear in 2–3 tight regions per map rather than being spread across the playable space.

### The numbers
Dividing each map into a 10×10 grid, the top 3 loot cells account for a disproportionate share of all loot events:

**AmbroseValley (9,955 loot events):**
- Top 3 zones: 589 + 568 + 527 events = **1,684 events** → **17% of all loot in just 3% of the map**

**Lockdown (2,050 loot events):**
- Top 3 zones: 156 + 155 + 124 events = **435 events** → **21% of all loot in 3% of the map**

**GrandRift (880 loot events):**
- Top 3 zones: 67 + 60 + 46 events = **173 events** → **20% of all loot in 3% of the map**

Across all maps, roughly 1/5 of all loot activity funnels into 3% of the map area.

### Why a level designer should care
Predictable loot concentration does two things: it turns those zones into permanent contest points (which can be good for designed conflict) but it also means players who don't control those zones have no viable path to gearing up. In an extraction shooter, this creates a rich-get-richer dynamic where early arrivals at hotspots dominate the rest of the match. It also explains the kill heatmap — kill clusters and loot clusters overlap heavily, meaning most combat is happening over loot rather than at tactically interesting positions.

### Actionable items
- **Metric to track:** Loot pickup distribution entropy — a flat distribution = healthy; a spiked distribution = problematic concentration.
- **Action:** Introduce secondary loot scatter zones in under-visited areas of the map (cross-reference the low-density movement zones from Insight 2) to pull players off the main hotspot rotation.
- **Action:** Test whether reducing spawn rates in the top 3 hotspot cells by 30% and redistributing to fringe areas changes player routing without destroying the expected conflict points.
