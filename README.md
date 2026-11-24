# College Leave Management System (CLMS)

A full-stack College Leave Management System for a four-role college workflow: **Administrator**, **Head of Department (HOD)**, **Class Tutor**, and **Parent / Guardian**.

Architecture: **MERN stack** (MongoDB, Express, React, Node.js) with Socket.IO for real-time notifications.

- **Backend** (`backend/`): Express 5 + Mongoose 9 + MongoDB + JWT access/refresh tokens + Socket.IO.
- **Frontend** (`frontend/`): React 18 + Vite 5 + React Router 6 + Tailwind CSS + React Hook Form + Zod + Recharts + jsPDF + socket.io-client.

Two deploy modes:
1. **Backend mode** (default): Real REST API + MongoDB; Socket.IO pushes live notifications.
2. **Demo mode**: Browser `localStorage` fallback (`demoStore.js`) when no backend token is present.

> This is an independent academic prototype. The visual design is inspired by the Thiagarajar College of Engineering (TCE) design language, but the project is not affiliated with or endorsed by the college. Do not use it for real administrative decisions.

---

## Tech stack

### Frontend (`frontend/`)

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Framework | React 18 | UI library |
| Build tool | Vite 5 | Dev server + production build |
| Routing | React Router 6 | Role-based protected routes |
| Styling | Tailwind CSS + PostCSS | Utility-first CSS |
| Forms | React Hook Form + Zod | Form state + schema validation |
| Charts | Recharts | Dashboard analytics |
| Icons | Lucide React | Iconography |
| Notifications | React Hot Toast | Toast alerts |
| PDF | jsPDF | Leave letter generation |
| HTTP client | Axios | API calls with auth retry / refresh |
| Real-time | socket.io-client | Live notification push |
| Testing | Node built-in test runner | Unit / integration tests |
| Linting | ESLint 9 | Code quality |

### Backend (`backend/`)

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Runtime | Node.js (>=20) | Server runtime |
| Framework | Express 5 | REST API |
| ODM | Mongoose 9 | MongoDB object modeling |
| Database | MongoDB | Document store |
| Password hashing | bcryptjs | Secure password storage |
| Auth tokens | jsonwebtoken | Access + refresh JWTs |
| Cookie handling | cookie-parser | HttpOnly refresh-token cookie |
| CORS | cors | Cross-origin requests |
| Security | helmet | HTTP header hardening |
| Rate limiting | express-rate-limit | Abuse protection |
| Logging | morgan | HTTP request logger |
| File uploads | multer | Supporting documents (<=5 MB) |
| Real-time | socket.io | WebSocket notifications |
| Validation | zod | Environment + input schemas |
| Dev tooling | nodemon | Auto-restart in dev |

### Infrastructure

| Item | Value |
|------|-------|
| Database | MongoDB 7 in Docker (`clms-mongo`), port 27017, database `clms` |
| Ports | Backend 5000, Frontend 5173 |
| Proxy | Vite dev server proxies `/api` and `/socket.io` to `http://localhost:5000` |

---

## Roles (4)

| Role | Scope |
|------|-------|
| **Administrator** | College-wide oversight. Manage departments, HODs, classes, tutors, students, parents. Read-only view of all leave requests and reports. |
| **Head of Department (HOD)** | Department-scoped. Review, approve, reject, or reopen leave applications for their own department only. |
| **Class Tutor** | View assigned classes and students. See only APPROVED leaves for their assigned classes. |
| **Parent / Guardian** | View linked children only. Submit leave applications. Track status and download approved leave letter PDFs. |

---

## Project structure

```
Leave-Letter-Management/
├── README.md
├── PROJECT_REPORT.txt   # Full project report (366 lines)
├── win_setup.txt        # Windows 10 setup guide
├── .gitignore
├── backend/             # Express + MongoDB + Socket.IO API
│   ├── package.json
│   ├── .env / .env.example
│   ├── scripts/         # seedAdmin, seedDepartments, seedAcademicData, verifyWorkflow
│   ├── tests/           # node --test suite (28 test files + support/)
│   └── src/
│       ├── server.js        # Entry: connect DB -> listen -> Socket.IO -> graceful shutdown
│       ├── app.js           # Express app factory (helmet, CORS, rate-limit, /api/v1)
│       ├── config/          # env.js (Zod-validated), db.js (mongoose)
│       ├── constants/       # Roles, statuses, leave types, document limits
│       ├── controllers/     # auth, hod, leave, department, class, tutor, student, parent, notification, health
│       ├── models/          # User, Department, Class, Student, ParentStudent, LeaveApplication, LeaveApplicationHistory, Notification, RefreshSession
│       ├── routes/          # All mounted under /api/v1
│       ├── middleware/      # authenticate, authorizeRoles, authorizeDepartment/ParentAccess, validate, rateLimiter, errorHandler, notFound, uploadSupportingDocument
│       ├── services/        # Business logic per resource + token, documentStorage, health
│       ├── realtime/        # socketServer.js (per-user rooms, auth via handshake token)
│       ├── utils/           # AppError, asyncHandler, cookies, databaseErrors, escapeRegExp
│       └── validators/      # Zod request schemas per resource
└── frontend/            # React + Vite + Tailwind SPA
    ├── package.json
    ├── vite.config.js       # Proxies /api and /socket.io to localhost:5000
    ├── tailwind.config.js
    ├── postcss.config.js
    ├── eslint.config.js
    ├── index.html
    └── src/
        ├── App.jsx            # Role-based protected routes
        ├── main.jsx
        ├── context/           # AuthProvider (JWT + socket), DataProvider (backend + demo modes)
        ├── services/api/      # Axios client + per-resource API modules + socketClient + transformers
        ├── services/demoStore.js   # localStorage fallback engine
        ├── pages/             # admin, hod, tutor, parent, auth, public, shared
        ├── components/        # Routing, layout, forms, CRUD, leave workflow
        ├── constants/         # Roles, navigation, leave metadata
        ├── lib/               # Date, CSV, PDF, utility helpers
        └── hooks/             # useAuth, useData, useLeavePdf, etc.
```

---

## Prerequisites

- **Node.js 18+** (v20 recommended, v24 installed on the reference machine)
- **npm** (bundled with Node.js)
- **Docker Desktop** (for MongoDB container)
- **Git** (optional, for cloning)

---

## Installation

From the repository root:

```bash
# Backend dependencies
cd backend
npm install

# Frontend dependencies
cd ../frontend
npm install
```

---

## Environment setup (backend)

Create `backend/.env` from `backend/.env.example`:

```bash
cd backend
cp .env.example .env
```

Edit `backend/.env`:

```dotenv
PORT=5000
NODE_ENV=development
MONGODB_URI=mongodb://localhost:27017/clms
CLIENT_URL=http://localhost:5173
JWT_ACCESS_SECRET=replace_with_a_long_random_secret_at_least_32_chars
JWT_REFRESH_SECRET=replace_with_another_long_random_secret_must_differ
ADMIN_NAME=Initial Administrator
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=DemoOnly!Admin123
```

Rules:
- `JWT_ACCESS_SECRET` and `JWT_REFRESH_SECRET` must each be at least 32 characters.
- The two secrets must not be identical.

Generate strong secrets:

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

Never commit `.env`. It is gitignored.

---

## Running locally (two terminals)

### 1. Start MongoDB (Docker)

```bash
docker run -d --name clms-mongo -p 27017:27017 \
  -v clms_mongo_data:/data/db mongo:7
```

Verify:

```bash
docker ps
```

To restart an existing stopped container:

```bash
docker start clms-mongo
```

### 2. Seed the database

From `backend/`:

```bash
npm run seed:admin
npm run seed:departments
npm run seed:phase4
```

### 3. Start the servers

**Terminal 1 — Backend:**

```bash
cd backend
npm run dev      # http://localhost:5000
```

**Terminal 2 — Frontend:**

```bash
cd frontend
npm run dev      # http://localhost:5173
```

Open `http://localhost:5173` in your browser.

---

## Default login credentials

| Role | Email | Password |
|------|-------|----------|
| Administrator | admin@example.com | DemoOnly!Admin123 |
| Administrator 2 | admin2@example.com | DemoOnly!Admin456 |
| HOD (CSE) | hod.dummy@example.test | DemoOnly!Hod789 |
| Tutor (CSE, CSE-A) | tutor.dummy@example.test | DemoOnly!Tutor012 |
| Parent | parent.dummy@example.test | DemoOnly!Parent345 |

Login URLs:
- Admin: `http://localhost:5173/admin/login`
- HOD: `http://localhost:5173/login/hod`
- Tutor: `http://localhost:5173/login/tutor`
- Parent: `http://localhost:5173/login/parent`

All passwords are development-only placeholders prefixed with `DemoOnly!`.

---

## API overview (`/api/v1`)

| Resource | Routes |
|----------|--------|
| health | `GET /health` |
| auth | `POST /login`, `/refresh`, `/logout`, `GET /me`, `POST /change-password` |
| departments | CRUD + status + HOD read |
| hods | CRUD + status (ADMIN only) |
| classes | CRUD + status + tutor-scoped reads |
| tutors | CRUD + status + self-service classes/students/approved-leaves |
| students | CRUD + status + parent-linked children |
| parents | CRUD + status + child links |
| notifications | List my, unread count, mark read, mark all read |
| leave-applications | Submit, approve, reject, reopen, list, my applications, history, document |

All mutating endpoints enforce role- and department-scoped authorization server-side.

---

## Leave workflow

1. **Submit:** Parent logs in, selects a linked child, fills leave type, date range, reason, optional document (<=5 MB). Status becomes `PENDING`.
2. **Notify:** HOD of the student's department receives a `new_application` notification (Socket.IO + in-app).
3. **Decide:** HOD reviews in `/hod/leave-requests`:
   - **Approve** (optional remarks) → `APPROVED`; tutor and parent notified.
   - **Reject** (mandatory reason >=15 chars) → `REJECTED`; parent notified; tutor never notified.
   - **Reopen** (mandatory reason >=15 chars) → `REOPENED`; prior decision preserved in history; parent notified to correct and resubmit.
4. **Tutor:** Sees only APPROVED leaves for assigned classes in `/tutor/approved-leaves`.
5. **Admin:** Read-only oversight of all leave requests and department reports.

---

## Authentication

- Access token (JWT, 15 minutes) returned in response body and held in memory.
- Refresh token (7 days) set as an HttpOnly cookie (`clms_refresh_token`).
- `authenticate` middleware reloads the user from MongoDB on every protected request — roles, departments, and status always come from the database, never from the client.
- Socket.IO connections authenticate with the access token via the handshake `auth` payload; each user joins only their own `user:<userId>` room.
- Refresh-token rotation on use; revocation on logout.

---

## Testing & linting

```bash
# Backend tests (requires MONGODB_URI_TEST)
cd backend
npm test

# Frontend lint
cd frontend
npm run lint

# Frontend build
cd frontend
npm run build
```

---

## Windows 10 setup

A step-by-step Windows 10 deployment guide (prerequisites, file export, backend/frontend setup, Docker, seeding, running, credentials, troubleshooting) is available in:

```
win_setup.txt
```

---

## Known limitations

- No production backend deployed; this is the local development backend.
- Browser storage is not secure or shared between users.
- Uploaded documents are local metadata only (no external/cloud storage configured by default).
- Approved-letter PDFs are demonstration documents, not official college certificates.
- A production deployment must: hash passwords server-side (already done), enforce role/department rules server-side (done), store files securely, keep secrets out of client code, and use HTTPS.

---

## Extending / adding users

**Via Admin UI:** Log in as admin at `/admin/login`, then use `/admin/hods`, `/admin/tutors`, `/admin/students`, `/admin/parents` to create accounts.

**Via API:** Authenticated admin calls `POST /api/v1/hods`, `/api/v1/tutors`, `/api/v1/students`, `/api/v1/parents` with a Bearer token.

**Via script:** Write a Node script using the backend `User` model so passwords are bcrypt-hashed (never insert raw via mongosh, or logins will fail).
