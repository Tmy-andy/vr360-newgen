# 04 — TECHNICAL IMPLEMENTATION SPECIFICATION
## Du Lịch Bà Rịa — Bản Đồ 360° Tourism Platform

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** 01-SYSTEM-ARCHITECTURE · 02-DATABASE-API-SPEC · 03-UI-UX-SPEC · 05-DEMO-HTML-SPEC

---

## Mục Lục

1. [File Structure](#1-file-structure)
2. [Core Functions](#2-core-functions)
3. [Pannellum Integration](#3-pannellum-integration)
4. [Leaflet Minimap](#4-leaflet-minimap)
5. [Lucide Icons Integration](#5-lucide-icons-integration)
6. [Smart UI System](#6-smart-ui-system)
7. [Minimap Resize & Drag](#7-minimap-resize--drag)
8. [Mobile Bottom Sheet (Swipe)](#8-mobile-bottom-sheet-swipe)
9. [Multi-Select Category Filter](#9-multi-select-category-filter)
10. [Performance Optimization](#10-performance-optimization)
11. [Sequence Diagrams](#11-sequence-diagrams)

---

## 1. File Structure

### 1.1 Single-File Architecture

```
tourism-map-360.html          (~1235 lines)
├── <head>
│   ├── Meta tags (viewport, charset)
│   ├── CDN: Pannellum CSS + JS
│   ├── CDN: Leaflet CSS + JS
│   ├── CDN: Inter font (Google Fonts)
│   ├── CDN: Lucide Icons JS (UMD)
│   └── <style> (~480 lines)
│       ├── Reset & globals
│       ├── CSS variables (:root)
│       ├── Panorama styles
│       ├── Loader
│       ├── UI layer
│       ├── Smart UI hide
│       ├── Topbar
│       ├── Filter chips
│       ├── POI panel
│       ├── POI cards
│       ├── Detail panel
│       ├── Minimap
│       ├── Minimap toolbar
│       ├── Minimap resize handles
│       ├── Scene indicator
│       ├── Hotspot markers
│       ├── Category colors
│       ├── Panel drag (mobile)
│       └── @media (max-width: 860px)
│
├── <body>
│   ├── Loader (#loader)
│   ├── Panorama (#panorama)
│   └── UI (#ui)
│       ├── Topbar
│       ├── POI Panel (#poiPanel)
│       │   ├── Panel drag bar
│       │   ├── Header (title, close, search, filters)
│       │   └── POI list (#poiList)
│       ├── Detail Panel (#detailPanel)
│       │   ├── Panel drag bar
│       │   └── Detail scroll (#detailScroll)
│       ├── Minimap Wrap (#minimapWrap)
│       │   ├── Toolbar (move, expand, close)
│       │   ├── Resize handle NE (#minimapResize)
│       │   ├── Resize handle BR (#minimapResizeBR)
│       │   └── Map container (#minimap)
│       ├── Scene Indicator (#sceneIndicator)
│       └── UI Restore (#uiRestore)
│
└── <script> (~720 lines)
    ├── DATA (CATEGORIES, OVERVIEW, POIS)
    ├── STATE variables
    ├── init()
    ├── Panorama functions
    ├── Category functions
    ├── POI list functions
    ├── POI detail functions
    ├── Minimap functions
    ├── Scene indicator functions
    ├── Toggle functions
    ├── Frame drag
    ├── Map expand
    ├── Minimap resize
    ├── Smart UI
    ├── Mobile panel drag
    └── DOMContentLoaded bootstrap
```

### 1.2 CDN Dependencies

```html
<!-- Pannellum 2.5.6 -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/pannellum@2.5.6/build/pannellum.css">
<script src="https://cdn.jsdelivr.net/npm/pannellum@2.5.6/build/pannellum.js"></script>

<!-- Leaflet 1.9.4 -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<!-- Inter Font -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">

<!-- Lucide Icons 0.468.0 -->
<script src="https://unpkg.com/lucide@0.468.0/dist/umd/lucide.js"></script>
```

---

## 2. Core Functions

### 2.1 Function Map

```
┌──────────────────────────────────────────────────┐
│                 INITIALIZATION                    │
│                                                   │
│  DOMContentLoaded                                 │
│  ├── lucide.createIcons()                        │
│  ├── init()                                      │
│  │   ├── renderCategories()                      │
│  │   ├── renderPOIList()                         │
│  │   ├── loadOverview()                          │
│  │   └── initMinimap()                           │
│  ├── bindSmartUI()                               │
│  ├── bindMinimapResize()                         │
│  ├── bindFrameDrag()                             │
│  ├── bindMobilePanelDrag(poiPanel)               │
│  └── bindMobilePanelDrag(detailPanel)            │
├──────────────────────────────────────────────────┤
│                 PANORAMA                          │
│                                                   │
│  loadOverview()    → Create overview scene        │
│  loadPOIScene(poi) → Create POI-specific scene    │
├──────────────────────────────────────────────────┤
│                 CATEGORIES                        │
│                                                   │
│  renderCategories()  → Build filter chip HTML     │
│  toggleCat(catId)    → Toggle cat in Set          │
│  getFilteredPOIs()   → Filter POIS by activeCats  │
├──────────────────────────────────────────────────┤
│                 POI LIST                          │
│                                                   │
│  renderPOIList()   → Render filtered cards        │
│  filterPOIs()      → Trigger re-render (search)   │
├──────────────────────────────────────────────────┤
│                 POI DETAIL                        │
│                                                   │
│  openPOI(id)       → Load scene + show detail     │
│  backToOverview()  → Reset to overview mode       │
├──────────────────────────────────────────────────┤
│                 MINIMAP                           │
│                                                   │
│  initMinimap()       → Leaflet map + markers      │
│  updateMinimap(poi)  → Zoom + highlight marker    │
├──────────────────────────────────────────────────┤
│                 UI CONTROLS                       │
│                                                   │
│  updateSceneIndicator(poi) → Update label/dot     │
│  togglePoiPanel()          → Show/hide POI panel  │
│  toggleMinimap()           → Show/hide minimap    │
│  toggleMapExpand()         → Fullscreen minimap   │
├──────────────────────────────────────────────────┤
│                 INTERACTION                       │
│                                                   │
│  bindSmartUI()          → Auto-hide on 360 drag   │
│  fadeOutUI() / fadeInUI()                         │
│  resetIdleTimer() / showAllUI()                   │
│  bindMinimapResize()    → NE + BR resize handles  │
│  bindFrameDrag()        → Drag minimap position   │
│  bindMobilePanelDrag()  → Bottom sheet swipe      │
└──────────────────────────────────────────────────┘
```

### 2.2 State Variables

```javascript
// Pannellum & Leaflet instances
let viewer = null;        // pannellum.viewer instance
let map = null;           // L.map instance

// Application state
let activeCats = new Set();  // Selected category IDs
let activePOI = null;        // Current POI object or null
let viewMode = "overview";   // "overview" | "poi"
let poiPanelOpen = true;     // POI list panel visibility
let minimapVisible = true;   // Minimap visibility
let mapExpanded = false;     // Minimap fullscreen mode
let panelExpanded = false;   // Mobile bottom sheet expanded

// Leaflet marker references
let markers = [];  // [{poi: POI, marker: L.circleMarker}]

// Smart UI state
let uiHidden = false;
let idleTimer = null;
let isInteracting = false;
```

---

## 3. Pannellum Integration

### 3.1 Overview Mode

```javascript
function loadOverview() {
  viewMode = "overview";
  activePOI = null;
  if (viewer) { viewer.destroy(); viewer = null; }

  // Build hotspots from filtered POIs
  const hs = getFilteredPOIs().map(p => ({
    pitch: p.pitch,
    yaw: p.yaw,
    type: "custom",
    createTooltipFunc: (el, a) => {
      el.className = "marker";
      el.innerHTML = `
        <div class="marker-pin cat-${a.cat}">${a.emoji}</div>
        <div class="marker-label">${a.name}</div>`;
      el.onclick = e => { e.stopPropagation(); openPOI(a.id); };
    },
    createTooltipArgs: p
  }));

  viewer = pannellum.viewer("panorama", {
    type: "equirectangular",
    panorama: OVERVIEW.panorama,
    autoLoad: true,
    showControls: false,
    showFullscreenCtrl: false,
    compass: false,
    friction: 0.15,
    yaw: OVERVIEW.yaw,           // 80
    pitch: OVERVIEW.pitch,       // 10
    hfov: OVERVIEW.hfov,         // 120
    minHfov: 60,
    maxHfov: 130,
    autoRotate: OVERVIEW.autoRotate, // 0.2
    hotSpots: hs,
    sceneFadeDuration: 0
  });

  // Loader dismiss on first load
  viewer.on("load", () => {
    const l = document.getElementById("loader");
    if (!l.classList.contains("done"))
      setTimeout(() => l.classList.add("done"), 400);
  });
}
```

### 3.2 POI Mode

```javascript
function loadPOIScene(poi) {
  viewMode = "poi";
  activePOI = poi;
  if (viewer) { viewer.destroy(); viewer = null; }

  viewer = pannellum.viewer("panorama", {
    type: "equirectangular",
    panorama: poi.panorama,
    autoLoad: true,
    showControls: false,
    compass: false,
    friction: 0.15,
    yaw: poi.pYaw || 0,
    pitch: poi.pPitch || 0,
    hfov: poi.pHfov || 100,
    minHfov: 50,
    maxHfov: 120,
    autoRotate: 0.4,
    hotSpots: [{
      pitch: 0,
      yaw: poi.pYaw + 180,    // Opposite direction
      type: "custom",
      createTooltipFunc: (el) => {
        el.className = "marker";
        el.innerHTML = `
          <div class="marker-pin cat-nature"
               style="background:rgba(255,255,255,.15);backdrop-filter:blur(8px)">
            ↩
          </div>
          <div class="marker-label">← Quay lại bản đồ</div>`;
        el.onclick = e => { e.stopPropagation(); loadOverview(); };
      },
      createTooltipArgs: {}
    }],
    sceneFadeDuration: 0
  });
}
```

### 3.3 Pannellum CSS Overrides

```css
/* Hide all default Pannellum controls */
.pnlm-controls-container,
.pnlm-about-msg,
.pnlm-load-button,
.pnlm-orientation-button,
.pnlm-compass           { display: none !important }

/* Custom hotspot styling */
.pnlm-hotspot-base      { background: none !important; width: auto !important; height: auto !important }
.pnlm-sprite             { display: none !important }
.pnlm-tooltip            { background: none !important; border: none !important; padding: 0 !important; box-shadow: none !important }
.pnlm-tooltip span       { display: none !important }
.pnlm-pointer            { display: none !important }
```

---

## 4. Leaflet Minimap

### 4.1 Initialization

```javascript
function initMinimap() {
  map = L.map("minimap", {
    zoomControl: false,
    attributionControl: false,
    doubleClickZoom: false
  }).setView([10.40, 107.15], 9);   // Center of BR-VT province

  L.tileLayer(
    "https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png",
    { maxZoom: 18, subdomains: 'abcd' }
  ).addTo(map);

  // Add circle markers for each POI
  POIS.forEach(p => {
    const cat = CATEGORIES.find(c => c.id === p.cat);
    const m = L.circleMarker([p.lat, p.lng], {
      radius: 6,
      fillColor: cat?.color || "#22D3A7",
      color: "#fff",
      weight: 1.5,
      fillOpacity: 1
    }).addTo(map);
    m.bindTooltip(p.name, { direction: 'top', className: 'map-tip' });
    m.on("click", () => openPOI(p.id));
    markers.push({ poi: p, marker: m });
  });

  // Double-click to expand/collapse
  document.getElementById("minimapWrap").addEventListener("dblclick", e => {
    if (e.target.closest(".minimap-toolbar") ||
        e.target.closest(".minimap-resize") ||
        e.target.closest(".minimap-resize-br")) return;
    toggleMapExpand();
  });
}
```

### 4.2 Map Update

```javascript
function updateMinimap(poi) {
  if (!map) return;
  if (poi) {
    map.setView([poi.lat, poi.lng], 13, { animate: true });
    markers.forEach(m => {
      m.marker.setStyle({
        radius: m.poi.id === poi.id ? 8 : 4,
        weight: m.poi.id === poi.id ? 2 : 1,
        fillOpacity: m.poi.id === poi.id ? 1 : 0.5
      });
    });
  } else {
    map.setView([10.40, 107.15], 9, { animate: true });
    markers.forEach(m => m.marker.setStyle({
      radius: 6, weight: 1.5, fillOpacity: 1
    }));
  }
}
```

### 4.3 Dark Mode CSS Filter

```css
#minimap {
  width: 100%;
  height: 100%;
  filter: brightness(.78) contrast(1.15) saturate(.85);
}
```

---

## 5. Lucide Icons Integration

### 5.1 Setup

```html
<script src="https://unpkg.com/lucide@0.468.0/dist/umd/lucide.js"></script>
```

### 5.2 Global CSS

```css
[data-lucide] {
  width: 14px;
  height: 14px;
  stroke: currentColor;
  stroke-width: 1.8;
  fill: none;
  display: inline-block;
  vertical-align: middle;
}
```

### 5.3 Usage Pattern

**Static (HTML):**
```html
<i data-lucide="map"></i>
```

**Dynamic (JS):**
```javascript
// After setting innerHTML with data-lucide attributes
element.innerHTML = `<i data-lucide="chevron-right"></i>`;
lucide.createIcons({ root: element });
```

### 5.4 All Dynamic Render Points

| Function | Icons Rendered | Root |
|----------|---------------|------|
| Page load | All static icons | `document` |
| `renderCategories()` | `check`, category icons | `#poiFilters` |
| `renderPOIList()` | `chevron-right` | `#poiList` |
| `openPOI()` | `chevron-left`, `map-pin`, `clock`, `navigation` | `#detailScroll` |
| `toggleMapExpand()` | `minimize-2` / `maximize-2` | `#btnMapExpand` |

### 5.5 Category Icon Mapping

```javascript
const CATEGORIES = [
  { id: "all",     icon: "globe" },
  { id: "beach",   icon: "waves" },
  { id: "temple",  icon: "church" },
  { id: "food",    icon: "utensils" },
  { id: "nature",  icon: "trees" },
  { id: "history", icon: "landmark" },
  { id: "resort",  icon: "hotel" },
];
```

---

## 6. Smart UI System

### 6.1 Mechanism

```javascript
function bindSmartUI() {
  const el = document.getElementById("panorama");

  // Start interaction → hide UI
  el.addEventListener("mousedown", () => {
    isInteracting = true;
    clearTimeout(idleTimer);
    fadeOutUI();
  });
  el.addEventListener("touchstart", () => {
    isInteracting = true;
    clearTimeout(idleTimer);
    fadeOutUI();
  }, { passive: true });

  // End interaction → start idle timer
  el.addEventListener("mouseup", () => {
    isInteracting = false;
    resetIdleTimer();
  });
  el.addEventListener("touchend", () => {
    isInteracting = false;
    resetIdleTimer();
  });
}

function fadeOutUI() {
  if (uiHidden) return;
  uiHidden = true;
  document.getElementById("ui").classList.add("hidden");
}

function fadeInUI() {
  if (!uiHidden) return;
  uiHidden = false;
  document.getElementById("ui").classList.remove("hidden");
}

function resetIdleTimer() {
  clearTimeout(idleTimer);
  idleTimer = setTimeout(() => {
    if (!isInteracting) fadeInUI();
  }, 4000);   // 4 second idle delay
}
```

### 6.2 CSS Hide Effects

```css
.ui.hidden .topbar          { opacity:0; pointer-events:none; transform:translateY(-20px) }
.ui.hidden .poi-panel,
.ui.hidden .detail-panel    { transform:translateX(calc(100% + 20px)) }  /* Desktop */
.ui.hidden .minimap-wrap    { opacity:0; pointer-events:none; transform:translateY(8px) }
.ui.hidden .scene-indicator { opacity:0; pointer-events:none; transform:translateY(8px) }

/* Mobile: panels slide down instead of right */
@media(max-width:860px) {
  .ui.hidden .poi-panel,
  .ui.hidden .detail-panel  { transform:translateY(100%) }
}

/* Restore button — only visible when UI hidden */
.ui-restore                 { opacity:0; pointer-events:none }
.ui.hidden .ui-restore      { opacity:1; pointer-events:auto; transform:none }
```

---

## 7. Minimap Resize & Drag

### 7.1 Resize Handles

Hai resize handles: **NE** (top-right, ne-resize) và **BR** (bottom-right, se-resize).

```javascript
function bindMinimapResize() {
  const wrap = document.getElementById("minimapWrap");
  const handleNE = document.getElementById("minimapResize");
  const handleBR = document.getElementById("minimapResizeBR");

  function setupHandle(handle, corner) {
    // corner = "ne" | "br"
    let dragging, startX, startY, startW, startH, startBottom;

    function onStart(e) {
      e.preventDefault(); e.stopPropagation();
      dragging = true;
      startX = ev.clientX; startY = ev.clientY;
      startW = wrap.offsetWidth; startH = wrap.offsetHeight;
      startBottom = parseInt(wrap.style.bottom) || 14;
      wrap.classList.add("resizing");
      // Add document listeners...
    }

    function onMove(e) {
      const dx = ev.clientX - startX;
      const maxW = window.innerWidth - rect.left - 8;

      if (corner === "ne") {
        // Drag UP = taller (startY - currentY)
        const dy = startY - ev.clientY;
        const maxH = rect.bottom - 8;
        wrap.style.width = clamp(startW + dx, 140, maxW) + "px";
        wrap.style.height = clamp(startH + dy, 100, maxH) + "px";
      } else {
        // Drag DOWN = taller (currentY - startY)
        const dy = ev.clientY - startY;
        const maxH = window.innerHeight - rect.top - 8;
        wrap.style.width = clamp(startW + dx, 140, maxW) + "px";
        wrap.style.height = clamp(startH + dy, 100, maxH) + "px";
        wrap.style.bottom = Math.max(0, startBottom - dy) + "px";
      }
      map.invalidateSize();
    }
  }

  setupHandle(handleNE, "ne");
  setupHandle(handleBR, "br");
}
```

### 7.2 Constraints

| Constraint | Value |
|-----------|-------|
| Min width | 140px |
| Min height | 100px |
| Max width | `calc(100vw - 28px)` |
| Max height | `calc(100vh - 28px)` |
| Edge padding | 8px from viewport edge |

### 7.3 Frame Drag (Move)

```javascript
function bindFrameDrag() {
  const wrap = document.getElementById("minimapWrap");
  const handle = document.getElementById("btnMapMove");

  // mousedown/touchstart on handle → track position
  // mousemove/touchmove → update left/bottom
  // Constraints: 0 ≤ left ≤ (viewport - wrap.width)
  //              0 ≤ bottom ≤ (viewport - wrap.height)
  // Sets: style.left, style.bottom
  // Clears: style.right, style.top → "auto"
}
```

### 7.4 Expand/Collapse

```javascript
function toggleMapExpand() {
  mapExpanded = !mapExpanded;
  const wrap = document.getElementById("minimapWrap");

  // Reset position to default corner
  wrap.style.left = "14px";
  wrap.style.bottom = "14px";
  wrap.style.right = "auto";
  wrap.style.top = "auto";

  wrap.classList.toggle("expanded", mapExpanded);

  // Swap icon: maximize-2 ↔ minimize-2
  btn.innerHTML = mapExpanded
    ? '<i data-lucide="minimize-2"></i>'
    : '<i data-lucide="maximize-2"></i>';
  lucide.createIcons({ root: btn });

  if (!mapExpanded) {
    // Clear inline styles to return to CSS defaults
    wrap.style.left = "";
    wrap.style.bottom = "";
    wrap.style.right = "";
    wrap.style.top = "";
  }

  setTimeout(() => map.invalidateSize(), 400);
}
```

---

## 8. Mobile Bottom Sheet (Swipe)

### 8.1 Architecture

```
bindMobilePanelDrag(panelEl)
│
├── onDragStart(e)
│   ├── Check: window.innerWidth > 860 → return
│   ├── Detect source: drag bar vs panel body
│   ├── Get current translateY
│   ├── Remove .expanded, add .dragging
│   └── Attach document listeners
│
├── onDragMove(e)
│   ├── dragSource = "bar":
│   │   └── Update translateY directly (smooth drag)
│   ├── dragSource = "panel":
│   │   └── Threshold-based snap (delta < -15 → expand, else collapse)
│   └── Clamp: 0 ≤ translateY ≤ (panelHeight - 160)
│
├── onDragEnd()
│   ├── Determine direction: delta < -20 → expand, delta > 20 → collapse
│   ├── Animate to target with transition
│   ├── On transitionend: clean up styles, set .expanded class
│   └── Remove document listeners
│
└── cancelDrag()
    └── Reset state if interaction cancelled
```

### 8.2 Key Parameters

| Parameter | Value | Mô tả |
|-----------|-------|--------|
| Peek height | 160px | Phần hiển thị khi collapsed |
| Max height | 90vh | Chiều cao tối đa khi expanded |
| Drag threshold | 20px | Minimum drag distance to trigger state change |
| Panel threshold | 15px | Minimum drag for panel-body source |
| Snap animation | 0.35s | `cubic-bezier(.16, 1, .3, 1)` |
| Idle timeout | 400ms | Fallback for transitionend |

### 8.3 State Transitions

```
          ┌──────────────┐
          │   COLLAPSED   │  translateY(calc(100% - 160px))
          │  (peek 160px) │
          └───────┬───────┘
                  │
        Swipe up  │  delta < -20
        (drag bar)│
                  ▼
          ┌──────────────┐
          │   EXPANDED    │  translateY(0)
          │  (full panel) │  .expanded class
          └───────┬───────┘
                  │
       Swipe down │  delta > 20
        (drag bar │  or panel body)
         or scroll│
                  ▼
          ┌──────────────┐
          │   COLLAPSED   │
          │  (peek 160px) │
          └──────────────┘
```

### 8.4 Drag Bar Visual Feedback

```css
/* Normal state */
.panel-drag-bar {
  width: 36px;
  height: 4px;
  background: var(--text-mute);
  border-radius: 2px;
}

/* While dragging */
.poi-panel.dragging .panel-drag-bar,
.detail-panel.dragging .panel-drag-bar {
  width: 48px;                     /* Wider */
  background: var(--accent);       /* Teal color */
}
```

### 8.5 State Cleanup

Khi navigate (openPOI, backToOverview, togglePoiPanel):

```javascript
// Reset mobile bottom sheet state
panelEl.classList.remove("expanded", "dragging");
panelEl.style.transform = "";
panelExpanded = false;
```

---

## 9. Multi-Select Category Filter

### 9.1 Data Structure

```javascript
let activeCats = new Set();  // Set of category IDs
```

### 9.2 Toggle Logic

```javascript
function toggleCat(catId) {
  if (activeCats.has(catId)) {
    activeCats.delete(catId);   // Deselect
  } else {
    activeCats.add(catId);      // Select
  }

  // Update CSS classes
  document.querySelectorAll(".filter-chip").forEach(b => {
    b.classList.toggle("active", activeCats.has(b.dataset.cat));
  });

  renderPOIList();               // Re-render filtered cards
  if (viewMode === "overview")
    loadOverview();              // Re-create hotspots
}
```

### 9.3 Filter Function

```javascript
function getFilteredPOIs() {
  if (activeCats.size === 0) return POIS;  // No filter = show all
  return POIS.filter(p => activeCats.has(p.cat));
}
```

### 9.4 Chip HTML Template

```javascript
`<button class="filter-chip ${isActive ? 'active' : ''}"
         data-cat="${c.id}" onclick="toggleCat('${c.id}')">
  <span class="chip-check">
    <i data-lucide="check" class="chip-check-icon"></i>
  </span>
  <i data-lucide="${c.icon}" class="chip-icon"></i>
  <span>${c.label}</span>
  <span class="chip-count">${count}</span>
</button>`
```

---

## 10. Performance Optimization

### 10.1 Techniques Used

| Technique | Mô tả |
|-----------|--------|
| Single file | No HTTP requests for app code |
| CDN libraries | Cached across sites, edge delivery |
| No framework | Zero JS framework overhead |
| Lazy thumbnails | `loading="lazy"` on POI thumbnail images |
| Destroy & recreate | Pannellum viewer destroyed between scenes (memory) |
| CSS transitions | GPU-accelerated transforms (translateY, opacity) |
| Inline styles | No external CSS file requests |
| Passive listeners | `{ passive: true }` for touch events (scroll perf) |
| requestAnimationFrame | Used for smooth swipe animations |
| Map invalidateSize | Called after resize/expand (Leaflet reflow) |

### 10.2 Memory Management

```javascript
// Pannellum viewer cleanup before scene change
if (viewer) {
  viewer.destroy();
  viewer = null;
}
// Then create new viewer
viewer = pannellum.viewer("panorama", { ... });
```

### 10.3 Event Listener Cleanup

Bottom sheet swipe listeners are added/removed per drag gesture:

```javascript
// On drag start: add document listeners
document.addEventListener("mousemove", onDragMove, { passive: false });
document.addEventListener("mouseup", onDragEnd);

// On drag end: remove document listeners
document.removeEventListener("mousemove", onDragMove);
document.removeEventListener("mouseup", onDragEnd);
```

### 10.4 Duplicate Bind Prevention

```javascript
function bindMobilePanelDrag(panelEl) {
  if (!panelEl || panelEl._dragBound) return;
  panelEl._dragBound = true;  // Prevent double-binding
  // ...
}
```

---

## 11. Sequence Diagrams

### 11.1 Full Page Lifecycle

```
Browser              CDN              Pannellum         Leaflet
  │                    │                  │                │
  │── GET HTML ───────►│                  │                │
  │◄── tourism-map.html│                  │                │
  │                    │                  │                │
  │── Load CSS/JS ────►│                  │                │
  │◄── pannellum.js ───┤                  │                │
  │◄── leaflet.js ─────┤                  │                │
  │◄── lucide.js ──────┤                  │                │
  │◄── inter font ─────┤                  │                │
  │                    │                  │                │
  │── DOMContentLoaded ──────────────────►│                │
  │── lucide.createIcons() ──────────────►│                │
  │── init() ─────────►│                  │                │
  │   renderCategories() ───────────────►│                │
  │   renderPOIList()  ────────────────►│                │
  │   loadOverview() ──►│                  │                │
  │                    │── GET panorama ─►│                │
  │                    │◄── JPG ─────────┤                │
  │                    │                  │── render       │
  │   initMinimap() ───►│                  │                │
  │                    │── GET tiles ────►│                │
  │                    │◄── PNG tiles ───┤                │
  │                    │                  │                │── render map
  │                    │                  │                │
  │── bindSmartUI()    │                  │                │
  │── bindMinimapResize()                 │                │
  │── bindFrameDrag()  │                  │                │
  │── bindMobilePanelDrag() × 2          │                │
  │                    │                  │                │
  │── panorama.on("load") ──────────────►│                │
  │   └── hide loader  │                  │                │
```

### 11.2 Scene Change Runtime

```
User            App State        Pannellum       Leaflet         DOM
  │                │                │               │              │
  │─ openPOI(3) ──►│                │               │              │
  │                │─ activePOI=poi │               │              │
  │                │                │               │              │
  │                │─ destroy() ───►│               │              │
  │                │─ new viewer() ►│               │              │
  │                │                │─ load pano    │              │
  │                │                │─ add hotspot  │              │
  │                │                │               │              │
  │                │─ updateMinimap(poi) ──────────►│              │
  │                │                │  ─ setView    │              │
  │                │                │  ─ highlight  │              │
  │                │                │               │              │
  │                │─ render detail HTML ──────────────────────────►│
  │                │─ lucide.createIcons() ────────────────────────►│
  │                │                │               │              │
  │                │─ hide poiPanel ─────────────────────────────►│
  │                │─ show detailPanel ──────────────────────────►│
  │                │─ reset expanded/dragging ───────────────────►│
  │                │                │               │              │
```

---

*End of Document*
