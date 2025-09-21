# Self‑Learning Agent (Mobile, Single‑File HTML)

An interactive, self‑contained simulation of tiny agents that learn to reach a goal while avoiding obstacles. It runs entirely in the browser (desktop or mobile) with no build tools or dependencies.

> **Highlights**
> - Single HTML file, no setup
> - Tiny neural nets + genetic algorithm (GA)
> - Sensor rays for obstacle awareness
> - Map only resets on **win** or **manual Reset**
> - Mobile‑friendly: tap to move the goal, drag to pan the start

---

## Quick Start

1. Save the file as `index.html`.
2. Open it in any modern browser (Chrome, Edge, Firefox, Safari).
3. Press **▶ Run** to start the simulation.

> Tip: On mobile, add it to your home screen for a full‑screen feel.

---

## Controls & UI

Top bar controls:

- **▶ Run / ⏸ Pause** — Start or pause the simulation loop.
- **Step** — Advance ~8 ticks while paused (debug/inspection).
- **Reset** — Hard reset: new course, clear stats, re‑bootstrap population.
- **Speed** — Simulation speed multiplier.
- **Population** — Number of agents per generation. Starts at **1** (leftmost).
- **Mutation %** — Per‑weight mutation magnitude applied when evolving.
- **Sensors** — How many sensor rays per agent (odd numbers, fan spread).
- **Obstacles** — How many rectangular obstacles to place on the course.

Footer legend:

- **Green** dot/path — Best agent this frame.
- **Blue** dots/paths — Other agents.
- **Gold** circle — Goal.
- **Red** boxes — Obstacles.
- **Tap canvas** — Move the goal. **Drag** — Pan the start. (**Shift+Drag** also moves goal.)

---

## Core Mechanics

### World

- Fixed logical world size (scaled to canvas).
- **Start**: `(x≈10%W, y≈50%H)` by default.
- **Goal**: `(x≈90%W, y≈50%H)`, radius = 18px (scaled).
- **Obstacles**: Rects with a bias to sometimes block the direct line to goal. New obstacle layouts are generated on:
  - **Win**, or
  - **Manual Reset**, or
  - **Obstacle count** slider changes.

### Sensors

- Each agent casts **N** rays (configurable) in a ±~108° fan around heading.
- Each ray returns normalized distance to the closest hit (obstacle or wall) up to 120px range.
- Inputs to the network:
  1. N ray distances (0..1)
  2. **Δangle to goal** normalized to [-1, 1]
  3. **Normalized distance to goal** in [0, 1]
  4. **Normalized speed** in [0, 1]

### Tiny Neural Net

- 1 hidden layer (**12** units, `tanh`), small weight init.
- Outputs:
  - `steer` in [-1, 1] → clamped & scaled by `steerLimit`
  - `throttle` in [0, 1] → accelerates/brakes (bias to move forward)

### Movement & Culling

- Max speed ≈ 3 px/tick; steering capped each tick.
- **Two full rotations** (≈`4π` cumulative yaw) flags a **spin‑out** ⇒ death.
- **Stale detection**: if an agent hasn’t improved its **min distance to goal** by more than `0.5px` for `~120` ticks ⇒ culled.
- Map edges and obstacle collisions also kill the agent.
- **Win** when center is within `(goal radius + agent radius)`. Winning ends the round and increments `Wins`.

### Fitness & Evolution (GA)

- Fitness is **distance‑only** (survival is not rewarded):
  - Reached: `1000`
  - Spun‑out: `0`
  - Otherwise: `1 / (1 + minDistToGoal)`
- **Bootstrapping**:
  - First **two generations** are fully random; best from each is archived.
- **Parents** after bootstrap:
  - Two archived best brains (A, B) **plus** the current generation’s best (C).
- **Children**:
  - Mix two parents’ weights *per‑weight* (random α), then
  - Apply **minimum jitter** (3%) to guarantee **no exact copies**, then
  - Apply user‑controlled **mutation %**.
- **Course regeneration**:
  - Only when **won** (or **Reset**). This preserves learned behavior across generations until a win.

---

## Workflow You’ll See

1. **Gen 0** starts with a single random agent.
2. Agents quickly die while exploring; fitness comes from getting **closer** to the goal.
3. After a couple of random gens, GA begins mixing archived/ current bests.
4. The **best path** emerges; the best agent is highlighted in **green** (with sensor rays drawn).
5. When a best agent finally **reaches the goal**, the course changes and the system **re‑bootstraps** on the new map.

---

## Design Choices (Why it avoids “farming” exploits)

- **No survival reward**: standing still or orbiting provides no benefit.
- **Distance‑only fitness**: an agent that died **closer** to goal beats one that lived but went nowhere.
- **Spin‑out cull**: two full rotations ⇒ eliminated.
- **Stale cull**: no meaningful improvement ⇒ eliminated.
- **Never copy exact brain**: always inject noise to prevent clones and mode collapse.

---

## Customize

You can tweak constants near the top of the script:

- `TWO_TURNS` — rotation limit before culling
- `STALE_LIMIT`, `STALE_EPS` — stagnation detection
- `ROUND_TICKS` — per‑round tick cap before forced evolution
- `GEN_STALE_LIMIT` — early evolve if the whole gen stalls
- Neural net size: `createBrain(nIn, nH=12)`
- Sensor range/spread: `maxR = 120`, `spread = Math.PI * 1.2`

---

## Known Limitations / Ideas

- No explicit path memory or replay buffer (pure GA + tiny NN).
- Obstacles are axis‑aligned rectangles only.
- Could add multi‑objective rewards (e.g., path length, smoothness).
- Could archive hall‑of‑fame elites across courses (currently re‑bootstraps on each win).

---

## Troubleshooting

- **“Nothing happens”**: Ensure you pressed **▶ Run** (it autostarts after ~300ms unless you interacted early).
- **All agents spin in circles**: Increase **Sensors** and/or **Mutation %**, or reduce **SteerLimit** in code.
- **Too easy/too hard**: Adjust **Obstacles** count. Remember: the course only changes on **win** or **Reset**.
- **Mobile performance**: Lower **Population** and **Sensors**, reduce **Speed**.

---

## File Structure

It’s a single file app:

```
SelfLearning.html
```

All logic, styles, and rendering live inside `index.html`. No external assets required.

---

Coded and written using ChatGPT

