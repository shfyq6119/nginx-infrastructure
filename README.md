# Astronomical Sighting App — Containerized Nginx Infrastructure

A progressively built, containerized web infrastructure for a mobile-first **Astronomical / Moonsighting Field Logger PWA**, designed for local development with **Docker Compose** and cloud deployment on **Google Cloud**.

---

## Repository Structure

```text
nginx-infrastructure/
├── docker-compose.yml     # Container orchestration, read-only mounts & healthcheck
├── .gitignore             # Excludes macOS metadata (.DS_Store) and secrets (.env)
├── nginx/
│   └── default.conf       # Hardened Nginx server configuration & /health endpoint
└── frontend/
    └── index.html         # Client-facing mobile viewport entry page
```

---

## Architecture & Hardening Decisions (Milestone 1)

| Feature / Directive | File | Engineering Purpose |
| :--- | :--- | :--- |
| `nginx:1.27-alpine` | [`docker-compose.yml`](docker-compose.yml) | Uses a lightweight, version-pinned Alpine Linux image instead of `:latest` for reproducible builds and a smaller attack surface. |
| Read-only bind mounts (`:ro`) | [`docker-compose.yml`](docker-compose.yml) | Mounts `./frontend` and `./nginx/default.conf` as read-only inside the container so running processes cannot modify web assets or server config. |
| `server_tokens off;` | [`nginx/default.conf`](nginx/default.conf) | Suppresses exact Nginx version numbers in HTTP response headers and error pages. |
| `X-Frame-Options` & `X-Content-Type-Options` | [`nginx/default.conf`](nginx/default.conf) | Adds baseline HTTP security headers against clickjacking (`SAMEORIGIN`) and MIME-sniffing (`nosniff`). |
| `gzip on;` | [`nginx/default.conf`](nginx/default.conf) | Compresses static assets (`text/css`, `application/javascript`, `application/json`) for faster loading over mobile networks. |
| `location /health` (`access_log off;`) | [`nginx/default.conf`](nginx/default.conf) | Dedicated `200 OK` heartbeat endpoint paired with Docker's `healthcheck`, with access logging disabled on `/health` to prevent automated probe noise. |

---

## Quick Start & Verification

### 1. Start the Container
```bash
docker compose up -d
```

### 2. Verify Health Endpoint & Security Headers
```bash
curl -i http://localhost:8080/health
```
Expected response includes `HTTP/1.1 200 OK`, `Server: nginx` (no version exposed), `X-Frame-Options: SAMEORIGIN`, `X-Content-Type-Options: nosniff`, and body `OK`.

### 3. Check Container Health Status
```bash
docker ps
```
Confirm the `STATUS` column reports `Up ... (healthy)`, then open `http://localhost:8080` in a browser.

### 4. Stop & Clean Up
```bash
docker compose down
```

---

## Project Roadmap

- [x] **Milestone 1:** Custom & hardened `nginx:1.27-alpine` container with read-only mounts, security headers, and `/health` monitoring
- [ ] **Milestone 2:** Mobile-first Moonsighting PWA frontend (interactive map, Sun/Moon astronomical metadata, and sighting input form)
- [ ] **Milestone 3:** Backend API container & Nginx reverse proxy (`location /api/`) over an internal Docker network
- [ ] **Milestone 4:** PostgreSQL database container with persistent volumes, `.csv` export, and Google Docs report generation
- [ ] **Milestone 5:** Google Cloud deployment with HTTPS/TLS and observer access control
