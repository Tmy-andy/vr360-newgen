# 05 — DEMO HTML SPECIFICATION (Corrected)
## Website 360 Thế Hệ Mới — Trang Demo Thuần HTML

**Version:** 2.0  
**Date:** 2026-03-10  

---

## 1. Concept Chính Xác

> **VR360 KHÔNG PHẢI tính năng phụ. VR360 CHÍNH LÀ website.**

Khi khách truy cập vào website, họ NGAY LẬP TỨC đứng trong không gian 360 của khách sạn. Nền trang web là panorama 360 tương tác được. Mọi thành phần website (menu, thông tin, booking, điều hướng) đều là overlay trên nền 360.

**KHÔNG CÓ trang listing truyền thống.** Toàn bộ website là MỘT trải nghiệm 360 liền mạch.

### So sánh

| Website truyền thống | Website 360 thế hệ mới |
|---------------------|----------------------|
| Homepage → Room listing → Room detail → 360 viewer | Vào web = đứng trong lobby 360 → click "Rooms" = chuyển scene 360 sang phòng → panel info overlay |
| 360 là button phụ để bấm vào xem | 360 là NỀN CỦA TOÀN BỘ WEBSITE |
| Trang có ảnh tĩnh + text | Trang có panorama 360 tương tác + text overlay |

---

## 2. Luồng Hoạt Động

```
[Truy cập website]
    │
    ▼
[Loading screen: "Đang tải không gian..."]
    │
    ▼
[Scene: LOBBY 360°]   ← Đây là "homepage"
│  - Nền: panorama 360 lobby khách sạn
│  - Overlay: logo, menu, hotel info panel (bên phải)
│  - Hotspot trong 360: "Phòng nghỉ →", "Nhà hàng →", "Vịnh Nha Trang (info)"
│  - User có thể: xoay 360, click hotspot, click menu
    │
    ├── Click menu "Rooms" hoặc hotspot "Phòng nghỉ"
    │       ▼
    │   [Scene: ROOMS 360°]   ← Scene 360 khác (overview phòng)
    │   │  - Nền: panorama 360 hành lang phòng / phòng mẫu
    │   │  - Panel phải: danh sách hạng phòng (thumb list)
    │   │  - Click một phòng → chuyển scene 360
    │       │
    │       ├── Click "Deluxe Twin Ocean View"
    │       │       ▼
    │       │   [Scene: DELUXE BEDROOM 360°]
    │       │   │  - Nền: panorama 360 phòng Deluxe
    │       │   │  - Panel: tên, mô tả, tiện nghi, giá, Book Now
    │       │   │  - Hotspot: "Ban công →", "Ocean View (info)", "Bathtub (info)"
    │       │   │  - Scene nav bar: [Bedroom] [Balcony]
    │       │       │
    │       │       ├── Click "Ban công →"
    │       │       │       ▼
    │       │       │   [Scene: DELUXE BALCONY 360°]
    │       │       │   - Nền: panorama 360 ban công
    │       │       │   - Panel: vẫn là Deluxe Twin info
    │       │       │   - Hotspot: "← Phòng ngủ", "Sunset Point"
    │       │       │
    │       │       └── Click "Book Now" → Mở booking engine
    │       │
    │       ├── Click "Suite Ocean View" → [Scene: SUITE 360°]
    │       └── Click "Pacific Double" → [Scene: PACIFIC 360°]
    │
    ├── Click menu "Dining"
    │       ▼
    │   [Scene: DINING 360°]
    │   - Nền: panorama 360 nhà hàng
    │   - Panel: danh sách nhà hàng
    │   - Hotspot: "← Lobby", "Tiện ích →"
    │
    ├── Click menu "Recreation"
    │       ▼
    │   [Scene: RECREATION 360°]
    │   - Nền: panorama 360 pool/spa area
    │   - Panel: danh sách tiện ích
    │
    └── Bất kỳ lúc nào: drag/touch panorama
            ▼
        [Smart UI]: Ẩn overlay, chỉ còn 360 thuần
        [Sau 5s idle]: Overlay hiện lại
        [Click nút 👁]: Toggle ẩn/hiện UI thủ công
```

---

## 3. Kiến Trúc UI

```
┌──────────────────────────────────────────────────────┐
│ Z-INDEX STACK (toàn bộ trên 1 viewport, không scroll) │
│                                                        │
│  z:0    PANORAMA 360 (fixed, full viewport)           │
│         ↕ User drag/touch/zoom tương tác trực tiếp    │
│                                                        │
│  z:10   HOTSPOTS (render bởi Pannellum trong scene)   │
│         ↕ Navigate hotspot → chuyển scene              │
│         ↕ Info hotspot → mở info card                  │
│                                                        │
│  z:20   CONTENT PANEL (fixed right / bottom sheet)    │
│         ↕ Nội dung thay đổi theo section hiện tại     │
│         ↕ Welcome / Room list / Room detail / Service  │
│                                                        │
│  z:30   SCENE NAV BAR (bottom center)                 │
│         ↕ Sub-scene tabs: [Bedroom] [Balcony]          │
│                                                        │
│  z:40   TOP BAR (logo + nav + booking CTA)            │
│         ↕ Transparent, gradient fade trên nền 360      │
│                                                        │
│  z:50   INFO CARDS, MODALS                            │
│                                                        │
│  z:100  MOBILE MENU OVERLAY                           │
│                                                        │
│  z:999  LOADING SCREEN (ẩn sau khi panorama loaded)   │
└──────────────────────────────────────────────────────┘
```

---

## 4. Data Structure (JSON)

Mỗi "section" = 1 scene 360, đồng thời là 1 "trang" của website:

```javascript
{
  id: "room-deluxe",          // Unique ID
  navLabel: null,              // null = không hiện trên main nav (sub-page)
  panorama: "url.jpg",         // Ảnh equirectangular
  yaw: -20, pitch: 0, hfov: 100, autoRotate: 0.4,  // Camera defaults
  
  panelType: "room-detail",    // Loại panel content
  badge: "Deluxe",
  title: "Deluxe Twin Room",
  subtitle: "Ocean View",
  description: "...",
  
  // Room-specific fields
  area: 35, maxAdults: 2, bedType: "2 Twin Beds",
  view: "Ocean", floor: "8-15",
  amenities: ["Ocean View", "Balcony", ...],
  price: "2,500,000",
  
  // Sub-scenes (Bedroom ↔ Balcony)
  subScenes: [
    {id: "room-deluxe", label: "Bedroom"},
    {id: "room-deluxe-balcony", label: "Balcony"}
  ],
  
  // Hotspots IN the 360 scene
  hotspots: [
    {pitch:-6, yaw:140, type:"nav", label:"Ban công →", target:"room-deluxe-balcony"},
    {pitch:8, yaw:-50, type:"info", label:"Ocean View", desc:"Tầm nhìn 180°..."},
  ]
}
```

### Panel Types:

| panelType | Hiển thị |
|-----------|---------|
| `welcome` | Giới thiệu KS, highlights, CTA "Khám phá phòng" |
| `room-listing` | Danh sách hạng phòng (clickable → chuyển scene) |
| `room-detail` | Thông tin phòng: mô tả, tiện nghi, giá, Book Now |
| `service` | Danh sách dịch vụ: nhà hàng / tiện ích |

---

## 5. Smart UI (Tương Tác Thông Minh)

```
STATE MACHINE:

[UI VISIBLE] ──user drag/touch 360──→ [UI HIDDEN]
      ↑                                     │
      │                                     │
      └───────idle 5 giây─────────────────┘
      
[UI HIDDEN] ──click nút 👁──→ [UI VISIBLE]
[UI VISIBLE] ──click nút 👁──→ [UI HIDDEN] (manual, không auto-show)
```

Khi UI hidden: chỉ còn panorama 360 thuần + 1 nút nhỏ góc phải dưới để restore UI.

---

## 6. Responsive

**Desktop (>860px):** Panel bên phải (380px), topbar đầy đủ, scene nav bottom center.

**Mobile (≤860px):** Panel thành bottom sheet (kéo lên/xuống), hamburger menu, scene nav nổi phía trên bottom sheet.

---

## 7. Deployment

File demo chỉ 1 file HTML duy nhất (inline CSS + JS + data). Mở trực tiếp trên browser hoặc deploy lên Netlify/Vercel.

---

*Xem file demo thực tế: **demo.html***
