# GuardPoint – Full Tickable Task List  
Product: QR-Code Security Patrol Check-in Web App  
Deadline: Sunday, December 14, 2025 (End of Week)  
Stack: React 19 + Vite + TypeScript + Tailwind 4 + shadcn/ui + Hono (Cloudflare Workers) + D1 Database  
Repository: https://github.com/decagondev/security-checkin

All tasks are written with SOLID principles & modular design enforced (Repository Pattern, Service Layer, Dependency Injection, Single Responsibility).

### Epic 0 – AI-Assisted Development Foundation (Memory Bank + Cursor Rules)

- [ ] PR 0.1 – Initialize Cursor AI memory bank (do NOT push to GitHub)
  - [ ] Create folder `.cursor/` at project root
  - [ ] Create `.cursor/rules.md` with:
    - Project context, naming conventions, SOLID enforcement
    - Always use Repository pattern for D1 access
    - All services must be injectable
    - Prefer Zod for validation
    - Tailwind 4 + shadcn/ui only
  - [ ] Create `.cursor/memory-bank.md` containing this entire PRD + task list
  - [ ] Add `.cursor/` to `.gitignore`
  - [ ] Commit: `chore: initialize cursor memory bank and rules (local only)`

### Epic 1 – D1 Database Schema & Migrations (SOLID Repository Ready)

- [ ] PR 1.1 – Create D1 database and schema
  - [ ] Run `npx wrangler d1 create guardpoint-db`
  - [ ] Create `drizzle/schema.ts` OR raw `schema.sql` with:
    - `guards (id INTEGER PRIMARY KEY, name TEXT, badge_number TEXT UNIQUE)`
    - `checkpoints (id INTEGER PRIMARY KEY, name TEXT, location TEXT)`
    - `patrol_logs (id INTEGER PRIMARY KEY AUTOINCREMENT, guard_id INTEGER, checkpoint_id INTEGER, scanned_at DATETIME DEFAULT CURRENT_TIMESTAMP, FOREIGN KEY(guard_id) REFERENCES guards(id), FOREIGN KEY(checkpoint_id) REFERENCES checkpoints(id))`
  - [ ] Create `drizzle/0000_initial.sql` migration
  - [ ] Create `drizzle/seed.sql` → 3 guards + 10 checkpoints
  - [ ] Commit: `feat(db): create D1 schema and initial migration`

- [ ] PR 1.2 – D1 Repository Layer (SOLID)
  - [ ] Create `src/core/repositories/PatrolLogRepository.ts` (interface + implementation)
  - [ ] Create `src/core/repositories/GuardRepository.ts` (interface + impl)
  - [ ] Create `src/core/repositories/CheckpointRepository.ts` (interface + impl)
  - [ ] Create `src/db/d1-client.ts` → returns env.DB (injected)
  - [ ] All repositories use prepared statements + typed results
  - [ ] Commit: `feat(db): add SOLID repository layer for D1`

### Epic 2 – Backend API with Hono (Modular, SOLID Services)

- [ ] PR 2.1 – Hono API skeleton + health check
  - [ ] Update `src/server.ts` → Hono app with `/health` → 200
  - [ ] Add `src/api/routes/v1/index.ts` router
  - [ ] Commit: `feat(api): hono skeleton with health endpoint`

- [ ] PR 2.2 – POST /api/v1/logs (scan endpoint)
  - [ ] Create `src/core/services/PatrolLogService.ts` (interface + impl)
  - [ ] Create `src/api/routes/v1/logs/post-log.ts`
  - [ ] Validate payload with Zod schema: `{ qrData: string }` → parse guardId|checkpointId or JSON
  - [ ] Service injects PatrolLogRepository
  - [ ] Return 201 with full log object (including guard & checkpoint names)
  - [ ] Global error handler → 400/500 JSON
  - [ ] Commit: `feat(api): POST /logs - create patrol log from QR scan`

- [ ] PR 2.3 – GET /api/v1/logs (supervisor list)
  - [ ] Add filters: guardId?, checkpointId?, dateFrom?, dateTo?, limit=50, offset=0
  - [ ] Use PatrolLogService.getAllWithDetails()
  - [ ] Return array with joined guard & checkpoint names
  - [ ] Commit: `feat(api): GET /logs with filters and pagination`

- [ ] PR 2.4 – Reference endpoints
  - [ ] GET /api/v1/guards → list all
  - [ ] GET /api/v1/checkpoints → list all
  - [ ] Commit: `feat(api): reference endpoints for guards and checkpoints`

### Epic 3 – Frontend Core (React + shadcn + Mobile-First PWA)

- [ ] PR 3.1 – PWA setup & mobile layout
  - [ ] Update `vite.config.ts` → base + PWA plugin (already in template)
  - [ ] Update `public/manifest.json` (name, icons, start_url)
  - [ ] Add meta tags for iOS & Android
  - [ ] Create `src/layouts/MobileLayout.tsx` → max-w-md mx-auto min-h-screen
  - [ ] Commit: `feat(ui): mobile-first PWA foundation`

- [ ] PR 3.2 – QR/Barcode Scanner Page
  - [ ] Install `@zxing/browser@0.1.4`
  - [ ] Create `src/pages/Scanner.tsx`
  - [ ] Full-screen video + overlay (shadcn Card + custom SVG overlay)
  - [ ] Auto-submit on successful read → POST /api/v1/logs
  - [ ] Fallback manual input field + button
  - [ ] Vibration (navigator.vibrate) + beep sound on success
  - [ ] Commit: `feat(scanner): live QR/barcode scanner with fallback`

- [ ] PR 3.3 – Scan Success Screen
  - [ ] Create `src/pages/Success.tsx`
  - [ ] Large green checkmark (lucide-react)
  - [ ] Show: Guard name, Checkpoint, Exact time
  - [ ] "Scan next checkpoint" button → navigate back
  - [ ] Auto-redirect after 3s (optional toggle)
  - [ ] Commit: `feat(ui): scan success feedback screen`

- [ ] PR 3.4 – Supervisor Dashboard
  - [ ] Create `src/pages/supervisor/Dashboard.tsx`
  - [ ] Filters: Date range (shadcn Calendar), Guard dropdown, Checkpoint dropdown
  - [ ] shadcn DataTable with columns: Time, Guard, Checkpoint, Actions
  - [ ] Sorting + pagination
  - [ ] Pull-to-refresh (mobile)
  - [ ] Export CSV button
  - [ ] Commit: `feat(dashboard): full supervisor activity view`

- [ ] PR 3.5 – Navigation & Role Switch
  - [ ] Bottom navigation bar (fixed) with: Scanner tab, Supervisor tab
  - [ ] Routes: `/` → Scanner, `/supervisor` → Dashboard
  - [ ] Use TanStack Router or React Router v6.26+
  - [ ] Commit: `feat(ui): role-based navigation`

### Epic 4 – QR Code Generation Tool (Printable Checkpoints)

- [ ] PR 4.1 – QR Generation Page
  - [ ] Create `/tools/generate` route (protected by simple password or hidden)
  - [ ] Fetch all checkpoints
  - [ ] Generate printable A4 page: 4 QR codes per row with label below
  - [ ] Use `qrcode.react` with high error correction
  - [ ] Print stylesheet (@media print)
  - [ ] Commit: `feat(tools): printable checkpoint QR code generator`

### Epic 5 – Polish & Production Readiness

- [ ] PR 5.1 – Global Error Handling & Toast
  - [ ] React: sonner toast on API errors
  - [ ] Hono: centralized error handler → { error: string }
  - [ ] Commit: `refactor: unified error handling frontend + backend`

- [ ] PR 5.2 – Basic Offline Queue (Stretch but doable)
  - [ ] Use idb-keyval to queue failed scans
  - [ ] Background sync when online
  - [ ] Show "X pending" badge
  - [ ] Commit: `feat(offline): queue scans when no connection`

- [ ] PR 5.3 – Loading States & Skeletons
  - [ ] Scanner: skeleton overlay while camera loads
  - [ ] Dashboard table: shadcn Skeleton rows
  - [ ] Commit: `ui: add loading skeletons everywhere`

- [ ] PR 5.4 – Final Deployment & Smoke Test
  - [ ] Deploy to Cloudflare Pages (frontend)
  - [ ] Deploy Worker with D1 prod binding
  - [ ] Verify logs appear in prod D1
  - [ ] Test on real Android + iPhone devices
  - [ ] Add README with setup + QR printing instructions
  - [ ] Commit: `chore: ship v1.0.0`

### Final Ship Checklist (All must be checked Sunday night)

- [ ] Guard can scan QR → log appears instantly
- [ ] Supervisor can filter and view all logs
- [ ] Works offline (at least shows "queued")
- [ ] QR generation tool prints scannable codes
- [ ] Deployed and accessible via shareable link
- [ ] No console errors on mobile
- [ ] SOLID principles followed (repositories, services, DI)
