# 03 — UI/UX DESIGN SPECIFICATION
## Du Lịch Bà Rịa — Bản Đồ 360° Tourism Platform

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** 01-SYSTEM-ARCHITECTURE · 02-DATABASE-API-SPEC · 04-TECHNICAL-IMPLEMENTATION

---

## Mục Lục

1. [Design Principles](#1-design-principles)
2. [Design Tokens](#2-design-tokens)
3. [Layout Architecture](#3-layout-architecture)
4. [Component Specifications](#4-component-specifications)
5. [Page-by-Page UX Flows](#5-page-by-page-ux-flows)
6. [Responsive Breakpoints](#6-responsive-breakpoints)
7. [Animation & Transition Spec](#7-animation--transition-spec)
8. [Sequence Diagrams](#8-sequence-diagrams)
9. [Iconography](#9-iconography)

---

## 1. Design Principles

### 1.1 Nguyên Tắc Chủ Đạo

| # | Nguyên tắc | Giải thích |
|---|-----------|-----------|
| 1 | **Map-First** | Bản đồ 360° là nền chính, UI là overlay |
| 2 | **Explore & Discover** | Du khách khám phá điểm đến qua bản đồ tương tác |
| 3 | **Show on Demand** | UI ẩn khi tương tác 360, hiện lại khi idle |
| 4 | **Category Navigation** | Multi-select filter để lọc theo loại điểm đến |
| 5 | **Touch-Optimized** | Bottom sheet swipe trên mobile, resize trên desktop |
| 6 | **Clean Dark Theme** | Dark background + teal accent, typography rõ ràng |

### 1.2 So Sánh Với Hotel Platform

| Tiêu chí | Hotel CMS | Tourism Map |
|-----------|----------|------------|
| Aesthetic | Luxury (gold, serif font) | Clean Modern (teal, sans font) |
| Navigation | Room list → Detail | Map overview → POI detail |
| Accent color | `#C9A861` (gold) | `#22D3A7` (teal) |
| Font family | Cormorant + Outfit | Inter |
| Map integration | Không | Leaflet minimap |
| Filter | Single category | Multi-select chips |

---

## 2. Design Tokens

### 2.1 CSS Variables

```css
:root {
  /* ── Colors ── */
  --accent: #22D3A7;                          /* Primary accent (teal) */
  --accent-glow: rgba(34,211,167,.35);        /* Glow/shadow effect */
  --accent-soft: rgba(34,211,167,.1);         /* Subtle background */

  /* Category Colors */
  --orange: #F59E0B;                          /* Temple / Tâm linh */
  --blue: #3B82F6;                            /* Nature / Thiên nhiên */
  --pink: #EC4899;                            /* Food / Ẩm thực */
  --red: #EF4444;                             /* Resort / Nghỉ dưỡng */
  --purple: #8B5CF6;                          /* History / Di tích */

  /* Backgrounds */
  --dark: #070b14;                            /* Page background */
  --panel: rgba(10,16,30,.88);                /* Panel bg (glassmorphism) */
  --panel-solid: rgba(10,16,30,.96);          /* Solid panel bg */

  /* Glass Effects */
  --glass: rgba(255,255,255,.05);             /* Glass layer */
  --glass-border: rgba(255,255,255,.08);      /* Subtle borders */
  --glass-hover: rgba(255,255,255,.12);       /* Hover state */

  /* Text */
  --text: #fff;                               /* Primary text */
  --text-dim: rgba(255,255,255,.65);          /* Secondary text */
  --text-mute: rgba(255,255,255,.35);         /* Muted text */

  /* Typography */
  --sans: 'Inter', system-ui, -apple-system, sans-serif;

  /* Animation */
  --ease: cubic-bezier(.16,1,.3,1);           /* Spring-like easing */
}
```

### 2.2 Category Color Mapping

| Category | Color | CSS Class | Dùng cho |
|----------|-------|-----------|---------|
| beach | `#22D3A7` | `.cat-beach` | Marker pin, badge, tags |
| temple | `#F59E0B` | `.cat-temple` | Marker pin, badge, tags |
| food | `#EC4899` | `.cat-food` | Marker pin, badge, tags |
| nature | `#3B82F6` | `.cat-nature` | Marker pin, badge, tags |
| history | `#8B5CF6` | `.cat-history` | Marker pin, badge, tags |
| resort | `#EF4444` | `.cat-resort` | Marker pin, badge, tags |

### 2.3 Typography Scale

| Element | Font | Size | Weight | Line Height |
|---------|------|------|--------|-------------|
| Brand H1 | Inter | 15px | 700 | 1.3 |
| Brand subtitle | Inter | 10.5px | 400 | — |
| Panel H2 | Inter | 17px | 700 | 1.3 |
| Detail name | Inter | 22px | 700 | 1.3 |
| POI card name | Inter | 13.5px | 600 | 1.35 |
| POI card addr | Inter | 11px | 400 | 1.3 |
| Body text | Inter | 13.5px | 400 | 1.75 |
| Filter chip | Inter | 12px | 500 | — |
| Tag | Inter | 10px | 500 | 1.4 |
| Button | Inter | 13px | 700 | — |
| Scene indicator | Inter | 13px | 400/600 | 1.4 |
| Loader title | Inter | 28px | 700 | — |

---

## 3. Layout Architecture

### 3.1 Desktop Layout (> 860px)

```
┌─────────────────────────────────────────────────────────┐
│ ┌─────────────────────────────────────────────────────┐ │
│ │                    TOPBAR                            │ │
│ │  [360 icon] Du Lịch Bà Rịa    [🔵 Bản đồ] [🔵 DS] │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│                                           ┌───────────┐ │
│                                           │           │ │
│                                           │   POI     │ │
│    ┌─────────────┐                        │  PANEL    │ │
│    │  MINIMAP    │    PANORAMA 360°        │  (350px)  │ │
│    │  (200x150)  │    (full screen bg)     │           │ │
│    │  [↔️][⬜][✕]│                        │  Filters  │ │
│    │  Leaflet    │                        │  Search   │ │
│    │  +markers   │                        │  Cards    │ │
│    └─────────────┘                        │           │ │
│                                           └───────────┘ │
│                                                         │
│              ┌──────────────────────┐                   │
│              │   SCENE INDICATOR    │                   │
│              │ 🟢 Tổng quan · BRVT │                   │
│              └──────────────────────┘                   │
└─────────────────────────────────────────────────────────┘
```

### 3.2 Mobile Layout (≤ 860px)

```
┌────────────────────────────────┐
│ ┌────────────────────────────┐ │
│ │ TOPBAR   [360] Du Lịch  [🗺️]│ │
│ └────────────────────────────┘ │
│                      ┌───────┐ │
│                      │MINIMAP│ │
│   PANORAMA 360°      │(top-R)│ │
│   (full screen bg)   └───────┘ │
│                                │
│  ┌──────────────────────┐      │
│  │   SCENE INDICATOR    │      │
│  └──────────────────────┘      │
│ ┌────────────────────────────┐ │
│ │ ─── (drag bar) ───         │ │
│ │                            │ │
│ │   BOTTOM SHEET             │ │
│ │   (POI Panel / Detail)     │ │
│ │   peek: 160px              │ │
│ │   expanded: 90vh           │ │
│ │   swipe up/down            │ │
│ └────────────────────────────┘ │
└────────────────────────────────┘
```

### 3.3 Z-Index Stack

| Layer | z-index | Element |
|-------|---------|---------|
| Panorama | 0 | `#panorama` (position:fixed) |
| UI Layer | 10 | `.ui` (pointer-events:none) |
| Scene Indicator | 25 | `.scene-indicator` |
| Minimap Resize | 30 | `.minimap-resize`, `.minimap-resize-br` |
| Minimap Toolbar | 35 | `.minimap-toolbar` |
| Topbar | 40 | `.topbar` |
| UI Restore | 50 | `.ui-restore` |
| Minimap Wrap | 100 | `.minimap-wrap` |
| Loader | 9999 | `.loader` |

---

## 4. Component Specifications

### 4.1 Loader

```
┌────────────────────────────────┐
│                                │
│        Bản Đồ 360°            │
│   DU LỊCH BÀ RỊA — VŨNG TÀU  │
│         [spinning ring]        │
│                                │
└────────────────────────────────┘
```

- **Background:** `var(--dark)` (#070b14)
- **Title:** 28px, 700 weight, `360°` có color `var(--accent)`
- **Subtitle:** 13px, letter-spacing .06em
- **Ring:** 40×40px, 2px border, `border-top-color: var(--accent)`, spin 0.7s
- **Exit animation:** opacity + visibility 0.8s transition khi `.done`

### 4.2 Topbar

```
┌──────────────────────────────────────────────────────────┐
│ [360] Du Lịch Bà Rịa                 [🟢 Bản đồ] [🟢 DS]│
│       BẢN ĐỒ THỰC TẾ ẢO 360°                            │
└──────────────────────────────────────────────────────────┘
```

- **Height:** Auto (padding 14px 22px)
- **Background:** `linear-gradient(180deg, rgba(7,11,20,.8), transparent)`
- **Brand icon:** 36×36px, `var(--accent)` background, rounded 9px, "360" text
- **Toggle buttons:** 34px height, glass background, Lucide icons
- **Active state:** `var(--accent-soft)` bg, accent border, accent text
- **Mobile:** padding 12px 14px, subtitle hidden, `#btnList` hidden

### 4.3 Filter Chips (Multi-Select)

```
┌────────────────────────────────────────────────────────┐
│ [☐ 🏖️ Biển 2] [☑ ⛪ Tâm linh 1] [☐ 🍴 Ẩm thực 1]  │
│ [☐ 🌲 Thiên nhiên 2] [☐ 🏛️ Di tích 1] [☐ 🏨 NĐ 1]  │
└────────────────────────────────────────────────────────┘
```

- **Layout:** flexbox, flex-wrap, gap 7px
- **Chip:** pill shape (border-radius 100px), padding 6px 12px
- **Checkbox:** 14×14px, border-radius 4px, border 1.5px
- **Active checkbox:** `var(--accent)` fill + white check icon
- **Icon:** Lucide icon (14×14px) + label text + count badge
- **Count badge:** 16×16px min, font-size 9px, weight 800
- **Multi-select behavior:** `activeCats` is a `Set`, toggling adds/removes

### 4.4 POI Card

```
┌───────────────────────────────────┐
│ ┌──────┐                         │
│ │ THUMB│ Bãi Trước             > │
│ │54×54 │ Tp. Vũng Tàu            │
│ │      │ [Biển] [Miễn phí]       │
│ └──────┘                         │
└───────────────────────────────────┘
```

- **Layout:** flex row, align-items center, gap 12px, padding 10px
- **Thumbnail:** 54×54px, border-radius 8px, object-fit cover
- **Name:** 13.5px, weight 600, text-overflow ellipsis
- **Address:** 11px, muted color, text-overflow ellipsis
- **Tags:** pill badges, 10px font, glass background
- **Arrow:** Lucide `chevron-right`, 14×14px
- **Active state:** `var(--accent-soft)` bg, accent border
- **Hover:** `var(--glass-hover)` bg

### 4.5 Detail Panel

```
┌─────────────────────────────┐
│  ← Quay lại bản đồ          │
│                              │
│  [Biển]                      │
│  Bãi Trước                  │
│  📍 Tp. Vũng Tàu            │
│  ──────────────────          │
│  Bãi Trước nằm ngay trung   │
│  tâm thành phố Vũng Tàu...  │
│                              │
│  ┌────────┐  ┌────────────┐  │
│  │🕐 Cả   │  │📍 0 km từ  │  │
│  │  ngày   │  │  trung tâm │  │
│  └────────┘  └────────────┘  │
│                              │
│  Tags                        │
│  [Biển] [Miễn phí] [HH]     │
│                              │
│  [  Mở Google Maps  ]        │
│  [    Quay lại      ]        │
└─────────────────────────────┘
```

- **Back button:** flex, gap 8px, Lucide `chevron-left`
- **Badge:** category color background (20% opacity), uppercase, 10px
- **Name:** 22px, weight 700
- **Address:** 13px, Lucide `map-pin` icon (accent color)
- **Info grid:** 2 columns, Lucide `clock` + `navigation` icons
- **CTA buttons:** `btn-accent` (teal solid) + `btn-outline` (border only)
- **btn-accent hover:** box-shadow glow + translateY(-1px)

### 4.6 Minimap

```
┌───────────────────────────────────┐
│ [↔️] [⬜] [✕]             [↗resize]│
│                                    │
│         LEAFLET MAP                │
│         CartoDB Voyager            │
│         + Circle Markers           │
│                                    │
│                            [↘resize]│
└───────────────────────────────────┘
```

- **Desktop:** bottom-left, 200×150px, border-radius 12px
- **Mobile:** top-right, 150×110px, under topbar (top:52px, right:6px)
- **Background:** CartoDB Voyager tiles + CSS filter (dark mode)
- **Toolbar:** 3 buttons (move, expand, close), 26×26px
- **Resize handles:** top-right (NE) + bottom-right (BR), 22×22px
- **Expanded:** full viewport minus 28px margin
- **Constraints:** min 140×100px, max calc(100vw-28px) × calc(100vh-28px)

### 4.7 Scene Indicator

```
┌──────────────────────────────────────┐
│  🟢  Tổng quan · Bà Rịa — Vũng Tàu  │
└──────────────────────────────────────┘
```

- **Position:** bottom center, translateX(-50%)
- **Desktop:** bottom 14px
- **Mobile:** bottom calc(160px + 12px) — sits above collapsed panel
- **Background:** `var(--panel)` + backdrop-filter blur(30px)
- **Shape:** pill (border-radius 100px)
- **Dot:** 8×8px circle, category color
- **Hidden when:** panel expanded or dragging (CSS sibling selector)

### 4.8 Hotspot Markers (Pannellum)

```
    ┌──────────┐
    │  emoji   │  ← 40×40px, category color bg
    │  (pin)   │
    └────┬─────┘
         ▼ (triangle arrow)
   ┌──────────────┐
   │  POI Name    │  ← label, glass bg
   └──────────────┘
```

- **Pin:** 40×40px, border-radius 12px, category color background
- **Arrow:** CSS pseudo-element `::after`, 6px borders
- **Label:** padding 4px 12px, glass bg, max-width 140px, text-overflow ellipsis
- **Animation:** `markerFloat` — translateY(0 → -4px → 0), 3s infinite
- **Hover:** scale(1.12)

---

## 5. Page-by-Page UX Flows

### 5.1 Overview Mode (Trang Chính)

1. User mở trang → Loader hiển thị (ring animation)
2. Pannellum load panorama tổng quan
3. Loader fade out 0.8s khi panorama sẵn sàng
4. User thấy:
   - 360° panorama (auto-rotate 0.2°/s)
   - Hotspot markers cho mỗi POI (float animation)
   - POI panel bên phải (desktop) / bottom sheet (mobile)
   - Minimap ở góc
   - Scene indicator "Tổng quan · Bà Rịa — Vũng Tàu"

### 5.2 POI Detail Mode

1. User click POI card / hotspot / minimap marker
2. Pannellum destroy → recreate với POI panorama
3. Detail panel slide in (desktop) / bottom sheet peek (mobile)
4. POI panel hidden
5. Scene indicator cập nhật tên POI + category color
6. Minimap zoom đến POI location
7. User có thể:
   - Xem mô tả, giờ mở cửa, khoảng cách
   - Mở Google Maps
   - Quay lại overview

### 5.3 Smart UI Hide

1. User mousedown/touchstart trên panorama
2. Toàn bộ UI fade out (opacity 0, pointer-events none)
3. Chỉ còn nút restore (eye icon) ở góc phải dưới
4. User mouseup/touchend → start 4s idle timer
5. Timer hết → UI fade back in
6. Hoặc user click restore → UI hiện ngay

---

## 6. Responsive Breakpoints

### 6.1 Breakpoint Duy Nhất: 860px

| Property | Desktop (> 860px) | Mobile (≤ 860px) |
|----------|-------------------|-------------------|
| POI Panel | Right side, 350px wide | Bottom sheet, full width |
| Detail Panel | Right side, 350px wide | Bottom sheet, full width |
| Panel interaction | Scroll only | Swipe up/down + scroll |
| Panel peek height | — | 160px |
| Panel max height | calc(100vh - 78px) | 90vh |
| Minimap position | Bottom-left | Top-right (under topbar) |
| Minimap size | 200×150px | 150×110px |
| Topbar padding | 14px 22px | 12px 14px |
| Topbar subtitle | Visible | Hidden |
| Close button | Visible | Hidden |
| List toggle button | Visible | Hidden |
| Filter chip size | 6px 12px, 12px font | 5px 10px, 11px font |
| Scene indicator | Bottom 14px | Bottom calc(160px + 12px) |
| UI restore button | 44×44px | 38×38px |

### 6.2 Bottom Sheet States

```
                    ┌─────────────────────────┐
                    │                         │
                    │   EXPANDED (90vh)        │
                    │   transform: translateY(0)│
                    │                         │
                    │   Full content visible   │
                    │   Scrollable             │
 ▲ Swipe Up        │                         │
 │                  ├─────────────────────────┤
 │                  │                         │
 ▼ Swipe Down      │   COLLAPSED (peek 160px)│
                    │   transform: translateY │
                    │   (calc(100% - 160px))  │
                    │                         │
                    │   Partial content shown  │
                    ├─────────────────────────┤
                    │                         │
                    │   HIDDEN                │
                    │   transform:translateY  │
                    │   (100%)                │
                    │                         │
                    └─────────────────────────┘
```

---

## 7. Animation & Transition Spec

### 7.1 Easing Function

```
--ease: cubic-bezier(.16, 1, .3, 1)
```
"Spring-like" easing — fast start, smooth deceleration.

### 7.2 Transition Table

| Element | Property | Duration | Easing |
|---------|----------|----------|--------|
| Panel slide | transform, opacity | 0.5s | var(--ease) |
| Panel swipe | transform | 0.35s | cubic-bezier(.16,1,.3,1) |
| Smart UI hide | opacity, transform | 0.4s | var(--ease) |
| Loader exit | opacity, visibility | 0.8s | linear |
| Filter chip | all | 0.25s | var(--ease) |
| POI card hover | all | 0.25s | linear |
| Button hover | all | 0.2s | linear |
| Minimap toggle | all | 0.4s | var(--ease) |
| Marker float | transform | 3s | ease-in-out (infinite) |
| Marker hover | transform | 0.3s | var(--ease) |
| Drag bar expand | width, bg | 0.2s | linear |

### 7.3 Disabled Transitions

- `.dragging` class: `transition: none !important` (panel swipe)
- `.resizing` class: `transition: none` (minimap resize)
- `.minimap-wrap.dragging`: `transition: none` (minimap frame drag)

---

## 8. Sequence Diagrams

### 8.1 POI Panel Interaction (Desktop)

```
User              POI Panel        Detail Panel      Pannellum
  │                  │                 │                 │
  │─ Click card ────►│                 │                 │
  │                  │─ openPOI() ────►│                 │
  │                  │                 │─ render HTML    │
  │                  │─ collapsed ─────│                 │
  │                  │                 │─ .visible ──────│
  │                  │                 │                 │─ loadPOIScene()
  │                  │                 │                 │─ new panorama
  │                  │                 │                 │
  │─ Click back ────►│                 │                 │
  │                  │                 │─ backToOverview()│
  │                  │─ show ──────────│─ hide           │
  │                  │                 │                 │─ loadOverview()
  │                  │                 │                 │
```

### 8.2 Bottom Sheet Swipe (Mobile)

```
User              Panel              CSS                JS State
  │                  │                 │                    │
  │─ Touch drag bar ►│                 │                    │
  │                  │─ onDragStart()  │                    │
  │                  │                 │─ .dragging         │
  │                  │                 │  (no transition)   │
  │                  │                 │                    │
  │─ Move finger ───►│                 │                    │
  │                  │─ onDragMove()   │                    │
  │                  │─ style.transform│                    │
  │                  │  = translateY() │                    │
  │                  │                 │                    │
  │─ Release ───────►│                 │                    │
  │                  │─ onDragEnd()    │                    │
  │                  │                 │                    │─ panelExpanded
  │                  │  if delta < -20 │                    │  = true
  │                  │  ─ animate to 0 │─ .expanded         │
  │                  │                 │  (translateY(0))   │
  │                  │  if delta > 20  │                    │  = false
  │                  │  ─ animate peek │─ remove .expanded  │
  │                  │                 │                    │
```

---

## 9. Iconography

### 9.1 Lucide Icons CDN

```
https://unpkg.com/lucide@0.468.0/dist/umd/lucide.js
```

### 9.2 Global Icon Style

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

### 9.3 Icon Usage Map

| Context | Icon Name | Dùng ở |
|---------|-----------|--------|
| Map toggle | `map` | Topbar button |
| List toggle | `list` | Topbar button |
| Close | `x` | POI panel, minimap |
| Search | `search` | POI search input |
| Check | `check` | Filter chip checkbox |
| Chevron right | `chevron-right` | POI card arrow |
| Chevron left | `chevron-left` | Detail back button |
| Map pin | `map-pin` | Detail address |
| Clock | `clock` | Detail hours |
| Navigation | `navigation` | Detail distance |
| Move | `move` | Minimap drag handle |
| Maximize | `maximize-2` | Minimap expand (collapsed) |
| Minimize | `minimize-2` | Minimap expand (expanded) |
| Eye | `eye` | UI restore button |
| Globe | `globe` | Category "all" |
| Waves | `waves` | Category "beach" |
| Church | `church` | Category "temple" |
| Utensils | `utensils` | Category "food" |
| Trees | `trees` | Category "nature" |
| Landmark | `landmark` | Category "history" |
| Hotel | `hotel` | Category "resort" |

### 9.4 Dynamic Icon Rendering

Lucide icons được render động sau khi DOM cập nhật:

```javascript
// Static icons (page load)
lucide.createIcons();

// After dynamic rendering (specific root)
lucide.createIcons({ root: element });
```

Áp dụng tại:
- `renderCategories()` → filter chips
- `renderPOIList()` → POI cards
- `openPOI()` → detail panel
- `toggleMapExpand()` → expand/minimize icon swap

---

*End of Document*
