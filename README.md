# FreelanceOS

A full-stack freelancer management platform built with React, Vite, Express, and Prisma.

FreelanceOS gives freelancers and agency owners a single place to manage projects, clients, tasks, invoices, earnings, messages, team members, proposals, and activity history — all backed by a real database and protected by JWT authentication.

---

## Features

### Core
- **Authentication** — register, login, logout, JWT sessions, protected routes
- **Dashboard** — real metrics (earnings, active/completed/pending projects, clients, invoices, proposals), earnings chart, project status distribution, client revenue, monthly performance, deadlines & alerts, recent projects, transactions, messages, notifications, and activity
- **Projects** — CRUD with search, status filters, sorting, pagination; progress tracking; per-project task manager, transactions, and AI suggestions
- **Tasks** — create, update, reorder, assign to team members, mark complete; auto-syncs project progress
- **Clients** — CRUD with search, status filters, sorting, date filters, pagination; per-client activity timeline; CSV import/export
- **Invoices** — full line-item invoicing with PDF generation, status workflow (Draft → Sent → Paid / Overdue / Cancelled), stats dashboard
- **Earnings & Transactions** — period-filtered earnings chart, transaction CRUD, status tracking
- **Messages** — three-pane messenger with real conversations, AI reply drafting
- **Notifications** — type-filtered, read/unread filtering, mark read, mark all read, clear all, delete individual
- **Profile** — public profile page with real stats
- **Settings** — 7 working tabs: Personal Info, Account, Password & Security, Notifications, Appearance, Privacy, Connected Accounts

### Agency
- **Agency Dashboard** — team size, projects, clients, earnings, tasks, deadlines, team workload, monthly earnings, recent projects/transactions/notifications
- **Team Management** — add team members by email, update roles, remove members (unassigns their tasks)
- **Workload** — per-member task breakdown, overdue tasks, upcoming tasks, unassigned tasks pool
- **Agency Earnings** — agency-wide earnings, member performance, top clients, recent transactions

### AI Assistant (optional, no API key required)
- **Smart Suggestions** on project detail — analyzes project scope and suggests tasks, milestones, timeline, and budget insight
- **Add All** — creates suggested tasks with one click
- **AI Reply Assist** in messages — drafts professional/client-appropriate replies
- **Summarize** — extractive summarization of text

### Cross-cutting
- Role-based access control (freelancer vs agency)
- Application-wide search across projects, clients, invoices, messages, notifications
- TanStack Query for server state (loading, error, caching, invalidation)
- Dark/Light/System theme with full component coverage
- Responsive layout (desktop, tablet, mobile)
- Centralized error handling (validation, auth, not-found, database errors)
- Activity log tracking project created/updated, client added, invoice created/paid, task completed/assigned, message sent

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite 8, Tailwind CSS 4, React Router 7, TanStack Query 5 |
| Backend | Node.js, Express 5, JWT (jsonwebtoken), bcryptjs |
| Database | PostgreSQL (production) / SQLite (local dev), Prisma ORM 6 |
| PDF | pdfkit (invoice PDF generation) |
| CSV | csv-parse / csv-stringify (client import/export) |
| File upload | multer (multipart handling) |

---

## Architecture

```
client/
├── prisma/
│   ├── schema.prisma          # Data model (User, Agency, Client, Project, Task,
│   │                          #   Invoice, InvoiceItem, Transaction, Message,
│   │                          #   Conversation, Notification, ActivityLog, Proposal,
│   │                          #   AgencyMember, TeamMember)
│   ├── seed.cjs               # Development seed data
│   └── setup.cjs              # Database provisioning helper
├── server/                    # Express API (CommonJS, runs on Node)
│   ├── index.js               # App setup, middleware, route registration
│   ├── middleware/
│   │   ├── auth.js            # JWT verification + Prisma client + env
│   │   ├── errorHandler.js    # Centralized error middleware
│   │   └── rbac.js            # Role-based access utilities
│   ├── utils/
│   │   ├── AppError.js        # Safe error types (badRequest, unauthorized, notFound)
│   │   ├── asyncHandler.js    # try/catch wrapper for route handlers
│   │   └── validate.js        # Request validation helpers
│   ├── controllers/           # Business logic (auth, dashboard, projects, tasks,
│   │                          #   messages, earnings, profile, clients, invoices,
│   │                          #   notifications, agency, activity, proposals)
│   └── routes/                # Express routers (RESTful, resource-based)
├── src/                       # React frontend (ES modules)
│   ├── api/client.js          # Fetch-based API client with auth, error handling
│   ├── components/            # UI (icons, shared, ui primitives, layout, feature)
│   ├── context/               # Auth, Theme, Permissions, Toast, Search contexts
│   ├── hooks/                 # Custom hooks
│   ├── layouts/               # AppShell, Sidebar, Topbar
│   ├── pages/                 # Route components (one per feature)
│   ├── services/              # themeService
│   ├── utils/                 # Shared frontend utilities
│   ├── App.jsx                # Router + QueryClient + providers
│   └── main.jsx               # Entry point
├── vite.config.js             # Vite config (port 5173, /api proxy to :3001)
├── package.json               # Frontend deps + scripts
└── server/package.json        # Backend deps (separate, CommonJS)
```

### API design
- RESTful resource routes under `/api/`
- Stateless JWT authentication on every protected route
- Consistent JSON error responses with `error` + `code` fields
- Pagination on large-list endpoints (`page`, `limit`, `total`, `totalPages`)
- Search, filter, and sort query parameters where applicable
- Backend validates input; frontend also validates for UX

### Data model highlights
- **User** is the central entity; every resource (Project, Client, Task, Invoice, Transaction, Message, Notification, ActivityLog, Proposal) belongs to a user
- **Agency** is a first-class entity with `AgencyMember` relationships; agencies can have team members and shared projects
- **TeamMember** kept for backward compatibility (manager → user relationship)
- Foreign keys with `onDelete: Cascade` for user-owned data; `onDelete: SetNull` where a relationship can survive its parent (e.g., task assignee, project agency)
- Proper indexes on foreign keys and common query filters

---

## Installation

### Prerequisites
- Node.js 18+
- npm

### 1. Install dependencies

```bash
# Frontend + shared deps
npm install

# Backend deps (separate package.json in server/)
cd server && npm install && cd ..
```

### 2. Environment variables

Copy `.env.example` to `.env` and fill in your values:

```bash
cp .env.example .env
```

Edit `.env`:

```ini
# Database — PostgreSQL in production; SQLite for local dev
DATABASE_URL="file:./dev.db"

# Authentication — use a strong random secret in production
JWT_SECRET="your-production-jwt-secret-here"

# Server port
PORT=3001

# Frontend URL — used for CORS allowlist
CLIENT_URL="http://localhost:5173"
```

**Never commit `.env`.** It is listed in `.gitignore`.

### 3. Database setup

```bash
# Push the Prisma schema to the database (creates tables)
npm run db:push

# Generate the Prisma Client
npx prisma generate
```

For production with PostgreSQL, set `DATABASE_URL` to a PostgreSQL connection string and use Prisma migrations:

```bash
npx prisma migrate dev --name init    # development
npx prisma migrate deploy             # production (no seed)
```

### 4. Seed the database (development only)

```bash
npm run db:reset     # push schema + seed (fresh start)
# or
npm run db:seed      # seed without resetting
```

The seed creates realistic sample users, clients, projects, tasks, invoices, transactions, messages, notifications, and activity. Seed data is for development/testing only — the application reads all data through the API.

### 5. Run

```bash
# Backend (port 3001)
node server/index.js

# Frontend (port 5173) — in another terminal
npm run dev
```

Vite proxies `/api` requests to the backend, so the frontend calls `http://localhost:5173/api/...` which forwards to `http://localhost:3001/api/...`.

**Demo account** (after seeding): `zarnabfatima2005@gmail.com` / `password123`

Or start both in one terminal:

```bash
npm run dev:all
```

---

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `DATABASE_URL` | Yes | Database connection string. SQLite: `file:./dev.db`. PostgreSQL: `postgresql://user:pass@host:5432/dbname` |
| `JWT_SECRET` | Yes | Secret used to sign/verify JWTs. Use a strong random value in production. |
| `PORT` | No | Backend port. Default: `3001` |
| `CLIENT_URL` | No | Frontend origin used for CORS. Default: `http://localhost:5173` |

### Production PostgreSQL

For production, use a hosted PostgreSQL database (e.g., Neon, Supabase, AWS RDS, Render Postgres). Set `DATABASE_URL` to the connection string provided by your host. Run `npx prisma migrate deploy` on the production database (not `db:push`).

---

## Prisma Migrations

Development (creates migration + applies + seeds):

```bash
npx prisma migrate dev --name <description>
```

Production (applies pending migrations without creating new ones):

```bash
npx prisma migrate deploy
```

After changing `prisma/schema.prisma`:

```bash
npx prisma migrate dev --name <describe-change>
npx prisma generate
```

View the schema visually (optional):

```bash
npx prisma studio
```

---

## API Reference

Base URL: `http://localhost:3001/api` (dev). All protected endpoints require `Authorization: Bearer <token>`.

### Authentication

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/auth/register` | Register a new user. Body: `{ email, password, firstName, lastName, accountType? }`. Returns `{ user, token }`. |
| `POST` | `/auth/login` | Login. Body: `{ email, password }`. Returns `{ user, token }`. |
| `POST` | `/auth/logout` | Logout (clears client-side token; server is stateless). Returns `{ message }`. |
| `GET` | `/auth/me` | Get current user profile. Returns user object (no password). |
| `GET` | `/auth/permissions` | Get current user's permissions/roles. |

### Dashboard

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/dashboard` | Full dashboard data: stats, analytics (project status distribution, client revenue, monthly performance), deadlines (approaching, overdue projects, overdue tasks, urgent tasks), active projects list, recent projects, recent transactions, recent messages, recent notifications, pending proposals list, unread message count, monthly earnings. |

### Projects

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/projects` | List projects. Query: `status`, `search`, `sort`, `page`, `limit`. Returns `{ projects, pagination }`. |
| `GET` | `/projects/:id` | Get a single project with client, tasks, and transactions. |
| `POST` | `/projects` | Create a project. Body: `{ title, description?, budget?, deadline?, status?, clientId }`. Returns created project. |
| `PUT` | `/projects/:id` | Update a project. |
| `DELETE` | `/projects/:id` | Delete a project. |
| `POST` | `/projects/:id/tasks` | Add a task to a project. Body: `{ title, description?, status?, priority?, dueDate?, assigneeId? }`. |
| `PUT` | `/projects/:id/tasks/reorder` | Bulk update task statuses. Body: `{ taskUpdates: [{ id, status?, completed? }] }`. |
| `PUT` | `/tasks/:taskId` | Update a task. |
| `DELETE` | `/tasks/:taskId` | Delete a task. |

### Tasks (top-level)

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/tasks` | List tasks. Query: `status`, `search`, `sort`, `page`, `limit`. |
| `POST` | `/tasks` | Create a task. |

### Clients

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/clients` | List clients. Query: `search`, `status`, `sort`, `createdAfter`. Returns array of clients with stats. |
| `GET` | `/clients/:id` | Get a single client with projects, transactions, messages, and stats. |
| `GET` | `/clients/:id/activity` | Aggregated activity timeline for a client. |
| `POST` | `/clients` | Create a client. Body: `{ name, email?, phone?, role?, org?, location?, status?, notes? }`. |
| `PUT` | `/clients/:id` | Update a client. |
| `DELETE` | `/clients/:id` | Delete a client. |
| `GET` | `/clients/csv/export` | Export clients as CSV (returns file download). |
| `POST` | `/clients/csv/import` | Import clients from CSV. Body: CSV file content (`text/csv`). |

### Invoices

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/invoices` | List invoices. Query: `status`, `search`, `page`, `limit`. Returns `{ invoices, stats, pagination }`. |
| `GET` | `/invoices/:id` | Get a single invoice with client, project, and line items. |
| `POST` | `/invoices` | Create an invoice. Body: `{ clientId, projectId?, dueDate?, notes?, tax?, items: [{ description, quantity, unitPrice }] }`. Returns created invoice. |
| `PUT` | `/invoices/:id` | Update an invoice (including replacing line items). |
| `PUT` | `/invoices/:id/status` | Quick status update. Body: `{ status: 'Draft' | 'Sent' | 'Paid' | 'Overdue' | 'Cancelled' }`. |
| `DELETE` | `/invoices/:id` | Delete an invoice. |
| `GET` | `/invoices/:id/pdf` | Download invoice as PDF (returns file download). |

### Earnings / Transactions

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/earnings` | Earnings data. Query: `period` (1D/1W/1M/6M/1Y/ALL), `status`, `search`, `page`, `limit`. Returns `{ stats, transactions, pagination, monthlyData }`. |
| `GET` | `/transactions` | Alias for `/earnings` (RESTful). |
| `POST` | `/earnings` | Create a transaction. Body: `{ invoice, amount, date?, method?, status?, clientId?, projectId? }`. |
| `PUT` | `/earnings/:id` | Update a transaction. |
| `DELETE` | `/earnings/:id` | Delete a transaction. |

### Messages

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/messages` | List conversations (clients with last message + unread count). |
| `GET` | `/messages/:clientId` | Get messages for a conversation. Marks unread messages as read. |
| `POST` | `/messages` | Send a message. Body: `{ clientId, text }`. |

### Notifications

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/notifications` | List notifications. Query: `page`, `limit`, `type`, `read`, `search`. Returns `{ notifications, unreadCount, pagination }`. |
| `GET` | `/notifications/unread-count` | Get unread notification count (lightweight). |
| `PUT` | `/notifications/:id/read` | Mark a notification as read. |
| `PUT` | `/notifications/read-all` | Mark all notifications as read. |
| `DELETE` | `/notifications/:id` | Delete a notification. |
| `DELETE` | `/notifications/clear` | Clear all notifications. |

### Profile

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/profile` | Get full profile (user + extended fields). |
| `PUT` | `/profile` | Update profile. Body: profile fields. |
| `GET` | `/users/me` | Alias for `/auth/me` (RESTful). |
| `PUT` | `/users/me` | Alias for profile update (RESTful). |

### Activity

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/activity` | Recent activity log. Query: `limit`. Returns `{ activities }`. |

### Proposals

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/proposals` | List proposals. Query: `status`, `search`, `sort`, `page`, `limit`. Returns `{ proposals, pagination }`. |
| `GET` | `/proposals/stats` | Proposal statistics (total, draft, sent, accepted, rejected, expired, values). |
| `GET` | `/proposals/:id` | Get a single proposal with client and project. |
| `POST` | `/proposals` | Create a proposal. Body: `{ title, description?, price?, clientId, projectId?, expirationDate?, status? }`. |
| `PUT` | `/proposals/:id` | Update a proposal. |
| `PUT` | `/proposals/:id/status` | Update proposal status. Body: `{ status: 'Draft' | 'Sent' | 'Accepted' | 'Rejected' | 'Expired' }`. |
| `DELETE` | `/proposals/:id` | Delete a proposal. |

### Agency

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/agency/dashboard` | Agency dashboard: stats, team workload, deadlines, monthly earnings, recent projects/transactions/notifications. |
| `GET` | `/agency/team` | List team members with stats (task counts, project counts). |
| `POST` | `/agency/team` | Add a team member by email. Body: `{ email, role? }`. |
| `PUT` | `/agency/team/:memberId` | Update a team member's role. Body: `{ role }`. |
| `DELETE` | `/agency/team/:memberId` | Remove a team member (unassigns their tasks). |
| `GET` | `/agency/workload` | Team workload overview: per-member task breakdown, overdue/upcoming tasks, unassigned tasks, owner projects. |
| `GET` | `/agency/earnings` | Agency-wide earnings: stats, monthly data, top clients, member performance, recent transactions. |
| `GET` | `/agency/clients` | Agency-scoped client list with project/transaction stats. |

### AI Assistant (optional)

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/ai/suggest-tasks` | Analyze a project and suggest tasks, milestones, timeline, and budget insight. Body: `{ title, description?, budget?, status?, clientId? }`. Returns `{ suggestions }`. No API key required — uses a local rule-based engine. |
| `POST` | `/ai/summarize` | Summarize text. Body: `{ text, maxLength? }`. Returns `{ summary }`. |
| `POST` | `/ai/draft-reply` | Draft a professional reply to a message. Body: `{ lastMessage, context?, tone? }`. Returns `{ draft }`. |

### Error handling
All errors return JSON:
```json
{ "error": "Human-readable message", "code": "ERROR_CODE" }
```

Common codes: `BAD_REQUEST`, `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `CONFLICT`, `VALIDATION_ERROR`, `DATABASE_ERROR`, `NETWORK_ERROR`.

Server errors never expose stack traces or database internals to the client.

---

## Project Structure

```
client/
├── prisma/
│   ├── schema.prisma
│   ├── seed.cjs
│   └── setup.cjs
├── server/
│   ├── index.js
│   ├── package.json
│   ├── controllers/
│   ├── middleware/
│   ├── routes/
│   └── utils/
├── src/
│   ├── api/client.js
│   ├── components/
│   ├── context/
│   ├── hooks/
│   ├── layouts/
│   ├── pages/
│   ├── services/
│   └── utils/
├── .env
├── .env.example
├── .gitignore
├── index.html
├── vite.config.js
├── package.json
└── README.md
```

---

## Scripts

| Script | Description |
|---|---|
| `npm run dev` | Start Vite dev server on port 5173 |
| `npm run dev:server` | Start Express backend on port 3001 |
| `npm run dev:all` | Start both frontend and backend concurrently |
| `npm run build` | Production build to `dist/` |
| `npm run preview` | Preview the production build locally |
| `npm run lint` | Run oxlint |
| `npm run db:push` | Push Prisma schema to database |
| `npm run db:seed` | Run seed script |
| `npm run db:reset` | Push schema + seed (fresh start) |

---

## Deployment

### Frontend (standalone)

1. Set `CLIENT_URL` to your production frontend URL in the backend's environment.
2. Build the frontend:
   ```bash
   npm run build
   ```
3. The production build output is in `dist/`. Serve it with any static host (Vercel, Netlify, Cloudflare Pages, GitHub Pages, Nginx, etc.):
   ```bash
   npm run preview   # local preview of the production build
   ```

The frontend must be able to reach the backend API. In production, configure the API base URL. The current setup proxies `/api` through Vite in development — for production deployment, either:
- Serve the frontend and backend from the same origin (backend serves `dist/`), or
- Point the API client to the backend URL directly (update `src/api/client.js` `API_BASE` for production), or
- Use a reverse proxy (Nginx, etc.) that routes `/api` to the backend and serves `dist/` for everything else.

### Backend

1. Set production environment variables (`DATABASE_URL`, `JWT_SECRET`, `PORT`, `CLIENT_URL`).
2. Use a hosted PostgreSQL database. Run `npx prisma migrate deploy` to apply migrations.
3. Start the server:
   ```bash
   node server/index.js
   ```
4. For process management in production, use a process manager (PM2, systemd, Docker, etc.).

### Environment-specific configuration

- `.env` — local development (gitignored)
- `.env.example` — template with placeholder values (committed)
- Production: set environment variables through your hosting platform's environment configuration (never commit `.env` with real secrets)

### CORS
The backend restricts CORS to `CLIENT_URL`. Make sure `CLIENT_URL` matches your production frontend origin exactly, otherwise the browser will block cross-origin requests.

---

## Project Paper

A presentation-ready project paper is in [PROJECT_PAPER.md](PROJECT_PAPER.md). It includes a short presentation brief (talking points) and a full write-up of the product, architecture, data flow, design approach, and engineering quality.

## License

MIT
