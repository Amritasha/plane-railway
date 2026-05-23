# Plane — Railway Template

One-click self-hosted Plane (open-source Jira/Linear alternative) on Railway using official Docker images.

## Architecture

| Service | Image | Port |
|---------|-------|------|
| `plane-web` | `makeplane/plane-frontend:stable` | 3000 |
| `plane-api` | `makeplane/plane-backend:stable` | 8000 |
| `plane-worker` | `makeplane/plane-backend:stable` | — |
| `plane-beat-worker` | `makeplane/plane-backend:stable` | — |
| Postgres | Railway plugin | — |
| Redis | Railway plugin | — |

`plane-api` automatically runs database migrations on every start before the server comes up.

---

## Deploy on Railway

### 1. Fork this repo

Fork to your GitHub account so Railway can link to it.

### 2. Create a Railway project

Go to [railway.app](https://railway.app) → **New Project** → **Empty project**.

### 3. Add Postgres and Redis plugins

In the project dashboard: **+ New** → **Database** → add **PostgreSQL**, then repeat for **Redis**.  
Railway injects `DATABASE_URL` and `REDIS_URL` automatically into all services.

### 4. Create the four app services

Repeat for each service below — **+ New** → **GitHub Repo** → select your fork → set the root directory:

| Railway service name | Root directory |
|----------------------|----------------|
| `plane-web` | `web` |
| `plane-api` | `api` |
| `plane-worker` | `worker` |
| `plane-beat-worker` | `beat-worker` |

### 5. Set environment variables

Set these on **`plane-api`**, **`plane-worker`**, and **`plane-beat-worker`** (all three share the same backend config):

| Variable | Value |
|----------|-------|
| `SECRET_KEY` | 50-char random string (use `openssl rand -hex 25`) |
| `DATABASE_URL` | `${{Postgres.DATABASE_URL}}` |
| `REDIS_URL` | `${{Redis.REDIS_URL}}` |
| `WEB_URL` | public domain of `plane-web` |
| `CORS_ALLOWED_ORIGINS` | public domain of `plane-web` |
| `DJANGO_SETTINGS_MODULE` | `plane.settings.production` |
| `GUNICORN_WORKERS` | `2` |

Set this on **`plane-web`** only:

| Variable | Value |
|----------|-------|
| `NEXT_PUBLIC_API_BASE_URL` | public domain of `plane-api` |

### 6. Deploy order

Deploy in this order to avoid startup failures:
1. Postgres + Redis (auto-start)
2. `plane-api` (runs migrations, then starts)
3. `plane-worker` + `plane-beat-worker`
4. `plane-web`

### 7. Create your account

Once `plane-api` is healthy, visit `https://<plane-web-domain>` and register.  
The first user to register becomes the workspace owner.

---

## File storage

The default config has no persistent file storage — uploads are lost on redeploy.  
For production, set the S3 variables in `.env.example` (works with AWS S3, Cloudflare R2, Backblaze B2, etc.).

---

## Publishing to Railway Marketplace

1. Push your fork to GitHub.
2. Go to [railway.app/button](https://railway.app/button) or **Dashboard → Templates → New Template**.
3. Add each service (web, api, worker, beat-worker), point to the correct root directory.
4. Add the Postgres and Redis plugins.
5. Wire up the environment variables using Railway's `${{service.VAR}}` reference syntax.
6. Submit for review — Railway pays template creators a kickback from usage fees.
