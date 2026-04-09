# stalwart

> Click To Deploy Stalwart — All-in-One Mail & Collaboration Server

[![Sync](https://github.com/opensaasapps/stalwart/actions/workflows/sync.yml/badge.svg)](https://github.com/opensaasapps/stalwart/actions/workflows/sync.yml) [![Docker](https://github.com/opensaasapps/stalwart/actions/workflows/docker.yml/badge.svg)](https://github.com/opensaasapps/stalwart/actions/workflows/docker.yml) [![Docker Pulls](https://img.shields.io/docker/pulls/thefractionalpm/stalwart)](https://hub.docker.com/r/thefractionalpm/stalwart)

Upstream: [stalwartlabs/mail-server](https://github.com/stalwartlabs/mail-server) · Auto-synced daily

---

## One-Command Deploy

```bash
cp .env.example .env && nano .env
docker compose up -d
```

## Coolify / Dokploy

1. New service → **Docker Compose**
2. Paste `docker-compose.yml`
3. Set env vars in UI
4. Deploy

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `TZ` | ⚪ | |

## Image

```
docker pull stalwartlabs/stalwart:latest
docker pull thefractionalpm/stalwart:latest
```

## Ports

| Port | Service |
|---|---|
| `8080` | Main app |

---

*Part of the [OpenSaaSApps](https://github.com/opensaasapps) Click-To-Deploy collection.*
