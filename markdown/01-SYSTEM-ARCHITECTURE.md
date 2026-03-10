# 01 — SYSTEM ARCHITECTURE SPECIFICATION
## Website 360 Thế Hệ Mới — Hotel CMS Platform

**Version:** 1.0  
**Date:** 2026-03-10  
**Status:** Draft  
**Project:** Website 360 New Generation — Luxury Hotel CMS  
**Sample Site:** Boton Blue Hotel & Spa (hotel.botonblue.com)

---

## 1. Tổng Quan Kiến Trúc

### 1.1 Triết Lý Thiết Kế

> "Khách vào xem một hạng mục là bước ngay vào không gian đó."

Website 360 thế hệ mới không phải là website truyền thống gắn thêm tour 360, mà là **nền tảng nội dung sống trên không gian 360 tương tác**. Kiến trúc được thiết kế theo mô hình 3 lớp tách biệt, giao tiếp qua API, có khả năng mở rộng cho nhiều khách sạn 3–5 sao.

### 1.2 Mô Hình 3 Lớp (Three-Layer Architecture)

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENT (Browser)                      │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │         LỚP 2 — 360 INTERFACE LAYER              │   │
│  │  ┌─────────────┐  ┌──────────────┐  ┌─────────┐  │   │
│  │  │ Pannellum   │  │ Overlay UI   │  │Hotspot  │  │   │
│  │  │ VR Engine   │  │ Panels/Cards │  │Manager  │  │   │
│  │  └──────┬──────┘  └──────┬───────┘  └────┬────┘  │   │
│  │         │                │               │        │   │
│  │  ┌──────┴────────────────┴───────────────┴────┐   │   │
│  │  │       React Application Shell               │   │   │
│  │  │  (Next.js App Router / React 18+)           │   │   │
│  │  └──────────────────┬──────────────────────────┘   │   │
│  └─────────────────────┼──────────────────────────────┘   │
│                        │                                   │
│  ┌─────────────────────┼──────────────────────────────┐   │
│  │         LỚP 1 — CMS WEBSITE LAYER                 │   │
│  │                     │                               │   │
│  │  ┌─────────┐  ┌────┴─────┐  ┌──────────┐          │   │
│  │  │ Content │  │ Routing  │  │ SEO/AIEO │          │   │
│  │  │ Modules │  │ & Nav    │  │ & i18n   │          │   │
│  │  └─────────┘  └──────────┘  └──────────┘          │   │
│  └─────────────────────┬──────────────────────────────┘   │
└────────────────────────┼──────────────────────────────────┘
                         │ REST API / GraphQL
┌────────────────────────┼──────────────────────────────────┐
│            BACKEND + LỚP 3 — VR ENGINE                    │
│                        │                                   │
│  ┌─────────────────────┴──────────────────────────────┐   │
│  │              API Gateway (Node.js / Express)        │   │
│  │                                                      │   │
│  │  ┌──────────┐  ┌──────────┐  ┌────────────────┐    │   │
│  │  │ CMS API  │  │ Scene    │  │ Booking Bridge │    │   │
│  │  │ CRUD     │  │ Mapping  │  │ (External)     │    │   │
│  │  │          │  │ API      │  │                │    │   │
│  │  └────┬─────┘  └────┬─────┘  └───────┬────────┘    │   │
│  │       │              │                │              │   │
│  │  ┌────┴──────────────┴────────────────┴──────────┐  │   │
│  │  │         PostgreSQL / MySQL Database            │  │   │
│  │  └───────────────────────────────────────────────┘  │   │
│  │                                                      │   │
│  │  ┌───────────────────────────────────────────────┐  │   │
│  │  │     Link VR Engine (External 360 Service)      │  │   │
│  │  │     — Tour liền mạch, dẫn tour, chia sẻ góc   │  │   │
│  │  └───────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              File Storage (S3 / CDN)                  │   │
│  │   — Equirectangular images, gallery, assets           │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Chi Tiết Từng Lớp

### 2.1 Lớp 1 — CMS Website Layer

**Vai trò:** Quản lý toàn bộ nội dung, cấu trúc điều hướng, SEO, đa ngôn ngữ, phân quyền.

| Module | Chức năng |
|--------|-----------|
| Rooms | CRUD loại phòng, thông tin tiện nghi, giá, scene mapping |
| Dining | Nhà hàng, bar, menu, giờ phục vụ |
| Recreation | Spa, pool, fitness, beach, meetings |
| Promotions | Ưu đãi, thời hạn, điều kiện, banner |
| News / Blog | Bài viết, tin tức, SEO articles |
| Gallery | Quản lý ảnh theo category, tag |
| Pages | Trang tĩnh tùy chỉnh (About, Contact, Policy) |
| Settings | Logo, thông tin KS, social links, booking URL |
| i18n | Đa ngôn ngữ (vi, en, ru, ko, zh) |
| SEO/AIEO | Meta tags, structured data, sitemap, AI search optimization |
| Users | Phân quyền admin, editor, viewer |

**Dashboard Admin:** Standalone dashboard (không SaaS), mỗi khách sạn có instance riêng.

### 2.2 Lớp 2 — 360 Interface Layer

**Vai trò:** Lớp giao diện tạo khác biệt — render scene 360 làm nền trang, overlay UI panels, hotspot tương tác.

| Component | Mô tả |
|-----------|-------|
| SceneViewer | Render equirectangular panorama (Pannellum / Three.js) toàn màn hình hoặc bán toàn |
| OverlayManager | Quản lý panel thông tin (show/hide/animate) |
| HotspotEngine | Quản lý hotspot vị trí, loại (navigation, info, CTA) |
| SceneNavigator | Chuyển scene trong cùng hạng mục (thumbnail bar) |
| BookingWidget | CTA đặt phòng nổi, nhúng trong trải nghiệm 360 |
| GalleryOverlay | Lightbox gallery ảnh thật trên nền 360 |
| InfoPanel | Drawer/floating card: tên, mô tả, tiện nghi, diện tích |
| SmartUI | Auto show/hide UI dựa trên tương tác người dùng |

### 2.3 Lớp 3 — Link VR Engine (External)

**Vai trò:** Giữ nguyên hệ thống tour 360 hiện tại của Link. Website mới khai thác nó như engine.

| Khả năng | Cách tích hợp |
|----------|---------------|
| Tour liền mạch | Nhúng iframe hoặc gọi scene riêng qua embed URL |
| Chia sẻ từng góc | Deep link đến scene cụ thể |
| Dẫn tour online | Giữ nguyên giao diện Link riêng cho sales |
| Scene data | API hoặc JSON config mapping scene ID → hạng mục |

---

## 3. Tech Stack

### 3.1 Frontend

| Công nghệ | Vai trò | Lý do chọn |
|-----------|---------|-------------|
| **Next.js 14+** (App Router) | Framework React chính | SSR/SSG cho SEO, routing, i18n, image optimization |
| **React 18+** | UI Library | Concurrent rendering, Suspense cho scene loading |
| **TypeScript** | Type safety | Giảm bug, tăng DX cho team dev |
| **Tailwind CSS 3+** | Styling | Utility-first, responsive nhanh, custom design tokens |
| **Framer Motion** | Animations | Overlay show/hide, panel transitions, page transitions |
| **Pannellum** (hoặc Photo Sphere Viewer) | 360 Viewer | Lightweight, tùy biến cao, hotspot API tốt |
| **Zustand** | State management | Nhẹ, đủ cho scene state, UI state, filter state |
| **React Query (TanStack)** | Server state | Cache, revalidate, optimistic updates cho CMS data |
| **next-intl** | i18n | Đa ngôn ngữ tích hợp Next.js |
| **next-seo** | SEO | Meta tags, JSON-LD, Open Graph |

### 3.2 Backend

| Công nghệ | Vai trò |
|-----------|---------|
| **Node.js 20+ / Express** hoặc **Next.js API Routes** | API server |
| **Prisma ORM** | Database access, migration, type-safe queries |
| **PostgreSQL 15+** | Primary database |
| **Redis** | Cache layer (scene config, menu, settings) |
| **JWT + bcrypt** | Authentication cho admin dashboard |
| **Multer + Sharp** | Upload và xử lý ảnh |
| **AWS S3 / Cloudflare R2** | File storage (panorama, gallery images) |
| **Cloudflare CDN** | Asset delivery, edge caching |

### 3.3 DevOps & Deployment

| Công cụ | Vai trò |
|---------|---------|
| **Docker + Docker Compose** | Containerization |
| **Nginx** | Reverse proxy, SSL termination |
| **GitHub Actions** | CI/CD pipeline |
| **VPS (Contabo / Hetzner / DO)** | Hosting standalone (không SaaS) |
| **Let's Encrypt** | SSL certificates |
| **PM2** | Process manager cho Node.js |

---

## 4. Luồng Dữ Liệu Chính

### 4.1 Khách truy cập trang phòng

```
[Browser] → GET /rooms/deluxe-ocean
    │
    ├── Next.js SSR/SSG
    │   ├── Fetch room data từ API (name, desc, amenities, price...)
    │   ├── Fetch scene mapping (room_slug → scene_id, panorama_url)
    │   ├── Fetch hotspots config cho scene
    │   └── Fetch gallery images
    │
    ├── Render HTML với SEO meta (SSR)
    │
    └── Client Hydration
        ├── Pannellum init với panorama_url
        ├── Load hotspots vào scene
        ├── Mount OverlayManager (InfoPanel, BookingCTA)
        └── SmartUI bắt đầu theo dõi user interaction
```

### 4.2 Admin cập nhật nội dung

```
[Admin Dashboard] → Login (JWT)
    │
    ├── CRUD Room
    │   ├── Tên, mô tả, tiện nghi, diện tích, occupancy
    │   ├── Upload gallery images → S3
    │   ├── Chọn scene 360 mapping (dropdown từ scene list)
    │   ├── Cấu hình hotspots (vị trí, loại, nội dung)
    │   └── SEO fields (meta title, description, slug)
    │
    ├── Save → API → Database
    │   └── Invalidate cache (Redis + CDN)
    │
    └── Preview → Xem trang 360 với nội dung mới
```

---

## 5. Chiến Lược SEO / AIEO

### 5.1 Technical SEO

- SSR/SSG đảm bảo crawler đọc được nội dung đầy đủ
- Mỗi trang có `<title>`, `<meta description>`, canonical URL
- JSON-LD structured data (Hotel, Room, Restaurant schema)
- Sitemap XML tự động generate
- Hreflang tags cho đa ngôn ngữ
- Core Web Vitals: LCP < 2.5s (panorama lazy load), CLS < 0.1

### 5.2 AIEO (AI Engine Optimization)

- Structured data giúp AI search engines hiểu nội dung
- FAQ schema cho câu hỏi phổ biến
- Breadcrumb schema cho navigation
- Review/Rating schema khi có

---

## 6. Deployment Architecture (Standalone)

```
┌─────────────────────────────────────────┐
│              VPS Server                  │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │           Nginx (Port 80/443)       │  │
│  │   SSL Termination + Reverse Proxy   │  │
│  └───────────────┬────────────────────┘  │
│                  │                        │
│  ┌───────────────┴────────────────────┐  │
│  │   Docker Compose                    │  │
│  │                                     │  │
│  │  ┌────────────┐  ┌──────────────┐  │  │
│  │  │ Next.js App│  │ PostgreSQL   │  │  │
│  │  │ (PM2)      │  │ (Container)  │  │  │
│  │  │ Port 3000  │  │ Port 5432    │  │  │
│  │  └────────────┘  └──────────────┘  │  │
│  │                                     │  │
│  │  ┌────────────┐  ┌──────────────┐  │  │
│  │  │ Redis      │  │ File Upload  │  │  │
│  │  │ Port 6379  │  │ Volume       │  │  │
│  │  └────────────┘  └──────────────┘  │  │
│  └─────────────────────────────────────┘  │
└───────────────────────────────────────────┘
         │
         │ CDN (Static assets, panoramas)
         ▼
┌─────────────────────┐
│  Cloudflare / S3    │
└─────────────────────┘
```

---

## 7. Phân Phase Triển Khai

| Phase | Nội dung | Thời gian ước tính |
|-------|---------|-------------------|
| **Phase 1** | CMS Backend + Admin Dashboard + API | 4–6 tuần |
| **Phase 2** | Frontend: Homepage, Room pages với 360-first layout | 4–6 tuần |
| **Phase 3** | 360 Interface Layer: Pannellum integration, hotspots, overlays | 3–4 tuần |
| **Phase 4** | Service pages (Dining, Spa, Pool, Meeting) + Booking integration | 3–4 tuần |
| **Phase 5** | SEO/AIEO, i18n, performance optimization, testing | 2–3 tuần |
| **Phase 6** | Deploy, QA, UAT, go-live | 1–2 tuần |

**Tổng ước tính:** 17–25 tuần (4–6 tháng)

---

## 8. Yêu Cầu Phi Chức Năng

| Tiêu chí | Yêu cầu |
|----------|---------|
| Performance | First Contentful Paint < 1.5s, LCP < 2.5s, panorama load < 3s trên 4G |
| Responsive | Desktop, tablet, mobile (ưu tiên mobile-first) |
| Browser | Chrome 90+, Safari 15+, Firefox 90+, Edge 90+ |
| Accessibility | WCAG 2.1 AA (keyboard nav, alt text, contrast) |
| Security | HTTPS, CSRF protection, input sanitization, rate limiting |
| Uptime | 99.5%+ |
| Backup | Daily automated database backup |

---

*Tài liệu tiếp theo: **02-DATABASE-API-SPEC.md** — Chi tiết Database Schema và API Endpoints*
