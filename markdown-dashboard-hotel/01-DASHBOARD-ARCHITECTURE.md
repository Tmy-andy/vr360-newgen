# 01 — DASHBOARD ARCHITECTURE SPECIFICATION
## Admin Dashboard — Hotel CMS Platform

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** markdown-hotel/01~05 · markdown-dashboard-hotel/02~05

---

## Mục Lục

1. [Tổng Quan Dashboard](#1-tổng-quan-dashboard)
2. [Kiến Trúc Hệ Thống](#2-kiến-trúc-hệ-thống)
3. [Authentication & Authorization](#3-authentication--authorization)
4. [Tech Stack](#4-tech-stack)
5. [Module Map](#5-module-map)
6. [Routing Structure](#6-routing-structure)
7. [Deployment Architecture](#7-deployment-architecture)
8. [Sequence Diagrams](#8-sequence-diagrams)

---

## 1. Tổng Quan Dashboard

### 1.1 Triết Lý

> "Admin Dashboard là trung tâm quản lý toàn bộ nội dung khách sạn — từ phòng, dịch vụ, ảnh 360, hotspot, đến cấu hình booking. Mọi thứ khách nhìn thấy trên website đều được quản lý từ đây."

### 1.2 Đặc Điểm

| Tiêu chí | Giá trị |
|----------|---------|
| **Mô hình** | Standalone — mỗi khách sạn 1 instance (không SaaS) |
| **Truy cập** | `https://{domain}/admin` |
| **Responsive** | Desktop-first, hỗ trợ tablet (≥768px) |
| **Ngôn ngữ giao diện** | Tiếng Anh (mặc định), có thể thêm i18n |
| **Phân quyền** | 4 role: Super Admin, Admin, Editor, Viewer |
| **Theme** | Light mode mặc định, hỗ trợ dark mode |

### 1.3 So Sánh Admin Dashboard vs Public Website

| Tiêu chí | Public Website | Admin Dashboard |
|----------|---------------|----------------|
| Layout | 360-first immersive | Sidebar + content area |
| Theme | Dark luxury (gold) | Light professional |
| Target user | Du khách, khách hàng | Nhân viên khách sạn |
| Auth | Không cần | JWT + RBAC |
| Pannellum | Full viewer | Editor mode (click-to-place hotspot) |
| Data | Read-only (API GET) | Full CRUD (API GET/POST/PUT/DELETE) |

---

## 2. Kiến Trúc Hệ Thống

### 2.1 Tổng Quan

```
┌─────────────────────────────────────────────────────────────────┐
│                    ADMIN DASHBOARD                               │
│                 (Next.js App Router /admin)                       │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    PRESENTATION LAYER                        │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐   │ │
│  │  │ Dashboard │ │  Forms   │ │  Tables  │ │ 360 Editor   │   │ │
│  │  │ Overview  │ │ (CRUD)   │ │ (Lists)  │ │ (Hotspot     │   │ │
│  │  │ Charts    │ │ Rich Ed. │ │ Filters  │ │  Placement)  │   │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────────┘   │ │
│  └───────────────────────────┬─────────────────────────────────┘ │
│                              │                                    │
│  ┌───────────────────────────┼─────────────────────────────────┐ │
│  │                  APPLICATION LAYER                           │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐   │ │
│  │  │ Auth     │ │ State    │ │ API      │ │ Upload       │   │ │
│  │  │ Provider │ │ Manager  │ │ Client   │ │ Manager      │   │ │
│  │  │ (JWT)    │ │ (Zustand)│ │ (fetch)  │ │ (S3/R2)      │   │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────────┘   │ │
│  └───────────────────────────┬─────────────────────────────────┘ │
│                              │                                    │
│  ┌───────────────────────────┼─────────────────────────────────┐ │
│  │                    API LAYER                                 │ │
│  │  ┌─────────────────────────────────────────────────────────┐ │ │
│  │  │ /api/v1/admin/*                                         │ │ │
│  │  │ Auth middleware → Route handlers → Prisma → PostgreSQL  │ │ │
│  │  └─────────────────────────────────────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│  ┌───────────────────────────┼─────────────────────────────────┐ │
│  │                  INFRASTRUCTURE                              │ │
│  │  ┌───────────┐ ┌──────────┐ ┌───────────┐ ┌─────────────┐  │ │
│  │  │PostgreSQL │ │  Redis   │ │ S3 / R2   │ │ CDN         │  │ │
│  │  │(Database) │ │  (Cache) │ │ (Storage) │ │(Cloudflare) │  │ │
│  │  └───────────┘ └──────────┘ └───────────┘ └─────────────┘  │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Admin Nằm Trong Cùng Next.js App

Dashboard admin được đặt bên trong cùng Next.js project với public website, dưới route group `/admin`:

```
src/app/
├── (marketing)/       ← Public CMS pages (Layout B)
├── (immersive)/       ← Public 360 pages (Layout A)
├── api/v1/            ← REST API (public + admin)
└── admin/             ← ★ Admin Dashboard (Layout riêng)
    ├── layout.tsx     ← AdminLayout (sidebar + topbar)
    ├── page.tsx       ← Dashboard overview
    ├── rooms/         ← Room management
    ├── services/      ← Service management
    ├── scenes/        ← Scene & hotspot editor
    ├── promotions/    ← Promotion management
    ├── posts/         ← Blog/news management
    ├── gallery/       ← Gallery management
    ├── menus/         ← Menu management
    ├── booking/       ← ★ Booking configuration
    ├── translations/  ← i18n management
    ├── settings/      ← Hotel settings
    ├── users/         ← User management
    └── analytics/     ← Analytics dashboard
```

---

## 3. Authentication & Authorization

### 3.1 Auth Flow

```
[Admin truy cập /admin]
    │
    ├── Chưa login → Redirect /admin/login
    │       │
    │       ▼
    │   [Login Page]
    │   ├── Email + Password
    │   ├── POST /api/v1/admin/auth/login
    │   ├── Server validate → generate JWT (access + refresh)
    │   ├── Set httpOnly cookies
    │   └── Redirect /admin (dashboard)
    │
    ├── Đã login → Verify JWT
    │       │
    │       ├── Valid → Render dashboard
    │       ├── Expired → Try refresh token
    │       │       ├── Refresh OK → New access token
    │       │       └── Refresh fail → Redirect /admin/login
    │       └── Invalid → Redirect /admin/login
    │
    └── Mỗi API request:
            Header: Authorization: Bearer {access_token}
            Middleware kiểm tra: valid? role đủ quyền? → proceed/reject
```

### 3.2 Role-Based Access Control (RBAC)

| Role | Quyền |
|------|-------|
| **super_admin** | Toàn quyền: CRUD tất cả, quản lý users, settings hệ thống, xem analytics đầy đủ |
| **admin** | CRUD nội dung (rooms, services, promotions, posts, gallery, menus), xem analytics, quản lý translations, cấu hình booking |
| **editor** | Tạo/sửa nội dung (rooms, services, posts), upload media, KHÔNG xóa, KHÔNG settings |
| **viewer** | Chỉ xem (read-only), xem analytics cơ bản |

### 3.3 Permission Matrix

| Module | super_admin | admin | editor | viewer |
|--------|:-----------:|:-----:|:------:|:------:|
| Dashboard Overview | ✅ | ✅ | ✅ | ✅ |
| Rooms CRUD | ✅ | ✅ | Create/Edit | Read |
| Services CRUD | ✅ | ✅ | Create/Edit | Read |
| Scene/Hotspot Editor | ✅ | ✅ | Edit | Read |
| Promotions | ✅ | ✅ | Create/Edit | Read |
| Posts/News | ✅ | ✅ | Create/Edit | Read |
| Gallery | ✅ | ✅ | Upload | Read |
| Menu Management | ✅ | ✅ | ❌ | ❌ |
| Booking Config | ✅ | ✅ | ❌ | ❌ |
| Translations | ✅ | ✅ | Edit | Read |
| Hotel Settings | ✅ | ✅ | ❌ | ❌ |
| User Management | ✅ | ❌ | ❌ | ❌ |
| Analytics (full) | ✅ | ✅ | ❌ | ❌ |
| Analytics (basic) | ✅ | ✅ | ✅ | ✅ |
| Upload Media | ✅ | ✅ | ✅ | ❌ |
| Delete Content | ✅ | ✅ | ❌ | ❌ |
| Publish/Unpublish | ✅ | ✅ | ❌ | ❌ |

### 3.4 JWT Token Structure

```json
{
  "sub": "user-uuid",
  "email": "admin@botonblue.com",
  "role": "admin",
  "hotel_id": "hotel-uuid",
  "iat": 1710000000,
  "exp": 1710003600
}
```

| Token | TTL | Storage |
|-------|-----|---------|
| Access Token | 1 giờ | httpOnly cookie |
| Refresh Token | 7 ngày | httpOnly cookie + DB whitelist |

---

## 4. Tech Stack

### 4.1 Frontend (Admin Dashboard)

| Technology | Vai trò |
|-----------|---------|
| **Next.js 14+** (App Router) | Framework, routing, SSR |
| **React 18+** | UI components |
| **TypeScript** | Type safety |
| **Tailwind CSS 3+** | Styling |
| **shadcn/ui** | Component library (Button, Table, Dialog, Form, etc.) |
| **React Hook Form** + **Zod** | Form validation |
| **TanStack Table** | Data table with sort, filter, pagination |
| **TanStack Query** | Server state management, caching |
| **Zustand** | Client state (UI state, form wizard) |
| **Recharts** | Analytics charts |
| **Tiptap** | Rich text editor (for posts, descriptions) |
| **Pannellum** | 360 viewer in hotspot editor |
| **react-dropzone** | File upload drag & drop |
| **date-fns** | Date formatting |
| **Lucide React** | Icons |
| **Sonner** | Toast notifications |

### 4.2 Backend (API)

| Technology | Vai trò |
|-----------|---------|
| **Next.js API Routes** | REST API endpoints |
| **Prisma ORM** | Database queries |
| **PostgreSQL 15+** | Primary database |
| **Redis** | Cache layer, session store |
| **bcrypt** | Password hashing |
| **jose** | JWT token management |
| **sharp** | Image resizing (panorama + thumbnails) |
| **@aws-sdk/client-s3** | S3-compatible storage upload |
| **zod** | Request body validation |

---

## 5. Module Map

### 5.1 Dashboard Modules

```
┌─────────────────────────────────────────────────────────────────┐
│                      ADMIN DASHBOARD                             │
│                                                                   │
│  ┌─ CONTENT MANAGEMENT ──────────────────────────────────────┐   │
│  │                                                            │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌──────────────┐    │   │
│  │  │ Rooms   │ │Services │ │Gallery  │ │ Pages        │    │   │
│  │  │ CRUD    │ │ CRUD    │ │ Upload  │ │ (Custom CMS) │    │   │
│  │  │ + Scene │ │ + Scene │ │ Manage  │ │              │    │   │
│  │  │ + Amen. │ │ + CTA   │ │ Albums  │ │              │    │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └──────────────┘    │   │
│  │                                                            │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌──────────────┐    │   │
│  │  │Promot.  │ │ Posts   │ │ Menus   │ │ Translations │    │   │
│  │  │ CRUD    │ │ Blog    │ │ Header  │ │ Multi-lang   │    │   │
│  │  │ Dates   │ │ Rich Ed │ │ Footer  │ │ Editor       │    │   │
│  │  │ Rooms   │ │ Tags    │ │ Mobile  │ │              │    │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └──────────────┘    │   │
│  └────────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─ 360° MANAGEMENT ─────────────────────────────────────────┐   │
│  │                                                            │   │
│  │  ┌─────────────────┐ ┌───────────────────────────────┐    │   │
│  │  │ Scene Configs   │ │ Hotspot Visual Editor         │    │   │
│  │  │ Upload panorama │ │ Click-to-place on 360         │    │   │
│  │  │ Camera settings │ │ Drag-to-reposition            │    │   │
│  │  │ Auto-rotate     │ │ Type: info/navigate/cta/gallery│   │   │
│  │  │ FOV limits      │ │ Preview mode                  │    │   │
│  │  └─────────────────┘ └───────────────────────────────────┘│   │
│  └────────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─ BOOKING CONFIGURATION ★ ─────────────────────────────────┐   │
│  │                                                            │   │
│  │  ┌─────────────────┐ ┌─────────────────────────────────┐  │   │
│  │  │ Mode Selection  │ │ Booking Modal Config            │  │   │
│  │  │ □ External Link │ │ Form fields builder             │  │   │
│  │  │ □ In-page Modal │ │ Notification settings           │  │   │
│  │  │                 │ │ Confirmation template           │  │   │
│  │  │ Per-room config │ │ Email integration               │  │   │
│  │  │ Per-service cfg │ │                                 │  │   │
│  │  └─────────────────┘ └─────────────────────────────────┘  │   │
│  └────────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─ SYSTEM ──────────────────────────────────────────────────┐   │
│  │                                                            │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌──────────────┐    │   │
│  │  │ Hotel   │ │ Users   │ │Analytics│ │ SEO / AIEO   │    │   │
│  │  │Settings │ │ Manage  │ │ Charts  │ │ Meta tags    │    │   │
│  │  │ Logo    │ │ Roles   │ │ Reports │ │ JSON-LD      │    │   │
│  │  │ Colors  │ │ Invite  │ │ Export  │ │ Sitemap      │    │   │
│  │  │ Contact │ │ Audit   │ │         │ │              │    │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └──────────────┘    │   │
│  └────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Routing Structure

### 6.1 Admin Routes

| Route | Module | Mô tả |
|-------|--------|--------|
| `/admin` | Dashboard | Tổng quan: stats, quick actions, recent activity |
| `/admin/login` | Auth | Login page |
| `/admin/rooms` | Rooms | Danh sách phòng |
| `/admin/rooms/new` | Rooms | Tạo phòng mới |
| `/admin/rooms/[id]` | Rooms | Chỉnh sửa phòng |
| `/admin/rooms/[id]/scenes` | Rooms | Quản lý scene 360 của phòng |
| `/admin/rooms/[id]/booking` | Booking | ★ Cấu hình booking cho phòng |
| `/admin/services` | Services | Danh sách dịch vụ |
| `/admin/services/new` | Services | Tạo dịch vụ mới |
| `/admin/services/[id]` | Services | Chỉnh sửa dịch vụ |
| `/admin/scenes` | Scenes | Danh sách tất cả scene configs |
| `/admin/scenes/[id]/editor` | Scenes | ★ Hotspot visual editor |
| `/admin/promotions` | Promotions | CRUD khuyến mãi |
| `/admin/promotions/new` | Promotions | Tạo promotion |
| `/admin/promotions/[id]` | Promotions | Chỉnh sửa promotion |
| `/admin/posts` | Posts | Blog / news management |
| `/admin/posts/new` | Posts | Tạo bài viết (rich editor) |
| `/admin/posts/[id]` | Posts | Chỉnh sửa bài viết |
| `/admin/gallery` | Gallery | Quản lý ảnh (albums, upload) |
| `/admin/menus` | Menus | Header + Footer menu builder |
| `/admin/booking` | Booking | ★ Booking global settings |
| `/admin/translations` | Translations | Multi-language editor |
| `/admin/settings` | Settings | Hotel info, branding, SEO |
| `/admin/users` | Users | User management (super_admin only) |
| `/admin/analytics` | Analytics | Charts, reports, export |

---

## 7. Deployment Architecture

### 7.1 Production

```
┌──────────────────────────────────────────────────────────────┐
│                        VPS / Cloud                            │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ Docker Compose                                            │ │
│  │                                                            │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │ │
│  │  │  Next.js App │  │  PostgreSQL  │  │    Redis     │    │ │
│  │  │  (Web +      │  │  15+         │  │  (Cache +    │    │ │
│  │  │   Admin +    │  │              │  │   Sessions)  │    │ │
│  │  │   API)       │  │              │  │              │    │ │
│  │  │  Port: 3000  │  │  Port: 5432  │  │  Port: 6379  │    │ │
│  │  └──────┬───────┘  └──────────────┘  └──────────────┘    │ │
│  │         │                                                  │ │
│  └─────────┼──────────────────────────────────────────────────┘ │
│            │                                                    │
│  ┌─────────┼──────────────────────────────────────────────────┐ │
│  │  Nginx Reverse Proxy                                       │ │
│  │  ├── / → Next.js (public website)                          │ │
│  │  ├── /admin → Next.js (admin dashboard)                    │ │
│  │  ├── /api → Next.js (REST API)                             │ │
│  │  └── /uploads → S3/R2 redirect (or local static)          │ │
│  └────────────────────────────────────────────────────────────┘ │
│            │                                                    │
└────────────┼────────────────────────────────────────────────────┘
             │
    ┌────────┴────────┐
    │   Cloudflare    │
    │   CDN + SSL     │
    │   DDoS protect  │
    └─────────────────┘
```

### 7.2 Tách Admin (Tùy chọn Scale)

Khi cần tách admin ra ứng dụng riêng (scale hoặc bảo mật):

```
Option A (Recommended): Cùng Next.js app, cùng domain
  → /admin/* routes, middleware auth check
  → Đơn giản, share components, share API

Option B (Advanced): Admin là app riêng
  → admin.{domain}.com
  → Next.js app riêng, gọi API cùng backend
  → Phức tạp hơn, nhưng tách biệt deploy
```

---

## 8. Sequence Diagrams

### 8.1 Admin Login Flow

```
Admin               Login Page          API Server          Database           Redis
  │                     │                │                  │                  │
  │  /admin             │                │                  │                  │
  │  (no token)         │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ render login   │                  │                  │
  │  ◄── login form ────┤               │                  │                  │
  │                     │                │                  │                  │
  │  email + password   │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ POST /admin/   │                  │                  │
  │                     │ auth/login     │                  │                  │
  │                     ├───────────────►│                  │                  │
  │                     │                │ find user by     │                  │
  │                     │                │ email            │                  │
  │                     │                ├─────────────────►│                  │
  │                     │                │ ◄── user row ────┤                  │
  │                     │                │                  │                  │
  │                     │                │ bcrypt.compare   │                  │
  │                     │                │ (password, hash) │                  │
  │                     │                │                  │                  │
  │                     │                │ generate JWT     │                  │
  │                     │                │ (access 1h +     │                  │
  │                     │                │  refresh 7d)     │                  │
  │                     │                │                  │                  │
  │                     │                │ store refresh ───────────────────────►
  │                     │                │ token in Redis   │                  │
  │                     │                │                  │                  │
  │                     │ ◄── set cookies│                  │                  │
  │                     │  (httpOnly)    │                  │                  │
  │  ◄── redirect /admin ┤              │                  │                  │
  │                     │                │                  │                  │
```

### 8.2 Admin CRUD — Create Room

```
Admin              Room Form           API Server          Database           Cache
  │                     │                │                  │                  │
  │  /admin/rooms/new   │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ render form    │                  │                  │
  │                     │ (amenities,    │                  │                  │
  │                     │  room types,   │                  │                  │
  │                     │  scene list)   │                  │                  │
  │                     │                │                  │                  │
  │  fill form data     │                │                  │                  │
  │  upload thumbnail   │                │                  │                  │
  │  select amenities   │                │                  │                  │
  │  configure booking  │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ validate (Zod) │                  │                  │
  │                     │ POST /admin/   │                  │                  │
  │                     │ rooms          │                  │                  │
  │                     ├───────────────►│                  │                  │
  │                     │                │ auth middleware   │                  │
  │                     │                │ role >= editor    │                  │
  │                     │                │                  │                  │
  │                     │                │ validate body    │                  │
  │                     │                │ (Zod schema)     │                  │
  │                     │                │                  │                  │
  │                     │                │ prisma.room      │                  │
  │                     │                │ .create()        │                  │
  │                     │                ├─────────────────►│                  │
  │                     │                │ ◄── created ─────┤                  │
  │                     │                │                  │                  │
  │                     │                │ prisma.roomAmen  │                  │
  │                     │                │ .createMany()    │                  │
  │                     │                ├─────────────────►│                  │
  │                     │                │                  │                  │
  │                     │                │ invalidate cache ────────────────────►
  │                     │                │ rooms:list:*     │                  │
  │                     │                │                  │                  │
  │                     │ ◄── 201 + room │                  │                  │
  │  ◄── redirect       │                │                  │                  │
  │  /admin/rooms/[id]  │                │                  │                  │
  │                     │                │                  │                  │
  │  toast: "Room       │                │                  │                  │
  │   created!"         │                │                  │                  │
```

### 8.3 Media Upload Pipeline

```
Admin              Upload Component     API Server          Sharp              S3/R2
  │                     │                │                  │                  │
  │  drag & drop image  │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ validate       │                  │                  │
  │                     │ (type, size,   │                  │                  │
  │                     │  dimensions)   │                  │                  │
  │                     │                │                  │                  │
  │                     │ POST /admin/   │                  │                  │
  │                     │ upload/image   │                  │                  │
  │                     │ (multipart)    │                  │                  │
  │                     ├───────────────►│                  │                  │
  │                     │                │ receive file     │                  │
  │                     │                │                  │                  │
  │                     │                │ sharp resize ────►                  │
  │                     │                │                  │ original: keep   │
  │                     │                │                  │ large: 1920w     │
  │                     │                │                  │ medium: 800w     │
  │                     │                │                  │ thumb: 300w      │
  │                     │                │                  │ webp convert     │
  │                     │                │ ◄── buffers ─────┤                  │
  │                     │                │                  │                  │
  │                     │                │ upload to S3 ──────────────────────►│
  │                     │                │                  │                  │ /uploads/
  │                     │                │                  │                  │ 2026/03/
  │                     │                │                  │                  │ {hash}.jpg
  │                     │                │                  │                  │ {hash}-lg.webp
  │                     │                │                  │                  │ {hash}-md.webp
  │                     │                │                  │                  │ {hash}-thumb.webp
  │                     │                │ ◄── URLs ────────────────────────────┤
  │                     │                │                  │                  │
  │                     │ ◄── response   │                  │                  │
  │                     │ {url, thumb,   │                  │                  │
  │                     │  width, height,│                  │                  │
  │                     │  size_bytes}   │                  │                  │
  │  ◄── preview shown  │               │                  │                  │
  │  (thumbnail)        │                │                  │                  │
```

### 8.4 Panorama Upload — Special Pipeline

```
Admin              PanoramaUploader     API Server          Sharp              S3/R2
  │                     │                │                  │                  │
  │  upload panorama    │                │                  │                  │
  │  (equirectangular)  │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ validate       │                  │                  │
  │                     │ (≥4096×2048,   │                  │                  │
  │                     │  2:1 ratio,    │                  │                  │
  │                     │  JPEG/PNG)     │                  │                  │
  │                     │                │                  │                  │
  │                     │ POST /admin/   │                  │                  │
  │                     │ upload/panorama│                  │                  │
  │                     ├───────────────►│                  │                  │
  │                     │                │                  │                  │
  │                     │                │ generate sizes ──►                  │
  │                     │                │                  │ 8192×4096 (HQ)   │
  │                     │                │                  │ 4096×2048 (std)  │
  │                     │                │                  │ 2048×1024 (mob)  │
  │                     │                │                  │ 512×256 (blur)   │
  │                     │                │                  │ 300×150 (thumb)  │
  │                     │                │ ◄── buffers ─────┤                  │
  │                     │                │                  │                  │
  │                     │                │ upload all ─────────────────────────►│
  │                     │                │                  │                  │ /pano/
  │                     │                │                  │                  │ {hash}-8192.jpg
  │                     │                │                  │                  │ {hash}-4096.jpg
  │                     │                │                  │                  │ {hash}-2048.jpg
  │                     │                │                  │                  │ {hash}-blur.jpg
  │                     │                │                  │                  │ {hash}-thumb.jpg
  │                     │                │ ◄── URLs ────────────────────────────┤
  │                     │                │                  │                  │
  │                     │ ◄── response   │                  │                  │
  │                     │ {panorama_url, │                  │                  │
  │                     │  mobile_url,   │                  │                  │
  │                     │  blur_url,     │                  │                  │
  │                     │  thumb_url}    │                  │                  │
  │  ◄── mini 360       │               │                  │                  │
  │  preview (Pannellum)│                │                  │                  │
```

---

*Tài liệu tiếp theo: **02-DASHBOARD-DATABASE-API.md** — Admin API endpoints chi tiết, request/response schemas*
