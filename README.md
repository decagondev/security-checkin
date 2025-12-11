# GuardPoint – QR-Code Security Patrol Check-in System  
**Instant, real-time patrol logging for security teams – fully mobile-responsive**

Live Demo: https://security-checkin.decagondev.workers.dev  
(Deploy your own instance in under 2 minutes)

## Features

- Scan any QR/barcode at a checkpoint → instantly logs guard ID, checkpoint, and exact timestamp  
- Works perfectly on mobile browsers (Chrome, Safari) – no app install required  
- Fully responsive design – desktop, tablet, phone optimized  
- Real-time supervisor dashboard with filtering, sorting, and CSV export  
- Built-in printable QR code generator (4 per A4 page)  
- Zero authentication (MVP) – guard identity embedded in QR code  
- Powered by Cloudflare edge (Workers + D1 + Pages) → global sub-500ms latency  

## Tech Stack

| Layer            | Technology                                      |
|------------------|--------------------------------------------------|
| Frontend         | React 19 + Vite + TypeScript + Tailwind CSS v4 + shadcn/ui |
| Styling          | Fully mobile-responsive (no PWA manifest)       |
| Backend          | Hono on Cloudflare Workers                      |
| Database         | Cloudflare D1 (edge SQLite)                     |
| QR/Barcode Scan  | ZXing Browser (pure web, rear camera support)   |
| Deployment       | Cloudflare Pages + Workers (zero server ops)    |

Architecture follows SOLID principles with full separation of concerns: repositories, services, routes, and UI components.

## Quick Start

### Option 1: One-Click Deploy to Cloudflare (Recommended)

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/decagondev/security-checkin)

1. Click the button above  
2. Log in to Cloudflare  
3. Cloudflare automatically:
   - Forks this repo
   - Creates a D1 database
   - Deploys the React frontend via Pages
   - Deploys the Hono API via Workers
4. Your secure, private instance is live instantly!

### Option 2: Local Development (npm only)

```bash
# 1. Clone the repository
git clone https://github.com/decagondev/security-checkin.git
cd security-checkin

# 2. Install dependencies (npm only)
npm install

# 3. Set up local D1 database
npm run db:create      # creates guardpoint_db locally
npm run db:migrate     # applies schema + migrations
npm run db:seed        # adds 3 sample guards + 10 checkpoints

# 4. Start development servers
npm run dev
```

Open http://localhost:8788

## Usage

### For Security Guards
1. Open the link on any phone browser
2. Tap **Scanner** tab
3. Point camera at checkpoint QR code
4. Log recorded instantly (works even with poor connectivity)

### For Supervisors
1. Open the same link → go to **Supervisor** tab
2. View all patrol logs in real time
3. Filter by guard, checkpoint, or date
4. Export full report as CSV

### Print Checkpoint QR Codes
1. Visit `/tools/generate` (hidden admin route)
2. Click **Print** → generates clean A4 sheets (4 QR codes per page)

**Supported QR formats** (both work):
```json
{"g": 3, "c": 7}     // Recommended: guard ID 3, checkpoint ID 7
```
or legacy:
```
3|7
```

## Project Structure (Clean & SOLID)

```
src/
├── api/routes/v1/          # Hono API route handlers
├── core/
│   ├── repositories/       # D1 data access (interfaces + implementations)
│   └── services/           # Business logic (dependency injected)
├── pages/                  # React pages (Scanner, Dashboard, Tools)
├── components/ui/          # shadcn/ui + custom components
├── layouts/                # Responsive mobile-first layout
└── db/                     # D1 schema, migrations, seed data
```

## npm Scripts

```bash
npm run dev          # Start Vite (frontend) + Worker (backend) in parallel
npm run db:create    # Create local D1 database
npm run db:migrate   # Apply migrations
npm run db:seed      # Insert sample guards & checkpoints
npm run build        # Build for production
npm run preview      # Preview production build locally
```

## Why No PWA (by design)

- Many enterprise environments block "Add to Home Screen"
- Full compatibility with server-side Workers and future auth systems
- Simpler deployment and caching strategy
- Still feels native on mobile thanks to responsive Tailwind + shadcn design

## Contributing

1. Fork the repo
2. Create a feature branch (`git checkout -b feature/your-idea`)
3. Use Conventional Commits
4. Open a Pull Request

We strictly enforce SOLID principles and modular, testable design.

## License

MIT © Decagon Dev

---

**Built in one week. Runs forever on the edge.**

Questions? Open an issue or reach out on X → @decagondev

Made with speed, precision, and zero server management.
