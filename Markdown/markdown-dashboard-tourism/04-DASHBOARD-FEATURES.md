# 04 — DASHBOARD FEATURES & WORKFLOWS
## Tourism Map 360° — Admin Dashboard

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** `01-DASHBOARD-ARCHITECTURE` · `02-DASHBOARD-DATABASE-API` · `03-DASHBOARD-UI-UX`

---

## Mục Lục

1. [Province Management](#1-province-management)
2. [Category Management](#2-category-management)
3. [POI Management (Core)](#3-poi-management-core)
4. [Tag Management](#4-tag-management)
5. [Map Configuration](#5-map-configuration)
6. [Gallery Management](#6-gallery-management)
7. [Scene Management (Multi-Scene)](#7-scene-management-multi-scene)
8. [Panorama & Hotspot Editor](#8-panorama--hotspot-editor)
9. [Publish & Export System](#9-publish--export-system)
10. [User Management](#10-user-management)
11. [Analytics Dashboard](#11-analytics-dashboard)
12. [Settings](#12-settings)

---

## 1. Province Management

### 1.1 Tổng Quan

Province (Tỉnh/Thành) là entity cấp cao nhất — mỗi province có overview panorama riêng, danh sách POIs, và cấu hình bản đồ.

### 1.2 CRUD Flow

```
[Province List Page]
    │
    ├── [+ Thêm Tỉnh] → Create form
    │       ├── Nhập tên, slug (auto-generate), mô tả
    │       ├── Upload overview panorama
    │       ├── Cấu hình camera (yaw/pitch/hfov/autoRotate)
    │       ├── Đặt map center + zoom
    │       └── [Lưu] → POST /api/admin/provinces
    │
    ├── [✏️ Edit] → Edit form (tabbed)
    │       ├── Tab 1: Thông tin cơ bản
    │       ├── Tab 2: Overview Panorama (Pannellum preview)
    │       ├── Tab 3: Cấu hình bản đồ (Leaflet preview)
    │       └── [Lưu] → PATCH /api/admin/provinces/:id
    │
    ├── [📤 Publish] → Toggle is_published
    │       └── PATCH /api/admin/provinces/:id/publish
    │
    └── [🗑 Xoá] → Confirm dialog
            ├── Warning: "Xoá tỉnh sẽ xoá toàn bộ X điểm đến"
            └── DELETE /api/admin/provinces/:id
```

### 1.3 Overview Panorama Config

Khi admin upload panorama overview cho tỉnh:

1. Upload equirectangular image → S3/R2
2. Pannellum preview hiển thị inline
3. Admin xoay panorama đến góc nhìn mong muốn
4. Click **"📸 Capture Current View"** → tự động lưu `yaw`, `pitch`, `hfov` hiện tại
5. Hoặc nhập thủ công giá trị yaw/pitch/hfov/autoRotate

### 1.4 Map Center Config

1. Leaflet map hiển thị với tất cả POI markers
2. Admin click trên map → marker xuất hiện tại center
3. Lat/Lng auto-fill vào input fields
4. Chọn default zoom level (5–18)
5. Preview bản đồ với cài đặt mới

### 1.5 Validations

| Field | Rule |
|-------|------|
| `name` | Required, 2–200 chars |
| `slug` | Required, unique, auto-generate from name |
| `overview_panorama` | Optional, must be valid image URL |
| `overview_yaw` | Number, -180 to 180 |
| `overview_pitch` | Number, -90 to 90 |
| `overview_hfov` | Number, 30 to 150 |
| `overview_auto_rotate` | Number, 0 to 5 |
| `map_center_lat` | Number, -90 to 90 |
| `map_center_lng` | Number, -180 to 180 |
| `map_default_zoom` | Integer, 1 to 18 |

---

## 2. Category Management

### 2.1 Tổng Quan

Categories (Danh mục) phân loại POI — mỗi category có tên, Lucide icon, và hex color. Tương ứng với CATEGORIES array trong frontend.

### 2.2 CRUD Flow

```
[Category List Page — Sortable Table]
    │
    ├── [+ Thêm Danh Mục]
    │       ├── Tên: [input text]
    │       ├── Slug: [auto-generate]
    │       ├── Icon: [Icon Picker — Lucide]
    │       ├── Màu: [Color Picker — hex]
    │       └── [Lưu] → POST /api/admin/categories
    │
    ├── [≡ Drag to Reorder]
    │       └── PATCH /api/admin/categories/reorder
    │           Body: { ids: [3, 1, 5, 2, 4, 6] }
    │
    ├── [Inline Edit] → Click row to expand edit form
    │       └── PATCH /api/admin/categories/:id
    │
    └── [🗑 Xoá]
            ├── Nếu category có POIs → Block delete
            │   "Danh mục này đang có 5 điểm đến. Chuyển POIs sang danh mục khác trước."
            └── Nếu không có POIs → DELETE (confirm)
```

### 2.3 Lucide Icon Picker Workflow

1. Admin click "Pick Icon" → IconPicker dialog opens
2. Grid hiển thị tất cả Lucide icons (tourism-related icons ở đầu)
3. Search input filter icons by name
4. **Suggested icons cho tourism:**
   - `waves`, `church`, `utensils`, `trees`, `landmark`, `hotel`
   - `mountain`, `palm-tree`, `anchor`, `compass`, `tent`, `map`
   - `camera`, `sun`, `umbrella`, `fish`, `ship`, `train`
5. Click icon → chọn, preview hiển thị bên cạnh
6. Icon name lưu vào DB (e.g., `"waves"`)

### 2.4 Color Picker Workflow

1. Admin click "Pick Color" → ColorPicker popover opens
2. **Preset swatches (matching frontend):**
   - `#22D3A7` (teal/beach)
   - `#F59E0B` (amber/temple)
   - `#EC4899` (pink/food)
   - `#3B82F6` (blue/nature)
   - `#8B5CF6` (purple/history)
   - `#EF4444` (red/resort)
3. Custom color wheel (react-colorful)
4. Manual hex input
5. Preview: Badge với icon + color hiển thị realtime

### 2.5 Validations

| Field | Rule |
|-------|------|
| `name` | Required, 2–100 chars, unique |
| `slug` | Required, unique, auto-generate |
| `icon` | Required, must be valid Lucide icon name |
| `color` | Required, valid hex color (#XXXXXX) |

---

## 3. POI Management (Core)

### 3.1 Tổng Quan

POI (Point of Interest) là entity chính — mỗi POI là một điểm du lịch trên bản đồ 360°.

**POI bao gồm:**
- Thông tin cơ bản (tên, mô tả, giờ mở cửa, khoảng cách...)
- Vị trí trên bản đồ (lat/lng — chọn qua Map Picker)
- Panorama 360° riêng (upload + camera config)
- Vị trí hotspot trên overview (pitch/yaw — chọn qua Hotspot Editor)
- Thumbnail image
- Tags (multi-select)
- Gallery ảnh
- Multi-scene (mở rộng)

### 3.2 POI Create/Edit Flow

```
[POI Form — Tabbed Interface]
    │
    ├── Tab 1: THÔNG TIN CƠ BẢN
    │   ├── Tên: [input text] → auto-generate slug
    │   ├── Slug: [input text, editable]
    │   ├── Tỉnh: [Select — province dropdown]
    │   ├── Danh mục: [Select — category dropdown, with color dot]
    │   ├── Emoji: [Emoji Picker]
    │   ├── Địa chỉ: [input text]
    │   ├── Mô tả: [Textarea — rich text optional]
    │   ├── Giờ mở cửa: [input text — "Cả ngày", "8:00 - 17:00"]
    │   ├── Khoảng cách: [input text — "0 km từ trung tâm"]
    │   ├── Tags: [Multi-select — existing tags + create new]
    │   └── Published: [Toggle switch]
    │
    ├── Tab 2: VỊ TRÍ BẢN ĐỒ (Map Picker)
    │   ├── Leaflet Map — Click to place marker
    │   ├── Marker draggable
    │   ├── Lat: [number input — synced with map]
    │   ├── Lng: [number input — synced with map]
    │   └── Search: [Geocoding search — Nominatim]
    │
    ├── Tab 3: PANORAMA 360°
    │   ├── Upload equirectangular image
    │   ├── Pannellum Preview — interactive 360°
    │   ├── Scene Yaw: [number — or capture from viewer]
    │   ├── Scene Pitch: [number]
    │   ├── Scene Hfov: [number]
    │   ├── [📸 Capture Current View]
    │   └── Thumbnail: [Upload — or auto-generate from panorama]
    │
    ├── Tab 4: VỊ TRÍ HOTSPOT
    │   ├── Pannellum Viewer — Province overview panorama
    │   ├── Click trên overview = đặt hotspot position
    │   ├── Hotspot Pitch: [number — auto-fill from click]
    │   ├── Hotspot Yaw: [number — auto-fill from click]
    │   └── Preview: hiển thị marker tại vị trí đã chọn
    │
    └── Tab 5: GALLERY ẢNH
        ├── Drag & drop upload zone
        ├── Grid hiển thị ảnh đã upload
        ├── Drag to reorder
        ├── Mỗi ảnh: [Edit caption] [Delete]
        └── Max 20 ảnh per POI
```

### 3.3 Map Picker — Chi Tiết

```
[User clicks "Vị trí" tab]
    │
    ▼
[Leaflet Map loads]
    ├── Center: Province map center
    ├── Zoom: Province default zoom
    ├── Tiles: CartoDB Voyager (same as frontend)
    ├── Existing POI markers (grey, small) for context
    ├── Current POI marker (large, red, draggable)
    │
    ├── User clicks on map
    │   ├── Marker moves to click position
    │   ├── Lat/Lng inputs update immediately
    │   └── Reverse geocoding → auto-fill address (optional)
    │
    ├── User drags marker
    │   └── Same: Lat/Lng update on dragend
    │
    ├── User types Lat/Lng manually
    │   └── Map pans + marker moves to new position
    │
    └── User searches address
        ├── Nominatim API → geocode
        ├── Map pans to result
        └── Marker placed at result location
```

### 3.4 Hotspot Editor — Chi Tiết

Đặt vị trí hotspot (pin) của POI trên panorama overview:

```
[User clicks "Hotspot" tab]
    │
    ▼
[Load Province overview panorama in Pannellum]
    ├── Show all existing POI hotspots (semi-transparent markers)
    ├── Show current POI hotspot (highlighted, pulsing)
    ├── Cursor: crosshair when hovering panorama
    │
    ├── User clicks on panorama
    │   ├── Get click position → {pitch, yaw}
    │   ├── Move marker to new position
    │   ├── Update Hotspot Pitch/Yaw inputs
    │   └── Realtime preview: marker appears at position
    │
    ├── User adjusts pitch/yaw inputs manually
    │   └── Marker moves in panorama
    │
    └── [📍 Center View on Hotspot]
        └── Pan panorama to show hotspot at center
```

### 3.5 Tag Multi-Select

```
[Tags field on POI form]
    │
    ├── Input: [Biển ✕] [Miễn phí ✕] [_________]
    │          (existing selected tags as pills)
    │
    ├── User types in input
    │   └── Dropdown shows matching tags
    │       ├── [Biển] → select existing
    │       ├── [Biển xanh] → select existing
    │       └── [Tạo "Biển cả" →] → create new tag inline
    │
    ├── User clicks tag pill "✕"
    │   └── Remove tag from POI
    │
    └── API: POST/PATCH POI with tagIds array
```

### 3.6 POI List — Bulk Actions

| Action | Mô tả | Confirm |
|--------|--------|---------|
| Publish selected | Set `is_published = true` for selected | No |
| Unpublish selected | Set `is_published = false` for selected | No |
| Delete selected | Delete all selected POIs | Yes — "Xoá X điểm đến?" |
| Change category | Move selected to different category | No |
| Change province | Move selected to different province | No |

### 3.7 Validations

| Field | Rule |
|-------|------|
| `name` | Required, 2–200 chars |
| `slug` | Required, unique, auto-generate |
| `provinceId` | Required, valid province ID |
| `categoryId` | Required, valid category ID |
| `lat` | Required, -90 to 90 |
| `lng` | Required, -180 to 180 |
| `emoji` | Optional, single emoji char |
| `panoramaUrl` | Optional, valid image URL |
| `sceneYaw` | Number, -180 to 180 |
| `scenePitch` | Number, -90 to 90 |
| `sceneHfov` | Number, 30 to 150 |
| `hotspotPitch` | Number, -90 to 90 |
| `hotspotYaw` | Number, -180 to 180 |
| `thumbUrl` | Optional, valid image URL |
| `description` | Optional, max 5000 chars |
| `hours` | Optional, max 100 chars |
| `distance` | Optional, max 100 chars |
| `tagIds` | Optional, array of valid tag IDs |

---

## 4. Tag Management

### 4.1 Tổng Quan

Tags là labels tự do gán cho POI — hiển thị dưới dạng pills trên frontend.

### 4.2 Features

| Feature | Mô tả |
|---------|--------|
| Card grid display | Tags hiển thị dạng grid cards |
| Search | Real-time filter tags |
| POI count | Hiển thị số POIs sử dụng tag |
| Inline edit | Click tag → edit name inline |
| Create from POI form | Tạo tag mới trực tiếp từ POI edit form |
| Merge tags | Gộp 2 tags thành 1 (admin only) |
| Delete | Xoá tag (remove from all POIs) |

### 4.3 Tag Merge Workflow

```
[Admin selects 2 tags: "Biển" and "Bãi biển"]
    │
    ▼
[Merge Dialog]
    ├── "Gộp 2 tag thành 1?"
    ├── Giữ lại: [Biển ▼]
    ├── Xoá: "Bãi biển" (3 POIs sẽ chuyển sang "Biển")
    └── [Gộp] → API: merge tag IDs
```

---

## 5. Map Configuration

### 5.1 Tổng Quan

Cấu hình bản đồ Leaflet cho mỗi tỉnh — center point, zoom, tile provider, CSS filter.

### 5.2 Configurable Settings

| Setting | Type | Default | Mô tả |
|---------|------|---------|--------|
| `center_lat` | Decimal | 10.40 | Latitude trung tâm bản đồ |
| `center_lng` | Decimal | 107.15 | Longitude trung tâm bản đồ |
| `default_zoom` | Int | 9 | Zoom mặc định khi mở bản đồ |
| `min_zoom` | Int | 5 | Zoom nhỏ nhất (tối thiểu) |
| `max_zoom` | Int | 18 | Zoom lớn nhất (tối đa) |
| `tile_provider` | String | CartoDB Voyager | URL template cho map tiles |
| `tile_filter_css` | String | `brightness(.78)...` | CSS filter cho dark mode |

### 5.3 Tile Provider Options

| Provider | URL | Phong cách |
|----------|-----|-----------|
| CartoDB Voyager | `cartocdn.com/.../voyager/...` | Clean, labeled |
| CartoDB Dark Matter | `cartocdn.com/.../dark_all/...` | Dark themed |
| CartoDB Positron | `cartocdn.com/.../light_all/...` | Light, minimal |
| OpenStreetMap | `tile.openstreetmap.org/...` | Standard |
| Stamen Watercolor | `stamen-tiles.../watercolor/...` | Artistic |

### 5.4 Live Preview

Admin thay đổi settings → map preview cập nhật realtime:
- Thay đổi center → map pans
- Thay đổi zoom → map zooms
- Thay đổi tile provider → tiles reload
- Thay đổi CSS filter → filter applies

---

## 6. Gallery Management

### 6.1 Tổng Quan

Mỗi POI có gallery ảnh — admin upload, caption, reorder.

### 6.2 Upload Flow

```
[POI Edit → Gallery Tab]
    │
    ├── Drag & Drop Zone
    │   ├── "Kéo thả ảnh vào đây hoặc click để chọn"
    │   ├── Accept: JPEG, PNG, WebP
    │   ├── Max size: 10MB per file
    │   ├── Max files: 20 per POI
    │   └── Multiple upload supported
    │
    ├── Upload Progress
    │   ├── Each file: progress bar + filename
    │   ├── Sharp processing: resize to max 1920px width
    │   ├── Generate thumbnail (300×200)
    │   └── Upload to S3/R2
    │
    └── Gallery Grid (after upload)
        ├── 4-column grid of images
        ├── Drag to reorder
        ├── Each image:
        │   ├── Thumbnail preview
        │   ├── Caption input (inline edit)
        │   ├── [🗑 Delete] button
        │   └── Sort handle (drag)
        └── PATCH /api/admin/pois/:id/gallery/reorder
```

### 6.3 Image Processing Pipeline

```
[User drops image file]
    │
    ▼
[Client-side validation]
    ├── Check file type (JPEG/PNG/WebP)
    ├── Check file size (≤ 10MB)
    └── Generate local preview (URL.createObjectURL)
    │
    ▼
[Upload to API]
    │  POST /api/admin/upload/gallery
    │  Content-Type: multipart/form-data
    ▼
[Server-side processing (Sharp)]
    ├── Resize to max 1920×1080
    ├── Generate thumbnail 300×200 (crop center)
    ├── Convert to JPEG 85%
    └── Strip EXIF data
    │
    ▼
[Upload to S3/R2]
    ├── Original: /gallery/{poiId}/{uuid}.jpg
    └── Thumbnail: /gallery/{poiId}/{uuid}_thumb.jpg
    │
    ▼
[Return URLs to client]
    └── { imageUrl, thumbUrl }
```

---

## 7. Scene Management (Multi-Scene)

### 7.1 Tổng Quan

Mở rộng cho tương lai — mỗi POI có thể có nhiều scene 360° (thay vì 1 panorama duy nhất).

### 7.2 Scene CRUD (trong POI Edit)

```
[POI Edit → Scenes Tab]
    │
    ├── Default Scene (from POI panorama_url)
    │   ├── [🖼 Panorama preview]
    │   ├── Label: "Mặc định"
    │   └── Yaw/Pitch/Hfov config
    │
    ├── Additional Scenes
    │   ├── [+ Thêm Scene]
    │   │   ├── Upload panorama
    │   │   ├── Label: [input — "View biển", "Sảnh chính"]
    │   │   └── Camera config (yaw/pitch/hfov)
    │   │
    │   ├── Scene 2: "View biển"
    │   │   ├── [🖼 Pannellum preview]
    │   │   └── [Edit] [Set Default] [Delete]
    │   │
    │   └── Scene 3: "Ban công"
    │       ├── [🖼 Pannellum preview]
    │       └── [Edit] [Set Default] [Delete]
    │
    └── [Drag to reorder scenes]
```

### 7.3 Frontend Integration

Khi POI có multi-scene, frontend sẽ hiển thị:
- Scene selector buttons bên dưới (giống hotel room multi-scene)
- Navigation hotspots giữa các scenes
- Default scene load đầu tiên

---

## 8. Panorama & Hotspot Editor

### 8.1 Tổng Hợp Editor Components

| Editor | Dùng ở | Mục đích |
|--------|--------|---------|
| PanoramaPreview | Province Edit (Tab 2) | Preview + config overview panorama |
| PanoramaPreview | POI Edit (Tab 3) | Preview + config POI panorama |
| HotspotEditor | POI Edit (Tab 4) | Đặt vị trí POI hotspot trên overview |
| MapPicker | POI Edit (Tab 2) | Chọn tọa độ lat/lng |

### 8.2 Panorama Upload Workflow

```
[User clicks "Upload Panorama"]
    │
    ├── File picker opens (accept: image/*)
    ├── Or user drags file onto upload zone
    │
    ▼
[Client validation]
    ├── File type: JPEG, PNG (equirectangular)
    ├── File size: ≤ 30MB
    ├── Recommended ratio: 2:1 (equirectangular)
    └── Show preview thumbnail
    │
    ▼
[Upload to server]
    ├── POST /api/admin/upload/panorama
    ├── Progress bar during upload
    ├── Server: Sharp resize to max 8192px wide
    ├── Server: Generate thumb 400×200
    ├── Server: Upload to S3/R2
    └── Return: { panoramaUrl, thumbUrl }
    │
    ▼
[Pannellum loads new panorama]
    ├── Interactive 360° preview
    ├── Admin rotates to desired initial view
    └── Click "📸 Capture" → save yaw/pitch/hfov
```

### 8.3 Capture Camera Settings

Khi admin click "📸 Capture Current View":

```javascript
// Get current viewer state
const yaw = viewer.getYaw();
const pitch = viewer.getPitch();
const hfov = viewer.getHfov();

// Auto-fill form fields
setFieldValue("sceneYaw", Math.round(yaw * 100) / 100);
setFieldValue("scenePitch", Math.round(pitch * 100) / 100);
setFieldValue("sceneHfov", Math.round(hfov * 100) / 100);
```

---

## 9. Publish & Export System

### 9.1 Tổng Quan

Dashboard export dữ liệu từ DB → JSON format tương thích với frontend `tourism-map-360.html`.

### 9.2 Publish Flow

```
[Admin clicks "📤 Publish" on Province]
    │
    ▼
[System checks]
    ├── Province has overview panorama? ✅
    ├── Province has at least 1 published POI? ✅
    ├── All published POIs have panorama? ⚠️ (warning nếu thiếu)
    ├── All POIs have hotspot position? ⚠️
    └── All POIs have lat/lng? ✅
    │
    ▼
[Generate Export JSON]
    ├── Build CATEGORIES array (active categories only)
    ├── Build OVERVIEW object (from province overview config)
    ├── Build POIS array (published POIs only, matching demo schema)
    ├── Build MAP_CONFIG object
    └── Format to match frontend data structure
    │
    ▼
[Output Options]
    ├── Option A: Download JSON file
    │   └── Admin manually injects into HTML template
    │
    ├── Option B: Auto-generate HTML file
    │   ├── Use HTML template
    │   ├── Inject JSON data into <script>
    │   └── Download ready-to-deploy HTML
    │
    └── Option C: API endpoint (if API-backed public site)
        └── Data served from /api/public/provinces/:slug/pois
```

### 9.3 Export JSON Schema

```json
{
  "exportedAt": "2025-01-15T10:30:00Z",
  "province": {
    "name": "Bà Rịa — Vũng Tàu",
    "slug": "ba-ria-vung-tau"
  },
  "CATEGORIES": [
    { "id": "beach", "label": "Biển", "icon": "waves", "color": "#22D3A7" }
  ],
  "OVERVIEW": {
    "panorama": "https://cdn.example.com/panoramas/brvt-overview.jpg",
    "yaw": 80, "pitch": 10, "hfov": 120, "autoRotate": 0.2
  },
  "POIS": [
    {
      "id": 1,
      "cat": "beach",
      "name": "Bãi Trước",
      "addr": "Tp. Vũng Tàu",
      "lat": 10.346, "lng": 107.072,
      "pitch": -10, "yaw": 40,
      "panorama": "https://cdn.example.com/panoramas/bai-truoc.jpg",
      "pYaw": 30, "pPitch": 0, "pHfov": 105,
      "thumb": "https://cdn.example.com/thumbs/bai-truoc.jpg",
      "desc": "Bãi Trước nằm ngay trung tâm...",
      "tags": ["Biển", "Miễn phí"],
      "hours": "Cả ngày",
      "distance": "0 km từ trung tâm",
      "emoji": "🏖️"
    }
  ],
  "MAP_CONFIG": {
    "center": [10.40, 107.15],
    "zoom": 9,
    "minZoom": 5,
    "maxZoom": 18,
    "tileProvider": "https://{s}.basemaps.cartocdn.com/...",
    "tileFilterCss": "brightness(.78) contrast(1.15) saturate(.85)"
  }
}
```

### 9.4 Data Transformation Rules

| DB Field | Export Field | Note |
|----------|-------------|------|
| `category.slug` | `poi.cat` | Map category_id → slug |
| `category.name` | `category.label` | Rename field |
| `poi.scene_yaw` | `poi.pYaw` | Rename to match frontend |
| `poi.scene_pitch` | `poi.pPitch` | |
| `poi.scene_hfov` | `poi.pHfov` | |
| `poi.hotspot_pitch` | `poi.pitch` | |
| `poi.hotspot_yaw` | `poi.yaw` | |
| `poi.panorama_url` | `poi.panorama` | |
| `poi.thumb_url` | `poi.thumb` | |
| `poi.description` | `poi.desc` | |
| `poi_tags → tag.name[]` | `poi.tags` | Join from pivot table |

---

## 10. User Management

### 10.1 Roles

| Role | Mô tả |
|------|--------|
| `admin` | Full access, quản lý users & settings, publish |
| `editor` | Quản lý nội dung: CRUD provinces, categories, POIs, tags |
| `viewer` | Chỉ xem, không chỉnh sửa |

### 10.2 User CRUD (Admin Only)

```
[User List Page]
    │
    ├── [+ Thêm User]
    │   ├── Email: [input — unique]
    │   ├── Tên: [input]
    │   ├── Mật khẩu: [input — min 8 chars]
    │   ├── Role: [Select — admin/editor/viewer]
    │   └── [Tạo] → POST /api/admin/users
    │
    ├── [✏️ Edit]
    │   ├── Tên, Role, Active toggle
    │   ├── Reset password option
    │   └── [Lưu] → PATCH /api/admin/users/:id
    │
    └── [🗑 Xoá]
        ├── Không thể tự xoá chính mình
        └── DELETE (confirm)
```

### 10.3 Profile (Self-Edit)

Mọi user có thể chỉnh sửa profile của mình:
- Đổi tên
- Đổi avatar
- Đổi mật khẩu
- Không thể tự đổi role

---

## 11. Analytics Dashboard

### 11.1 Overview Stats

| Metric | Query | Visualize |
|--------|-------|-----------|
| Total POIs | `COUNT(*) FROM pois` | Number card |
| Published POIs | `COUNT(*) WHERE is_published` | Number card |
| Total provinces | `COUNT(*) FROM provinces` | Number card |
| Total views (30d) | `COUNT(*) FROM analytics WHERE type='poi_view' AND date > -30d` | Number card + trend |
| Top 5 POIs | `GROUP BY poi_id ORDER BY count DESC LIMIT 5` | Bar chart / list |
| Category breakdown | `GROUP BY category_id` | Donut chart |
| Device types | `GROUP BY device_type` | Donut chart |
| Views timeline | `GROUP BY DATE(created_at)` | Line chart (30 days) |

### 11.2 Charts Library

Sử dụng **Recharts** hoặc **Chart.js** (React):

| Chart | Data |
|-------|------|
| Line chart | Views over 30 days |
| Bar chart | Top POIs by views |
| Donut chart | Device types (mobile/desktop/tablet) |
| Donut chart | Views by category |
| Heatmap | Views by day of week × hour (optional) |

### 11.3 Filters

| Filter | Options |
|--------|---------|
| Date range | Last 7d, 30d, 90d, custom |
| Province | All / specific province |
| Category | All / specific category |

---

## 12. Settings

### 12.1 General Settings

```
[Settings Page]
    │
    ├── Branding
    │   ├── Site name: [Du Lịch Bà Rịa]
    │   ├── Subtitle: [BẢN ĐỒ THỰC TẾ ẢO 360°]
    │   └── Logo: [Upload]
    │
    ├── Frontend Theme
    │   ├── Accent color: [#22D3A7] (with preview)
    │   ├── Font family: [Inter ▼]
    │   └── Dark mode: [Always dark ▼]
    │
    ├── Smart UI
    │   ├── Auto-hide delay: [4] seconds
    │   ├── Auto-rotate speed: [0.2] °/s
    │   └── Enable minimap: [✅ toggle]
    │
    ├── Publishing
    │   ├── Default export format: [JSON ▼]
    │   ├── Auto-generate slug: [✅ toggle]
    │   └── CDN base URL: [https://cdn.example.com]
    │
    └── [💾 Lưu Cài Đặt]
```

### 12.2 Sequence: Settings → Frontend

```
[Admin changes accent color to #FF6B6B]
    │
    ▼
[Save to settings table]
    │
    ▼
[Next export/publish includes new color]
    │
    ▼
[Frontend CSS variable --accent: #FF6B6B]
```

---

*End of Document*
