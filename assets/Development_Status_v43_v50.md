# Development Status Update  v50.2
## Timeline Project – Since v43

This document describes the **current development status and architecture** of the timeline project, relative to the last formally documented version (**v43**).

It is **not a migration guide**, but a snapshot of how the system is now structured, why key decisions were made, and which earlier assumptions no longer apply.

---

## 1. Scope of This Document

- Describes **what the system looks like now**
- Explains **how responsibilities are divided**
- Highlights **conceptual changes since v43**
- Intended as background documentation for future work

---

## 2. High-Level Architecture (Current)

The system is now clearly split into three layers:

1. **Data preparation**
2. **Pure rendering**
3. **Editing & workflow tooling**

Each layer has a strict responsibility boundary.

---

## 3. Rendering Layer (`timeline.js`)

### Current Role
`timeline.js` is now a **pure renderer**.

It:
- Does **not load data**
- Does **not mutate data**
- Does **not contain editor logic**
- Renders only what is provided via global state

### Inputs
- `window.TS_DATA`
- Current zoom / pan transform
- Viewport dimensions

### Outputs
- SVG rendering only
- No side effects outside rendering

### Key Principle
> The renderer does not care *where* data comes from, only *how* to draw it.

---

## 4. Data Flow & Ownership

### Current Model
- All timeline data is prepared **before** rendering
- Data lives in:
  - `eventsDB.json` (persistent storage)
  - in-memory structures prepared in `index.html`

### Renderer Contract
- `TS_DATA` is treated as **read-only**
- Live updates use `window.updateTimeline()` to re-render safely

### Consequence
- Rendering is deterministic
- Editor bugs cannot corrupt renderer state

---

## 5. Editor & Workflow Layer (`editor.js`)

### Responsibilities
- Preview themes
- Draft events
- Commit workflows
- Import / export of JSON data

### Explicit Non-Responsibilities
- No SVG manipulation
- No rendering decisions
- No direct calls into timeline internals

### Communication Model
Editor → Data → Renderer  
(no direct coupling)

---

## 6. Prefocus System (Redesigned)

### Then (v43)
- DOM-measurement based
- Sensitive to jitter and mobile layout changes
- Feedback loops between layout and logic

### Now
- Fully **data-driven**
- Uses a single conceptual anchor (center band)
- Includes:
  - hysteresis
  - sticky tolerance
  - present-event suppression during idle

### Result
- Stable behavior on mobile
- No layout feedback loops
- Predictable focus selection

---

## 7. Layer & Z-Order Model

Rendering layers are now **explicit and fixed**:

1. Base cards (`g.cards`)
2. Global dim layer
3. Active card overlay (clone)
4. Axis (never dimmed)
5. Zoom bar and UI overlays

Cards are **never re-parented** during interaction.

Visual emphasis is handled by overlays only.

---

## 8. Active Card Handling

- Active card is **cloned** into a bright overlay layer
- Original card remains in its original position
- Prefocus logic applies consistently to both

This avoids:
- DOM jumps
- Broken transitions
- Mobile rendering glitches

---

## 9. Public Control API

A minimal external API is now exposed:

```js
window.TimelineAPI = {
  scaleBy(),
  translateBy(),
  animScaleBy(),
  animTranslateBy(),
  selectTheme(),
  getCenter()
}
```

Purpose:
- Scripted navigation
- Editor coordination
- Future story / guided modes

---

## 10. What Has Changed Conceptually Since v43

No longer assumed:
- Renderer loads data
- Editor influences rendering directly
- DOM geometry drives logic
- Cards move between layers

Now assumed:
- Data is authoritative
- Rendering is stateless per frame
- Visual emphasis ≠ structural change
- Editor is a data tool, not a renderer

---

## 11. Design Priorities Going Forward

- Deterministic rendering
- Mobile stability
- Clear responsibility boundaries
- Editor safety
- Long-term extensibility

---

## 12. Summary

Since v43, the project has shifted from an **experimentally evolving system** to a **deliberately structured architecture**.

The current state emphasizes:
- clarity over cleverness
- data flow over DOM tricks
- predictability over implicit behavior

This document should be read as the **new baseline description** of the system.
