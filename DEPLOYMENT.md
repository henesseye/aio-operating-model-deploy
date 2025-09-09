# 🐳 Production Deployment - AIO Betriebsmodell

## Für die Docker-Crew

### 📦 Was Sie von mir erhalten

**Dateien die Sie brauchen:**
1. `docker-compose.prod.yml` - Production Docker Compose Configuration
2. `data/model.json` - Vollständige Betriebsmodell-Daten mit allen 45+ Prozessen
3. Diese `DEPLOYMENT.md` - Anleitung

**Docker Images auf Docker Hub:**
- `henesseye/aio-operating-model-api:latest` (oder spezifische Version)
- `henesseye/aio-operating-model-web:latest` (oder spezifische Version)

### 🚀 Deployment auf Ihrem Docker Host

#### 1. **Dateien vorbereiten**
```bash
# Projekt-Verzeichnis erstellen
mkdir aio-operating-model
cd aio-operating-model

# Dateien von mir erhalten:
# - docker-compose.prod.yml
# - data/model.json (in data/ Unterordner)

# Verzeichnisstruktur sollte so aussehen:
# ./
# ├── docker-compose.prod.yml
# └── data/
#     └── model.json
```

#### 2. **Environment konfigurieren**
```bash
# .env Datei erstellen für Ihre Umgebung:
cat > .env << EOF
JWT_SECRET=IhrSicheresJWTSecret2024Mindestens32ZeichenLang!
ADMIN_PASSWORD=IhrSuperSicheresAdminPasswort2024!
PUBLIC_PORT=80
EOF
```

#### 3. **Deployen**
```bash
# Images pullen (automatisch latest oder spezifische Version)
docker-compose -f docker-compose.prod.yml pull

# Starten im Daemon-Modus
docker-compose -f docker-compose.prod.yml up -d

# Status prüfen
docker-compose -f docker-compose.prod.yml ps
```

#### 4. **Zugriff testen**
- **Web-Anwendung:** `http://YOUR-DOCKER-HOST:80` (oder Ihr PUBLIC_PORT)
- **Admin-Login:** Username: `admin`, Passwort: Wie in .env definiert

### ⚙️ Konfiguration

#### Environment Variablen (.env)
| Variable | Beschreibung | Beispiel |
|----------|--------------|----------|
| `JWT_SECRET` | Sicherer JWT-Schlüssel (min. 32 Zeichen) | `MySecureJWTKey2024...` |
| `ADMIN_PASSWORD` | Admin-Passwort für Login | `SecureAdmin123!` |
| `PUBLIC_PORT` | Externer Port für Web-App | `80`, `8080`, `443` |

#### Ports & Services
- **Web-App:** `http://host:${PUBLIC_PORT}`
- **API:** Läuft intern auf Port 3000 (nicht extern erreichbar)
- **Volumes:** `./data` → Persistent für alle Konfigurationen

### 🔧 Versionierung

#### Latest verwenden (Empfohlen für einfaches Update)
```yaml
# In docker-compose.prod.yml - bereits so konfiguriert
image: henesseye/aio-operating-model-api:latest
image: henesseye/aio-operating-model-web:latest
```

#### Spezifische Version (für stabiles Production)
```yaml
# In docker-compose.prod.yml anpassen:
image: henesseye/aio-operating-model-api:1.0.0
image: henesseye/aio-operating-model-web:1.0.0
```

### 🔄 Updates

#### Mit latest Tag:
```bash
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d
```

#### Mit spezifischer Version:
```bash
# 1. docker-compose.prod.yml bearbeiten (neue Versionsnummer)
# 2. Dann:
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d
```

### 📁 Datenpersistierung

```
./data/
├── model.json      # Betriebsmodell-Daten (von mir bereitgestellt)
└── users.json      # User-Daten (wird automatisch erstellt)
```

**⚠️ Wichtig:** Der `./data` Ordner muss persistent sein!
- **Backup:** Sichern Sie regelmäßig den `./data` Ordner
- **Permissions:** Stellen Sie sicher, dass Docker schreiben kann (uid 1000)

### 🛠️ Troubleshooting

#### Service Status prüfen:
```bash
docker-compose -f docker-compose.prod.yml ps
docker-compose -f docker-compose.prod.yml logs -f
```

#### Häufige Probleme:

**1. Port bereits belegt:**
```bash
# .env anpassen:
PUBLIC_PORT=8080
# Neustart:
docker-compose -f docker-compose.prod.yml up -d
```

**2. Admin Login funktioniert nicht:**
```bash
# .env prüfen - ADMIN_PASSWORD korrekt gesetzt?
# API Container neustarten:
docker-compose -f docker-compose.prod.yml restart api
```

**3. model.json nicht gefunden:**
```bash
ls -la ./data/model.json
# Muss vorhanden sein, sonst von mir erhalten!
```

**4. Container startet nicht:**
```bash
# Logs detailliert anzeigen:
docker-compose -f docker-compose.prod.yml logs api
docker-compose -f docker-compose.prod.yml logs web
```

### 🔐 Sicherheit

#### Produktions-Checkliste:
- [ ] **JWT_SECRET** mindestens 32 Zeichen, zufällig generiert
- [ ] **ADMIN_PASSWORD** komplex und eindeutig
- [ ] **Firewall** nur notwendige Ports geöffnet
- [ ] **HTTPS** Reverse Proxy (nginx/Apache) vor der Anwendung
- [ ] **Backup** `./data` Ordner regelmäßig sichern
- [ ] **Updates** Monitoring für neue Image-Versionen

#### HTTPS Setup (Optional):
```yaml
# In Ihrem Reverse Proxy (nginx/Apache)
# HTTP → HTTPS Redirect
# SSL-Terminierung vor der Anwendung
# Proxy zu http://localhost:${PUBLIC_PORT}
```

### 🏭 Production Best Practices

#### 1. **Monitoring hinzufügen:**
```yaml
# Optional: Monitoring Container hinzufügen
  monitoring:
    image: prom/prometheus
    # oder Ihr bevorzugtes Monitoring
```

#### 2. **Resource Limits:**
```yaml
# In docker-compose.prod.yml erweitern:
  api:
    image: henesseye/aio-operating-model-api:latest
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
```

#### 3. **Health Checks:**
```yaml
  api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### 📞 Support & Updates

**Bei technischen Problemen:**
- Logs sammeln: `docker-compose logs > deployment-logs.txt`
- Systeminfo: `docker info > docker-info.txt`
- An: Daniel Heiniger

**Neue Versionen:**
- Sie erhalten Benachrichtigung bei neuen Releases
- Update-Anweisungen werden mitgeliefert
- Breaking Changes werden im Voraus kommuniziert

### 🎯 Deployment-Checkliste

- [ ] `docker-compose.prod.yml` erhalten
- [ ] `data/model.json` erhalten und platziert
- [ ] `.env` mit sicheren Passwörtern erstellt
- [ ] Images gepullt: `docker-compose pull`
- [ ] Services gestartet: `docker-compose up -d`
- [ ] Status geprüft: `docker-compose ps`
- [ ] Web-App erreichbar unter konfigurierten Port
- [ ] Admin-Login getestet
- [ ] Backup-Strategie für `./data` Ordner eingerichtet

**Ready für Production! 🚀**