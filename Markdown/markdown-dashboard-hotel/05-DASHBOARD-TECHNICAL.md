# 05 — DASHBOARD TECHNICAL IMPLEMENTATION
## Admin Dashboard — Hotel CMS Platform

**Version:** 1.0  
**Date:** 2025-01-15  
**Status:** Draft  
**Related:** 01~04-DASHBOARD · markdown-hotel/04-TECHNICAL-IMPLEMENTATION

---

## Mục Lục

1. [Component Architecture](#1-component-architecture)
2. [State Management](#2-state-management)
3. [Form System](#3-form-system)
4. [Booking Modal — Frontend Implementation ★](#4-booking-modal--frontend-implementation)
5. [Hotspot Editor — Implementation](#5-hotspot-editor--implementation)
6. [API Layer](#6-api-layer)
7. [Auth Middleware](#7-auth-middleware)
8. [Email Service](#8-email-service)
9. [File Upload System](#9-file-upload-system)
10. [Testing Strategy](#10-testing-strategy)

---

## 1. Component Architecture

### 1.1 Admin Component Tree

```
src/app/admin/
├── layout.tsx                     ← AdminLayout (auth guard + shell)
│   ├── <AuthProvider>             ← JWT context, auto-refresh
│   ├── <QueryClientProvider>      ← TanStack Query
│   ├── <AdminShell>               ← Sidebar + Topbar + Content area
│   │   ├── <Sidebar>              ← Navigation, collapsible
│   │   ├── <Topbar>               ← Search, notifications, user menu
│   │   └── <main>{children}</main>← Page content slot
│   └── <Toaster>                  ← Sonner toast container
│
├── page.tsx                       ← Dashboard Overview
│   ├── <StatsCards>               ← 4 KPI cards (views, 360, bookings, content)
│   ├── <ViewsChart>               ← Line chart (Recharts)
│   ├── <TopScenesChart>           ← Bar chart
│   ├── <RecentActivity>           ← Activity feed list
│   └── <QuickActions>             ← Button group
│
├── rooms/
│   ├── page.tsx                   ← Room List
│   │   └── <DataTable>            ← TanStack Table + columns config
│   ├── new/page.tsx               ← Create Room
│   │   └── <RoomForm>             ← React Hook Form + Zod
│   └── [id]/page.tsx              ← Edit Room
│       └── <RoomForm>             ← Same form, pre-filled
│           ├── <TabGeneral>       ← Name, slug, desc, thumbnail
│           ├── <TabDetails>       ← Area, amenities, price
│           ├── <TabScenes>        ← Scene assignment, reorder
│           ├── <TabGallery>       ← Image upload grid
│           ├── <TabBooking>       ← ★ Booking mode override
│           └── <TabSEO>           ← Meta tags, JSON-LD preview
│
├── booking/
│   ├── page.tsx                   ← ★ Booking Configuration
│   │   ├── <BookingModeSelector>  ← External vs Modal radio
│   │   ├── <ExternalLinkConfig>   ← URL, target
│   │   ├── <ModalConfig>          ← Title, subtitle, button
│   │   ├── <FormFieldsBuilder>   ← Drag-to-reorder field list
│   │   ├── <ModalPreview>         ← Live preview of modal
│   │   └── <NotificationConfig>   ← Emails, webhook, auto-reply
│   └── requests/
│       └── page.tsx               ← ★ Booking Requests
│           ├── <RequestTabs>      ← Status filter tabs
│           ├── <RequestTable>     ← TanStack Table
│           └── <RequestDetail>    ← Slide-over panel
│
├── scenes/
│   ├── page.tsx                   ← Scene List
│   │   └── <DataTable>
│   └── [id]/
│       └── editor/page.tsx        ← ★ Hotspot Visual Editor
│           ├── <PanoramaEditor>   ← Pannellum (edit mode)
│           ├── <HotspotList>      ← Sidebar hotspot list
│           ├── <HotspotForm>      ← Detail form per type
│           └── <EditorToolbar>    ← Add, mode toggle, scene select
│
└── ... (other modules follow same pattern)
```

### 1.2 Shared Admin Components

```
src/components/admin/
├── layout/
│   ├── Sidebar.tsx                ← Collapsible nav with sections
│   ├── Topbar.tsx                 ← Search, notifications, user menu
│   ├── Breadcrumb.tsx             ← Auto-generated from route
│   ├── PageHeader.tsx             ← Title + action buttons
│   └── AdminShell.tsx             ← Combines sidebar + topbar
│
├── data/
│   ├── DataTable.tsx              ← TanStack Table wrapper
│   ├── DataTableToolbar.tsx       ← Search, filters, sort, export
│   ├── DataTablePagination.tsx    ← Page navigation
│   ├── DataTableRowActions.tsx    ← ⋮ menu per row
│   └── DataTableColumnHeader.tsx  ← Sortable header
│
├── forms/
│   ├── FormField.tsx              ← Label + input + error message
│   ├── RichTextEditor.tsx         ← Tiptap wrapper
│   ├── ImageUploader.tsx          ← Drag-drop + preview
│   ├── PanoramaUploader.tsx       ← Special panorama upload
│   ├── ColorPicker.tsx            ← Color input
│   ├── DatePicker.tsx             ← Date input
│   ├── TagInput.tsx               ← Multi-tag input
│   ├── SlugField.tsx              ← Auto-gen from name
│   └── SortableList.tsx           ← Drag-to-reorder
│
├── feedback/
│   ├── ConfirmDialog.tsx          ← "Are you sure?" modal
│   ├── LoadingOverlay.tsx         ← Full-page loading
│   ├── EmptyState.tsx             ← No data placeholder
│   └── ErrorBoundary.tsx          ← Error fallback UI
│
└── booking/                        ← ★ Booking-specific components
    ├── BookingModeSelector.tsx     ← External vs Modal radio cards
    ├── FormFieldsBuilder.tsx      ← Drag-to-reorder field list
    ├── FormFieldEditor.tsx        ← Edit single field (dialog)
    ├── ModalPreview.tsx           ← Live modal preview
    ├── NotificationConfig.tsx     ← Email/webhook settings
    ├── RequestTable.tsx           ← Booking requests table
    └── RequestDetail.tsx          ← Request detail slide-over
```

---

## 2. State Management

### 2.1 TanStack Query — Server State

```typescript
// src/hooks/admin/useRooms.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { adminApi } from '@/lib/admin-api';

// List rooms
export function useRooms(params?: { status?: string; type?: string }) {
  return useQuery({
    queryKey: ['admin', 'rooms', params],
    queryFn: () => adminApi.getRooms(params),
    staleTime: 30 * 1000, // 30s
  });
}

// Get single room
export function useRoom(id: string) {
  return useQuery({
    queryKey: ['admin', 'rooms', id],
    queryFn: () => adminApi.getRoom(id),
    enabled: !!id,
  });
}

// Create room
export function useCreateRoom() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: adminApi.createRoom,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin', 'rooms'] });
      toast.success('Room created successfully!');
    },
    onError: (err) => {
      toast.error(err.message || 'Failed to create room');
    },
  });
}

// Update room
export function useUpdateRoom(id: string) {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: Partial<Room>) => adminApi.updateRoom(id, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin', 'rooms', id] });
      queryClient.invalidateQueries({ queryKey: ['admin', 'rooms'] });
      toast.success('Room saved!');
    },
  });
}

// Publish/unpublish
export function usePublishRoom(id: string) {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (publish: boolean) => adminApi.publishRoom(id, publish),
    onSuccess: (_, publish) => {
      queryClient.invalidateQueries({ queryKey: ['admin', 'rooms'] });
      toast.success(publish ? 'Room published!' : 'Room unpublished');
    },
  });
}
```

### 2.2 TanStack Query — Booking

```typescript
// src/hooks/admin/useBooking.ts

// Get booking config
export function useBookingConfig() {
  return useQuery({
    queryKey: ['admin', 'booking', 'config'],
    queryFn: adminApi.getBookingConfig,
  });
}

// Update booking config
export function useUpdateBookingConfig() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: adminApi.updateBookingConfig,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin', 'booking', 'config'] });
      queryClient.invalidateQueries({ queryKey: ['public', 'booking', 'config'] });
      toast.success('Booking configuration saved!');
    },
  });
}

// Get booking requests
export function useBookingRequests(params?: { status?: string; page?: number }) {
  return useQuery({
    queryKey: ['admin', 'booking', 'requests', params],
    queryFn: () => adminApi.getBookingRequests(params),
    refetchInterval: 30 * 1000, // Auto-refresh every 30s (for new requests)
  });
}

// Update request status
export function useUpdateBookingRequest(id: string) {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: { status: string; notes?: string }) =>
      adminApi.updateBookingRequest(id, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin', 'booking', 'requests'] });
      toast.success('Request updated!');
    },
  });
}

// Get form fields
export function useBookingFormFields() {
  return useQuery({
    queryKey: ['admin', 'booking', 'fields'],
    queryFn: adminApi.getBookingFormFields,
  });
}

// Reorder form fields
export function useReorderBookingFields() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (fieldIds: string[]) => adminApi.reorderBookingFields(fieldIds),
    onMutate: async (fieldIds) => {
      // Optimistic update
      await queryClient.cancelQueries({ queryKey: ['admin', 'booking', 'fields'] });
      const previous = queryClient.getQueryData(['admin', 'booking', 'fields']);
      // Reorder in cache
      queryClient.setQueryData(['admin', 'booking', 'fields'], (old: any[]) =>
        fieldIds.map((id, i) => ({ ...old.find(f => f.id === id), sort_order: i }))
      );
      return { previous };
    },
    onError: (_, __, context) => {
      queryClient.setQueryData(['admin', 'booking', 'fields'], context?.previous);
    },
  });
}
```

### 2.3 Zustand — Client UI State

```typescript
// src/stores/adminStore.ts
import { create } from 'zustand';

interface AdminUIState {
  sidebarCollapsed: boolean;
  toggleSidebar: () => void;

  // Hotspot editor
  editorMode: 'edit' | 'preview';
  selectedHotspotId: string | null;
  setEditorMode: (mode: 'edit' | 'preview') => void;
  selectHotspot: (id: string | null) => void;

  // Booking request detail
  selectedRequestId: string | null;
  setSelectedRequest: (id: string | null) => void;
}

export const useAdminStore = create<AdminUIState>((set) => ({
  sidebarCollapsed: false,
  toggleSidebar: () => set((s) => ({ sidebarCollapsed: !s.sidebarCollapsed })),

  editorMode: 'edit',
  selectedHotspotId: null,
  setEditorMode: (editorMode) => set({ editorMode }),
  selectHotspot: (selectedHotspotId) => set({ selectedHotspotId }),

  selectedRequestId: null,
  setSelectedRequest: (selectedRequestId) => set({ selectedRequestId }),
}));
```

---

## 3. Form System

### 3.1 React Hook Form + Zod Pattern

```typescript
// src/app/admin/rooms/[id]/page.tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useRoom, useUpdateRoom } from '@/hooks/admin/useRooms';

const roomFormSchema = z.object({
  name: z.string().min(3, 'Name must be at least 3 characters'),
  slug: z.string().regex(/^[a-z0-9-]+$/, 'Slug must be lowercase with hyphens'),
  room_type: z.enum(['standard', 'superior', 'deluxe', 'suite', 'villa']),
  short_desc: z.string().max(500).optional(),
  description: z.string().optional(),
  area_sqm: z.number().positive().optional(),
  max_adults: z.number().int().min(1).max(20).default(2),
  price_from: z.number().positive().optional(),
  booking_mode: z.enum(['external', 'modal']).optional().nullable(),
  booking_url: z.string().url().optional().nullable(),
  is_published: z.boolean().default(false),
  amenity_ids: z.array(z.string()),
  // ... other fields
});

type RoomFormValues = z.infer<typeof roomFormSchema>;

export default function RoomEditPage({ params }: { params: { id: string } }) {
  const { data: room, isLoading } = useRoom(params.id);
  const updateRoom = useUpdateRoom(params.id);

  const form = useForm<RoomFormValues>({
    resolver: zodResolver(roomFormSchema),
    defaultValues: room,
    mode: 'onBlur', // Validate on blur
  });

  const onSubmit = (data: RoomFormValues) => {
    updateRoom.mutate(data);
  };

  // Unsaved changes warning
  const isDirty = form.formState.isDirty;
  useBeforeUnload(isDirty, 'You have unsaved changes. Leave anyway?');

  // Auto-generate slug from name
  const name = form.watch('name');
  useEffect(() => {
    if (!form.getValues('slug') || isNewRoom) {
      form.setValue('slug', slugify(name));
    }
  }, [name]);

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)}>
        <PageHeader
          title={room?.name || 'New Room'}
          actions={
            <>
              <Button variant="outline" type="submit" disabled={updateRoom.isPending}>
                {updateRoom.isPending ? <Spinner /> : 'Save Draft'}
              </Button>
              <Button onClick={() => handlePublish()} variant="default">
                Publish
              </Button>
            </>
          }
        />
        <Tabs defaultValue="general">
          <TabsList>
            <TabsTrigger value="general">General</TabsTrigger>
            <TabsTrigger value="details">Details</TabsTrigger>
            <TabsTrigger value="scenes">Scenes</TabsTrigger>
            <TabsTrigger value="gallery">Gallery</TabsTrigger>
            <TabsTrigger value="booking">Booking</TabsTrigger>
            <TabsTrigger value="seo">SEO</TabsTrigger>
          </TabsList>
          <TabsContent value="general"><TabGeneral form={form} /></TabsContent>
          <TabsContent value="details"><TabDetails form={form} /></TabsContent>
          <TabsContent value="scenes"><TabScenes roomId={params.id} /></TabsContent>
          <TabsContent value="gallery"><TabGallery roomId={params.id} /></TabsContent>
          <TabsContent value="booking"><TabBooking form={form} /></TabsContent>
          <TabsContent value="seo"><TabSEO form={form} /></TabsContent>
        </Tabs>
      </form>
    </Form>
  );
}
```

---

## 4. Booking Modal — Frontend Implementation ★

### 4.1 Public-facing BookingModal Component

```typescript
// src/components/overlay/BookingModal.tsx
'use client';

import { useState, useEffect } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useBookingConfig } from '@/hooks/usePublicBooking';

interface BookingModalProps {
  isOpen: boolean;
  onClose: () => void;
  room?: { id: string; name: string; slug: string };
  service?: { id: string; name: string };
}

export function BookingModal({ isOpen, onClose, room, service }: BookingModalProps) {
  const { data: config } = useBookingConfig();
  const [status, setStatus] = useState<'idle' | 'submitting' | 'success' | 'error'>('idle');

  // Build Zod schema dynamically from config fields
  const schema = useMemo(() => {
    if (!config?.modal?.fields) return z.object({});

    const shape: Record<string, z.ZodTypeAny> = {};
    for (const field of config.modal.fields) {
      let fieldSchema: z.ZodTypeAny;
      switch (field.type) {
        case 'email': fieldSchema = z.string().email(); break;
        case 'tel': fieldSchema = z.string().min(8); break;
        case 'number': fieldSchema = z.number().int(); break;
        case 'date': fieldSchema = z.string().regex(/^\d{4}-\d{2}-\d{2}$/); break;
        case 'textarea': fieldSchema = z.string().max(2000); break;
        default: fieldSchema = z.string().min(1);
      }
      if (field.validation?.min) fieldSchema = (fieldSchema as any).min(field.validation.min);
      if (field.validation?.max) fieldSchema = (fieldSchema as any).max(field.validation.max);
      shape[field.name] = field.required ? fieldSchema : fieldSchema.optional();
    }
    return z.object(shape);
  }, [config]);

  const form = useForm({
    resolver: zodResolver(schema),
    defaultValues: buildDefaults(config?.modal?.fields),
  });

  const onSubmit = async (data: Record<string, any>) => {
    setStatus('submitting');
    try {
      await fetch('/api/v1/booking/request', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          ...data,
          room_id: room?.id,
          room_name: room?.name,
          service_id: service?.id,
          source_page: window.location.pathname,
          source_scene: getCurrentSceneId(), // from sceneStore
          session_id: getSessionId(),
        }),
      });
      setStatus('success');
      track('booking_request_submit', { room: room?.slug, source: 'modal' });
      setTimeout(onClose, 5000); // Auto-close after 5s
    } catch (err) {
      setStatus('error');
    }
  };

  if (!isOpen) return null;

  return (
    <div className="booking-modal-overlay" onClick={onClose}>
      <div className="booking-modal" onClick={e => e.stopPropagation()}>
        <button className="close-btn" onClick={onClose}>×</button>

        {status === 'success' ? (
          <SuccessView
            message={config?.modal?.success_msg}
            guestEmail={form.getValues('email')}
            roomName={room?.name}
          />
        ) : (
          <>
            <h2>{config?.modal?.title || 'Book Your Stay'}</h2>
            {config?.modal?.subtitle && <p>{config.modal.subtitle}</p>}

            {room && (
              <div className="room-badge">
                Room: {room.name}
              </div>
            )}

            <form onSubmit={form.handleSubmit(onSubmit)}>
              {config?.modal?.fields?.map(field => (
                <DynamicField
                  key={field.name}
                  field={field}
                  register={form.register}
                  error={form.formState.errors[field.name]}
                />
              ))}

              {/* Honeypot field (spam prevention) */}
              <input
                type="text"
                name="website"
                style={{ display: 'none' }}
                tabIndex={-1}
                autoComplete="off"
              />

              <button
                type="submit"
                disabled={status === 'submitting'}
                style={{ backgroundColor: config?.modal?.btn_color || '#C9A861' }}
              >
                {status === 'submitting' ? (
                  <Spinner />
                ) : (
                  config?.modal?.btn_label || 'Send Booking Request'
                )}
              </button>
            </form>
          </>
        )}
      </div>
    </div>
  );
}

// Dynamic field renderer
function DynamicField({ field, register, error }) {
  switch (field.type) {
    case 'textarea':
      return <textarea {...register(field.name)} placeholder={field.placeholder} />;
    case 'select':
      return (
        <select {...register(field.name)}>
          {field.options?.map(opt => (
            <option key={opt.value} value={opt.value}>{opt.label}</option>
          ))}
        </select>
      );
    case 'date':
      return <input type="date" {...register(field.name)} />;
    case 'number':
      return (
        <input
          type="number"
          {...register(field.name, { valueAsNumber: true })}
          min={field.validation?.min}
          max={field.validation?.max}
        />
      );
    default:
      return (
        <input
          type={field.type}
          {...register(field.name)}
          placeholder={field.placeholder}
        />
      );
  }
}
```

### 4.2 CTA Button — Mode Resolution

```typescript
// src/components/overlay/BookingCTA.tsx

import { useBookingConfig } from '@/hooks/usePublicBooking';

interface BookingCTAProps {
  room?: Room;
  service?: Service;
  variant: 'panel' | 'floating' | 'hotspot';
}

export function BookingCTA({ room, service, variant }: BookingCTAProps) {
  const { data: config } = useBookingConfig();
  const [modalOpen, setModalOpen] = useState(false);

  const resolvedMode = resolveBookingMode(room, service, config);

  const handleClick = () => {
    track('booking_click', {
      room: room?.slug,
      service: service?.slug,
      source: variant,
    });

    if (resolvedMode.mode === 'external') {
      window.open(resolvedMode.url, config?.external?.target || '_blank');
    } else {
      setModalOpen(true);
    }
  };

  return (
    <>
      <button className="booking-cta" onClick={handleClick}>
        {room?.cta_label || 'Book Now'}
      </button>

      {resolvedMode.mode === 'modal' && (
        <BookingModal
          isOpen={modalOpen}
          onClose={() => setModalOpen(false)}
          room={room}
          service={service}
        />
      )}
    </>
  );
}

// Resolution logic
function resolveBookingMode(
  room?: Room,
  service?: Service,
  config?: BookingConfig
): { mode: 'external' | 'modal'; url?: string } {
  const entity = room || service;

  // 1. Entity-level override
  if (entity?.booking_mode === 'external') {
    return { mode: 'external', url: entity.booking_url || config?.external?.url };
  }
  if (entity?.booking_mode === 'modal') {
    return { mode: 'modal' };
  }

  // 2. Global config
  if (config?.mode === 'modal') {
    return { mode: 'modal' };
  }

  // 3. Default: external
  return { mode: 'external', url: entity?.booking_url || config?.external?.url || '#' };
}
```

### 4.3 Booking Modal Styling (trên nền 360°)

```css
/* Booking Modal — matches luxury dark theme */
.booking-modal-overlay {
  position: fixed;
  inset: 0;
  z-index: 500;                    /* Above all 360 UI */
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  animation: fadeIn 0.3s ease;
}

.booking-modal {
  background: var(--panel-solid, rgba(12, 20, 38, 0.96));
  border: 1px solid var(--glass-border);
  border-radius: 16px;
  padding: 40px;
  max-width: 480px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  animation: scaleIn 0.3s var(--ease);
  color: var(--text);
}

.booking-modal h2 {
  font-family: var(--serif);
  font-size: 28px;
  color: var(--gold);
  margin-bottom: 8px;
}

.booking-modal button[type="submit"] {
  width: 100%;
  padding: 14px;
  background: var(--gold);
  color: var(--dark);
  border: none;
  border-radius: 8px;
  font-weight: 600;
  font-size: 16px;
  cursor: pointer;
  margin-top: 16px;
  transition: opacity 0.2s;
}

/* Mobile: full-screen bottom sheet */
@media (max-width: 860px) {
  .booking-modal-overlay {
    align-items: flex-end;
  }
  .booking-modal {
    max-width: 100%;
    width: 100%;
    border-radius: 20px 20px 0 0;
    max-height: 85vh;
    padding: 32px 24px;
    animation: slideUpFromBottom 0.35s var(--ease);
  }
}

@keyframes scaleIn {
  from { opacity: 0; transform: scale(0.95); }
  to { opacity: 1; transform: scale(1); }
}

@keyframes slideUpFromBottom {
  from { transform: translateY(100%); }
  to { transform: translateY(0); }
}
```

---

## 5. Hotspot Editor — Implementation

### 5.1 Pannellum Editor Mode

```typescript
// src/app/admin/scenes/[id]/editor/page.tsx
'use client';

import { useRef, useCallback, useEffect } from 'react';
import { useAdminStore } from '@/stores/adminStore';
import { useScene, useHotspots, useCreateHotspot, useUpdateHotspotPosition }
  from '@/hooks/admin/useScenes';

export default function HotspotEditorPage({ params }: { params: { id: string } }) {
  const { data: scene } = useScene(params.id);
  const { data: hotspots } = useHotspots(params.id);
  const createHotspot = useCreateHotspot(params.id);
  const updatePosition = useUpdateHotspotPosition();
  const viewerRef = useRef<any>(null);
  const containerRef = useRef<HTMLDivElement>(null);
  const { editorMode, selectedHotspotId, selectHotspot, setEditorMode } = useAdminStore();

  // Init Pannellum in editor mode
  useEffect(() => {
    if (!scene || !containerRef.current) return;

    const viewer = pannellum.viewer(containerRef.current, {
      type: 'equirectangular',
      panorama: scene.panorama_url,
      autoLoad: true,
      showControls: false,
      yaw: scene.default_yaw,
      pitch: scene.default_pitch,
      hfov: scene.default_hfov,
    });

    viewerRef.current = viewer;

    // Click-to-place in edit mode
    viewer.on('mouseup', (event: MouseEvent) => {
      if (editorMode !== 'edit') return;
      // Only if no hotspot was clicked
      if ((event.target as HTMLElement).closest('.hotspot-marker')) return;

      const coords = viewer.mouseEventToCoords(event);
      handlePlaceHotspot(coords);
    });

    return () => viewer.destroy();
  }, [scene]);

  // Render hotspots
  useEffect(() => {
    if (!viewerRef.current || !hotspots) return;
    // Clear existing
    viewerRef.current.removeAllHotSpots();

    hotspots.forEach(hs => {
      viewerRef.current.addHotSpot({
        pitch: hs.pitch,
        yaw: hs.yaw,
        type: 'custom',
        cssClass: `hotspot-editor ${selectedHotspotId === hs.id ? 'selected' : ''}`,
        createTooltipFunc: (el) => renderEditorHotspot(el, hs),
        clickHandlerFunc: () => selectHotspot(hs.id),
        // Draggable in edit mode
        draggable: editorMode === 'edit',
        dragHandlerFunc: (event, coords) => {
          updatePosition.mutate({ id: hs.id, pitch: coords[0], yaw: coords[1] });
        },
      });
    });
  }, [hotspots, selectedHotspotId, editorMode]);

  const handlePlaceHotspot = (coords: [number, number]) => {
    // Show type selection dialog
    setPlacementCoords(coords);
    setShowTypeDialog(true);
  };

  return (
    <div className="hotspot-editor-layout">
      <EditorToolbar
        mode={editorMode}
        onModeChange={setEditorMode}
        onAddHotspot={() => { /* enter placement mode */ }}
        sceneId={params.id}
      />
      <div className="editor-content">
        <div className="panorama-editor" ref={containerRef} />
        <div className="hotspot-sidebar">
          <HotspotList
            hotspots={hotspots || []}
            selectedId={selectedHotspotId}
            onSelect={selectHotspot}
          />
          {selectedHotspotId && (
            <HotspotForm
              hotspot={hotspots?.find(h => h.id === selectedHotspotId)}
              sceneId={params.id}
            />
          )}
        </div>
      </div>
    </div>
  );
}
```

---

## 6. API Layer

### 6.1 Admin API Client

```typescript
// src/lib/admin-api.ts

class AdminAPIClient {
  private async fetch<T>(path: string, options?: RequestInit): Promise<T> {
    const res = await fetch(`/api/v1/admin${path}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options?.headers,
      },
      credentials: 'include', // Send httpOnly cookies
    });

    if (res.status === 401) {
      // Try refresh
      const refreshed = await this.refreshToken();
      if (refreshed) {
        return this.fetch(path, options); // Retry
      }
      window.location.href = '/admin/login';
      throw new Error('Unauthorized');
    }

    if (!res.ok) {
      const error = await res.json().catch(() => ({ message: 'Unknown error' }));
      throw new APIError(res.status, error.message);
    }

    return res.json();
  }

  // Auth
  async login(email: string, password: string) {
    return this.fetch('/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    });
  }

  // Rooms
  async getRooms(params?: any) { return this.fetch('/rooms?' + toQuery(params)); }
  async getRoom(id: string) { return this.fetch(`/rooms/${id}`); }
  async createRoom(data: any) { return this.fetch('/rooms', { method: 'POST', body: JSON.stringify(data) }); }
  async updateRoom(id: string, data: any) { return this.fetch(`/rooms/${id}`, { method: 'PUT', body: JSON.stringify(data) }); }
  async deleteRoom(id: string) { return this.fetch(`/rooms/${id}`, { method: 'DELETE' }); }
  async publishRoom(id: string, publish: boolean) {
    return this.fetch(`/rooms/${id}/publish`, { method: 'PUT', body: JSON.stringify({ is_published: publish }) });
  }

  // Booking ★
  async getBookingConfig() { return this.fetch('/booking/config'); }
  async updateBookingConfig(data: any) { return this.fetch('/booking/config', { method: 'PUT', body: JSON.stringify(data) }); }
  async getBookingFormFields() { return this.fetch('/booking/fields'); }
  async createBookingField(data: any) { return this.fetch('/booking/fields', { method: 'POST', body: JSON.stringify(data) }); }
  async updateBookingField(id: string, data: any) { return this.fetch(`/booking/fields/${id}`, { method: 'PUT', body: JSON.stringify(data) }); }
  async deleteBookingField(id: string) { return this.fetch(`/booking/fields/${id}`, { method: 'DELETE' }); }
  async reorderBookingFields(ids: string[]) { return this.fetch('/booking/fields/sort', { method: 'PUT', body: JSON.stringify({ field_ids: ids }) }); }
  async getBookingRequests(params?: any) { return this.fetch('/booking/requests?' + toQuery(params)); }
  async getBookingRequest(id: string) { return this.fetch(`/booking/requests/${id}`); }
  async updateBookingRequest(id: string, data: any) { return this.fetch(`/booking/requests/${id}`, { method: 'PUT', body: JSON.stringify(data) }); }
  async sendTestEmail() { return this.fetch('/booking/test-email', { method: 'POST' }); }

  // Scenes & Hotspots
  async getScenes() { return this.fetch('/scenes'); }
  async getScene(id: string) { return this.fetch(`/scenes/${id}`); }
  async getHotspots(sceneId: string) { return this.fetch(`/scenes/${sceneId}/hotspots`); }
  async createHotspot(sceneId: string, data: any) { return this.fetch(`/scenes/${sceneId}/hotspots`, { method: 'POST', body: JSON.stringify(data) }); }
  async updateHotspot(id: string, data: any) { return this.fetch(`/hotspots/${id}`, { method: 'PUT', body: JSON.stringify(data) }); }
  async updateHotspotPosition(id: string, pos: { pitch: number; yaw: number }) { return this.fetch(`/hotspots/${id}/position`, { method: 'PUT', body: JSON.stringify(pos) }); }
  async deleteHotspot(id: string) { return this.fetch(`/hotspots/${id}`, { method: 'DELETE' }); }

  // Upload
  async uploadImage(file: File) {
    const formData = new FormData();
    formData.append('file', file);
    return fetch('/api/v1/admin/upload/image', {
      method: 'POST',
      body: formData,
      credentials: 'include',
    }).then(r => r.json());
  }

  async uploadPanorama(file: File) {
    const formData = new FormData();
    formData.append('file', file);
    return fetch('/api/v1/admin/upload/panorama', {
      method: 'POST',
      body: formData,
      credentials: 'include',
    }).then(r => r.json());
  }

  // ... other endpoints
}

export const adminApi = new AdminAPIClient();
```

---

## 7. Auth Middleware

```typescript
// src/app/api/v1/admin/middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { verifyJWT } from '@/lib/auth';

type Role = 'super_admin' | 'admin' | 'editor' | 'viewer';

export function withAuth(handler: Function, requiredRole?: Role) {
  return async (req: NextRequest, ...args: any[]) => {
    const token = req.cookies.get('access_token')?.value;

    if (!token) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    try {
      const payload = await verifyJWT(token);
      const user = { id: payload.sub, email: payload.email, role: payload.role };

      // Check role hierarchy
      if (requiredRole && !hasPermission(user.role, requiredRole)) {
        return NextResponse.json({ error: 'Forbidden' }, { status: 403 });
      }

      // Attach user to request
      (req as any).user = user;
      return handler(req, ...args);
    } catch (err) {
      return NextResponse.json({ error: 'Invalid token' }, { status: 401 });
    }
  };
}

const ROLE_HIERARCHY: Record<Role, number> = {
  super_admin: 4,
  admin: 3,
  editor: 2,
  viewer: 1,
};

function hasPermission(userRole: Role, requiredRole: Role): boolean {
  return ROLE_HIERARCHY[userRole] >= ROLE_HIERARCHY[requiredRole];
}
```

---

## 8. Email Service

```typescript
// src/lib/email.ts
import { Resend } from 'resend';

const resend = new Resend(process.env.RESEND_API_KEY);

export async function sendBookingNotification(
  request: BookingRequest,
  config: BookingConfig,
  room?: Room
) {
  // 1. Send to hotel staff
  await resend.emails.send({
    from: 'noreply@botonblue.com',
    to: config.notifyEmails,
    replyTo: request.guestEmail,
    subject: `🏨 New Booking Request — ${room?.name || 'General Inquiry'}`,
    html: renderHotelNotificationEmail(request, room),
  });

  // 2. Send auto-reply to guest (if enabled)
  if (config.autoReplyEnabled) {
    await resend.emails.send({
      from: 'noreply@botonblue.com',
      to: [request.guestEmail],
      subject: config.autoReplySubject,
      html: config.autoReplyTemplate
        ? renderCustomTemplate(config.autoReplyTemplate, request, room)
        : renderDefaultAutoReply(request, room),
    });
  }

  // 3. Send webhook (if configured)
  if (config.notifyWebhookUrl) {
    await fetch(config.notifyWebhookUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        type: 'booking_request',
        data: { ...request, room_name: room?.name },
      }),
    }).catch(console.warn); // Don't fail on webhook error
  }
}
```

---

## 9. File Upload System

### 9.1 Upload Route Handler

```typescript
// src/app/api/v1/admin/upload/image/route.ts
import { NextRequest, NextResponse } from 'next/server';
import sharp from 'sharp';
import { uploadToS3 } from '@/lib/upload';
import { withAuth } from '../middleware';

export const POST = withAuth(async (req: NextRequest) => {
  const formData = await req.formData();
  const file = formData.get('file') as File;

  if (!file) {
    return NextResponse.json({ error: 'No file provided' }, { status: 400 });
  }

  // Validate
  const validTypes = ['image/jpeg', 'image/png', 'image/webp'];
  if (!validTypes.includes(file.type)) {
    return NextResponse.json({ error: 'Invalid file type' }, { status: 400 });
  }
  if (file.size > 10 * 1024 * 1024) {
    return NextResponse.json({ error: 'File too large (max 10MB)' }, { status: 400 });
  }

  const buffer = Buffer.from(await file.arrayBuffer());
  const hash = createHash('sha256').update(buffer).digest('hex').slice(0, 12);
  const now = new Date();
  const prefix = `uploads/${now.getFullYear()}/${String(now.getMonth() + 1).padStart(2, '0')}`;

  // Generate sizes
  const sizes = [
    { suffix: '', width: null },         // original
    { suffix: '-lg', width: 1920 },      // large
    { suffix: '-md', width: 800 },       // medium
    { suffix: '-thumb', width: 300 },    // thumbnail
  ];

  const urls: Record<string, string> = {};

  for (const size of sizes) {
    let processed = sharp(buffer);
    if (size.width) {
      processed = processed.resize(size.width, null, { withoutEnlargement: true });
    }
    const webpBuffer = await processed.webp({ quality: 85 }).toBuffer();
    const key = `${prefix}/${hash}${size.suffix}.webp`;
    urls[size.suffix || 'original'] = await uploadToS3(key, webpBuffer, 'image/webp');
  }

  return NextResponse.json({
    url: urls.original,
    large: urls['-lg'],
    medium: urls['-md'],
    thumbnail: urls['-thumb'],
  });
}, 'editor');
```

---

## 10. Testing Strategy

### 10.1 Test Pyramid

| Level | Tools | Scope |
|-------|-------|-------|
| Unit | Vitest | Utility functions, Zod schemas, helpers |
| Component | Vitest + Testing Library | Form components, data table, modal |
| Integration | Vitest + MSW | API hooks (TanStack Query + mock API) |
| E2E | Playwright | Critical flows: login, create room, booking config |

### 10.2 Critical E2E Tests

```typescript
// e2e/booking-config.spec.ts
test('admin can switch booking mode from external to modal', async ({ page }) => {
  await page.goto('/admin/login');
  await page.fill('[name="email"]', 'admin@botonblue.com');
  await page.fill('[name="password"]', 'password');
  await page.click('button[type="submit"]');

  await page.goto('/admin/booking');
  await page.click('text=In-Page Booking Modal');

  // Customize modal
  await page.fill('[name="modal_title"]', 'Reserve Your Room');
  await page.click('text=Save Changes');
  await expect(page.locator('.toast-success')).toBeVisible();

  // Verify on public site
  await page.goto('/rooms/deluxe-ocean');
  await page.click('text=Book Now');
  await expect(page.locator('.booking-modal')).toBeVisible();
  await expect(page.locator('.booking-modal h2')).toContainText('Reserve Your Room');
});
```

---

*Hoàn tất bộ 5 tài liệu Dashboard Hotel. Tiếp theo: `markdown-dashboard-tourism/`*
