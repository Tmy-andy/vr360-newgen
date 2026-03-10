# 05 — DEMO HTML SPECIFICATION (Corrected)
## Website 360 Thế Hệ Mới — Trang Demo Thuần HTML

**Version:** 3.1  
**Date:** 2026-03-10  
**Updated:** Thêm sequence diagrams (Scene Navigation, Drag Interaction, Smart UI, Loader)  
**Related:** 01-SYSTEM-ARCHITECTURE · 03-UI-UX-SPEC · 04-TECHNICAL-IMPLEMENTATION  

---

## Mục Lục

1. [Concept Chính Xác](#1-concept-chính-xác)
2. [Luồng Hoạt Động](#2-luồng-hoạt-động)
3. [Kiến Trúc UI](#3-kiến-trúc-ui)
4. [Data Structure (JSON)](#4-data-structure-json)
5. [Smart UI (Tương Tác Thông Minh)](#5-smart-ui-tương-tác-thông-minh)
6. [Responsive](#6-responsive)
7. [Loader](#7-loader)
8. [Sequence Diagrams](#8-sequence-diagrams)
9. [Design Tokens](#9-design-tokens-demo-implementation)
10. [Deployment](#10-deployment)

---

## 1. Concept Chính Xác

> **VR360 KHÔNG PHẢI tính năng phụ. VR360 CHÍNH LÀ website.**

Khi khách truy cập vào website, họ NGAY LẬP TỨC đứng trong không gian 360 của khách sạn. Nền trang web là panorama 360 tương tác được. Mọi thành phần website (menu, thông tin, booking, điều hướng) đều là overlay trên nền 360.

**KHÔNG CÓ trang listing truyền thống.** Toàn bộ website là MỘT trải nghiệm 360 liền mạch.

### So sánh

| Website truyền thống | Website 360 thế hệ mới |
|---------------------|----------------------|
| Homepage → Room listing → Room detail → 360 viewer | Vào web = đứng trong lobby 360 → click "Rooms" = chuyển scene 360 sang phòng → panel info overlay |
| 360 là button phụ để bấm vào xem | 360 là NỀN CỦA TOÀN BỘ WEBSITE |
| Trang có ảnh tĩnh + text | Trang có panorama 360 tương tác + text overlay |

---

## 2. Luồng Hoạt Động

```
[Truy cập website]
    │
    ▼
[Loading screen]
│  - Logo khách sạn (ảnh gốc, 100px, màu nguyên bản)
│  - Progress bar (gold) với 3 giai đoạn:
│    30% "Tải giao diện..."
│    55% "Kết nối không gian 360°..."
│    80% "Chuẩn bị trải nghiệm..."
│  - Khi panorama loaded → 100% "Chào mừng!" → fade out 500ms
    │
    ▼
[Scene: LOBBY 360°]   ← Đây là "homepage"
│  - Nền: panorama 360 lobby khách sạn
│  - Overlay: logo, menu, hotel info panel (bên phải)
│  - Hotspot trong 360: "Phòng nghỉ →", "Nhà hàng →", "Vịnh Nha Trang (info)"
│  - User có thể: xoay 360, click hotspot, click menu
    │
    ├── Click menu "Rooms" hoặc hotspot "Phòng nghỉ"
    │       ▼
    │   [Scene: ROOMS 360°]   ← Scene 360 khác (overview phòng)
    │   │  - Nền: panorama 360 hành lang phòng / phòng mẫu
    │   │  - Panel phải: danh sách hạng phòng (thumb list)
    │   │  - Click một phòng → chuyển scene 360
    │       │
    │       ├── Click "Deluxe Twin Ocean View"
    │       │       ▼
    │       │   [Scene: DELUXE BEDROOM 360°]
    │       │   │  - Nền: panorama 360 phòng Deluxe
    │       │   │  - Panel: tên, mô tả, tiện nghi, giá, Book Now
    │       │   │  - Hotspot: "Ban công →", "Ocean View (info)", "Bathtub (info)"
    │       │   │  - Scene nav bar: [Bedroom] [Balcony]
    │       │       │
    │       │       ├── Click "Ban công →"
    │       │       │       ▼
    │       │       │   [Scene: DELUXE BALCONY 360°]
    │       │       │   - Nền: panorama 360 ban công
    │       │       │   - Panel: vẫn là Deluxe Twin info
    │       │       │   - Hotspot: "← Phòng ngủ", "Sunset Point"
    │       │       │
    │       │       └── Click "Book Now" → Mở booking engine
    │       │
    │       ├── Click "Suite Ocean View" → [Scene: SUITE 360°]
    │       └── Click "Pacific Double" → [Scene: PACIFIC 360°]
    │
    ├── Click menu "Dining"
    │       ▼
    │   [Scene: DINING 360°]
    │   - Nền: panorama 360 nhà hàng
    │   - Panel: danh sách nhà hàng
    │   - Hotspot: "← Lobby", "Tiện ích →"
    │
    ├── Click menu "Recreation"
    │       ▼
    │   [Scene: RECREATION 360°]
    │   - Nền: panorama 360 pool/spa area
    │   - Panel: danh sách tiện ích
    │
    └── Bất kỳ lúc nào: drag/touch panorama
            ▼
        [Smart UI]: Ẩn overlay, chỉ còn 360 thuần
        [Sau 5s idle]: Overlay hiện lại
        [Click nút 👁]: Toggle ẩn/hiện UI thủ công
```

---

## 3. Kiến Trúc UI

```
┌──────────────────────────────────────────────────────┐
│ Z-INDEX STACK (toàn bộ trên 1 viewport, không scroll) │
│                                                        │
│  z:0    PANORAMA 360 (fixed, full viewport)           │
│         ↕ User drag/touch/zoom tương tác trực tiếp    │
│                                                        │
│  z:10   UI LAYER (pointer-events: none container)     │
│         ↕ Chứa toàn bộ overlay UI                      │
│         ↕ class "hidden" → ẩn tất cả con trừ restore  │
│                                                        │
│  z:10   ├── TOPBAR (logo + nav + booking + hamburger) │
│         │   ↕ gradient fade trên nền 360               │
│         │                                              │
│         ├── CONTENT PANEL (right / bottom sheet)       │
│         │   ↕ Nội dung thay đổi theo section           │
│         │   ↕ Welcome / Room list / Room detail / Svc  │
│         │   ↕ Panel-drag-bar (mobile)                  │
│         │                                              │
│         ├── SCENE NAV BAR (bottom center)              │
│         │   ↕ Sub-scene tabs: [Bedroom] [Balcony]      │
│         │                                              │
│         ├── LOCATION PILL (bottom left)                │
│         │   ↕ "Nha Trang, Việt Nam · 28°C"            │
│         │                                              │
│         └── UI RESTORE BUTTON (bottom right)           │
│             ↕ Chỉ hiện khi UI hidden                   │
│                                                        │
│  z:50   INFO CARDS (floating, positioned by click)    │
│                                                        │
│  z:200  MOBILE MENU OVERLAY                           │
│         ↕ Full-screen, header/body/footer layout       │
│         ↕ Numbered nav links, stagger animation        │
│         ↕ Contact info + social links footer           │
│                                                        │
│  z:9999 LOADING SCREEN (ẩn sau khi panorama loaded)   │
└──────────────────────────────────────────────────────┘
```

---

## 4. Data Structure (JSON)

Mỗi "section" = 1 scene 360, đồng thời là 1 "trang" của website:

```javascript
{
  id: "room-deluxe",          // Unique ID
  navLabel: null,              // null = không hiện trên main nav (sub-page)
  panorama: "url.jpg",         // Ảnh equirectangular
  yaw: -20, pitch: 0, hfov: 100, autoRotate: 0.4,  // Camera defaults
  
  panelType: "room-detail",    // Loại panel content
  badge: "Deluxe",
  title: "Deluxe Twin Room",
  subtitle: "Ocean View",
  description: "...",
  
  // Room-specific fields
  area: 35, maxAdults: 2, bedType: "2 Twin Beds",
  view: "Ocean", floor: "8-15",
  amenities: ["Ocean View", "Balcony", ...],
  price: "2,500,000",
  
  // Sub-scenes (Bedroom ↔ Balcony)
  subScenes: [
    {id: "room-deluxe", label: "Bedroom"},
    {id: "room-deluxe-balcony", label: "Balcony"}
  ],
  
  // Hotspots IN the 360 scene
  hotspots: [
    {pitch:-6, yaw:140, type:"nav", label:"Ban công →", target:"room-deluxe-balcony"},
    {pitch:8, yaw:-50, type:"info", label:"Ocean View", desc:"Tầm nhìn 180°..."},
  ]
}
```

### Panel Types:

| panelType | Hiển thị |
|-----------|---------|
| `welcome` | Giới thiệu KS, highlights grid, CTA "Khám Phá Phòng Nghỉ" + "Check Availability" |
| `room-listing` | Danh sách hạng phòng (clickable thumb cards → chuyển scene) |
| `room-detail` | Thông tin phòng: meta row (area/adults/bed/view), tiện nghi grid, giá, Book Now + Check Availability + Xem phòng khác |
| `service` | Danh sách dịch vụ: thumb cards (nhà hàng / tiện ích) + Enquire Now |

### Demo Data: 9 Sections

| id | navLabel | panelType | Mô tả |
|----|----------|-----------|-------|
| `lobby` | Home | welcome | Lobby khách sạn, điểm bắt đầu |
| `rooms` | Rooms | room-listing | Danh sách 4 hạng phòng |
| `room-deluxe` | — | room-detail | Deluxe Twin Ocean View, 35m² |
| `room-deluxe-balcony` | — | room-detail | Ban công phòng Deluxe (sub-scene) |
| `room-superior` | — | room-detail | Superior Twin Mountain View, 28m² |
| `room-pacific` | — | room-detail | Pacific Double Balcony Bathtub, 42m² |
| `room-suite` | — | room-detail | Suite Exclusive Ocean View, 80m² |
| `dining` | Dining | service | 3 nhà hàng + 1 bar |
| `recreation` | Recreation | service | 5 tiện ích (pool, spa, fitness, beach, meeting) |

---

## 5. Smart UI (Tương Tác Thông Minh)

```
STATE MACHINE:

[UI VISIBLE] ──user drag/touch 360──→ [UI HIDDEN]
      ↑                                     │
      │                                     │
      └───────idle 5 giây─────────────────┘
      
[UI HIDDEN] ──click nút 👁──→ [UI VISIBLE]
[UI VISIBLE] ──click nút 👁──→ [UI HIDDEN] (manual, không auto-show)
```

Khi UI hidden:
- `.ui-layer` thêm class `hidden`
- Tất cả con element: `opacity:0`, `transform:translateY(8px)`, `pointer-events:none`
- Ngoại trừ `.ui-restore`: vẫn visible, cho phép user bấm để restore UI
- Transition: `opacity .4s var(--ease)`

### Sibling Hiding (Mobile)

Khi content panel ở trạng thái `expanded` hoặc `dragging`:
- `.location-pill` và `.scene-nav` tự động ẩn (CSS sibling selector `~`)
- Tránh UI chồng lấn khi panel chiếm hết màn hình mobile

---

## 6. Responsive

### Desktop (>860px)

- Panel bên phải: `width:380px`, `right:28px`, `top:84px`, `bottom:28px`
- Topbar đầy đủ navigation links
- Scene nav: bottom center
- Location pill: bottom left
- Panel drag bar: `display:none`

### Desktop nhỏ (860px–1100px)

- Panel: `width:340px`, `right:16px`, `top:74px`, `bottom:16px`

### Mobile (≤860px)

- Topbar: nav links ẩn, hamburger hiện
- Panel: bottom sheet toàn chiều rộng, `max-height:90vh`
- Panel collapsed: `transform:translateY(calc(100% - 160px))` — lộ 160px
- Panel expanded: `transform:translateY(0)`
- Panel drag bar hiện: `width:36px`, `height:4px`, gold khi đang drag
- Scene nav: `bottom:calc(160px + 16px)` — nổi phía trên panel
- Location pill: `bottom:calc(160px + 16px)`

### Mobile Panel Drag System

```
┌──────────────────────────────────────────────────────┐
│ HAI NGUỒN DRAG (dragSource):                         │
│                                                       │
│ 1. DRAG-BAR ("bar"):                                 │
│    - Kéo tự do theo ngón tay                          │
│    - Snap dựa trên HƯỚNG KÉO (direction-based):      │
│      · Kéo lên >= 20px → expand (translateY=0)        │
│      · Kéo xuống >= 20px → collapse                   │
│      · Kéo < 20px → giữ nguyên state cũ              │
│    - Cho phép user nhìn thấy panel di chuyển mượt     │
│                                                       │
│ 2. PANEL BODY ("panel"):                              │
│    - Chỉ hoạt động khi scroll đang ở top (scrollTop=0)│
│    - Auto-snap ngay khi detect 15px gesture:           │
│      · Vuốt xuống 15px → collapse ngay (không kéo theo)│
│      · Vuốt lên 15px khi đã expanded → cancel drag    │
│        (cho phép scroll nội dung bình thường)          │
│      · Vuốt lên 15px khi collapsed → expand           │
│    - Không kéo tự do — chỉ detect hướng rồi snap     │
│                                                       │
│ ANIMATION PATTERN (requestAnimationFrame):            │
│  1. Bỏ class "dragging" (có transition:none!important)│
│  2. requestAnimationFrame(() => {                     │
│       set transition inline                           │
│       set target transform                            │
│       listen transitionend + setTimeout fallback      │
│     })                                                │
│  3. Khi transition xong:                              │
│       remove inline styles                            │
│       add/remove class "expanded"                     │
│                                                       │
│ ANTI-FLASH PATTERN:                                   │
│  Khi bắt đầu drag từ trạng thái expanded:            │
│  1. getCurrentTranslateY() TRƯỚC khi bỏ class        │
│  2. Set inline transform để lock vị trí               │
│  3. Rồi mới bỏ class "expanded"                      │
│  4. Thêm class "dragging"                             │
│  → Không bị flash/jump khi chuyển từ class → inline   │
│                                                       │
│ EVENT BINDING:                                        │
│  - MutationObserver trên document.body                │
│  - Tự động bind mousedown/touchstart khi panel xuất hiện│
│  - Flag _dragBound tránh bind trùng                   │
└──────────────────────────────────────────────────────┘
```

### Mobile Menu Overlay

```
┌──────────────────────────────────────────────────────┐
│ STRUCTURE: 3 phần (header / body / footer)            │
│                                                       │
│ ┌─ MOB-HEADER ─────────────────────────────────────┐ │
│ │ [Logo img (white)] [×]                            │ │
│ └───────────────────────────────────────────────────┘ │
│                                                       │
│ ┌─ MOB-BODY ───────────────────────────────────────┐ │
│ │ ┌─────────────────────────────────────────────┐   │ │
│ │ │ 01   Home                              →    │   │ │
│ │ ├─────────────────────────────────────────────┤   │ │
│ │ │ 02   Rooms                             →    │   │ │
│ │ ├─────────────────────────────────────────────┤   │ │
│ │ │ 03   Dining                            →    │   │ │
│ │ ├─────────────────────────────────────────────┤   │ │
│ │ │ 04   Recreation                        →    │   │ │
│ │ └─────────────────────────────────────────────┘   │ │
│ │                                                   │ │
│ │ [📅 Book Now]                                     │ │
│ └───────────────────────────────────────────────────┘ │
│                                                       │
│ ┌─ MOB-FOOTER ─────────────────────────────────────┐ │
│ │ 📞 +84 258 352 9999                              │ │
│ │ ✉  info@botonblue.com                            │ │
│ │ 📍 86 Trần Phú, Nha Trang                        │ │
│ │                                                   │ │
│ │ [Facebook] [Instagram] [YouTube]                  │ │
│ └───────────────────────────────────────────────────┘ │
│                                                       │
│ ANIMATIONS:                                           │
│ - Nav links: stagger translateX(-20px→0) + opacity    │
│   delay: 80ms + i*60ms per item                       │
│ - CTA: translateX(-20px→0), delay 400ms               │
│ - Footer: translateY(10px→0) + opacity, delay 200ms   │
│ - Active link + hover: color gold                     │
│ - Arrow icon: opacity 0→1, translateX(-8px→0) on hover│
│                                                       │
│ TYPOGRAPHY:                                           │
│ - Nav links: font-family serif, 32px, font-weight 400 │
│ - Number prefix: font-family sans, 11px, font-weight 600│
│ - Contact items: 12px, color text-mute                │
│                                                       │
│ BACKGROUND: rgba(6,10,16,0.97) + backdrop-filter blur(50px)│
│ Z-INDEX: 200                                          │
└──────────────────────────────────────────────────────┘
```

---

## 7. Loader

```
┌──────────────────────────────────────────────────────┐
│ LOADER SCREEN (z:9999, fixed, full viewport)          │
│                                                       │
│              [Logo Image]                             │
│              100px height                             │
│              Màu nguyên bản (không filter)            │
│                                                       │
│           ═══════════════════                         │
│           Progress bar (gold, 200px width)            │
│                                                       │
│           "Đang tải không gian..."                    │
│           12px, uppercase, letter-spacing .1em        │
│                                                       │
│ PROGRESS STEPS (600ms interval):                      │
│   Step 1: 30% → "Tải giao diện..."                   │
│   Step 2: 55% → "Kết nối không gian 360°..."         │
│   Step 3: 80% → "Chuẩn bị trải nghiệm..."           │
│   Panorama loaded: 100% → "Chào mừng!" → done 500ms  │
│                                                       │
│ DONE: opacity→0, visibility→hidden (transition .8s)   │
└──────────────────────────────────────────────────────┘
```

---

## 8. Sequence Diagrams

### 8.1 Scene Navigation Flow

```
  User              navigateTo()       Pannellum          Panel/UI           SceneNav
   │                     │                │                  │                  │
   │  click nav/hotspot  │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ closeCard()    │                  │                  │
   │                     ├──────────────────────────────────►│                  │
   │                     │                │                  │ slide-out panel  │
   │                     │                │                  │ (translateX+30,  │
   │                     │                │                  │  opacity→0)      │
   │                     │                │                  │                  │
   │                     │ viewer.destroy()                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ remove instance  │                  │
   │                     │                │                  │                  │
   │                     │ initViewer(sec)│                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ create viewer    │                  │
   │                     │                │ load panorama    │                  │
   │                     │                │ add hotSpots     │                  │
   │                     │                │                  │                  │
   │                     │ renderPanel(sec)                  │                  │
   │                     ├──────────────────────────────────►│                  │
   │                     │                │                  │ build HTML       │
   │                     │                │                  │ (panelType:      │
   │                     │                │                  │  welcome/room/   │
   │                     │                │                  │  listing/service)│
   │                     │                │                  │ slide-in (0.5s)  │
   │                     │                │                  │                  │
   │                     │ renderSceneNav(sec)               │                  │
   │                     ├─────────────────────────────────────────────────────►│
   │                     │                │                  │                  │ build tabs
   │                     │                │                  │                  │ if subScenes
   │                     │                │                  │                  │ ≥ 2
   │                     │                │                  │                  │
   │                     │ updateNav(id)  │                  │                  │
   │                     ├──────────────────────────────────►│                  │
   │                     │                │                  │ highlight active │
   │                     │                │                  │ (parent mapping) │
   │  ◄─────────────────────────────────── scene ready ──────────────────────  │
   │  panorama interactive                │                  │                  │
```

### 8.2 Mobile Bottom Sheet Drag Flow (Drag-bar Source)

```
  User (touch)      onDragStart()      onDragMove()       onDragEnd()        Panel DOM
   │                     │                │                  │                  │
   │  touchstart on      │                │                  │                  │
   │  .panel-drag        │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ getCurrentTranslateY()            │                  │
   │                     │ (getComputedStyle → matrix parse) │                  │
   │                     │                │                  │                  │
   │                     │ if expanded:   │                  │                  │
   │                     │  set inline transform (lock pos)  │                  │
   │                     │  THEN remove .expanded            │                  │
   │                     │  → NO visual flash ──────────────────────────────────►
   │                     │                │                  │                  │
   │                     │ add .dragging  │                  │  transition:none │
   │                     │ (transition:none!important)───────────────────────────►
   │                     │ dragSource="bar"                  │                  │
   │                     │                │                  │                  │
   │  touchmove (finger) │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ clamp Y between  │                  │
   │                     │                │ 0 and collapsed  │                  │
   │                     │                │ set inline       │                  │
   │                     │                │ transform ───────────────────────────►
   │                     │                │ (free follow)    │  follows finger  │
   │                     │                │                  │                  │
   │  touchend           │                │                  │                  │
   ├───────────────────────────────────────────────────────►│                  │
   │                     │                │                  │                  │
   │                     │                │  dragDelta = currentY - startTranslate
   │                     │                │                  │                  │
   │                     │                │  < -20px (up)    │  rAF(() => {     │
   │                     │                │  ────────────────►  transition .35s │
   │                     │                │                  │  translateY(0)   │
   │                     │                │                  │  → EXPAND        │
   │                     │                │                  │ })               │
   │                     │                │                  │                  │
   │                     │                │  > 20px (down)   │  rAF(() => {     │
   │                     │                │  ────────────────►  transition .35s │
   │                     │                │                  │  translateY(col) │
   │                     │                │                  │  → COLLAPSE      │
   │                     │                │                  │ })               │
   │                     │                │                  │                  │
   │                     │                │  |delta| < 20px  │  → RESTORE       │
   │                     │                │  ────────────────►  previous state  │
   │                     │                │                  │                  │
   │                     │                │                  │  transitionend:  │
   │                     │                │                  │  remove inline   │
   │                     │                │                  │  toggle .expanded│
   │  ◄──────────────────────────────────── animation done ─┤                  │
```

### 8.3 Mobile Bottom Sheet Drag Flow (Panel Body Source)

```
  User (touch)      onDragStart()      onDragMove()       onDragEnd()        Panel DOM
   │                     │                │                  │                  │
   │  touchstart on      │                │                  │                  │
   │  .panel-scroll      │                │                  │                  │
   │  (scrollTop === 0)  │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ dragSource="panel"                │                  │
   │                     │ add .dragging ──────────────────────────────────────►│
   │                     │                │                  │                  │
   │  touchmove          │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │                  │                  │
   │                     │                │ delta = clientY - startY            │
   │                     │                │                  │                  │
   │                     │                │ |delta| < 15px   │                  │
   │                     │                │ → wait (no action)                  │
   │                     │                │                  │                  │
   │                     │                │ delta < -15 (up) │                  │
   │                     │                │ AND expanded?    │                  │
   │                     │                │ → cancelDrag()   │                  │
   │                     │                │   allow scroll ──────────────────────►
   │                     │                │                  │                  │
   │                     │                │ delta < -15 (up) │                  │
   │                     │                │ AND collapsed?   │                  │
   │                     │                │ → SNAP EXPAND ───────────────────────►
   │                     │                │   (immediate,    │  rAF → .expanded │
   │                     │                │    no free drag) │                  │
   │                     │                │                  │                  │
   │                     │                │ delta > 15 (down)│                  │
   │                     │                │ → SNAP COLLAPSE ─────────────────────►
   │                     │                │   (immediate)    │  rAF → collapse  │
   │                     │                │                  │                  │
   │  ◄──────────────────────────────────── snap done ──────┤                  │
```

### 8.4 Smart UI Interaction Flow

```
  User             PanoramaViewer       SmartUI Logic       UI Layer          Idle Timer
   │                     │                │                  │                  │
   │  drag/touch 360     │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ mousedown/     │                  │                  │
   │                     │ touchstart     │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ add .hidden      │                  │
   │                     │                ├─────────────────►│                  │
   │                     │                │                  │ opacity → 0      │
   │                     │                │                  │ translateY(8px)  │
   │                     │                │                  │ pointer-events:  │
   │                     │                │                  │ none             │
   │                     │                │                  │                  │
   │                     │                │                  │ EXCEPT:          │
   │                     │                │                  │ .ui-restore      │
   │                     │                │                  │ remains visible  │
   │                     │                │                  │                  │
   │  release            │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ mouseup/       │                  │                  │
   │                     │ touchend       │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ clearTimeout     │                  │
   │                     │                ├─────────────────────────────────────►│
   │                     │                │                  │                  │ start 5s
   │                     │                │                  │                  │
   │                     │                │                  │                  │
   │  (idle 5 seconds)   │                │                  │                  │
   │                     │                │◄─────────────────────── timeout ────┤
   │                     │                │ remove .hidden   │                  │
   │                     │                ├─────────────────►│                  │
   │                     │                │                  │ opacity → 1      │
   │                     │                │                  │ translateY(0)    │
   │  ◄──────────────────────────────────── UI visible ─────┤                  │
   │                     │                │                  │                  │
   │                     │                │                  │                  │
   │  click 👁 button    │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ toggle .hidden   │                  │
   │                     │                ├─────────────────►│                  │
   │                     │                │ (manual mode,    │ toggle visibility│
   │                     │                │  no auto-show)   │                  │
   │  ◄──────────────────────────────────── UI toggled ─────┤                  │
```

### 8.5 Loader & Panorama Init Flow

```
  Browser            Loader Screen       Progress Bar       Pannellum         navigateTo()
   │                     │                │                  │                  │
   │  page load          │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ show (z:9999)  │                  │                  │
   │                     │ display logo   │                  │                  │
   │                     │                │                  │                  │
   │                     │ setInterval    │                  │                  │
   │                     │ (600ms)        │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ 30% "Tải         │                  │
   │                     │                │ giao diện..."    │                  │
   │                     │                │                  │                  │
   │                     │  +600ms        │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ 55% "Kết nối     │                  │
   │                     │                │ không gian 360°" │                  │
   │                     │                │                  │                  │
   │                     │  +600ms        │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ 80% "Chuẩn bị    │                  │
   │                     │                │ trải nghiệm..."  │                  │
   │                     │                │                  │                  │
   │                     │                │                  │                  │
   │                     │  navigateTo("lobby", false)       │                  │
   │                     ├──────────────────────────────────────────────────────►│
   │                     │                │                  │                  │ initViewer()
   │                     │                │                  │◄─────────────────┤
   │                     │                │                  │ create viewer    │
   │                     │                │                  │ load panorama    │
   │                     │                │                  │                  │
   │                     │                │                  │ panorama loaded  │
   │                     │◄──────────────────────────────────┤ callback         │
   │                     │                │                  │                  │
   │                     │ clearInterval  │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ 100% "Chào mừng!"│                  │
   │                     │                │                  │                  │
   │                     │ setTimeout     │                  │                  │
   │                     │ (500ms)        │                  │                  │
   │                     │ opacity → 0    │                  │                  │
   │                     │ visibility →   │                  │                  │
   │                     │ hidden (.8s)   │                  │                  │
   │  ◄──────────────────┤               │                  │                  │
   │  panorama visible   │               │                  │                  │
```

### 8.6 Mobile Menu Overlay Flow

```
  User             Hamburger Btn       Mobile Overlay      Nav Links          Footer
   │                     │                │                  │                  │
   │  tap ☰              │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ add .active    │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ opacity 0→1      │                  │
   │                     │                │ (0.4s)           │                  │
   │                     │                │                  │                  │
   │                     │                │ stagger links ──►│                  │
   │                     │                │                  │ each link:       │
   │                     │                │                  │ translateX(-20→0)│
   │                     │                │                  │ opacity 0→1      │
   │                     │                │                  │ delay:           │
   │                     │                │                  │ 80ms + i×60ms    │
   │                     │                │                  │                  │
   │                     │                │ CTA button ──────►                  │
   │                     │                │ delay 400ms      │ translateX(-20→0)│
   │                     │                │                  │                  │
   │                     │                │ footer ──────────────────────────────►
   │                     │                │ delay 200ms      │                  │ translateY
   │                     │                │                  │                  │ (10→0)
   │                     │                │                  │                  │ opacity 0→1
   │  ◄──────────────────── menu ready ──┤                  │                  │
   │                     │                │                  │                  │
   │  tap nav link       │                │                  │                  │
   ├──────────────────────────────────────────────────────►│                  │
   │                     │                │                  │ navigateTo(id)   │
   │                     │ remove .active │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ opacity 1→0      │                  │
   │                     │                │ (0.4s)           │                  │
   │  ◄──────────────────── menu closed ─┤                  │                  │
```

---

## 9. Design Tokens (Demo Implementation)

```css
:root {
  --gold: #C9A861;                          /* Primary accent */
  --gold-soft: rgba(201,168,97,.12);        /* Subtle gold bg */
  --gold-mid: rgba(201,168,97,.3);          /* Medium gold */
  --gold-glow: rgba(201,168,97,.45);        /* Glow/shadow */
  --dark: #060a10;                          /* Background */
  --panel: rgba(8,14,26,.82);               /* Panel background */
  --panel-solid: rgba(12,20,38,.94);        /* Solid panel (info cards) */
  --glass: rgba(255,255,255,.05);           /* Glass morphism */
  --glass-border: rgba(255,255,255,.08);    /* Subtle borders */
  --text: #fff;                             /* Primary text */
  --text-dim: rgba(255,255,255,.6);         /* Secondary text */
  --text-mute: rgba(255,255,255,.35);       /* Muted text */
  --serif: 'Cormorant Garamond', Georgia, serif;
  --sans: 'Outfit', system-ui, sans-serif;
  --ease: cubic-bezier(.16,1,.3,1);         /* Expo ease-out */
}
```

---

## 10. Deployment

File demo chỉ 1 file `index.html` duy nhất (inline CSS + JS + data). Mở trực tiếp trên browser hoặc deploy lên Netlify/Vercel/GitHub Pages.

**Dependencies (CDN):**
- Pannellum 2.5.6 (CSS + JS)
- Google Fonts: Cormorant Garamond + Outfit

---

*Xem file demo thực tế: **index.html***
