# 01 — SYSTEM ARCHITECTURE SPECIFICATION
## Du Lịch Bà Rịa — Bản Đồ 360° Tourism Platform

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Project:** Tourism Map 360° — Bà Rịa — Vũng Tàu  
**File:** `tourism-map-360.html`

---

## Mục Lục

1. [Tổng Quan Kiến Trúc](#1-tổng-quan-kiến-trúc)
2. [Chi Tiết Từng Lớp](#2-chi-tiết-từng-lớp)
3. [Tech Stack](#3-tech-stack)
4. [Luồng Dữ Liệu Chính](#4-luồng-dữ-liệu-chính)
5. [Sequence Diagrams](#5-sequence-diagrams)
6. [Deployment Architecture](#6-deployment-architecture)
7. [Yêu Cầu Phi Chức Năng](#7-yêu-cầu-phi-chức-năng)

---

## 1. Tổng Quan Kiến Trúc

### 1.1 Triết Lý Thiết Kế

> "Du khách mở bản đồ là đã đứng trong không gian 360° của điểm đến."

Tourism Map 360° là ứng dụng web **single-page** (SPA) thuần HTML/CSS/JS, không dùng framework. Kiến trúc được thiết kế theo mô hình **2 lớp trên client**, không có backend — toàn bộ dữ liệu được nhúng trực tiếp trong file HTML.

### 1.2 Mô Hình 2 Lớp (Two-Layer Client Architecture)

```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT (Browser)                         │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              LỚP 1 — 360° MAP LAYER                   │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │  │
│  │  │  Pannellum   │  │   Leaflet    │  │   Hotspot    │ │  │
│  │  │  VR Engine   │  │   Minimap    │  │   Markers    │ │  │
│  │  │  (360° view) │  │  (2D map)    │  │  (POI pins)  │ │  │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘ │  │
│  │         │                 │                  │         │  │
│  │  ┌──────┴─────────────────┴──────────────────┴──────┐  │  │
│  │  │            Core Application Logic                │  │  │
│  │  │     (Vanilla JS — State, Render, Events)         │  │  │
│  │  └──────────────────────┬───────────────────────────┘  │  │
│  └─────────────────────────┼──────────────────────────────┘  │
│                            │                                  │
│  ┌─────────────────────────┼──────────────────────────────┐  │
│  │              LỚP 2 — UI OVERLAY LAYER                  │  │
│  │  ┌─────────┐ ┌────────┐ ┌──────────┐ ┌─────────────┐  │  │
│  │  │ Topbar  │ │  POI   │ │  Detail  │ │   Scene     │  │  │
│  │  │         │ │ Panel  │ │  Panel   │ │  Indicator  │  │  │
│  │  └─────────┘ └────────┘ └──────────┘ └─────────────┘  │  │
│  │  ┌─────────┐ ┌────────┐ ┌──────────┐ ┌─────────────┐  │  │
│  │  │ Filter  │ │Minimap │ │  Smart   │ │   Lucide    │  │  │
│  │  │ Chips   │ │Toolbar │ │ UI Hide  │ │   Icons     │  │  │
│  │  └─────────┘ └────────┘ └──────────┘ └─────────────┘  │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                    DATA LAYER                          │  │
│  │  ┌──────────────────────────────────────────────────┐  │  │
│  │  │  Inline JS Constants: CATEGORIES, POIS, OVERVIEW │  │  │
│  │  │  State variables: activeCats, activePOI, viewer  │  │  │
│  │  └──────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 So Sánh Với Hotel CMS Platform

| Tiêu chí | Hotel CMS (index.html) | Tourism Map (tourism-map-360.html) |
|-----------|----------------------|-----------------------------------|
| Mục đích | Khám phá phòng khách sạn | Bản đồ du lịch nhiều điểm đến |
| Kiến trúc | 3 lớp (Client + API + DB) | 2 lớp (Client-only, no backend) |
| Navigation | Danh sách phòng → Scene 360° | Bản đồ tổng quan → POI 360° |
| Map | Không có | Leaflet minimap tương tác |
| Filter | Theo loại phòng | Multi-select category chips |
| Mobile | Bottom sheet swipe | Bottom sheet swipe (tương tự) |
| Icons | Inline SVG | Lucide Icons CDN |
| Font | Cormorant Garamond + Outfit | Inter |
| Theme | Luxury Dark (gold accent) | Tourism Dark (teal accent) |

---

## 2. Chi Tiết Từng Lớp

### 2.1 Lớp 1 — 360° Map Layer

#### Pannellum VR Engine
- **Phiên bản:** 2.5.6 (CDN)
- **Vai trò:** Render panorama 360° equirectangular, quản lý hotspot markers
- **Cấu hình chung:**

```
showControls: false
showFullscreenCtrl: false
compass: false
friction: 0.15
minHfov: 50–60
maxHfov: 120–130
autoRotate: 0.2–0.4
sceneFadeDuration: 0
```

- **2 chế độ:**
  - **Overview Mode:** Hiển thị panorama tổng quan + hotspot cho mỗi POI
  - **POI Mode:** Hiển thị panorama riêng của POI + hotspot "quay lại"

#### Leaflet Minimap
- **Phiên bản:** 1.9.4 (CDN)
- **Tile provider:** CartoDB Voyager (`rastertiles/voyager`)
- **CSS filter:** `brightness(.78) contrast(1.15) saturate(.85)` — tối cho dark theme
- **Circle markers:** Mỗi POI = `L.circleMarker` với màu theo category
- **Tương tác:** Click marker → `openPOI()`, double-click → expand/collapse

#### Hotspot Markers (Custom)
- Sử dụng `type: "custom"` của Pannellum
- `createTooltipFunc` render HTML markers trực tiếp
- Mỗi marker có emoji icon + tên + click handler
- CSS animation `markerFloat` (3s infinite)

### 2.2 Lớp 2 — UI Overlay Layer

Toàn bộ UI nằm trong `.ui` div (`position:fixed; inset:0; z-index:10`), với `pointer-events:none` (chỉ child có pointer events).

| Component | Vai trò | z-index |
|-----------|---------|---------|
| Topbar | Brand, toggle buttons (Map, List) | 40 |
| POI Panel | Danh sách POI + filter chips + search | — |
| Detail Panel | Chi tiết POI (badge, desc, info, CTA) | — |
| Minimap Wrap | Leaflet minimap + toolbar + resize | 100 |
| Scene Indicator | Hiển thị scene hiện tại (bottom center) | 25 |
| UI Restore | Nút khôi phục UI khi bị ẩn | 50 |
| Loader | Loading screen (highest) | 9999 |

### 2.3 Data Layer

Không có backend/API — toàn bộ dữ liệu hardcoded trong `<script>`:

- **`CATEGORIES`** — 7 entries (incl. "all"): id, label, icon (Lucide name), color
- **`OVERVIEW`** — panorama URL + camera params cho scene tổng quan
- **`POIS`** — 8 entries: id, cat, name, addr, lat/lng, panorama, camera params, thumb, desc, tags, hours, distance, emoji
- **State variables:** `viewer`, `map`, `activeCats` (Set), `activePOI`, `viewMode`, `poiPanelOpen`, `minimapVisible`, `markers[]`

---

## 3. Tech Stack

### 3.1 CDN Dependencies

| Library | Version | CDN | Vai trò |
|---------|---------|-----|---------|
| Pannellum | 2.5.6 | jsdelivr | 360° panorama viewer |
| Leaflet | 1.9.4 | unpkg | 2D interactive map |
| Lucide | 0.468.0 | unpkg (UMD) | Icon library |
| Inter | — | Google Fonts | Typography |
| CartoDB Voyager | — | basemaps.cartocdn.com | Map tiles |

### 3.2 Không Có Backend

```
┌─────────────────────────────────────────────────┐
│               STATIC FILE HOSTING                │
│  (GitHub Pages / Netlify / Vercel / S3)          │
│                                                   │
│  tourism-map-360.html  ← Single file, ~1235 LOC  │
│                                                   │
│  External CDN:                                    │
│  ├── Pannellum JS + CSS (jsdelivr)               │
│  ├── Leaflet JS + CSS (unpkg)                    │
│  ├── Lucide Icons JS (unpkg)                     │
│  ├── Inter Font (Google Fonts)                   │
│  └── CartoDB Tiles (cartocdn.com)                │
│                                                   │
│  Panorama Images:                                 │
│  └── pannellum.org/images/ (demo)                │
│                                                   │
│  Thumbnail Images:                                │
│  └── unsplash.com (demo)                         │
└─────────────────────────────────────────────────┘
```

### 3.3 Browser Support

- **Target:** Modern browsers (Chrome 80+, Firefox 78+, Safari 13+, Edge 80+)
- **Mobile:** iOS Safari 13+, Chrome Android 80+
- **Yêu cầu:** WebGL (Pannellum), CSS Grid, Flexbox, `backdrop-filter`
- **Touch:** Full touch support (swipe, pinch-to-zoom via Pannellum)

---

## 4. Luồng Dữ Liệu Chính

### 4.1 Application Boot

```
[Page Load]
    │
    ▼
[DOMContentLoaded]
    │
    ├── lucide.createIcons()          ← Render static Lucide icons
    ├── init()
    │   ├── renderCategories()        ← Build filter chips
    │   ├── renderPOIList()           ← Build POI card list
    │   ├── loadOverview()            ← Init Pannellum overview scene
    │   └── initMinimap()             ← Init Leaflet map + markers
    │
    ├── bindSmartUI()                 ← 360 interaction → auto-hide UI
    ├── bindMinimapResize()           ← NE + BR resize handles
    ├── bindFrameDrag()               ← Minimap drag-to-move
    ├── bindMobilePanelDrag(poiPanel) ← Bottom sheet swipe
    └── bindMobilePanelDrag(detailPanel)
```

### 4.2 POI Navigation Flow

```
[User Action]
    │
    ├─ Click POI card / marker / minimap marker
    │      │
    │      ▼
    │  openPOI(id)
    │      ├── loadPOIScene(poi)     ← Destroy & recreate Pannellum
    │      ├── Render detail panel HTML
    │      ├── lucide.createIcons()  ← Icons trong detail panel
    │      ├── Show detail panel, hide POI panel
    │      └── updateMinimap(poi)    ← Zoom to POI on Leaflet
    │
    └─ Click "Quay lại" / back hotspot
           │
           ▼
       backToOverview()
           ├── Hide detail panel
           ├── Show POI panel
           ├── loadOverview()         ← Reload overview scene
           └── updateMinimap(null)    ← Reset Leaflet view
```

### 4.3 Filter Flow

```
[Click filter chip]
    │
    ▼
toggleCat(catId)
    ├── activeCats.add/delete(catId)
    ├── Update chip CSS classes
    ├── renderPOIList()               ← Re-filter & re-render cards
    └── if overview → loadOverview()  ← Re-create hotspots
```

---

## 5. Sequence Diagrams

### 5.1 Page Load → First Interaction

```
Browser          Pannellum       Leaflet        Lucide         UI
  │                 │               │              │            │
  │─ Load HTML ────►│               │              │            │
  │─ Load CDN JS ──►│◄──────────────┤◄─────────────┤            │
  │                 │               │              │            │
  │─ DOMContentLoaded ─────────────────────────────────────────►│
  │                 │               │              │            │
  │                 │               │    createIcons()          │
  │                 │               │◄─────────────┤            │
  │                 │               │              │            │
  │─ init() ───────►│               │              │            │
  │                 │               │              │            │
  │  renderCategories() ───────────────────────────────────────►│
  │  renderPOIList()   ────────────────────────────────────────►│
  │                 │               │              │            │
  │  loadOverview()─►│               │              │            │
  │                 │─ create viewer │              │            │
  │                 │─ load panorama │              │            │
  │                 │─ add hotspots  │              │            │
  │                 │               │              │            │
  │  initMinimap()──►               │              │            │
  │                 │  ─ L.map() ──►│              │            │
  │                 │  ─ tileLayer ─►│              │            │
  │                 │  ─ markers ──►│              │            │
  │                 │               │              │            │
  │  on("load") ───►│               │              │            │
  │                 │──────────────────────────────── hide loader│
  │                 │               │              │            │
```

### 5.2 Smart UI Hide/Show

```
User             Panorama(360°)       UI Layer         Timer
  │                    │                  │               │
  │─ mousedown/touch ─►│                  │               │
  │                    │─ onStart() ─────►│               │
  │                    │                  │─ fadeOutUI() ─►│
  │                    │                  │  .ui.hidden    │
  │                    │                  │               │
  │─ mouseup/touchend─►│                  │               │
  │                    │─ onEnd() ───────►│               │
  │                    │                  │─ resetTimer()─►│
  │                    │                  │               │
  │                    │                  │   (4000ms)    │
  │                    │                  │◄──────────────┤
  │                    │                  │─ fadeInUI()    │
  │                    │                  │  .ui remove   │
  │                    │                  │  .hidden      │
  │                    │                  │               │
  │─ Click restore ───►│                  │               │
  │                    │  ─ showAllUI() ─►│               │
  │                    │                  │─ fadeInUI()    │
  │                    │                  │               │
```

---

## 6. Deployment Architecture

### 6.1 Static Hosting (Đề xuất)

```
┌──────────────────────────────────────┐
│          Static File Host            │
│  (GitHub Pages / Netlify / Vercel)   │
│                                      │
│  /tourism-map-360.html               │
│                                      │
│  → CDN-cached, gzip compressed       │
│  → No server-side processing         │
│  → ~1235 lines, ~45KB uncompressed   │
└──────────────────────────────────────┘
        │
        │ HTTPS
        ▼
┌──────────────────────────────────────┐
│           External CDNs              │
│  jsdelivr  (Pannellum)               │
│  unpkg     (Leaflet, Lucide)         │
│  Google    (Inter font)              │
│  CartoDB   (Map tiles)               │
│  Unsplash  (Thumbnail images)        │
│  Pannellum.org (Demo panoramas)      │
└──────────────────────────────────────┘
```

### 6.2 Production Deployment

Để chuyển sang production:

1. **Panorama images:** Upload equirectangular panorama JPGs riêng lên CDN/S3
2. **Thumbnails:** Dùng ảnh thực tế thay Unsplash demo
3. **Data:** Chuyển POIS array → API endpoint hoặc JSON file riêng
4. **Domain:** Custom domain + SSL
5. **Analytics:** Thêm Google Analytics / tracking

---

## 7. Yêu Cầu Phi Chức Năng

### 7.1 Performance

| Chỉ số | Target | Giải pháp |
|--------|--------|-----------|
| First Paint | < 1s | Inline CSS, loader animation |
| Panorama Load | < 3s | CDN, lazy scene loading |
| UI Interaction | < 100ms | Vanilla JS, no framework overhead |
| Minimap Tiles | < 500ms | CartoDB CDN, browser cache |
| Bundle Size | ~45KB | Single file, no build step |

### 7.2 Accessibility

| Yêu cầu | Trạng thái | Ghi chú |
|----------|-----------|---------|
| Keyboard navigation | Partial | Button focus, không có keyboard 360° nav |
| Screen reader | Partial | Semantic HTML, thiếu aria-labels |
| Color contrast | Pass | White on dark background |
| Touch target | Pass | Min 32px buttons |
| Reduced motion | Not yet | Cần thêm `prefers-reduced-motion` |

### 7.3 Responsive

| Breakpoint | Hành vi |
|-----------|---------|
| Desktop (> 860px) | Side panels, minimap bottom-left, resize handles |
| Mobile (≤ 860px) | Bottom sheet panels, minimap top-right, swipe gestures |

---

*End of Document*
