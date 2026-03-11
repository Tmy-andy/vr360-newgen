# 03 — DASHBOARD UI/UX SPECIFICATION
## Tourism Map 360° — Admin Dashboard

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** `01-DASHBOARD-ARCHITECTURE` · `02-DASHBOARD-DATABASE-API` · `markdown-tourism/03-UI-UX-SPEC`

---

## Mục Lục

1. [Design Principles](#1-design-principles)
2. [Design Tokens](#2-design-tokens)
3. [Layout Architecture](#3-layout-architecture)
4. [Component Library](#4-component-library)
5. [Page Specifications](#5-page-specifications)
6. [Responsive Breakpoints](#6-responsive-breakpoints)
7. [Animations & Transitions](#7-animations--transitions)
8. [Dark Mode](#8-dark-mode)

---

## 1. Design Principles

### 1.1 Nguyên Tắc Chung

| # | Nguyên tắc | Giải thích |
|---|-----------|-----------|
| 1 | **Map-Centric Admin** | Quản lý POI qua bản đồ tương tác — click để đặt vị trí |
| 2 | **Visual Preview** | Preview panorama 360° ngay trong editor |
| 3 | **Simple & Clean** | Ít entity hơn hotel dashboard → UI đơn giản hơn |
| 4 | **Consistent with Frontend** | Dùng cùng color scheme (teal accent) |
| 5 | **Efficiency** | Bulk actions, quick edit, keyboard shortcuts |

### 1.2 So Sánh Với Hotel Dashboard Theme

| Aspect | Hotel Dashboard | Tourism Dashboard |
|--------|----------------|------------------|
| Primary Color | Indigo `#4F46E5` | Teal `#0D9488` |
| Secondary | Slate `#64748B` | Slate `#64748B` |
| Accent | Indigo variations | Teal variations |
| Sidebar | Indigo dark | Teal dark |
| Brand feel | Professional CMS | Explorer / Map-centric |
| Unique element | Booking config panels | Map picker, panorama preview |

---

## 2. Design Tokens

### 2.1 Color System

```css
:root {
  /* ── Primary (Teal — consistent with frontend accent) ── */
  --primary-50:  #F0FDFA;
  --primary-100: #CCFBF1;
  --primary-200: #99F6E4;
  --primary-300: #5EEAD4;
  --primary-400: #2DD4BF;
  --primary-500: #14B8A6;
  --primary-600: #0D9488;       /* ← PRIMARY */
  --primary-700: #0F766E;
  --primary-800: #115E59;
  --primary-900: #134E4A;
  
  /* ── Neutral (Slate) ── */
  --slate-50:  #F8FAFC;
  --slate-100: #F1F5F9;
  --slate-200: #E2E8F0;
  --slate-300: #CBD5E1;
  --slate-400: #94A3B8;
  --slate-500: #64748B;
  --slate-600: #475569;
  --slate-700: #334155;
  --slate-800: #1E293B;
  --slate-900: #0F172A;
  
  /* ── Semantic ── */
  --success: #10B981;           /* Emerald-500 */
  --warning: #F59E0B;           /* Amber-500 */
  --error:   #EF4444;           /* Red-500 */
  --info:    #3B82F6;           /* Blue-500 */
  
  /* ── Category Colors (from frontend) ── */
  --cat-beach:   #22D3A7;
  --cat-temple:  #F59E0B;
  --cat-food:    #EC4899;
  --cat-nature:  #3B82F6;
  --cat-history: #8B5CF6;
  --cat-resort:  #EF4444;
  
  /* ── Surface ── */
  --bg-page:    var(--slate-50);
  --bg-card:    #FFFFFF;
  --bg-sidebar: var(--primary-900);
  --bg-header:  #FFFFFF;
  
  /* ── Typography ── */
  --font-sans: 'Inter', system-ui, -apple-system, sans-serif;
  
  /* ── Spacing ── */
  --sidebar-width: 260px;
  --sidebar-collapsed: 72px;
  --header-height: 64px;
  
  /* ── Border Radius ── */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  
  /* ── Shadow ── */
  --shadow-sm: 0 1px 2px rgba(0,0,0,.05);
  --shadow-md: 0 4px 6px -1px rgba(0,0,0,.1);
  --shadow-lg: 0 10px 15px -3px rgba(0,0,0,.1);
}
```

### 2.2 Typography Scale

| Element | Size | Weight | Color |
|---------|------|--------|-------|
| Page title (H1) | 24px | 700 | `--slate-900` |
| Section title (H2) | 20px | 600 | `--slate-800` |
| Card title (H3) | 16px | 600 | `--slate-800` |
| Body text | 14px | 400 | `--slate-700` |
| Small text | 13px | 400 | `--slate-500` |
| Label | 13px | 500 | `--slate-600` |
| Button | 14px | 500 | — |
| Table header | 12px | 600 | `--slate-500` (uppercase) |
| Badge | 12px | 500 | — |
| Sidebar item | 14px | 500 | `white/70%` |
| Sidebar active | 14px | 600 | `white` |

---

## 3. Layout Architecture

### 3.1 Main Layout

```
┌─────────────────────────────────────────────────────────────────┐
│ ┌────────────┐ ┌──────────────────────────────────────────────┐ │
│ │            │ │  HEADER (64px)                               │ │
│ │            │ │  ┌──────────┐              ┌────┐ ┌────────┐ │ │
│ │            │ │  │ Breadcrumb│              │🔔  │ │ Avatar │ │ │
│ │            │ │  └──────────┘              └────┘ └────────┘ │ │
│ │  SIDEBAR   │ ├──────────────────────────────────────────────┤ │
│ │  (260px)   │ │                                              │ │
│ │            │ │               MAIN CONTENT                   │ │
│ │  Logo      │ │                                              │ │
│ │  ────────  │ │  ┌──────────────────────────────────────┐    │ │
│ │  Dashboard │ │  │                                      │    │ │
│ │  ────────  │ │  │  Page Content (padded 24px)          │    │ │
│ │  NỘI DUNG  │ │  │                                      │    │ │
│ │  Tỉnh/Thành│ │  │  Tables, Forms, Maps, Editors        │    │ │
│ │  Danh mục  │ │  │                                      │    │ │
│ │  Điểm đến  │ │  │                                      │    │ │
│ │  Tags      │ │  │                                      │    │ │
│ │  ────────  │ │  └──────────────────────────────────────┘    │ │
│ │  TRỰC QUAN │ │                                              │ │
│ │  Bản đồ   │ │                                              │ │
│ │  Gallery   │ │                                              │ │
│ │  ────────  │ │                                              │ │
│ │  HỆ THỐNG  │ │                                              │ │
│ │  Users     │ │                                              │ │
│ │  Settings  │ │                                              │ │
│ │  Analytics │ │                                              │ │
│ │            │ │                                              │ │
│ └────────────┘ └──────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Sidebar Detail

```
┌────────────────────┐
│  ┌──────────────┐  │
│  │ 🗺 Tourism   │  │  ← Logo area
│  │    Admin     │  │
│  └──────────────┘  │
│  ──────────────── │
│                    │
│  📊 Dashboard     │  ← Active: bg primary-700, left border accent
│  ──────────────── │
│                    │
│  NỘI DUNG          │  ← Section label (11px, uppercase, dim)
│  🏛 Tỉnh/Thành    │
│  📂 Danh mục      │
│  📍 Điểm đến  ①  │  ← Badge: pending count
│  🏷 Tags          │
│  ──────────────── │
│                    │
│  TRỰC QUAN         │
│  🗺 Cấu hình BĐ  │
│  🖼 Gallery       │
│  ──────────────── │
│                    │
│  HỆ THỐNG          │
│  👥 Người dùng    │
│  ⚙ Cài đặt       │
│  📈 Thống kê      │
│                    │
│  ──────────────── │
│  ┌──────────────┐  │
│  │ 👤 Admin     │  │  ← User info + logout
│  │    admin@..  │  │
│  └──────────────┘  │
└────────────────────┘
```

### 3.3 Sidebar Specs

| Property | Value |
|----------|-------|
| Width (expanded) | 260px |
| Width (collapsed) | 72px |
| Background | `--primary-900` (#134E4A) |
| Text color | `rgba(255,255,255,.7)` |
| Active item bg | `--primary-700` |
| Active left border | 3px `--primary-400` |
| Active text | `#fff` |
| Section label | 11px, 600 weight, uppercase, `rgba(255,255,255,.35)` |
| Hover bg | `rgba(255,255,255,.08)` |
| Icon size | 20×20px |
| Item padding | 10px 16px |
| Item gap | 2px |
| Collapse toggle | Hamburger icon at bottom |

---

## 4. Component Library

### 4.1 shadcn/ui Base Components

| Component | Variant | Dùng cho |
|-----------|---------|---------|
| Button | primary, outline, ghost, destructive | Actions, CTA |
| Input | default, with icon, with addon | Text fields |
| Select | default, multi-select | Dropdowns |
| Textarea | default, autosize | Descriptions |
| Dialog | default | Confirmations, forms |
| Sheet | right, bottom | Slide-over panels |
| Table | default, sortable | Data lists |
| Card | default | Content containers |
| Badge | default, outline, colored | Status, category |
| Tabs | default, underline | Section switching |
| Switch | default | Toggle on/off |
| Checkbox | default | Multi-select |
| Dropdown Menu | default | Actions menu |
| Toast | success, error, info | Notifications |
| Skeleton | default | Loading states |
| Tooltip | default | Hints |
| Command | default | Search/command palette |
| Pagination | default | Table pagination |

### 4.2 Custom Components

#### 4.2.1 MapPicker

Dùng Leaflet để chọn tọa độ cho POI.

```
┌─────────────────────────────────────────────────────┐
│  Chọn vị trí trên bản đồ                           │
│  ┌─────────────────────────────────────────────┐    │
│  │                                             │    │
│  │         LEAFLET MAP (interactive)           │    │
│  │         Click to place marker               │    │
│  │                    📍                        │    │
│  │                                             │    │
│  │                                             │    │
│  └─────────────────────────────────────────────┘    │
│  Lat: [10.346000]  Lng: [107.072300]               │
│  📍 Search: [Tìm địa điểm...            ] [🔍]    │
└─────────────────────────────────────────────────────┘
```

| Property | Value |
|----------|-------|
| Height | 400px (expandable) |
| Default center | Province center |
| Default zoom | Province default zoom |
| Click | Place/move marker, update lat/lng inputs |
| Search | Geocoding search (Nominatim) |
| Marker | Draggable pin |
| Lat/Lng fields | Editable number inputs (sync with map) |

#### 4.2.2 PanoramaPreview

Preview panorama 360° inline bằng Pannellum.

```
┌─────────────────────────────────────────────────────┐
│  Preview Panorama                          [⛶ Full] │
│  ┌─────────────────────────────────────────────┐    │
│  │                                             │    │
│  │         PANNELLUM VIEWER (preview)          │    │
│  │         Interactive 360° view               │    │
│  │                                             │    │
│  └─────────────────────────────────────────────┘    │
│  Yaw: [80]  Pitch: [10]  Hfov: [120]  [📸 Capture]│
│                                                     │
│  [Upload panorama]  [Change]  [Remove]              │
└─────────────────────────────────────────────────────┘
```

| Property | Value |
|----------|-------|
| Height | 300px (expandable to 500px) |
| Pannellum config | showControls: true, autoLoad: true |
| Capture | Save current yaw/pitch/hfov as defaults |
| Upload | Drag & drop or click to upload |

#### 4.2.3 HotspotEditor

Đặt vị trí hotspot trên overview panorama.

```
┌─────────────────────────────────────────────────────┐
│  Đặt vị trí Hotspot trên Overview                   │
│  ┌─────────────────────────────────────────────┐    │
│  │                                             │    │
│  │   PANNELLUM VIEWER (overview panorama)      │    │
│  │   Click = đặt hotspot position              │    │
│  │              ✚ (crosshair cursor)           │    │
│  │                                             │    │
│  └─────────────────────────────────────────────┘    │
│  Hotspot Pitch: [-10.00]  Hotspot Yaw: [40.00]     │
│  [🎯 Click trên panorama để đặt vị trí]            │
└─────────────────────────────────────────────────────┘
```

| Property | Value |
|----------|-------|
| Mode | Click on overview panorama to set hotspot position |
| Output | `hotspot_pitch` + `hotspot_yaw` values |
| Visual | Red crosshair / pin at selected position |
| Preview | Show marker at current position in realtime |

#### 4.2.4 IconPicker (Lucide)

Chọn Lucide icon cho category.

```
┌─────────────────────────────────────────────────────┐
│  Chọn Icon                                          │
│  [🔍 Tìm icon...                              ]    │
│  ┌──────────────────────────────────────────────┐   │
│  │ 🏠 home   🏨 hotel   🏛 landmark  🗺 map    │   │
│  │ 🌊 waves  ⛪ church  🍴 utensils  🌲 trees  │   │
│  │ 🌐 globe  📍 map-pin 🕐 clock    👁 eye    │   │
│  │ ... (scrollable grid)                        │   │
│  └──────────────────────────────────────────────┘   │
│  Selected: waves                                    │
└─────────────────────────────────────────────────────┘
```

| Property | Value |
|----------|-------|
| Layout | Popover / Dialog |
| Grid | 8 columns, 40×40px cells |
| Search | Filter by icon name |
| Selected | Highlight + show name below |
| Common icons | Show tourism-related first |

#### 4.2.5 ColorPicker

Chọn màu hex cho category.

```
┌────────────────────────────┐
│  Chọn Màu                 │
│  ┌────────────────────┐   │
│  │  Color wheel /      │   │
│  │  react-colorful     │   │
│  └────────────────────┘   │
│  ┌──┐┌──┐┌──┐┌──┐┌──┐┌──┐│
│  │🟢││🟡││🩷││🔵││🟣││🔴││  ← Preset swatches
│  └──┘└──┘└──┘└──┘└──┘└──┘│
│  Hex: [#22D3A7]           │
└────────────────────────────┘
```

#### 4.2.6 EmojiPicker

Chọn emoji cho POI marker.

```
┌────────────────────────────┐
│  Chọn Emoji                │
│  [🔍 Tìm emoji...]        │
│  Thường dùng:              │
│  🏖️ 🌊 ⛰️ 🏛️ 🦀 ✝️ 🏨 🐢 │
│  🌅 🎣 🏄 🚣 🎪 🎭 🛕 🌺   │
│  ────────────────────────  │
│  Tất cả:                   │
│  [scrollable emoji grid]   │
└────────────────────────────┘
```

#### 4.2.7 DataTable

Bảng dữ liệu tái sử dụng với sort, filter, pagination.

```
┌────────────────────────────────────────────────────────────┐
│ ┌─────────────────────┐  ┌──────┐  ┌──────┐  ┌─────────┐ │
│ │ 🔍 Tìm kiếm...     │  │ Tỉnh ▼│  │ DM ▼ │  │+ Thêm   │ │
│ └─────────────────────┘  └──────┘  └──────┘  └─────────┘ │
│ ┌──────────────────────────────────────────────────────┐  │
│ │ ☐ │ Tên         │ Danh mục │ Tỉnh    │ TT │ Thao tác│  │
│ ├───┼─────────────┼──────────┼─────────┼────┼─────────┤  │
│ │ ☐ │ 🏖 Bãi Trước│ 🟢 Biển │ BR-VT  │ ✅ │ ✏️ 🗑   │  │
│ │ ☐ │ 🌊 Bãi Sau  │ 🟢 Biển │ BR-VT  │ ✅ │ ✏️ 🗑   │  │
│ │ ☐ │ ✝️ T.Chúa KT│ 🟡 T.linh│ BR-VT  │ ✅ │ ✏️ 🗑   │  │
│ │ ☐ │ ⛰️ Núi Dinh │ 🔵 T.nhiên│ BR-VT │ ⬜ │ ✏️ 🗑   │  │
│ └──────────────────────────────────────────────────────┘  │
│  ☐ 0 selected  [Bulk: Publish | Delete]                   │
│  Showing 1-10 of 45   [◀] 1 2 3 4 5 [▶]                  │
└────────────────────────────────────────────────────────────┘
```

| Feature | Mô tả |
|---------|--------|
| Search | Real-time filter by name/addr |
| Column sort | Click header to sort asc/desc |
| Filters | Province dropdown, Category dropdown, Status toggle |
| Checkbox | Select rows for bulk actions |
| Bulk actions | Publish/Unpublish, Delete selected |
| Pagination | 10/20/50 per page |
| Row actions | Edit (→ form), Delete (confirm dialog) |
| Empty state | Illustration + "Chưa có dữ liệu" |
| Loading | Skeleton rows |

---

## 5. Page Specifications

### 5.1 Dashboard (`/admin`)

```
┌─────────────────────────────────────────────────────────────┐
│  Dashboard                                                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ 📍 45    │ │ 📂 6     │ │ 🏛 3     │ │ 👁 12.5K │       │
│  │ Điểm đến │ │ Danh mục │ │ Tỉnh/TP  │ │ Lượt xem │       │
│  │ +5 tuần  │ │          │ │ 2 publish│ │ +23% ↑   │       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
│                                                              │
│  ┌────────────────────────┐  ┌───────────────────────┐      │
│  │ 📈 Lượt xem 30 ngày   │  │ 🏆 Top 5 Điểm Đến    │      │
│  │                        │  │                       │      │
│  │  [Line chart]          │  │  1. Bãi Trước  2.3K   │      │
│  │                        │  │  2. Côn Đảo    1.8K   │      │
│  │                        │  │  3. Bãi Sau    1.5K   │      │
│  │                        │  │  4. Hồ Tràm    1.2K   │      │
│  │                        │  │  5. Bạch Dinh  0.9K   │      │
│  └────────────────────────┘  └───────────────────────┘      │
│                                                              │
│  ┌────────────────────────┐  ┌───────────────────────┐      │
│  │ 📱 Thiết bị           │  │ 🕐 Hoạt động gần đây  │      │
│  │                        │  │                       │      │
│  │  [Donut chart]         │  │  • Admin đã sửa POI.. │      │
│  │  Mobile 65%            │  │  • Thêm 2 POI mới     │      │
│  │  Desktop 30%           │  │  • Publish tỉnh BRVT  │      │
│  │  Tablet 5%             │  │                       │      │
│  └────────────────────────┘  └───────────────────────┘      │
│                                                              │
│  Quick Actions                                               │
│  [+ Thêm Điểm Đến]  [+ Thêm Tỉnh]  [📤 Export JSON]       │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 Province List (`/admin/provinces`)

```
┌────────────────────────────────────────────────────────────┐
│  Quản Lý Tỉnh/Thành                         [+ Thêm Tỉnh]│
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Tên              │ POIs │ Published │ Updated │ ⚙    │  │
│  ├──────────────────┼──────┼───────────┼─────────┼──────┤  │
│  │ Bà Rịa—Vũng Tàu │  8   │ ✅ Live   │ 2h ago  │ ✏️🗑 │  │
│  │ Đà Nẵng          │  12  │ ⬜ Draft  │ 1d ago  │ ✏️🗑 │  │
│  │ Phú Quốc         │  6   │ ⬜ Draft  │ 3d ago  │ ✏️🗑 │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

### 5.3 Province Edit (`/admin/provinces/:id`)

```
┌────────────────────────────────────────────────────────────┐
│  ← Tỉnh/Thành  ›  Bà Rịa — Vũng Tàu                     │
│                                                            │
│  ┌─ Tab: [Thông tin] [Panorama] [Bản đồ] ────────────┐   │
│  │                                                     │   │
│  │  Tab 1: Thông tin                                   │   │
│  │  Tên:        [Bà Rịa — Vũng Tàu          ]         │   │
│  │  Slug:       [ba-ria-vung-tau             ] 🔗      │   │
│  │  Mô tả:     [                             ]         │   │
│  │              [  Textarea...               ]         │   │
│  │  Trạng thái: [🔵 Published] toggle                  │   │
│  │                                                     │   │
│  │  Tab 2: Panorama Overview                           │   │
│  │  ┌────────────────────────────────────────┐         │   │
│  │  │  PANNELLUM PREVIEW                     │         │   │
│  │  │  (overview panorama 360°)              │         │   │
│  │  └────────────────────────────────────────┘         │   │
│  │  [Upload Panorama] [Change]                         │   │
│  │  Yaw: [80]  Pitch: [10]  Hfov: [120]  Auto: [0.2] │   │
│  │  [📸 Capture Current View]                          │   │
│  │                                                     │   │
│  │  Tab 3: Cấu hình Bản đồ                            │   │
│  │  ┌────────────────────────────────────────┐         │   │
│  │  │  LEAFLET MAP (preview)                 │         │   │
│  │  │  Click to set center                   │         │   │
│  │  └────────────────────────────────────────┘         │   │
│  │  Center Lat: [10.40]  Lng: [107.15]                │   │
│  │  Default Zoom: [9]                                  │   │
│  │  Tile: [CartoDB Voyager ▼]                          │   │
│  │  CSS Filter: [brightness(.78) contrast(1.15)...]    │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                            │
│  [💾 Lưu]  [📤 Publish]  [🗑 Xoá]                        │
└────────────────────────────────────────────────────────────┘
```

### 5.4 POI List (`/admin/pois`)

Xem mô tả DataTable component ở section 4.2.7. POI list là trang sử dụng nhiều nhất — cần tối ưu UX.

### 5.5 POI Edit (`/admin/pois/:id`)

```
┌────────────────────────────────────────────────────────────┐
│  ← Điểm đến  ›  Bãi Trước                                │
│                                                            │
│  ┌─ Tab: [Cơ bản] [Vị trí] [Panorama] [Hotspot] [Gallery]│
│  │                                                     │   │
│  │  ═══ Tab 1: THÔNG TIN CƠ BẢN ═══                   │   │
│  │  Tên:        [Bãi Trước                   ]         │   │
│  │  Slug:       [bai-truoc                   ] 🔗      │   │
│  │  Tỉnh:       [Bà Rịa — Vũng Tàu  ▼]               │   │
│  │  Danh mục:   [🟢 Biển             ▼]               │   │
│  │  Emoji:      [🏖️] [Pick]                            │   │
│  │  Địa chỉ:    [Tp. Vũng Tàu               ]         │   │
│  │  Mô tả:     [                             ]         │   │
│  │              [  Textarea...               ]         │   │
│  │  Giờ mở cửa: [Cả ngày                    ]         │   │
│  │  Khoảng cách: [0 km từ trung tâm         ]         │   │
│  │  Tags:       [Biển ✕] [Miễn phí ✕] [+ Thêm tag]   │   │
│  │  Published:  [🔵 toggle]                            │   │
│  │                                                     │   │
│  │  ═══ Tab 2: VỊ TRÍ TRÊN BẢN ĐỒ ═══                │   │
│  │  ┌────────────────────────────────────────┐         │   │
│  │  │  MAP PICKER (Leaflet)                  │         │   │
│  │  │  Click to place marker                 │         │   │
│  │  │              📍                         │         │   │
│  │  └────────────────────────────────────────┘         │   │
│  │  Lat: [10.346000]  Lng: [107.072300]                │   │
│  │  [🔍 Search: Tìm địa điểm...]                      │   │
│  │                                                     │   │
│  │  ═══ Tab 3: PANORAMA 360° ═══                       │   │
│  │  ┌────────────────────────────────────────┐         │   │
│  │  │  PANNELLUM PREVIEW                     │         │   │
│  │  │  (POI panorama 360°)                   │         │   │
│  │  └────────────────────────────────────────┘         │   │
│  │  [Upload Panorama] [Change] [Remove]                │   │
│  │  Thumbnail: [Drop image / Click upload]             │   │
│  │  Scene Yaw: [30]  Pitch: [0]  Hfov: [105]          │   │
│  │  [📸 Capture Current View as Defaults]              │   │
│  │                                                     │   │
│  │  ═══ Tab 4: VỊ TRÍ HOTSPOT ═══                     │   │
│  │  ┌────────────────────────────────────────┐         │   │
│  │  │  PANNELLUM VIEWER (overview panorama)  │         │   │
│  │  │  Click = đặt hotspot cho POI này       │         │   │
│  │  │              ✚ (crosshair)             │         │   │
│  │  └────────────────────────────────────────┘         │   │
│  │  Hotspot Pitch: [-10.00]  Yaw: [40.00]             │   │
│  │                                                     │   │
│  │  ═══ Tab 5: GALLERY ẢNH ═══                        │   │
│  │  [Drop images here or click to upload]              │   │
│  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐              │   │
│  │  │ img1 │ │ img2 │ │ img3 │ │  +   │              │   │
│  │  │[✏️🗑]│ │[✏️🗑]│ │[✏️🗑]│ │      │              │   │
│  │  └──────┘ └──────┘ └──────┘ └──────┘              │   │
│  │  (drag to reorder)                                  │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                            │
│  [💾 Lưu]  [Preview ↗]  [🗑 Xoá]                         │
└────────────────────────────────────────────────────────────┘
```

### 5.6 Category List & Edit (`/admin/categories`)

```
┌────────────────────────────────────────────────────────────┐
│  Quản Lý Danh Mục                        [+ Thêm Danh Mục]│
│  ┌──────────────────────────────────────────────────────┐  │
│  │ ≡ │ Icon │ Tên        │ Slug    │ Màu   │ POIs│ ⚙   │  │
│  ├───┼──────┼────────────┼─────────┼───────┼─────┼─────┤  │
│  │ ≡ │ 🌊   │ Biển       │ beach   │ 🟢    │ 2   │ ✏️🗑│  │
│  │ ≡ │ ⛪   │ Tâm linh   │ temple  │ 🟡    │ 1   │ ✏️🗑│  │
│  │ ≡ │ 🍴   │ Ẩm thực   │ food    │ 🩷    │ 1   │ ✏️🗑│  │
│  │ ≡ │ 🌲   │ Thiên nhiên│ nature  │ 🔵    │ 2   │ ✏️🗑│  │
│  │ ≡ │ 🏛   │ Di tích    │ history │ 🟣    │ 1   │ ✏️🗑│  │
│  │ ≡ │ 🏨   │ Nghỉ dưỡng │ resort  │ 🔴    │ 1   │ ✏️🗑│  │
│  └──────────────────────────────────────────────────────┘  │
│  (≡ = drag handle for reorder)                             │
│                                                            │
│  ── Inline Edit (click row) ──                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Tên:  [Biển        ]  Icon: [waves] [Pick]           │  │
│  │ Slug: [beach       ]  Màu:  [#22D3A7] [🎨 Pick]     │  │
│  │ Active: [🔵 toggle]                  [Lưu] [Huỷ]    │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

### 5.7 Tags Management (`/admin/tags`)

```
┌────────────────────────────────────────────────────────────┐
│  Quản Lý Tags                                [+ Thêm Tag] │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ [🔍 Tìm tag...                                    ] │  │
│  │                                                      │  │
│  │ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐│  │
│  │ │ Biển  ✕  │ │ Miễn phí │ │ Hoàng hôn│ │ Lịch sử  ││  │
│  │ │   5 POIs │ │   3 POIs │ │   2 POIs │ │   2 POIs ││  │
│  │ └──────────┘ └──────────┘ └──────────┘ └──────────┘│  │
│  │ ┌──────────┐ ┌──────────┐ ┌──────────┐             │  │
│  │ │ Gia đình │ │ Check-in │ │ Hải sản  │             │  │
│  │ │   4 POIs │ │   1 POI  │ │   3 POIs │             │  │
│  │ └──────────┘ └──────────┘ └──────────┘             │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  Click tag to edit, ✕ to delete                            │
└────────────────────────────────────────────────────────────┘
```

### 5.8 Map Config (`/admin/map-config`)

```
┌────────────────────────────────────────────────────────────┐
│  Cấu Hình Bản Đồ                                          │
│  Tỉnh: [Bà Rịa — Vũng Tàu  ▼]                           │
│                                                            │
│  ┌─────────────────────────────┐  ┌─────────────────────┐ │
│  │                             │  │ Center Lat: [10.40] │ │
│  │  LEAFLET MAP (large)       │  │ Center Lng: [107.15]│ │
│  │  Shows all POI markers     │  │ Default Zoom: [9]   │ │
│  │  Editable center point     │  │ Min Zoom: [5]       │ │
│  │                             │  │ Max Zoom: [18]      │ │
│  │  Click to set center       │  │                     │ │
│  │                             │  │ Tile Provider:      │ │
│  │                             │  │ [CartoDB Voyager ▼] │ │
│  │                             │  │                     │ │
│  └─────────────────────────────┘  │ CSS Filter:         │ │
│                                    │ [brightness(.78)..] │ │
│  ┌─────────────────────────────┐  │                     │ │
│  │  Preview (with filter)      │  │ [💾 Lưu]           │ │
│  │  (dark mode preview)        │  │                     │ │
│  └─────────────────────────────┘  └─────────────────────┘ │
└────────────────────────────────────────────────────────────┘
```

---

## 6. Responsive Breakpoints

### 6.1 Breakpoints

| Breakpoint | Width | Sidebar | Layout |
|-----------|-------|---------|--------|
| Desktop XL | ≥ 1440px | Expanded (260px) | Full layout |
| Desktop | ≥ 1024px | Expanded (260px) | Full layout |
| Tablet | ≥ 768px | Collapsed (72px), overlay on click | Compact |
| Mobile | < 768px | Hidden, hamburger menu | Stack |

### 6.2 Mobile Adaptations

| Component | Desktop | Mobile |
|-----------|---------|--------|
| Sidebar | Fixed left, always visible | Overlay, hamburger toggle |
| DataTable | Full columns | Scroll horizontal, hide less important |
| MapPicker | 50% width beside form | Full width, stacked above form |
| PanoramaPreview | 50% width beside form | Full width, stacked |
| Form columns | 2 columns | 1 column stack |
| Quick stats | 4 columns | 2 columns |
| Charts | Side by side | Stacked |

---

## 7. Animations & Transitions

### 7.1 Transition Defaults

| Element | Property | Duration | Easing |
|---------|----------|----------|--------|
| Sidebar collapse | width | 200ms | ease-out |
| Sidebar mobile | transform (translateX) | 250ms | ease-out |
| Page transition | opacity | 150ms | ease-in-out |
| Dialog open | opacity, scale | 200ms | ease-out |
| Toast enter | translateX, opacity | 300ms | ease-out |
| Toast exit | translateX, opacity | 200ms | ease-in |
| Table row hover | background | 150ms | linear |
| Button hover | all | 150ms | linear |
| Tab switch | border, color | 200ms | ease-out |
| Drag & drop | transform | 200ms | ease-out |

### 7.2 Loading States

| State | Component | Mô tả |
|-------|-----------|--------|
| Page loading | Skeleton | Full page skeleton with shimmer |
| Table loading | Skeleton rows | 5 skeleton rows |
| Form loading | Spinner | Button spinner on submit |
| Upload | Progress bar | Linear progress with % |
| Panorama loading | Overlay spinner | Over Pannellum container |
| Map loading | Skeleton + spinner | Over Leaflet container |

---

## 8. Dark Mode

### 8.1 Dark Mode Token Overrides

```css
[data-theme="dark"] {
  --bg-page:    var(--slate-900);
  --bg-card:    var(--slate-800);
  --bg-header:  var(--slate-800);
  --bg-sidebar: #0A1F1E;            /* Darker teal */
  
  --text-primary:   var(--slate-100);
  --text-secondary: var(--slate-400);
  --text-muted:     var(--slate-500);
  
  --border:     var(--slate-700);
  --shadow-sm:  0 1px 2px rgba(0,0,0,.3);
  --shadow-md:  0 4px 6px rgba(0,0,0,.4);
}
```

### 8.2 Dark Mode Scope

- Dashboard hỗ trợ toggle dark/light mode
- Default: Light mode
- Preference saved in localStorage + system `prefers-color-scheme`
- Sidebar luôn dark (teal-900) cả 2 mode

---

*End of Document*
