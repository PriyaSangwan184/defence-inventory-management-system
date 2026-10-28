# Defence Inventory Management System (DIMS)

DIMS is a fictional, local-first Defence Inventory Management System built as an academic and portfolio project. It models quantity-based stock operations for a defence organization using fictional data only; it does not contain classified, operational, or real-world military data.

The system is being built incrementally as a secure modular monolith. Its goals are reliable stock accounting, traceable transactions, role-based access control, and a maintainable React and Node.js codebase.

## Current progress

**Phase 1 is complete.** The frontend and backend scaffolds, environment configuration, health endpoint, security middleware, build verification, and Git ignore rules are in place.

**Phases 2 and 3 are partially implemented.** The normalized Prisma data model, database/accounting documentation, Prisma client generation, and authentication/RBAC foundation are implemented. Their database-backed verification is pending local MySQL configuration and the first migration.

| Phase | Status | Notes |
| --- | --- | --- |
| 1. Initial project setup | Complete | React/Vite, Express, TypeScript, Tailwind, health endpoint, builds. |
| 2. Requirements, database, Prisma | In progress | Schema and documentation are complete; migration and seed execution need local MySQL credentials. |
| 3. Authentication and RBAC | In progress | Secure session, Argon2id password hashing, login/logout/me routes, permission middleware, and seed are implemented. |
| 4–13. Core inventory through delivery | Not started | Implemented incrementally after the database is running. |
| 14. Optional extensions | Deferred | Docker, cloud deployment, barcode support, and other non-core work. |

There are **13 required delivery phases** and one optional extension phase. **1 of 13 is fully complete**; two additional phases have their foundations in place. Most feature work—inventory workflows, stock ledger posting, requisitions, reports, frontend integration, tests, and documentation—remains. No business workflow is described as complete before it is database-backed and tested.

## Technology stack

- **Frontend:** React, TypeScript, Vite, Tailwind CSS
- **Backend:** Node.js, Express, TypeScript, Zod, Pino, Helmet, CORS, rate limiting
- **Database:** MySQL 8+, Prisma ORM
- **Security foundation:** Argon2id password hashes, server-side revocable sessions, HttpOnly cookies, role/permission checks, audit events
- **Testing:** Vitest and Supertest are installed; workflow tests are added with their corresponding modules

## Project structure

```text
defence-inventory-management-system/
├── frontend/                 # React/Vite browser application
│   ├── src/                  # Pages, components, routes, services, and UI utilities
│   ├── postcss.config.js     # Frontend-local Tailwind/PostCSS configuration
│   └── tailwind.config.js
├── backend/                  # Express API and Prisma application
│   ├── prisma/
│   │   ├── schema.prisma     # Normalized MySQL data model
│   │   └── seed.ts           # Fictional development roles and administrator
│   └── src/
│       ├── config/           # Prisma client configuration
│       ├── middleware/       # Authentication and permission middleware
│       ├── modules/auth/     # Login, logout, sessions, password utilities
│       ├── app.ts            # Express middleware and API route registration
│       └── server.ts         # API process entry point
├── docs/                     # Architecture, ERD, data dictionary, accounting, local setup
├── package.json              # Shared frontend tooling dependencies
└── README.md
```

### Implemented database design

The Prisma schema includes users, roles, permissions, organization and warehouse scopes, categories, items, bins, stock balances, suppliers, receipts, requisitions, approval cycles, reservations, stock transaction headers, movement lines, transfers, counts, notifications, audit logs, and sessions.

All inventory quantities use exact `Decimal(18,3)` fields. `available_qty` is always derived as `on_hand_qty - reserved_qty`; it is not stored as an independently mutable value. Historical relationships use restrictive foreign keys so records are deactivated or reversed rather than silently deleted.

Read [architecture.md](docs/architecture.md), [erd.md](docs/erd.md), [data-dictionary.md](docs/data-dictionary.md), and [stock-accounting-spec.md](docs/stock-accounting-spec.md) for the current design decisions.

## Prerequisites

- Fedora Linux or another supported Linux distribution
- Node.js 22 or newer
- npm 10 or newer
- MySQL 8 or newer

Verify your local tools:

```bash
node --version
npm --version
mysql --version
```

## Setup

Run all commands below from the repository root unless a command says otherwise.

### 1. Install dependencies

```bash
npm install
cd frontend && npm install
cd ../backend && npm install
```

The root install supplies the shared Tailwind/PostCSS tooling. The frontend and backend each retain their own lockfile and application dependencies.

### 2. Create the local MySQL database

Open MySQL using an account permitted to create a local development database and user:

```bash
mysql -u root -p
```

Then execute the following SQL, replacing the example password with your own local development password:

```sql
CREATE DATABASE dims CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'dims_user'@'localhost' IDENTIFIED BY 'replace-this-local-password';
GRANT ALL PRIVILEGES ON dims.* TO 'dims_user'@'localhost';
FLUSH PRIVILEGES;
```

### 3. Configure backend environment variables

Create the ignored local environment file:

```bash
cd backend
cp .env.example .env
```

Edit `backend/.env` and replace the placeholder database URL:

```dotenv
NODE_ENV=development
PORT=3000
FRONTEND_URL=http://localhost:5173
SESSION_TTL_HOURS=8
DATABASE_URL="mysql://dims_user:replace-this-local-password@127.0.0.1:3306/dims"
```

Never commit `.env`, real passwords, session data, or database URLs. See [local-environment.md](docs/local-environment.md) for the same setup details.

### 4. Create the schema and seed development data

```bash
cd backend
npm run db:migrate -- --name init
npm run db:seed
```

The seed creates fictional baseline roles and a development-only administrator account:

```text
Username: admin
Password: ChangeMe123!
```

Change this password immediately once account-management screens are available. Do not use this credential outside local development.

## Run the project

Start the API and frontend in two separate terminals.

Terminal 1 — backend API:

```bash
cd backend
npm run dev
```

Terminal 2 — frontend:

```bash
cd frontend
npm run dev
```

Open `http://localhost:5173` in a browser. The API listens on `http://127.0.0.1:3000` and exposes the currently available health endpoint:

```bash
curl http://127.0.0.1:3000/api/v1/health
```

The authentication API foundation provides:

```text
POST /api/v1/auth/login
POST /api/v1/auth/logout
GET  /api/v1/auth/me
```

The frontend authentication screens and the remaining business APIs are scheduled for later phases.

## Build and verify

```bash
cd frontend && npm run build
cd ../backend && npm run db:generate && npm run build
```

Other useful backend commands:

```bash
cd backend
npm run db:migrate  # Create/apply a Prisma migration in local development
npm run db:seed     # Seed fictional local development data
npm test            # Run tests as they are added with each feature module
```

## Security and operational notes

- Passwords are hashed using Argon2id; raw passwords are never stored.
- Browser sessions are random opaque tokens; only SHA-256 token hashes are persisted.
- Session cookies are `HttpOnly`, `SameSite=Lax`, and use `Secure` in production.
- Authentication and global API requests are rate limited.
- Permissions are evaluated by the backend; hiding a frontend control is never treated as authorization.
- Stock-affecting workflows are not yet implemented. When added, they must use Prisma transactions, idempotency keys, ledger rows, audit events, and stock-balance invariants described in the accounting specification.

## Next implementation milestones

1. Apply and verify the initial MySQL migration and development seed.
2. Complete account administration, role assignment, scopes, and authentication tests.
3. Build inventory, category, warehouse, bin, and stock-balance modules.
4. Build the stock ledger and goods-receipt posting flow.
5. Add requisitions, approvals, reservations, and stock issues.
6. Add transfers, adjustments, stock counts, dashboard/reporting, notifications, frontend integration, and full workflow testing.
