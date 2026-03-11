# 01 — DASHBOARD ARCHITECTURE SPECIFICATION
## Tourism Map 360° — Admin Dashboard

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Project:** Du Lịch Bà Rịa — Bản Đồ 360° Tourism Platform — Admin Dashboard  
**Related:** `markdown-tourism/01~05` (Frontend specs), `markdown-dashboard-hotel/01~05` (Hotel dashboard reference)

---

## Mục Lục

1. [Tổng Quan Dashboard](#1-tổng-quan-dashboard)
2. [Kiến Trúc Hệ Thống](#2-kiến-trúc-hệ-thống)
3. [Module Layout](#3-module-layout)
4. [Tech Stack](#4-tech-stack)
5. [So Sánh Với Hotel Dashboard](#5-so-sánh-với-hotel-dashboard)
6. [Vai Trò & Phân Quyền (RBAC)](#6-vai-trò--phân-quyền-rbac)
7. [Deployment Architecture](#7-deployment-architecture)

---

## 1. Tổng Quan Dashboard

### 1.1 Mục Đích

Admin Dashboard cho Tourism Map 360° phục vụ quản lý toàn bộ nội dung của bản đồ du lịch 360°:

- **Quản lý Tỉnh/Thành (Provinces):** Thêm/sửa/xoá tỉnh, cấu hình panorama overview cho mỗi tỉnh
- **Quản lý Danh Mục (Categories):** Tuỳ chỉnh danh mục POI, icon Lucide, màu sắc
- **Quản lý Điểm Đến (POIs):** CRUD điểm du lịch, upload panorama, đặt vị trí hotspot, cấu hình bản đồ
- **Quản lý Tags:** Tag tự do cho POIs
- **Quản lý Scenes:** Multi-scene per POI (nếu mở rộng)
- **Quản lý Gallery:** Ảnh gallery cho mỗi POI
- **Cấu hình Bản Đồ:** Map center, zoom, tile provider
- **Thống kê:** Lượt xem POI, category phổ biến, thiết bị truy cập

### 1.2 Triết Lý Thiết Kế

> "Admin quản lý bản đồ du lịch 360° — mỗi thao tác trên dashboard đều tạo ra trải nghiệm khám phá cho du khách."

- **Map-Centric:** Quản lý POI qua bản đồ tương tác (pick location bằng click trên map)
- **Visual Editing:** Xem trước panorama + đặt hotspot trực tiếp
- **Multi-Province Ready:** Kiến trúc hỗ trợ nhiều tỉnh/thành (scalable)
- **Đơn giản hơn Hotel Dashboard:** Ít entity hơn, không có booking/payment/i18n phức tạp

### 1.3 So Sánh Phạm Vi Với Hotel Dashboard

| Tính năng | Hotel Dashboard | Tourism Dashboard |
|-----------|:---:|:---:|
| Room/POI CRUD | ✅ (Rooms) | ✅ (POIs) |
| Category management | ✅ | ✅ |
| Scene/Panorama editor | ✅ | ✅ |
| Hotspot placement | ✅ | ✅ |
| Gallery management | ✅ | ✅ |
| Tag management | ❌ (built-in amenities) | ✅ |
| Province management | ❌ | ✅ |
| Map configuration | ❌ | ✅ |
| Booking/Payment | ✅ | ❌ |
| Multi-language (i18n) | ✅ | ❌ |
| Service CRUD | ✅ | ❌ |
| Post/Page CMS | ✅ | ❌ |
| Menu builder | ✅ | ❌ |
| Promotion management | ✅ | ❌ |
| SEO management | ✅ | ❌ |
| User RBAC | ✅ (4 roles) | ✅ (3 roles) |
| Analytics | ✅ | ✅ |

---

## 2. Kiến Trúc Hệ Thống

### 2.1 Mô Hình 3 Lớp (Full-Stack)

```
┌─────────────────────────────────────────────────────────────────┐
│                    ADMIN DASHBOARD (Browser)                     │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    Next.js App Router                       │  │
│  │  /admin/dashboard       ← Tổng quan thống kê              │  │
│  │  /admin/provinces       ← Quản lý tỉnh/thành              │  │
│  │  /admin/categories      ← Quản lý danh mục                │  │
│  │  /admin/pois            ← Quản lý điểm đến (POIs)         │  │
│  │  /admin/tags            ← Quản lý tags                     │  │
│  │  /admin/map-config      ← Cấu hình bản đồ                 │  │
│  │  /admin/users           ← Quản lý người dùng              │  │
│  │  /admin/settings        ← Cài đặt chung                   │  │
│  └────────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                         REST API                                 │
│                              │                                   │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    API Layer (Next.js)                      │  │
│  │  /api/admin/provinces    ← Province CRUD                   │  │
│  │  /api/admin/categories   ← Category CRUD                   │  │
│  │  /api/admin/pois         ← POI CRUD                        │  │
│  │  /api/admin/tags         ← Tag CRUD                        │  │
│  │  /api/admin/upload       ← File upload (panorama, thumb)   │  │
│  │  /api/admin/analytics    ← Dashboard stats                 │  │
│  │  /api/admin/users        ← User management                 │  │
│  │  /api/admin/map-config   ← Map settings                    │  │
│  └────────────────────────────────────────────────────────────┘  │
│                              │                                   │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    Database Layer                           │  │
│  │  PostgreSQL 15+  ·  Prisma ORM                             │  │
│  │  Tables: provinces, categories, pois, tags, poi_tags,      │  │
│  │          poi_scenes, poi_gallery, users, map_configs,      │  │
│  │          analytics_events                                   │  │
│  └────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Luồng Dữ Liệu Chính

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
│  Admin User  │────►│  Dashboard   │────►│  PostgreSQL DB   │
│  (Browser)   │◄────│  (Next.js)   │◄────│  (Prisma ORM)    │
└──────────────┘     └──────┬───────┘     └──────────────────┘
                            │
                     ┌──────┴───────┐
                     │  CDN / S3    │
                     │  (Panoramas, │
                     │   Thumbnails)│
                     └──────────────┘
                            │
                     ┌──────┴───────┐
                     │  Public Site │
                     │  (tourism-   │
                     │  map-360)    │
                     └──────────────┘
```

### 2.3 Data Flow: Dashboard → Public Site

Dashboard quản lý dữ liệu trong PostgreSQL. Public site (tourism-map-360) có 2 cách tiêu thụ dữ liệu:

**Option A — Static Export (Đơn giản):**
```
Dashboard → DB → Export JSON → Public HTML inject <script>
```
Admin bấm "Publish" → hệ thống generate file JSON/JS chứa CATEGORIES, OVERVIEW, POIS → inject vào HTML template → deploy static.

**Option B — API-Backed (Nâng cao):**
```
Dashboard → DB → Public API → Public SPA fetch
```
Public site gọi REST API để lấy data thay vì hardcode.

---

## 3. Module Layout

### 3.1 Sidebar Navigation

```
┌──────────────────────────────────────────────────────────────┐
│ ┌──────────┐ ┌────────────────────────────────────────────┐  │
│ │          │ │                                            │  │
│ │ SIDEBAR  │ │              MAIN CONTENT                  │  │
│ │          │ │                                            │  │
│ │ [🏠] Dashboard                                         │  │
│ │ ─────────│ │                                            │  │
│ │ NỘI DUNG │ │                                            │  │
│ │ [🏛] Tỉnh/Thành                                       │  │
│ │ [📂] Danh mục                                          │  │
│ │ [📍] Điểm đến                                          │  │
│ │ [🏷] Tags  │ │                                          │  │
│ │ ─────────│ │                                            │  │
│ │ TRỰC QUAN│ │                                            │  │
│ │ [🗺] Bản đồ                                            │  │
│ │ [🖼] Gallery                                            │  │
│ │ ─────────│ │                                            │  │
│ │ HỆ THỐNG │ │                                            │  │
│ │ [👥] Users │                                            │  │
│ │ [⚙] Settings                                           │  │
│ │ [📊] Analytics                                         │  │
│ │          │ │                                            │  │
│ └──────────┘ └────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### 3.2 Module Chi Tiết

| Module | Route | Mô tả | Entity chính |
|--------|-------|--------|-------------|
| Dashboard | `/admin` | Thống kê tổng quan, quick actions | — |
| Provinces | `/admin/provinces` | CRUD tỉnh/thành, overview panorama config | `provinces` |
| Categories | `/admin/categories` | CRUD danh mục, Lucide icon picker, color picker | `categories` |
| POIs | `/admin/pois` | CRUD điểm đến, map picker, panorama upload, hotspot editor | `pois` |
| Tags | `/admin/tags` | CRUD tags tự do | `tags` |
| Map Config | `/admin/map-config` | Center point, zoom, tile layer, map style | `map_configs` |
| Gallery | `/admin/gallery` | Quản lý ảnh gallery per POI | `poi_gallery` |
| Users | `/admin/users` | Quản lý tài khoản admin | `users` |
| Settings | `/admin/settings` | Cài đặt chung, branding, publish | — |
| Analytics | `/admin/analytics` | Thống kê lượt xem, top POIs, devices | `analytics_events` |

---

## 4. Tech Stack

### 4.1 Frontend

| Technology | Version | Vai trò |
|-----------|---------|---------|
| Next.js | 14+ | App Router, SSR, API Routes |
| React | 18+ | UI Components |
| TypeScript | 5+ | Type safety |
| Tailwind CSS | 3+ | Styling |
| shadcn/ui | latest | Component library (Button, Input, Dialog, Table, etc.) |
| Leaflet | 1.9+ | Map picker (chọn tọa độ POI) |
| react-leaflet | 4+ | React wrapper cho Leaflet |
| Pannellum | 2.5+ | Preview panorama trong editor |
| @tanstack/react-query | 5+ | Data fetching & caching |
| Zustand | 4+ | Client state management |
| react-hook-form | 7+ | Form handling |
| Zod | 3+ | Schema validation |
| Lucide React | latest | Icons (consistent với frontend) |
| react-beautiful-dnd | latest | Drag & drop sorting |
| react-colorful | latest | Color picker cho categories |

### 4.2 Backend

| Technology | Version | Vai trò |
|-----------|---------|---------|
| Next.js API Routes | 14+ | REST API endpoints |
| Prisma | 5+ | ORM, schema management, migrations |
| PostgreSQL | 15+ | Primary database |
| NextAuth.js | 5+ | Authentication |
| Sharp | latest | Image processing (panorama thumbnails) |
| AWS S3 / Cloudflare R2 | — | File storage (panoramas, thumbnails) |

### 4.3 DevOps

| Technology | Vai trò |
|-----------|---------|
| Docker | Containerization |
| Vercel / VPS | Hosting |
| GitHub Actions | CI/CD |
| Prisma Migrate | Database migrations |

---

## 5. So Sánh Với Hotel Dashboard

### 5.1 Kiến Trúc

| Aspect | Hotel Dashboard | Tourism Dashboard |
|--------|----------------|------------------|
| Framework | Next.js 14+ App Router | Next.js 14+ App Router |
| Auth | NextAuth.js (JWT) | NextAuth.js (JWT) |
| DB | PostgreSQL + Prisma | PostgreSQL + Prisma |
| State | Zustand | Zustand |
| Forms | react-hook-form + Zod | react-hook-form + Zod |
| UI Kit | shadcn/ui | shadcn/ui |
| Unique libs | — | Leaflet, react-leaflet (map picker) |
| i18n | ✅ Full i18n (translations table) | ❌ Vietnamese only |
| Booking | ✅ Dual mode (link/modal) | ❌ No booking |
| CMS Pages | ✅ Post/Page editor | ❌ No CMS |

### 5.2 Độ Phức Tạp

| Metric | Hotel | Tourism |
|--------|-------|---------|
| DB tables | ~15 | ~10 |
| RBAC roles | 4 (super_admin, admin, editor, viewer) | 3 (admin, editor, viewer) |
| Admin routes | ~12 | ~8 |
| API endpoints | ~30 | ~18 |
| Form complexity | High (booking config, i18n) | Medium (map picker, panorama) |

---

## 6. Vai Trò & Phân Quyền (RBAC)

### 6.1 Roles

| Role | Mô tả |
|------|--------|
| `admin` | Full access — quản lý mọi thứ, bao gồm users & settings |
| `editor` | Quản lý nội dung — CRUD provinces, categories, POIs, tags, gallery |
| `viewer` | Chỉ xem — không thêm/sửa/xoá |

### 6.2 Permission Matrix

| Action | admin | editor | viewer |
|--------|:-----:|:------:|:------:|
| View dashboard stats | ✅ | ✅ | ✅ |
| CRUD Provinces | ✅ | ✅ | ❌ |
| CRUD Categories | ✅ | ✅ | ❌ |
| CRUD POIs | ✅ | ✅ | ❌ |
| CRUD Tags | ✅ | ✅ | ❌ |
| Upload panoramas | ✅ | ✅ | ❌ |
| Manage Gallery | ✅ | ✅ | ❌ |
| Map config | ✅ | ❌ | ❌ |
| Manage users | ✅ | ❌ | ❌ |
| System settings | ✅ | ❌ | ❌ |
| Publish/Export | ✅ | ❌ | ❌ |
| View analytics | ✅ | ✅ | ✅ |

---

## 7. Deployment Architecture

### 7.1 Production

```
┌──────────────────────────────────────────────────────────────┐
│                        PRODUCTION                             │
│                                                               │
│  ┌────────────────────┐    ┌────────────────────────────┐    │
│  │   Vercel / VPS     │    │    PostgreSQL 15+           │    │
│  │   (Next.js App)    │───►│    (Managed / Docker)       │    │
│  │                    │    └────────────────────────────┘    │
│  │  /admin/* → Dashboard                                     │
│  │  /api/admin/* → API │    ┌────────────────────────────┐   │
│  │  /api/public/* → Public│  │    S3 / R2 Storage        │   │
│  │                    │───►│    (Panoramas, Thumbnails)  │   │
│  └────────────────────┘    └────────────────────────────┘    │
│           │                                                   │
│           │ Export JSON / Serve API                            │
│           ▼                                                   │
│  ┌────────────────────────────────────────────────────────┐   │
│  │              PUBLIC TOURISM SITE                        │   │
│  │  tourism-map-360.html (static) hoặc SPA (API-backed)   │   │
│  │  CDN: Cloudflare / Vercel Edge                         │   │
│  └────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

### 7.2 Development

```
Local Machine
├── next dev (port 3000)         ← Dashboard + API
├── PostgreSQL (Docker, port 5432) ← Database
├── tourism-map-360.html          ← Open in browser for testing
└── .env.local                    ← DB URL, S3 keys, NextAuth secret
```

---

*End of Document*
