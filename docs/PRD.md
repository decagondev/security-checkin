# Product Requirements Document (PRD)  
**Product Name:** GuardPoint – QR-Code Based Security Patrol Check-in System  
**Target Ship Date:** End of week (Sunday, December 14, 2025)  
**Core Value Proposition:** Enable security guards to instantly log patrol checkpoints via QR code scan using a mobile-friendly progressive web app, with real-time supervisor visibility.

### Tech Stack (Fixed & Final)
| Layer               | Technology                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| Frontend            | React 19 + Vite + TypeScript                                               |
| Styling             | Tailwind CSS v4 + shadcn/ui (already configured)                           |
| Backend             | Hono (Cloudflare Workers)                                                  |
| Database            | Cloudflare D1 (SQLite at the edge)                                         |
| Authentication      | None for MVP (guard ID comes from QR code or manual entry)                |
| Deployment          | Cloudflare Pages + Workers                                                 |
| Base Repository     | https://github.com/decagondev/security-checkin (forked & ready)           |

---

## High-Level Epics (MVP Scope – Ship by Sunday)

### Epic 0 – AI-Assisted Development Foundation (Memory Bank + Cursor Rules)
**Goal:** Set up local AI context so future development is 5-10× faster.

| PR # | Title                              | Commits / Subtasks                                                                                     | Owner | Est. |
|------|------------------------------------|-------------------------------------------------------------------------------------------------------|-------|------|
| 0.1  | Initialize Cursor memory bank     | • Create `.cursor/` folder<br>• Add `.cursor/rules.md` with project context, SOLID guidelines, naming conventions<br>• Add `.cursor/memory-bank.md` with this PRD summary<br>• Add `.cursor/` to `.gitignore` | Lead Dev | 0.5h |

---

### Epic 1 – Database Schema & Migrations
**Goal:** Persistent, queryable patrol logs in D1.

| PR # | Title                              | Commits / Subtasks                                                                                     | Owner | Est. |
|------|------------------------------------|-------------------------------------------------------------------------------------------------------|-------|------|
| 1.1  | Create D1 database & migrations   | • `npx wrangler d1 create guardpoint-db`<br>• Add `schema.sql` with tables: `guards`, `checkpoints`, `patrol_logs`<br>• Migration: guards (id PK, name, badge_number)<br>• checkpoints (id PK, name, location)<br>• patrol_logs (id PK, guard_id FK, checkpoint_id FK, scanned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP)<br>• Add seed data (3 guards, 10 checkpoints) | Backend | 2h |
| 1.2  | Wrangler + Hono D1 binding        | • Add binding in `wrangler.toml`<br>• Create `src/db/index.ts` with typed Drizzle or raw queries wrapper (SOLID – Repository pattern) | Backend | 1h |

---

### Epic 2 – Backend API (Hono on Workers)
**Goal:** Clean, modular, typed API following SOLID principles.

Folder structure: `src/api/routes/v1/*.ts`, `src/api/middleware/`, `src/core/services/`, `src/core/repositories/`

| PR # | Title                                    | Commits / Subtasks                                                                                           | Owner | Est. |
|------|------------------------------------------|-------------------------------------------------------------------------------------------------------------|-------|------|
| 2.1  | API skeleton & health check              | • `/health` → 200 OK<br>• `/api/v1/` base route                                                             | Backend | 0.5h |
| 2.2  | POST /api/v1/logs (scan endpoint)        | • Accept JSON { qrData: string } (contains guardId|checkpointId separated by pipe or JSON)<br>• Validate & parse<br>• Insert into patrol_logs<br>• Return 201 with log ID & timestamp<br>• Input validation middleware (Zod) | Backend | 2.5h |
| 2.3  | GET /api/v1/logs (supervisor dashboard)  | • Optional filters: ?guardId=&dateFrom=&dateTo=&checkpointId=<br>• Pagination (limit/offset)<br>• Return rich objects with guard name + checkpoint name | Backend | 2h |
| 2.4  | GET /api/v1/checkpoints & /guards        | • Simple lists for dropdowns/seeding                                                                       | Backend | 1h |

---

### Epic 3 – Frontend Core (React + shadcn + Tailwind 4)
**Goal:** Mobile-first PWA that works perfectly on Android/iOS Chrome.

| PR # | Title                                  | Commits / Subtasks                                                                                                 | Owner     | Est. |
|------|----------------------------------------|-------------------------------------------------------------------------------------------------------------------|-----------|------|
| 3.1  | Mobile-first layout & PWA manifest     | • Update `index.html` viewport & manifest<br>• Add install prompt component<br>• Responsive container max-w-md mx-auto | Frontend  | 1h   |
| 3.2  | QR/Barcode Scanner Page                | • Use `@zxing/browser` (lightweight, works with rear camera)<br>• Full-screen scanner with overlay<br>• Auto-submit on successful read<br>• Fallback manual entry (text field + button)<br>• Success toast + vibration + beep | Frontend  | 4h   |
| 3.3  | Scan Success / Confirmation Screen     | • Large checkmark<br>• Show guard name, checkpoint name, exact time<br>• "Scan next" button                          | Frontend  | 1h   |
| 3.4  | Supervisor Dashboard                   | • Date picker (shadcn Calendar)<br>• Filters (guard, checkpoint)<br>• Data table (shadcn Table with sorting)<br>• Real-time refresh every 15s or manual pull-to-refresh<br>• Export CSV button | Frontend  | 5h   |
| 3.5  | Navigation & Role Switch               | • Bottom nav: Scanner (guard mode) ↔ Dashboard (supervisor mode)<br>• Simple toggle or separate routes `/` vs `/supervisor` | Frontend  | 1h   |

---

### Epic 4 – QR Code Generation Tool (Bonus but in scope)
**Goal:** Supervisors can print/generate checkpoint QR codes.

| PR # | Title                         | Commits / Subtasks                                                                | Owner | Est. |
|------|-------------------------------|----------------------------------------------------------------------------------|-------|------|
| 4.1  | /tools/generate-qr page       | • List all checkpoints<br>• Generate printable page (4 QR per A4)<br>• Use qrcode.react | Frontend | 2h |

---

### Epic 5 – Polish, Error Handling & Production Readiness

| PR # | Title                              | Commits / Subtasks                                                                                 | Owner | Est. |
|------|------------------------------------|---------------------------------------------------------------------------------------------------|-------|------|
| 5.1  | Global error handling & toasts     | • Hono error handler → JSON errors<br>• React ErrorBoundary + toast on API failures              | Both  | 1h   |
| 5.2  | Offline support (basic)            | • Service worker caches static assets<br>• Queue scans when offline (IndexedDB) & sync on reconnect (stretch) | Frontend | 2h |
| 5.3  | Loading states & skeletons         | • All tables and scanner have nice skeleton loaders                                              | Frontend | 1h   |
| 5.4  | Final testing & deployment         | • Local end-to-end test<br>• Deploy to Cloudflare Pages + Workers<br>• Verify D1 prod binding     | Lead  | 2h   |

---

## Final Delivery Breakdown (By End of Sunday)

| Day       | Focus                                      | PRs to Merge                          |
|-----------|--------------------------------------------|---------------------------------------|
| Thursday  | Epic 0 + Epic 1                            | 0.1, 1.1, 1.2                         |
| Friday    | Epic 2 (entire backend API)                | 2.1 → 2.4                             |
| Saturday  | Epic 3 (Scanner + basic UI) + Epic 4       | 3.1 → 3.5, 4.1                        |
| Sunday    | Epic 5 + bug fixes + deploy                | 5.1 → 5.4                             |

Total estimated effort: ~32–36 hours → feasible for 1 senior full-stack dev with AI assistance.

---

## Non-Functional Requirements
- 99.9% uptime (Cloudflare edge)
- < 500ms scan-to-log latency globally
- Works offline for at least 50 scans (stretch)
- No auth for MVP (QR contains guard ID)
- All code must follow SOLID:
  - Single Responsibility (services, repositories, routes separated)
  - Open/Closed (use interfaces for repository)
  - Dependency Inversion (inject DB client)

---

**Ship Criteria (Must be Green Sunday Night)**
1. Guard can scan QR → log appears instantly
2. Supervisor opens /supervisor → sees table with latest 50 logs, filterable
3. Works on Android Chrome + iPhone Safari
4. QR generation tool prints valid codes
5. Deployed to custom domain or *.workers.dev
