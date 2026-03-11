# 03 — UI/UX DESIGN SPECIFICATION
## Website 360 Thế Hệ Mới — Hotel CMS Platform

**Version:** 1.2  
**Date:** 2026-03-10  
**Updated:** Thêm sequence diagrams (Room Page UX, Booking, Panel Interaction), chuẩn hóa bố cục  
**Related:** 01-SYSTEM-ARCHITECTURE · 02-DATABASE-API-SPEC · 04-TECHNICAL-IMPLEMENTATION

---

## Mục Lục

1. [Design Principles](#1-design-principles)
2. [Design Tokens](#12-design-tokens)
3. [Layout Architecture](#2-layout-architecture)
4. [Component Specifications](#3-component-specifications)
5. [Page-by-Page UX Flows](#4-page-by-page-ux-flows)
6. [Responsive Breakpoints](#5-responsive-breakpoints)
7. [Animation & Transition Spec](#6-animation--transition-spec)
8. [Sequence Diagrams](#sequence-diagrams)
9. [Accessibility (WCAG 2.1 AA)](#7-accessibility-wcag-21-aa)
10. [Iconography](#8-iconography)

---

## 1. Design Principles

### 1.1 Nguyên Tắc Chủ Đạo

| # | Nguyên tắc | Giải thích |
|---|-----------|-----------|
| 1 | **360-First** | Không gian 360 là nền chính, nội dung là overlay |
| 2 | **Immersive but Informative** | Trải nghiệm nhập vai nhưng vẫn đủ thông tin để quyết định đặt phòng |
| 3 | **Show on Demand** | UI thu gọn khi tương tác 360, hiện lại khi cần |
| 4 | **Book in Context** | CTA booking luôn trong tầm với, nằm trong trải nghiệm |
| 5 | **Mobile-First** | Thiết kế cho touch trước, adapt cho desktop |
| 6 | **Luxury Aesthetic** | Typography, spacing, animation phải cao cấp, tinh tế |

### 1.2 Design Tokens (CSS Variables)

```css
:root {
  /* Colors - Luxury Dark Theme (demo implementation) */
  --gold: #C9A861;                          /* Primary accent */
  --gold-soft: rgba(201,168,97,.12);        /* Subtle gold bg */
  --gold-mid: rgba(201,168,97,.3);          /* Medium gold */
  --gold-glow: rgba(201,168,97,.45);        /* Glow/shadow */
  --dark: #060a10;                          /* Background */
  --panel: rgba(8,14,26,.82);               /* Panel bg (glass) */
  --panel-solid: rgba(12,20,38,.94);        /* Solid panel (info cards) */
  --glass: rgba(255,255,255,.05);           /* Glass morphism */
  --glass-border: rgba(255,255,255,.08);    /* Subtle borders */
  --text: #fff;                             /* Primary text */
  --text-dim: rgba(255,255,255,.6);         /* Secondary text */
  --text-mute: rgba(255,255,255,.35);       /* Muted text */

  /* Typography */
  --serif: 'Cormorant Garamond', Georgia, serif;  /* Display headings */
  --sans: 'Outfit', system-ui, sans-serif;         /* Body text */

  /* Transitions */
  --ease: cubic-bezier(.16,1,.3,1);         /* Expo ease-out */
}
```

---

## 2. Page Layout Architecture

### 2.1 Hai Loại Page Layout

**Layout A — 360 Immersive Pages** (Rooms, Dining, Spa, Pool, Meeting)

```
┌──────────────────────────────────────────────────────┐
│ NAVBAR (transparent, z-40)                            │
│ [Logo]           [Rooms] [Dining] [Rec] ...  [Book]  │
├──────────────────────────────────────────────────────┤
│                                                       │
│              PANORAMA 360 VIEWER (z-0)                │
│              Full viewport width × height             │
│                                                       │
│  ┌─────────────┐                    ┌──────────────┐ │
│  │ INFO PANEL  │                    │              │ │
│  │ (z-30)      │                    │   HOTSPOTS   │ │
│  │             │                    │   (z-10)     │ │
│  │ Room Name   │                    │              │ │
│  │ Area / View │                    │  ⊕ Balcony   │ │
│  │ Amenities   │                    │  ⊕ Bathroom  │ │
│  │ Price       │                    │  ⊕ Ocean     │ │
│  │             │                    │              │ │
│  │ [Book Now]  │                    └──────────────┘ │
│  │ [Enquire]   │                                     │
│  └─────────────┘                                     │
│                                                       │
│  ┌──────────────────────────────────────────────────┐ │
│  │ BOTTOM BAR (z-20)                                 │ │
│  │ [Scene Nav Thumbnails]  [Gallery]  [Hide/Show UI]│ │
│  └──────────────────────────────────────────────────┘ │
│                                                       │
│  ┌──────────────────────────────────────────────────┐ │
│  │ FLOATING CTA (z-50) - Fixed bottom right          │ │
│  │ [Check Rate] [Book Now]                           │ │
│  └──────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

**Layout B — CMS Content Pages** (Home, News, Promotions, Contact, Gallery)

```
┌──────────────────────────────────────────────────────┐
│ NAVBAR (solid background)                             │
│ [Logo]           [Rooms] [Dining] [Rec] ...  [Book]  │
├──────────────────────────────────────────────────────┤
│                                                       │
│  HERO SECTION (optional 360 hoặc image slideshow)    │
│                                                       │
├──────────────────────────────────────────────────────┤
│                                                       │
│  CONTENT AREA                                         │
│  (Standard website layout — text, images, cards)     │
│                                                       │
├──────────────────────────────────────────────────────┤
│ FOOTER                                                │
│ [Logo] [Quick Links] [Contact] [Social] [Copyright]  │
└──────────────────────────────────────────────────────┘
```

### 2.2 Navbar Behavior

```
┌─────────────────────────────────────────────────────────────┐
│ State: DEFAULT (360 pages)                                   │
│ Background: transparent → rgba(13,17,23,0.8) on scroll       │
│ Items: white text, gold accent on hover                      │
│ Logo: white version                                          │
│ Booking CTA: gold border button                              │
├─────────────────────────────────────────────────────────────┤
│ State: IMMERSIVE (user đang tương tác 360)                   │
│ Navbar: auto-hide sau 3s không tương tác vùng nav            │
│ Hiện lại: hover vùng top 60px hoặc tap (mobile)             │
├─────────────────────────────────────────────────────────────┤
│ State: CMS PAGES (News, Contact...)                          │
│ Background: solid --color-primary                            │
│ Items: white text                                            │
│ Sticky on scroll                                             │
├─────────────────────────────────────────────────────────────┤
│ State: MOBILE (< 860px)                                      │
│ Hamburger menu → Full-screen overlay (z:200)                 │
│ Overlay: 3-section layout (header / body / footer)           │
│ Background: rgba(6,10,16,0.97) + blur(50px)                  │
│ Header: logo (white) + close button (circle, glass bg)       │
│ Body: numbered nav links (serif 32px) + stagger animation    │
│   "01 Home →", "02 Rooms →", etc.                            │
│   Active + hover: gold color, arrow slides in                │
│   Stagger delay: 80ms + i*60ms per link                      │
│ CTA: "Book Now" gold pill, delay 400ms                       │
│ Footer: contact (phone, email, address) + social icons       │
│   Facebook, Instagram, YouTube (circle glass buttons)        │
│   Footer delay 200ms, translateY(10px→0)                     │
│ Booking CTA prominent                                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Component Specifications

### 3.1 PanoramaViewer (Core 360 Component)

```
┌────────────────────────────────────────────────┐
│ Props:                                          │
│   panoramaUrl: string                           │
│   config: {                                     │
│     defaultYaw, defaultPitch, defaultHfov,      │
│     autoRotate, minHfov, maxHfov                │
│   }                                             │
│   hotspots: Hotspot[]                           │
│   onHotspotClick: (hotspot) => void             │
│   onUserInteraction: () => void                 │
│   onIdle: () => void                            │
│                                                  │
│ Behavior:                                        │
│   - Init Pannellum viewer full-viewport          │
│   - Load panorama with loading spinner           │
│   - Render hotspots from config                  │
│   - Emit onUserInteraction khi drag/touch        │
│   - Emit onIdle sau 4s không tương tác           │
│   - Auto-rotate khi idle (configurable)          │
│   - Support scene transition animation           │
│                                                  │
│ Loading States:                                  │
│   1. Blur placeholder (tiny image) → fade in     │
│   2. Loading ring animation ở center             │
│   3. Panorama loaded → fade out loading          │
│                                                  │
│ Scene Transition:                                │
│   1. Current scene fade to black (300ms)         │
│   2. Load new panorama                           │
│   3. Fade in new scene (500ms)                   │
└────────────────────────────────────────────────┘
```

### 3.2 InfoPanel (Side Panel)

```
DESKTOP (>= 1024px):
┌────────────────────┐
│ ★ DELUXE TWIN ROOM │  ← font-display, 24px
│ Ocean View         │  ← badge, gold accent
│────────────────────│
│ 35m² · 2 Adults    │  ← icon + text, 14px
│────────────────────│
│                     │
│ Mô tả ngắn phòng  │  ← font-body, 15px
│ với đầy đủ thông   │     max 3 dòng, expandable
│ tin tiện nghi...    │
│ [Xem thêm ▼]       │
│────────────────────│
│ ☐ Ocean View       │
│ ☐ Balcony          │  ← Amenities grid (2 col)
│ ☐ WiFi             │
│ ☐ Air Conditioning │
│ ☐ Bathtub          │
│ ☐ Mini Bar         │
│────────────────────│
│ Từ 2,500,000 VND   │  ← price highlight
│                     │
│ [Check Rate]        │  ← Primary CTA (gold)
│ [Book Now  ]        │  ← Secondary CTA (outline)
│ [Enquire   ]        │  ← Text link
└────────────────────┘
Width: 360px (fixed)
Position: left 24px, top 100px
Background: var(--color-bg-panel)
Backdrop-filter: blur(16px)
Border: 1px solid var(--color-border)
Border-radius: var(--radius-lg)
Animation: slideInFromLeft 500ms var(--ease-out-expo)

STATES:
- Expanded: full panel visible
- Collapsed: chỉ hiện tên phòng + nút expand
- Hidden: khi user đang tương tác 360 (fade out 300ms)
- Re-appear: sau 4s idle (fade in 500ms)

MOBILE (< 860px):
Position: bottom sheet (drag up/down)
Height: collapsed = 160px (badge + title + subtitle visible)
         expanded = max 90vh (full info, scrollable)
Border-radius: 18px 18px 0 0
Drag handle: 36×4px bar, top center, gold when dragging
Full-width (left:0, right:0, bottom:0)

DRAG SYSTEM (2 sources):
1. Drag-bar: free drag, direction-based snap (20px threshold)
   - Kéo lên ≥20px → expand
   - Kéo xuống ≥20px → collapse
   - Kéo <20px → giữ state cũ
2. Panel body: auto-snap with 15px gesture detection
   - Vuốt xuống 15px → collapse (no free drag)
   - Vuốt lên 15px khi collapsed → expand
   - Vuốt lên khi already expanded → cancel, allow scroll

Animation: requestAnimationFrame pattern
  - Remove .dragging (transition:none) 
  - rAF → set inline transition + target transform
  - transitionend → clean up, set class

Sibling hiding:
  - .expanded ~ .scene-nav → opacity:0, pointer-events:none
  - .expanded ~ .location-pill → opacity:0, pointer-events:none
  - Same for .dragging state
```

### 3.3 SceneNavigator (Bottom Thumbnail Bar)

```
┌──────────────────────────────────────────────────┐
│  [🔲 Bedroom]  [🔲 Balcony]  [🔲 Bathroom]      │
│   (active)                                        │
└──────────────────────────────────────────────────┘

Position: bottom center, above CTA
Layout: horizontal scroll on mobile, centered on desktop
Thumbnail: 80×60px, rounded, border khi active
Label: dưới thumbnail, 12px, uppercase
Active state: gold border 2px, slight scale up
Transition: smooth scroll, 300ms ease

MOBILE:
- Horizontal swipeable
- Active centered with snap scroll
- Height: 80px total
```

### 3.4 HotspotMarker (In-Scene Markers)

```
TYPES:

1. NAVIGATE (chuyển scene)
   ┌─────┐
   │  →  │  Circle 40px, white bg, icon arrow
   └─────┘
   Hover: scale 1.1 + tooltip "Go to Balcony"
   Click: trigger scene transition

2. INFO (xem thông tin)
   ┌─────┐
   │  i  │  Circle 36px, semi-transparent bg
   └─────┘
   Hover: expand to pill shape "Ocean View"
   Click: open InfoCard (floating)
   
   InfoCard:
   ┌────────────────────────┐
   │ 📷 [Detail Image]      │  200×150
   │ Ocean View             │  bold, 16px
   │ Tầm nhìn 180° ra      │  14px, 2 lines max
   │ vịnh Nha Trang         │
   │ [×]                    │  close button
   └────────────────────────┘

3. CTA (call to action)
   ┌──────────────────┐
   │ 💰 Check Rate    │  Pill shape, gold bg
   └──────────────────┘
   Pulse animation khi idle
   Click: open booking widget/redirect

4. GALLERY (xem ảnh chi tiết)
   ┌─────┐
   │  📷 │  Circle 36px
   └─────┘
   Click: open gallery lightbox at specific image
```

### 3.5 BookingWidget (Floating CTA)

```
DESKTOP:
Position: bottom-right, fixed
┌──────────────────────────────┐
│  From 2,500,000 VND/night    │
│                               │
│  [Check Rate] [Book Now →]   │
└──────────────────────────────┘
Background: var(--color-bg-panel) + blur
Always visible (z-50)

EXPANDED STATE (click "Check Rate"):
┌──────────────────────────────┐
│  CHECK AVAILABILITY           │
│                               │
│  Check-in:   [📅 Date]       │
│  Check-out:  [📅 Date]       │
│  Adults:     [2 ▼]           │
│  Children:   [0 ▼]           │
│                               │
│  [Search Availability]       │
│                               │
│  OR                           │
│  [Book Direct →]  [Enquire]  │
└──────────────────────────────┘
Animation: expand up from bottom-right

MOBILE:
Position: fixed bottom, full width
Height: 56px collapsed (just "Book Now" bar)
Swipe up: expanded form
```

### 3.6 GalleryOverlay (Lightbox)

```
┌──────────────────────────────────────────────────┐
│ [×]                                    1/12       │
│                                                    │
│      ┌─────────────────────────────────────┐      │
│      │                                     │      │
│  [<] │         FULL-SIZE IMAGE             │  [>] │
│      │                                     │      │
│      └─────────────────────────────────────┘      │
│                                                    │
│  "Ocean view from the private balcony"             │
│                                                    │
│  ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐       │
│  │  │ │  │ │  │ │  │ │  │ │  │ │  │ │  │       │
│  └──┘ └──┘ └──┘ └──┘ └──┘ └──┘ └──┘ └──┘       │
└──────────────────────────────────────────────────┘

Background: rgba(0,0,0,0.95)
Z-index: var(--z-modal)
Animation: fade in 300ms + image scale from 0.8 to 1
Navigation: swipe (mobile) hoặc arrow keys (desktop)
Thumbnail strip: horizontal scroll, active highlighted
Close: × button, Escape key, click outside
```

---

## 4. Page-by-Page UX Flow

### 4.1 Homepage

```
FLOW:
1. Load → Hero section (full-viewport video/slideshow hoặc 360 lobby scene)
2. Scroll → Hotel introduction (short, elegant)
3. Scroll → Featured Rooms (3 cards với panorama preview thumbnail)
4. Scroll → Services highlights (Dining, Spa, Pool — grid layout)
5. Scroll → Current Promotions (carousel)
6. Scroll → News / Blog preview (2-3 cards)
7. Scroll → Footer

HERO OPTIONS:
Option A — 360 Lobby: Khách "đứng" trong lobby, có hotspots đến các hạng mục
Option B — Video: Cinematic hotel video, autoplay muted
Option C — Image Slideshow: 4-6 ảnh đẹp nhất, crossfade

FEATURED ROOMS:
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ [Panorama    │ │ [Panorama    │ │ [Panorama    │
│  Preview     │ │  Preview     │ │  Preview     │
│  (mini 360)] │ │  (mini 360)] │ │  (mini 360)] │
│──────────────│ │──────────────│ │──────────────│
│ Deluxe Twin  │ │ Pacific Dbl  │ │ Suite Ocean  │
│ Ocean View   │ │ Balcony Bath │ │ View         │
│ From 2.5M    │ │ From 3.8M    │ │ From 5.5M    │
│ [Explore →]  │ │ [Explore →]  │ │ [Explore →]  │
└──────────────┘ └──────────────┘ └──────────────┘
```

### 4.2 Room Listing Page (/rooms)

```
FLOW:
1. Load → Header banner (panorama preview hoặc ảnh đại diện)
2. Room cards grid (2 columns desktop, 1 column mobile)
3. Each card: thumbnail + name + highlights + price + CTA

ROOM CARD:
┌─────────────────────────────────────────────────┐
│ [Room Thumbnail / Mini 360 Preview     ]         │
│                                                   │
│  ★ Deluxe Twin Room With Ocean View              │
│  35m² · Ocean View · Balcony                     │
│                                                   │
│  Từ 2,500,000 VND/đêm                           │
│                                                   │
│  [Explore 360 →]              [Book Now]         │
└─────────────────────────────────────────────────┘

Click "Explore 360" → Navigate to /rooms/deluxe-ocean (360-first page)
```

### 4.3 Room Detail Page (/rooms/:slug) — 360-FIRST

```
FLOW:
1. Load → Full-screen panorama 360 (default scene)
   - Loading: blur placeholder → spinner → fade in panorama
   - Auto-rotate slow (0.5 deg/frame)
   
2. After 1.5s → InfoPanel slides in from left
   - Tên phòng, badge, diện tích, mô tả ngắn
   - Amenities grid
   - Price + CTA

3. After 2s → SceneNavigator slides up from bottom
   - Thumbnails: Bedroom, Balcony, Bathroom, Entry

4. Hotspots appear after panorama loaded
   - Navigate hotspots (→ Balcony, → Bathroom)
   - Info hotspots (Ocean View detail, Bathtub detail)
   - CTA hotspot (Book Now)

5. User Interaction:
   a. Drag/touch panorama:
      - InfoPanel fades out (300ms)
      - Auto-rotate stops
      - Hotspots remain visible
   b. Stop interacting (4s idle):
      - InfoPanel fades back in (500ms)
      - Auto-rotate resumes (optional)
   c. Click navigate hotspot:
      - Scene transition (fade black → new scene → fade in)
      - InfoPanel updates if different area
      - SceneNavigator updates active thumbnail
   d. Click info hotspot:
      - InfoCard pops up near hotspot position
      - Shows detail image + description
   e. Click gallery (bottom bar):
      - GalleryOverlay opens
      - Photos of the room (lifestyle shots, details)
   f. Click "Book Now":
      - Redirect to booking engine OR
      - Open BookingWidget expanded form
   g. Toggle UI (☐ button):
      - Hide all overlays, full immersive mode
      - Show only small toggle button to bring UI back

6. Mobile-specific:
   - InfoPanel = bottom sheet (swipe up)
   - SceneNavigator = horizontal swipe
   - Hotspot tap = open card, tap again = close
   - Pinch zoom on panorama
```

### 4.4 Service Pages (Dining, Spa, Pool, Meeting)

Giống room page nhưng với CTA khác:

| Service | CTA Primary | CTA Secondary |
|---------|------------|---------------|
| Dining | Reserve Table | View Menu |
| Spa | Book Treatment | Spa Menu |
| Pool | (none — info only) | Gallery |
| Meeting | Event Enquiry | View Packages |
| Beach | (none — info only) | Gallery |
| Fitness | (none — info only) | Operating Hours |

### 4.5 Promotions Page (/promotions)

```
Layout B (CMS content page)
- Card grid: image + title + dates + short desc + CTA
- Click → detail page with full description
- Related rooms/services linked
```

### 4.6 News / Blog (/news)

```
Layout B
- Card list: featured image + title + excerpt + date
- Click → full article page
- Sidebar: latest posts, categories
```

### 4.7 Contact Page (/contact)

```
Layout B
- Map embed (Google Maps)
- Contact info cards (phone, email, address)
- Contact form
- Social links
```

---

## 5. Responsive Breakpoints

| Breakpoint | Width | Layout Changes |
|-----------|-------|---------------|
| Mobile S | < 375px | Compact spacing, stack all |
| Mobile | 375–860px | Bottom sheet InfoPanel, swipe scenes, hamburger menu |
| Desktop S | 860–1100px | Side panel narrower (340px), right:16px |
| Desktop | 1100–1439px | Full side panel (380px), comfortable spacing |
| Desktop L | ≥ 1440px | Max-width container, larger panels |

### 5.1 Mobile-Specific Patterns

```
BOTTOM SHEET (InfoPanel on mobile ≤860px):
┌──────────────────────────────┐
│         ═══  (drag handle)    │  ← 36×4px, gold khi dragging
│                                │
│ ★ DELUXE TWIN ROOM            │  ← badge + title
│ Ocean View                     │  ← subtitle
│ From 2,500,000 VND             │
│ [Book Now]  [Check Rate]       │
│                                │  ← Collapsed: 160px visible
├── ─ ─ ─ (swipe up) ─ ─ ─ ─ ─┤
│                                │
│ Mô tả phòng...                │
│                                │
│ 35m²  ·  2 Adults  ·  Twin    │  ← meta row with icons
│                                │
│ Amenities:                     │
│ ☐ Ocean ☐ Balcony ☐ WiFi     │
│ ☐ AC    ☐ Bathtub ☐ Minibar  │
│                                │
│ [Book Now] [Check Avail]       │
│ [← Xem phòng khác]            │
│                                │  ← Expanded: max-height 90vh
└──────────────────────────────┘

CSS States:
  .content-panel (collapsed): transform:translateY(calc(100% - 160px))
  .content-panel.expanded:    transform:translateY(0)
  .content-panel.dragging:    transition:none!important (follow finger)
  .content-panel.slide-out:   transform:translateY(100%); opacity:0

DRAG SYSTEM (2 dragSources):

1. Drag-bar ("bar"):
   - Trigger: mousedown/touchstart on .panel-drag
   - Behavior: free-follow finger movement
   - Snap: DIRECTION-BASED (20px threshold)
     · currentY - startTranslate < -20 → expand (translateY=0)
     · currentY - startTranslate > 20 → collapse
     · |delta| < 20 → restore previous state
   - Clamped between 0 and collapsedTranslate

2. Panel body ("panel"):
   - Trigger: mousedown/touchstart on .panel-scroll (when scrollTop=0)
   - Behavior: AUTO-SNAP (15px gesture detection, NO free drag)
     · |delta| < 15 → do nothing (wait for more movement)
     · delta < -15 (swipe up):
       - If already expanded (startTranslate≤0): cancelDrag(), allow scroll
       - Else: snap expand
     · delta > 15 (swipe down): snap collapse
   - Does not follow finger — detects direction then snaps immediately

Animation pattern (both sources):
  1. panel.classList.remove("dragging")   // re-enable transitions
  2. requestAnimationFrame(() => {
       panel.style.transition = "transform .35s cubic-bezier(.16,1,.3,1)"
       panel.style.transform = "translateY(" + targetY + "px)"
       // settled guard flag prevents double-fire
       panel.addEventListener("transitionend", onDone)
       setTimeout(onDone, 400)  // fallback
     })
  3. onDone: remove inline styles, toggle .expanded class

Anti-flash on drag start (from expanded state):
  1. getCurrentTranslateY() via getComputedStyle → matrix parse
  2. Set inline transform BEFORE removing .expanded class
  3. Then remove .expanded, add .dragging
  → No visual jump when switching from class-based to inline transform

Event binding: MutationObserver on document.body
  → auto-bind mousedown/touchstart when panel appears in DOM
  → _dragBound flag prevents duplicate binding

Sibling element hiding during drag/expand:
  .content-panel.expanded ~ .scene-nav { opacity:0; pointer-events:none }
  .content-panel.expanded ~ .location-pill { opacity:0; pointer-events:none }
  .content-panel.dragging ~ .scene-nav { opacity:0; pointer-events:none }
  .content-panel.dragging ~ .location-pill { opacity:0; pointer-events:none }

GESTURE MAP:
- Drag panorama: rotate 360 view, triggers Smart UI hide
- Pinch: zoom in/out (Pannellum built-in)
- Drag-bar swipe: free drag + direction-based snap (20px)
- Panel body swipe (at scroll top): auto-snap (15px detection)
- Tap hotspot: show info card
- Click nav hotspot: navigate to target scene
- Hamburger: open mobile menu overlay
```

---

## 6. Animation & Transition Spec

### 6.1 Page Transitions

| Transition | Animation | Duration |
|-----------|-----------|---------|
| Page → 360 Page | Crossfade + zoom in subtle | 500ms |
| 360 Page → Page | Crossfade + zoom out subtle | 400ms |
| Scene → Scene (in 360) | Fade to black → fade in | 300ms + 500ms |

### 6.2 Component Animations

| Component | Animation | Trigger |
|-----------|-----------|--------|
| InfoPanel (desktop) | slideInFromRight / fadeOut | Page load / user interaction |
| InfoPanel slide-out | translateX(30px) + opacity→0 | Scene transition |
| SceneNavigator | slideUpFromBottom | Page load (delay 500ms) |
| Hotspots | fadeIn + pulse (3s loop) | Panorama loaded |
| InfoCard (hotspot) | scaleIn from click origin | Hotspot click |
| GalleryOverlay | fadeIn + scaleFrom(0.9) | Gallery button click |
| BookingWidget expand | expandUp + fadeIn children | CTA click |
| UI Layer hide/show | opacity .4s ease | Smart UI interaction/idle |
| Bottom Sheet drag | transform follow finger (no transition) | Touch/mouse drag |
| Bottom Sheet snap | transform .35s cubic-bezier(.16,1,.3,1) | Drag end via rAF |
| Panel drag-bar | width 36→48px, color→gold | Dragging state |
| Mobile Menu overlay | opacity .4s | Hamburger toggle |
| Mobile Menu links | translateX(-20→0) + opacity, stagger 80+i*60ms | Menu open |
| Mobile Menu CTA | translateX(-20→0) + opacity, delay 400ms | Menu open |
| Mobile Menu footer | translateY(10→0) + opacity, delay 200ms | Menu open |
| Mobile Menu arrow | opacity 0→1, translateX(-8→0) | Link hover |
| Loader | opacity→0, visibility→hidden (.8s) | Panorama loaded |

### 6.3 Loading States

```
PANORAMA LOADING:
1. Tiny blurred placeholder image (< 5KB) → instant show
2. Animated loading ring (SVG, center)
3. Full panorama loaded → crossfade (500ms)
4. Hotspots animate in (staggered, 100ms delay each)

PAGE SKELETON:
- InfoPanel: shimmer placeholder blocks
- SceneNavigator: shimmer thumbnail shapes
- Text: shimmer lines
```

---

## Sequence Diagrams

### SD-1. Room Detail Page — User Journey

```
  Visitor            Browser/Next.js      Pannellum          InfoPanel          SceneNav
   │                     │                │                  │                  │
   │  visit /rooms/      │                │                  │                  │
   │  deluxe-ocean       │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ SSR render     │                  │                  │
   │                     │ (SEO meta,     │                  │                  │
   │                     │  room data,    │                  │                  │
   │                     │  JSON-LD)      │                  │                  │
   │ ◄─── HTML + hydrate ┤                │                  │                  │
   │                     │                │                  │                  │
   │                     │ init viewer    │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ load panorama    │                  │
   │                     │                │ render hotspots  │                  │
   │                     │                │                  │                  │
   │                     │ mount panel    │                  │                  │
   │                     ├──────────────────────────────────►│                  │
   │                     │                │                  │ room name/badge  │
   │                     │                │                  │ description      │
   │                     │                │                  │ amenities grid   │
   │                     │                │                  │ price + CTA      │
   │                     │                │                  │ slideInFromRight │
   │                     │                │                  │                  │
   │                     │ mount scene nav│                  │                  │
   │                     ├─────────────────────────────────────────────────────►│
   │                     │                │                  │                  │ [Bedroom]
   │                     │                │                  │                  │ [Balcony]
   │  ◄──────────────────── page ready ───┤                  │                  │
   │                     │                │                  │                  │
   │  drag panorama      │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ rotate view      │                  │
   │                     │ SmartUI ─ hide │                  │                  │
   │                     ├──────────────────────────────────►│ fade out 300ms   │
   │                     ├─────────────────────────────────────────────────────►│ fade out
   │                     │                │                  │                  │
   │  stop + idle 4s     │                │                  │                  │
   │                     │ SmartUI ─ show │                  │                  │
   │                     ├──────────────────────────────────►│ fade in 500ms    │
   │                     ├─────────────────────────────────────────────────────►│ fade in
   │                     │                │                  │                  │
   │  click hotspot      │                │                  │                  │
   │  "Ban công →"       │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ scene change     │                  │
   │                     │                │ fade → balcony   │                  │
   │                     │                │                  │                  │
   │                     │ update panel   │                  │                  │
   │                     ├──────────────────────────────────►│ keep room info   │
   │                     ├─────────────────────────────────────────────────────►│ highlight
   │                     │                │                  │                  │ [Balcony]
   │                     │                │                  │                  │
   │  click "Book Now"   │                │                  │                  │
   ├────────────────────────────────────────────────────────►│                  │
   │                     │                │                  │ track analytics  │
   │                     │                │                  │ open booking URL │
   │  ◄─────────────────── redirect to booking engine ───────┤                  │
```

### SD-2. Mobile Bottom Sheet — Complete Gesture Flow

```
  User Touch       ScrollTop Check     Drag System         Panel Animation    Sibling UI
   │                     │                │                  │                  │
   │  touch .panel-drag  │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ source = "bar"   │                  │
   │                     │                │ free-follow mode │                  │
   │                     │                │                  │                  │
   │  move finger        │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ set transform ──►│ follow finger    │
   │                     │                │                  │                  │
   │                     │                │ hide siblings ─────────────────────►│
   │                     │                │                  │                  │ .scene-nav
   │                     │                │                  │                  │ .location-pill
   │                     │                │                  │                  │ opacity → 0
   │  release            │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ direction snap   │                  │
   │                     │                │ (20px threshold) │                  │
   │                     │                │ rAF → animate ──►│ .35s ease-out    │
   │                     │                │                  │                  │
   │                     │                │ show/hide siblings ────────────────►│
   │                     │                │                  │                  │ based on
   │                     │                │                  │                  │ expand/
   │◄───────────────────── settled ───────┤                  │                  │ collapse
   │                     │                │                  │                  │
   ═══════════════════════════════════════════════════════════════════════════════
   │                     │                │                  │                  │
   │  touch .panel-scroll│                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ scrollTop = 0? │                  │                  │
   │                     │ YES ──────────►│ source = "panel" │                  │
   │                     │                │ auto-snap mode   │                  │
   │                     │                │                  │                  │
   │  swipe up 15px+     │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ if collapsed:    │                  │
   │                     │                │ → EXPAND ───────►│ snap to top      │
   │                     │                │                  │                  │
   │                     │                │ if expanded:     │                  │
   │                     │                │ → cancelDrag()   │                  │
   │                     │                │   let scroll ───►│ normal scroll    │
   │                     │                │                  │                  │
   │  swipe down 15px+   │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ → COLLAPSE ─────►│ snap to bottom   │
   │◄───────────────────── settled ───────┤                  │                  │
```

### SD-3. Booking Widget Interaction

```
  Visitor            InfoPanel            BookingWidget       Booking Engine    Analytics
   │                     │                │                  │                  │
   │  view room info     │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ show price:    │                  │                  │
   │                     │ "From X VND"   │                  │                  │
   │                     │ [Book Now]     │                  │                  │
   │                     │ [Check Rate]   │                  │                  │
   │                     │                │                  │                  │
   │  click "Book Now"   │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ track event ─────────────────────────────────────────►
   │                     │                │                  │                  │ booking_click
   │                     │                │                  │                  │ room: slug
   │                     │                │                  │                  │ source: panel
   │                     │                │                  │                  │
   │                     │  ─── Option A: Direct redirect ──────────────────────│
   │                     │                │                  │                  │
   │                     │ window.open() ────────────────────►                  │
   │  ◄──────────────────── new tab: booking engine ─────────┤                  │
   │                     │                │                  │                  │
   │                     │  ─── Option B: Inline widget expand ─────────────────│
   │                     │                │                  │                  │
   │  click "Check Rate" │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ expand widget  │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ expandUp anim    │                  │
   │                     │                │ fadeIn children  │                  │
   │                     │                │ show:            │                  │
   │                     │                │  📅 date picker  │                  │
   │                     │                │  👤 guest count  │                  │
   │                     │                │  [Submit]        │                  │
   │                     │                │                  │                  │
   │  fill form & submit │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ validate         │                  │
   │                     │                │ redirect ────────►                  │
   │  ◄──────────────────── booking engine with params ──────┤                  │
```

### SD-4. Scene Transition Animation Timeline

```
  User Action        Panel              Pannellum          Hotspots           SceneNav
   │                     │                │                  │                  │
   │  click scene or     │                │                  │                  │
   │  navigate hotspot   │                │                  │                  │
   │                     │                │                  │                  │
   │  t=0ms              │                │                  │                  │
   │                     │ slide-out      │                  │                  │
   │                     │ translateX(30) │                  │                  │
   │                     │ opacity → 0    │                  │                  │
   │                     │ (500ms)        │                  │                  │
   │                     │                │                  │                  │
   │  t=300ms            │                │                  │                  │
   │                     │                │ fade to black    │                  │
   │                     │                │ (300ms)          │                  │
   │                     │                │                  │ remove all       │
   │                     │                │                  │                  │
   │  t=500ms            │                │                  │                  │
   │                     │                │ load new scene   │                  │
   │                     │                │ fade in (500ms)  │                  │
   │                     │                │                  │                  │
   │  t=700ms            │                │                  │                  │
   │                     │ slide-in       │                  │                  │
   │                     │ translateX(0)  │                  │                  │
   │                     │ opacity → 1    │                  │                  │
   │                     │ (500ms)        │                  │                  │
   │                     │                │                  │                  │
   │  t=1000ms           │                │                  │                  │
   │                     │                │                  │ fadeIn + pulse   │
   │                     │                │                  │ stagger 100ms ea │
   │                     │                │                  │                  │
   │                     │                │                  │                  ├── update
   │                     │                │                  │                  │   active
   │                     │                │                  │                  │   thumbnail
   │  t=1200ms           │                │                  │                  │
   │  ◄──────────────────── transition complete ─────────────────────────────   │
```

---

## 7. Accessibility (WCAG 2.1 AA)

| Requirement | Implementation |
|------------|---------------|
| Keyboard navigation | Tab through hotspots, Enter to activate, Escape to close overlays |
| Screen reader | ARIA labels cho hotspots, panels, scene transitions |
| Color contrast | 4.5:1 minimum cho text trên overlay backgrounds |
| Reduced motion | `prefers-reduced-motion`: disable auto-rotate, fade instead of slide |
| Focus visible | Gold outline ring cho focused elements |
| Alt text | Mọi ảnh gallery có alt text từ CMS |
| Language | `lang` attribute chuyển theo locale |

---

## 8. Iconography

Sử dụng **Lucide Icons** (consistent, clean, MIT license).

| Icon | Dùng cho |
|------|---------|
| `eye` | View / 360 toggle |
| `maximize-2` | Fullscreen |
| `minimize-2` | Exit fullscreen |
| `chevron-left/right` | Gallery navigation |
| `x` | Close |
| `info` | Info hotspot |
| `navigation` | Navigate hotspot |
| `calendar` | Check-in/out |
| `users` | Adults/children |
| `wifi` | WiFi amenity |
| `snowflake` | AC amenity |
| `bath` | Bathtub amenity |
| `waves` | Ocean view |
| `mountain` | Mountain view |
| `bed-double` | Bed type |
| `ruler` | Area |
| `phone` | Contact |
| `mail` | Email |
| `map-pin` | Location |
| `share-2` | Share |
| `globe` | Language |
| `menu` | Mobile hamburger |

---

*Tài liệu tiếp theo: **04-TECHNICAL-IMPLEMENTATION.md** — React component tree, state management, 360 integration code*
