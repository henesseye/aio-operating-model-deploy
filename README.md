# 🚀 AIO Operating Model - Production Deployment

Dieses Repository enthält **nur die Files für Production Deployment**.

Das Haupt-Repository mit Source Code ist separat - hier gibt es nur das was die Docker-Crew braucht!

## 🎯 Quick Start

```bash
# 1. Repository clonen
git clone [YOUR-REPO-URL] aio-operating-model-deploy
cd aio-operating-model-deploy

# 2. Environment konfigurieren
cp env.example .env
# .env bearbeiten (JWT_SECRET, ADMIN_PASSWORD, PUBLIC_PORT)

# 3. Deployen
docker-compose -f docker-compose.prod.yml up -d

# 4. Testen
curl http://localhost:80/health
```

## 🔄 Updates

```bash
# Neue Files holen
git pull

# Images aktualisieren
docker-compose -f docker-compose.prod.yml pull

# Neustart mit Updates
docker-compose -f docker-compose.prod.yml up -d
```

## 📁 Repository Inhalt

- `docker-compose.prod.yml` - Production Docker Compose
- `DEPLOYMENT.md` - Detaillierte Deployment-Anleitung  
- `env.example` - Environment Template
- `data/model.json` - Betriebsmodell-Daten (45+ Prozesse)

## 📞 Support

Bei technischen Problemen wenden Sie sich an Daniel Heiniger.

**Dieses Repo ist ~1MB statt 15MB - nur das Nötige für Production! 🎯**
