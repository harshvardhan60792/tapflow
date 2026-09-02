# tapflow Docker Image

The official Docker image for **tapflow**, providing a self-hosted relay server and web dashboard in a single lightweight container.

```sh
docker pull tapflow/tapflow:latest
```

## Quick Start

Run the tapflow relay with persistent storage:

```sh
docker run -d \
  --name tapflow-relay \
  -p 4000:4000 \
  -v tapflow-data:/app/.tapflow/data \
  --restart unless-stopped \
  tapflow/tapflow:latest
```

Open the dashboard in your browser: `http://localhost:4000` (or `http://<LAN_IP>:4000`).

---

## Docker Compose

Save as `docker-compose.yml`:

```yaml
version: '3.8'

services:
  tapflow-relay:
    image: tapflow/tapflow:latest
    container_name: tapflow-relay
    restart: unless-stopped
    ports:
      - "4000:4000"
    volumes:
      - tapflow-data:/app/.tapflow/data
    environment:
      - NODE_ENV=production
      # - JWT_SECRET=your-custom-jwt-secret
      # - TAPFLOW_PORT=4000

volumes:
  tapflow-data:
    name: tapflow-data
```

Start the service:

```sh
docker compose up -d
```

---

## Scope & Container Placement

### Relay-only container

This container packages **only the relay and dashboard**. The iOS and Android agents require native macOS access (SimulatorKit, Xcode, ADB) and must run natively on your Mac.

### Where to run the container

Every video and audio frame between an agent and a browser passes through the relay (`packages/relay/src/RelayServer.ts`).

- **Recommended**: Run the container on your local Mac or an always-on server/NAS on the **same LAN**.
- **Avoid public cloud VMs**: Running the relay on an external cloud VM routes raw application video streams across the internet and introduces RTT latency that degrades the real-time tap-to-glass streaming experience.

---

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `TAPFLOW_PORT` | `4000` | Port for HTTP dashboard and WebSocket traffic |
| `JWT_SECRET` | *(auto-generated)* | Key for signing JWT session tokens. Persisted in `.tapflow/data/jwt-secret` if unset |
| `TAPFLOW_CORS_ORIGIN` | `*` | Allowed CORS origins for the API |
| `NODE_ENV` | `production` | Node runtime environment |

---

## Volumes & Persistence

Mount `/app/.tapflow/data` to persist:
- `tapflow.sqlite` (session records, auth metadata)
- `jwt-secret` (per-install auto-generated secret)
- Custom configurations and authentication tokens

---

## Publishing from a Fork

To build and publish images from your own fork using GitHub Actions (`.github/workflows/docker-publish.yml`):

1. Create a Docker Hub account and repository (e.g. `<username>/tapflow`).
2. Generate an Access Token on Docker Hub (**Account Settings → Security → New Access Token** with Read & Write permissions).
3. In your GitHub repository, go to **Settings → Secrets and variables → Actions** and add:
   - `DOCKERHUB_USERNAME`: Your Docker Hub username
   - `DOCKERHUB_TOKEN`: Your Docker Hub personal access token
4. Pushing tags (`v*`) or commits to `main` will build multi-arch images (`linux/amd64`, `linux/arm64`) and publish them to your Docker Hub repository.
