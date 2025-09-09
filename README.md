# 🚀 AIO Operating Model - Production Deployment

**Version: 1.0.0**

## Quick Start

```bash
git clone https://github.com/henesseye/aio-operating-model-deploy.git
cd aio-operating-model-deploy
cp env.example .env
# .env bearbeiten (Passwörter ändern!)
docker-compose -f docker-compose.prod.yml up -d
```

## Images auf Docker Hub

- `henesseye/aio-operating-model-api:1.0.0`
- `henesseye/aio-operating-model-web:1.0.0`
- `henesseye/aio-operating-model-api:latest`
- `henesseye/aio-operating-model-web:latest`

## Update

```bash
git pull
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d
```

Letzte Aktualisierung: Di  9 Sep 2025 14:46:55 CEST
