# IncidentWatch — Full MERN Stack App

A real-time incident monitoring platform. The React frontend you started with is now
backed by a complete MongoDB + Express + Node API, with JWT authentication, role-based
access (user / police / hospital / admin), and live updates over Socket.io.

```
incident-monitoring-mern/
├── backend/     Express + MongoDB API (auth, incidents, hospitals, police units, admin)
└── frontend/    React + Vite dashboard (unchanged design, now wired to live data)
```

## What was added

- **Backend API** (`/backend`) — Express, Mongoose, JWT auth with bcrypt password
  hashing, role-based route protection, and Socket.io for real-time incident broadcasts.
- **Auth wired end-to-end** — Signup/Login now call the real API and store a JWT;
  `/dashboard`, `/police`, `/hospitals`, `/admin` are protected routes, with `/police`
  and `/admin` further restricted by role.
- **Live data everywhere** — Dashboard, Police, Hospitals and Admin pages now fetch
  real incidents, hospitals, police units, users and stats from MongoDB instead of
  hardcoded arrays.
- **"Report incident" flow** — a modal form on the Dashboard posts a new incident to
  the API and broadcasts it to every connected client instantly via Socket.io.
- **Role-based actions** — police accounts can change a unit's status and dispatch
  units to incidents; hospital accounts can adjust bed availability; admins can manage
  user roles/verification and see system health.
- **Seed script** — populates demo hospitals, police units and incidents around
  Vijayawada (matching the original mock data) plus a ready-made admin account.

## Prerequisites

- [Node.js](https://nodejs.org/) 18+
- A MongoDB database — either [install MongoDB locally](https://www.mongodb.com/docs/manual/installation/)
  or create a free cluster on [MongoDB Atlas](https://www.mongodb.com/atlas) and copy
  its connection string.

## 1. Set up the backend

```bash
cd backend
cp .env.example .env
# open .env and set MONGO_URI (and JWT_SECRET to any long random string)
npm install
npm run seed   # optional but recommended: populates demo data + an admin login
npm run dev    # starts the API on http://localhost:5000
```

The seed script creates an admin account you can log in with immediately:

```
email:    admin@incidentwatch.dev
password: admin1234
```

## 2. Set up the frontend

In a second terminal:

```bash
cd frontend
cp .env.example .env
# defaults already point to http://localhost:5000, only change if your API runs elsewhere
npm install
npm run dev    # starts the app on http://localhost:5173
```

Open the printed local URL, sign in with the admin account (or sign up as any role),
and the dashboard will populate with live data from MongoDB.

## API overview

All routes are prefixed with `/api` and (except `/auth/signup` and `/auth/login`)
require an `Authorization: Bearer <token>` header.

| Method | Route | Notes |
|---|---|---|
| POST | `/auth/signup` | Create an account (`role`: user / police / hospital / admin) |
| POST | `/auth/login` | Returns a JWT + user profile |
| GET | `/auth/me` | Current user |
| GET | `/incidents` | List incidents (`?status=`, `?severity=`, `?limit=`) |
| GET | `/incidents/stats/summary` | Active / resolved-today counts |
| POST | `/incidents` | Report a new incident |
| PATCH | `/incidents/:id` | Update status/severity/assigned unit — police or admin |
| DELETE | `/incidents/:id` | Admin only |
| GET | `/hospitals` | List hospitals |
| POST | `/hospitals` | Admin only |
| PATCH | `/hospitals/:id` | Update capacity — hospital or admin |
| GET | `/police/units` | List response units |
| PATCH | `/police/units/:id` | Update unit status — police or admin |
| GET | `/admin/stats` | User/role/incident counts — admin only |
| GET | `/admin/health` | System health snapshot — admin only |
| GET/PATCH/DELETE | `/admin/users` | User management — admin only |

Incident, hospital and police-unit changes are also broadcast in real time over
Socket.io (`incident:new`, `incident:update`, `incident:delete`, `hospital:update`,
`unit:update`), which the dashboard subscribes to automatically.

## Notes

- This is a learning/demo-grade setup: secrets live in `.env` files (never commit
  them — both `.gitignore` files already exclude them), and there's no rate limiting,
  email verification, or production hardening yet.
- To deploy, point `MONGO_URI` at an Atlas cluster, set a strong `JWT_SECRET`, deploy
  `/backend` (e.g. Render, Railway, Fly.io) and `/frontend` (e.g. Vercel, Netlify) —
  then update `VITE_API_URL` / `VITE_SOCKET_URL` and `CLIENT_ORIGIN` accordingly.
