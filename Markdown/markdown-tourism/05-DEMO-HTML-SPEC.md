# 05 — DEMO HTML SPECIFICATION
## Du Lịch Bà Rịa — Bản Đồ 360° Tourism Platform

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** 01-SYSTEM-ARCHITECTURE · 03-UI-UX-SPEC · 04-TECHNICAL-IMPLEMENTATION  
**File:** `tourism-map-360.html` (~1235 lines, single file)

---

## Mục Lục

1. [Concept Chính Xác](#1-concept-chính-xác)
2. [Luồng Hoạt Động](#2-luồng-hoạt-động)
3. [Kiến Trúc UI](#3-kiến-trúc-ui)
4. [Data Structure (JS)](#4-data-structure-js)
5. [Smart UI (Tương Tác Thông Minh)](#5-smart-ui-tương-tác-thông-minh)
6. [Responsive & Mobile](#6-responsive--mobile)
7. [Loader](#7-loader)
8. [Sequence Diagrams](#8-sequence-diagrams)
9. [Design Tokens](#9-design-tokens)
10. [Deployment](#10-deployment)

---

## 1. Concept Chính Xác

> **Bản đồ 360° CHÍNH LÀ cách du khách khám phá Bà Rịa — Vũng Tàu.**

Khi du khách mở trang, họ NGAY LẬP TỨC đứng trong không gian 360° tổng quan của tỉnh. Mỗi điểm du lịch là một hotspot trên panorama — click vào để "bay" đến điểm đó và xem panorama riêng.

### So Sánh Với Trang Du Lịch Truyền Thống

| Trang du lịch truyền thống | Tourism Map 360° |
|---------------------------|-----------------|
| Homepage → Listing → Detail → Gallery | Mở web = đứng trong panorama 360° → click marker = bay đến điểm đến |
| Ảnh tĩnh + text | Panorama 360° tương tác + overlay thông tin |
| Google Maps nhúng | Leaflet minimap tích hợp + Pannellum panorama |
| Filter dropdown | Multi-select filter chips |
| Pagination | Scroll list + search |

### So Sánh Với Hotel Demo (index.html)

| Hotel Demo | Tourism Map |
|-----------|------------|
| 1 khách sạn, nhiều phòng | 1 tỉnh, nhiều điểm đến |
| Scene navigation (lobby → room) | Map navigation (overview → POI) |
| Room types (Deluxe, Suite...) | Categories (Biển, Ẩm thực...) |
| Booking CTA | Google Maps CTA |
| Gold accent (#C9A861) | Teal accent (#22D3A7) |
| Serif + Sans fonts | Sans only (Inter) |
| No minimap | Leaflet minimap |
| Inline SVG icons | Lucide Icons CDN |

---

## 2. Luồng Hoạt Động

### 2.1 Main Flow

```
[Truy cập trang]
    │
    ▼
[Loading screen]
│  - "Bản Đồ 360°" (accent: 360°)
│  - "DU LỊCH BÀ RỊA — VŨNG TÀU"
│  - Spinning ring animation
│  - Khi panorama loaded → fade out 0.8s
    │
    ▼
[Scene: OVERVIEW 360°]   ← Đây là "homepage"
│  - Nền: panorama 360° tổng quan (auto-rotate 0.2°/s)
│  - Hotspot markers: 8 điểm du lịch (emoji + tên)
│  - POI Panel: danh sách + filter + search (bên phải)
│  - Minimap: Leaflet map + circle markers
│  - Scene indicator: "Tổng quan · Bà Rịa — Vũng Tàu"
│
│  User chọn filter chip (multi-select)
│  → POI list & hotspots cập nhật
│
│  User search tên/địa chỉ
│  → POI list lọc realtime
│
│  User click POI (card / hotspot / minimap)
    │
    ▼
[Scene: POI 360°]
│  - Nền: panorama 360° riêng của POI (auto-rotate 0.4°/s)
│  - Hotspot: "← Quay lại bản đồ" (đối diện camera)
│  - Detail Panel: badge, tên, địa chỉ, mô tả, info, tags, CTA
│  - Minimap: zoom đến POI, highlight marker
│  - Scene indicator: "Tên POI · Địa chỉ"
│
│  User click "Quay lại" / hotspot back
    │
    ▼
[Scene: OVERVIEW 360°]   ← Quay lại trang chính
```

### 2.2 Smart UI Flow

```
[Đang xem panorama]
│  User mousedown/touchstart trên 360°
    │
    ▼
[UI Auto-Hide]
│  - Topbar: slide up + fade
│  - Panels: slide right (desktop) / slide down (mobile)
│  - Minimap: fade + slide down
│  - Scene indicator: fade + slide down
│  - Nút restore (👁) xuất hiện ở góc phải dưới
│
│  User mouseup/touchend
│  → Start 4s idle timer
│  → Timer hết → UI fade back in
│
│  HOẶC user click nút restore
│  → UI hiện ngay lập tức
```

---

## 3. Kiến Trúc UI

### 3.1 DOM Structure

```html
<body>
  <!-- 1. LOADER (z-index: 9999) -->
  <div class="loader" id="loader">
    <div class="loader-title">Bản Đồ <span>360°</span></div>
    <div class="loader-sub">DU LỊCH BÀ RỊA — VŨNG TÀU</div>
    <div class="loader-ring"></div>
  </div>

  <!-- 2. PANORAMA BACKGROUND (z-index: 0) -->
  <div id="panorama"></div>

  <!-- 3. UI LAYER (z-index: 10, pointer-events: none) -->
  <div class="ui" id="ui">

    <!-- 3a. TOPBAR (z-index: 40) -->
    <div class="topbar">
      <div class="topbar-brand">
        <div class="topbar-icon">360</div>
        <div class="topbar-text">
          <h1>Du Lịch Bà Rịa</h1>
          <p>BẢN ĐỒ THỰC TẾ ẢO 360°</p>
        </div>
      </div>
      <div class="topbar-right">
        <button id="btnMap">Bản đồ</button>     <!-- Toggle minimap -->
        <button id="btnList">Danh sách</button>  <!-- Toggle POI panel -->
      </div>
    </div>

    <!-- 3b. POI PANEL -->
    <div class="poi-panel" id="poiPanel">
      <div class="panel-drag">...</div>           <!-- Mobile drag bar -->
      <div class="poi-header">
        <h2>Điểm đến</h2>
        <button class="poi-close">✕</button>      <!-- Hidden on mobile -->
        <div class="poi-search">
          <input placeholder="Tìm kiếm địa điểm...">
        </div>
        <div class="poi-filters" id="poiFilters">
          <!-- Rendered by renderCategories() -->
        </div>
      </div>
      <div class="poi-list" id="poiList">
        <!-- Rendered by renderPOIList() -->
      </div>
    </div>

    <!-- 3c. DETAIL PANEL -->
    <div class="detail-panel" id="detailPanel">
      <div class="panel-drag">...</div>           <!-- Mobile drag bar -->
      <div class="detail-scroll" id="detailScroll">
        <!-- Rendered by openPOI() -->
      </div>
    </div>

    <!-- 3d. MINIMAP (z-index: 100) -->
    <div class="minimap-wrap" id="minimapWrap">
      <div class="minimap-toolbar">
        <button id="btnMapMove">↔</button>        <!-- Drag handle -->
        <button id="btnMapExpand">⬜</button>      <!-- Expand/collapse -->
        <button>✕</button>                         <!-- Close minimap -->
      </div>
      <div class="minimap-resize" id="minimapResize"></div>
      <div class="minimap-resize-br" id="minimapResizeBR"></div>
      <div id="minimap"></div>                     <!-- Leaflet container -->
    </div>

    <!-- 3e. SCENE INDICATOR (z-index: 25) -->
    <div class="scene-indicator" id="sceneIndicator">
      <div class="scene-dot" id="sceneDot"></div>
      <span id="sceneLabel">Tổng quan · Bà Rịa — Vũng Tàu</span>
    </div>

    <!-- 3f. UI RESTORE (z-index: 50) -->
    <button class="ui-restore" id="uiRestore">👁</button>

  </div>
</body>
```

### 3.2 Component Hierarchy

```
body
├── .loader                          ← Loading overlay
├── #panorama                        ← Pannellum 360° viewer
└── .ui                              ← All interactive UI
    ├── .topbar                      ← Brand + toggles
    ├── .poi-panel                   ← POI list (right side / bottom sheet)
    │   ├── .panel-drag              ← Mobile swipe handle
    │   ├── .poi-header
    │   │   ├── h2 + .poi-close
    │   │   ├── .poi-search
    │   │   └── .poi-filters         ← Multi-select category chips
    │   └── .poi-list                ← Scrollable POI cards
    │       └── .poi-card × N        ← Individual POI entries
    ├── .detail-panel                ← POI detail view
    │   ├── .panel-drag              ← Mobile swipe handle
    │   └── .detail-scroll
    │       ├── .detail-back         ← Back navigation
    │       ├── .detail-head         ← Badge, name, address
    │       ├── .detail-divider
    │       └── .detail-body         ← Description, info grid, tags, CTA
    ├── .minimap-wrap                ← Leaflet minimap container
    │   ├── .minimap-toolbar         ← Move, expand, close buttons
    │   ├── .minimap-resize          ← NE resize handle
    │   ├── .minimap-resize-br       ← BR resize handle
    │   └── #minimap                 ← Leaflet map
    ├── .scene-indicator             ← Current scene label
    └── .ui-restore                  ← Show UI button
```

---

## 4. Data Structure (JS)

### 4.1 CATEGORIES Array

```javascript
const CATEGORIES = [
  { id:"all",     label:"Tất cả",      icon:"globe",    color:undefined },
  { id:"beach",   label:"Biển",        icon:"waves",    color:"#22D3A7" },
  { id:"temple",  label:"Tâm linh",    icon:"church",   color:"#F59E0B" },
  { id:"food",    label:"Ẩm thực",     icon:"utensils", color:"#EC4899" },
  { id:"nature",  label:"Thiên nhiên", icon:"trees",    color:"#3B82F6" },
  { id:"history", label:"Di tích",     icon:"landmark", color:"#8B5CF6" },
  { id:"resort",  label:"Nghỉ dưỡng",  icon:"hotel",    color:"#EF4444" },
];
```

### 4.2 OVERVIEW Config

```javascript
const OVERVIEW = {
  panorama: "https://pannellum.org/images/cerro-toco-0.jpg",
  yaw: 80, pitch: 10, hfov: 120, autoRotate: 0.2
};
```

### 4.3 POIS Array (8 entries)

| # | Name | Cat | Lat/Lng | Emoji |
|---|------|-----|---------|-------|
| 1 | Bãi Trước | beach | 10.346, 107.072 | 🏖️ |
| 2 | Bãi Sau | beach | 10.338, 107.085 | 🌊 |
| 3 | Tượng Chúa Kitô | temple | 10.328, 107.085 | ✝️ |
| 4 | Núi Dinh | nature | 10.503, 107.098 | ⛰️ |
| 5 | Chợ Hải Sản VT | food | 10.350, 107.075 | 🦀 |
| 6 | Bạch Dinh | history | 10.340, 107.072 | 🏛️ |
| 7 | Hồ Tràm Strip | resort | 10.476, 107.375 | 🏨 |
| 8 | Côn Đảo | nature | 8.683, 106.609 | 🐢 |

### 4.4 POI Object Full Schema

```javascript
{
  id: Number,           // Unique ID
  cat: String,          // Category ID
  name: String,         // Display name
  addr: String,         // Short address
  lat: Number,          // Latitude
  lng: Number,          // Longitude
  pitch: Number,        // Hotspot position on overview (vertical)
  yaw: Number,          // Hotspot position on overview (horizontal)
  panorama: String,     // Panorama URL for POI scene
  pYaw: Number,         // Camera yaw for POI scene
  pPitch: Number,       // Camera pitch for POI scene
  pHfov: Number,        // Camera hfov for POI scene
  thumb: String,        // Thumbnail URL (120×120)
  desc: String,         // Long description
  tags: String[],       // Tag labels
  hours: String,        // Operating hours
  distance: String,     // Distance from center
  emoji: String         // Emoji for marker pin
}
```

---

## 5. Smart UI (Tương Tác Thông Minh)

### 5.1 Nguyên Tắc

| Rule | Mô tả |
|------|--------|
| **Ẩn khi tương tác** | mousedown/touchstart trên panorama → toàn bộ UI fade out |
| **Hiện khi idle** | mouseup/touchend → 4s timer → UI fade back in |
| **Nút restore** | Luôn có nút 👁 ở góc phải dưới khi UI ẩn |
| **Click restore** | Click nút → UI hiện ngay, reset timer |

### 5.2 CSS Implementation

```css
/* Transition cho mỗi component */
.topbar         { transition: opacity .4s, transform .4s }
.poi-panel      { transition: opacity .5s, transform .5s }
.detail-panel   { transition: opacity .5s, transform .5s }
.minimap-wrap   { transition: all .4s }
.scene-indicator{ transition: opacity .4s, transform .4s }

/* Hidden state */
.ui.hidden .topbar          { opacity:0; transform:translateY(-20px) }
.ui.hidden .poi-panel       { transform:translateX(calc(100% + 20px)) }
.ui.hidden .detail-panel    { transform:translateX(calc(100% + 20px)) }
.ui.hidden .minimap-wrap    { opacity:0; transform:translateY(8px) }
.ui.hidden .scene-indicator { opacity:0; transform:translateY(8px) }

/* Restore button — inverse visibility */
.ui-restore         { opacity:0; pointer-events:none }
.ui.hidden .ui-restore { opacity:1; pointer-events:auto }

/* Mobile override: panels slide down */
@media(max-width:860px) {
  .ui.hidden .poi-panel,
  .ui.hidden .detail-panel { transform:translateY(100%) }
}
```

---

## 6. Responsive & Mobile

### 6.1 Breakpoint: 860px

Toàn bộ responsive logic nằm trong `@media(max-width:860px)`.

### 6.2 Mobile Bottom Sheet

| Property | Value |
|----------|-------|
| Position | `bottom:0; left:0; right:0` |
| Width | `100%` |
| Max height | `90vh` |
| Border radius | `16px 16px 0 0` |
| Peek height | `160px` |
| Collapsed | `transform: translateY(calc(100% - 160px))` |
| Expanded | `transform: translateY(0)` |
| Hidden (POI panel) | `transform: translateY(100%)` |
| Drag bar | `display: flex` (hidden on desktop) |

### 6.3 Mobile-Specific Behavior

| Feature | Desktop | Mobile |
|---------|---------|--------|
| Panel close button | Visible | `display: none` |
| List toggle (#btnList) | Visible | `display: none` |
| Panel interaction | Scroll only | Swipe + scroll |
| Minimap position | bottom-left (14px) | top-right (52px top, 6px right) |
| Minimap size | 200×150px | 150×110px |
| Topbar subtitle | Visible | Hidden |
| Filter chips | 12px, 6px padding | 11px, 5px padding |
| UI restore button | 44×44px | 38×38px |
| Scene indicator | bottom 14px | bottom calc(160px + 12px) |

### 6.4 Scene Indicator Auto-Hide (Mobile)

Scene indicator tự ẩn khi bottom sheet expanded hoặc đang drag:

```css
.poi-panel.expanded ~ .scene-indicator,
.poi-panel.dragging ~ .scene-indicator,
.detail-panel.visible ~ .scene-indicator {
  opacity: 0;
  pointer-events: none;
  transform: translateX(-50%) translateY(16px);
}
```

---

## 7. Loader

### 7.1 Thiết Kế

```
┌─────────────────────────────────────┐
│                                     │
│          Bản Đồ 360°               │
│     DU LỊCH BÀ RỊA — VŨNG TÀU     │
│                                     │
│           ◠ (spinning)              │
│                                     │
└─────────────────────────────────────┘
```

### 7.2 Implementation

```html
<div class="loader" id="loader">
  <div class="loader-title">Bản Đồ <span>360°</span></div>
  <div class="loader-sub">DU LỊCH BÀ RỊA — VŨNG TÀU</div>
  <div class="loader-ring"></div>
</div>
```

### 7.3 Dismiss Logic

```javascript
viewer.on("load", () => {
  const l = document.getElementById("loader");
  if (!l.classList.contains("done"))
    setTimeout(() => l.classList.add("done"), 400);
});
```

- Trigger: Pannellum `load` event (panorama ready)
- Delay: 400ms trước khi thêm `.done`
- Exit animation: `opacity: 0; visibility: hidden` với `transition: 0.8s`
- Chỉ trigger 1 lần (check `.done` class)

---

## 8. Sequence Diagrams

### 8.1 Scene Navigation

```
User              App              Pannellum          Leaflet
  │                 │                  │                  │
  │── Click POI ───►│                  │                  │
  │                 │                  │                  │
  │                 │── destroy() ────►│                  │
  │                 │── new viewer() ─►│                  │
  │                 │                  │── load panorama  │
  │                 │                  │── render hotspot │
  │                 │                  │                  │
  │                 │── updateMinimap() ─────────────────►│
  │                 │                  │  ── setView()    │
  │                 │                  │  ── highlight    │
  │                 │                  │                  │
  │                 │── render detail  │                  │
  │                 │── toggle panels  │                  │
  │                 │── lucide.createIcons()              │
  │                 │                  │                  │
  │                 │── update indicator                  │
  │                 │                  │                  │
  │── Click back ──►│                  │                  │
  │                 │── destroy() ────►│                  │
  │                 │── loadOverview()►│                  │
  │                 │                  │── load panorama  │
  │                 │                  │── add hotspots   │
  │                 │                  │                  │
  │                 │── updateMinimap(null) ─────────────►│
  │                 │                  │  ── reset view   │
  │                 │── toggle panels  │                  │
  │                 │                  │                  │
```

### 8.2 Drag Interaction (Minimap Resize)

```
User              Handle         Minimap Wrap      Leaflet
  │                  │                │                │
  │── mousedown ────►│                │                │
  │                  │── onStart() ──►│                │
  │                  │                │── .resizing     │
  │                  │                │   (no transition)│
  │                  │                │                │
  │── mousemove ────►│                │                │
  │                  │── onMove() ───►│                │
  │                  │                │── style.width   │
  │                  │                │── style.height  │
  │                  │                │                │
  │                  │                │── invalidateSize() ──►│
  │                  │                │                │── reflow │
  │                  │                │                │
  │── mouseup ──────►│                │                │
  │                  │── onEnd() ────►│                │
  │                  │                │── .resizing     │
  │                  │                │   removed       │
  │                  │                │                │
  │                  │                │── invalidateSize() ──►│
  │                  │                │                │
```

### 8.3 Filter Interaction

```
User           Filter Chips        POI List        Panorama
  │                │                   │                │
  │── Click ──────►│                   │                │
  │  "Biển" chip   │                   │                │
  │                │── toggleCat("beach")               │
  │                │── activeCats.add("beach")          │
  │                │── .active class   │                │
  │                │                   │                │
  │                │── renderPOIList()─►│                │
  │                │                   │── filter POIS  │
  │                │                   │── rebuild DOM  │
  │                │                   │                │
  │                │── loadOverview() ─────────────────►│
  │                │                   │   rebuild      │
  │                │                   │   hotspots     │
  │                │                   │                │
  │── Click ──────►│                   │                │
  │  "Biển" again  │                   │                │
  │                │── toggleCat("beach")               │
  │                │── activeCats.delete("beach")       │
  │                │── remove .active  │                │
  │                │                   │                │
  │                │── renderPOIList()─►│                │
  │                │── loadOverview() ─────────────────►│
  │                │                   │                │
```

---

## 9. Design Tokens

### 9.1 Full Token List

```css
:root {
  /* Primary */
  --accent:        #22D3A7;
  --accent-glow:   rgba(34,211,167,.35);
  --accent-soft:   rgba(34,211,167,.1);

  /* Category */
  --orange:        #F59E0B;
  --blue:          #3B82F6;
  --pink:          #EC4899;
  --red:           #EF4444;
  --purple:        #8B5CF6;

  /* Surface */
  --dark:          #070b14;
  --panel:         rgba(10,16,30,.88);
  --panel-solid:   rgba(10,16,30,.96);
  --glass:         rgba(255,255,255,.05);
  --glass-border:  rgba(255,255,255,.08);
  --glass-hover:   rgba(255,255,255,.12);

  /* Text */
  --text:          #fff;
  --text-dim:      rgba(255,255,255,.65);
  --text-mute:     rgba(255,255,255,.35);

  /* Font */
  --sans:          'Inter', system-ui, -apple-system, sans-serif;

  /* Motion */
  --ease:          cubic-bezier(.16, 1, .3, 1);
}
```

### 9.2 Global Reset & Base

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0 }
html, body {
  width: 100%; height: 100%; overflow: hidden;
  font-family: var(--sans);
  background: var(--dark);
  color: #fff;
  -webkit-font-smoothing: antialiased;
}
```

---

## 10. Deployment

### 10.1 Demo Deployment

```
tourism-map-360.html → Upload to any static host
                     → No build step required
                     → No server-side dependencies
                     → All assets via CDN
```

### 10.2 Production Checklist

| # | Task | Status |
|---|------|--------|
| 1 | Replace demo panoramas with real equirectangular photos | ⬜ |
| 2 | Replace Unsplash thumbnails with actual POI photos | ⬜ |
| 3 | Update POI data (coordinates, descriptions, hours) | ⬜ |
| 4 | Add more POIs for comprehensive coverage | ⬜ |
| 5 | Custom domain + SSL certificate | ⬜ |
| 6 | Add Google Analytics / event tracking | ⬜ |
| 7 | Add `prefers-reduced-motion` media query | ⬜ |
| 8 | Add ARIA labels for accessibility | ⬜ |
| 9 | Add service worker for offline support | ⬜ |
| 10 | Optimize panorama images (progressive JPEG, resize) | ⬜ |
| 11 | Add Open Graph / social media meta tags | ⬜ |
| 12 | Consider API backend for dynamic POI management | ⬜ |

### 10.3 External Dependencies

| Resource | URL | Fallback |
|----------|-----|----------|
| Pannellum | cdn.jsdelivr.net | Self-host JS+CSS |
| Leaflet | unpkg.com | Self-host JS+CSS |
| Lucide | unpkg.com | Self-host JS |
| Inter font | fonts.googleapis.com | System font fallback |
| Map tiles | basemaps.cartocdn.com | OpenStreetMap tiles |
| Demo panoramas | pannellum.org/images | Self-host images |
| Demo thumbs | images.unsplash.com | Self-host images |

---

*End of Document*
