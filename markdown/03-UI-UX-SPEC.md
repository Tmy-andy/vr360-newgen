# 03 — UI/UX DESIGN SPECIFICATION
## Website 360 Thế Hệ Mới — Hotel CMS Platform

**Version:** 1.0  
**Date:** 2026-03-10  
**Related:** 01-SYSTEM-ARCHITECTURE.md, 02-DATABASE-API-SPEC.md

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
  /* Colors - Luxury Dark Theme cho 360 pages */
  --color-primary:      #1B2D4F;     /* Deep navy */
  --color-accent:       #C8A96E;     /* Gold accent */
  --color-accent-hover: #D4BA85;
  --color-bg-dark:      #0D1117;
  --color-bg-overlay:   rgba(13, 17, 23, 0.75);
  --color-bg-panel:     rgba(27, 45, 79, 0.85);
  --color-bg-card:      rgba(255, 255, 255, 0.08);
  --color-text-primary: #FFFFFF;
  --color-text-secondary: rgba(255, 255, 255, 0.7);
  --color-text-muted:   rgba(255, 255, 255, 0.45);
  --color-border:       rgba(200, 169, 110, 0.2);
  --color-border-active: rgba(200, 169, 110, 0.5);

  /* Colors - Light Theme cho CMS pages (News, Contact...) */
  --color-light-bg:     #FAFAF8;
  --color-light-text:   #1B2D4F;
  --color-light-muted:  #6B7280;
  --color-light-border: #E5E7EB;

  /* Typography */
  --font-display:   'Playfair Display', Georgia, serif;
  --font-body:      'DM Sans', 'Segoe UI', sans-serif;
  --font-mono:      'JetBrains Mono', monospace;

  /* Spacing Scale (8px base) */
  --space-1: 4px;   --space-2: 8px;   --space-3: 12px;
  --space-4: 16px;  --space-5: 20px;  --space-6: 24px;
  --space-8: 32px;  --space-10: 40px; --space-12: 48px;
  --space-16: 64px; --space-20: 80px;

  /* Border Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-soft:  0 4px 24px rgba(0,0,0,0.15);
  --shadow-card:  0 8px 32px rgba(0,0,0,0.25);
  --shadow-modal: 0 16px 64px rgba(0,0,0,0.4);

  /* Transitions */
  --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
  --duration-fast: 200ms;
  --duration-normal: 350ms;
  --duration-slow: 600ms;

  /* Z-Index Scale */
  --z-panorama:   0;
  --z-hotspot:    10;
  --z-overlay:    20;
  --z-panel:      30;
  --z-nav:        40;
  --z-cta:        50;
  --z-modal:      60;
  --z-toast:      70;
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
│ State: MOBILE (< 768px)                                      │
│ Hamburger menu → Full-screen overlay                         │
│ Menu items: stacked, animated stagger in                     │
│ Language selector visible                                    │
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

MOBILE (< 768px):
Position: bottom sheet (drag up/down)
Height: collapsed = 120px (tên + price + CTA)
         expanded = 70vh (full info)
Drag handle ở top center
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
| Mobile | 375–767px | Bottom sheet InfoPanel, swipe scenes |
| Tablet | 768–1023px | Side panel narrower (280px), 2-col grids |
| Desktop | 1024–1439px | Full side panel (360px), comfortable spacing |
| Desktop L | ≥ 1440px | Max-width container (1400px), larger panels |

### 5.1 Mobile-Specific Patterns

```
BOTTOM SHEET (InfoPanel on mobile):
┌──────────────────────────────┐
│         ═══  (drag handle)    │  ← 40px tap target
│                                │
│ Deluxe Twin Room · Ocean View │
│ From 2,500,000 VND            │
│ [Book Now]  [Check Rate]      │
│                                │  ← Collapsed: 120px
├── ─ ─ ─ (swipe up) ─ ─ ─ ─ ─┤
│                                │
│ Mô tả phòng...                │
│                                │
│ Amenities:                     │
│ ☐ Ocean ☐ Balcony ☐ WiFi     │
│ ☐ AC    ☐ Bathtub ☐ Minibar  │
│                                │
│ Gallery: [thumb] [thumb] ...   │
│                                │  ← Expanded: 70vh
└──────────────────────────────┘

GESTURE MAP:
- Tap panorama: toggle UI visibility
- Drag panorama: rotate 360 view
- Pinch: zoom in/out
- Swipe up on bottom sheet: expand info
- Swipe down on bottom sheet: collapse
- Tap hotspot: show info card
- Double-tap hotspot: navigate (if navigate type)
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
| InfoPanel | slideInFromLeft / fadeOut | Page load / user interaction |
| SceneNavigator | slideUpFromBottom | Page load (delay 500ms) |
| Hotspots | fadeIn + pulse | Panorama loaded |
| InfoCard (hotspot) | scaleIn from hotspot origin | Hotspot click |
| GalleryOverlay | fadeIn + scaleFrom(0.9) | Gallery button click |
| BookingWidget expand | expandUp + fadeIn children | CTA click |
| Navbar hide/show | translateY(-100%) / translateY(0) | Scroll / idle |
| Bottom Sheet (mobile) | spring physics (swipe gesture) | Swipe up/down |
| Room Card hover | image slight zoom + shadow deepen | Mouse enter |

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
