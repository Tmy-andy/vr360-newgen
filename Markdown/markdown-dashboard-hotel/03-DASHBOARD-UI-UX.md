# 03 — DASHBOARD UI/UX SPECIFICATION
## Admin Dashboard — Hotel CMS Platform

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** 01-DASHBOARD-ARCHITECTURE · 02-DASHBOARD-DATABASE-API · 04~05

---

## Mục Lục

1. [Design Principles](#1-design-principles)
2. [Design Tokens](#2-design-tokens)
3. [Layout Architecture](#3-layout-architecture)
4. [Component Specifications](#4-component-specifications)
5. [Page-by-Page UI Specs](#5-page-by-page-ui-specs)
6. [Booking Configuration UI ★](#6-booking-configuration-ui)
7. [360° Hotspot Editor UI](#7-360-hotspot-editor-ui)
8. [Responsive & Tablet Support](#8-responsive--tablet-support)
9. [Sequence Diagrams](#9-sequence-diagrams)

---

## 1. Design Principles

### 1.1 Nguyên Tắc Admin Dashboard

| # | Nguyên tắc | Giải thích |
|---|-----------|-----------|
| 1 | **Efficiency First** | Admin cần thao tác nhanh, ít click, keyboard shortcuts |
| 2 | **Clear Hierarchy** | Sidebar → Page → Section → Form/Table rõ ràng |
| 3 | **Consistent Patterns** | Mọi CRUD page follow cùng 1 pattern |
| 4 | **Visual Feedback** | Toast, loading states, success/error rõ ràng |
| 5 | **Data Dense** | Bảng hiển thị nhiều thông tin, compact layout |
| 6 | **Light Theme** | Professional, sáng, dễ đọc khi làm việc lâu |

---

## 2. Design Tokens

### 2.1 CSS Variables (Admin Theme)

```css
:root {
  /* Colors — Light Theme */
  --bg-primary: #ffffff;
  --bg-secondary: #f8fafc;
  --bg-tertiary: #f1f5f9;
  --bg-sidebar: #0f172a;
  --bg-sidebar-hover: #1e293b;
  --bg-sidebar-active: rgba(201, 168, 97, 0.15);

  /* Brand */
  --brand-primary: #1a365d;          /* Hotel primary color */
  --brand-accent: #c6a052;           /* Gold accent */
  --brand-accent-light: #f5eedd;

  /* Text */
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #94a3b8;
  --text-sidebar: #cbd5e1;
  --text-sidebar-active: #c6a052;

  /* Status */
  --status-success: #22c55e;
  --status-warning: #f59e0b;
  --status-error: #ef4444;
  --status-info: #3b82f6;

  /* Borders */
  --border: #e2e8f0;
  --border-focus: #c6a052;

  /* Typography */
  --font-sans: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.07);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.1);

  /* Radius */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;

  /* Sizing */
  --sidebar-width: 260px;
  --sidebar-collapsed: 64px;
  --topbar-height: 56px;
}
```

---

## 3. Layout Architecture

### 3.1 Admin Shell Layout

```
┌──────────────────────────────────────────────────────────────┐
│ ADMIN DASHBOARD SHELL                                         │
│                                                                │
│ ┌─────────┬──────────────────────────────────────────────────┐│
│ │         │ TOPBAR (h:56px, z:40)                            ││
│ │         │ [☰ Collapse] [Search...        ] [🔔] [Avatar ▼]││
│ │         ├──────────────────────────────────────────────────┤│
│ │ SIDEBAR │                                                  ││
│ │(w:260px)│ PAGE CONTENT                                     ││
│ │         │                                                  ││
│ │ [Logo]  │ ┌─ BREADCRUMB ─────────────────────────────────┐││
│ │         │ │ Dashboard / Rooms / Deluxe Ocean              │││
│ │ ─────── │ └───────────────────────────────────────────────┘││
│ │         │                                                  ││
│ │ 📊 Dash │ ┌─ PAGE HEADER ────────────────────────────────┐││
│ │ 🛏 Rooms│ │ Room Detail          [Save Draft] [Publish]  │││
│ │ 🍽 Svc  │ └───────────────────────────────────────────────┘││
│ │ 🎨 Scene│                                                  ││
│ │ 🎫 Promo│ ┌─ CONTENT ───────────────────────────────────┐││
│ │ 📰 Posts│ │                                               │││
│ │ 🖼 Gall │ │  (Forms, Tables, Editors, Charts)            │││
│ │ 📋 Menu │ │                                               │││
│ │         │ │                                               │││
│ │ ─────── │ │                                               │││
│ │ 📅 Book │ │                                               │││
│ │ 🌐 i18n │ │                                               │││
│ │ ⚙ Sett. │ │                                               │││
│ │ 👥 Users│ └───────────────────────────────────────────────┘││
│ │ 📈 Analy│                                                  ││
│ └─────────┴──────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────────┘
```

### 3.2 Sidebar Navigation

```
┌──────────────────────┐
│   [Hotel Logo]        │
│   Boton Blue Hotel    │
│                       │
│ ═══════════════════   │
│                       │
│ MAIN                  │
│ ├── 📊 Dashboard      │  ← /admin
│ │                     │
│ CONTENT               │
│ ├── 🛏️ Rooms          │  ← /admin/rooms
│ ├── 🍽️ Services       │  ← /admin/services
│ ├── 🎨 Scenes & 360°  │  ← /admin/scenes
│ ├── 🎫 Promotions     │  ← /admin/promotions
│ ├── 📰 Posts / News   │  ← /admin/posts
│ ├── 🖼️ Gallery        │  ← /admin/gallery
│ ├── 📋 Menus          │  ← /admin/menus
│ │                     │
│ BOOKING ★             │
│ ├── 📅 Configuration  │  ← /admin/booking
│ ├── 📝 Requests       │  ← /admin/booking/requests
│ │                     │
│ SYSTEM                │
│ ├── 🌐 Translations   │  ← /admin/translations
│ ├── ⚙️ Settings       │  ← /admin/settings
│ ├── 👥 Users          │  ← /admin/users (super_admin)
│ ├── 📈 Analytics      │  ← /admin/analytics
│ └── 📋 Audit Log      │  ← /admin/audit-logs
│                       │
│ ═══════════════════   │
│ [Collapse ◀]          │
└──────────────────────┘

STATES:
- Default: expanded (260px)
- Collapsed: icon-only (64px), tooltip on hover
- Mobile (<768px): overlay drawer (z:50)
- Active item: gold left border + bg highlight
- Hover: lighter bg
- Section headers: uppercase, 11px, text-muted
```

### 3.3 Topbar

```
┌──────────────────────────────────────────────────────────────┐
│ [☰]  [🔍 Search rooms, services, posts...              ]    │
│                                                              │
│                    [🔔 3]  [EN ▼]  [👤 Admin Name ▼]        │
└──────────────────────────────────────────────────────────────┘

Components:
- Hamburger: toggle sidebar collapse / mobile drawer
- Search: cmd+K shortcut, searches across all entities
- Notifications: booking requests, system alerts
- Language: switch admin UI language
- Avatar dropdown: Profile, Theme toggle, Logout
```

---

## 4. Component Specifications

### 4.1 Data Table (Reusable)

```
┌────────────────────────────────────────────────────────────────┐
│ PAGE HEADER                                                     │
│ Rooms                                    [+ Add Room]          │
│                                                                 │
│ ┌─ TOOLBAR ──────────────────────────────────────────────────┐ │
│ │ [🔍 Search...]  [Status ▼] [Type ▼]  [Sort ▼]  [Export]   │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ TABLE ────────────────────────────────────────────────────┐ │
│ │ ☐  Thumbnail  Name           Type    Price    Status  ···  │ │
│ │ ── ──────── ─────────────── ─────── ──────── ──────── ──── │ │
│ │ ☐  [📷]      Deluxe Twin    Deluxe  2.5M VND ● Pub   [⋮] │ │
│ │ ☐  [📷]      Superior Twin  Sup.    1.8M VND ● Pub   [⋮] │ │
│ │ ☐  [📷]      Pacific Double Pacific 3.8M VND ○ Draft [⋮] │ │
│ │ ☐  [📷]      Suite Ocean    Suite   5.5M VND ● Pub   [⋮] │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ PAGINATION ───────────────────────────────────────────────┐ │
│ │ Showing 1-4 of 4 rooms            [◀ Prev] [1] [Next ▶]   │ │
│ └────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘

Row actions [⋮] dropdown:
- Edit
- Duplicate
- View on website (new tab)
- Publish / Unpublish
- Delete (with confirmation dialog)

Table features (TanStack Table):
- Sortable columns (click header)
- Multi-select rows (bulk actions)
- Column visibility toggle
- Row drag-to-reorder (sort_order)
- Inline status toggle
```

### 4.2 Form Layout (Create / Edit)

```
┌────────────────────────────────────────────────────────────────┐
│ BREADCRUMB: Dashboard / Rooms / Edit: Deluxe Twin              │
│                                                                 │
│ ┌─ PAGE HEADER ──────────────────────────────────────────────┐ │
│ │ Deluxe Twin Room           [Save Draft] [Preview] [Publish]│ │
│ │ Last saved: 5 min ago · Draft                               │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ TABS ─────────────────────────────────────────────────────┐ │
│ │ [General] [Details] [Scenes] [Gallery] [Booking] [SEO]     │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ TAB: GENERAL ─────────────────────────────────────────────┐ │
│ │                                                              │ │
│ │  ┌─ LEFT (2/3) ───────────┐ ┌─ RIGHT (1/3) ──────────────┐│ │
│ │  │                        │ │                              ││ │
│ │  │ Room Name *            │ │ Status                       ││ │
│ │  │ [Deluxe Twin Room   ]  │ │ ○ Draft  ● Published        ││ │
│ │  │                        │ │                              ││ │
│ │  │ Slug                   │ │ Room Type                    ││ │
│ │  │ [deluxe-twin    ] 🔗   │ │ [Deluxe         ▼]          ││ │
│ │  │                        │ │                              ││ │
│ │  │ Short Description      │ │ Thumbnail                    ││ │
│ │  │ [Ocean view with... ]  │ │ ┌──────────────┐            ││ │
│ │  │                        │ │ │  [Drop image │            ││ │
│ │  │ Full Description       │ │ │   or click]  │            ││ │
│ │  │ ┌──────────────────┐   │ │ └──────────────┘            ││ │
│ │  │ │ Rich Text Editor │   │ │                              ││ │
│ │  │ │ (Tiptap)         │   │ │ Sort Order                   ││ │
│ │  │ │                  │   │ │ [0]                          ││ │
│ │  │ │                  │   │ │                              ││ │
│ │  │ └──────────────────┘   │ │                              ││ │
│ │  │                        │ │                              ││ │
│ │  └────────────────────────┘ └──────────────────────────────┘│ │
│ │                                                              │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ TAB: DETAILS ─────────────────────────────────────────────┐ │
│ │ Area (m²): [35   ]   Max Adults: [2]   Max Children: [1]  │ │
│ │ Bed Type:  [Twin ▼]  View Type:  [Ocean ▼]                │ │
│ │ Price From: [2500000] Currency: [VND ▼]                    │ │
│ │ Highlights: [+ Add] [Panorama ocean view ×] [Balcony ×]   │ │
│ │ Amenities:  [✓] Ocean View [✓] Balcony [✓] WiFi [✓] AC   │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ TAB: BOOKING ★ ───────────────────────────────────────────┐ │
│ │ → Xem mục 6: Booking Configuration UI                       │ │
│ └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 4.3 Image Upload Component

```
┌───────────────────────────────────────────────┐
│  Gallery Images (4 uploaded)                   │
│                                                │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────────┐│
│  │ 📷  │ │ 📷  │ │ 📷  │ │ 📷  │ │ + Drop  ││
│  │     │ │     │ │     │ │     │ │  images  ││
│  │ [×] │ │ [×] │ │ [×] │ │ [×] │ │  here   ││
│  └──┬──┘ └─────┘ └─────┘ └─────┘ └─────────┘│
│     │ Drag to reorder                          │
│                                                │
│  Upload: JPEG, PNG, WebP · Max 10MB each       │
│  Auto-generates: original, 1920w, 800w, thumb  │
└───────────────────────────────────────────────┘

Features:
- Drag & drop upload (react-dropzone)
- Multi-file upload
- Progress bar per file
- Preview thumbnails
- Drag-to-reorder (sort_order)
- Click [×] to delete (with confirm)
- Alt text input per image
- Caption input per image
```

### 4.4 Toast Notifications

```
┌──────────────────────────────────────┐
│ ✅ Room published successfully!       │  ← Success (green)
└──────────────────────────────────────┘

┌──────────────────────────────────────┐
│ ❌ Failed to save. Please try again. │  ← Error (red)
└──────────────────────────────────────┘

┌──────────────────────────────────────┐
│ ⚠️ Unsaved changes will be lost.     │  ← Warning (yellow)
└──────────────────────────────────────┘

┌──────────────────────────────────────┐
│ 📅 New booking request from          │  ← Info (blue)
│    Nguyễn Văn A · Deluxe Twin        │
│    [View Request →]                   │
└──────────────────────────────────────┘

Position: top-right
Duration: 4s (auto-dismiss), permanent for errors
Library: Sonner
```

---

## 5. Page-by-Page UI Specs

### 5.1 Dashboard Overview (/admin)

```
┌────────────────────────────────────────────────────────────────┐
│ Dashboard                                    Today: Mar 10, 2026│
│                                                                 │
│ ┌─ STATS CARDS ──────────────────────────────────────────────┐ │
│ │                                                              │ │
│ │ ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐   │ │
│ │ │ 📊 Views  │ │ 🔥 360°   │ │ 📅 Booking│ │ 📝 Content│   │ │
│ │ │           │ │  Interact │ │  Requests │ │           │   │ │
│ │ │ 12,450    │ │ 3,280     │ │ 47 new    │ │ 24 rooms  │   │ │
│ │ │ +15% ↑    │ │ +8% ↑     │ │ +23% ↑    │ │ 12 svc    │   │ │
│ │ └───────────┘ └───────────┘ └───────────┘ └───────────┘   │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ CHARTS ───────────────────────────────────────────────────┐ │
│ │ ┌─ Views Over Time (30d) ───┐ ┌─ Top Scenes ────────────┐ │ │
│ │ │ [Line chart — Recharts]   │ │ 1. Lobby        2,100    │ │ │
│ │ │ x: dates, y: page views  │ │ 2. Deluxe Bed   1,450    │ │ │
│ │ │                           │ │ 3. Pool         980      │ │ │
│ │ │                           │ │ 4. Dining       870      │ │ │
│ │ └───────────────────────────┘ │ 5. Suite Bed    650      │ │ │
│ │                               └──────────────────────────┘ │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ RECENT ACTIVITY ──────────────────────────────────────────┐ │
│ │ 🟢 Admin published "Deluxe Twin Room"         2 min ago    │ │
│ │ 📅 New booking request from Nguyễn Văn A       15 min ago   │ │
│ │ 📝 Editor updated "Spa Treatment Menu"          1 hour ago  │ │
│ │ 🖼 Admin uploaded 5 gallery images               3 hours ago│ │
│ │ 🔧 Super Admin updated hotel settings            Yesterday  │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ QUICK ACTIONS ────────────────────────────────────────────┐ │
│ │ [+ New Room] [+ New Service] [+ New Post] [📅 View Bookings]│ │
│ └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 5.2 Room List (/admin/rooms)

→ Dùng Data Table pattern (mục 4.1) với columns:
- Checkbox (select)
- Thumbnail (60×40px)
- Name (link to edit)
- Type (badge: Deluxe, Superior, Suite, etc.)
- Price (formatted)
- Scenes (count)
- Status (Published / Draft badge)
- Updated at
- Actions (⋮)

### 5.3 Room Edit (/admin/rooms/[id])

→ Dùng Form Layout pattern (mục 4.2) với tabs:
- **General:** Name, slug, short desc, full desc (rich editor), thumbnail
- **Details:** Area, occupancy, bed type, view, price, highlights, amenities
- **Scenes:** Scene mapping (drag-to-reorder), link to hotspot editor
- **Gallery:** Image upload/manage (drag-to-reorder)
- **Booking:** ★ Booking mode override (→ mục 6)
- **SEO:** Title, description, OG image, JSON-LD preview

### 5.4 Scene & Hotspot Editor (/admin/scenes/[id]/editor)

→ Chi tiết tại mục 7.

### 5.5 Analytics (/admin/analytics)

```
┌────────────────────────────────────────────────────────────────┐
│ Analytics                        [Date Range: Last 30 days ▼]  │
│                                                                 │
│ ┌─ TAB BAR ──────────────────────────────────────────────────┐ │
│ │ [Overview] [Scenes] [Hotspots] [Booking] [Export]          │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ OVERVIEW TAB ─────────────────────────────────────────────┐ │
│ │                                                              │ │
│ │ KPI Cards:                                                   │ │
│ │ [Total Views] [Unique Sessions] [Avg Duration] [Bounce Rate]│ │
│ │                                                              │ │
│ │ Charts:                                                      │ │
│ │ [Views over time — line]                                     │ │
│ │ [Top pages — bar]                                            │ │
│ │ [Device breakdown — donut]                                   │ │
│ │ [Referrer sources — horizontal bar]                          │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ BOOKING TAB ★ ────────────────────────────────────────────┐ │
│ │                                                              │ │
│ │ KPI Cards:                                                   │ │
│ │ [Total Requests] [Conversion Rate] [Avg Lead Time]          │ │
│ │                                                              │ │
│ │ Charts:                                                      │ │
│ │ [Booking requests over time — line]                          │ │
│ │ [Booking by room — bar]                                      │ │
│ │ [Booking by source (scene/page) — pie]                       │ │
│ │ [CTA click vs booking submit — funnel]                       │ │
│ │                                                              │ │
│ │ Table: Recent booking requests                               │ │
│ │ [Name] [Room] [Dates] [Status] [Source Scene]               │ │
│ └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

---

## 6. Booking Configuration UI ★

### 6.1 Global Booking Settings (/admin/booking)

```
┌────────────────────────────────────────────────────────────────┐
│ Booking Configuration                                           │
│                                                                 │
│ ┌─ MODE SELECTION ───────────────────────────────────────────┐ │
│ │                                                              │ │
│ │  How should the "Book Now" button work?                      │ │
│ │                                                              │ │
│ │  ┌─ OPTION 1 ──────────────────────────────────────────────┐│ │
│ │  │ ◉ External Link                                         ││ │
│ │  │                                                          ││ │
│ │  │ Redirect guests to an external booking/scheduling page   ││ │
│ │  │ (e.g., Book Direct Online, Booking.com, custom URL)      ││ │
│ │  │                                                          ││ │
│ │  │ Default URL:                                             ││ │
│ │  │ [https://book-directonline.com/properties/botonblue  ]   ││ │
│ │  │                                                          ││ │
│ │  │ Open in: ◉ New tab (_blank)  ○ Same tab (_self)          ││ │
│ │  └──────────────────────────────────────────────────────────┘│ │
│ │                                                              │ │
│ │  ┌─ OPTION 2 ──────────────────────────────────────────────┐│ │
│ │  │ ○ In-Page Booking Modal                                  ││ │
│ │  │                                                          ││ │
│ │  │ Show a booking form directly on your website.            ││ │
│ │  │ Guests fill in their info, and you receive the request   ││ │
│ │  │ via email and in this dashboard.                          ││ │
│ │  │                                                          ││ │
│ │  │ ┌─ Modal Preview ─────────────────────────────────────┐ ││ │
│ │  │ │                                                      │ ││ │
│ │  │ │        ★ Book Your Stay                              │ ││ │
│ │  │ │        Fill in your details and we'll                │ ││ │
│ │  │ │        confirm your reservation                      │ ││ │
│ │  │ │                                                      │ ││ │
│ │  │ │        [Full Name          ]                         │ ││ │
│ │  │ │        [Email              ]                         │ ││ │
│ │  │ │        [Phone              ]                         │ ││ │
│ │  │ │        [Check-in] [Check-out]                        │ ││ │
│ │  │ │        [Adults] [Children   ]                        │ ││ │
│ │  │ │        [Special Requests    ]                        │ ││ │
│ │  │ │                                                      │ ││ │
│ │  │ │        [Send Booking Request]                        │ ││ │
│ │  │ │                                                      │ ││ │
│ │  │ └──────────────────────────────────────────────────────┘ ││ │
│ │  └──────────────────────────────────────────────────────────┘│ │
│ │                                                              │ │
│ │                                              [Save Changes]  │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ MODAL SETTINGS (visible khi chọn "In-Page Modal") ───────┐ │
│ │                                                              │ │
│ │  Modal Title:     [Book Your Stay                         ]  │ │
│ │  Subtitle:        [Fill in your details and we'll...      ]  │ │
│ │  Button Label:    [Send Booking Request                   ]  │ │
│ │  Button Color:    [#C9A861] 🎨                               │ │
│ │  Success Message: [Thank you! We will contact you...      ]  │ │
│ │                                                              │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ FORM FIELDS BUILDER (visible khi chọn "In-Page Modal") ──┐ │
│ │                                                              │ │
│ │  Drag to reorder:                                            │ │
│ │                                                              │ │
│ │  ┌─ 1 ────────────────────────────────────────────────────┐ │ │
│ │  │ ≡  Full Name        [text]    [Required ✓]   [✏️] [🗑] │ │ │
│ │  └────────────────────────────────────────────────────────┘ │ │
│ │  ┌─ 2 ────────────────────────────────────────────────────┐ │ │
│ │  │ ≡  Email Address    [email]   [Required ✓]   [✏️] [🗑] │ │ │
│ │  └────────────────────────────────────────────────────────┘ │ │
│ │  ┌─ 3 ────────────────────────────────────────────────────┐ │ │
│ │  │ ≡  Phone Number     [tel]     [Required ✓]   [✏️] [🗑] │ │ │
│ │  └────────────────────────────────────────────────────────┘ │ │
│ │  ┌─ 4 ────────────────────────────────────────────────────┐ │ │
│ │  │ ≡  Check-in Date    [date]    [Required ✓]   [✏️] [🗑] │ │ │
│ │  └────────────────────────────────────────────────────────┘ │ │
│ │  ┌─ 5 ────────────────────────────────────────────────────┐ │ │
│ │  │ ≡  Check-out Date   [date]    [Required ✓]   [✏️] [🗑] │ │ │
│ │  └────────────────────────────────────────────────────────┘ │ │
│ │  ┌─ 6 ────────────────────────────────────────────────────┐ │ │
│ │  │ ≡  Adults           [number]  [Required ✓]   [✏️] [🗑] │ │ │
│ │  └────────────────────────────────────────────────────────┘ │ │
│ │  ┌─ 7 ────────────────────────────────────────────────────┐ │ │
│ │  │ ≡  Children         [number]  [Required ✗]   [✏️] [🗑] │ │ │
│ │  └────────────────────────────────────────────────────────┘ │ │
│ │  ┌─ 8 ────────────────────────────────────────────────────┐ │ │
│ │  │ ≡  Special Requests [textarea][Required ✗]   [✏️] [🗑] │ │ │
│ │  └────────────────────────────────────────────────────────┘ │ │
│ │                                                              │ │
│ │  [+ Add Custom Field]                                        │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ NOTIFICATION SETTINGS ────────────────────────────────────┐ │
│ │                                                              │ │
│ │  Receive booking notifications at:                           │ │
│ │  [sales@botonblue.com] [×]                                   │ │
│ │  [manager@botonblue.com] [×]                                 │ │
│ │  [+ Add email]                                               │ │
│ │                                                              │ │
│ │  Webhook URL (optional):                                     │ │
│ │  [https://hooks.slack.com/services/xxx           ]           │ │
│ │                                                              │ │
│ │  ☑ Send auto-reply confirmation to guests                    │ │
│ │  Subject: [Booking Request Received — Boton Blue Hotel    ]  │ │
│ │  Template: [Rich text editor for email template           ]  │ │
│ │                                                              │ │
│ │  [Send Test Email]  [Save Changes]                           │ │
│ └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 6.2 Room-Level Booking Override (/admin/rooms/[id] → Tab "Booking")

```
┌────────────────────────────────────────────────────────────────┐
│ TAB: BOOKING                                                    │
│                                                                 │
│ ┌─ BOOKING MODE ─────────────────────────────────────────────┐ │
│ │                                                              │ │
│ │  Booking mode for this room:                                 │ │
│ │                                                              │ │
│ │  ◉ Use global settings (currently: In-Page Modal)            │ │
│ │  ○ Override: External Link                                   │ │
│ │  ○ Override: In-Page Modal                                   │ │
│ │                                                              │ │
│ │  ─── If External Link selected: ───                          │ │
│ │  Booking URL for this room:                                  │ │
│ │  [https://book-directonline.com/properties/...?room=deluxe] │ │
│ │  (Leave empty to use global URL)                             │ │
│ │                                                              │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ CTA CUSTOMIZATION ───────────────────────────────────────┐ │
│ │                                                              │ │
│ │  Primary CTA Label:    [Book Now              ]              │ │
│ │  Secondary CTA Label:  [Check Availability    ]              │ │
│ │  Show price in CTA:    ☑ Yes                                 │ │
│ │                                                              │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│                                              [Save Changes]     │
└────────────────────────────────────────────────────────────────┘
```

### 6.3 Booking Requests List (/admin/booking/requests)

```
┌────────────────────────────────────────────────────────────────┐
│ Booking Requests                        [Export CSV] [Filter ▼]│
│                                                                 │
│ ┌─ STATUS TABS ──────────────────────────────────────────────┐ │
│ │ [All (47)] [New (12)] [Contacted (8)] [Confirmed (25)]     │ │
│ │ [Cancelled (2)]                                             │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ TABLE ────────────────────────────────────────────────────┐ │
│ │ Guest         Room           Check-in    Status    Created  │ │
│ │ ────────────  ─────────────  ─────────  ────────  ──────── │ │
│ │ Nguyễn Văn A  Deluxe Twin    Apr 15     🟢 New    2h ago   │ │
│ │ guest@e.com   Ocean View     → Apr 18                      │ │
│ │ +84 901...    2A, 1C                    [View →]           │ │
│ │ ──────────────────────────────────────────────────────────  │ │
│ │ John Smith    Suite Ocean    Apr 20     🟡 Cont.  1d ago   │ │
│ │ john@e.com    View           → Apr 23                      │ │
│ │ +1 555...     2A, 0C                    [View →]           │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ─── CLICK A ROW → DETAIL SLIDE-OVER ─────────────────────────  │
│                                                                 │
│ ┌─ DETAIL PANEL (slide from right) ─────────────────────────┐ │
│ │ ★ Booking Request #BR-2026-0315                            │ │
│ │                                                    [×]     │ │
│ │ ──────────────────────────────────────────────────────────  │ │
│ │                                                            │ │
│ │ Guest: Nguyễn Văn A                                        │ │
│ │ Email: guest@email.com                                     │ │
│ │ Phone: +84 901 234 567                                     │ │
│ │                                                            │ │
│ │ Room:  Deluxe Twin Ocean View                              │ │
│ │ Dates: Apr 15 → Apr 18 (3 nights)                         │ │
│ │ Guests: 2 Adults, 1 Child                                 │ │
│ │                                                            │ │
│ │ Special Requests:                                          │ │
│ │ "Late check-in around 10pm"                                │ │
│ │                                                            │ │
│ │ Source: /rooms/deluxe-ocean (scene: bedroom)               │ │
│ │ Submitted: Mar 15, 2026 at 14:32                           │ │
│ │                                                            │ │
│ │ ── STATUS ──                                               │ │
│ │ [New ▼] → Contacted / Confirmed / Cancelled                │ │
│ │                                                            │ │
│ │ ── ADMIN NOTES ──                                          │ │
│ │ [                                                 ]        │ │
│ │ [Called guest, confirmed. Room 812 assigned.       ]        │ │
│ │                                                            │ │
│ │ Assigned to: [Admin Name ▼]                                │ │
│ │                                                            │ │
│ │ [Save] [Reply Email] [Delete]                              │ │
│ └────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

---

## 7. 360° Hotspot Editor UI

### 7.1 Editor Layout

```
┌────────────────────────────────────────────────────────────────┐
│ Hotspot Editor — Deluxe Bedroom                [Preview] [Save]│
│                                                                 │
│ ┌─ TOOLBAR ──────────────────────────────────────────────────┐ │
│ │ [+ Add Hotspot]  [Scene: Bedroom ▼]  [Zoom: 100%]         │ │
│ │ Mode: ◉ Edit  ○ Preview                                    │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─────────────────────────────────────┬──────────────────────┐ │
│ │                                     │ HOTSPOT LIST          │ │
│ │                                     │                      │ │
│ │       360° PANORAMA VIEWER          │ 1. → Balcony (nav)   │ │
│ │       (Pannellum — click to place)  │ 2. ℹ Ocean View      │ │
│ │                                     │ 3. 💰 Book Now       │ │
│ │                                     │ 4. 📷 Room Detail    │ │
│ │       ⊕ Click here to place        │                      │ │
│ │          a new hotspot              │ ───────────────────   │ │
│ │                                     │ Selected: #1          │ │
│ │       ⊕ (existing hotspot,         │                      │ │
│ │           draggable)                │ Type: [Navigate ▼]   │ │
│ │                                     │ Label: [Balcony]     │ │
│ │                                     │ Icon: [arrow ▼]      │ │
│ │                                     │ Target: [Balcony ▼]  │ │
│ │                                     │ Pitch: [-5.2]        │ │
│ │                                     │ Yaw: [120.5]         │ │
│ │                                     │                      │ │
│ │                                     │ [Delete] [Duplicate] │ │
│ └─────────────────────────────────────┴──────────────────────┘ │
└────────────────────────────────────────────────────────────────┘

Interactions:
- Click on panorama → place new hotspot → opens type selection
- Drag existing hotspot → update pitch/yaw
- Click hotspot → select → show details in sidebar
- Preview mode → test as visitor (click hotspots, see cards)
- Scene dropdown → switch between scenes of same room
```

### 7.2 Hotspot Type Selection (after click to place)

```
┌───────────────────────────────┐
│ Choose Hotspot Type            │
│                                │
│ ┌───────────────────────────┐ │
│ │ → Navigate                │ │
│ │   Link to another scene   │ │
│ ├───────────────────────────┤ │
│ │ ℹ Information             │ │
│ │   Show details on click   │ │
│ ├───────────────────────────┤ │
│ │ 💰 Call to Action         │ │
│ │   Booking button / link   │ │
│ ├───────────────────────────┤ │
│ │ 📷 Gallery                │ │
│ │   Open photo gallery      │ │
│ └───────────────────────────┘ │
└───────────────────────────────┘
```

---

## 8. Responsive & Tablet Support

### 8.1 Breakpoints

| Breakpoint | Width | Layout |
|-----------|-------|--------|
| Desktop L | ≥1440px | Sidebar expanded, spacious content |
| Desktop | 1024–1439px | Sidebar expanded, standard content |
| Tablet | 768–1023px | Sidebar collapsed (icon-only), smaller tables |
| Mobile | <768px | Sidebar = overlay drawer, stacked forms |

### 8.2 Tablet Adjustments

```
- Sidebar: collapsed by default, expand on click
- Tables: horizontal scroll, fewer visible columns
- Forms: single column layout
- 360 Editor: full width, hotspot list below (not beside)
- Charts: simplified, fewer data points
```

---

## 9. Sequence Diagrams

### 9.1 Admin — Full Room CRUD Flow

```
Admin              List Page           Edit Page          API Server          Database
  │                     │                │                  │                  │
  │  /admin/rooms       │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ GET /admin/    │                  │                  │
  │                     │ rooms          │                  │                  │
  │                     ├───────────────►│                  │                  │
  │                     │ ◄── room list ─┤                  │                  │
  │  ◄── table view ────┤               │                  │                  │
  │                     │                │                  │                  │
  │  click "Edit"       │                │                  │                  │
  │  on Deluxe Twin     │                │                  │                  │
  ├──────────────────────────────────────►                  │                  │
  │                     │                │ GET /admin/      │                  │
  │                     │                │ rooms/:id        │                  │
  │                     │                ├─────────────────►│                  │
  │                     │                │ ◄── full room ───┤                  │
  │  ◄── edit form ──────────────────────┤                  │                  │
  │  (tabs: general,    │                │                  │                  │
  │   details, scenes,  │                │                  │                  │
  │   gallery, booking, │                │                  │                  │
  │   seo)              │                │                  │                  │
  │                     │                │                  │                  │
  │  modify fields      │                │                  │                  │
  │  click "Save Draft" │                │                  │                  │
  ├──────────────────────────────────────►                  │                  │
  │                     │                │ PUT /admin/      │                  │
  │                     │                │ rooms/:id        │                  │
  │                     │                ├─────────────────►│                  │
  │                     │                │ ◄── updated ─────┤                  │
  │  ◄── toast: saved ──┤               │                  │                  │
  │                     │                │                  │                  │
  │  click "Publish"    │                │                  │                  │
  ├──────────────────────────────────────►                  │                  │
  │                     │                │ PUT /admin/      │                  │
  │                     │                │ rooms/:id/publish│                  │
  │                     │                ├─────────────────►│ UPDATE           │
  │                     │                │                  │ is_published=true│
  │                     │                │                  │                  │
  │                     │                │ invalidate cache │                  │
  │                     │                │ purge CDN        │                  │
  │                     │                │ revalidate ISR   │                  │
  │                     │                │                  │                  │
  │  ◄── toast:         │                │                  │                  │
  │  "Published!"       │                │                  │                  │
```

### 9.2 Booking Config — Mode Switch Flow

```
Admin              Config Page          API Server          Database           Frontend
  │                     │                │                  │                  │
  │  /admin/booking     │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ GET /admin/    │                  │                  │
  │                     │ booking/config │                  │                  │
  │                     ├───────────────►│                  │                  │
  │                     │ ◄── config ────┤                  │                  │
  │  ◄── config page ───┤               │                  │                  │
  │  (current: external)│                │                  │                  │
  │                     │                │                  │                  │
  │  switch to          │                │                  │                  │
  │  "In-Page Modal"    │                │                  │                  │
  │                     │                │                  │                  │
  │  customize:         │                │                  │                  │
  │  - modal title      │                │                  │                  │
  │  - form fields      │                │                  │                  │
  │  - notify emails    │                │                  │                  │
  │                     │                │                  │                  │
  │  click "Save"       │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ PUT /admin/    │                  │                  │
  │                     │ booking/config │                  │                  │
  │                     ├───────────────►│                  │                  │
  │                     │                │ UPDATE           │                  │
  │                     │                │ booking_configs  │                  │
  │                     │                ├─────────────────►│                  │
  │                     │                │                  │                  │
  │                     │                │ invalidate cache │                  │
  │                     │                │ (booking config) │                  │
  │                     │                │                  │                  │
  │                     │ ◄── 200 ───────┤                  │                  │
  │  ◄── toast: saved ──┤               │                  │                  │
  │                     │                │                  │                  │
  │                     │                │                  │                  │
  │  ── Next visitor clicks "Book Now" on website ──────────────────────────  │
  │                     │                │                  │                  │
  │                     │                │                  │                  ├── GET /booking
  │                     │                │                  │                  │   /config
  │                     │                │                  │                  │   mode="modal"
  │                     │                │                  │                  │   → render modal
  │                     │                │                  │                  │   form overlay
```

---

*Tài liệu tiếp theo: **04-DASHBOARD-FEATURES.md** — Chi tiết từng module, workflows, edge cases*
