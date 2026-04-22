# CarNexus — 3D Vehicle Customization Platform

Production-oriented monorepo with a **Spring Boot + PostgreSQL + JWT** backend and a **React + Tailwind + React Three Fiber** frontend for interactive GLB customization (paint, wheel presets, material modes), workflow statuses, asset uploads, admin roles, STOMP updates over SockJS, and PNG screenshot export.

## Prerequisites

- **JDK 17** and **Maven 3.9+**
- **Node.js 20+** (tested with Node 22)
- **Docker** (optional, for PostgreSQL only)

## 1. Database

```bash
docker compose up -d postgres
```

This exposes PostgreSQL on `localhost:5432` with database/user/password **`carnexus`** (see `docker-compose.yml`). Override via environment variables in `backend/src/main/resources/application.yml`: `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`.

## 2. Backend (`/backend`)

```bash
cd backend
mvn spring-boot:run
```

The API listens on **http://localhost:8080**.

- JWT secret and expiration: `carnexus.jwt.*` in `application.yml` (override `JWT_SECRET` in production).
- Uploaded assets are stored under `./data/uploads` (override `UPLOAD_DIR`).
- Bundled sample GLB: `backend/src/main/resources/static/models/sample-car.glb` (Khronos Box binary, valid GLB). Vehicles seeded in `DataInitializer` reference `/models/sample-car.glb`.

### Role seed accounts

| Email               | Password    | Role     |
|---------------------|-------------|----------|
| admin@carnexus.dev  | Admin123!   | ADMIN    |
| modifier@carnexus.dev | Modifier123! | MODIFIER |
| user@carnexus.dev   | User123!    | USER     |

### Primary HTTP endpoints

- `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me`
- `GET /api/vehicles`, `POST /api/vehicles` (ADMIN)
- `POST /api/customizations` (USER/ADMIN), `GET /api/customizations/user`, `GET /api/customizations` (MODIFIER/ADMIN), `PUT /api/customizations/{id}/status`
- `POST /api/assets/upload` (multipart: `file`, `type`=MODEL|TEXTURE)
- `GET /api/admin/users`, `PATCH /api/admin/users/{id}/role`
- Static: `/models/**` (classpath), `/files/**` (upload dir)
- WebSocket (SockJS): `/ws` — STOMP topic `/topic/customizations`

## 3. Frontend (`/frontend`)

```bash
cd frontend
npm install
npm run dev
```

Open **http://localhost:5173**. Vite proxies `/api`, `/files`, `/models`, and `/ws` to the backend (see `vite.config.js`). For a production build served separately, set `VITE_API_URL` to the API origin (see `.env.example`).

### Features

- JWT in `localStorage`, protected routes, role-based navigation
- Vehicle catalog → **3D viewer** with OrbitControls, Environment lighting, shadows, live color/material/wheel presets, save configuration (JSON), PNG screenshot (`preserveDrawingBuffer`)
- Modifier review queue + asset upload form
- Admin user management
- Real-time STOMP subscription on `/topic/customizations` (dashboard toast + list refresh)

## Testing the 3D path

1. Sign in as `user@carnexus.dev` / `User123!`.
2. Open **Vehicles** → **Open 3D viewer** on a seeded vehicle.
3. Adjust color and presets; confirm the mesh updates immediately.
4. **Save configuration** — creates a customization in `REQUESTED`.
5. Sign in as modifier or admin, open **Review queue**, advance statuses (`APPROVED` is **ADMIN-only**).

## Project layout

```
/backend   Spring Boot (controllers, services, JWT, WebSocket, uploads)
/frontend  Vite + React + Tailwind + R3F
docker-compose.yml   Local PostgreSQL
```

## Security notes

Change `JWT_SECRET` and database credentials for any shared or production deployment. The default JWT signing secret in `application.yml` is for local development only.
