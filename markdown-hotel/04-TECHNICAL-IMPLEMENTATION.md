# 04 — TECHNICAL IMPLEMENTATION SPECIFICATION
## Website 360 Thế Hệ Mới — React / Next.js

**Version:** 1.2  
**Date:** 2026-03-10  
**Updated:** Thêm sequence diagrams (Client Hydration, Scene Change Runtime, Analytics, Hotspot Editor)  
**Related:** 01-SYSTEM-ARCHITECTURE · 02-DATABASE-API-SPEC · 03-UI-UX-SPEC · 05-DEMO-HTML-SPEC

---

## Mục Lục

1. [Project Structure](#1-project-structure-nextjs-app-router)
2. [Core Component Specifications](#2-core-component-specifications)
3. [Data Flow: Room 360 Page](#3-data-flow-room-360-page-chi-tiết)
4. [API Client & Data Fetching](#4-api-client--data-fetching)
5. [Scene Mapping Strategy](#5-scene-mapping-strategy)
6. [Performance Optimization](#6-performance-optimization)
7. [Analytics Integration](#7-analytics-integration)
8. [Admin Dashboard — Hotspot Visual Editor](#8-admin-dashboard--hotspot-visual-editor)
9. [Demo HTML Implementation (Phase 0)](#9-demo-html-implementation-phase-0)
10. [Sequence Diagrams](#10-sequence-diagrams)

---

## 1. Project Structure (Next.js App Router)

```
hotel-360/
├── .env.local                    # Environment variables
├── .env.example
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── prisma/
│   ├── schema.prisma             # Database schema
│   └── migrations/
│       └── ...
├── public/
│   ├── fonts/
│   │   ├── PlayfairDisplay-*.woff2
│   │   └── DMSans-*.woff2
│   ├── icons/
│   └── images/
│       └── placeholder-blur.jpg
│
├── src/
│   ├── app/                      # Next.js App Router
│   │   ├── layout.tsx            # Root layout (fonts, metadata, providers)
│   │   ├── page.tsx              # Homepage
│   │   ├── globals.css           # Design tokens, Tailwind base
│   │   │
│   │   ├── (marketing)/          # Layout group: CMS content pages
│   │   │   ├── layout.tsx        # Header solid + Footer
│   │   │   ├── promotions/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [slug]/page.tsx
│   │   │   ├── news/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [slug]/page.tsx
│   │   │   ├── contact/page.tsx
│   │   │   ├── gallery/page.tsx
│   │   │   └── videos/page.tsx
│   │   │
│   │   ├── (immersive)/          # Layout group: 360-first pages
│   │   │   ├── layout.tsx        # Transparent header, no footer
│   │   │   ├── rooms/
│   │   │   │   ├── page.tsx      # Room listing
│   │   │   │   └── [slug]/
│   │   │   │       └── page.tsx  # ★ 360-first room page
│   │   │   ├── dining/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [slug]/page.tsx
│   │   │   ├── recreation/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [slug]/page.tsx
│   │   │   └── meetings/
│   │   │       └── page.tsx
│   │   │
│   │   ├── api/                  # API Routes
│   │   │   └── v1/
│   │   │       ├── hotel/route.ts
│   │   │       ├── rooms/
│   │   │       │   ├── route.ts
│   │   │       │   └── [slug]/route.ts
│   │   │       ├── services/
│   │   │       │   ├── route.ts
│   │   │       │   └── [slug]/route.ts
│   │   │       ├── promotions/route.ts
│   │   │       ├── posts/route.ts
│   │   │       ├── scenes/route.ts
│   │   │       ├── analytics/route.ts
│   │   │       └── admin/
│   │   │           ├── auth/route.ts
│   │   │           ├── rooms/route.ts
│   │   │           ├── services/route.ts
│   │   │           ├── scenes/route.ts
│   │   │           ├── hotspots/route.ts
│   │   │           ├── upload/route.ts
│   │   │           └── ...
│   │   │
│   │   └── admin/                # Admin Dashboard (separate layout)
│   │       ├── layout.tsx
│   │       ├── page.tsx          # Dashboard overview
│   │       ├── rooms/
│   │       ├── services/
│   │       ├── scenes/
│   │       ├── promotions/
│   │       ├── posts/
│   │       ├── menus/
│   │       ├── translations/
│   │       ├── settings/
│   │       └── analytics/
│   │
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Navbar.tsx
│   │   │   ├── NavbarTransparent.tsx
│   │   │   ├── Footer.tsx
│   │   │   ├── MobileMenu.tsx
│   │   │   ├── LanguageSwitcher.tsx
│   │   │   └── BookingBar.tsx
│   │   │
│   │   ├── panorama/             # ★ 360 Core Components
│   │   │   ├── PanoramaViewer.tsx
│   │   │   ├── PanoramaLoader.tsx
│   │   │   ├── SceneNavigator.tsx
│   │   │   ├── HotspotMarker.tsx
│   │   │   ├── HotspotInfoCard.tsx
│   │   │   ├── HotspotCTA.tsx
│   │   │   └── SceneTransition.tsx
│   │   │
│   │   ├── overlay/              # ★ Overlay UI Components
│   │   │   ├── InfoPanel.tsx
│   │   │   ├── InfoPanelMobile.tsx   # Bottom sheet variant
│   │   │   ├── FloatingCTA.tsx
│   │   │   ├── BookingWidget.tsx
│   │   │   ├── GalleryOverlay.tsx
│   │   │   ├── UIToggle.tsx
│   │   │   └── SmartUIController.tsx
│   │   │
│   │   ├── rooms/
│   │   │   ├── RoomCard.tsx
│   │   │   ├── RoomGrid.tsx
│   │   │   ├── AmenityBadge.tsx
│   │   │   └── PriceDisplay.tsx
│   │   │
│   │   ├── services/
│   │   │   ├── ServiceCard.tsx
│   │   │   └── ServiceGrid.tsx
│   │   │
│   │   ├── content/
│   │   │   ├── PromotionCard.tsx
│   │   │   ├── PostCard.tsx
│   │   │   ├── HeroSection.tsx
│   │   │   ├── SectionHeading.tsx
│   │   │   └── RichContent.tsx
│   │   │
│   │   ├── ui/                   # Shared UI primitives
│   │   │   ├── Button.tsx
│   │   │   ├── Badge.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Skeleton.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Drawer.tsx
│   │   │   ├── BottomSheet.tsx
│   │   │   ├── Tooltip.tsx
│   │   │   ├── ImageOptimized.tsx
│   │   │   └── LoadingSpinner.tsx
│   │   │
│   │   └── admin/                # Admin Dashboard components
│   │       ├── Sidebar.tsx
│   │       ├── DataTable.tsx
│   │       ├── FormBuilder.tsx
│   │       ├── ImageUploader.tsx
│   │       ├── PanoramaUploader.tsx
│   │       ├── HotspotEditor.tsx   # Visual hotspot placement
│   │       ├── SceneConfigEditor.tsx
│   │       ├── TranslationEditor.tsx
│   │       └── AnalyticsCharts.tsx
│   │
│   ├── hooks/
│   │   ├── usePanorama.ts        # Pannellum instance management
│   │   ├── useSmartUI.ts         # Auto show/hide UI logic
│   │   ├── useSceneTransition.ts # Scene change animation
│   │   ├── useIdleTimer.ts       # Detect user idle
│   │   ├── useBreakpoint.ts      # Responsive breakpoint
│   │   ├── useBottomSheet.ts     # Mobile bottom sheet gesture
│   │   ├── useAnalytics.ts       # Track events
│   │   └── useTranslation.ts     # i18n helper
│   │
│   ├── stores/
│   │   ├── sceneStore.ts         # Current scene, loading state
│   │   ├── uiStore.ts            # Panel visibility, UI mode
│   │   ├── bookingStore.ts       # Booking form state
│   │   └── adminStore.ts         # Admin dashboard state
│   │
│   ├── lib/
│   │   ├── api.ts                # API client (fetch wrapper)
│   │   ├── prisma.ts             # Prisma client singleton
│   │   ├── auth.ts               # JWT utils
│   │   ├── cache.ts              # Redis client
│   │   ├── upload.ts             # S3 upload utils
│   │   ├── seo.ts                # SEO helpers (JSON-LD, sitemap)
│   │   ├── analytics.ts          # Analytics event sender
│   │   └── constants.ts          # App-wide constants
│   │
│   ├── types/
│   │   ├── hotel.ts
│   │   ├── room.ts
│   │   ├── service.ts
│   │   ├── scene.ts
│   │   ├── hotspot.ts
│   │   ├── promotion.ts
│   │   ├── post.ts
│   │   └── api.ts                # API response types
│   │
│   └── utils/
│       ├── formatPrice.ts
│       ├── formatDate.ts
│       ├── slugify.ts
│       └── cn.ts                 # className merger (clsx + twMerge)
│
└── scripts/
    ├── seed.ts                   # Database seeder
    └── migrate.ts
```

---

## 2. Core Component Specifications

### 2.1 PanoramaViewer — Pannellum Integration

```typescript
// src/components/panorama/PanoramaViewer.tsx

import { useEffect, useRef, useCallback } from 'react';
import { useSceneStore } from '@/stores/sceneStore';
import { useUIStore } from '@/stores/uiStore';

interface PanoramaViewerProps {
  scenes: Scene[];
  hotspots: Hotspot[];
  defaultSceneId: string;
  onSceneChange?: (sceneId: string) => void;
}

/**
 * INTEGRATION APPROACH:
 * 
 * Pannellum (https://pannellum.org/) được load qua CDN hoặc npm package.
 * 
 * Luồng hoạt động:
 * 1. Component mount → init pannellum.viewer() trên div ref
 * 2. Config multiScene nếu có nhiều scene
 * 3. Mỗi scene có hotspots riêng, được render bởi Pannellum
 * 4. Custom hotspot renderer cho UI đặc biệt (info cards, CTA)
 * 5. Event listeners: mousedown/touchstart → báo user interaction
 * 6. Idle timer: sau 4s → báo idle
 * 
 * Pannellum config structure:
 */
const pannellumConfig = {
  default: {
    firstScene: 'bedroom',
    autoLoad: true,
    showControls: false,        // Ẩn controls mặc định, dùng custom UI
    showFullscreenCtrl: false,
    friction: 0.15,             // Smooth dragging
    mouseZoom: true,
    touchPanSpeedCoeffFactor: 1,
    compass: false,
    sceneFadeDuration: 500,     // Scene transition 500ms
  },
  scenes: {
    bedroom: {
      panorama: 'https://cdn.../deluxe-ocean-bedroom.jpg',
      yaw: 0,
      pitch: 0,
      hfov: 100,
      autoRotate: 0.5,
      hotSpots: [
        {
          pitch: -5.2,
          yaw: 120.5,
          type: 'custom',       // Custom renderer
          cssClass: 'hotspot-navigate',
          createTooltipFunc: hotspotNavigateRenderer,
          createTooltipArgs: {
            targetScene: 'balcony',
            label: 'Balcony'
          },
          clickHandlerFunc: handleSceneChange,
          clickHandlerArgs: { sceneId: 'balcony' }
        },
        {
          pitch: 10.0,
          yaw: -45.3,
          type: 'custom',
          cssClass: 'hotspot-info',
          createTooltipFunc: hotspotInfoRenderer,
          createTooltipArgs: {
            label: 'Ocean View',
            description: 'Tầm nhìn 180° ra vịnh Nha Trang',
            imageUrl: 'https://cdn.../ocean-view-detail.jpg'
          }
        }
      ]
    },
    balcony: {
      panorama: 'https://cdn.../deluxe-ocean-balcony.jpg',
      yaw: -30,
      pitch: 5,
      hfov: 110,
      hotSpots: [/* ... */]
    },
    bathroom: {
      panorama: 'https://cdn.../deluxe-ocean-bathroom.jpg',
      // ...
    }
  }
};

/**
 * ALTERNATIVE: Photo Sphere Viewer (three.js based)
 * 
 * Nếu cần hiệu ứng transition phức tạp hơn (3D fade, zoom tunnel),
 * có thể dùng Photo Sphere Viewer (https://photo-sphere-viewer.js.org/)
 * với plugin architecture mạnh hơn.
 * 
 * Trade-off: nặng hơn (~200KB vs ~80KB), nhưng extensible hơn.
 */
```

### 2.2 SmartUIController — Auto Show/Hide Logic

```typescript
// src/hooks/useSmartUI.ts

import { useEffect, useCallback, useRef } from 'react';
import { useUIStore } from '@/stores/uiStore';

/**
 * LOGIC:
 * 
 * STATE MACHINE:
 * 
 *   ┌──────────┐  user interacts   ┌──────────┐
 *   │  VISIBLE  │ ────────────────> │  HIDDEN   │
 *   │  (panels  │                   │  (panels  │
 *   │  showing) │ <──────────────── │  hidden)  │
 *   └──────────┘  idle 4s timeout   └──────────┘
 *        │                               │
 *        │  user toggles UI              │
 *        ▼                               ▼
 *   ┌──────────┐                    ┌──────────┐
 *   │  FORCED   │                   │  FORCED   │
 *   │  HIDDEN   │ ← user clicks    │  VISIBLE  │
 *   │           │   toggle button   │           │
 *   └──────────┘                    └──────────┘
 * 
 * RULES:
 * 1. Khi user bắt đầu drag/touch panorama → hide panels (300ms fade)
 * 2. Khi user dừng tương tác 4s → show panels (500ms fade)
 * 3. Khi user click toggle button → force hide/show (override auto)
 * 4. Khi user click hotspot → show relevant info, keep panels hidden
 * 5. BookingCTA luôn visible (trừ forced hidden mode)
 * 6. SceneNavigator luôn visible (compact mode khi panels hidden)
 * 7. Navbar: ẩn khi user tương tác, hiện khi hover top 60px
 */

interface SmartUIConfig {
  idleTimeout: number;        // 4000ms default
  fadeOutDuration: number;    // 300ms
  fadeInDuration: number;     // 500ms
  keepVisibleElements: string[]; // ['booking-cta', 'scene-nav']
}

export function useSmartUI(config: SmartUIConfig) {
  const { setUIVisibility, uiMode } = useUIStore();
  const idleTimerRef = useRef<NodeJS.Timeout>();
  const isInteractingRef = useRef(false);

  const handleInteractionStart = useCallback(() => {
    if (uiMode === 'forced') return; // Don't auto-hide in forced mode
    
    isInteractingRef.current = true;
    clearTimeout(idleTimerRef.current);
    setUIVisibility('hidden');
  }, [uiMode]);

  const handleInteractionEnd = useCallback(() => {
    if (uiMode === 'forced') return;
    
    isInteractingRef.current = false;
    idleTimerRef.current = setTimeout(() => {
      if (!isInteractingRef.current) {
        setUIVisibility('visible');
      }
    }, config.idleTimeout);
  }, [uiMode, config.idleTimeout]);

  // Return handlers to attach to PanoramaViewer
  return {
    handleInteractionStart,
    handleInteractionEnd,
    toggleUI: () => setUIVisibility(
      uiMode === 'forced-hidden' ? 'forced-visible' : 'forced-hidden'
    ),
  };
}
```

### 2.3 State Management (Zustand)

```typescript
// src/stores/sceneStore.ts

import { create } from 'zustand';

interface SceneState {
  // Current state
  currentSceneId: string | null;
  isLoading: boolean;
  isTransitioning: boolean;
  loadProgress: number;           // 0-100
  
  // Scene data
  scenes: Scene[];
  hotspots: Record<string, Hotspot[]>;  // sceneId → hotspots
  
  // Actions
  setCurrentScene: (sceneId: string) => void;
  setLoading: (loading: boolean) => void;
  setTransitioning: (transitioning: boolean) => void;
  setLoadProgress: (progress: number) => void;
  initScenes: (scenes: Scene[], hotspots: Record<string, Hotspot[]>) => void;
}

export const useSceneStore = create<SceneState>((set) => ({
  currentSceneId: null,
  isLoading: true,
  isTransitioning: false,
  loadProgress: 0,
  scenes: [],
  hotspots: {},
  
  setCurrentScene: (sceneId) => set({ currentSceneId: sceneId }),
  setLoading: (isLoading) => set({ isLoading }),
  setTransitioning: (isTransitioning) => set({ isTransitioning }),
  setLoadProgress: (loadProgress) => set({ loadProgress }),
  initScenes: (scenes, hotspots) => set({ scenes, hotspots }),
}));


// src/stores/uiStore.ts

import { create } from 'zustand';

type UIVisibility = 'visible' | 'hidden' | 'forced-visible' | 'forced-hidden';
type PanelState = 'expanded' | 'collapsed' | 'hidden';

interface UIState {
  // Visibility
  uiVisibility: UIVisibility;
  navbarVisible: boolean;
  
  // Panels
  infoPanelState: PanelState;
  galleryOpen: boolean;
  bookingWidgetExpanded: boolean;
  activeHotspotCard: string | null;  // hotspot ID
  
  // Mobile
  bottomSheetHeight: number;  // 0-1 (percentage of viewport)
  
  // Actions
  setUIVisibility: (v: UIVisibility) => void;
  setNavbarVisible: (v: boolean) => void;
  setInfoPanelState: (s: PanelState) => void;
  toggleGallery: () => void;
  toggleBookingWidget: () => void;
  setActiveHotspotCard: (id: string | null) => void;
  setBottomSheetHeight: (h: number) => void;
}

export const useUIStore = create<UIState>((set) => ({
  uiVisibility: 'visible',
  navbarVisible: true,
  infoPanelState: 'expanded',
  galleryOpen: false,
  bookingWidgetExpanded: false,
  activeHotspotCard: null,
  bottomSheetHeight: 0.15,  // 15% default (collapsed)
  
  setUIVisibility: (uiVisibility) => set({ uiVisibility }),
  setNavbarVisible: (navbarVisible) => set({ navbarVisible }),
  setInfoPanelState: (infoPanelState) => set({ infoPanelState }),
  toggleGallery: () => set((s) => ({ galleryOpen: !s.galleryOpen })),
  toggleBookingWidget: () => set((s) => ({ 
    bookingWidgetExpanded: !s.bookingWidgetExpanded 
  })),
  setActiveHotspotCard: (activeHotspotCard) => set({ activeHotspotCard }),
  setBottomSheetHeight: (bottomSheetHeight) => set({ bottomSheetHeight }),
}));
```

---

## 3. Data Flow: Room 360 Page (Chi Tiết)

```
┌─────────────────────────────────────────────────────────┐
│ 1. SERVER-SIDE (Next.js generateStaticParams + SSG)      │
│                                                           │
│    // src/app/(immersive)/rooms/[slug]/page.tsx           │
│                                                           │
│    export async function generateStaticParams() {         │
│      const rooms = await prisma.room.findMany({           │
│        where: { isPublished: true },                      │
│        select: { slug: true }                             │
│      });                                                  │
│      return rooms.map(r => ({ slug: r.slug }));           │
│    }                                                      │
│                                                           │
│    export default async function RoomPage({ params }) {   │
│      const room = await getRoomWithRelations(params.slug);│
│      const scenes = await getScenesForRoom(room.id);      │
│      const hotspots = await getHotspotsForScenes(         │
│        scenes.map(s => s.sceneId)                         │
│      );                                                   │
│      const gallery = await getGalleryImages(              │
│        'room', room.id                                    │
│      );                                                   │
│                                                           │
│      return (                                             │
│        <>                                                 │
│          <SEOHead room={room} />                          │
│          <RoomImmersivePage                               │
│            room={room}                                    │
│            scenes={scenes}                                │
│            hotspots={hotspots}                             │
│            gallery={gallery}                              │
│          />                                               │
│        </>                                                │
│      );                                                   │
│    }                                                      │
│                                                           │
│    // SEO: JSON-LD HotelRoom schema                       │
│    // OG image: thumbnail_url                             │
│    // Canonical: /rooms/{slug}                            │
└───────────────────────────┬─────────────────────────────┘
                            │ (HTML + JSON props)
                            ▼
┌─────────────────────────────────────────────────────────┐
│ 2. CLIENT HYDRATION                                      │
│                                                           │
│    RoomImmersivePage (client component)                   │
│    ├── PanoramaViewer                                     │
│    │   ├── Init Pannellum with scenes config              │
│    │   ├── Load default scene panorama                    │
│    │   ├── Render hotspots                                │
│    │   └── Attach interaction event listeners             │
│    │                                                      │
│    ├── SmartUIController                                  │
│    │   └── Attach to PanoramaViewer events                │
│    │                                                      │
│    ├── InfoPanel (desktop) / InfoPanelMobile (mobile)     │
│    │   ├── Room name, type badge, area, occupancy         │
│    │   ├── Short description (expandable)                 │
│    │   ├── Amenities grid                                 │
│    │   └── Price + CTA buttons                            │
│    │                                                      │
│    ├── SceneNavigator                                     │
│    │   ├── Scene thumbnails                               │
│    │   └── Active scene indicator                         │
│    │                                                      │
│    ├── FloatingCTA                                        │
│    │   ├── Price display                                  │
│    │   └── Book Now / Check Rate buttons                  │
│    │                                                      │
│    ├── BookingWidget (expandable)                         │
│    │   ├── Date picker                                    │
│    │   ├── Guest count                                    │
│    │   └── Submit → redirect to booking engine            │
│    │                                                      │
│    ├── GalleryOverlay (conditional)                       │
│    │   ├── Lightbox image viewer                          │
│    │   └── Thumbnail strip                                │
│    │                                                      │
│    └── UIToggle button                                    │
└─────────────────────────────────────────────────────────┘
                            │
                            │ (User interactions)
                            ▼
┌─────────────────────────────────────────────────────────┐
│ 3. RUNTIME INTERACTIONS                                  │
│                                                           │
│    [User drags panorama]                                  │
│    → PanoramaViewer emits 'interaction-start'             │
│    → SmartUIController hides panels (300ms fade)          │
│    → sceneStore remains unchanged                         │
│                                                           │
│    [User stops dragging, 4s passes]                       │
│    → SmartUIController detects idle                       │
│    → Panels fade back in (500ms)                          │
│                                                           │
│    [User clicks navigate hotspot → "Balcony"]             │
│    → PanoramaViewer triggers scene change                 │
│    → sceneStore.setTransitioning(true)                    │
│    → Pannellum: fade out current scene                    │
│    → Pannellum: load + fade in new scene                  │
│    → sceneStore.setCurrentScene('balcony')                │
│    → sceneStore.setTransitioning(false)                   │
│    → SceneNavigator updates active thumbnail              │
│    → analytics.track('scene_view', { scene: 'balcony' }) │
│                                                           │
│    [User clicks info hotspot → "Ocean View"]              │
│    → HotspotInfoCard appears (scale in from hotspot)      │
│    → uiStore.setActiveHotspotCard(hotspotId)              │
│    → Click outside or × → close card                      │
│    → analytics.track('hotspot_click', { hotspot: '...' }) │
│                                                           │
│    [User clicks "Book Now"]                               │
│    → analytics.track('booking_click', { room: slug })     │
│    → Option A: window.open(bookingUrl, '_blank')          │
│    → Option B: uiStore.toggleBookingWidget() → expand     │
│                                                           │
│    [User opens gallery]                                   │
│    → uiStore.toggleGallery()                              │
│    → GalleryOverlay mounts with fade in                   │
│    → Image navigation (swipe/arrow)                       │
│    → Close → unmount with fade out                        │
└─────────────────────────────────────────────────────────┘
```

---

## 4. API Client & Data Fetching

```typescript
// src/lib/api.ts

const API_BASE = process.env.NEXT_PUBLIC_API_URL || '/api/v1';

class APIClient {
  private baseUrl: string;
  private locale: string;

  constructor(locale: string = 'en') {
    this.baseUrl = API_BASE;
    this.locale = locale;
  }

  private async fetch<T>(path: string, options?: RequestInit): Promise<T> {
    const url = `${this.baseUrl}${path}`;
    const headers: HeadersInit = {
      'Content-Type': 'application/json',
      'Accept-Language': this.locale,
      ...options?.headers,
    };

    const res = await fetch(url, { ...options, headers, next: { revalidate: 300 } });
    
    if (!res.ok) {
      throw new APIError(res.status, await res.text());
    }
    
    return res.json();
  }

  // Public endpoints
  async getHotel(): Promise<Hotel> {
    return this.fetch('/hotel');
  }

  async getMenu(): Promise<MenuItem[]> {
    return this.fetch('/hotel/menu');
  }

  async getRooms(): Promise<RoomSummary[]> {
    return this.fetch('/rooms');
  }

  async getRoom(slug: string): Promise<RoomDetail> {
    return this.fetch(`/rooms/${slug}`);
  }

  async getServices(category?: string): Promise<ServiceSummary[]> {
    const query = category ? `?category=${category}` : '';
    return this.fetch(`/services${query}`);
  }

  async getService(slug: string): Promise<ServiceDetail> {
    return this.fetch(`/services/${slug}`);
  }

  async getPromotions(): Promise<Promotion[]> {
    return this.fetch('/promotions');
  }

  async getPosts(params?: { category?: string; page?: number }): Promise<PaginatedPosts> {
    const query = new URLSearchParams(params as any).toString();
    return this.fetch(`/posts?${query}`);
  }

  // Analytics
  async trackEvent(event: AnalyticsEvent): Promise<void> {
    await this.fetch('/analytics/event', {
      method: 'POST',
      body: JSON.stringify(event),
    });
  }
}

export const api = new APIClient();


// Server-side data fetching (trong page.tsx)
// Dùng Prisma trực tiếp cho SSG/SSR (không qua API)

// src/lib/data/rooms.ts

import { prisma } from '@/lib/prisma';

export async function getRoomWithRelations(slug: string, locale: string = 'en') {
  const room = await prisma.room.findUnique({
    where: { hotelId_slug: { hotelId: HOTEL_ID, slug } },
    include: {
      amenities: { include: { amenity: true } },
      scenes: { orderBy: { sortOrder: 'asc' } },
      gallery: { orderBy: { sortOrder: 'asc' } },
    },
  });

  if (!room) return null;

  // Fetch translations if locale !== default
  if (locale !== 'en') {
    const translations = await prisma.translation.findMany({
      where: {
        translatableType: 'room',
        translatableId: room.id,
        locale,
      },
    });
    // Merge translations into room object
    return mergeTranslations(room, translations);
  }

  return room;
}
```

---

## 5. Scene Mapping Strategy

### 5.1 Mapping Room ↔ Scene

```
DATABASE:
rooms table:
  slug: "deluxe-ocean"
  default_scene_id: "deluxe-ocean-bedroom"

room_scenes table:
  room_id → rooms.id
  scene_id: "deluxe-ocean-bedroom"   | is_default: true
  scene_id: "deluxe-ocean-balcony"   | is_default: false
  scene_id: "deluxe-ocean-bathroom"  | is_default: false

scene_configs table:
  scene_id: "deluxe-ocean-bedroom"
  panorama_url: "https://cdn.../pano/deluxe-ocean-bedroom-8192x4096.jpg"
  default_yaw: 0
  default_pitch: 0
  default_hfov: 100
  auto_rotate: 0.5

hotspots table:
  scene_id: "deluxe-ocean-bedroom"
  type: "navigate"
  target_scene_id: "deluxe-ocean-balcony"
  pitch: -5.2
  yaw: 120.5
  label: "Go to Balcony"
```

### 5.2 Fallback khi không có 360

Nếu một phòng/dịch vụ chưa có panorama 360:

```
if (room.panoramaUrl) {
  → Render 360-first layout (Layout A)
} else {
  → Render traditional layout (Layout B) với image hero
  → Show "360 coming soon" badge
}
```

---

## 6. Performance Optimization

### 6.1 Panorama Image Strategy

| Resolution | Dùng cho | File size est. |
|-----------|---------|---------------|
| 512×256 | Blur placeholder (inline base64) | < 5KB |
| 2048×1024 | Mobile / initial load | 200-400KB |
| 4096×2048 | Tablet / desktop default | 500KB-1MB |
| 8192×4096 | Desktop high-res (lazy upgrade) | 2-4MB |

```typescript
// Progressive loading strategy
const panoramaUrls = {
  placeholder: 'data:image/jpeg;base64,...',  // Inline blur
  mobile: `${baseUrl}/pano-2048.jpg`,
  desktop: `${baseUrl}/pano-4096.jpg`,
  highRes: `${baseUrl}/pano-8192.jpg`,  // Load after initial render
};
```

### 6.2 Code Splitting

```typescript
// Dynamic imports cho heavy components
const PanoramaViewer = dynamic(
  () => import('@/components/panorama/PanoramaViewer'),
  { 
    loading: () => <PanoramaLoader />,
    ssr: false  // Pannellum cần browser APIs
  }
);

const GalleryOverlay = dynamic(
  () => import('@/components/overlay/GalleryOverlay'),
  { loading: () => null }
);

const BookingWidget = dynamic(
  () => import('@/components/overlay/BookingWidget')
);
```

### 6.3 Caching Headers

```typescript
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/api/v1/:path*',
        headers: [
          { key: 'Cache-Control', value: 's-maxage=300, stale-while-revalidate=600' },
        ],
      },
      {
        source: '/:path*.jpg',
        headers: [
          { key: 'Cache-Control', value: 'public, max-age=2592000, immutable' },
        ],
      },
    ];
  },
};
```

---

## 7. Analytics Integration

```typescript
// src/hooks/useAnalytics.ts

export function useAnalytics() {
  const sessionId = useRef(generateSessionId());

  const track = useCallback(async (
    eventType: 'scene_view' | 'hotspot_click' | 'booking_click' | 'gallery_open',
    data: Record<string, any> = {}
  ) => {
    try {
      await api.trackEvent({
        event_type: eventType,
        session_id: sessionId.current,
        page_path: window.location.pathname,
        ...data,
      });
    } catch (e) {
      // Silent fail — analytics should never break UX
      console.warn('Analytics error:', e);
    }
  }, []);

  return { track };
}

// Usage trong PanoramaViewer:
const { track } = useAnalytics();

// Khi scene thay đổi
track('scene_view', { scene_id: newSceneId, room_slug: room.slug });

// Khi click hotspot
track('hotspot_click', { hotspot_id: hotspot.id, hotspot_type: hotspot.type });

// Khi click booking CTA
track('booking_click', { room_slug: room.slug, source: 'floating_cta' });
```

---

## 8. Admin Dashboard — Hotspot Visual Editor

```
Tính năng quan trọng nhất cho admin: kéo-thả hotspot trên panorama.

FLOW:
1. Admin chọn Scene từ danh sách
2. Panorama load trong editor view
3. Admin click vào vị trí trong panorama
4. Pannellum trả về pitch/yaw coordinates
5. Form popup: chọn type, nhập label, description, target
6. Save → hotspot xuất hiện trên panorama
7. Drag hotspot → cập nhật pitch/yaw
8. Preview mode: xem như khách truy cập

IMPLEMENTATION:
- Dùng Pannellum viewer trong admin page
- pannellum.viewer.mouseEventToCoords(event) → {pitch, yaw}
- Render hotspots dưới dạng draggable markers
- Save changes via Admin API
```

---

## 9. Demo HTML Implementation (Phase 0)

> Trang demo `index.html` là proof-of-concept thuần HTML/CSS/JS, không dùng framework.
> Mục tiêu: validate toàn bộ UX concept trước khi triển khai React/Next.js.

### 9.1 Architecture Overview

```
index.html (single file, ~700 lines)
├── Inline CSS (~200 lines)
│   ├── CSS Variables (design tokens)
│   ├── Component styles
│   ├── Responsive breakpoints (@media 1100px, 860px)
│   └── Mobile bottom sheet states
├── Inline HTML (~100 lines)
│   ├── Loader screen
│   ├── Panorama container
│   ├── UI Layer (topbar, panel, scene-nav, location-pill, restore btn)
│   └── Mobile overlay menu
├── Inline JS (~400 lines)
│   ├── SECTIONS data array (9 scenes)
│   ├── Navigation & scene management
│   ├── Panel content rendering (4 panelTypes)
│   ├── Smart UI (interaction/idle detection)
│   ├── Mobile panel drag IIFE
│   └── Loader progress simulation
└── CDN Dependencies
    ├── Pannellum 2.5.6 (CSS + JS)
    └── Google Fonts (Cormorant Garamond + Outfit)
```

### 9.2 Mobile Panel Drag System

```javascript
// IIFE pattern — self-contained module
(function(){
  // State
  let startY, startTranslate, currentY, isDragging, dragSource;
  
  // Key functions:
  // getCurrentTranslateY(el) — parse computed transform matrix
  // onDragStart(e) — detect source (bar vs panel), read position
  // onDragMove(e) — free drag (bar) or auto-snap (panel body)
  // onDragEnd() — direction-based snap with rAF
  // cancelDrag() — restore state when scroll should take over
  
  // Event binding via MutationObserver
  // → auto-bind when contentPanel appears in DOM
})();
```

**Direction-based snap (drag-bar):**
```
dragDelta = currentY - startTranslate
if (dragDelta < -20) → expand    // kéo lên
if (dragDelta > 20)  → collapse  // kéo xuống  
else                 → restore previous state
```

**Auto-snap (panel body):**
```
delta = clientY - startY
if (|delta| < 15) return;        // chưa đủ gesture
if (delta < 0 && already expanded) → cancelDrag, allow scroll
if (delta < 0) → expand
if (delta > 0) → collapse
```

**requestAnimationFrame pattern:**
```
panel.classList.remove("dragging");  // has transition:none!important
requestAnimationFrame(() => {
  panel.style.transition = "transform .35s cubic-bezier(.16,1,.3,1)";
  panel.style.transform = "translateY(" + targetY + "px)";
  let settled = false;
  const onDone = () => {
    if (settled) return; settled = true;
    panel.style.transition = ""; panel.style.transform = "";
    panel.classList.toggle("expanded", shouldExpand);
  };
  panel.addEventListener("transitionend", onDone);
  setTimeout(onDone, 400);  // fallback
});
```

### 9.3 Smart UI Implementation

```javascript
// Panorama event listeners
el.addEventListener("mousedown", onS);   // → fadeOut (hide UI)
el.addEventListener("touchstart", onS);
el.addEventListener("mouseup", onE);     // → resetIdle (5s timer)
el.addEventListener("touchend", onE);

// State: uiHidden (boolean), idleTimer, isInteracting
// fadeOut(): add .hidden to .ui-layer
// fadeIn(): remove .hidden from .ui-layer
// resetIdle(): clearTimeout + setTimeout(5000ms)
// toggleUI(): manual toggle (btn-icon click)
// showAllUI(): force show + reset idle timer
```

### 9.4 Scene Navigation

```javascript
// navigateTo(id, anim=true)
// 1. Find section in SECTIONS array
// 2. Close any open info card
// 3. If first load: initViewer(sec) — create Pannellum instance
// 4. If animated: slide-out panel → destroy viewer → initViewer → slide-in
// 5. renderPanel(sec) — rebuild panel HTML based on panelType
// 6. renderSceneNav(sec) — show sub-scene tabs if subScenes.length >= 2
// 7. updateNav(id) — highlight active nav link (map sub-scenes to parent)
```

### 9.5 Custom Hotspot Rendering

```javascript
// Pannellum custom hotspot renderer
createTooltipFunc: (el, args) => {
  el.classList.add("hs", type === "nav" ? "hs-nav" : "hs-info");
  el.innerHTML = icon_svg + '<span>' + args.label + '</span>';
  el.onclick = () => {
    if (args.type === "nav") navigateTo(args.target);
    else showCard(args, event);  // floating info card
  };
}
// Hotspot styles: pill shape, backdrop-filter blur, pulse animation
```

### 9.6 Mapping to Future React Components

| Demo JS Function | Future React Component |
|------------------|----------------------|
| `navigateTo()` | `usePanorama` hook + `PanoramaViewer` |
| `renderPanel()` | `InfoPanel` / `InfoPanelMobile` |
| `renderSceneNav()` | `SceneNavigator` |
| `onS/onE/fadeOut/fadeIn` | `useSmartUI` hook |
| Mobile drag IIFE | `useBottomSheet` hook |
| `renderNav()` | `Navbar` + `MobileMenu` |
| `showCard/closeCard` | `HotspotInfoCard` |
| SECTIONS array | API response from `getRoomWithRelations()` |
| Loader IIFE | `PanoramaLoader` component |

---

## 10. Sequence Diagrams

### 10.1 Client Hydration — Full Init Flow

```
  Next.js SSR       React Hydration     PanoramaViewer     SmartUI Hook       Zustand Stores
   │                     │                │                  │                  │
   │  HTML + JSON props  │                │                  │                  │
   │  (room, scenes,     │                │                  │                  │
   │   hotspots, gallery)│                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ hydrate        │                  │                  │
   │                     │ RoomImmersive  │                  │                  │
   │                     │ Page           │                  │                  │
   │                     │                │                  │                  │
   │                     │ initScenes() ───────────────────────────────────────►│
   │                     │                │                  │                  │ sceneStore
   │                     │                │                  │                  │ set scenes[]
   │                     │                │                  │                  │ set hotspots{}
   │                     │                │                  │                  │
   │                     │ dynamic import │                  │                  │
   │                     │ (ssr: false)   │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ Pannellum CDN    │                  │
   │                     │                │ loaded           │                  │
   │                     │                │                  │                  │
   │                     │                │ init viewer()    │                  │
   │                     │                │ config:          │                  │
   │                     │                │  autoLoad: true  │                  │
   │                     │                │  showControls:   │                  │
   │                     │                │  false           │                  │
   │                     │                │  friction: 0.15  │                  │
   │                     │                │  sceneFadeDur:   │                  │
   │                     │                │  500ms           │                  │
   │                     │                │                  │                  │
   │                     │                │ load panorama    │                  │
   │                     │                │ add hotSpots     │                  │
   │                     │                │                  │                  │
   │                     │                │ attach events ──►│                  │
   │                     │                │ (mousedown,      │ bind interaction │
   │                     │                │  touchstart,     │ listeners        │
   │                     │                │  mouseup,        │ init idle timer  │
   │                     │                │  touchend)       │ (4s timeout)     │
   │                     │                │                  │                  │
   │                     │                │ setLoading(false)────────────────────►
   │                     │                │                  │                  │ sceneStore
   │                     │                │                  │                  │ isLoading=false
   │                     │                │                  │                  │
   │                     │ mount UI ──────────────────────────────────────────►│
   │                     │ components     │                  │                  │ uiStore
   │                     │                │                  │                  │ visibility=
   │                     │                │                  │                  │ 'visible'
   │  ◄─────────────────── page interactive ───────────────────────────────   │
```

### 10.2 Scene Change — Runtime Flow

```
  User               HotspotMarker       PanoramaViewer     sceneStore         InfoPanel
   │                     │                │                  │                  │
   │  click navigate     │                │                  │                  │
   │  hotspot            │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ clickHandler   │                  │                  │
   │                     │ (sceneId)      │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ setTransitioning ────────────────────►
   │                     │                │ (true)           │                  │ transitioning
   │                     │                │                  │                  │
   │                     │                │ Pannellum:       │                  │
   │                     │                │ fade out current │                  │
   │                     │                │ (sceneFadeDur    │                  │
   │                     │                │  500ms)          │                  │
   │                     │                │                  │                  │
   │                     │                │ load new         │                  │
   │                     │                │ panorama_url     │                  │
   │                     │                │                  │                  │
   │                     │                │ render new       │                  │
   │                     │                │ hotSpots         │                  │
   │                     │                │                  │                  │
   │                     │                │ fade in new      │                  │
   │                     │                │ scene (500ms)    │                  │
   │                     │                │                  │                  │
   │                     │                │ setCurrentScene ─────────────────────►
   │                     │                │ (newSceneId)     │                  │ update
   │                     │                │                  │                  │ currentSceneId
   │                     │                │                  │                  │
   │                     │                │ setTransitioning ────────────────────►
   │                     │                │ (false)          │                  │ transitioning
   │                     │                │                  │                  │ = false
   │                     │                │                  │                  │
   │                     │                │                  │                  ├── react to
   │                     │                │                  │                  │   store change
   │                     │                │                  │                  │   update panel
   │                     │                │                  │                  │   content
   │                     │                │                  │                  │
   │                     │                │ analytics.track ─────────────────   │
   │                     │                │ ('scene_view',   │ → API POST      │
   │                     │                │  {scene_id,      │                  │
   │                     │                │   room_slug})    │                  │
   │  ◄──────────────────── new scene visible ─────────────────────────────   │
```

### 10.3 Analytics Event Pipeline

```
  User Action        useAnalytics()      API Client          Server API        Database
   │                     │                │                  │                  │
   │  scene_view         │                │                  │                  │
   │  (auto on scene     │                │                  │                  │
   │   change)           │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ track(         │                  │                  │
   │                     │  'scene_view', │                  │                  │
   │                     │  {scene_id,    │                  │                  │
   │                     │   room_slug})  │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ POST /api/v1/    │                  │
   │                     │                │ analytics/event  │                  │
   │                     │                │ {event_type,     │                  │
   │                     │                │  session_id,     │                  │
   │                     │                │  page_path,      │                  │
   │                     │                │  ...data}        │                  │
   │                     │                ├─────────────────►│                  │
   │                     │                │                  │ validate         │
   │                     │                │                  │ enrich (IP,      │
   │                     │                │                  │  user-agent,     │
   │                     │                │                  │  geo)            │
   │                     │                │                  │ INSERT ──────────►
   │                     │                │                  │                  │ analytics_
   │                     │                │                  │                  │ events table
   │                     │                │  ◄── 200 OK ─────┤                  │
   │                     │  ◄── resolved ─┤                  │                  │
   │                     │                │                  │                  │
   │  hotspot_click      │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ track(         │                  │                  │
   │                     │  'hotspot_     │                  │                  │
   │                     │   click',      │                  │                  │
   │                     │  {hotspot_id,  │                  │                  │
   │                     │   type})       │                  │                  │
   │                     ├───────────────►│ POST ───────────►│ INSERT ──────────►
   │                     │                │                  │                  │
   │  booking_click      │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ track(         │                  │                  │
   │                     │  'booking_     │                  │                  │
   │                     │   click',      │                  │                  │
   │                     │  {room_slug,   │                  │                  │
   │                     │   source})     │                  │                  │
   │                     ├───────────────►│ POST ───────────►│ INSERT ──────────►
   │                     │                │                  │                  │
   │  gallery_open       │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ track(         │                  │                  │
   │                     │  'gallery_     │                  │                  │
   │                     │   open',       │                  │                  │
   │                     │  {room_slug})  │                  │                  │
   │                     ├───────────────►│ POST ───────────►│ INSERT ──────────►
   │                     │                │                  │                  │
   │                     │ NOTE: all      │                  │                  │
   │                     │ track() calls  │                  │                  │
   │                     │ try/catch      │                  │                  │
   │                     │ → silent fail  │                  │                  │
   │                     │ (never break   │                  │                  │
   │                     │  UX)           │                  │                  │
```

### 10.4 Admin Hotspot Visual Editor Flow

```
  Admin              SceneSelector       PanoramaEditor     HotspotForm        Server API
   │                     │                │                  │                  │
   │  select scene       │                │                  │                  │
   │  from list          │                │                  │                  │
   ├────────────────────►│                │                  │                  │
   │                     │ load scene     │                  │                  │
   │                     │ config         │                  │                  │
   │                     ├───────────────►│                  │                  │
   │                     │                │ init Pannellum   │                  │
   │                     │                │ load panorama    │                  │
   │                     │                │ render existing  │                  │
   │                     │                │ hotspots as      │                  │
   │                     │                │ draggable markers│                  │
   │  ◄──────────────────── editor ready ─┤                  │                  │
   │                     │                │                  │                  │
   │  click on panorama  │                │                  │                  │
   │  (place hotspot)    │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ mouseEventTo     │                  │
   │                     │                │ Coords(event)    │                  │
   │                     │                │ → {pitch, yaw}   │                  │
   │                     │                │                  │                  │
   │                     │                │ show form ──────►│                  │
   │                     │                │                  │ type: dropdown   │
   │                     │                │                  │ (info/navigate/  │
   │                     │                │                  │  cta/gallery)    │
   │                     │                │                  │ label: text      │
   │                     │                │                  │ description:     │
   │                     │                │                  │ textarea         │
   │                     │                │                  │ target: select   │
   │                     │                │                  │                  │
   │  fill form & save   │                │                  │                  │
   ├──────────────────────────────────────────────────────►│                  │
   │                     │                │                  │ validate         │
   │                     │                │                  │ POST /admin/     │
   │                     │                │                  │ hotspots ────────►
   │                     │                │                  │                  │ INSERT
   │                     │                │                  │                  │ hotspots
   │                     │                │                  │  ◄── 201 ────────┤
   │                     │                │  ◄── add marker ─┤                  │
   │                     │                │  render new      │                  │
   │                     │                │  hotspot on pano │                  │
   │  ◄──────────────────── hotspot placed ────────────────────────────────   │
   │                     │                │                  │                  │
   │  drag existing      │                │                  │                  │
   │  hotspot            │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ update pitch/yaw │                  │
   │                     │                │ on drag          │                  │
   │                     │                │                  │                  │
   │  release            │                │                  │                  │
   ├──────────────────────────────────────►                  │                  │
   │                     │                │ PATCH /admin/    │                  │
   │                     │                │ hotspots/:id ────────────────────────►
   │                     │                │ {pitch, yaw}     │                  │ UPDATE
   │                     │                │  ◄── 200 ────────────────────────────┤
   │  ◄──────────────────── position saved ────────────────────────────────   │
```

---

*Tài liệu tiếp theo: **05-DEMO-HTML-SPEC.md** — Spec chi tiết cho trang demo HTML thuần*
