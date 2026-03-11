# 02 — DASHBOARD DATABASE & API SPECIFICATION
## Admin Dashboard — Hotel CMS Platform

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** markdown-hotel/02-DATABASE-API-SPEC · 01-DASHBOARD-ARCHITECTURE · 03~05

---

## Mục Lục

1. [Database Schema Bổ Sung (Dashboard-specific)](#1-database-schema-bổ-sung)
2. [Booking Configuration Schema ★](#2-booking-configuration-schema)
3. [Admin API Endpoints Chi Tiết](#3-admin-api-endpoints-chi-tiết)
4. [Request / Response Schemas](#4-request--response-schemas)
5. [Booking API ★](#5-booking-api)
6. [Validation Rules](#6-validation-rules)
7. [Sequence Diagrams](#7-sequence-diagrams)

---

## 1. Database Schema Bổ Sung

> Schema gốc đã có trong `markdown-hotel/02-DATABASE-API-SPEC.md`. Phần này bổ sung các bảng dành riêng cho dashboard & booking configuration.

### 1.1 Bảng Bổ Sung

```sql
-- ============================================
-- BOOKING CONFIGURATION ★
-- ============================================
CREATE TABLE booking_configs (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id      UUID NOT NULL REFERENCES hotels(id),

  -- Global booking mode
  -- 'external': redirect to external booking URL
  -- 'modal': in-page booking modal (form submission)
  -- 'both': admin chọn per-room/per-service
  global_mode   VARCHAR(20) NOT NULL DEFAULT 'external',

  -- External mode settings
  external_url  VARCHAR(500),                -- Default booking URL
  external_target VARCHAR(10) DEFAULT '_blank', -- '_blank' | '_self'

  -- Modal mode settings
  modal_title         VARCHAR(200) DEFAULT 'Book Your Stay',
  modal_subtitle      VARCHAR(500),
  modal_success_msg   TEXT DEFAULT 'Thank you! We will contact you shortly.',
  modal_btn_label     VARCHAR(100) DEFAULT 'Send Booking Request',
  modal_btn_color     VARCHAR(7),            -- Hex override (null = dùng theme)

  -- Notification settings (khi guest submit modal form)
  notify_emails       TEXT[],                -- Danh sách email nhận thông báo
  notify_webhook_url  VARCHAR(500),          -- Optional webhook (Slack, Zalo, etc.)
  auto_reply_enabled  BOOLEAN DEFAULT true,  -- Gửi email xác nhận cho khách
  auto_reply_subject  VARCHAR(300) DEFAULT 'Booking Request Received',
  auto_reply_template TEXT,                  -- HTML email template

  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- BOOKING FORM FIELDS (Modal mode)
-- ============================================
CREATE TABLE booking_form_fields (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  config_id     UUID NOT NULL REFERENCES booking_configs(id) ON DELETE CASCADE,
  field_name    VARCHAR(50) NOT NULL,        -- 'full_name', 'email', 'phone', etc.
  field_type    VARCHAR(30) NOT NULL,        -- 'text'|'email'|'tel'|'date'|'select'|'textarea'|'number'
  label         VARCHAR(200) NOT NULL,       -- Display label
  placeholder   VARCHAR(200),
  is_required   BOOLEAN DEFAULT false,
  options       JSONB,                       -- For 'select' type: [{value, label}]
  validation    JSONB,                       -- {min, max, pattern, message}
  sort_order    INT DEFAULT 0,
  is_active     BOOLEAN DEFAULT true,
  created_at    TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- BOOKING REQUESTS (Modal mode submissions)
-- ============================================
CREATE TABLE booking_requests (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  hotel_id      UUID NOT NULL REFERENCES hotels(id),
  room_id       UUID REFERENCES rooms(id),
  service_id    UUID REFERENCES services(id),

  -- Guest info (from modal form)
  guest_name    VARCHAR(200),
  guest_email   VARCHAR(200),
  guest_phone   VARCHAR(50),
  check_in      DATE,
  check_out     DATE,
  adults        INT DEFAULT 2,
  children      INT DEFAULT 0,
  special_requests TEXT,
  form_data     JSONB DEFAULT '{}',          -- All form fields as JSON

  -- Status tracking
  status        VARCHAR(20) DEFAULT 'new',   -- 'new'|'contacted'|'confirmed'|'cancelled'
  notes         TEXT,                        -- Admin notes
  assigned_to   UUID REFERENCES users(id),

  -- Meta
  source_page   VARCHAR(200),               -- Page where form was submitted
  source_scene  VARCHAR(100),               -- 360 scene ID (if from 360 view)
  user_agent    TEXT,
  ip_hash       VARCHAR(64),
  session_id    VARCHAR(100),

  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_booking_requests_hotel ON booking_requests(hotel_id, created_at DESC);
CREATE INDEX idx_booking_requests_status ON booking_requests(status);

-- ============================================
-- ROOM BOOKING OVERRIDES
-- ============================================
ALTER TABLE rooms ADD COLUMN IF NOT EXISTS
  booking_mode VARCHAR(20) DEFAULT NULL;
  -- NULL = dùng global_mode
  -- 'external' = override sang external
  -- 'modal' = override sang modal
  -- Room đã có booking_url (external URL override)

-- ============================================
-- SERVICE BOOKING OVERRIDES
-- ============================================
-- services table đã có: cta_type, cta_label, cta_url
-- Thêm trường cho modal mode:
ALTER TABLE services ADD COLUMN IF NOT EXISTS
  booking_mode VARCHAR(20) DEFAULT NULL;
  -- NULL = dùng global_mode
  -- 'external' = redirect sang cta_url
  -- 'modal' = mở booking modal

-- ============================================
-- AUDIT LOG
-- ============================================
CREATE TABLE audit_logs (
  id            BIGSERIAL PRIMARY KEY,
  hotel_id      UUID NOT NULL REFERENCES hotels(id),
  user_id       UUID REFERENCES users(id),
  action        VARCHAR(50) NOT NULL,        -- 'create'|'update'|'delete'|'publish'|'login'
  entity_type   VARCHAR(50) NOT NULL,        -- 'room'|'service'|'promotion'|'post'|'scene'|'hotspot'
  entity_id     UUID,
  changes       JSONB,                       -- {field: {old, new}} diff
  ip_address    VARCHAR(45),
  user_agent    TEXT,
  created_at    TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_audit_hotel_date ON audit_logs(hotel_id, created_at DESC);
CREATE INDEX idx_audit_user ON audit_logs(user_id, created_at DESC);

-- ============================================
-- ACTIVITY / SESSION TRACKING
-- ============================================
CREATE TABLE admin_sessions (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID NOT NULL REFERENCES users(id),
  refresh_token VARCHAR(500) NOT NULL,
  ip_address    VARCHAR(45),
  user_agent    TEXT,
  expires_at    TIMESTAMPTZ NOT NULL,
  created_at    TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_sessions_user ON admin_sessions(user_id);
CREATE INDEX idx_sessions_token ON admin_sessions(refresh_token);
```

### 1.2 Prisma Schema Bổ Sung

```prisma
model BookingConfig {
  id              String   @id @default(uuid())
  hotelId         String   @unique @map("hotel_id")
  globalMode      String   @default("external") @map("global_mode")
  externalUrl     String?  @map("external_url")
  externalTarget  String   @default("_blank") @map("external_target")
  modalTitle      String   @default("Book Your Stay") @map("modal_title")
  modalSubtitle   String?  @map("modal_subtitle")
  modalSuccessMsg String?  @map("modal_success_msg")
  modalBtnLabel   String   @default("Send Booking Request") @map("modal_btn_label")
  modalBtnColor   String?  @map("modal_btn_color")
  notifyEmails    String[] @map("notify_emails")
  notifyWebhookUrl String? @map("notify_webhook_url")
  autoReplyEnabled Boolean @default(true) @map("auto_reply_enabled")
  autoReplySubject String  @default("Booking Request Received") @map("auto_reply_subject")
  autoReplyTemplate String? @map("auto_reply_template")
  createdAt       DateTime @default(now()) @map("created_at")
  updatedAt       DateTime @updatedAt @map("updated_at")

  hotel           Hotel    @relation(fields: [hotelId], references: [id])
  formFields      BookingFormField[]

  @@map("booking_configs")
}

model BookingFormField {
  id          String   @id @default(uuid())
  configId    String   @map("config_id")
  fieldName   String   @map("field_name")
  fieldType   String   @map("field_type")
  label       String
  placeholder String?
  isRequired  Boolean  @default(false) @map("is_required")
  options     Json?
  validation  Json?
  sortOrder   Int      @default(0) @map("sort_order")
  isActive    Boolean  @default(true) @map("is_active")
  createdAt   DateTime @default(now()) @map("created_at")

  config      BookingConfig @relation(fields: [configId], references: [id], onDelete: Cascade)

  @@map("booking_form_fields")
}

model BookingRequest {
  id              String   @id @default(uuid())
  hotelId         String   @map("hotel_id")
  roomId          String?  @map("room_id")
  serviceId       String?  @map("service_id")
  guestName       String?  @map("guest_name")
  guestEmail      String?  @map("guest_email")
  guestPhone      String?  @map("guest_phone")
  checkIn         DateTime? @map("check_in") @db.Date
  checkOut        DateTime? @map("check_out") @db.Date
  adults          Int      @default(2)
  children        Int      @default(0)
  specialRequests String?  @map("special_requests")
  formData        Json     @default("{}") @map("form_data")
  status          String   @default("new")
  notes           String?
  assignedTo      String?  @map("assigned_to")
  sourcePage      String?  @map("source_page")
  sourceScene     String?  @map("source_scene")
  userAgent       String?  @map("user_agent")
  ipHash          String?  @map("ip_hash")
  sessionId       String?  @map("session_id")
  createdAt       DateTime @default(now()) @map("created_at")
  updatedAt       DateTime @updatedAt @map("updated_at")

  hotel           Hotel    @relation(fields: [hotelId], references: [id])
  room            Room?    @relation(fields: [roomId], references: [id])
  service         Service? @relation(fields: [serviceId], references: [id])
  assignee        User?    @relation(fields: [assignedTo], references: [id])

  @@map("booking_requests")
}

model AuditLog {
  id          BigInt   @id @default(autoincrement())
  hotelId     String   @map("hotel_id")
  userId      String?  @map("user_id")
  action      String
  entityType  String   @map("entity_type")
  entityId    String?  @map("entity_id")
  changes     Json?
  ipAddress   String?  @map("ip_address")
  userAgent   String?  @map("user_agent")
  createdAt   DateTime @default(now()) @map("created_at")

  hotel       Hotel    @relation(fields: [hotelId], references: [id])
  user        User?    @relation(fields: [userId], references: [id])

  @@map("audit_logs")
}
```

---

## 2. Booking Configuration Schema ★

### 2.1 Hai Chế Độ Booking

```
┌─────────────────────────────────────────────────────────────────┐
│                   BOOKING CONFIGURATION                          │
│                                                                   │
│  ┌─ OPTION 1: EXTERNAL LINK ──────────────────────────────────┐ │
│  │                                                              │ │
│  │  Khi khách click "Book Now" → mở tab mới (hoặc cùng tab)   │ │
│  │  → URL booking engine bên ngoài (Book Direct Online, etc.)  │ │
│  │                                                              │ │
│  │  Config:                                                     │ │
│  │  • Global external_url: "https://book-directonline.com/..." │ │
│  │  • Per-room override: rooms.booking_url                     │ │
│  │  • Per-service override: services.cta_url                   │ │
│  │  • Target: _blank (tab mới) hoặc _self (cùng tab)          │ │
│  │                                                              │ │
│  │  Flow: Click CTA → window.open(url, target) → DONE         │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌─ OPTION 2: IN-PAGE BOOKING MODAL ──────────────────────────┐ │
│  │                                                              │ │
│  │  Khi khách click "Book Now" → mở modal form trên cùng page │ │
│  │  → Khách điền thông tin → Submit → Hotel nhận notification  │ │
│  │                                                              │ │
│  │  Config:                                                     │ │
│  │  • Modal title, subtitle, success message                   │ │
│  │  • Form fields (dynamic, configurable from admin)           │ │
│  │  • Notification emails (hotel receives request)             │ │
│  │  • Auto-reply email template (guest receives confirmation)  │ │
│  │  • Optional webhook (Slack, Zalo OA, Telegram)              │ │
│  │                                                              │ │
│  │  Flow:                                                       │ │
│  │  Click CTA → Modal opens → Fill form → Submit               │ │
│  │  → POST /api/v1/booking/request                             │ │
│  │  → Save to booking_requests table                           │ │
│  │  → Send email to hotel notify_emails                        │ │
│  │  → Send auto-reply to guest email                           │ │
│  │  → Show success message in modal                            │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌─ GLOBAL vs PER-ITEM OVERRIDE ──────────────────────────────┐ │
│  │                                                              │ │
│  │  Priority chain (highest → lowest):                         │ │
│  │  1. Room/Service booking_mode (if set)                      │ │
│  │  2. Room/Service booking_url (if set, for external)         │ │
│  │  3. booking_configs.global_mode                             │ │
│  │  4. booking_configs.external_url (default external URL)     │ │
│  │                                                              │ │
│  │  Ví dụ:                                                      │ │
│  │  • Hotel global = 'modal'                                   │ │
│  │  • Room "Suite" override = 'external' (VIP booking engine)  │ │
│  │  • Room "Deluxe" = null → dùng global 'modal'              │ │
│  │  • Service "Spa" override = 'modal' (custom form)           │ │
│  │  • Service "Meeting" = null → dùng global                  │ │
│  └──────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Default Form Fields (Modal Mode)

Khi admin bật modal mode lần đầu, hệ thống tạo sẵn các fields mặc định:

| # | field_name | field_type | label | required | Mô tả |
|---|-----------|-----------|-------|----------|-------|
| 1 | `full_name` | text | Full Name | ✅ | Tên khách |
| 2 | `email` | email | Email Address | ✅ | Email khách |
| 3 | `phone` | tel | Phone Number | ✅ | SĐT khách |
| 4 | `check_in` | date | Check-in Date | ✅ | Ngày nhận phòng |
| 5 | `check_out` | date | Check-out Date | ✅ | Ngày trả phòng |
| 6 | `adults` | number | Adults | ✅ | Số người lớn (min:1, max:10) |
| 7 | `children` | number | Children | ❌ | Số trẻ em (min:0, max:10) |
| 8 | `special_requests` | textarea | Special Requests | ❌ | Yêu cầu đặc biệt |

Admin có thể:
- Thêm/xóa/sắp xếp fields
- Thay đổi required status
- Thêm trường tùy chỉnh (select cho room type, nationality, etc.)

---

## 3. Admin API Endpoints Chi Tiết

### 3.1 Authentication

| Method | Endpoint | Body | Response |
|--------|----------|------|----------|
| POST | `/admin/auth/login` | `{email, password}` | `{user, access_token}` + Set-Cookie |
| POST | `/admin/auth/refresh` | Cookie refresh_token | New access_token |
| POST | `/admin/auth/logout` | — | Invalidate tokens |
| GET | `/admin/auth/me` | — | Current user info |
| PUT | `/admin/auth/password` | `{current, new}` | 200 OK |

### 3.2 Rooms Management

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/admin/rooms` | Danh sách (incl. draft, sort, filter) |
| POST | `/admin/rooms` | Tạo phòng mới |
| GET | `/admin/rooms/:id` | Chi tiết phòng (full relations) |
| PUT | `/admin/rooms/:id` | Cập nhật phòng |
| DELETE | `/admin/rooms/:id` | Xóa phòng (soft delete) |
| PUT | `/admin/rooms/:id/publish` | Publish / Unpublish |
| PUT | `/admin/rooms/:id/sort` | Đổi thứ tự |
| GET | `/admin/rooms/:id/scenes` | Danh sách scenes của phòng |
| PUT | `/admin/rooms/:id/scenes` | Cập nhật scene mapping |
| GET | `/admin/rooms/:id/amenities` | Danh sách amenities |
| PUT | `/admin/rooms/:id/amenities` | Cập nhật amenities |
| GET | `/admin/rooms/:id/gallery` | Gallery images |
| POST | `/admin/rooms/:id/gallery` | Upload ảnh gallery |
| DELETE | `/admin/rooms/:id/gallery/:imgId` | Xóa ảnh |
| PUT | `/admin/rooms/:id/gallery/sort` | Sắp xếp gallery |
| PUT | `/admin/rooms/:id/booking` | ★ Cập nhật booking config override |

### 3.3 Services Management

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/admin/services` | Danh sách |
| POST | `/admin/services` | Tạo dịch vụ |
| GET | `/admin/services/:id` | Chi tiết |
| PUT | `/admin/services/:id` | Cập nhật |
| DELETE | `/admin/services/:id` | Xóa |
| PUT | `/admin/services/:id/publish` | Publish / Unpublish |
| PUT | `/admin/services/:id/booking` | ★ Cập nhật booking override |

### 3.4 Scenes & Hotspots

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/admin/scenes` | Tất cả scene configs |
| POST | `/admin/scenes` | Tạo scene (+ upload panorama) |
| GET | `/admin/scenes/:id` | Chi tiết scene config |
| PUT | `/admin/scenes/:id` | Cập nhật config (yaw, pitch, etc.) |
| DELETE | `/admin/scenes/:id` | Xóa scene |
| GET | `/admin/scenes/:id/hotspots` | Hotspots của scene |
| POST | `/admin/scenes/:id/hotspots` | Thêm hotspot (click-to-place) |
| PUT | `/admin/hotspots/:id` | Sửa hotspot (drag-to-reposition) |
| DELETE | `/admin/hotspots/:id` | Xóa hotspot |
| PUT | `/admin/hotspots/:id/position` | Cập nhật pitch/yaw (drag) |

### 3.5 Promotions

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/admin/promotions` | Danh sách (incl. expired) |
| POST | `/admin/promotions` | Tạo mới |
| GET | `/admin/promotions/:id` | Chi tiết |
| PUT | `/admin/promotions/:id` | Cập nhật |
| DELETE | `/admin/promotions/:id` | Xóa |
| PUT | `/admin/promotions/:id/publish` | Publish / Unpublish |

### 3.6 Posts / Blog

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/admin/posts` | Danh sách (filter: status, category) |
| POST | `/admin/posts` | Tạo bài viết |
| GET | `/admin/posts/:id` | Chi tiết |
| PUT | `/admin/posts/:id` | Cập nhật |
| DELETE | `/admin/posts/:id` | Xóa |
| PUT | `/admin/posts/:id/publish` | Publish / Unpublish |

### 3.7 Booking Configuration ★

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/admin/booking/config` | Lấy booking config hiện tại |
| PUT | `/admin/booking/config` | Cập nhật global booking config |
| GET | `/admin/booking/fields` | Danh sách form fields |
| POST | `/admin/booking/fields` | Thêm form field |
| PUT | `/admin/booking/fields/:id` | Sửa form field |
| DELETE | `/admin/booking/fields/:id` | Xóa form field |
| PUT | `/admin/booking/fields/sort` | Sắp xếp fields |
| GET | `/admin/booking/requests` | Danh sách booking requests |
| GET | `/admin/booking/requests/:id` | Chi tiết request |
| PUT | `/admin/booking/requests/:id` | Cập nhật status / notes |
| DELETE | `/admin/booking/requests/:id` | Xóa request |
| POST | `/admin/booking/test-email` | Gửi test email thông báo |

### 3.8 Other Modules

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET/PUT | `/admin/settings` | Hotel settings (name, logo, colors, contact) |
| GET | `/admin/amenities` | Master list amenities |
| POST/PUT/DELETE | `/admin/amenities/:id` | CRUD amenities |
| GET/PUT | `/admin/menus` | Menu items (header, footer, mobile) |
| GET/PUT | `/admin/translations/:type/:id` | i18n translations |
| GET/POST/PUT/DELETE | `/admin/users` | User management (super_admin) |
| POST | `/admin/upload/image` | Upload ảnh thường |
| POST | `/admin/upload/panorama` | Upload ảnh 360 equirectangular |
| GET | `/admin/analytics/overview` | Dashboard stats |
| GET | `/admin/analytics/scenes` | Scene view rankings |
| GET | `/admin/analytics/hotspots` | Hotspot click rankings |
| GET | `/admin/analytics/booking` | Booking analytics |
| GET | `/admin/analytics/export` | Export CSV/Excel |
| GET | `/admin/audit-logs` | Activity audit logs |

---

## 4. Request / Response Schemas

### 4.1 Create Room — POST `/admin/rooms`

**Request:**
```json
{
  "name": "Deluxe Twin Room With Ocean View",
  "slug": "deluxe-ocean",
  "room_type": "deluxe",
  "area_sqm": 35.0,
  "max_adults": 2,
  "max_children": 1,
  "bed_type": "twin",
  "description": "<p>Phòng Deluxe Twin với tầm nhìn ra biển...</p>",
  "short_desc": "Tầm nhìn 180° ra vịnh Nha Trang",
  "price_from": 2500000,
  "currency": "VND",
  "view_type": "ocean",
  "highlights": ["Panorama ocean view", "Balcony", "Wireless Internet"],
  "booking_url": null,
  "booking_mode": null,
  "seo_title": "Deluxe Twin Ocean View | Boton Blue Hotel",
  "seo_description": "Experience the breathtaking ocean view...",
  "is_published": false,
  "amenity_ids": ["uuid-1", "uuid-2", "uuid-3"],
  "scene_ids": ["deluxe-ocean-bedroom", "deluxe-ocean-balcony"]
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "name": "Deluxe Twin Room With Ocean View",
  "slug": "deluxe-ocean",
  "is_published": false,
  "created_at": "2026-03-10T10:00:00Z",
  "message": "Room created successfully"
}
```

### 4.2 Update Room — PUT `/admin/rooms/:id`

**Request:** Same fields as create (partial update supported).

### 4.3 Scene Config — PUT `/admin/scenes/:id`

**Request:**
```json
{
  "scene_id": "deluxe-ocean-bedroom",
  "panorama_url": "https://cdn.../pano/deluxe-ocean-bedroom-8192.jpg",
  "mobile_panorama_url": "https://cdn.../pano/deluxe-ocean-bedroom-2048.jpg",
  "blur_url": "https://cdn.../pano/deluxe-ocean-bedroom-blur.jpg",
  "thumbnail_url": "https://cdn.../pano/deluxe-ocean-bedroom-thumb.jpg",
  "label": "Bedroom",
  "default_yaw": 0,
  "default_pitch": 0,
  "default_hfov": 100,
  "min_hfov": 50,
  "max_hfov": 120,
  "auto_rotate": 0.5
}
```

### 4.4 Create Hotspot — POST `/admin/scenes/:id/hotspots`

**Request:**
```json
{
  "type": "navigate",
  "pitch": -5.2,
  "yaw": 120.5,
  "label": "Go to Balcony",
  "icon": "arrow",
  "target_scene_id": "deluxe-ocean-balcony",
  "description": null,
  "image_url": null,
  "cta_url": null,
  "cta_label": null
}
```

### 4.5 Hotspot Position Update — PUT `/admin/hotspots/:id/position`

**Request (lightweight, cho drag):**
```json
{
  "pitch": -5.5,
  "yaw": 118.2
}
```

---

## 5. Booking API ★

### 5.1 Public Booking API (Guest-facing)

#### GET `/api/v1/booking/config`

Trả về booking configuration cho frontend render CTA buttons.

**Response:**
```json
{
  "mode": "modal",
  "external": {
    "url": "https://book-directonline.com/properties/botonblue",
    "target": "_blank"
  },
  "modal": {
    "title": "Book Your Stay",
    "subtitle": "Fill in your details and we'll confirm your reservation",
    "btn_label": "Send Booking Request",
    "btn_color": "#C9A861",
    "fields": [
      {
        "name": "full_name",
        "type": "text",
        "label": "Full Name",
        "placeholder": "Enter your full name",
        "required": true
      },
      {
        "name": "email",
        "type": "email",
        "label": "Email Address",
        "placeholder": "your@email.com",
        "required": true
      },
      {
        "name": "phone",
        "type": "tel",
        "label": "Phone Number",
        "placeholder": "+84...",
        "required": true
      },
      {
        "name": "check_in",
        "type": "date",
        "label": "Check-in Date",
        "required": true
      },
      {
        "name": "check_out",
        "type": "date",
        "label": "Check-out Date",
        "required": true
      },
      {
        "name": "adults",
        "type": "number",
        "label": "Adults",
        "required": true,
        "validation": {"min": 1, "max": 10}
      },
      {
        "name": "children",
        "type": "number",
        "label": "Children",
        "required": false,
        "validation": {"min": 0, "max": 10}
      },
      {
        "name": "special_requests",
        "type": "textarea",
        "label": "Special Requests",
        "placeholder": "Any special requirements...",
        "required": false
      }
    ]
  }
}
```

#### POST `/api/v1/booking/request`

Guest submit booking form (modal mode).

**Request:**
```json
{
  "room_id": "uuid",
  "room_name": "Deluxe Twin Ocean View",
  "full_name": "Nguyễn Văn A",
  "email": "guest@email.com",
  "phone": "+84 901 234 567",
  "check_in": "2026-04-15",
  "check_out": "2026-04-18",
  "adults": 2,
  "children": 1,
  "special_requests": "Late check-in around 10pm",
  "source_page": "/rooms/deluxe-ocean",
  "source_scene": "deluxe-ocean-bedroom",
  "session_id": "abc123"
}
```

**Response (201):**
```json
{
  "success": true,
  "message": "Booking request received! We'll contact you within 24 hours.",
  "request_id": "BR-2026-0315"
}
```

**Server-side actions (sau khi save):**
1. Insert vào `booking_requests` table
2. Send email to `booking_configs.notify_emails`
3. Send auto-reply email to `guest_email` (if enabled)
4. Send webhook notification (if configured)
5. Track analytics event `booking_request_submit`

### 5.2 Room Booking Mode Resolution

```typescript
// Hàm resolve booking mode cho 1 room
function resolveBookingMode(
  room: Room,
  globalConfig: BookingConfig
): { mode: 'external' | 'modal'; url?: string } {
  // 1. Room-level override
  if (room.bookingMode === 'external') {
    return {
      mode: 'external',
      url: room.bookingUrl || globalConfig.externalUrl
    };
  }
  if (room.bookingMode === 'modal') {
    return { mode: 'modal' };
  }

  // 2. Global config
  if (globalConfig.globalMode === 'external') {
    return {
      mode: 'external',
      url: room.bookingUrl || globalConfig.externalUrl
    };
  }
  if (globalConfig.globalMode === 'modal') {
    return { mode: 'modal' };
  }

  // 3. Default: external
  return {
    mode: 'external',
    url: globalConfig.externalUrl || '#'
  };
}
```

### 5.3 Booking Config trong Room Detail API

Mở rộng response `/api/v1/rooms/:slug` để include booking info:

```json
{
  "id": "uuid",
  "name": "Deluxe Twin Room",
  "...": "...",
  "booking": {
    "mode": "modal",
    "external_url": null,
    "cta_label": "Book Now",
    "cta_secondary_label": "Check Availability"
  }
}
```

---

## 6. Validation Rules

### 6.1 Room Validation (Zod)

```typescript
const roomSchema = z.object({
  name: z.string().min(3).max(200),
  slug: z.string().regex(/^[a-z0-9-]+$/).min(3).max(100),
  room_type: z.enum(['standard', 'superior', 'deluxe', 'suite', 'villa']),
  area_sqm: z.number().positive().max(500).optional(),
  max_adults: z.number().int().min(1).max(20).default(2),
  max_children: z.number().int().min(0).max(10).default(1),
  bed_type: z.string().max(50).optional(),
  description: z.string().max(10000).optional(),
  short_desc: z.string().max(500).optional(),
  price_from: z.number().positive().optional(),
  currency: z.enum(['VND', 'USD', 'EUR']).default('VND'),
  view_type: z.string().max(50).optional(),
  highlights: z.array(z.string().max(100)).max(20).default([]),
  booking_url: z.string().url().optional().nullable(),
  booking_mode: z.enum(['external', 'modal']).optional().nullable(),
  seo_title: z.string().max(70).optional(),
  seo_description: z.string().max(160).optional(),
  is_published: z.boolean().default(false),
  amenity_ids: z.array(z.string().uuid()).default([]),
  scene_ids: z.array(z.string()).default([]),
});
```

### 6.2 Booking Request Validation

```typescript
const bookingRequestSchema = z.object({
  room_id: z.string().uuid().optional(),
  service_id: z.string().uuid().optional(),
  full_name: z.string().min(2).max(200),
  email: z.string().email(),
  phone: z.string().min(8).max(20),
  check_in: z.string().regex(/^\d{4}-\d{2}-\d{2}$/),
  check_out: z.string().regex(/^\d{4}-\d{2}-\d{2}$/),
  adults: z.number().int().min(1).max(20).default(2),
  children: z.number().int().min(0).max(10).default(0),
  special_requests: z.string().max(2000).optional(),
  source_page: z.string().max(200).optional(),
  source_scene: z.string().max(100).optional(),
}).refine(data => {
  if (data.check_in && data.check_out) {
    return new Date(data.check_out) > new Date(data.check_in);
  }
  return true;
}, { message: "Check-out must be after check-in" });
```

### 6.3 Panorama Upload Validation

```typescript
const panoramaUploadSchema = z.object({
  file: z.custom<File>()
    .refine(f => ['image/jpeg', 'image/png'].includes(f.type), 'Must be JPEG or PNG')
    .refine(f => f.size <= 50 * 1024 * 1024, 'Max 50MB'),
  // Server-side: check aspect ratio ~ 2:1
});
```

---

## 7. Sequence Diagrams

### 7.1 Booking Modal — Guest Submission Flow

```
Guest             Website Frontend     API Server          Database        Email Service
  │                     │                │                  │                  │
  │  click "Book Now"   │                │                  │                  │
  │  (room page)        │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │                │                  │                  │
  │                     │ check booking  │                  │                  │
  │                     │ mode for room  │                  │                  │
  │                     │                │                  │                  │
  │                     │ mode = "modal" │                  │                  │
  │                     │ → open modal   │                  │                  │
  │                     │ overlay        │                  │                  │
  │                     │                │                  │                  │
  │  ◄── modal opens ───┤               │                  │                  │
  │  (form with         │                │                  │                  │
  │   dynamic fields)   │                │                  │                  │
  │                     │                │                  │                  │
  │  fill form +        │                │                  │                  │
  │  submit             │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ client-side    │                  │                  │
  │                     │ validate (Zod) │                  │                  │
  │                     │                │                  │                  │
  │                     │ POST /booking/ │                  │                  │
  │                     │ request        │                  │                  │
  │                     ├───────────────►│                  │                  │
  │                     │                │ server validate  │                  │
  │                     │                │ rate limit check │                  │
  │                     │                │ honeypot check   │                  │
  │                     │                │                  │                  │
  │                     │                │ INSERT booking   │                  │
  │                     │                │ request ─────────►                  │
  │                     │                │                  │ booking_requests │
  │                     │                │                  │                  │
  │                     │                │ send hotel ──────────────────────────►
  │                     │                │ notification     │                  │ Email to
  │                     │                │                  │                  │ notify_emails
  │                     │                │                  │                  │
  │                     │                │ send guest ──────────────────────────►
  │                     │                │ auto-reply       │                  │ Confirmation
  │                     │                │                  │                  │ to guest
  │                     │                │                  │                  │
  │                     │                │ send webhook ────►                  │
  │                     │                │ (optional)       │ Slack/Zalo/etc.  │
  │                     │                │                  │                  │
  │                     │ ◄── 201 ───────┤                  │                  │
  │                     │                │                  │                  │
  │  ◄── success msg ───┤               │                  │                  │
  │  "Thank you! We     │                │                  │                  │
  │   will contact you  │                │                  │                  │
  │   within 24 hours." │                │                  │                  │
```

### 7.2 Admin — Manage Booking Requests

```
Admin              Booking Requests    API Server          Database           Actions
  │                     │                │                  │                  │
  │  /admin/booking/    │                │                  │                  │
  │  requests           │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ GET /admin/    │                  │                  │
  │                     │ booking/       │                  │                  │
  │                     │ requests       │                  │                  │
  │                     ├───────────────►│                  │                  │
  │                     │                │ SELECT * FROM    │                  │
  │                     │                │ booking_requests │                  │
  │                     │                │ WHERE hotel_id=  │                  │
  │                     │                │ ORDER BY created │                  │
  │                     │                │ DESC             │                  │
  │                     │                ├─────────────────►│                  │
  │                     │                │ ◄── rows ────────┤                  │
  │                     │ ◄── list ──────┤                  │                  │
  │  ◄── table view ────┤               │                  │                  │
  │  (name, email,      │                │                  │                  │
  │   room, dates,      │                │                  │                  │
  │   status, actions)  │                │                  │                  │
  │                     │                │                  │                  │
  │  click request row  │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ show detail    │                  │                  │
  │  ◄── detail modal ──┤               │                  │                  │
  │                     │                │                  │                  │
  │  change status:     │                │                  │                  │
  │  "new" → "contacted"│                │                  │                  │
  │  add notes          │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ PUT /admin/    │                  │                  │
  │                     │ booking/       │                  │                  │
  │                     │ requests/:id   │                  │                  │
  │                     ├───────────────►│                  │                  │
  │                     │                │ UPDATE status,   │                  │
  │                     │                │ notes            │                  │
  │                     │                ├─────────────────►│                  │
  │                     │                │                  │                  │
  │                     │                │ log audit ───────►                  │
  │                     │                │                  │ audit_logs       │
  │                     │ ◄── 200 ───────┤                  │                  │
  │  ◄── updated ───────┤               │                  │                  │
  │  toast: "Status     │                │                  │                  │
  │   updated"          │                │                  │                  │
```

### 7.3 Booking Mode Resolution — Frontend

```
Frontend            Room API            BookingConfig       Resolve Logic      UI Render
  │                     │                │                  │                  │
  │  load room page     │                │                  │                  │
  ├────────────────────►│                │                  │                  │
  │                     │ GET /rooms/    │                  │                  │
  │                     │ deluxe-ocean   │                  │                  │
  │                     │ (includes      │                  │                  │
  │                     │  booking info) │                  │                  │
  │                     │                │                  │                  │
  │  ◄── room data ─────┤               │                  │                  │
  │  (booking.mode =    │                │                  │                  │
  │   "modal")          │                │                  │                  │
  │                     │                │                  │                  │
  │  resolve CTA ──────────────────────────────────────────►│                  │
  │                     │                │                  │                  │
  │                     │                │                  │ room.booking_mode│
  │                     │                │                  │ = null           │
  │                     │                │                  │ → check global   │
  │                     │                │                  │                  │
  │                     │                │ global_mode =    │                  │
  │                     │                │ "modal"          │                  │
  │                     │                │                  │                  │
  │                     │                │                  │ result = "modal" │
  │  ◄── mode resolved ────────────────────────────────────┤                  │
  │                     │                │                  │                  │
  │  render CTA ────────────────────────────────────────────────────────────────►
  │                     │                │                  │                  │
  │                     │                │                  │                  │ mode = "modal":
  │                     │                │                  │                  │ [Book Now] →
  │                     │                │                  │                  │ opens modal
  │                     │                │                  │                  │
  │                     │                │                  │                  │ mode = "external":
  │                     │                │                  │                  │ [Book Now] →
  │                     │                │                  │                  │ window.open()
```

---

*Tài liệu tiếp theo: **03-DASHBOARD-UI-UX.md** — Giao diện admin dashboard, component specs, layout*
