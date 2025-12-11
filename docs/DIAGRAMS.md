Here’s a complete set of **Mermaid diagrams** that fully describe **GuardPoint** — ready to paste into Markdown, Obsidian, Cursor, or Notion.

### 1. High-Level Product Architecture (C4-style Context + Containers)

```mermaid
flowchart TB
    subgraph MobileDevice["Mobile Browser (PWA)"]
        A["Guard Scanner Page"]
        B["Supervisor Dashboard"]
        C["QR Generator Tool"]
    end

    subgraph CloudflareEdge["Cloudflare Edge"]
        D["Cloudflare Pages<br/>(React + Vite + Tailwind + shadcn)"]
        E["Cloudflare Worker<br/>(Hono API)"]
        F["Cloudflare D1<br/>(SQLite @ Edge)"]
    end

    A -->|"HTTPS"| E
    B -->|"HTTPS"| E
    C -->|"HTTPS"| E
    E -->|"SQL"| F

    click A "/scanner"
    click B "/supervisor"
    click C "/tools/generate"

    style MobileDevice fill:#e0f2fe
    style CloudflareEdge fill:#fbbf24,color:black
```

### 2. User Flow – Security Guard (Primary Happy Path)

```mermaid
flowchart TD
    Start["Open App / PWA"] --> A["Home → Scanner Tab"]
    A --> B["Camera Permission Granted?"]
    B -->|"Yes"| C["Live QR Scanner Active"]
    B -->|"No"| D["Show Manual Entry Field"]
    C --> E["QR Detected & Decoded"]
    D --> E
    E --> F["POST /api/v1/logs"]
    F -->|"201"| G["Success Screen<br/>✓ Checkpoint Logged<br/>Guard: John D.<br/>Location: Gate 3<br/>Time: 14:23"]
    G --> H{"Vibrate + Beep"}
    H --> I["Auto redirect in 3s<br/>or tap 'Next Checkpoint'"]
    I --> C

    style G fill:#dcfce7,color:#166534
    style F fill:#bfdbfe
```

### 3. User Flow – Security Supervisor

```mermaid
flowchart TD
    S[Open App → Supervisor Tab] --> A[Dashboard Loads Latest 50 Logs]
    A --> B[Optional: Apply Filters<br/>• Date Range<br/>• Guard<br/>• Checkpoint]
    B --> C[GET /api/v1/logs?filters...]
    C --> D[Data Table Renders<br/>with Sorting & Pagination]
    D --> E[Export CSV]
    D --> F[Pull to Refresh]
    F --> C

    style D fill:#f0f9ff
```

### 4. QR Code Data Flow & Format

```mermaid
flowchart LR
    subgraph PhysicalWorld["Physical World"]
        QR["Printed QR Code<br/>Data: {g:3,c:7}<br/>or legacy: 3|7"]
    end

    subgraph Scanner
        ZX["ZXing Browser"] --> Decode["Decode → JSON or pipe format"]
    end

    subgraph APILayer["API Layer"]
        Decode --> Validate["Zod Schema"]
        Validate --> Service["PatrolLogService.create()"]
        Service --> Repo["PatrolLogRepository.insert()"]
        Repo --> D1[("D1 Database")]
    end

    QR --> ZX
    style QR fill:#e0e7ff
    style D1 fill:#fecaca
```

### 5. Backend API Routes (Hono)

```mermaid
flowchart LR
    H["Hono App"] --> v1["v1 Router"]

    subgraph GET
        v1 --> g1["/api/v1/logs"]
        v1 --> g2["/api/v1/guards"]
        v1 --> g3["/api/v1/checkpoints"]
        v1 --> g4["/health"]
    end

    subgraph POST
        v1 --> p1["/api/v1/logs"]
    end

    p1 --> PatrolService["PatrolLogService"]
    g1 --> PatrolService
    PatrolService --> Repo[("Repositories → D1")]

    style p1 fill:#bbf7d0
    style g1 fill:#bfdbfe
```

### 6. Folder Structure (Modular & SOLID-compliant)

```bash
src/
├── api/
│   └── routes/v1/
│       ├── logs/
│       │   ├── post-log.ts
│       │   └── get-logs.ts
│       ├── guards.ts
│       └── checkpoints.ts
├── core/
│   ├── repositories/
│   │   ├── PatrolLogRepository.ts
│   │   ├── GuardRepository.ts
│   │   └── CheckpointRepository.ts
│   ├── services/
│   │   └── PatrolLogService.ts
│   └── types/
├── db/
│   ├── d1-client.ts
│   └── schema.ts
├── pages/
│   ├── Scanner.tsx
│   ├── Success.tsx
│   ├── supervisor/Dashboard.tsx
│   └── tools/GenerateQR.tsx
├── components/
│   ├── ui/ (shadcn)
│   ├── ScannerOverlay.tsx
│   └── BottomNav.tsx
└── layouts/MobileLayout.tsx
```

### 7. Offline Queue Flow (Bonus Feature)

```mermaid
sequenceDiagram
    participant Guard
    participant App
    participant IndexedDB
    participant API

    Guard->>App: Scan QR (offline)
    App->>IndexedDB: Queue log
    App->>Guard: Show "Queued (1)"
    Note over App,IndexedDB: Background sync registered

    loop Every 10s when online
        App->>IndexedDB: Get queued items
        IndexedDB-->>App: Return logs
        App->>API: POST /api/v1/logs
        alt Success
            API-->>App: 201
            App->>IndexedDB: Remove from queue
            App->>Guard: Toast "Synced!"
        end
    end
```

### 8. Deployment & CI/CD (Cloudflare)

```mermaid

graph TD
    A["GitHub Repo"] --> B{"git push"}
    B --> C["Cloudflare Pages Build<br/>(React → static)"]
    B --> D["Cloudflare Workers Deploy<br/>(Hono + D1 binding)"]
    C --> E["https://guardpoint.pages.dev"]
    D --> F["API: https://worker.project.workers.dev"]
    F --> G[("D1 Prod Database")]

    style E fill:#10b981,color:white
    style F fill:#8b5cf6,color:white
```
