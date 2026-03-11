# 02 — DASHBOARD DATABASE & API SPECIFICATION
## Tourism Map 360° — Admin Dashboard

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** `01-DASHBOARD-ARCHITECTURE` · `markdown-tourism/02-DATABASE-API-SPEC`

---

## Mục Lục

1. [Database Schema Tổng Quan](#1-database-schema-tổng-quan)
2. [Table Definitions](#2-table-definitions)
3. [Prisma Schema](#3-prisma-schema)
4. [Admin API Endpoints](#4-admin-api-endpoints)
5. [Public API Endpoints](#5-public-api-endpoints)
6. [File Upload API](#6-file-upload-api)
7. [Authentication API](#7-authentication-api)
8. [Response & Error Format](#8-response--error-format)

---

## 1. Database Schema Tổng Quan

### 1.1 Entity Relationship Diagram

```
┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  provinces   │────<│      pois        │────<│    poi_tags      │
│              │     │                  │     └────────┬─────────┘
│  id          │     │  id              │              │
│  name        │     │  province_id(FK) │     ┌────────┴─────────┐
│  slug        │     │  category_id(FK) │     │      tags        │
│  description │     │  name            │     │  id, name, slug  │
│  overview_*  │     │  slug            │     └──────────────────┘
│  map_center_*│     │  addr            │
│  is_published│     │  lat, lng        │     ┌──────────────────┐
│  created_at  │     │  panorama_url    │────<│   poi_scenes     │
│  updated_at  │     │  thumb_url       │     │  id, poi_id(FK)  │
└──────────────┘     │  description     │     │  panorama_url    │
                     │  hours, distance │     │  yaw/pitch/hfov  │
┌──────────────┐     │  emoji           │     │  label           │
│  categories  │────<│  hotspot_pitch   │     │  is_default      │
│              │     │  hotspot_yaw     │     │  sort_order      │
│  id          │     │  scene_yaw/pitch │     └──────────────────┘
│  name        │     │  scene_hfov      │
│  slug        │     │  is_published    │     ┌──────────────────┐
│  icon        │     │  sort_order      │────<│   poi_gallery    │
│  color       │     │  created_at      │     │  id, poi_id(FK)  │
│  sort_order  │     │  updated_at      │     │  image_url       │
│  is_active   │     └──────────────────┘     │  caption         │
└──────────────┘                              │  sort_order      │
                                              └──────────────────┘
┌──────────────┐
│    users     │     ┌──────────────────┐     ┌──────────────────┐
│  id          │     │  map_configs     │     │ analytics_events │
│  email       │     │  id              │     │  id              │
│  password    │     │  province_id(FK) │     │  event_type      │
│  name        │     │  center_lat      │     │  poi_id(FK)      │
│  role        │     │  center_lng      │     │  province_id(FK) │
│  avatar_url  │     │  default_zoom    │     │  session_id      │
│  is_active   │     │  min_zoom        │     │  device_type     │
│  last_login  │     │  max_zoom        │     │  created_at      │
│  created_at  │     │  tile_provider   │     └──────────────────┘
└──────────────┘     │  tile_filter_css │
                     │  updated_at      │
                     └──────────────────┘
```

### 1.2 Bảng Tổng Hợp

| Table | Mô tả | Rows (estimate) |
|-------|--------|:---:|
| `provinces` | Tỉnh/Thành phố | 5–63 |
| `categories` | Danh mục POI (Biển, Ẩm thực...) | 6–15 |
| `pois` | Điểm đến du lịch | 50–500 |
| `tags` | Tags tự do (Miễn phí, Hoàng hôn...) | 20–100 |
| `poi_tags` | Quan hệ N:N POI ↔ Tag | 100–2000 |
| `poi_scenes` | Multi-scene per POI (mở rộng) | 50–500 |
| `poi_gallery` | Ảnh gallery per POI | 100–2000 |
| `users` | Tài khoản admin | 1–20 |
| `map_configs` | Cấu hình bản đồ per tỉnh | 5–63 |
| `analytics_events` | Log sự kiện | 10K–1M+ |

---

## 2. Table Definitions

### 2.1 `provinces`

```sql
CREATE TABLE provinces (
  id                  SERIAL PRIMARY KEY,
  name                VARCHAR(200) NOT NULL,
  slug                VARCHAR(200) UNIQUE NOT NULL,
  description         TEXT,
  
  -- Overview panorama config
  overview_panorama   VARCHAR(500),          -- URL panorama tổng quan
  overview_yaw        DECIMAL(6,2) DEFAULT 0,
  overview_pitch      DECIMAL(6,2) DEFAULT 0,
  overview_hfov       DECIMAL(6,2) DEFAULT 120,
  overview_auto_rotate DECIMAL(4,2) DEFAULT 0.2,
  
  -- Map center (cho Leaflet)
  map_center_lat      DECIMAL(10,6) DEFAULT 10.40,
  map_center_lng      DECIMAL(10,6) DEFAULT 107.15,
  map_default_zoom    INT DEFAULT 9,
  
  -- Publishing
  is_published        BOOLEAN DEFAULT false,
  
  -- Metadata
  created_at          TIMESTAMPTZ DEFAULT NOW(),
  updated_at          TIMESTAMPTZ DEFAULT NOW()
);
```

### 2.2 `categories`

```sql
CREATE TABLE categories (
  id              SERIAL PRIMARY KEY,
  name            VARCHAR(100) NOT NULL,
  slug            VARCHAR(100) UNIQUE NOT NULL,
  icon            VARCHAR(50) NOT NULL,           -- Lucide icon name (e.g. "waves")
  color           VARCHAR(7),                     -- Hex color (e.g. "#22D3A7")
  sort_order      INT DEFAULT 0,
  is_active       BOOLEAN DEFAULT true,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Seed data (matching demo)
INSERT INTO categories (name, slug, icon, color, sort_order) VALUES
  ('Biển',        'beach',   'waves',    '#22D3A7', 1),
  ('Tâm linh',   'temple',  'church',   '#F59E0B', 2),
  ('Ẩm thực',    'food',    'utensils', '#EC4899', 3),
  ('Thiên nhiên', 'nature',  'trees',    '#3B82F6', 4),
  ('Di tích',    'history', 'landmark', '#8B5CF6', 5),
  ('Nghỉ dưỡng', 'resort',  'hotel',    '#EF4444', 6);
```

### 2.3 `pois`

```sql
CREATE TABLE pois (
  id              SERIAL PRIMARY KEY,
  province_id     INT NOT NULL REFERENCES provinces(id) ON DELETE CASCADE,
  category_id     INT NOT NULL REFERENCES categories(id),
  
  -- Basic info
  name            VARCHAR(200) NOT NULL,
  slug            VARCHAR(200) UNIQUE NOT NULL,
  addr            VARCHAR(300),
  description     TEXT,
  hours           VARCHAR(100),                   -- "Cả ngày", "8:00 - 17:00"
  distance        VARCHAR(100),                   -- "0 km từ trung tâm"
  emoji           VARCHAR(10),                    -- "🏖️", "⛰️"
  
  -- Geolocation (for Leaflet map)
  lat             DECIMAL(10,6) NOT NULL,
  lng             DECIMAL(10,6) NOT NULL,
  
  -- Hotspot position on overview panorama
  hotspot_pitch   DECIMAL(6,2) DEFAULT 0,
  hotspot_yaw     DECIMAL(6,2) DEFAULT 0,
  
  -- POI scene camera (khi click vào POI)
  panorama_url    VARCHAR(500),                   -- URL equirectangular panorama
  scene_yaw       DECIMAL(6,2) DEFAULT 0,
  scene_pitch     DECIMAL(6,2) DEFAULT 0,
  scene_hfov      DECIMAL(6,2) DEFAULT 100,
  
  -- Thumbnail
  thumb_url       VARCHAR(500),
  
  -- Status
  is_published    BOOLEAN DEFAULT true,
  sort_order      INT DEFAULT 0,
  
  -- Metadata
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_pois_province ON pois(province_id);
CREATE INDEX idx_pois_category ON pois(category_id);
CREATE INDEX idx_pois_published ON pois(is_published);
CREATE INDEX idx_pois_sort ON pois(sort_order);
```

### 2.4 `tags` & `poi_tags`

```sql
CREATE TABLE tags (
  id              SERIAL PRIMARY KEY,
  name            VARCHAR(50) NOT NULL,
  slug            VARCHAR(50) UNIQUE NOT NULL,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE poi_tags (
  poi_id          INT REFERENCES pois(id) ON DELETE CASCADE,
  tag_id          INT REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (poi_id, tag_id)
);
```

### 2.5 `poi_scenes`

```sql
CREATE TABLE poi_scenes (
  id              SERIAL PRIMARY KEY,
  poi_id          INT NOT NULL REFERENCES pois(id) ON DELETE CASCADE,
  panorama_url    VARCHAR(500) NOT NULL,
  yaw             DECIMAL(6,2) DEFAULT 0,
  pitch           DECIMAL(6,2) DEFAULT 0,
  hfov            DECIMAL(6,2) DEFAULT 100,
  label           VARCHAR(100),                   -- "Sảnh chính", "View biển"
  is_default      BOOLEAN DEFAULT false,
  sort_order      INT DEFAULT 0,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_poi_scenes_poi ON poi_scenes(poi_id);
```

### 2.6 `poi_gallery`

```sql
CREATE TABLE poi_gallery (
  id              SERIAL PRIMARY KEY,
  poi_id          INT NOT NULL REFERENCES pois(id) ON DELETE CASCADE,
  image_url       VARCHAR(500) NOT NULL,
  caption         VARCHAR(300),
  sort_order      INT DEFAULT 0,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_poi_gallery_poi ON poi_gallery(poi_id);
```

### 2.7 `users`

```sql
CREATE TABLE users (
  id              SERIAL PRIMARY KEY,
  email           VARCHAR(255) UNIQUE NOT NULL,
  password_hash   VARCHAR(255) NOT NULL,
  name            VARCHAR(100) NOT NULL,
  role            VARCHAR(20) NOT NULL DEFAULT 'viewer'
                  CHECK (role IN ('admin', 'editor', 'viewer')),
  avatar_url      VARCHAR(500),
  is_active       BOOLEAN DEFAULT true,
  last_login_at   TIMESTAMPTZ,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 2.8 `map_configs`

```sql
CREATE TABLE map_configs (
  id              SERIAL PRIMARY KEY,
  province_id     INT UNIQUE REFERENCES provinces(id) ON DELETE CASCADE,
  center_lat      DECIMAL(10,6) NOT NULL DEFAULT 10.40,
  center_lng      DECIMAL(10,6) NOT NULL DEFAULT 107.15,
  default_zoom    INT NOT NULL DEFAULT 9,
  min_zoom        INT DEFAULT 5,
  max_zoom        INT DEFAULT 18,
  tile_provider   VARCHAR(500) DEFAULT 'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png',
  tile_filter_css VARCHAR(200) DEFAULT 'brightness(.78) contrast(1.15) saturate(.85)',
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 2.9 `analytics_events`

```sql
CREATE TABLE analytics_events (
  id              BIGSERIAL PRIMARY KEY,
  event_type      VARCHAR(50) NOT NULL,            -- 'poi_view', 'category_filter', 'map_click', 'page_view'
  poi_id          INT REFERENCES pois(id) ON DELETE SET NULL,
  province_id     INT REFERENCES provinces(id) ON DELETE SET NULL,
  session_id      VARCHAR(100),
  device_type     VARCHAR(20),                     -- 'desktop', 'mobile', 'tablet'
  user_agent      TEXT,
  ip_address      VARCHAR(45),
  referrer        VARCHAR(500),
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_analytics_type ON analytics_events(event_type);
CREATE INDEX idx_analytics_poi ON analytics_events(poi_id);
CREATE INDEX idx_analytics_date ON analytics_events(created_at);
```

---

## 3. Prisma Schema

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Province {
  id                  Int       @id @default(autoincrement())
  name                String    @db.VarChar(200)
  slug                String    @unique @db.VarChar(200)
  description         String?   @db.Text
  
  overviewPanorama    String?   @map("overview_panorama") @db.VarChar(500)
  overviewYaw         Decimal   @default(0) @map("overview_yaw") @db.Decimal(6,2)
  overviewPitch       Decimal   @default(0) @map("overview_pitch") @db.Decimal(6,2)
  overviewHfov        Decimal   @default(120) @map("overview_hfov") @db.Decimal(6,2)
  overviewAutoRotate  Decimal   @default(0.2) @map("overview_auto_rotate") @db.Decimal(4,2)
  
  mapCenterLat        Decimal   @default(10.40) @map("map_center_lat") @db.Decimal(10,6)
  mapCenterLng        Decimal   @default(107.15) @map("map_center_lng") @db.Decimal(10,6)
  mapDefaultZoom      Int       @default(9) @map("map_default_zoom")
  
  isPublished         Boolean   @default(false) @map("is_published")
  createdAt           DateTime  @default(now()) @map("created_at")
  updatedAt           DateTime  @updatedAt @map("updated_at")
  
  pois                Poi[]
  mapConfig           MapConfig?
  analyticsEvents     AnalyticsEvent[]

  @@map("provinces")
}

model Category {
  id          Int       @id @default(autoincrement())
  name        String    @db.VarChar(100)
  slug        String    @unique @db.VarChar(100)
  icon        String    @db.VarChar(50)
  color       String?   @db.VarChar(7)
  sortOrder   Int       @default(0) @map("sort_order")
  isActive    Boolean   @default(true) @map("is_active")
  createdAt   DateTime  @default(now()) @map("created_at")
  
  pois        Poi[]

  @@map("categories")
}

model Poi {
  id            Int       @id @default(autoincrement())
  provinceId    Int       @map("province_id")
  categoryId    Int       @map("category_id")
  
  name          String    @db.VarChar(200)
  slug          String    @unique @db.VarChar(200)
  addr          String?   @db.VarChar(300)
  description   String?   @db.Text
  hours         String?   @db.VarChar(100)
  distance      String?   @db.VarChar(100)
  emoji         String?   @db.VarChar(10)
  
  lat           Decimal   @db.Decimal(10,6)
  lng           Decimal   @db.Decimal(10,6)
  
  hotspotPitch  Decimal   @default(0) @map("hotspot_pitch") @db.Decimal(6,2)
  hotspotYaw    Decimal   @default(0) @map("hotspot_yaw") @db.Decimal(6,2)
  
  panoramaUrl   String?   @map("panorama_url") @db.VarChar(500)
  sceneYaw      Decimal   @default(0) @map("scene_yaw") @db.Decimal(6,2)
  scenePitch    Decimal   @default(0) @map("scene_pitch") @db.Decimal(6,2)
  sceneHfov     Decimal   @default(100) @map("scene_hfov") @db.Decimal(6,2)
  
  thumbUrl      String?   @map("thumb_url") @db.VarChar(500)
  
  isPublished   Boolean   @default(true) @map("is_published")
  sortOrder     Int       @default(0) @map("sort_order")
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")
  
  province      Province  @relation(fields: [provinceId], references: [id], onDelete: Cascade)
  category      Category  @relation(fields: [categoryId], references: [id])
  tags          PoiTag[]
  scenes        PoiScene[]
  gallery       PoiGallery[]
  analyticsEvents AnalyticsEvent[]

  @@index([provinceId])
  @@index([categoryId])
  @@index([isPublished])
  @@map("pois")
}

model Tag {
  id          Int       @id @default(autoincrement())
  name        String    @db.VarChar(50)
  slug        String    @unique @db.VarChar(50)
  createdAt   DateTime  @default(now()) @map("created_at")
  
  pois        PoiTag[]

  @@map("tags")
}

model PoiTag {
  poiId       Int       @map("poi_id")
  tagId       Int       @map("tag_id")
  
  poi         Poi       @relation(fields: [poiId], references: [id], onDelete: Cascade)
  tag         Tag       @relation(fields: [tagId], references: [id], onDelete: Cascade)

  @@id([poiId, tagId])
  @@map("poi_tags")
}

model PoiScene {
  id            Int       @id @default(autoincrement())
  poiId         Int       @map("poi_id")
  panoramaUrl   String    @map("panorama_url") @db.VarChar(500)
  yaw           Decimal   @default(0) @db.Decimal(6,2)
  pitch         Decimal   @default(0) @db.Decimal(6,2)
  hfov          Decimal   @default(100) @db.Decimal(6,2)
  label         String?   @db.VarChar(100)
  isDefault     Boolean   @default(false) @map("is_default")
  sortOrder     Int       @default(0) @map("sort_order")
  createdAt     DateTime  @default(now()) @map("created_at")
  
  poi           Poi       @relation(fields: [poiId], references: [id], onDelete: Cascade)

  @@index([poiId])
  @@map("poi_scenes")
}

model PoiGallery {
  id          Int       @id @default(autoincrement())
  poiId       Int       @map("poi_id")
  imageUrl    String    @map("image_url") @db.VarChar(500)
  caption     String?   @db.VarChar(300)
  sortOrder   Int       @default(0) @map("sort_order")
  createdAt   DateTime  @default(now()) @map("created_at")
  
  poi         Poi       @relation(fields: [poiId], references: [id], onDelete: Cascade)

  @@index([poiId])
  @@map("poi_gallery")
}

model User {
  id            Int       @id @default(autoincrement())
  email         String    @unique @db.VarChar(255)
  passwordHash  String    @map("password_hash") @db.VarChar(255)
  name          String    @db.VarChar(100)
  role          String    @default("viewer") @db.VarChar(20)
  avatarUrl     String?   @map("avatar_url") @db.VarChar(500)
  isActive      Boolean   @default(true) @map("is_active")
  lastLoginAt   DateTime? @map("last_login_at")
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  @@map("users")
}

model MapConfig {
  id            Int       @id @default(autoincrement())
  provinceId    Int       @unique @map("province_id")
  centerLat     Decimal   @default(10.40) @map("center_lat") @db.Decimal(10,6)
  centerLng     Decimal   @default(107.15) @map("center_lng") @db.Decimal(10,6)
  defaultZoom   Int       @default(9) @map("default_zoom")
  minZoom       Int       @default(5) @map("min_zoom")
  maxZoom       Int       @default(18) @map("max_zoom")
  tileProvider  String    @default("https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png") @map("tile_provider") @db.VarChar(500)
  tileFilterCss String    @default("brightness(.78) contrast(1.15) saturate(.85)") @map("tile_filter_css") @db.VarChar(200)
  updatedAt     DateTime  @updatedAt @map("updated_at")
  
  province      Province  @relation(fields: [provinceId], references: [id], onDelete: Cascade)

  @@map("map_configs")
}

model AnalyticsEvent {
  id          BigInt    @id @default(autoincrement())
  eventType   String    @map("event_type") @db.VarChar(50)
  poiId       Int?      @map("poi_id")
  provinceId  Int?      @map("province_id")
  sessionId   String?   @map("session_id") @db.VarChar(100)
  deviceType  String?   @map("device_type") @db.VarChar(20)
  userAgent   String?   @map("user_agent") @db.Text
  ipAddress   String?   @map("ip_address") @db.VarChar(45)
  referrer    String?   @db.VarChar(500)
  createdAt   DateTime  @default(now()) @map("created_at")
  
  poi         Poi?      @relation(fields: [poiId], references: [id], onDelete: SetNull)
  province    Province? @relation(fields: [provinceId], references: [id], onDelete: SetNull)

  @@index([eventType])
  @@index([poiId])
  @@index([createdAt])
  @@map("analytics_events")
}
```

---

## 4. Admin API Endpoints

### 4.1 Provinces

| Method | Endpoint | Mô tả | Auth |
|--------|----------|--------|------|
| GET | `/api/admin/provinces` | List all provinces (paginated) | editor+ |
| GET | `/api/admin/provinces/:id` | Get province detail | editor+ |
| POST | `/api/admin/provinces` | Create province | editor+ |
| PATCH | `/api/admin/provinces/:id` | Update province | editor+ |
| DELETE | `/api/admin/provinces/:id` | Delete province + cascade POIs | admin |
| PATCH | `/api/admin/provinces/:id/publish` | Toggle publish status | admin |

#### POST/PATCH Body — Province

```json
{
  "name": "Bà Rịa — Vũng Tàu",
  "slug": "ba-ria-vung-tau",
  "description": "Tỉnh ven biển phía Đông Nam...",
  "overviewPanorama": "https://cdn.example.com/panoramas/brvt-overview.jpg",
  "overviewYaw": 80,
  "overviewPitch": 10,
  "overviewHfov": 120,
  "overviewAutoRotate": 0.2,
  "mapCenterLat": 10.40,
  "mapCenterLng": 107.15,
  "mapDefaultZoom": 9
}
```

### 4.2 Categories

| Method | Endpoint | Mô tả | Auth |
|--------|----------|--------|------|
| GET | `/api/admin/categories` | List categories (sorted) | editor+ |
| POST | `/api/admin/categories` | Create category | editor+ |
| PATCH | `/api/admin/categories/:id` | Update category | editor+ |
| DELETE | `/api/admin/categories/:id` | Delete (fails if has POIs) | admin |
| PATCH | `/api/admin/categories/reorder` | Reorder categories | editor+ |

#### POST/PATCH Body — Category

```json
{
  "name": "Biển",
  "slug": "beach",
  "icon": "waves",
  "color": "#22D3A7",
  "sortOrder": 1,
  "isActive": true
}
```

### 4.3 POIs

| Method | Endpoint | Mô tả | Auth |
|--------|----------|--------|------|
| GET | `/api/admin/pois` | List POIs (paginated, filterable) | editor+ |
| GET | `/api/admin/pois/:id` | Get POI detail + tags + scenes + gallery | editor+ |
| POST | `/api/admin/pois` | Create POI | editor+ |
| PATCH | `/api/admin/pois/:id` | Update POI | editor+ |
| DELETE | `/api/admin/pois/:id` | Delete POI + cascade | editor+ |
| PATCH | `/api/admin/pois/:id/publish` | Toggle publish | editor+ |
| PATCH | `/api/admin/pois/reorder` | Reorder POIs | editor+ |

#### Query Parameters — GET `/api/admin/pois`

| Param | Type | Mô tả |
|-------|------|--------|
| `page` | number | Page number (default 1) |
| `limit` | number | Items per page (default 20) |
| `province_id` | number | Filter by province |
| `category_id` | number | Filter by category |
| `q` | string | Search by name/addr |
| `is_published` | boolean | Filter by status |
| `sort` | string | `name`, `sort_order`, `created_at` |
| `order` | string | `asc`, `desc` |

#### POST/PATCH Body — POI

```json
{
  "provinceId": 1,
  "categoryId": 2,
  "name": "Bãi Trước",
  "slug": "bai-truoc",
  "addr": "Tp. Vũng Tàu",
  "description": "Bãi Trước nằm ngay trung tâm...",
  "hours": "Cả ngày",
  "distance": "0 km từ trung tâm",
  "emoji": "🏖️",
  "lat": 10.3460,
  "lng": 107.0723,
  "hotspotPitch": -10,
  "hotspotYaw": 40,
  "panoramaUrl": "https://cdn.example.com/panoramas/bai-truoc.jpg",
  "sceneYaw": 30,
  "scenePitch": 0,
  "sceneHfov": 105,
  "thumbUrl": "https://cdn.example.com/thumbs/bai-truoc.jpg",
  "tagIds": [1, 3, 5],
  "isPublished": true,
  "sortOrder": 1
}
```

### 4.4 Tags

| Method | Endpoint | Mô tả | Auth |
|--------|----------|--------|------|
| GET | `/api/admin/tags` | List all tags | editor+ |
| POST | `/api/admin/tags` | Create tag | editor+ |
| PATCH | `/api/admin/tags/:id` | Update tag | editor+ |
| DELETE | `/api/admin/tags/:id` | Delete tag (remove from POIs) | editor+ |

### 4.5 POI Scenes

| Method | Endpoint | Mô tả | Auth |
|--------|----------|--------|------|
| GET | `/api/admin/pois/:poiId/scenes` | List scenes for POI | editor+ |
| POST | `/api/admin/pois/:poiId/scenes` | Add scene to POI | editor+ |
| PATCH | `/api/admin/pois/:poiId/scenes/:id` | Update scene | editor+ |
| DELETE | `/api/admin/pois/:poiId/scenes/:id` | Delete scene | editor+ |
| PATCH | `/api/admin/pois/:poiId/scenes/reorder` | Reorder scenes | editor+ |

### 4.6 POI Gallery

| Method | Endpoint | Mô tả | Auth |
|--------|----------|--------|------|
| GET | `/api/admin/pois/:poiId/gallery` | List gallery images | editor+ |
| POST | `/api/admin/pois/:poiId/gallery` | Add image to gallery | editor+ |
| PATCH | `/api/admin/pois/:poiId/gallery/:id` | Update caption/order | editor+ |
| DELETE | `/api/admin/pois/:poiId/gallery/:id` | Delete image | editor+ |
| PATCH | `/api/admin/pois/:poiId/gallery/reorder` | Reorder images | editor+ |

### 4.7 Map Config

| Method | Endpoint | Mô tả | Auth |
|--------|----------|--------|------|
| GET | `/api/admin/map-config/:provinceId` | Get map config for province | editor+ |
| PUT | `/api/admin/map-config/:provinceId` | Update map config | admin |

#### PUT Body — Map Config

```json
{
  "centerLat": 10.40,
  "centerLng": 107.15,
  "defaultZoom": 9,
  "minZoom": 5,
  "maxZoom": 18,
  "tileProvider": "https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png",
  "tileFilterCss": "brightness(.78) contrast(1.15) saturate(.85)"
}
```

### 4.8 Analytics

| Method | Endpoint | Mô tả | Auth |
|--------|----------|--------|------|
| GET | `/api/admin/analytics/overview` | Dashboard stats (total POIs, views, etc.) | viewer+ |
| GET | `/api/admin/analytics/top-pois` | Top viewed POIs | viewer+ |
| GET | `/api/admin/analytics/categories` | Views by category | viewer+ |
| GET | `/api/admin/analytics/devices` | Device type breakdown | viewer+ |
| GET | `/api/admin/analytics/timeline` | Views over time | viewer+ |

### 4.9 Users

| Method | Endpoint | Mô tả | Auth |
|--------|----------|--------|------|
| GET | `/api/admin/users` | List all users | admin |
| POST | `/api/admin/users` | Create user | admin |
| PATCH | `/api/admin/users/:id` | Update user (name, role, active) | admin |
| DELETE | `/api/admin/users/:id` | Delete user | admin |

### 4.10 Publish / Export

| Method | Endpoint | Mô tả | Auth |
|--------|----------|--------|------|
| POST | `/api/admin/publish/:provinceId` | Export province data as JSON for static site | admin |
| GET | `/api/admin/export/:provinceId` | Download JSON export file | admin |

#### Export JSON Format (for static site injection)

```json
{
  "CATEGORIES": [
    {"id":"beach","label":"Biển","icon":"waves","color":"#22D3A7"}
  ],
  "OVERVIEW": {
    "panorama": "https://...",
    "yaw": 80, "pitch": 10, "hfov": 120, "autoRotate": 0.2
  },
  "POIS": [
    {
      "id": 1, "cat": "beach", "name": "Bãi Trước",
      "addr": "Tp. Vũng Tàu", "lat": 10.346, "lng": 107.072,
      "pitch": -10, "yaw": 40,
      "panorama": "https://...",
      "pYaw": 30, "pPitch": 0, "pHfov": 105,
      "thumb": "https://...",
      "desc": "...", "tags": ["Biển","Miễn phí"],
      "hours": "Cả ngày", "distance": "0 km từ trung tâm",
      "emoji": "🏖️"
    }
  ],
  "MAP_CONFIG": {
    "center": [10.40, 107.15],
    "zoom": 9,
    "tileProvider": "https://..."
  }
}
```

---

## 5. Public API Endpoints

Nếu chọn **Option B (API-Backed)** thay vì static export:

| Method | Endpoint | Mô tả | Auth |
|--------|----------|--------|------|
| GET | `/api/public/provinces` | List published provinces | None |
| GET | `/api/public/provinces/:slug` | Province detail + overview config | None |
| GET | `/api/public/provinces/:slug/pois` | POIs of province (published only) | None |
| GET | `/api/public/categories` | Active categories | None |
| GET | `/api/public/pois/:id` | POI detail + tags + gallery | None |
| POST | `/api/public/analytics` | Track event (poi_view, page_view) | None |

---

## 6. File Upload API

### 6.1 Endpoints

| Method | Endpoint | Mô tả | Max Size |
|--------|----------|--------|----------|
| POST | `/api/admin/upload/panorama` | Upload equirectangular panorama | 30MB |
| POST | `/api/admin/upload/thumbnail` | Upload POI thumbnail | 5MB |
| POST | `/api/admin/upload/gallery` | Upload gallery image | 10MB |
| DELETE | `/api/admin/upload/:key` | Delete file from storage | — |

### 6.2 Upload Flow

```
Client                  API                    S3/R2
  │                       │                      │
  │─ POST /upload/panorama│                      │
  │  (multipart/form-data)│                      │
  │──────────────────────►│                      │
  │                       │─ Validate (type, size)│
  │                       │─ Generate unique key  │
  │                       │─ Sharp: create thumb  │
  │                       │─ Upload original ────►│
  │                       │─ Upload thumbnail ───►│
  │                       │◄─ URLs ──────────────┤
  │◄── { url, thumbUrl } ─┤                      │
  │                       │                      │
```

### 6.3 Image Processing

| Type | Original | Thumbnail | Format |
|------|----------|-----------|--------|
| Panorama | Keep original (max 8192px wide) | 400×200 crop | JPEG 85% |
| Thumbnail | 400×400 max | 120×120 crop center | JPEG 80% |
| Gallery | 1920px max width | 300×200 crop | JPEG 85% |

---

## 7. Authentication API

### 7.1 Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| POST | `/api/auth/login` | Email + password login |
| POST | `/api/auth/logout` | Clear session |
| GET | `/api/auth/me` | Get current user info |
| PATCH | `/api/auth/password` | Change password |

### 7.2 NextAuth.js Config

```typescript
// auth.config.ts
export const authConfig = {
  providers: [
    CredentialsProvider({
      credentials: {
        email: { type: "email" },
        password: { type: "password" }
      },
      authorize: async (credentials) => {
        // Validate against users table
        // bcrypt.compare(password, passwordHash)
        // Return user object with role
      }
    })
  ],
  callbacks: {
    jwt: ({ token, user }) => {
      if (user) { token.role = user.role; }
      return token;
    },
    session: ({ session, token }) => {
      session.user.role = token.role;
      return session;
    }
  }
};
```

---

## 8. Response & Error Format

### 8.1 Success Response

```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "total": 45,
    "page": 1,
    "limit": 20,
    "totalPages": 3
  }
}
```

### 8.2 Error Response

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Name is required",
    "details": [
      { "field": "name", "message": "Name must be at least 2 characters" }
    ]
  }
}
```

### 8.3 Error Codes

| Code | HTTP Status | Mô tả |
|------|:-----------:|--------|
| `VALIDATION_ERROR` | 400 | Invalid input data |
| `UNAUTHORIZED` | 401 | Not logged in |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `CONFLICT` | 409 | Duplicate slug/email |
| `UPLOAD_ERROR` | 413 | File too large |
| `SERVER_ERROR` | 500 | Internal error |

---

*End of Document*
