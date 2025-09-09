# 📦 Docker Crew Deployment Package

## Übersicht der Dateien für Production Deployment

### 🎯 Das bekommen Sie von mir:

#### **Pflicht-Dateien (MÜSSEN Sie haben):**
```
📁 AIO-Operating-Model-Deployment/
├── 📄 docker-compose.prod.yml    # ← Production Docker Compose
├── 📄 DEPLOYMENT.md              # ← Diese Anleitung (ausführlich)
├── 📁 data/
│   └── 📄 model.json            # ← Vollständige Betriebsmodell-Daten
└── 📄 .env.example              # ← Environment Template
```

#### **Optional (für Entwickler/Wartung):**
```
📁 Optional-Files/
├── 📄 BUILD.md                  # ← Build-Prozess Dokumentation
├── 📄 docker-compose.yml        # ← Development Compose (mit Build)
├── 📄 deployment-anleitung.md   # ← Alte Anleitung (deprecated)
└── 📄 portainer-deployment.md   # ← Spezielle Portainer-Anleitung
```

### 🐳 Docker Images (automatisch von Docker Hub)

**Verfügbar auf Docker Hub:**
- `henesseye/aio-operating-model-api:latest`
- `henesseye/aio-operating-model-web:latest`
- `henesseye/aio-operating-model-api:1.0.0` (spezifische Versionen)
- `henesseye/aio-operating-model-web:1.0.0`

### ⚡ Schnellstart für Docker-Crew

```bash
# 1. Dateien von Daniel erhalten und entpacken
tar -xzf aio-operating-model-deployment.tar.gz
cd AIO-Operating-Model-Deployment/

# 2. Environment konfigurieren
cp .env.example .env
# .env bearbeiten (JWT_SECRET, ADMIN_PASSWORD, PUBLIC_PORT)

# 3. Deployen
docker-compose -f docker-compose.prod.yml up -d

# 4. Testen
curl http://localhost:80/health
```

### 🔑 Minimum-Konfiguration (.env)

```bash
# Beispiel .env - MÜSSEN Sie anpassen!
JWT_SECRET=IhrSicheresJWTSecret2024Mindestens32ZeichenLang!
ADMIN_PASSWORD=IhrSuperSicheresAdminPasswort2024!
PUBLIC_PORT=80
```

### 📋 Deployment-Checkliste für Docker-Crew

#### Vorbereitung:
- [ ] Alle Dateien von Daniel erhalten
- [ ] Docker & Docker Compose auf Zielsystem installiert
- [ ] Ports verfügbar (Standard: 80, oder in .env konfiguriert)
- [ ] Ausreichend Disk Space (ca. 500MB für Images)

#### Deployment:
- [ ] `docker-compose -f docker-compose.prod.yml pull` → Images laden
- [ ] `.env` konfiguriert (sichere Passwörter!)
- [ ] `docker-compose -f docker-compose.prod.yml up -d` → Starten
- [ ] `docker-compose -f docker-compose.prod.yml ps` → Status OK
- [ ] Web-App unter http://host:port erreichbar
- [ ] Admin-Login funktioniert (admin / IhrPasswort)

#### Nach Deployment:
- [ ] Backup-Strategie für `./data` Ordner
- [ ] Monitoring/Alerting hinzugefügt
- [ ] SSL/HTTPS via Reverse Proxy (empfohlen)
- [ ] Firewall-Regeln angepasst

### 🆘 Was tun bei Problemen?

#### Sofort-Hilfe:
```bash
# Status prüfen
docker-compose -f docker-compose.prod.yml ps

# Logs anzeigen
docker-compose -f docker-compose.prod.yml logs -f

# Services neustarten
docker-compose -f docker-compose.prod.yml restart
```

#### Support anfordern:
1. Logs sammeln: `docker-compose logs > problem-logs.txt`
2. Config prüfen: `cat .env` (Passwörter schwärzen!)
3. System-Info: `docker info > system-info.txt`
4. An Daniel Heiniger senden mit Problembeschreibung

### 🔄 Update-Prozess

#### Mit latest Tag (einfach):
```bash
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d
```

#### Mit spezifischen Versionen (kontrolliert):
1. Daniel informiert über neue Version (z.B. 1.0.1)
2. `docker-compose.prod.yml` bearbeiten → neue Versionsnummer
3. `docker-compose pull && docker-compose up -d`

### 📞 Kontakt

**Bei technischen Fragen zum Deployment:**
- Daniel Heiniger
- GitHub Issues: [Link wenn verfügbar]

**Bei fachlichen Fragen zur Anwendung:**
- AIO Team Kanton Solothurn

---

## 🎯 TL;DR für erfahrene Docker-Crews:

```bash
# Dateien erhalten → .env konfigurieren → Deployen
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d
# → http://host:80 aufrufen, admin/passwort testen
```

**Das war's! 🚀**