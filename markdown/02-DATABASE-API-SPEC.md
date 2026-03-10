# 02 — DATABASE & API SPECIFICATION
## Website 360 Thế Hệ Mới — Hotel CMS Platform

**Version:** 1.0  
**Date:** 2026-03-10  
**Related:** 01-SYSTEM-ARCHITECTURE.md

---

## 1. Database Schema (PostgreSQL)

### 1.1 Entity Relationship Diagram (ERD)

```
┌──────────────┐     ┌──────────────────┐     ┌───────────────┐
│   hotels     │────<│   rooms          │────<│ room_amenities│
│              │     │                  │     └───────────────┘
│  id          │     │  id              │            │
│  name        │     │  hotel_id (FK)   │     ┌──────┴────────┐
│  slug        │     │  name            │     │  amenities    │
│  logo_url    │     │  slug            │     │  id, name,    │
│  description │     │  type            │     │  icon         │
│  address     │     │  area_sqm        │     └───────────────┘
│  phone       │     │  max_adults      │
│  email       │     │  max_children    │     ┌───────────────┐
│  social_json │     │  bed_type        │────<│ room_scenes   │
│  settings    │     │  description     │     │               │
│  created_at  │     │  short_desc      │     │  id           │
│  updated_at  │     │  price_from      │     │  room_id (FK) │
└──────┬───────┘     │  panorama_url    │     │  scene_id     │
       │             │  thumbnail_url   │     │  panorama_url │
       │             │  seo_title       │     │  label        │
       │             │  seo_desc        │     │  is_default   │
       │             │  is_published    │     │  sort_order   │
       │             │  sort_order      │     └───────────────┘
       │             │  created_at      │
       │             │  updated_at      │     ┌───────────────┐
       │             └──────────────────┘────<│ room_gallery  │
       │                                      │  id           │
       │                                      │  room_id (FK) │
       │                                      │  image_url    │
       │                                      │  caption      │
       │                                      │  sort_order   │
       │                                      └───────────────┘
       │
       │             ┌──────────────────┐     ┌───────────────┐
       ├────────────<│   services       │────<│service_scenes │
       │             │                  │     └───────────────┘
       │             │  id              │
       │             │  hotel_id (FK)   │     ┌───────────────┐
       │             │  category        │────<│service_gallery│
       │             │  name            │     └───────────────┘
       │             │  slug            │
       │             │  description     │
       │             │  short_desc      │
       │             │  opening_hours   │
       │             │  panorama_url    │
       │             │  thumbnail_url   │
       │             │  cta_type        │
       │             │  cta_label       │
       │             │  cta_url         │
       │             │  seo_title       │
       │             │  seo_desc        │
       │             │  is_published    │
       │             │  sort_order      │
       │             └──────────────────┘
       │
       │             ┌──────────────────┐
       ├────────────<│   hotspots       │
       │             │                  │
       │             │  id              │
       │             │  scene_id        │
       │             │  type            │  ← 'info' | 'navigate' | 'cta' | 'gallery'
       │             │  target_type     │  ← 'room' | 'service' | 'external'
       │             │  target_id       │
       │             │  pitch           │  ← tọa độ Y trong panorama
       │             │  yaw             │  ← tọa độ X trong panorama
       │             │  label           │
       │             │  description     │
       │             │  icon            │
       │             │  image_url       │
       │             │  cta_url         │
       │             └──────────────────┘
       │
       │             ┌──────────────────┐
       ├────────────<│   promotions     │
       │             │  id              │
       │             │  hotel_id (FK)   │
       │             │  title           │
       │             │  slug            │
       │             │  description     │
       │             │  image_url       │
       │             │  valid_from      │
       │             │  valid_to        │
       │             │  discount_type   │
       │             │  discount_value  │
       │             │  conditions      │
       │             │  is_published    │
       │             └──────────────────┘
       │
       │             ┌──────────────────┐
       ├────────────<│   posts          │
       │             │  id              │
       │             │  hotel_id (FK)   │
       │             │  category        │  ← 'news' | 'blog' | 'announcement'
       │             │  title           │
       │             │  slug            │
       │             │  excerpt         │
       │             │  content (TEXT)  │
       │             │  featured_image  │
       │             │  author_id (FK)  │
       │             │  seo_title       │
       │             │  seo_desc        │
       │             │  is_published    │
       │             │  published_at    │
       │             └──────────────────┘
       │
       │             ┌──────────────────┐
       ├────────────<│   pages          │
       │             │  id              │
       │             │  hotel_id (FK)   │
       │             │  title           │
       │             │  slug            │
       │             │  content (TEXT)  │
       │             │  template        │  ← 'default' | '360' | 'gallery'
       │             │  panorama_url    │
       │             │  is_published    │
       │             └──────────────────┘
       │
       │             ┌──────────────────┐
       ├────────────<│   menus          │
       │             │  id              │
       │             │  hotel_id (FK)   │
       │             │  location        │  ← 'header' | 'footer'
       │             │  parent_id       │
       │             │  label           │
       │             │  url             │
       │             │  target_type     │  ← 'room' | 'service' | 'page' | 'external'
       │             │  target_id       │
       │             │  sort_order      │
       │             │  is_visible      │
       │             └──────────────────┘
       │
       │             ┌──────────────────┐
       └────────────<│   translations   │
                     │  id              │
                     │  translatable_id │
                     │  translatable_type│ ← 'room' | 'service' | 'post' | ...
                     │  locale          │ ← 'vi' | 'en' | 'ru' | 'ko' | 'zh'
                     │  field           │ ← 'name' | 'description' | ...
                     │  value (TEXT)    │
                     └──────────────────┘

┌──────────────────┐
│   users          │
│  id              │
│  hotel_id (FK)   │
│  email           │
│  password_hash   │
│  name            │
│  role            │  ← 'super_admin' | 'admin' | 'editor' | 'viewer'
│  avatar_url      │
│  last_login_at   │
│  is_active       │
│  created_at      │
└──────────────────┘

┌──────────────────┐
│  scene_configs   │  ← Cấu hình scene 360 toàn cục
│  id              │
│  hotel_id (FK)   │
│  scene_id        │  ← unique ID cho scene
│  label           │
│  panorama_url    │
│  default_yaw     │  ← góc nhìn mặc định khi load
│  default_pitch   │
│  default_hfov    │  ← field of view mặc định
│  auto_rotate     │  ← boolean
│  category        │  ← 'room' | 'dining' | 'recreation' | 'common'
│  linked_to_type  │
│  linked_to_id    │
│  created_at      │
└──────────────────┘

┌──────────────────┐
│  analytics_events│  ← Tracking scene views, hotspot clicks, booking CTA
│  id              │
│  hotel_id        │
│  event_type      │  ← 'scene_view' | 'hotspot_click' | 'booking_click' | 'gallery_open'
│  scene_id        │
│  hotspot_id      │
│  page_path       │
│  user_agent      │
│  ip_hash         │  ← hashed for privacy
│  session_id      │
│  created_at      │
└──────────────────┘
```

### 1.2 SQL Schema Chính

```sql
-- ============================================
-- HOTEL
-- ============================================
CREATE TABLE hotels (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name          VARCHAR(255) NOT NULL,
  slug          VARCHAR(255) UNIQUE NOT NULL,
  logo_url      TEXT,
  description   TEXT,
  address       TEXT,
  city          VARCHAR(100),
  country       VARCHAR(100) DEFAULT 'Vietnam',
  phone         VARCHAR(50),
  email         VARCHAR(255),
  website_url   TEXT,
  booking_url   TEXT,           -- Link đến booking engine bên ngoài
  social_links  JSONB DEFAULT '{}',
  settings      JSONB DEFAULT '{}',  -- theme colors, fonts, meta defaults
  star_rating   SMALLINT DEFAULT 4,
  is_active     BOOLEAN DEFAULT true,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- ROOMS
-- ============================================
CREATE TABLE rooms (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id        UUID NOT NULL REFERENCES hotels(id) ON DELETE CASCADE,
  name            VARCHAR(255) NOT NULL,
  slug            VARCHAR(255) NOT NULL,
  room_type       VARCHAR(50) DEFAULT 'standard',  -- 'standard','deluxe','suite','presidential'
  area_sqm        DECIMAL(6,1),
  max_adults      SMALLINT DEFAULT 2,
  max_children    SMALLINT DEFAULT 1,
  bed_type        VARCHAR(100),   -- 'double', 'twin', 'king', 'twin+sofa'
  description     TEXT,
  short_desc      VARCHAR(500),
  price_from      DECIMAL(12,2),
  currency        VARCHAR(3) DEFAULT 'VND',
  view_type       VARCHAR(50),    -- 'ocean', 'mountain', 'city', 'garden'
  floor_range     VARCHAR(20),    -- '5-10', '15-20'
  highlights      JSONB DEFAULT '[]',  -- ["Balcony bathtub","Ocean view","Private pool"]
  panorama_url    TEXT,            -- URL ảnh equirectangular mặc định
  thumbnail_url   TEXT,
  default_scene_id VARCHAR(100),  -- scene mặc định khi load trang
  booking_url     TEXT,            -- Override booking URL cho phòng cụ thể
  seo_title       VARCHAR(255),
  seo_description VARCHAR(500),
  is_published    BOOLEAN DEFAULT false,
  sort_order      SMALLINT DEFAULT 0,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(hotel_id, slug)
);

-- ============================================
-- AMENITIES
-- ============================================
CREATE TABLE amenities (
  id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id  UUID REFERENCES hotels(id) ON DELETE CASCADE,
  name      VARCHAR(100) NOT NULL,
  icon      VARCHAR(50),       -- icon class hoặc SVG name
  category  VARCHAR(50),       -- 'room', 'bathroom', 'technology', 'view'
  sort_order SMALLINT DEFAULT 0
);

CREATE TABLE room_amenities (
  room_id    UUID REFERENCES rooms(id) ON DELETE CASCADE,
  amenity_id UUID REFERENCES amenities(id) ON DELETE CASCADE,
  PRIMARY KEY (room_id, amenity_id)
);

-- ============================================
-- SCENES (360 Configuration)
-- ============================================
CREATE TABLE scene_configs (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id        UUID NOT NULL REFERENCES hotels(id) ON DELETE CASCADE,
  scene_id        VARCHAR(100) NOT NULL,   -- unique identifier
  label           VARCHAR(255),
  panorama_url    TEXT NOT NULL,
  thumbnail_url   TEXT,
  default_yaw     DECIMAL(8,3) DEFAULT 0,
  default_pitch   DECIMAL(8,3) DEFAULT 0,
  default_hfov    DECIMAL(5,1) DEFAULT 100,
  min_hfov        DECIMAL(5,1) DEFAULT 50,
  max_hfov        DECIMAL(5,1) DEFAULT 120,
  auto_rotate     DECIMAL(4,2) DEFAULT 0,  -- degrees per frame, 0 = off
  category        VARCHAR(50),
  linked_to_type  VARCHAR(50),  -- 'room' | 'service' | null
  linked_to_id    UUID,
  sort_order      SMALLINT DEFAULT 0,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(hotel_id, scene_id)
);

-- ============================================
-- ROOM SCENES (một phòng có nhiều scene/góc nhìn)
-- ============================================
CREATE TABLE room_scenes (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  room_id     UUID NOT NULL REFERENCES rooms(id) ON DELETE CASCADE,
  scene_id    VARCHAR(100) NOT NULL,
  panorama_url TEXT NOT NULL,
  label       VARCHAR(100),     -- 'Bedroom', 'Balcony', 'Bathroom', 'Entry View'
  is_default  BOOLEAN DEFAULT false,
  sort_order  SMALLINT DEFAULT 0
);

-- ============================================
-- HOTSPOTS
-- ============================================
CREATE TABLE hotspots (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id        UUID NOT NULL REFERENCES hotels(id) ON DELETE CASCADE,
  scene_id        VARCHAR(100) NOT NULL,
  type            VARCHAR(20) NOT NULL,  -- 'info','navigate','cta','gallery'
  pitch           DECIMAL(8,3) NOT NULL, -- Y coordinate in panorama
  yaw             DECIMAL(8,3) NOT NULL, -- X coordinate in panorama
  label           VARCHAR(255),
  description     TEXT,
  icon            VARCHAR(50) DEFAULT 'info',
  image_url       TEXT,
  target_scene_id VARCHAR(100),          -- for navigate type
  target_type     VARCHAR(50),           -- 'room','service','external'
  target_id       UUID,
  cta_url         TEXT,
  cta_label       VARCHAR(100),
  css_class       VARCHAR(100),
  is_visible      BOOLEAN DEFAULT true,
  sort_order      SMALLINT DEFAULT 0,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- SERVICES (Dining, Spa, Pool, Meeting...)
-- ============================================
CREATE TABLE services (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id        UUID NOT NULL REFERENCES hotels(id) ON DELETE CASCADE,
  category        VARCHAR(50) NOT NULL,   -- 'dining','spa','pool','fitness','beach','meeting'
  name            VARCHAR(255) NOT NULL,
  slug            VARCHAR(255) NOT NULL,
  description     TEXT,
  short_desc      VARCHAR(500),
  opening_hours   JSONB DEFAULT '{}',     -- {"mon":"07:00-22:00", ...}
  capacity        VARCHAR(100),
  location_desc   VARCHAR(255),           -- "Tầng 20, Rooftop"
  highlights      JSONB DEFAULT '[]',
  panorama_url    TEXT,
  thumbnail_url   TEXT,
  default_scene_id VARCHAR(100),
  cta_type        VARCHAR(20) DEFAULT 'enquiry', -- 'booking','reserve','enquiry'
  cta_label       VARCHAR(100),
  cta_url         TEXT,
  phone           VARCHAR(50),
  seo_title       VARCHAR(255),
  seo_description VARCHAR(500),
  is_published    BOOLEAN DEFAULT false,
  sort_order      SMALLINT DEFAULT 0,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(hotel_id, slug)
);

-- ============================================
-- GALLERY (shared across rooms + services)
-- ============================================
CREATE TABLE gallery_images (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id        UUID NOT NULL REFERENCES hotels(id) ON DELETE CASCADE,
  parent_type     VARCHAR(50) NOT NULL,   -- 'room','service','hotel','promotion'
  parent_id       UUID NOT NULL,
  image_url       TEXT NOT NULL,
  thumbnail_url   TEXT,
  caption         VARCHAR(500),
  alt_text        VARCHAR(255),
  width           INT,
  height          INT,
  sort_order      SMALLINT DEFAULT 0,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- PROMOTIONS
-- ============================================
CREATE TABLE promotions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id        UUID NOT NULL REFERENCES hotels(id) ON DELETE CASCADE,
  title           VARCHAR(255) NOT NULL,
  slug            VARCHAR(255) NOT NULL,
  description     TEXT,
  short_desc      VARCHAR(500),
  image_url       TEXT,
  banner_url      TEXT,
  valid_from      DATE,
  valid_to        DATE,
  discount_type   VARCHAR(20),  -- 'percent','fixed','package'
  discount_value  DECIMAL(12,2),
  conditions      TEXT,
  booking_url     TEXT,
  is_featured     BOOLEAN DEFAULT false,
  is_published    BOOLEAN DEFAULT false,
  sort_order      SMALLINT DEFAULT 0,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(hotel_id, slug)
);

-- ============================================
-- POSTS (News / Blog)
-- ============================================
CREATE TABLE posts (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id        UUID NOT NULL REFERENCES hotels(id) ON DELETE CASCADE,
  category        VARCHAR(50) DEFAULT 'news',
  title           VARCHAR(500) NOT NULL,
  slug            VARCHAR(500) NOT NULL,
  excerpt         VARCHAR(1000),
  content         TEXT,
  featured_image  TEXT,
  author_id       UUID REFERENCES users(id),
  tags            TEXT[] DEFAULT '{}',
  seo_title       VARCHAR(255),
  seo_description VARCHAR(500),
  is_published    BOOLEAN DEFAULT false,
  published_at    TIMESTAMPTZ,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(hotel_id, slug)
);

-- ============================================
-- TRANSLATIONS (Polymorphic i18n)
-- ============================================
CREATE TABLE translations (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id          UUID NOT NULL REFERENCES hotels(id) ON DELETE CASCADE,
  translatable_type VARCHAR(50) NOT NULL,  -- 'room','service','post','promotion','page','amenity'
  translatable_id   UUID NOT NULL,
  locale            VARCHAR(5) NOT NULL,   -- 'vi','en','ru','ko','zh'
  field             VARCHAR(100) NOT NULL, -- 'name','description','short_desc','seo_title'...
  value             TEXT NOT NULL,
  created_at        TIMESTAMPTZ DEFAULT NOW(),
  updated_at        TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(translatable_type, translatable_id, locale, field)
);

CREATE INDEX idx_translations_lookup
  ON translations(translatable_type, translatable_id, locale);

-- ============================================
-- USERS (Admin Dashboard)
-- ============================================
CREATE TABLE users (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id      UUID REFERENCES hotels(id) ON DELETE CASCADE,
  email         VARCHAR(255) UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  name          VARCHAR(255),
  role          VARCHAR(20) DEFAULT 'editor', -- 'super_admin','admin','editor','viewer'
  avatar_url    TEXT,
  last_login_at TIMESTAMPTZ,
  is_active     BOOLEAN DEFAULT true,
  created_at    TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- MENUS
-- ============================================
CREATE TABLE menus (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id    UUID NOT NULL REFERENCES hotels(id) ON DELETE CASCADE,
  location    VARCHAR(20) DEFAULT 'header',
  parent_id   UUID REFERENCES menus(id) ON DELETE CASCADE,
  label       VARCHAR(100) NOT NULL,
  url         TEXT,
  target_type VARCHAR(50),
  target_id   UUID,
  icon        VARCHAR(50),
  open_in_new BOOLEAN DEFAULT false,
  is_visible  BOOLEAN DEFAULT true,
  sort_order  SMALLINT DEFAULT 0
);

-- ============================================
-- ANALYTICS
-- ============================================
CREATE TABLE analytics_events (
  id          BIGSERIAL PRIMARY KEY,
  hotel_id    UUID NOT NULL REFERENCES hotels(id),
  event_type  VARCHAR(50) NOT NULL,
  scene_id    VARCHAR(100),
  hotspot_id  UUID,
  page_path   VARCHAR(500),
  referrer    TEXT,
  user_agent  TEXT,
  ip_hash     VARCHAR(64),
  session_id  VARCHAR(100),
  metadata    JSONB DEFAULT '{}',
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_analytics_hotel_date
  ON analytics_events(hotel_id, created_at DESC);
CREATE INDEX idx_analytics_type
  ON analytics_events(event_type, created_at DESC);
```

---

## 2. REST API Specification

### 2.1 Base URL & Authentication

```
Base URL:     https://{domain}/api/v1
Auth Header:  Authorization: Bearer {jwt_token}
Content-Type: application/json
```

**Public endpoints:** Không cần auth (cho frontend website)  
**Admin endpoints:** Cần JWT token (cho admin dashboard)

### 2.2 Public API Endpoints (Frontend)

#### Hotel Info

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/hotel` | Thông tin khách sạn (settings, logo, contacts) |
| GET | `/hotel/menu` | Menu navigation (header + footer) |

**Response `/hotel`:**
```json
{
  "id": "uuid",
  "name": "Boton Blue Hotel & Spa",
  "slug": "boton-blue",
  "logo_url": "https://cdn.../logo.png",
  "description": "...",
  "address": "86 Trần Phú, Nha Trang",
  "phone": "+84 258 383 6868",
  "email": "sales@botonblue.com",
  "booking_url": "https://book-directonline.com/properties/botbluhotspadirect",
  "social_links": {
    "facebook": "https://...",
    "instagram": "https://...",
    "tiktok": "https://..."
  },
  "settings": {
    "primary_color": "#1a365d",
    "accent_color": "#c6a052",
    "default_locale": "en",
    "available_locales": ["en","vi","ru","ko","zh"]
  },
  "star_rating": 4
}
```

#### Rooms

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/rooms` | Danh sách phòng (published) |
| GET | `/rooms/:slug` | Chi tiết phòng + scenes + hotspots + gallery |

**Response `/rooms/:slug`:**
```json
{
  "id": "uuid",
  "name": "Deluxe Twin Room With Ocean View",
  "slug": "deluxe-ocean",
  "room_type": "deluxe",
  "area_sqm": 35.0,
  "max_adults": 2,
  "max_children": 1,
  "bed_type": "twin",
  "description": "Mô tả đầy đủ...",
  "short_desc": "Mô tả ngắn...",
  "price_from": 2500000,
  "currency": "VND",
  "view_type": "ocean",
  "highlights": ["Panorama ocean view", "Balcony", "Wireless Internet"],
  "booking_url": "https://...",
  "seo": {
    "title": "Deluxe Twin Ocean View | Boton Blue Hotel",
    "description": "..."
  },
  "amenities": [
    { "id": "uuid", "name": "Ocean View", "icon": "waves", "category": "view" },
    { "id": "uuid", "name": "Balcony", "icon": "door-open", "category": "room" },
    { "id": "uuid", "name": "WiFi", "icon": "wifi", "category": "technology" },
    { "id": "uuid", "name": "Air Conditioning", "icon": "snowflake", "category": "room" },
    { "id": "uuid", "name": "Bathtub", "icon": "bath", "category": "bathroom" }
  ],
  "scenes": [
    {
      "scene_id": "deluxe-ocean-bedroom",
      "panorama_url": "https://cdn.../deluxe-ocean-bedroom.jpg",
      "label": "Bedroom",
      "is_default": true,
      "config": {
        "default_yaw": 0,
        "default_pitch": 0,
        "default_hfov": 100,
        "auto_rotate": 0.5
      }
    },
    {
      "scene_id": "deluxe-ocean-balcony",
      "panorama_url": "https://cdn.../deluxe-ocean-balcony.jpg",
      "label": "Balcony",
      "is_default": false
    },
    {
      "scene_id": "deluxe-ocean-bathroom",
      "panorama_url": "https://cdn.../deluxe-ocean-bathroom.jpg",
      "label": "Bathroom",
      "is_default": false
    }
  ],
  "hotspots": [
    {
      "id": "uuid",
      "scene_id": "deluxe-ocean-bedroom",
      "type": "navigate",
      "pitch": -5.2,
      "yaw": 120.5,
      "label": "Balcony",
      "icon": "arrow",
      "target_scene_id": "deluxe-ocean-balcony"
    },
    {
      "id": "uuid",
      "scene_id": "deluxe-ocean-bedroom",
      "type": "info",
      "pitch": 10.0,
      "yaw": -45.3,
      "label": "Ocean View",
      "description": "Tầm nhìn 180° ra vịnh Nha Trang",
      "image_url": "https://cdn.../ocean-view-detail.jpg"
    },
    {
      "id": "uuid",
      "scene_id": "deluxe-ocean-bedroom",
      "type": "cta",
      "pitch": -20.0,
      "yaw": 200.0,
      "label": "Book Now",
      "cta_url": "https://...",
      "cta_label": "Check Availability"
    }
  ],
  "gallery": [
    {
      "id": "uuid",
      "image_url": "https://cdn.../gallery-1.jpg",
      "thumbnail_url": "https://cdn.../gallery-1-thumb.jpg",
      "caption": "Ocean view from balcony",
      "alt_text": "Deluxe ocean view balcony"
    }
  ]
}
```

#### Services

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/services` | Danh sách dịch vụ (filter: ?category=dining) |
| GET | `/services/:slug` | Chi tiết dịch vụ + scenes + hotspots + gallery |

#### Promotions

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/promotions` | Danh sách ưu đãi đang active |
| GET | `/promotions/:slug` | Chi tiết ưu đãi |

#### Posts

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/posts` | Danh sách bài viết (?category=news&page=1&limit=10) |
| GET | `/posts/:slug` | Chi tiết bài viết |

#### Scenes

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/scenes` | Danh sách tất cả scene configs |
| GET | `/scenes/:scene_id` | Chi tiết scene + hotspots |

#### Gallery

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/gallery` | Ảnh gallery (?parent_type=room&parent_id=uuid) |

#### Analytics (Public, write-only)

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| POST | `/analytics/event` | Ghi sự kiện (scene_view, hotspot_click, booking_click) |

**Request body:**
```json
{
  "event_type": "scene_view",
  "scene_id": "deluxe-ocean-bedroom",
  "page_path": "/rooms/deluxe-ocean",
  "session_id": "abc123",
  "metadata": { "duration_seconds": 15 }
}
```

### 2.3 Admin API Endpoints (Dashboard)

#### Authentication

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| POST | `/admin/auth/login` | Đăng nhập → JWT token |
| POST | `/admin/auth/refresh` | Refresh token |
| POST | `/admin/auth/logout` | Invalidate token |
| GET | `/admin/auth/me` | Thông tin user hiện tại |

#### CRUD Rooms

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/admin/rooms` | Danh sách (bao gồm draft) |
| POST | `/admin/rooms` | Tạo phòng mới |
| GET | `/admin/rooms/:id` | Chi tiết |
| PUT | `/admin/rooms/:id` | Cập nhật |
| DELETE | `/admin/rooms/:id` | Xóa |
| POST | `/admin/rooms/:id/gallery` | Upload ảnh gallery |
| DELETE | `/admin/rooms/:id/gallery/:img_id` | Xóa ảnh |
| PUT | `/admin/rooms/:id/scenes` | Cập nhật scene mapping |
| PUT | `/admin/rooms/:id/amenities` | Cập nhật tiện nghi |
| PUT | `/admin/rooms/:id/sort` | Đổi thứ tự |

#### CRUD Services, Promotions, Posts, Pages, Menus

Tương tự pattern CRUD rooms — mỗi entity có full CRUD + gallery upload.

#### Hotspot Management

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/admin/scenes/:scene_id/hotspots` | Danh sách hotspots |
| POST | `/admin/scenes/:scene_id/hotspots` | Thêm hotspot |
| PUT | `/admin/hotspots/:id` | Sửa hotspot |
| DELETE | `/admin/hotspots/:id` | Xóa hotspot |

#### Scene Management

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/admin/scenes` | Danh sách scene configs |
| POST | `/admin/scenes` | Tạo scene (upload panorama) |
| PUT | `/admin/scenes/:id` | Cập nhật cấu hình |
| DELETE | `/admin/scenes/:id` | Xóa scene |

#### Translations

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/admin/translations/:type/:id` | Lấy bản dịch |
| PUT | `/admin/translations/:type/:id` | Cập nhật bản dịch |

#### Upload

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| POST | `/admin/upload/image` | Upload ảnh thường (auto resize + thumb) |
| POST | `/admin/upload/panorama` | Upload ảnh 360 equirectangular |

**Response:**
```json
{
  "url": "https://cdn.../uploads/2026/03/image.jpg",
  "thumbnail_url": "https://cdn.../uploads/2026/03/image-thumb.jpg",
  "width": 1920,
  "height": 1080,
  "size_bytes": 524288
}
```

#### Dashboard Analytics

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/admin/analytics/overview` | Tổng quan (views, clicks, top scenes) |
| GET | `/admin/analytics/scenes` | Scene views ranked |
| GET | `/admin/analytics/hotspots` | Hotspot click ranked |
| GET | `/admin/analytics/booking-cta` | Booking CTA click stats |

---

## 3. Prisma Schema (Tóm tắt)

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Hotel {
  id          String   @id @default(uuid())
  name        String
  slug        String   @unique
  logoUrl     String?  @map("logo_url")
  description String?
  address     String?
  city        String?
  country     String   @default("Vietnam")
  phone       String?
  email       String?
  bookingUrl  String?  @map("booking_url")
  socialLinks Json     @default("{}") @map("social_links")
  settings    Json     @default("{}") 
  starRating  Int      @default(4) @map("star_rating")
  isActive    Boolean  @default(true) @map("is_active")
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")

  rooms        Room[]
  services     Service[]
  promotions   Promotion[]
  posts        Post[]
  menus        Menu[]
  users        User[]
  sceneConfigs SceneConfig[]
  hotspots     Hotspot[]
  translations Translation[]
  analytics    AnalyticsEvent[]

  @@map("hotels")
}

model Room {
  id             String   @id @default(uuid())
  hotelId        String   @map("hotel_id")
  name           String
  slug           String
  roomType       String   @default("standard") @map("room_type")
  areaSqm        Decimal? @map("area_sqm")
  maxAdults      Int      @default(2) @map("max_adults")
  maxChildren    Int      @default(1) @map("max_children")
  bedType        String?  @map("bed_type")
  description    String?
  shortDesc      String?  @map("short_desc")
  priceFrom      Decimal? @map("price_from")
  currency       String   @default("VND")
  viewType       String?  @map("view_type")
  highlights     Json     @default("[]")
  panoramaUrl    String?  @map("panorama_url")
  thumbnailUrl   String?  @map("thumbnail_url")
  defaultSceneId String?  @map("default_scene_id")
  bookingUrl     String?  @map("booking_url")
  seoTitle       String?  @map("seo_title")
  seoDescription String?  @map("seo_description")
  isPublished    Boolean  @default(false) @map("is_published")
  sortOrder      Int      @default(0) @map("sort_order")
  createdAt      DateTime @default(now()) @map("created_at")
  updatedAt      DateTime @updatedAt @map("updated_at")

  hotel     Hotel          @relation(fields: [hotelId], references: [id])
  amenities RoomAmenity[]
  scenes    RoomScene[]
  gallery   GalleryImage[]

  @@unique([hotelId, slug])
  @@map("rooms")
}

// ... (các model khác tương tự theo SQL schema ở trên)
```

---

## 4. Caching Strategy

### 4.1 Redis Cache Layers

| Cache Key Pattern | TTL | Invalidation |
|------------------|-----|-------------|
| `hotel:{slug}` | 1 giờ | Khi update hotel settings |
| `menu:{hotel_id}:{location}` | 30 phút | Khi update menu |
| `room:{slug}:{locale}` | 15 phút | Khi update room |
| `rooms:list:{hotel_id}:{locale}` | 15 phút | Khi update bất kỳ room |
| `service:{slug}:{locale}` | 15 phút | Khi update service |
| `scenes:{hotel_id}` | 30 phút | Khi update scene config |
| `hotspots:{scene_id}` | 15 phút | Khi update hotspots |
| `promotions:active:{hotel_id}` | 10 phút | Khi update promotions |

### 4.2 CDN Cache

- Ảnh panorama: Cache 30 ngày (immutable filename with hash)
- Gallery images: Cache 7 ngày
- Static assets (JS/CSS): Cache 1 năm (versioned)

---

*Tài liệu tiếp theo: **03-UI-UX-SPEC.md** — Luồng giao diện, component hierarchy, interaction patterns*
