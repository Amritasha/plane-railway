# Plane — Railway deployment

Self-hosted Jira alternative using official Plane images.

## Architecture

| Service | Image | Railway service |
|---------|-------|-----------------|
| web | makeplane/plane-frontend | `plane-web` |
| api | makeplane/plane-backend | `plane-api` |
| worker | makeplane/plane-backend | `plane-worker` |
| Postgres | Railway plugin | — |
| Redis | Railway plugin | — |

## Deploy on Railway

### 1. Create services

In your Railway project, create **5 services**:

| Service | Root directory | railway.toml location |
|---------|---------------|----------------------|
| plane-web | `web/` | `web/railway.toml` |
| plane-api | `api/` | `api/railway.toml` |
| plane-worker | `worker/` | `worker/railway.toml` |
| Postgres | plugin | — |
| Redis | plugin | — |

### 2. Set environment variables

Copy `.env.example` → fill in values → set on **api** and **worker** services (they share the same vars). The **web** service only needs:

```
NEXT_PUBLIC_API_BASE_URL=https://<plane-api-domain>
```

### 3. Run migrations (first deploy only)

In the Railway shell for `plane-api`:
```sh
python manage.py migrate
python manage.py create_bucket   # if using MinIO
```

### 4. Create superuser

```sh
python manage.py createsuperuser
```

## Default port

- web: 3000
- api: 8000
- worker: no HTTP port needed
