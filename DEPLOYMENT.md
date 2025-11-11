# Production Deployment - AIO Betriebsmodell

## Sicherheitsstatus

### CVE-Status (Stand: 2025-11-11)

**API Image:**
- ✅ **0 CRITICAL, 0 HIGH CVEs**
- CVE-2025-9230 (OpenSSL 3.3.5) - behoben
- CVE-2024-21538 (npm cross-spawn 7.0.5) - behoben

**Web Image:**
- ⚠️ **0 CRITICAL, 2 HIGH CVEs (nicht fixbar)**
- CVE-2025-9230 (OpenSSL 3.3.5) - behoben
- CVE-2025-5399, CVE-2025-9086 (curl 8.14.1-r2) - behoben
- CVE-2025-6021 (libxml2) - kein Fix in Alpine 3.20 verfügbar
- CVE-2025-31498 (c-ares) - kein Fix in Alpine 3.20 verfügbar

Die verbleibenden CVEs betreffen System-Bibliotheken ohne direkten Angriffsvektor in dieser Architektur. Web-Container läuft als non-root User.

**CVE-Scan:**
```bash
# Docker Scout CVE-Analyse
docker scout cves henesseye/aio-operating-model-api:1.0.6
docker scout cves henesseye/aio-operating-model-web:1.0.6
```

### Health Checks

Beide Images verfügen über integrierte Health Checks:

**API Container:**
- Endpoint: `http://localhost:3000/health`
- Interval: 30s, Timeout: 10s, Retries: 3

**Web Container:**
- Endpoint: `http://localhost:8080/health`
- Interval: 30s, Timeout: 10s, Retries: 3
- Startet erst nach erfolgreicher API-Initialisierung

Status-Überprüfung:
```bash
docker-compose ps                    # Status: healthy
docker inspect <container> --format='{{.State.Health.Status}}'
```

---

## Image Transfer ins interne Repository

Die Docker-Crew muss Images von Docker Hub ins interne Repository transferieren.

**Aktuelle Version:** `1.0.6`
**Docker Hub:** https://hub.docker.com/r/henesseye/aio-operating-model-api/tags

### Transfer-Prozess

```bash
# 1. Images von Docker Hub pullen
VERSION=1.0.6
docker pull henesseye/aio-operating-model-api:${VERSION}
docker pull henesseye/aio-operating-model-web:${VERSION}

# 2. Für internes Repository taggen
INTERNAL_REGISTRY="docker-registry.internal.company.com:5000"
INTERNAL_PATH="aio"

docker tag henesseye/aio-operating-model-api:${VERSION} \
  ${INTERNAL_REGISTRY}/${INTERNAL_PATH}/aio-operating-model-api:${VERSION}

docker tag henesseye/aio-operating-model-web:${VERSION} \
  ${INTERNAL_REGISTRY}/${INTERNAL_PATH}/aio-operating-model-web:${VERSION}

# 3. Ins interne Repository pushen
docker login ${INTERNAL_REGISTRY}
docker push ${INTERNAL_REGISTRY}/${INTERNAL_PATH}/aio-operating-model-api:${VERSION}
docker push ${INTERNAL_REGISTRY}/${INTERNAL_PATH}/aio-operating-model-web:${VERSION}

# 4. Verify
docker pull ${INTERNAL_REGISTRY}/${INTERNAL_PATH}/aio-operating-model-api:${VERSION}
```

---

## Deployment

### Vorbereitung

**Benötigte Dateien:**
- `docker-compose.prod.yml` - Production Compose-Konfiguration
- `env.example` - Konfigurationsvorlage
- `data/model.json` - Betriebsmodell-Daten

**Verzeichnisstruktur:**
```
aio-operating-model/
├── docker-compose.prod.yml
├── .env
└── data/
    └── model.json
```

### Konfiguration

`.env` Datei erstellen und anpassen:

```bash
# Version (ERFORDERLICH - keine latest Tags!)
VERSION=1.0.6

# Internes Registry (mit trailing slash!)
DOCKER_REGISTRY=docker-registry.internal.company.com:5000/aio/

# Sicherheit (MUSS geändert werden!)
JWT_SECRET=<mindestens-32-zeichen-zufälliger-string>
ADMIN_PASSWORD=<sicheres-passwort>

# Netzwerk
PUBLIC_PORT=80
```

**Wichtig:**
- `VERSION` darf nicht `latest` sein
- `DOCKER_REGISTRY` muss mit `/` enden
- `JWT_SECRET` mindestens 32 Zeichen
- Images müssen im Registry verfügbar sein vor dem Deployment

### Deployment-Prozess

```bash
# 1. Images vom internen Registry pullen
docker-compose -f docker-compose.prod.yml pull

# 2. Konfiguration prüfen
docker-compose -f docker-compose.prod.yml config

# 3. Services starten
docker-compose -f docker-compose.prod.yml up -d

# 4. Status prüfen
docker-compose -f docker-compose.prod.yml ps
docker-compose -f docker-compose.prod.yml logs -f

# 5. Health Status überprüfen
# Beide Services sollten Status "healthy" zeigen
```

### Zugriff

- **Web-Anwendung:** `http://<docker-host>:80` (oder konfigurierter PUBLIC_PORT)
- **Admin-Login:** Username: `admin`, Passwort: wie in `.env` definiert
- **API:** Intern auf Port 3000 (nicht extern erreichbar)

---

## Updates

### Update-Prozess

```bash
# 1. Neue Version ins interne Registry transferieren
NEW_VERSION=1.0.6
# (Image Transfer wie oben beschrieben durchführen)

# 2. .env aktualisieren
VERSION=1.0.6

# 3. Update deployen
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d

# 4. Status überprüfen
docker-compose -f docker-compose.prod.yml ps
docker-compose -f docker-compose.prod.yml logs -f
```

### Rollback

```bash
# .env auf vorherige Version setzen
VERSION=1.0.6

# Alte Version deployen
docker-compose -f docker-compose.prod.yml up -d
```

---

## Datenpersistierung

Der `./data` Ordner ist persistent und enthält:
- `model.json` - Betriebsmodell-Daten (wird mitgeliefert)
- `users.json` - User-Daten (wird automatisch erstellt)

**Backup-Strategie erforderlich** für `./data` Ordner.

---

## Troubleshooting

### Häufige Probleme

**Port bereits belegt:**
```bash
# PUBLIC_PORT in .env ändern
PUBLIC_PORT=8080
docker-compose -f docker-compose.prod.yml up -d
```

**Health Check schlägt fehl:**
```bash
# Logs prüfen
docker-compose -f docker-compose.prod.yml logs api
docker-compose -f docker-compose.prod.yml logs web

# Container neu starten
docker-compose -f docker-compose.prod.yml restart
```

**Admin Login fehlgeschlagen:**
```bash
# ADMIN_PASSWORD in .env prüfen
# API Container neu starten
docker-compose -f docker-compose.prod.yml restart api
```

**model.json nicht gefunden:**
```bash
# Datei muss vorhanden sein
ls -la ./data/model.json
# Falls fehlend: model.json bereitstellen
```

### Logs und Diagnostics

```bash
# Alle Logs anzeigen
docker-compose -f docker-compose.prod.yml logs

# Service-spezifische Logs
docker-compose -f docker-compose.prod.yml logs api
docker-compose -f docker-compose.prod.yml logs web

# Live-Logs verfolgen
docker-compose -f docker-compose.prod.yml logs -f

# Container Status
docker-compose -f docker-compose.prod.yml ps
```

---

## Deployment-Checkliste

### Image Transfer
- [ ] Spezifische Version von Docker Hub gepullt (z.B. 1.0.6)
- [ ] Images für internes Registry getaggt
- [ ] Images ins interne Registry gepusht
- [ ] Pull-Test vom internen Registry erfolgreich

### Deployment-Vorbereitung
- [ ] `docker-compose.prod.yml` platziert
- [ ] `data/model.json` platziert
- [ ] `.env` erstellt und konfiguriert:
  - [ ] `VERSION` gesetzt (keine latest Tags!)
  - [ ] `DOCKER_REGISTRY` konfiguriert (mit trailing `/`)
  - [ ] `JWT_SECRET` gesetzt (min. 32 Zeichen)
  - [ ] `ADMIN_PASSWORD` gesetzt
  - [ ] `PUBLIC_PORT` angepasst (falls erforderlich)

### Deployment-Durchführung
- [ ] Images gepullt: `docker-compose -f docker-compose.prod.yml pull`
- [ ] Services gestartet: `docker-compose -f docker-compose.prod.yml up -d`
- [ ] Status geprüft: beide Services `Up` und `healthy`
- [ ] Logs überprüft: keine Errors

### Funktionstests
- [ ] Web-App unter `http://<host>:<port>` erreichbar
- [ ] Admin-Login erfolgreich (Username: `admin`)
- [ ] Prozesse/Services Toggle funktioniert
- [ ] Admin-Panel erreichbar
- [ ] Verbindungsvisualisierung funktioniert

### Security & Betrieb
- [ ] Firewall-Regeln konfiguriert
- [ ] Backup-Strategie für `./data` eingerichtet
- [ ] HTTPS Reverse Proxy konfiguriert (empfohlen)
- [ ] Update-Prozess dokumentiert

### Security-Compliance
- [x] API Image: 0 HIGH/CRITICAL CVEs
- [x] Web Image: Alle fixbaren CVEs behoben
- [x] Health Checks aktiv
- [x] Nur versionierte Images (keine latest Tags)
- [x] Non-root User für Web-Container
