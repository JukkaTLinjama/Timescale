# Timescale Logarithmic Timeline — Version 43

## 🔍 Summary

Version 43 establishes a cleaner **metadata-driven architecture** for the Timescale project.  
Theme colors, language labels, and real-time configuration now come directly from `eventsDB.json`, removing hard-coded dependencies.  
This version also lays the groundwork for the **inline JSON editor** and the upcoming **real-time sliding present theme**.

---

## 🧩 Architecture Overview

### 1. High-Level Structure

| Layer | File | Purpose |
|-------|------|----------|
| **Frontend Host** | `index.html` | Entry point, initializes timeline and UI elements |
| **Visualization Engine** | `timeline.js` | D3.js-based renderer for logarithmic timeline, zooming, and focus logic |
| **Styles** | `style.css` | Layout, color variables, adaptive design for mobile/desktop |
| **Event Data** | `eventsDB.json` | Hierarchical JSON with metadata, theme definitions, and events |
| **Documentation** | `README.md`, `readmev43.md` | Meta overview and detailed technical notes |

---

### 2. Runtime Data Flow

| Step | Source | Function / Object | Output / Role |
|------|---------|------------------|----------------|
| 1 | `eventsDB.json` | Metadata + Events | Provides global UI configuration and raw event data |
| 2 | `loadData()` | `timeline.js` | Fetches, parses, and flattens events by theme |
| 3 | `state` | Global simulation state | Stores current zoom, layout, and flattened event list |
| 4 | `computeDomainFromData()` | `timeline.js` | Computes min/max of `time_years` for log scaling |
| 5 | `drawAxis()` | D3 SVG | Draws logarithmic grid (10^n ticks) |
| 6 | `drawCards()` | D3 SVG | Groups events by theme; renders cards and labels |
| 7 | `applyZoom()` | Zoom controller | Updates scale and triggers redraws |
| 8 | `window.TimelineAPI` | Exposed API | Allows external control (e.g., animations, editor hooks) |

---

### 3. Internal Modules (timeline.js)

#### a) State Management
```js
const state = {
  width: 0, height: 0,
  minYears: .01, maxYears: 1e10,
  events: [],
  themes: [],
  themeColors: new Map(),
  yBase: null, y: null,
  activeTheme: null
};
```
This keeps layout, domain, and active data references in one shared structure.

#### b) Layout System
- `layout()` → updates SVG bounds, clipping paths, and theme containers  
- `layoutCenterOverlay()` → draws the dashed midline and label (“zoom <->”)

#### c) Data Pipeline (`loadData()`)
1. Fetches `eventsDB.json`  
2. Reads `metadata.ui.themeColors`, `metadata.ui.themeOrder`, and `metadata.locale_default`  
3. Flattens groups: `{ theme, events[] }`  
4. Creates derived `display_label` for multilingual support  
5. Computes domain and initializes color mapping

#### d) Rendering
- **Axis Layer:** decades (10^n) and minor gridlines  
- **Cards Layer:** one card per theme with background rectangle and title  
- **Events Layer:** per-card event groups, each with line + label  
- **Decollision Logic:** pixel-based vertical spacing for identical timestamps (render-time only)

#### e) Interaction Layer
| Interaction | Method | Notes |
|--------------|--------|-------|
| Scroll / Pinch | D3 zoom handler | Scales Y logarithmically |
| Swipe (center zone) | Custom pointer logic | Horizontal gestures mapped to zoom |
| Click | `showEventInfo()` | Opens animated info popover |
| Prefocus | `updatePrefocusNow()` | Highlights label closest to centerline |
| External Control | `window.TimelineAPI` | Enables scripted zoom or theme selection |

---

### 4. Metadata Structure (v1.7)

```json
{
  "metadata": {
    "version": "1.7",
    "author": "Jukka Linjama",
    "updated": "2025-10-08",
    "locale_default": "fi",
    "locales": ["fi", "en"],
    "ui": {
      "themeOrder": ["kosmos", "biologia", "ihmiskunta", "historia", "teknologia"],
      "themeColors": {
        "kosmos": "#6372b2",
        "biologia": "#70a8c6",
        "ihmiskunta": "#4b9fa8",
        "historia": "#368d60",
        "teknologia": "#5a9646"
      }
    },
    "realtime": {
      "enable_clock": true,
      "scales": ["second", "minute", "hour", "day", "week", "month", "year"]
    }
  },
  "events": [...]
}
```

---

### 5. Rendering Stack (simplified)

```text
<svg id="timeline">
 ├─ <g class="axis"> → logarithmic ticks and labels
 ├─ <g class="minor-grid"> → minor tick lines
 ├─ <g class="cards"> → theme cards
 │   ├─ <g class="card" theme="cosmos">
 │   │   ├─ <rect> background
 │   │   ├─ <text class="card-title">Cosmos</text>
 │   │   └─ <g class="events"> event lines and labels
 │   └─ ... other themes
 ├─ <g class="zoomBar"> → right-hand zoom track
 └─ <g class="active-layer"> → lifted active card overlay
```

---

### 6. Key Improvements in v43

| Area | Improvement |
|-------|--------------|
| **Metadata Integration** | Reads colors, order, and localization directly from JSON |
| **Code Simplification** | Unified flatten + i18n label logic |
| **Rendering Stability** | Prevents overlapping labels with pixel offsets |
| **Mobile Input Handling** | Smooth swipe-to-zoom and overscroll bounds |
| **Layer Management** | Consistent Z-order (axis → cards → active → overlay) |
| **Popover System** | Animated info box with ESC + click-to-close |

---

## 🔮 Planned Next Steps

### 1️⃣ Inline Editor Integration
- Directly edit `eventsDB.json` through pop-over fields.  
- Add new events, modify labels, or delete existing ones.  
- Visual auto-refresh without reload.  
- Optional export/import button for JSON.

### 2️⃣ Real-Time Theme (“Now” Layer)
- Introduce a continuous **sliding present band** that updates once per second.  
- Auto-generate ephemeral events at previous intervals (minute, hour, day, etc.).  
- The band fades older events while introducing new ones dynamically.

### 3️⃣ Data Tools and Localization
- Integrate a dedicated event editor and metadata validator.  
- Expand multilingual UI (Finnish/English).  
- Standardize event importance and certainty metrics.

---

## 🔗 Integration with Meta Repository

`readmev43.md` expands the architecture section of the main [README.md](./README.md).  
For a general overview and version history, see that file.

---

## © 2025 Jukka Linjama / Timescale Project
Licensed under [Creative Commons BY 4.0](https://creativecommons.org/licenses/by/4.0/).
