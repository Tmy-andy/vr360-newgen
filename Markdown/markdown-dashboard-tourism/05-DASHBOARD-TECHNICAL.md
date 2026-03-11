# 05 — DASHBOARD TECHNICAL IMPLEMENTATION
## Tourism Map 360° — Admin Dashboard

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** `01-DASHBOARD-ARCHITECTURE` · `02-DASHBOARD-DATABASE-API` · `03-DASHBOARD-UI-UX` · `04-DASHBOARD-FEATURES`

---

## Mục Lục

1. [Project Structure](#1-project-structure)
2. [Next.js App Router Configuration](#2-nextjs-app-router-configuration)
3. [Component Architecture](#3-component-architecture)
4. [State Management](#4-state-management)
5. [API Implementation](#5-api-implementation)
6. [Form Handling & Validation](#6-form-handling--validation)
7. [Map Integration (Leaflet)](#7-map-integration-leaflet)
8. [Panorama Integration (Pannellum)](#8-panorama-integration-pannellum)
9. [File Upload Pipeline](#9-file-upload-pipeline)
10. [Authentication & Authorization](#10-authentication--authorization)
11. [Data Export System](#11-data-export-system)
12. [Deployment & DevOps](#12-deployment--devops)

---

## 1. Project Structure

### 1.1 Directory Layout

```
tourism-dashboard/
├── prisma/
│   ├── schema.prisma                  ← DB schema (from 02-spec)
│   ├── seed.ts                        ← Seed categories, demo data
│   └── migrations/
│
├── src/
│   ├── app/
│   │   ├── layout.tsx                 ← Root layout
│   │   ├── page.tsx                   ← Redirect to /admin
│   │   │
│   │   ├── admin/
│   │   │   ├── layout.tsx             ← Admin layout (sidebar + header)
│   │   │   ├── page.tsx               ← Dashboard home
│   │   │   │
│   │   │   ├── provinces/
│   │   │   │   ├── page.tsx           ← Province list
│   │   │   │   ├── new/page.tsx       ← Create province
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx       ← Edit province (tabbed)
│   │   │   │
│   │   │   ├── categories/
│   │   │   │   └── page.tsx           ← Category list + inline edit
│   │   │   │
│   │   │   ├── pois/
│   │   │   │   ├── page.tsx           ← POI list (DataTable)
│   │   │   │   ├── new/page.tsx       ← Create POI
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx       ← Edit POI (tabbed)
│   │   │   │
│   │   │   ├── tags/
│   │   │   │   └── page.tsx           ← Tag management
│   │   │   │
│   │   │   ├── map-config/
│   │   │   │   └── page.tsx           ← Map configuration
│   │   │   │
│   │   │   ├── users/
│   │   │   │   └── page.tsx           ← User management
│   │   │   │
│   │   │   ├── analytics/
│   │   │   │   └── page.tsx           ← Analytics dashboard
│   │   │   │
│   │   │   └── settings/
│   │   │       └── page.tsx           ← General settings
│   │   │
│   │   ├── api/
│   │   │   ├── auth/
│   │   │   │   ├── [...nextauth]/route.ts
│   │   │   │   └── password/route.ts
│   │   │   │
│   │   │   ├── admin/
│   │   │   │   ├── provinces/
│   │   │   │   │   ├── route.ts       ← GET (list), POST (create)
│   │   │   │   │   └── [id]/
│   │   │   │   │       ├── route.ts   ← GET, PATCH, DELETE
│   │   │   │   │       └── publish/route.ts
│   │   │   │   │
│   │   │   │   ├── categories/
│   │   │   │   │   ├── route.ts
│   │   │   │   │   ├── [id]/route.ts
│   │   │   │   │   └── reorder/route.ts
│   │   │   │   │
│   │   │   │   ├── pois/
│   │   │   │   │   ├── route.ts
│   │   │   │   │   ├── [id]/
│   │   │   │   │   │   ├── route.ts
│   │   │   │   │   │   ├── publish/route.ts
│   │   │   │   │   │   ├── scenes/
│   │   │   │   │   │   │   ├── route.ts
│   │   │   │   │   │   │   ├── [sceneId]/route.ts
│   │   │   │   │   │   │   └── reorder/route.ts
│   │   │   │   │   │   └── gallery/
│   │   │   │   │   │       ├── route.ts
│   │   │   │   │   │       ├── [galleryId]/route.ts
│   │   │   │   │   │       └── reorder/route.ts
│   │   │   │   │   └── reorder/route.ts
│   │   │   │   │
│   │   │   │   ├── tags/
│   │   │   │   │   ├── route.ts
│   │   │   │   │   └── [id]/route.ts
│   │   │   │   │
│   │   │   │   ├── map-config/
│   │   │   │   │   └── [provinceId]/route.ts
│   │   │   │   │
│   │   │   │   ├── users/
│   │   │   │   │   ├── route.ts
│   │   │   │   │   └── [id]/route.ts
│   │   │   │   │
│   │   │   │   ├── analytics/
│   │   │   │   │   ├── overview/route.ts
│   │   │   │   │   ├── top-pois/route.ts
│   │   │   │   │   ├── categories/route.ts
│   │   │   │   │   ├── devices/route.ts
│   │   │   │   │   └── timeline/route.ts
│   │   │   │   │
│   │   │   │   ├── upload/
│   │   │   │   │   ├── panorama/route.ts
│   │   │   │   │   ├── thumbnail/route.ts
│   │   │   │   │   ├── gallery/route.ts
│   │   │   │   │   └── [key]/route.ts
│   │   │   │   │
│   │   │   │   └── publish/
│   │   │   │       └── [provinceId]/route.ts
│   │   │   │
│   │   │   └── public/
│   │   │       ├── provinces/route.ts
│   │   │       ├── categories/route.ts
│   │   │       └── pois/
│   │   │           └── [id]/route.ts
│   │   │
│   │   └── login/
│   │       └── page.tsx               ← Login page
│   │
│   ├── components/
│   │   ├── ui/                        ← shadcn/ui components
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── select.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── table.tsx
│   │   │   ├── card.tsx
│   │   │   ├── badge.tsx
│   │   │   ├── tabs.tsx
│   │   │   ├── switch.tsx
│   │   │   ├── toast.tsx
│   │   │   ├── skeleton.tsx
│   │   │   ├── dropdown-menu.tsx
│   │   │   ├── command.tsx
│   │   │   ├── pagination.tsx
│   │   │   └── sheet.tsx
│   │   │
│   │   ├── layout/
│   │   │   ├── AdminSidebar.tsx       ← Sidebar navigation
│   │   │   ├── AdminHeader.tsx        ← Top header (breadcrumb, user)
│   │   │   ├── MobileMenu.tsx         ← Mobile sidebar overlay
│   │   │   └── Breadcrumb.tsx         ← Dynamic breadcrumb
│   │   │
│   │   ├── shared/
│   │   │   ├── DataTable.tsx          ← Reusable data table
│   │   │   ├── ConfirmDialog.tsx      ← Delete confirmation
│   │   │   ├── EmptyState.tsx         ← No data illustration
│   │   │   ├── LoadingSkeleton.tsx    ← Page loading skeleton
│   │   │   ├── PageHeader.tsx         ← Page title + actions
│   │   │   ├── StatusBadge.tsx        ← Published/Draft badge
│   │   │   └── SearchInput.tsx        ← Debounced search input
│   │   │
│   │   ├── editors/
│   │   │   ├── MapPicker.tsx          ← Leaflet map coordinate picker
│   │   │   ├── PanoramaPreview.tsx    ← Pannellum 360° preview
│   │   │   ├── HotspotEditor.tsx      ← Hotspot placement on overview
│   │   │   ├── IconPicker.tsx         ← Lucide icon selector
│   │   │   ├── ColorPicker.tsx        ← Hex color picker
│   │   │   ├── EmojiPicker.tsx        ← Emoji selector
│   │   │   └── GalleryUploader.tsx    ← Multi-image upload + reorder
│   │   │
│   │   ├── forms/
│   │   │   ├── ProvinceForm.tsx       ← Province create/edit form
│   │   │   ├── CategoryForm.tsx       ← Category inline edit form
│   │   │   ├── PoiForm.tsx            ← POI create/edit form (tabbed)
│   │   │   ├── TagForm.tsx            ← Tag create/edit
│   │   │   ├── MapConfigForm.tsx      ← Map configuration form
│   │   │   ├── UserForm.tsx           ← User create/edit
│   │   │   └── SettingsForm.tsx       ← General settings
│   │   │
│   │   ├── dashboard/
│   │   │   ├── StatsCard.tsx          ← Number stat card
│   │   │   ├── ViewsChart.tsx         ← Line chart (views over time)
│   │   │   ├── TopPoisChart.tsx       ← Bar chart (top POIs)
│   │   │   ├── DevicesChart.tsx       ← Donut chart (devices)
│   │   │   ├── CategoriesChart.tsx    ← Donut chart (categories)
│   │   │   ├── RecentActivity.tsx     ← Activity log list
│   │   │   └── QuickActions.tsx       ← Quick action buttons
│   │   │
│   │   └── pois/
│   │       ├── PoiTable.tsx           ← POI DataTable with filters
│   │       ├── PoiFilters.tsx         ← Province/category/status filters
│   │       ├── PoiBulkActions.tsx     ← Bulk actions toolbar
│   │       └── TagMultiSelect.tsx     ← Tag multi-select input
│   │
│   ├── hooks/
│   │   ├── useProvinces.ts            ← React Query: provinces CRUD
│   │   ├── useCategories.ts           ← React Query: categories CRUD
│   │   ├── usePois.ts                 ← React Query: POIs CRUD
│   │   ├── useTags.ts                 ← React Query: tags CRUD
│   │   ├── useMapConfig.ts            ← React Query: map config
│   │   ├── useUsers.ts               ← React Query: users CRUD
│   │   ├── useAnalytics.ts           ← React Query: analytics data
│   │   ├── useUpload.ts              ← File upload hook
│   │   ├── useExport.ts              ← Export/publish hook
│   │   ├── useDebounce.ts            ← Debounce helper
│   │   └── useAuth.ts                ← Auth state & permissions
│   │
│   ├── stores/
│   │   ├── uiStore.ts                 ← Sidebar, theme, mobile menu
│   │   └── filterStore.ts            ← POI list filter state
│   │
│   ├── lib/
│   │   ├── prisma.ts                  ← Prisma client singleton
│   │   ├── auth.ts                    ← NextAuth config
│   │   ├── s3.ts                      ← S3/R2 client
│   │   ├── upload.ts                  ← Upload utilities
│   │   ├── export.ts                  ← Export JSON generator
│   │   ├── slug.ts                    ← Slug generation utility
│   │   ├── permissions.ts            ← RBAC permission checks
│   │   ├── api-response.ts           ← Standard API response helpers
│   │   └── validations/
│   │       ├── province.ts            ← Zod schemas for province
│   │       ├── category.ts
│   │       ├── poi.ts
│   │       ├── tag.ts
│   │       ├── user.ts
│   │       └── map-config.ts
│   │
│   ├── types/
│   │   ├── province.ts
│   │   ├── category.ts
│   │   ├── poi.ts
│   │   ├── tag.ts
│   │   ├── user.ts
│   │   ├── map-config.ts
│   │   ├── analytics.ts
│   │   └── api.ts                     ← Generic API response types
│   │
│   └── styles/
│       └── globals.css                ← Tailwind + custom CSS variables
│
├── public/
│   ├── images/
│   │   └── logo.svg
│   └── favicon.ico
│
├── .env.local                         ← Environment variables
├── .env.example
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── package.json
├── docker-compose.yml                 ← PostgreSQL dev container
└── README.md
```

---

## 2. Next.js App Router Configuration

### 2.1 Root Layout

```typescript
// src/app/layout.tsx
import { Inter } from 'next/font/google';
import './styles/globals.css';

const inter = Inter({ subsets: ['latin', 'vietnamese'] });

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="vi">
      <body className={inter.className}>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

### 2.2 Admin Layout

```typescript
// src/app/admin/layout.tsx
import { getServerSession } from 'next-auth';
import { redirect } from 'next/navigation';
import { AdminSidebar } from '@/components/layout/AdminSidebar';
import { AdminHeader } from '@/components/layout/AdminHeader';

export default async function AdminLayout({ children }: { children: React.ReactNode }) {
  const session = await getServerSession(authConfig);
  if (!session) redirect('/login');

  return (
    <div className="flex h-screen bg-slate-50">
      <AdminSidebar user={session.user} />
      <div className="flex-1 flex flex-col overflow-hidden">
        <AdminHeader user={session.user} />
        <main className="flex-1 overflow-y-auto p-6">
          {children}
        </main>
      </div>
    </div>
  );
}
```

### 2.3 Middleware (Auth Guard)

```typescript
// middleware.ts
import { withAuth } from 'next-auth/middleware';

export default withAuth({
  callbacks: {
    authorized: ({ token, req }) => {
      if (req.nextUrl.pathname.startsWith('/admin')) {
        return !!token;
      }
      if (req.nextUrl.pathname.startsWith('/api/admin')) {
        return !!token;
      }
      return true;
    },
  },
});

export const config = {
  matcher: ['/admin/:path*', '/api/admin/:path*'],
};
```

---

## 3. Component Architecture

### 3.1 Component Tree

```
<RootLayout>
└── <Providers> (QueryClient, NextAuth, Zustand, Theme)
    └── <AdminLayout>
        ├── <AdminSidebar>
        │   ├── <SidebarLogo>
        │   ├── <SidebarNav>
        │   │   ├── <SidebarSection label="NỘI DUNG">
        │   │   │   ├── <SidebarItem icon="building" href="/admin/provinces" />
        │   │   │   ├── <SidebarItem icon="folder" href="/admin/categories" />
        │   │   │   ├── <SidebarItem icon="map-pin" href="/admin/pois" badge={3} />
        │   │   │   └── <SidebarItem icon="tag" href="/admin/tags" />
        │   │   ├── <SidebarSection label="TRỰC QUAN">
        │   │   │   ├── <SidebarItem icon="map" href="/admin/map-config" />
        │   │   │   └── <SidebarItem icon="image" href="/admin/gallery" />
        │   │   └── <SidebarSection label="HỆ THỐNG">
        │   │       ├── <SidebarItem icon="users" href="/admin/users" />
        │   │       ├── <SidebarItem icon="settings" href="/admin/settings" />
        │   │       └── <SidebarItem icon="bar-chart-2" href="/admin/analytics" />
        │   └── <SidebarFooter> (user info, collapse toggle)
        │
        ├── <AdminHeader>
        │   ├── <MobileMenuToggle>
        │   ├── <Breadcrumb>
        │   ├── <ThemeToggle>
        │   ├── <NotificationBell>
        │   └── <UserMenu> (profile, logout)
        │
        └── <main>  ← Page content changes per route
            └── (page.tsx content)
```

### 3.2 POI Edit Component Tree (Most Complex Page)

```
<PoiEditPage>
├── <PageHeader title="Bãi Trước" backHref="/admin/pois" />
├── <Tabs>
│   ├── <TabContent value="basic">
│   │   └── <PoiBasicForm>
│   │       ├── <Input name="name" />
│   │       ├── <Input name="slug" />
│   │       ├── <ProvinceSelect />
│   │       ├── <CategorySelect />         ← Color dot + name
│   │       ├── <EmojiPicker />
│   │       ├── <Input name="addr" />
│   │       ├── <Textarea name="description" />
│   │       ├── <Input name="hours" />
│   │       ├── <Input name="distance" />
│   │       ├── <TagMultiSelect />          ← Multi-select + create
│   │       └── <Switch name="isPublished" />
│   │
│   ├── <TabContent value="location">
│   │   └── <MapPicker>                     ← Leaflet interactive
│   │       ├── <LeafletMap />
│   │       ├── <Input name="lat" />
│   │       ├── <Input name="lng" />
│   │       └── <GeoSearchInput />
│   │
│   ├── <TabContent value="panorama">
│   │   └── <PanoramaSection>
│   │       ├── <FileUpload accept="image/*" />
│   │       ├── <PanoramaPreview>            ← Pannellum viewer
│   │       ├── <Input name="sceneYaw" />
│   │       ├── <Input name="scenePitch" />
│   │       ├── <Input name="sceneHfov" />
│   │       ├── <CaptureButton />
│   │       └── <ThumbnailUpload />
│   │
│   ├── <TabContent value="hotspot">
│   │   └── <HotspotEditor>                 ← Overview panorama
│   │       ├── <PannellumViewer />          ← Province overview
│   │       ├── <HotspotMarker />
│   │       ├── <Input name="hotspotPitch" />
│   │       └── <Input name="hotspotYaw" />
│   │
│   └── <TabContent value="gallery">
│       └── <GalleryUploader>
│           ├── <DropZone />
│           ├── <UploadProgress />
│           └── <SortableGalleryGrid>
│               └── <GalleryItem /> × N
│
└── <FormActions>
    ├── <Button>💾 Lưu</Button>
    ├── <Button variant="outline">Preview ↗</Button>
    └── <Button variant="destructive">🗑 Xoá</Button>
```

---

## 4. State Management

### 4.1 Zustand Stores

#### UI Store

```typescript
// src/stores/uiStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface UIState {
  sidebarCollapsed: boolean;
  mobileMenuOpen: boolean;
  theme: 'light' | 'dark' | 'system';
  toggleSidebar: () => void;
  setMobileMenu: (open: boolean) => void;
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
}

export const useUIStore = create<UIState>()(
  persist(
    (set) => ({
      sidebarCollapsed: false,
      mobileMenuOpen: false,
      theme: 'light',
      toggleSidebar: () => set((s) => ({ sidebarCollapsed: !s.sidebarCollapsed })),
      setMobileMenu: (open) => set({ mobileMenuOpen: open }),
      setTheme: (theme) => set({ theme }),
    }),
    { name: 'tourism-admin-ui' }
  )
);
```

#### Filter Store

```typescript
// src/stores/filterStore.ts
import { create } from 'zustand';

interface FilterState {
  poiFilters: {
    provinceId?: number;
    categoryId?: number;
    isPublished?: boolean;
    search: string;
    page: number;
    limit: number;
    sort: string;
    order: 'asc' | 'desc';
  };
  setPoiFilter: (key: string, value: any) => void;
  resetPoiFilters: () => void;
}

const defaultFilters = {
  search: '',
  page: 1,
  limit: 20,
  sort: 'sort_order',
  order: 'asc' as const,
};

export const useFilterStore = create<FilterState>((set) => ({
  poiFilters: defaultFilters,
  setPoiFilter: (key, value) =>
    set((s) => ({
      poiFilters: { ...s.poiFilters, [key]: value, page: key !== 'page' ? 1 : value },
    })),
  resetPoiFilters: () => set({ poiFilters: defaultFilters }),
}));
```

### 4.2 React Query Hooks

```typescript
// src/hooks/usePois.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export function usePois(filters: PoiFilters) {
  return useQuery({
    queryKey: ['pois', filters],
    queryFn: () => fetchPois(filters),
    keepPreviousData: true,
  });
}

export function usePoi(id: number) {
  return useQuery({
    queryKey: ['poi', id],
    queryFn: () => fetchPoi(id),
    enabled: !!id,
  });
}

export function useCreatePoi() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: createPoi,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['pois'] });
    },
  });
}

export function useUpdatePoi() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: ({ id, data }: { id: number; data: UpdatePoiData }) =>
      updatePoi(id, data),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: ['pois'] });
      queryClient.invalidateQueries({ queryKey: ['poi', id] });
    },
  });
}

export function useDeletePoi() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: deletePoi,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['pois'] });
    },
  });
}
```

---

## 5. API Implementation

### 5.1 API Route Pattern

```typescript
// src/app/api/admin/pois/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/prisma';
import { checkPermission } from '@/lib/permissions';
import { poiCreateSchema } from '@/lib/validations/poi';
import { successResponse, errorResponse } from '@/lib/api-response';

// GET /api/admin/pois — List POIs with filters
export async function GET(req: NextRequest) {
  const session = await getServerSession(authConfig);
  if (!session) return errorResponse('UNAUTHORIZED', 401);
  if (!checkPermission(session.user.role, 'poi:read'))
    return errorResponse('FORBIDDEN', 403);

  const { searchParams } = new URL(req.url);
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '20');
  const provinceId = searchParams.get('province_id');
  const categoryId = searchParams.get('category_id');
  const search = searchParams.get('q');
  const isPublished = searchParams.get('is_published');
  const sort = searchParams.get('sort') || 'sort_order';
  const order = searchParams.get('order') || 'asc';

  const where: any = {};
  if (provinceId) where.provinceId = parseInt(provinceId);
  if (categoryId) where.categoryId = parseInt(categoryId);
  if (isPublished !== null) where.isPublished = isPublished === 'true';
  if (search) {
    where.OR = [
      { name: { contains: search, mode: 'insensitive' } },
      { addr: { contains: search, mode: 'insensitive' } },
    ];
  }

  const [pois, total] = await Promise.all([
    prisma.poi.findMany({
      where,
      include: {
        province: { select: { id: true, name: true } },
        category: { select: { id: true, name: true, icon: true, color: true } },
        tags: { include: { tag: true } },
        _count: { select: { gallery: true, scenes: true } },
      },
      orderBy: { [sort]: order },
      skip: (page - 1) * limit,
      take: limit,
    }),
    prisma.poi.count({ where }),
  ]);

  return successResponse(pois, { total, page, limit, totalPages: Math.ceil(total / limit) });
}

// POST /api/admin/pois — Create POI
export async function POST(req: NextRequest) {
  const session = await getServerSession(authConfig);
  if (!session) return errorResponse('UNAUTHORIZED', 401);
  if (!checkPermission(session.user.role, 'poi:write'))
    return errorResponse('FORBIDDEN', 403);

  const body = await req.json();
  const validation = poiCreateSchema.safeParse(body);
  if (!validation.success) {
    return errorResponse('VALIDATION_ERROR', 400, validation.error.flatten());
  }

  const { tagIds, ...poiData } = validation.data;

  const poi = await prisma.poi.create({
    data: {
      ...poiData,
      tags: tagIds?.length
        ? { create: tagIds.map((tagId: number) => ({ tagId })) }
        : undefined,
    },
    include: {
      province: true,
      category: true,
      tags: { include: { tag: true } },
    },
  });

  return successResponse(poi, undefined, 201);
}
```

### 5.2 Permission Check Helper

```typescript
// src/lib/permissions.ts
type Role = 'admin' | 'editor' | 'viewer';

const permissions: Record<string, Role[]> = {
  // Province
  'province:read': ['admin', 'editor', 'viewer'],
  'province:write': ['admin', 'editor'],
  'province:delete': ['admin'],
  'province:publish': ['admin'],

  // Category
  'category:read': ['admin', 'editor', 'viewer'],
  'category:write': ['admin', 'editor'],
  'category:delete': ['admin'],

  // POI
  'poi:read': ['admin', 'editor', 'viewer'],
  'poi:write': ['admin', 'editor'],
  'poi:delete': ['admin', 'editor'],

  // Tags
  'tag:read': ['admin', 'editor', 'viewer'],
  'tag:write': ['admin', 'editor'],
  'tag:merge': ['admin'],

  // Map config
  'map-config:read': ['admin', 'editor', 'viewer'],
  'map-config:write': ['admin'],

  // Users
  'user:read': ['admin'],
  'user:write': ['admin'],

  // Settings
  'settings:read': ['admin'],
  'settings:write': ['admin'],

  // Publish
  'publish': ['admin'],

  // Analytics
  'analytics:read': ['admin', 'editor', 'viewer'],
};

export function checkPermission(role: Role, permission: string): boolean {
  return permissions[permission]?.includes(role) ?? false;
}
```

---

## 6. Form Handling & Validation

### 6.1 Zod Schemas

```typescript
// src/lib/validations/poi.ts
import { z } from 'zod';

export const poiCreateSchema = z.object({
  name: z.string().min(2).max(200),
  slug: z.string().min(2).max(200).regex(/^[a-z0-9-]+$/),
  provinceId: z.number().int().positive(),
  categoryId: z.number().int().positive(),
  addr: z.string().max(300).optional(),
  description: z.string().max(5000).optional(),
  hours: z.string().max(100).optional(),
  distance: z.string().max(100).optional(),
  emoji: z.string().max(10).optional(),
  lat: z.number().min(-90).max(90),
  lng: z.number().min(-180).max(180),
  hotspotPitch: z.number().min(-90).max(90).default(0),
  hotspotYaw: z.number().min(-180).max(180).default(0),
  panoramaUrl: z.string().url().optional().or(z.literal('')),
  sceneYaw: z.number().min(-180).max(180).default(0),
  scenePitch: z.number().min(-90).max(90).default(0),
  sceneHfov: z.number().min(30).max(150).default(100),
  thumbUrl: z.string().url().optional().or(z.literal('')),
  isPublished: z.boolean().default(true),
  sortOrder: z.number().int().default(0),
  tagIds: z.array(z.number().int().positive()).optional(),
});

export const poiUpdateSchema = poiCreateSchema.partial();

export type PoiCreateInput = z.infer<typeof poiCreateSchema>;
export type PoiUpdateInput = z.infer<typeof poiUpdateSchema>;
```

```typescript
// src/lib/validations/category.ts
import { z } from 'zod';

export const categorySchema = z.object({
  name: z.string().min(2).max(100),
  slug: z.string().min(2).max(100).regex(/^[a-z0-9-]+$/),
  icon: z.string().min(1).max(50),    // Lucide icon name
  color: z.string().regex(/^#[0-9A-Fa-f]{6}$/),
  sortOrder: z.number().int().default(0),
  isActive: z.boolean().default(true),
});
```

### 6.2 React Hook Form Integration

```typescript
// In PoiForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { poiCreateSchema, PoiCreateInput } from '@/lib/validations/poi';

export function PoiForm({ poi, onSubmit }: Props) {
  const form = useForm<PoiCreateInput>({
    resolver: zodResolver(poiCreateSchema),
    defaultValues: poi || {
      name: '', slug: '', lat: 0, lng: 0,
      sceneYaw: 0, scenePitch: 0, sceneHfov: 100,
      hotspotPitch: 0, hotspotYaw: 0,
      isPublished: true, sortOrder: 0,
    },
  });

  // Auto-generate slug from name
  const name = form.watch('name');
  useEffect(() => {
    if (!poi) { // Only auto-generate for new POI
      form.setValue('slug', generateSlug(name));
    }
  }, [name]);

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)}>
        {/* Form fields */}
      </form>
    </Form>
  );
}
```

---

## 7. Map Integration (Leaflet)

### 7.1 MapPicker Component

```typescript
// src/components/editors/MapPicker.tsx
'use client';

import { useEffect, useRef, useState } from 'react';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';

interface MapPickerProps {
  lat: number;
  lng: number;
  zoom?: number;
  onLocationChange: (lat: number, lng: number) => void;
  existingPois?: { lat: number; lng: number; name: string }[];
}

export function MapPicker({ lat, lng, zoom = 9, onLocationChange, existingPois }: MapPickerProps) {
  const mapRef = useRef<HTMLDivElement>(null);
  const mapInstance = useRef<L.Map | null>(null);
  const markerRef = useRef<L.Marker | null>(null);

  useEffect(() => {
    if (!mapRef.current || mapInstance.current) return;

    const map = L.map(mapRef.current).setView([lat, lng], zoom);
    
    L.tileLayer(
      'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png',
      { maxZoom: 18, subdomains: 'abcd' }
    ).addTo(map);

    // Add existing POI markers (grey, small)
    existingPois?.forEach((p) => {
      L.circleMarker([p.lat, p.lng], {
        radius: 4,
        fillColor: '#94A3B8',
        color: '#fff',
        weight: 1,
        fillOpacity: 0.5,
      })
        .addTo(map)
        .bindTooltip(p.name);
    });

    // Add main marker (draggable)
    const marker = L.marker([lat, lng], { draggable: true }).addTo(map);
    marker.on('dragend', () => {
      const pos = marker.getLatLng();
      onLocationChange(pos.lat, pos.lng);
    });

    // Click to move marker
    map.on('click', (e: L.LeafletMouseEvent) => {
      marker.setLatLng(e.latlng);
      onLocationChange(e.latlng.lat, e.latlng.lng);
    });

    mapInstance.current = map;
    markerRef.current = marker;

    return () => { map.remove(); mapInstance.current = null; };
  }, []);

  // Sync marker when lat/lng props change
  useEffect(() => {
    if (markerRef.current) {
      markerRef.current.setLatLng([lat, lng]);
      mapInstance.current?.panTo([lat, lng]);
    }
  }, [lat, lng]);

  return (
    <div className="space-y-3">
      <div ref={mapRef} className="h-[400px] rounded-lg border" />
      <div className="flex gap-4">
        <Input label="Lat" type="number" value={lat} onChange={...} step="0.000001" />
        <Input label="Lng" type="number" value={lng} onChange={...} step="0.000001" />
      </div>
    </div>
  );
}
```

### 7.2 Dynamic Import (No SSR)

```typescript
// Leaflet needs browser APIs — dynamic import required
import dynamic from 'next/dynamic';

const MapPicker = dynamic(() => import('@/components/editors/MapPicker'), {
  ssr: false,
  loading: () => <Skeleton className="h-[400px]" />,
});
```

---

## 8. Panorama Integration (Pannellum)

### 8.1 PanoramaPreview Component

```typescript
// src/components/editors/PanoramaPreview.tsx
'use client';

import { useEffect, useRef, useCallback } from 'react';

interface PanoramaPreviewProps {
  panoramaUrl: string;
  yaw?: number;
  pitch?: number;
  hfov?: number;
  onCameraChange?: (yaw: number, pitch: number, hfov: number) => void;
  height?: number;
}

export function PanoramaPreview({
  panoramaUrl, yaw = 0, pitch = 0, hfov = 100,
  onCameraChange, height = 300
}: PanoramaPreviewProps) {
  const containerRef = useRef<HTMLDivElement>(null);
  const viewerRef = useRef<any>(null);

  useEffect(() => {
    if (!containerRef.current || !panoramaUrl) return;

    // Load Pannellum dynamically
    const script = document.createElement('script');
    script.src = 'https://cdn.jsdelivr.net/npm/pannellum@2.5.6/build/pannellum.js';
    script.onload = () => {
      viewerRef.current = (window as any).pannellum.viewer(containerRef.current!, {
        type: 'equirectangular',
        panorama: panoramaUrl,
        autoLoad: true,
        showControls: true,
        yaw, pitch, hfov,
        friction: 0.15,
        sceneFadeDuration: 0,
      });
    };
    document.head.appendChild(script);

    return () => {
      viewerRef.current?.destroy();
      viewerRef.current = null;
    };
  }, [panoramaUrl]);

  const handleCapture = useCallback(() => {
    if (!viewerRef.current || !onCameraChange) return;
    const y = viewerRef.current.getYaw();
    const p = viewerRef.current.getPitch();
    const h = viewerRef.current.getHfov();
    onCameraChange(
      Math.round(y * 100) / 100,
      Math.round(p * 100) / 100,
      Math.round(h * 100) / 100
    );
  }, [onCameraChange]);

  return (
    <div className="space-y-3">
      <div
        ref={containerRef}
        style={{ height }}
        className="rounded-lg border overflow-hidden"
      />
      {onCameraChange && (
        <Button variant="outline" size="sm" onClick={handleCapture}>
          📸 Capture Current View
        </Button>
      )}
    </div>
  );
}
```

### 8.2 HotspotEditor Component

```typescript
// src/components/editors/HotspotEditor.tsx
'use client';

interface HotspotEditorProps {
  overviewPanorama: string;           // Province overview panorama URL
  overviewYaw: number;
  existingHotspots: { pitch: number; yaw: number; name: string; emoji: string }[];
  currentPitch: number;
  currentYaw: number;
  onChange: (pitch: number, yaw: number) => void;
}

export function HotspotEditor({
  overviewPanorama, overviewYaw,
  existingHotspots, currentPitch, currentYaw, onChange
}: HotspotEditorProps) {
  const containerRef = useRef<HTMLDivElement>(null);
  const viewerRef = useRef<any>(null);

  useEffect(() => {
    if (!containerRef.current || !overviewPanorama) return;

    // Initialize Pannellum with overview panorama
    viewerRef.current = pannellum.viewer(containerRef.current, {
      type: 'equirectangular',
      panorama: overviewPanorama,
      autoLoad: true,
      showControls: true,
      yaw: overviewYaw,
      hfov: 120,
      hotSpots: [
        // Existing POI hotspots (semi-transparent)
        ...existingHotspots.map(h => ({
          pitch: h.pitch, yaw: h.yaw,
          type: 'custom',
          createTooltipFunc: (el: HTMLElement) => {
            el.innerHTML = `<div class="opacity-40">${h.emoji} ${h.name}</div>`;
          },
        })),
        // Current POI hotspot (highlighted)
        {
          pitch: currentPitch, yaw: currentYaw,
          type: 'custom',
          createTooltipFunc: (el: HTMLElement) => {
            el.innerHTML = `<div class="text-red-500 text-2xl animate-pulse">✚</div>`;
          },
        },
      ],
    });

    // Click to set hotspot position
    viewerRef.current.on('mousedown', (event: any) => {
      // Convert screen coords to pitch/yaw
      const coords = viewerRef.current.mouseEventToCoords(event);
      onChange(
        Math.round(coords[0] * 100) / 100,
        Math.round(coords[1] * 100) / 100
      );
    });

    return () => { viewerRef.current?.destroy(); };
  }, [overviewPanorama]);

  return (
    <div className="space-y-3">
      <div ref={containerRef} className="h-[400px] rounded-lg border cursor-crosshair" />
      <div className="flex gap-4">
        <Input label="Hotspot Pitch" type="number" value={currentPitch} readOnly />
        <Input label="Hotspot Yaw" type="number" value={currentYaw} readOnly />
      </div>
      <p className="text-sm text-slate-500">
        🎯 Click trên panorama để đặt vị trí hotspot
      </p>
    </div>
  );
}
```

---

## 9. File Upload Pipeline

### 9.1 Upload API Route

```typescript
// src/app/api/admin/upload/panorama/route.ts
import { NextRequest } from 'next/server';
import sharp from 'sharp';
import { uploadToS3 } from '@/lib/s3';
import { v4 as uuid } from 'uuid';

export async function POST(req: NextRequest) {
  const session = await getServerSession(authConfig);
  if (!session || !checkPermission(session.user.role, 'poi:write'))
    return errorResponse('FORBIDDEN', 403);

  const formData = await req.formData();
  const file = formData.get('file') as File;
  
  if (!file) return errorResponse('VALIDATION_ERROR', 400, 'No file provided');
  if (file.size > 30 * 1024 * 1024) return errorResponse('UPLOAD_ERROR', 413, 'File too large (max 30MB)');

  const buffer = Buffer.from(await file.arrayBuffer());
  const key = `panoramas/${uuid()}`;

  // Process with Sharp
  const original = await sharp(buffer)
    .resize(8192, null, { withoutEnlargement: true })
    .jpeg({ quality: 85 })
    .toBuffer();

  const thumbnail = await sharp(buffer)
    .resize(400, 200, { fit: 'cover' })
    .jpeg({ quality: 80 })
    .toBuffer();

  // Upload to S3/R2
  const [panoramaUrl, thumbUrl] = await Promise.all([
    uploadToS3(`${key}.jpg`, original, 'image/jpeg'),
    uploadToS3(`${key}_thumb.jpg`, thumbnail, 'image/jpeg'),
  ]);

  return successResponse({ panoramaUrl, thumbUrl });
}
```

### 9.2 Client Upload Hook

```typescript
// src/hooks/useUpload.ts
import { useState } from 'react';

export function useUpload() {
  const [uploading, setUploading] = useState(false);
  const [progress, setProgress] = useState(0);

  const upload = async (file: File, type: 'panorama' | 'thumbnail' | 'gallery') => {
    setUploading(true);
    setProgress(0);

    const formData = new FormData();
    formData.append('file', file);

    try {
      const response = await fetch(`/api/admin/upload/${type}`, {
        method: 'POST',
        body: formData,
      });

      if (!response.ok) throw new Error('Upload failed');
      const result = await response.json();
      setProgress(100);
      return result.data;
    } finally {
      setUploading(false);
    }
  };

  return { upload, uploading, progress };
}
```

---

## 10. Authentication & Authorization

### 10.1 NextAuth Configuration

```typescript
// src/lib/auth.ts
import NextAuth from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import bcrypt from 'bcryptjs';
import { prisma } from '@/lib/prisma';

export const authConfig = {
  providers: [
    CredentialsProvider({
      name: 'credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) return null;

        const user = await prisma.user.findUnique({
          where: { email: credentials.email, isActive: true },
        });

        if (!user) return null;
        
        const valid = await bcrypt.compare(credentials.password, user.passwordHash);
        if (!valid) return null;

        // Update last login
        await prisma.user.update({
          where: { id: user.id },
          data: { lastLoginAt: new Date() },
        });

        return {
          id: String(user.id),
          email: user.email,
          name: user.name,
          role: user.role,
          image: user.avatarUrl,
        };
      },
    }),
  ],
  callbacks: {
    jwt({ token, user }) {
      if (user) {
        token.role = user.role;
        token.id = user.id;
      }
      return token;
    },
    session({ session, token }) {
      if (session.user) {
        session.user.role = token.role as string;
        session.user.id = token.id as string;
      }
      return session;
    },
  },
  pages: {
    signIn: '/login',
  },
  session: {
    strategy: 'jwt',
    maxAge: 24 * 60 * 60, // 24 hours
  },
};
```

### 10.2 Login Page

```typescript
// src/app/login/page.tsx
'use client';

export default function LoginPage() {
  const [error, setError] = useState('');
  
  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    const result = await signIn('credentials', {
      email, password, redirect: false,
    });
    
    if (result?.error) setError('Email hoặc mật khẩu không đúng');
    else router.push('/admin');
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-slate-50">
      <Card className="w-full max-w-md">
        <CardHeader>
          <div className="flex justify-center mb-4">
            <div className="w-12 h-12 bg-teal-600 rounded-xl flex items-center justify-center text-white font-bold">
              360
            </div>
          </div>
          <CardTitle className="text-center">Tourism Admin</CardTitle>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit}>
            <Input label="Email" type="email" />
            <Input label="Mật khẩu" type="password" />
            {error && <p className="text-red-500 text-sm">{error}</p>}
            <Button type="submit" className="w-full bg-teal-600">
              Đăng nhập
            </Button>
          </form>
        </CardContent>
      </Card>
    </div>
  );
}
```

---

## 11. Data Export System

### 11.1 Export Service

```typescript
// src/lib/export.ts
import { prisma } from '@/lib/prisma';

export async function generateExportJSON(provinceId: number) {
  const province = await prisma.province.findUnique({
    where: { id: provinceId },
    include: {
      pois: {
        where: { isPublished: true },
        include: {
          category: true,
          tags: { include: { tag: true } },
        },
        orderBy: { sortOrder: 'asc' },
      },
      mapConfig: true,
    },
  });

  if (!province) throw new Error('Province not found');

  const categories = await prisma.category.findMany({
    where: { isActive: true },
    orderBy: { sortOrder: 'asc' },
  });

  // Transform to frontend format
  return {
    exportedAt: new Date().toISOString(),
    province: { name: province.name, slug: province.slug },
    
    CATEGORIES: categories.map(c => ({
      id: c.slug,
      label: c.name,
      icon: c.icon,
      color: c.color,
    })),
    
    OVERVIEW: {
      panorama: province.overviewPanorama,
      yaw: Number(province.overviewYaw),
      pitch: Number(province.overviewPitch),
      hfov: Number(province.overviewHfov),
      autoRotate: Number(province.overviewAutoRotate),
    },
    
    POIS: province.pois.map(p => ({
      id: p.id,
      cat: p.category.slug,
      name: p.name,
      addr: p.addr,
      lat: Number(p.lat),
      lng: Number(p.lng),
      pitch: Number(p.hotspotPitch),
      yaw: Number(p.hotspotYaw),
      panorama: p.panoramaUrl,
      pYaw: Number(p.sceneYaw),
      pPitch: Number(p.scenePitch),
      pHfov: Number(p.sceneHfov),
      thumb: p.thumbUrl,
      desc: p.description,
      tags: p.tags.map(t => t.tag.name),
      hours: p.hours,
      distance: p.distance,
      emoji: p.emoji,
    })),
    
    MAP_CONFIG: province.mapConfig ? {
      center: [Number(province.mapConfig.centerLat), Number(province.mapConfig.centerLng)],
      zoom: province.mapConfig.defaultZoom,
      minZoom: province.mapConfig.minZoom,
      maxZoom: province.mapConfig.maxZoom,
      tileProvider: province.mapConfig.tileProvider,
      tileFilterCss: province.mapConfig.tileFilterCss,
    } : undefined,
  };
}
```

### 11.2 Export API Route

```typescript
// src/app/api/admin/publish/[provinceId]/route.ts
export async function POST(req: NextRequest, { params }: { params: { provinceId: string } }) {
  const session = await getServerSession(authConfig);
  if (!checkPermission(session?.user?.role, 'publish'))
    return errorResponse('FORBIDDEN', 403);

  const provinceId = parseInt(params.provinceId);
  
  try {
    const exportData = await generateExportJSON(provinceId);
    
    // Validate completeness
    const warnings: string[] = [];
    if (!exportData.OVERVIEW.panorama) warnings.push('Province chưa có overview panorama');
    if (exportData.POIS.length === 0) warnings.push('Không có POI nào được publish');
    const poisWithoutPanorama = exportData.POIS.filter(p => !p.panorama);
    if (poisWithoutPanorama.length > 0) {
      warnings.push(`${poisWithoutPanorama.length} POI chưa có panorama`);
    }

    return successResponse({
      export: exportData,
      warnings,
      stats: {
        totalPois: exportData.POIS.length,
        totalCategories: exportData.CATEGORIES.length,
      },
    });
  } catch (error) {
    return errorResponse('SERVER_ERROR', 500, (error as Error).message);
  }
}
```

---

## 12. Deployment & DevOps

### 12.1 Environment Variables

```bash
# .env.local
DATABASE_URL="postgresql://user:pass@localhost:5432/tourism_dashboard"
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-secret-key"

# S3/R2 Storage
S3_ENDPOINT="https://your-account.r2.cloudflarestorage.com"
S3_ACCESS_KEY_ID="your-access-key"
S3_SECRET_ACCESS_KEY="your-secret-key"
S3_BUCKET="tourism-assets"
S3_PUBLIC_URL="https://cdn.example.com"

# Optional
NOMINATIM_URL="https://nominatim.openstreetmap.org"
```

### 12.2 Docker Compose (Development)

```yaml
# docker-compose.yml
version: '3.8'
services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: tourism_dashboard
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### 12.3 Setup Commands

```bash
# 1. Install dependencies
npm install

# 2. Start PostgreSQL
docker-compose up -d

# 3. Run Prisma migrations
npx prisma migrate dev --name init

# 4. Seed database
npx prisma db seed

# 5. Start development server
npm run dev
```

### 12.4 Seed Script

```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcryptjs';

const prisma = new PrismaClient();

async function main() {
  // Seed admin user
  await prisma.user.create({
    data: {
      email: 'admin@tourism360.vn',
      passwordHash: await bcrypt.hash('admin123', 12),
      name: 'Admin',
      role: 'admin',
    },
  });

  // Seed categories (matching demo)
  const categories = [
    { name: 'Biển', slug: 'beach', icon: 'waves', color: '#22D3A7', sortOrder: 1 },
    { name: 'Tâm linh', slug: 'temple', icon: 'church', color: '#F59E0B', sortOrder: 2 },
    { name: 'Ẩm thực', slug: 'food', icon: 'utensils', color: '#EC4899', sortOrder: 3 },
    { name: 'Thiên nhiên', slug: 'nature', icon: 'trees', color: '#3B82F6', sortOrder: 4 },
    { name: 'Di tích', slug: 'history', icon: 'landmark', color: '#8B5CF6', sortOrder: 5 },
    { name: 'Nghỉ dưỡng', slug: 'resort', icon: 'hotel', color: '#EF4444', sortOrder: 6 },
  ];

  for (const cat of categories) {
    await prisma.category.create({ data: cat });
  }

  // Seed province
  const province = await prisma.province.create({
    data: {
      name: 'Bà Rịa — Vũng Tàu',
      slug: 'ba-ria-vung-tau',
      description: 'Tỉnh ven biển phía Đông Nam Bộ',
      overviewPanorama: 'https://pannellum.org/images/cerro-toco-0.jpg',
      overviewYaw: 80,
      overviewPitch: 10,
      overviewHfov: 120,
      overviewAutoRotate: 0.2,
      isPublished: true,
    },
  });

  // Seed map config
  await prisma.mapConfig.create({
    data: {
      provinceId: province.id,
      centerLat: 10.40,
      centerLng: 107.15,
      defaultZoom: 9,
    },
  });

  // Seed tags
  const tags = ['Biển', 'Miễn phí', 'Hoàng hôn', 'Gia đình', 'Check-in', 'Hải sản', 'Lịch sử', 'Trekking'];
  for (const name of tags) {
    await prisma.tag.create({
      data: { name, slug: name.toLowerCase().replace(/\s+/g, '-') },
    });
  }

  console.log('✅ Database seeded successfully');
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

### 12.5 Package Dependencies

```json
{
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "@prisma/client": "^5.0.0",
    "next-auth": "^5.0.0-beta",
    "@tanstack/react-query": "^5.0.0",
    "zustand": "^4.5.0",
    "react-hook-form": "^7.50.0",
    "@hookform/resolvers": "^3.3.0",
    "zod": "^3.22.0",
    "bcryptjs": "^2.4.3",
    "sharp": "^0.33.0",
    "@aws-sdk/client-s3": "^3.500.0",
    "uuid": "^9.0.0",
    "leaflet": "^1.9.4",
    "react-leaflet": "^4.2.0",
    "lucide-react": "^0.468.0",
    "recharts": "^2.12.0",
    "react-colorful": "^5.6.0",
    "react-beautiful-dnd": "^13.1.0",
    "@radix-ui/react-dialog": "^1.0.0",
    "@radix-ui/react-dropdown-menu": "^2.0.0",
    "@radix-ui/react-tabs": "^1.0.0",
    "@radix-ui/react-switch": "^1.0.0",
    "@radix-ui/react-select": "^2.0.0",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.1.0",
    "tailwind-merge": "^2.2.0"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "prisma": "^5.0.0",
    "@types/react": "^18.2.0",
    "@types/leaflet": "^1.9.0",
    "@types/bcryptjs": "^2.4.0",
    "@types/uuid": "^9.0.0",
    "tailwindcss": "^3.4.0",
    "autoprefixer": "^10.4.0",
    "postcss": "^8.4.0",
    "eslint": "^8.50.0",
    "eslint-config-next": "^14.0.0"
  }
}
```

---

*End of Document*
