# 02 — DATABASE & API SPECIFICATION
## Du Lịch Bà Rịa — Bản Đồ 360° Tourism Platform

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** 01-SYSTEM-ARCHITECTURE · 03-UI-UX-SPEC · 04-TECHNICAL-IMPLEMENTATION  
**Note:** Demo hiện tại dùng inline JS arrays. Tài liệu này mô tả cấu trúc dữ liệu hiện tại và đề xuất schema cho production API.

---

## Mục Lục

1. [Data Structure Hiện Tại (Inline JS)](#1-data-structure-hiện-tại-inline-js)
2. [Chi Tiết Từng Entity](#2-chi-tiết-từng-entity)
3. [Đề Xuất Database Schema (PostgreSQL)](#3-đề-xuất-database-schema-postgresql)
4. [Đề Xuất REST API](#4-đề-xuất-rest-api)
5. [Caching Strategy](#5-caching-strategy)
6. [Sequence Diagrams](#6-sequence-diagrams)

---

## 1. Data Structure Hiện Tại (Inline JS)

### 1.1 Tổng Quan

Demo hiện tại (`tourism-map-360.html`) lưu toàn bộ dữ liệu trực tiếp trong `<script>` tag dưới dạng 3 JavaScript constants:

```
┌─────────────────────────────────────────────────────┐
│                   DATA LAYER                         │
│                                                      │
│  const CATEGORIES = [...]   ← 7 entries             │
│  const OVERVIEW   = {...}   ← 1 object              │
│  const POIS       = [...]   ← 8 entries             │
│                                                      │
│  State variables:                                    │
│  ├── viewer        (Pannellum instance)              │
│  ├── map           (Leaflet instance)                │
│  ├── activeCats    (Set — selected filter cats)      │
│  ├── activePOI     (current POI object | null)       │
│  ├── viewMode      ("overview" | "poi")              │
│  ├── poiPanelOpen  (boolean)                         │
│  ├── minimapVisible(boolean)                         │
│  ├── mapExpanded   (boolean)                         │
│  ├── panelExpanded (boolean)                         │
│  └── markers[]     ({poi, marker} — Leaflet refs)    │
└─────────────────────────────────────────────────────┘
```

---

## 2. Chi Tiết Từng Entity

### 2.1 CATEGORIES

Mảng chứa danh sách category để phân loại POI.

```javascript
const CATEGORIES = [
  {id:"all",    label:"Tất cả",      icon:"globe",    color:undefined},
  {id:"beach",  label:"Biển",        icon:"waves",    color:"#22D3A7"},
  {id:"temple", label:"Tâm linh",    icon:"church",   color:"#F59E0B"},
  {id:"food",   label:"Ẩm thực",     icon:"utensils", color:"#EC4899"},
  {id:"nature", label:"Thiên nhiên", icon:"trees",    color:"#3B82F6"},
  {id:"history",label:"Di tích",     icon:"landmark", color:"#8B5CF6"},
  {id:"resort", label:"Nghỉ dưỡng",  icon:"hotel",    color:"#EF4444"},
];
```

#### Schema

| Field | Type | Required | Mô tả |
|-------|------|----------|--------|
| `id` | string | ✅ | Unique identifier, dùng làm CSS class (`cat-{id}`) |
| `label` | string | ✅ | Tên hiển thị trên filter chip |
| `icon` | string | ✅ | Tên Lucide icon (`waves`, `church`, ...) |
| `color` | string | ❌ | Hex color cho category (null cho "all") |

#### Lưu ý
- Entry `{id:"all"}` **không được render** trong filter chips (`CATEGORIES.filter(c=>c.id!=="all")`)
- `icon` field lưu **tên Lucide icon** (không phải inline SVG)
- `color` được dùng cho: filter chip active state, marker pin background, detail badge, tags border

### 2.2 OVERVIEW

Object chứa cấu hình scene tổng quan (khi chưa chọn POI).

```javascript
const OVERVIEW = {
  panorama: "https://pannellum.org/images/cerro-toco-0.jpg",
  yaw: 80,
  pitch: 10,
  hfov: 120,
  autoRotate: 0.2
};
```

#### Schema

| Field | Type | Required | Mô tả |
|-------|------|----------|--------|
| `panorama` | string (URL) | ✅ | URL ảnh equirectangular panorama |
| `yaw` | number | ✅ | Góc xoay ngang ban đầu (độ) |
| `pitch` | number | ✅ | Góc nhìn dọc ban đầu (độ) |
| `hfov` | number | ✅ | Horizontal field of view (độ) |
| `autoRotate` | number | ✅ | Tốc độ tự xoay (độ/giây, 0 = tắt) |

### 2.3 POIS (Points of Interest)

Mảng chứa 8 điểm đến du lịch.

```javascript
{
  id: 1,                           // Unique ID
  cat: "beach",                    // Category ID (FK → CATEGORIES.id)
  name: "Bãi Trước",              // Tên hiển thị
  addr: "Tp. Vũng Tàu",           // Địa chỉ ngắn
  lat: 10.3460,                    // Vĩ độ (Leaflet marker)
  lng: 107.0723,                   // Kinh độ (Leaflet marker)
  pitch: -10,                      // Vị trí hotspot trên overview (pitch)
  yaw: 40,                         // Vị trí hotspot trên overview (yaw)
  panorama: "https://...",         // URL panorama riêng của POI
  pYaw: 30,                        // Camera yaw cho POI scene
  pPitch: 0,                       // Camera pitch cho POI scene
  pHfov: 105,                      // Camera hfov cho POI scene
  thumb: "https://...",             // URL thumbnail (120x120)
  desc: "Mô tả chi tiết...",       // Mô tả dài
  tags: ["Biển","Miễn phí"],       // Tags hiển thị
  hours: "Cả ngày",                // Giờ mở cửa
  distance: "0 km từ trung tâm",   // Khoảng cách
  emoji: "🏖️"                      // Emoji cho hotspot marker
}
```

#### Schema Đầy Đủ

| Field | Type | Required | Mô tả |
|-------|------|----------|--------|
| `id` | number | ✅ | Unique identifier |
| `cat` | string | ✅ | Category ID (`beach`, `temple`, `food`, `nature`, `history`, `resort`) |
| `name` | string | ✅ | Tên POI |
| `addr` | string | ✅ | Địa chỉ ngắn gọn |
| `lat` | number | ✅ | Latitude (WGS84) |
| `lng` | number | ✅ | Longitude (WGS84) |
| `pitch` | number | ✅ | Vị trí hotspot trên overview panorama (trục dọc, độ) |
| `yaw` | number | ✅ | Vị trí hotspot trên overview panorama (trục ngang, độ) |
| `panorama` | string (URL) | ✅ | URL equirectangular panorama cho POI scene |
| `pYaw` | number | ✅ | Camera yaw khi mở POI scene |
| `pPitch` | number | ✅ | Camera pitch khi mở POI scene |
| `pHfov` | number | ✅ | Camera hfov khi mở POI scene |
| `thumb` | string (URL) | ✅ | URL thumbnail image |
| `desc` | string | ✅ | Mô tả chi tiết (HTML-safe) |
| `tags` | string[] | ✅ | Danh sách tags |
| `hours` | string | ✅ | Giờ mở cửa / hoạt động |
| `distance` | string | ✅ | Khoảng cách từ trung tâm |
| `emoji` | string | ✅ | Emoji icon cho hotspot marker |

### 2.4 Danh Sách POI Hiện Tại

| # | ID | Cat | Tên | Addr | Lat/Lng |
|---|-----|-----|-----|------|---------|
| 1 | 1 | beach | Bãi Trước | Tp. Vũng Tàu | 10.346, 107.072 |
| 2 | 2 | beach | Bãi Sau | Tp. Vũng Tàu | 10.338, 107.085 |
| 3 | 3 | temple | Tượng Chúa Kitô | Núi Nhỏ, Vũng Tàu | 10.328, 107.085 |
| 4 | 4 | nature | Núi Dinh | H. Tân Thành | 10.503, 107.098 |
| 5 | 5 | food | Chợ Hải Sản Vũng Tàu | Đường Trần Phú | 10.350, 107.075 |
| 6 | 6 | history | Bạch Dinh | Tp. Vũng Tàu | 10.340, 107.072 |
| 7 | 7 | resort | Hồ Tràm Strip | Xuyên Mộc | 10.476, 107.375 |
| 8 | 8 | nature | Côn Đảo | H. Côn Đảo | 8.683, 106.609 |

### 2.5 Phân Bổ Theo Category

| Category | Count | POIs |
|----------|-------|------|
| beach | 2 | Bãi Trước, Bãi Sau |
| temple | 1 | Tượng Chúa Kitô |
| food | 1 | Chợ Hải Sản Vũng Tàu |
| nature | 2 | Núi Dinh, Côn Đảo |
| history | 1 | Bạch Dinh |
| resort | 1 | Hồ Tràm Strip |

---

## 3. Đề Xuất Database Schema (PostgreSQL)

### 3.1 Entity Relationship Diagram

```
┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  provinces   │────<│    pois          │────<│   poi_tags       │
│              │     │                  │     └──────────────────┘
│  id          │     │  id              │            │
│  name        │     │  province_id(FK) │     ┌──────┴───────────┐
│  slug        │     │  category_id(FK) │     │     tags         │
│  description │     │  name            │     │  id, name        │
│  overview_   │     │  slug            │     └──────────────────┘
│  panorama    │     │  addr            │
│  overview_   │     │  lat             │     ┌──────────────────┐
│  yaw/pitch/  │     │  lng             │────<│   poi_scenes     │
│  hfov        │     │  panorama_url    │     │                  │
│  created_at  │     │  thumb_url       │     │  id              │
│  updated_at  │     │  description     │     │  poi_id (FK)     │
└──────────────┘     │  hours           │     │  panorama_url    │
                     │  distance        │     │  yaw/pitch/hfov  │
                     │  emoji           │     │  label           │
┌──────────────┐     │  hotspot_pitch   │     │  is_default      │
│  categories  │────<│  hotspot_yaw     │     │  sort_order      │
│              │     │  p_yaw/pitch/    │     └──────────────────┘
│  id          │     │  p_hfov          │
│  name        │     │  is_published    │     ┌──────────────────┐
│  slug        │     │  sort_order      │────<│   poi_gallery    │
│  icon        │     │  created_at      │     │  id              │
│  color       │     │  updated_at      │     │  poi_id (FK)     │
│  sort_order  │     └──────────────────┘     │  image_url       │
└──────────────┘                              │  caption         │
                                              │  sort_order      │
                                              └──────────────────┘
```

### 3.2 Table Definitions

#### `provinces`

```sql
CREATE TABLE provinces (
  id            SERIAL PRIMARY KEY,
  name          VARCHAR(200) NOT NULL,
  slug          VARCHAR(200) UNIQUE NOT NULL,
  description   TEXT,
  overview_panorama VARCHAR(500),
  overview_yaw  DECIMAL(6,2) DEFAULT 0,
  overview_pitch DECIMAL(6,2) DEFAULT 0,
  overview_hfov DECIMAL(6,2) DEFAULT 120,
  overview_auto_rotate DECIMAL(4,2) DEFAULT 0.2,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);
```

#### `categories`

```sql
CREATE TABLE categories (
  id            SERIAL PRIMARY KEY,
  name          VARCHAR(100) NOT NULL,
  slug          VARCHAR(100) UNIQUE NOT NULL,
  icon          VARCHAR(50) NOT NULL,        -- Lucide icon name
  color         VARCHAR(7),                  -- Hex color (#22D3A7)
  sort_order    INT DEFAULT 0,
  created_at    TIMESTAMPTZ DEFAULT NOW()
);
```

#### `pois`

```sql
CREATE TABLE pois (
  id            SERIAL PRIMARY KEY,
  province_id   INT REFERENCES provinces(id),
  category_id   INT REFERENCES categories(id),
  name          VARCHAR(200) NOT NULL,
  slug          VARCHAR(200) UNIQUE NOT NULL,
  addr          VARCHAR(300),
  lat           DECIMAL(10,6) NOT NULL,
  lng           DECIMAL(10,6) NOT NULL,
  panorama_url  VARCHAR(500),
  thumb_url     VARCHAR(500),
  description   TEXT,
  hours         VARCHAR(100),
  distance      VARCHAR(100),
  emoji         VARCHAR(10),
  hotspot_pitch DECIMAL(6,2) DEFAULT 0,
  hotspot_yaw   DECIMAL(6,2) DEFAULT 0,
  scene_yaw     DECIMAL(6,2) DEFAULT 0,
  scene_pitch   DECIMAL(6,2) DEFAULT 0,
  scene_hfov    DECIMAL(6,2) DEFAULT 100,
  is_published  BOOLEAN DEFAULT true,
  sort_order    INT DEFAULT 0,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_pois_province ON pois(province_id);
CREATE INDEX idx_pois_category ON pois(category_id);
CREATE INDEX idx_pois_location ON pois USING gist (
  ST_SetSRID(ST_MakePoint(lng, lat), 4326)
);
```

#### `tags` & `poi_tags`

```sql
CREATE TABLE tags (
  id            SERIAL PRIMARY KEY,
  name          VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE poi_tags (
  poi_id        INT REFERENCES pois(id) ON DELETE CASCADE,
  tag_id        INT REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (poi_id, tag_id)
);
```

---

## 4. Đề Xuất REST API

### 4.1 Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| GET | `/api/provinces` | Danh sách provinces |
| GET | `/api/provinces/:slug` | Chi tiết province + overview config |
| GET | `/api/provinces/:slug/pois` | Danh sách POIs của province |
| GET | `/api/categories` | Danh sách categories |
| GET | `/api/pois/:id` | Chi tiết 1 POI |
| GET | `/api/pois?cat=beach&q=bãi` | Search + filter POIs |

### 4.2 Response Format

#### GET `/api/provinces/ba-ria-vung-tau/pois`

```json
{
  "data": [
    {
      "id": 1,
      "category": {
        "id": "beach",
        "label": "Biển",
        "icon": "waves",
        "color": "#22D3A7"
      },
      "name": "Bãi Trước",
      "addr": "Tp. Vũng Tàu",
      "lat": 10.3460,
      "lng": 107.0723,
      "panorama_url": "https://cdn.example.com/panoramas/bai-truoc.jpg",
      "thumb_url": "https://cdn.example.com/thumbs/bai-truoc.jpg",
      "description": "Bãi Trước nằm ngay trung tâm...",
      "tags": ["Biển", "Miễn phí", "Hoàng hôn"],
      "hours": "Cả ngày",
      "distance": "0 km từ trung tâm",
      "emoji": "🏖️",
      "hotspot": { "pitch": -10, "yaw": 40 },
      "camera": { "yaw": 30, "pitch": 0, "hfov": 105 }
    }
  ],
  "meta": {
    "total": 8,
    "province": "Bà Rịa — Vũng Tàu"
  }
}
```

### 4.3 Query Parameters

| Param | Type | Mô tả |
|-------|------|--------|
| `cat` | string | Filter by category slug (multi: `cat=beach,nature`) |
| `q` | string | Search by name/addr (fuzzy) |
| `lat` / `lng` / `radius` | number | Geo search (km) |
| `page` / `limit` | number | Pagination |
| `sort` | string | `name`, `distance`, `created_at` |

---

## 5. Caching Strategy

### 5.1 Demo (Static File)

Không cần cache strategy — toàn bộ data đã inline trong HTML.

### 5.2 Production (API-backed)

| Resource | TTL | Strategy |
|----------|-----|----------|
| Categories | 24h | CDN cache + localStorage |
| POI list | 1h | CDN cache, revalidate on filter |
| POI detail | 1h | CDN cache |
| Panorama images | 30d | CDN cache (immutable URLs) |
| Thumbnails | 7d | CDN cache |
| Map tiles | Browser default | CartoDB CDN handles |

---

## 6. Sequence Diagrams

### 6.1 Data Flow: Demo (Current)

```
Browser                     Inline JS
  │                            │
  │─ DOMContentLoaded ────────►│
  │                            │
  │  Read CATEGORIES ─────────►│──► renderCategories()
  │  Read POIS ───────────────►│──► renderPOIList()
  │  Read OVERVIEW ───────────►│──► loadOverview()
  │                            │
  │  toggleCat() ─────────────►│
  │  │  activeCats.add/delete  │
  │  │  getFilteredPOIs() ────►│──► filter POIS by Set
  │  │  renderPOIList() ──────►│──► rebuild DOM
  │  │  loadOverview() ───────►│──► recreate hotspots
  │                            │
```

### 6.2 Data Flow: Production (Proposed)

```
Browser           API Server          Database         CDN
  │                   │                  │               │
  │─ GET /categories ►│                  │               │
  │                   │─ SELECT * ──────►│               │
  │                   │◄─ rows ─────────┤               │
  │◄── JSON ─────────┤                  │               │
  │                   │                  │               │
  │─ GET /pois?cat= ─►│                  │               │
  │                   │─ SELECT...WHERE ►│               │
  │                   │◄─ rows ─────────┤               │
  │◄── JSON ─────────┤                  │               │
  │                   │                  │               │
  │─ Load panorama ──►│                  │               │
  │                   │──────────────────────── GET ────►│
  │◄──────────────────────────────────────── image ────┤│
  │                   │                  │               │
```

---

*End of Document*
