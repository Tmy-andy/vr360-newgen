# 04 — DASHBOARD FEATURES & WORKFLOWS
## Admin Dashboard — Hotel CMS Platform

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** 01-DASHBOARD-ARCHITECTURE · 02-DASHBOARD-DATABASE-API · 03-DASHBOARD-UI-UX · 05

---

## Mục Lục

1. [Room Management Workflow](#1-room-management-workflow)
2. [Service Management Workflow](#2-service-management-workflow)
3. [360° Scene & Hotspot Management](#3-360-scene--hotspot-management)
4. [Booking System ★ (Dual Mode)](#4-booking-system-dual-mode)
5. [Promotion Management](#5-promotion-management)
6. [Post / Blog Management](#6-post--blog-management)
7. [Gallery Management](#7-gallery-management)
8. [Menu Builder](#8-menu-builder)
9. [Translation Management](#9-translation-management)
10. [Hotel Settings](#10-hotel-settings)
11. [User Management](#11-user-management)
12. [Analytics & Reporting](#12-analytics--reporting)
13. [SEO / AIEO Management](#13-seo--aieo-management)

---

## 1. Room Management Workflow

### 1.1 CRUD Flow

```
[Admin /admin/rooms]
    │
    ├── View Room List
    │   ├── Table: thumbnail, name, type, price, scenes count, status
    │   ├── Filter: by type, by status (published/draft)
    │   ├── Sort: by name, price, sort_order, updated_at
    │   ├── Search: by name
    │   └── Bulk actions: publish, unpublish, delete
    │
    ├── Create Room [+ Add Room]
    │   ├── Tab "General": name, slug (auto-gen), short desc, full desc (Tiptap), thumbnail
    │   ├── Tab "Details": area, occupancy, bed type, view, price, highlights, amenities
    │   ├── Tab "Scenes": assign scenes to room, drag-to-reorder, set default scene
    │   ├── Tab "Gallery": upload images, drag-to-reorder, alt text, caption
    │   ├── Tab "Booking" ★: booking mode override, CTA labels
    │   ├── Tab "SEO": meta title, meta desc, OG image, canonical URL
    │   └── Actions: [Save Draft], [Publish], [Preview]
    │
    ├── Edit Room
    │   └── Same tabs as create, pre-filled
    │
    ├── Delete Room
    │   ├── Confirmation dialog: "Delete Deluxe Twin Room? This cannot be undone."
    │   ├── Soft delete (is_active = false) hoặc hard delete
    │   └── Cascade: remove room_scenes, room_amenities, gallery refs
    │
    └── Publish / Unpublish
        ├── Toggle is_published
        ├── Invalidate cache (Redis + CDN)
        └── Revalidate ISR pages
```

### 1.2 Room — Scene Assignment

```
┌────────────────────────────────────────────────────────────────┐
│ TAB: SCENES                                                     │
│                                                                 │
│ Assigned Scenes (drag to reorder):                              │
│                                                                 │
│ ┌─ 1 ⭐ ────────────────────────────────────────────────────┐ │
│ │ ≡  [📷 mini-preview]  Bedroom (deluxe-ocean-bedroom)      │ │
│ │    Default scene ✓     [Edit Hotspots →] [Remove]          │ │
│ └────────────────────────────────────────────────────────────┘ │
│ ┌─ 2 ────────────────────────────────────────────────────────┐ │
│ │ ≡  [📷 mini-preview]  Balcony (deluxe-ocean-balcony)       │ │
│ │    2 hotspots          [Edit Hotspots →] [Remove]          │ │
│ └────────────────────────────────────────────────────────────┘ │
│ ┌─ 3 ────────────────────────────────────────────────────────┐ │
│ │ ≡  [📷 mini-preview]  Bathroom (deluxe-ocean-bathroom)     │ │
│ │    1 hotspot           [Edit Hotspots →] [Remove]          │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ [+ Add Existing Scene]  [+ Upload New Panorama]                 │
│                                                                 │
│ Set default scene: [Bedroom (deluxe-ocean-bedroom) ▼]          │
└────────────────────────────────────────────────────────────────┘
```

### 1.3 Amenity Management

```
┌────────────────────────────────────────────────────────────────┐
│ TAB: DETAILS — Amenities Section                                │
│                                                                 │
│ Select amenities for this room:                                 │
│                                                                 │
│ VIEW:                                         ROOM:             │
│ ☑ Ocean View    ☑ Garden View                ☑ Air Condition   │
│ ☐ Mountain View ☐ City View                  ☑ Minibar         │
│ ☐ Pool View                                  ☑ Safe Box        │
│                                              ☑ Wardrobe        │
│ BATHROOM:                                    ☐ Iron            │
│ ☑ Bathtub       ☑ Rain Shower                                  │
│ ☑ Hair Dryer    ☐ Bidet                      TECHNOLOGY:       │
│                                              ☑ WiFi            │
│ OUTDOOR:                                     ☑ Smart TV        │
│ ☑ Balcony       ☐ Terrace                    ☐ USB Charging    │
│ ☐ Private Pool                               ☑ Telephone       │
│                                                                 │
│ [Manage Amenities →] (go to master amenity list)               │
└────────────────────────────────────────────────────────────────┘
```

---

## 2. Service Management Workflow

### 2.1 Service Types

| Category | Ví dụ | CTA Options |
|----------|-------|-------------|
| `dining` | Nhà hàng Skylight, Lounge Bar | Reserve Table, View Menu |
| `spa` | Spa Treatment, Massage | Book Treatment, Spa Menu |
| `recreation` | Pool, Fitness, Beach | Info only, Gallery |
| `meeting` | Meeting Room, Ballroom | Event Enquiry, Packages |
| `other` | Laundry, Airport Transfer | Enquire |

### 2.2 Service CTA Configuration

```
┌────────────────────────────────────────────────────────────────┐
│ TAB: CTA & BOOKING                                              │
│                                                                 │
│ CTA Type: [Reserve ▼]    (booking | reserve | enquiry | none)  │
│                                                                 │
│ CTA Label:    [Reserve Table        ]                           │
│ Secondary:    [View Menu            ]                           │
│                                                                 │
│ Booking Mode:                                                   │
│ ◉ Use global settings (In-Page Modal)                           │
│ ○ External Link: [URL...]                                       │
│ ○ In-Page Modal (custom form)                                   │
│                                                                 │
│ ── If Modal, service-specific fields: ──                        │
│ ☑ Include "Date" field                                          │
│ ☑ Include "Time" field (for restaurant)                         │
│ ☑ Include "Number of guests" field                              │
│ ☐ Include "Package selection" (for spa)                         │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

---

## 3. 360° Scene & Hotspot Management

### 3.1 Scene List (/admin/scenes)

```
┌────────────────────────────────────────────────────────────────┐
│ Scenes & 360°                              [+ Upload Panorama]  │
│                                                                 │
│ ┌─ TABLE ────────────────────────────────────────────────────┐ │
│ │ Preview   Scene ID              Used In    Hotspots  Acts  │ │
│ │ ──────── ─────────────────────  ─────────  ────────  ──── │ │
│ │ [🌐]     lobby                  Homepage   3         [⋮]  │ │
│ │ [🌐]     rooms-overview         Rooms      4         [⋮]  │ │
│ │ [🌐]     deluxe-ocean-bedroom   Deluxe     3         [⋮]  │ │
│ │ [🌐]     deluxe-ocean-balcony   Deluxe     2         [⋮]  │ │
│ │ [🌐]     deluxe-ocean-bathroom  Deluxe     1         [⋮]  │ │
│ │ [🌐]     dining-skylight        Dining     2         [⋮]  │ │
│ │ [🌐]     pool-main              Pool       1         [⋮]  │ │
│ │ [🌐]     spa-lobby              Spa        2         [⋮]  │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ Row actions:                                                    │
│ - Edit Config (yaw, pitch, hfov, auto-rotate)                   │
│ - Edit Hotspots → Opens Visual Editor                           │
│ - Replace Panorama                                              │
│ - Delete                                                        │
└────────────────────────────────────────────────────────────────┘
```

### 3.2 Scene Upload Flow

```
[+ Upload Panorama]
    │
    ▼
┌─ Upload Dialog ─────────────────────────────────────────┐
│                                                          │
│  [Drop panorama image here or click to browse]           │
│                                                          │
│  Requirements:                                           │
│  • Equirectangular format (2:1 ratio)                    │
│  • Minimum 4096×2048 pixels                              │
│  • JPEG or PNG                                           │
│  • Max 50MB                                              │
│                                                          │
│  ── After upload: ──                                     │
│                                                          │
│  Scene ID: [dining-skylight-v2        ]                  │
│  Label:    [Skylight Restaurant       ]                  │
│                                                          │
│  ┌─ Mini Preview ──────────────────────────────────────┐│
│  │ [Pannellum preview of uploaded panorama]             ││
│  │ Drag to set default viewing angle                    ││
│  └──────────────────────────────────────────────────────┘│
│                                                          │
│  Default Yaw:   [auto-detected   ]                       │
│  Default Pitch: [auto-detected   ]                       │
│  Default HFOV:  [100             ]                       │
│  Auto Rotate:   [0.5             ]                       │
│                                                          │
│  Assign to: [Room: Dining ▼] (optional)                  │
│                                                          │
│  [Cancel]                         [Save Scene Config]    │
└──────────────────────────────────────────────────────────┘
```

### 3.3 Hotspot Visual Editor — Detailed Workflow

```
[/admin/scenes/:id/editor]

1. EDITOR LOAD:
   - Load scene config from API
   - Init Pannellum viewer (editor mode — click enabled)
   - Load existing hotspots as draggable markers
   - Show hotspot list in sidebar

2. ADD NEW HOTSPOT:
   Click [+ Add Hotspot]
   → Cursor changes to crosshair
   → Click on panorama
   → Pannellum: mouseEventToCoords() → {pitch, yaw}
   → Show type selection popup (navigate/info/cta/gallery)
   → Show detail form in sidebar
   → Fill details → Save → marker appears on panorama

3. EDIT HOTSPOT:
   Click existing marker on panorama (or from sidebar list)
   → Marker highlighted (gold ring)
   → Detail form in sidebar
   → Edit fields → Auto-save on blur (debounced 1s)

4. DRAG HOTSPOT:
   Drag marker on panorama
   → Real-time pitch/yaw update in sidebar
   → On drop → PATCH API (position only)
   → Optimistic update + revert on error

5. DELETE HOTSPOT:
   Select marker → Click [Delete] in sidebar
   → Confirm dialog: "Delete hotspot 'Balcony →'?"
   → DELETE API → Remove marker from panorama

6. PREVIEW MODE:
   Toggle "Preview" → viewer behaves like public website
   → Click navigate hotspot → scene change
   → Click info hotspot → card popup
   → Click CTA → simulated booking action
   → Toggle back to "Edit" mode

7. SCENE SWITCHING:
   Dropdown [Scene: Bedroom ▼]
   → Load different scene's hotspots
   → Navigate hotspots show target preview
```

### 3.4 Hotspot Detail Form (per type)

```
── TYPE: NAVIGATE ──
Label:          [Go to Balcony     ]
Icon:           [arrow ▼]  (arrow, door-open, stairs)
Target Scene:   [deluxe-ocean-balcony ▼]
Preview target: [Mini 360 preview of target scene]

── TYPE: INFO ──
Label:          [Ocean View        ]
Description:    [Tầm nhìn 180° ra vịnh Nha Trang]
Image:          [Upload detail image — 300×200]
Show on hover:  ☑ Expand label  ☐ Show card immediately

── TYPE: CTA ──
Label:          [Book Now          ]
CTA Label:      [Check Availability]
Action:         ◉ Follow booking config  ○ Custom URL
Custom URL:     [https://...       ] (if custom)
Pulse animation: ☑ Enabled

── TYPE: GALLERY ──
Label:          [View Room Photos  ]
Gallery source: ◉ Room gallery  ○ Custom selection
Start at image: [Image #3 ▼]
```

---

## 4. Booking System ★ (Dual Mode)

### 4.1 Complete Booking Flow — External Link Mode

```
┌─────────────────────────────────────────────────────────────────┐
│ EXTERNAL LINK MODE — Luồng hoàn chỉnh                           │
│                                                                   │
│  ADMIN SETUP:                                                    │
│  1. /admin/booking → Select "External Link"                      │
│  2. Enter default booking URL                                    │
│  3. Choose target: new tab / same tab                            │
│  4. (Optional) Override URL per-room at /admin/rooms/:id/booking│
│                                                                   │
│  GUEST EXPERIENCE:                                               │
│  1. Visit /rooms/deluxe-ocean (360° page)                        │
│  2. See "Book Now" button in InfoPanel + FloatingCTA             │
│  3. Click "Book Now"                                             │
│  4. → analytics.track('booking_click', {room, source})           │
│  5. → Resolve URL: room.booking_url || global.external_url       │
│  6. → window.open(url, target)                                   │
│  7. → Guest lands on external booking engine                     │
│  8. → Complete booking on external platform                      │
│                                                                   │
│  ADMIN TRACKING:                                                 │
│  - Analytics: booking_click events (scene, room, page)           │
│  - No booking requests in dashboard (handled externally)         │
│  - Can track conversion via UTM params on external URL           │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Complete Booking Flow — In-Page Modal Mode

```
┌─────────────────────────────────────────────────────────────────┐
│ IN-PAGE MODAL MODE — Luồng hoàn chỉnh                           │
│                                                                   │
│  ADMIN SETUP:                                                    │
│  1. /admin/booking → Select "In-Page Booking Modal"              │
│  2. Customize modal: title, subtitle, success message            │
│  3. Configure form fields (add/remove/reorder)                   │
│  4. Set notification emails                                      │
│  5. Configure auto-reply email template                          │
│  6. (Optional) Add webhook URL (Slack, Zalo)                    │
│  7. [Send Test Email] to verify                                  │
│                                                                   │
│  GUEST EXPERIENCE:                                               │
│  1. Visit /rooms/deluxe-ocean (360° page)                        │
│  2. See "Book Now" button in InfoPanel + FloatingCTA             │
│  3. Click "Book Now"                                             │
│  4. → Modal overlay opens (on same page, 360° visible behind)   │
│  5. → Modal renders dynamic form fields from config              │
│  6. → Room name auto-filled (readonly)                           │
│  7. → Guest fills: name, email, phone, dates, guests, notes     │
│  8. → Client-side validation (Zod)                               │
│  9. → Submit → POST /api/v1/booking/request                     │
│  10. → Loading spinner on button                                │
│  11. → Success: show success message, close modal after 3s      │
│  12. → Error: show error message, retry button                  │
│                                                                   │
│  SERVER-SIDE (on submit):                                        │
│  1. Validate request body (Zod)                                  │
│  2. Rate limit check (max 5 requests per IP per hour)           │
│  3. Honeypot field check (spam prevention)                       │
│  4. INSERT INTO booking_requests                                │
│  5. Send notification email to hotel:                            │
│     - Subject: "New Booking Request — Deluxe Twin"              │
│     - Body: guest info, dates, room, special requests           │
│     - Reply-to: guest email                                     │
│  6. Send auto-reply to guest (if enabled):                      │
│     - Subject: "Booking Request Received — Boton Blue Hotel"    │
│     - Body: confirmation, hotel contact, what to expect next    │
│  7. Send webhook (if configured):                               │
│     - POST to webhook URL with booking data JSON               │
│  8. Track analytics: booking_request_submit                     │
│                                                                   │
│  ADMIN MANAGEMENT:                                               │
│  1. /admin/booking/requests → See all requests                  │
│  2. Filter by status: New, Contacted, Confirmed, Cancelled      │
│  3. Click request → detail panel                                │
│  4. Change status, add notes, assign to staff                   │
│  5. Reply to guest email directly from dashboard                │
│  6. Export to CSV/Excel                                          │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3 Modal Form — Frontend Component

```typescript
// BookingModal component
interface BookingModalProps {
  isOpen: boolean;
  onClose: () => void;
  room?: { id: string; name: string; slug: string };
  service?: { id: string; name: string; slug: string };
}

// Flow:
// 1. Fetch /api/v1/booking/config → get form fields
// 2. Render dynamic form based on fields
// 3. Validate with Zod schema (built from field configs)
// 4. Submit → POST /api/v1/booking/request
// 5. Show result (success/error)
```

### 4.4 Booking Widget on 360° Page

```
── WHEN MODE = EXTERNAL ──

InfoPanel CTA:
┌──────────────────────────┐
│ From 2,500,000 VND/night │
│                          │
│ [Book Now →]             │  ← window.open(url)
│ [Check Availability]     │  ← window.open(url + params)
└──────────────────────────┘

FloatingCTA (bottom-right):
┌──────────────────────────┐
│ [Book Now →]             │  ← window.open(url)
└──────────────────────────┘


── WHEN MODE = MODAL ──

InfoPanel CTA:
┌──────────────────────────┐
│ From 2,500,000 VND/night │
│                          │
│ [Book Now →]             │  ← opens BookingModal
│ [Enquire]                │  ← opens BookingModal (enquiry mode)
└──────────────────────────┘

FloatingCTA (bottom-right):
┌──────────────────────────┐
│ [Book Now →]             │  ← opens BookingModal
└──────────────────────────┘

Modal Overlay:
┌──────────────────────────────────────────────────┐
│                    [×]                            │
│                                                   │
│       ★ Book Your Stay                            │
│       Fill in your details and we'll              │
│       confirm your reservation                    │
│                                                   │
│       Room: Deluxe Twin Ocean View (readonly)     │
│                                                   │
│       Full Name *                                 │
│       [                                    ]      │
│                                                   │
│       Email Address *                             │
│       [                                    ]      │
│                                                   │
│       Phone Number *                              │
│       [                                    ]      │
│                                                   │
│       Check-in *          Check-out *             │
│       [📅 Apr 15]         [📅 Apr 18]             │
│                                                   │
│       Adults *            Children                │
│       [2 ▼]               [0 ▼]                   │
│                                                   │
│       Special Requests                            │
│       [                                    ]      │
│       [Late check-in around 10pm            ]     │
│                                                   │
│       [   Send Booking Request   ]                │
│                                                   │
│  Background: 360° panorama visible (dimmed)       │
│  Animation: fadeIn + scaleFrom(0.95)              │
│  Close: × button, Escape key, click backdrop      │
└──────────────────────────────────────────────────┘

Success State:
┌──────────────────────────────────────────────────┐
│                                                   │
│       ✅ Thank you!                               │
│                                                   │
│       We have received your booking request       │
│       for Deluxe Twin Ocean View.                 │
│                                                   │
│       We will contact you within 24 hours         │
│       to confirm your reservation.                │
│                                                   │
│       A confirmation email has been sent          │
│       to guest@email.com                          │
│                                                   │
│       [Close]                                     │
│                                                   │
│  Auto-close after 5s                              │
└──────────────────────────────────────────────────┘
```

### 4.5 Email Templates

```
── HOTEL NOTIFICATION EMAIL ──

Subject: 🏨 New Booking Request — Deluxe Twin Ocean View
From: noreply@botonblue.com
To: sales@botonblue.com, manager@botonblue.com
Reply-To: guest@email.com

Body:
┌─────────────────────────────────────────────────┐
│ New Booking Request                              │
│                                                   │
│ Guest: Nguyễn Văn A                              │
│ Email: guest@email.com                           │
│ Phone: +84 901 234 567                           │
│                                                   │
│ Room: Deluxe Twin Ocean View                     │
│ Check-in: April 15, 2026                         │
│ Check-out: April 18, 2026 (3 nights)            │
│ Guests: 2 Adults, 1 Child                       │
│                                                   │
│ Special Requests:                                │
│ "Late check-in around 10pm"                      │
│                                                   │
│ Source: /rooms/deluxe-ocean                      │
│ Scene: deluxe-ocean-bedroom                      │
│ Submitted: Mar 15, 2026 at 14:32                │
│                                                   │
│ [View in Dashboard →]                            │
│                                                   │
│ Reply directly to this email to contact guest.   │
└─────────────────────────────────────────────────┘


── GUEST AUTO-REPLY EMAIL ──

Subject: Booking Request Received — Boton Blue Hotel & Spa
From: noreply@botonblue.com
To: guest@email.com

Body:
┌─────────────────────────────────────────────────┐
│ [Hotel Logo]                                     │
│                                                   │
│ Dear Nguyễn Văn A,                               │
│                                                   │
│ Thank you for your booking request at            │
│ Boton Blue Hotel & Spa.                          │
│                                                   │
│ Your Request Details:                            │
│ • Room: Deluxe Twin Ocean View                   │
│ • Check-in: April 15, 2026                       │
│ • Check-out: April 18, 2026                      │
│ • Guests: 2 Adults, 1 Child                     │
│                                                   │
│ Our team will review your request and contact    │
│ you within 24 hours to confirm availability      │
│ and finalize your reservation.                   │
│                                                   │
│ If you need immediate assistance:                │
│ 📞 +84 258 383 6868                              │
│ ✉ sales@botonblue.com                            │
│                                                   │
│ We look forward to welcoming you!                │
│                                                   │
│ Warm regards,                                     │
│ Boton Blue Hotel & Spa                           │
│ 86 Trần Phú, Nha Trang                          │
└─────────────────────────────────────────────────┘
```

---

## 5. Promotion Management

### 5.1 Promotion CRUD

```
Fields:
- Title, slug
- Description (rich text)
- Cover image
- Start date, end date
- Discount type: percentage | fixed amount | free upgrade
- Discount value
- Applicable rooms: all | specific rooms (multi-select)
- Terms & conditions (rich text)
- CTA label, CTA URL (or booking modal)
- Status: draft | published | expired (auto by date)
- SEO fields

Special features:
- Auto-expire when end_date passes
- Badge on room cards: "🔥 20% OFF"
- Countdown timer on promotion page
- Related rooms list
```

---

## 6. Post / Blog Management

### 6.1 Post Editor

```
Fields:
- Title, slug
- Category: news | blog | events | guides
- Tags (multi-select, create new)
- Cover image
- Excerpt (auto-gen from content if empty)
- Content: Tiptap rich text editor
  - Headings H2-H4
  - Bold, italic, underline
  - Links
  - Images (inline upload)
  - Blockquotes
  - Ordered/unordered lists
  - Code blocks
  - Horizontal rule
  - Tables
  - YouTube embed
- Author (auto from current user)
- Published date (schedulable)
- Status: draft | published | scheduled
- SEO fields
- Related posts
- Related rooms/services
```

---

## 7. Gallery Management

### 7.1 Gallery Overview

```
┌────────────────────────────────────────────────────────────────┐
│ Gallery                                    [+ Upload Images]    │
│                                                                 │
│ ┌─ FILTER ───────────────────────────────────────────────────┐ │
│ │ [All] [Rooms] [Services] [Hotel] [Unassigned]              │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ IMAGE GRID ───────────────────────────────────────────────┐ │
│ │                                                              │ │
│ │ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐         │ │
│ │ │ 📷  │ │ 📷  │ │ 📷  │ │ 📷  │ │ 📷  │ │ 📷  │         │ │
│ │ │     │ │     │ │     │ │     │ │     │ │     │         │ │
│ │ │ ☐   │ │ ☐   │ │ ☐   │ │ ☐   │ │ ☐   │ │ ☐   │         │ │
│ │ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘         │ │
│ │ Pool    Lobby   Deluxe  Suite   Dining  Spa              │ │
│ │ Room    Room    Bedroom Bed     Hall    Pool             │ │
│ │                                                              │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ Selected: 3 images   [Assign to Room ▼] [Delete]               │
└────────────────────────────────────────────────────────────────┘

Features:
- Grid view (default) / list view toggle
- Multi-select for bulk operations
- Assign images to rooms/services
- Edit alt text, caption
- Replace image (keep references)
- Image info: dimensions, size, URL, usage count
```

---

## 8. Menu Builder

### 8.1 Menu Editor

```
┌────────────────────────────────────────────────────────────────┐
│ Menu Management                                                 │
│                                                                 │
│ ┌─ TABS ─────────────────────────────────────────────────────┐ │
│ │ [Header Menu] [Footer Menu] [Mobile Menu]                   │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ ┌─ HEADER MENU ──────────────────────────────────────────────┐ │
│ │                                                              │ │
│ │ Drag to reorder:                                             │ │
│ │                                                              │ │
│ │ ┌─ 1 ────────────────────────────────────────────────────┐  │ │
│ │ │ ≡  Home          URL: /         Visible: ✓     [✏️][🗑]│  │ │
│ │ └────────────────────────────────────────────────────────┘  │ │
│ │ ┌─ 2 ────────────────────────────────────────────────────┐  │ │
│ │ │ ≡  Rooms         URL: /rooms    Visible: ✓     [✏️][🗑]│  │ │
│ │ │    └─ Deluxe     URL: /rooms/deluxe-ocean              │  │ │
│ │ │    └─ Suite      URL: /rooms/suite-ocean               │  │ │
│ │ └────────────────────────────────────────────────────────┘  │ │
│ │ ┌─ 3 ────────────────────────────────────────────────────┐  │ │
│ │ │ ≡  Dining        URL: /dining   Visible: ✓     [✏️][🗑]│  │ │
│ │ └────────────────────────────────────────────────────────┘  │ │
│ │ ┌─ 4 ────────────────────────────────────────────────────┐  │ │
│ │ │ ≡  Recreation    URL: /rec      Visible: ✓     [✏️][🗑]│  │ │
│ │ └────────────────────────────────────────────────────────┘  │ │
│ │                                                              │ │
│ │ [+ Add Menu Item]  [+ Add Dropdown]                          │ │
│ │                                                              │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ Special: "Book Now" button (always last, gold accent)           │
│ Position: [right ▼]  Style: [button ▼]                          │
│                                                                 │
│                                              [Save Menu]        │
└────────────────────────────────────────────────────────────────┘
```

---

## 9. Translation Management

### 9.1 Translation Editor

```
┌────────────────────────────────────────────────────────────────┐
│ Translations                                                    │
│                                                                 │
│ Entity: [Rooms ▼]  Item: [Deluxe Twin Room ▼]                  │
│                                                                 │
│ ┌─ FIELD EDITOR ─────────────────────────────────────────────┐ │
│ │                                                              │ │
│ │ Field: name                                                  │ │
│ │ ┌──────────┬──────────┬──────────┬──────────┬──────────┐    │ │
│ │ │ 🇬🇧 EN   │ 🇻🇳 VI   │ 🇷🇺 RU   │ 🇰🇷 KO   │ 🇨🇳 ZH   │    │ │
│ │ ├──────────┼──────────┼──────────┼──────────┼──────────┤    │ │
│ │ │ Deluxe   │ Phòng    │ Номер    │ 디럭스    │ 豪华双    │    │ │
│ │ │ Twin     │ Deluxe   │ Делюкс   │ 트윈 룸  │ 床房     │    │ │
│ │ │ Room     │ Twin     │ Твин     │          │          │    │ │
│ │ └──────────┴──────────┴──────────┴──────────┴──────────┘    │ │
│ │                                                              │ │
│ │ Field: description                                           │ │
│ │ ┌──────────┬──────────┬──────────┬──────────┬──────────┐    │ │
│ │ │ 🇬🇧 EN   │ 🇻🇳 VI   │ 🇷🇺 RU   │ 🇰🇷 KO   │ 🇨🇳 ZH   │    │ │
│ │ ├──────────┼──────────┼──────────┼──────────┼──────────┤    │ │
│ │ │[Rich     │[Rich     │[Rich     │[Rich     │[Rich     │    │ │
│ │ │ text     │ text     │ text     │ text     │ text     │    │ │
│ │ │ editor]  │ editor]  │ editor]  │ editor]  │ editor]  │    │ │
│ │ └──────────┴──────────┴──────────┴──────────┴──────────┘    │ │
│ │                                                              │ │
│ │ Progress: EN ████████ 100%  VI ██████░░ 75%  RU ████░░░░ 50%│ │
│ │                                                              │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                 │
│                                     [Save Translations]         │
└────────────────────────────────────────────────────────────────┘
```

---

## 10. Hotel Settings

### 10.1 Settings Page

```
Tabs:
- General: Hotel name, slug, description, star rating
- Branding: Logo upload, primary color, accent color, favicon
- Contact: Address, phone, email, Google Maps embed
- Social: Facebook, Instagram, TikTok, YouTube, LinkedIn
- Locales: Default language, available languages
- Advanced: Custom CSS, custom JS, analytics snippets (GA, GTM)
```

---

## 11. User Management

### 11.1 User List (super_admin only)

```
┌────────────────────────────────────────────────────────────────┐
│ Users                                           [+ Invite User] │
│                                                                 │
│ ┌─ TABLE ────────────────────────────────────────────────────┐ │
│ │ Avatar  Name           Email              Role      Status │ │
│ │ ─────── ─────────────  ────────────────── ──────── ─────── │ │
│ │ [👤]    Trần Minh Anh  admin@botonblue    Admin    Active  │ │
│ │ [👤]    Nguyễn Thị B   editor@botonblue   Editor   Active  │ │
│ │ [👤]    Lê Văn C       viewer@botonblue   Viewer   Active  │ │
│ │ [👤]    Jane Doe       jane@botonblue     Admin    Invited │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ Invite flow:                                                    │
│ 1. Enter email → Select role → [Send Invite]                   │
│ 2. Email sent with setup link (expires 48h)                    │
│ 3. Invitee sets password → account active                      │
└────────────────────────────────────────────────────────────────┘
```

---

## 12. Analytics & Reporting

### 12.1 Analytics Dashboard

```
Key Metrics:
- Total page views (trend line)
- Unique sessions
- Average session duration
- Top pages by views
- Top 360° scenes (view count, avg time in scene)
- Top hotspots (click count, click rate)
- Booking CTA clicks (by room, by source)
- Booking requests (if modal mode) — submitted, conversion rate
- Device breakdown (desktop/mobile/tablet)
- Geographic breakdown (country)
- Referrer sources

Export Options:
- Date range picker
- Export CSV
- Export Excel
- Scheduled email reports (weekly/monthly)
```

---

## 13. SEO / AIEO Management

### 13.1 Per-Page SEO

Mỗi room, service, promotion, post đều có:
- Meta title (max 70 chars, character counter)
- Meta description (max 160 chars, character counter)
- OG image (auto from thumbnail, overrideable)
- Canonical URL (auto-generated)
- No-index option
- JSON-LD structured data preview

### 13.2 Global SEO Settings

```
- Sitemap: auto-generated, /sitemap.xml
- Robots.txt: editable
- Default OG image (fallback)
- Google Analytics ID
- Google Tag Manager ID
- Facebook Pixel ID
- Custom meta tags
- AIEO: AI-friendly structured data for AI search engines
```

---

*Tài liệu tiếp theo: **05-DASHBOARD-TECHNICAL.md** — Implementation details, component code, state management*
