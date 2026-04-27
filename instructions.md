# Run Plane Locally (Docker)

## Prerequisites

- Docker Desktop running
- This repo cloned locally

## 1) Initial setup (one-time)

From repo root:

```powershell
Copy-Item .env.example .env
Copy-Item .\apps\api\.env.example .\apps\api\.env
```

Open `apps/api/.env` and add:

```env
SECRET_KEY="plane-local-dev-secret-key-change-in-production"
```

If XAMPP/Apache is using port 80, set these in `.env`:

```env
LISTEN_HTTP_PORT=18080
LISTEN_HTTPS_PORT=18443
```

## 2) Start services

First run:

```powershell
docker compose up -d --build
```

Open app:

- `http://localhost:18080` (if using alternate ports)
- otherwise `http://localhost`

## 3) Normal daily commands

Start:

```powershell
docker compose up -d
```

Stop:

```powershell
docker compose down
```

Check status/logs:

```powershell
docker compose ps
docker compose logs -f api
```

## 4) Without internet (possible options)

Important: a true "from scratch" install on a brand-new machine is not possible without any preloaded files. You must prepare an offline bundle once on an online machine.

### Option A: Same machine, after first successful build

Run offline with already-built images:

```powershell
docker compose up -d --no-build
```

### Option B: Move to another offline machine

On online machine:

```powershell
docker compose build
docker save plane-preview-api plane-preview-web plane-preview-admin plane-preview-space plane-preview-live plane-preview-worker plane-preview-beat-worker plane-preview-migrator plane-preview-proxy postgres:15.7-alpine valkey/valkey:7.2.11-alpine rabbitmq:3.13.6-management-alpine minio/minio -o plane-images.tar
```

On offline machine:

```powershell
docker load -i plane-images.tar
docker compose up -d --no-build
```

### Option C: From scratch on a brand-new machine (fully offline)

#### Step 1: Prepare offline bundle on an online machine

1. Create a folder, for example `plane-offline-bundle`.
2. Copy complete Plane repo into that folder.
3. Build and export all required Docker images:

```powershell
docker compose build
docker save plane-preview-api plane-preview-web plane-preview-admin plane-preview-space plane-preview-live plane-preview-worker plane-preview-beat-worker plane-preview-migrator plane-preview-proxy postgres:15.7-alpine valkey/valkey:7.2.11-alpine rabbitmq:3.13.6-management-alpine minio/minio -o plane-images.tar
```

4. Put `plane-images.tar` inside `plane-offline-bundle`.
5. Also add Docker Desktop installer (for Windows) to the same bundle.
6. Copy the whole `plane-offline-bundle` to USB/external drive.

#### Step 2: Run on offline target machine

1. Install Docker Desktop from the installer in your bundle.
2. Open repo folder from bundle.
3. Create env files:

```powershell
Copy-Item .env.example .env
Copy-Item .\apps\api\.env.example .\apps\api\.env
```

4. Set required key in `apps/api/.env`:

```env
SECRET_KEY="plane-local-dev-secret-key-change-in-production"
```

5. Load images and run without build:

```powershell
docker load -i .\plane-images.tar
docker compose up -d --no-build
```

6. Open app:

- `http://localhost:18080` (if using alternate ports)
- otherwise `http://localhost`

## 5) Quick fix if UI says Plane did not start correctly

1. Ensure `apps/api/.env` has `SECRET_KEY`.
2. Restart backend services:

```powershell
docker compose up -d api worker beat-worker migrator
```
