# Todo-Docker - Deployment Documentation

Eine vollständige Anleitung zur Bereitstellung der Todo-Anwendung mit Docker und Docker Compose.

## Übersicht

Das Todo-Docker-Projekt ist eine vollständige Todo-Anwendung bestehend aus einem Next.js Frontend und einem Fastify Backend, die über Docker Compose orchestriert werden[^1]. Die Anwendung nutzt Submodule für die Trennung von Frontend und Backend.

## Voraussetzungen

- **Git** mit Submodule-Unterstützung
- **Docker** und **Docker Compose**
- **Node.js** und **pnpm** (für lokale Entwicklung)


## Installation

### 1. Repository klonen

```bash
git clone --recurse-submodules https://github.com/dhbw-webengineering/todo-docker.git
cd todo-docker
```

**Wichtig**: Das `--recurse-submodules` Flag ist erforderlich, um die Submodule `todo-app` und `todo-backend` automatisch zu klonen[^1].


### 2. Umgebungsvariablen konfigurieren

Kopieren Sie die Vorlage für die Umgebungsvariablen:

```bash
cp .env.template .env
```

Bearbeiten Sie die `.env`-Datei entsprechend Ihrer Konfiguration:

```env
# -------------- BACKEND CONFIGURATION --------------
# SMTP-Konfiguration
SMTP_HOST=smtp.example.com           # z. B. mail.knoep.de
SMTP_PORT=465                        # üblicherweise 465 (SSL) oder 587 (TLS)
SMTP_USER=your@email.com             # SMTP-Benutzername
SMTP_PASS=your-password              # SMTP-Passwort
SMTP_FROM=your@email.com             # Absenderadresse für ausgehende Mails

# Datenbankverbindung
DATABASE_URL=postgresql://user:password@host:port/dbname?sslmode=require
# Beispiel: postgresql://neondb_owner:pass@ep-example-pooler.neon.tech/neondb?sslmode=require

# Authentifizierungs-Keys
COOKIE_SECRET=your-cookie-secret     # Sicherer zufälliger String (z. B. 64 Zeichen)
JWT_SECRET=your-jwt-secret           # Ebenfalls sicherer zufälliger String

# Backend-Port
PORT=3001                            # Port, auf dem das Backend läuft

# Erlaubte Frontend-URL (für CORS)
FRONTEND_URL=http://localhost:3000   # URL des Frontends


# -------------- FRONTEND CONFIGURATION --------------
NEXT_PUBLIC_BACKEND_BASE_PATH=http://localhost:3001 # public prefix -> das ist für den nutzer einsehbar
```


## Deployment

### Mit Docker Compose

Die Anwendung kann mit einem einzigen Befehl gestartet werden:

```bash
docker-compose up --build
```


#### Container-Konfiguration

Das `docker-compose.yml` definiert zwei Services[^2]:

**Backend (todo-backend)**:

- Port: `3001`
- Nutzt die `.env`-Datei für Umgebungsvariablen
- Automatischer Neustart: `unless-stopped`

**Frontend (todo-app)**:

- Port: `3000`
- Abhängig vom Backend-Service
- Nutzt die `.env`-Datei für für Backend-URL
- Automatischer Neustart: `unless-stopped`


### Zugriff auf die Anwendung

Nach dem Start sind folgende Services verfügbar:

- **Frontend**: [http://localhost:3000](http://localhost:3000)
- **Backend-API**: [http://localhost:3001](http://localhost:3001)


## Projektstruktur

```
todo-docker/
├── todo-app/              # Frontend-Submodule (Next.js)
├── todo-backend/          # Backend-Submodule (Fastify)
├── docker-compose.yml     # Docker Compose-Konfiguration
├── .env.template         # Vorlage für Umgebungsvariablen
├── .env                  # Ihre Konfiguration (erstellen Sie diese)
└── .gitignore           # Git-Ignorier-Regeln
```


## Backend-Architektur

Das Backend folgt einer strukturierten Architektur[^3]:

### Hauptkomponenten

- **`src/app.ts`**: Einstiegspunkt der Anwendung
- **`src/plugins/`**: Fastify-Plugins (ua. JWT-Authentifizierung)
- **`src/routes/`**: HTTP-Endpunkte
- **`src/controllers/`**: Bedienung der Endpunkte
- **`src/services/`**: Datenzugriff und externe Integrationen
- **`src/prisma/`**: Datenbankschema und Client


### Verfügbare Endpunkte

Das Backend stellt folgende Hauptfunktionen bereit:

- **Authentifizierung**: Registrierung und Login
- **Todo-Management**: CRUD-Operationen für Todos und deren Eigenschaften (Kategorien, Tags)
- **Passwort-Reset**: E-Mail-basierte Passwort-Wiederherstellung


## Entwicklung

### Lokale Entwicklung

Für die lokale Entwicklung kann der Services auch einzeln gestartet werden:

```bash
# Backend
cd todo-backend
pnpm install
pnpm run dev

# Frontend
cd todo-app
pnpm install
pnpm run dev
```


### Logs einsehen

```bash
# Alle Services
docker-compose logs

# Spezifischer Service
docker-compose logs todo-backend
docker-compose logs todo-app
```


### Services stoppen

```bash
# Stoppen
docker-compose down

# Stoppen und Volumes löschen
docker-compose down -v
```


## Fehlerbehebung

### Häufige Probleme

1. **Submodule nicht geladen**: Stellen Sie sicher, dass Sie `--recurse-submodules` beim Klonen verwendet haben
2. **Port-Konflikte**: Überprüfen Sie, ob die Ports 3000 und 3001 verfügbar sind
3. **Datenbankverbindung**: Verifizieren Sie die `DATABASE_URL` in der `.env`-Datei
4. **SMTP-Konfiguration**: Testen Sie die E-Mail-Einstellungen für den Passwort-Reset

### Neu erstellen

Falls Probleme auftreten, können Sie die Container neu erstellen:

```bash
docker-compose down
docker-compose up --build --force-recreate
```

## Wichtig zu beachten

1. **Sichere Secrets**: Verwenden Sie starke, zufällige Werte für `COOKIE_SECRET` und `JWT_SECRET`
2. **Produktions-Datenbank**: Konfigurieren Sie eine dedizierte PostgreSQL-Instanz
3. **HTTPS**: Richten Sie einen Reverse-Proxy (nginx) mit SSL-Terminierung ein
4. **Monitoring**: Implementieren Sie Logging und Monitoring-Lösungen

[^1]: https://github.com/dhbw-webengineering/todo-docker

[^2]: https://github.com/dhbw-webengineering/todo-docker/blob/main/docker-compose.yml

[^3]: https://github.com/dhbw-webengineering/todo-app/tree/93374a8c9b05d597ee3e46e7016f4c07556db06e

[^4]: https://github.com/dhbw-webengineering/todo-docker/blob/main/.env.template

[^5]: https://github.com/dhbw-webengineering/todo-backend/tree/75c2004bb33278ff582f536d3532e6eb8f9d8a54/documentation

[^6]: https://github.com/dhbw-webengineering/todo-backend/blob/75c2004bb33278ff582f536d3532e6eb8f9d8a54/documentation/Projektstruktur.md

[^7]: https://www.markdownguide.org/tools/docsify/

[^8]: https://davewasmer.github.io/docify/customizing-docify

[^9]: https://www.mkdocs.org/user-guide/writing-your-docs/

[^10]: https://github.com/docsifyjs/docsify

[^11]: https://git-scm.com/docs/git

[^12]: https://github.com/BrickMMO/template-documentation-markdown

[^13]: https://github.com/docsifyjs/docsify/blob/develop/docs/markdown.md

[^14]: https://docs.github.com/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax

[^15]: https://experienceleague.adobe.com/en/docs/contributor/contributor-guide/writing-essentials/markdown

[^16]: https://docsify.js.org

[^17]: https://www.reddit.com/r/selfhosted/comments/1jufe6l/is_there_something_like_git_but_for_docs/

[^18]: https://community.fibery.io/t/is-there-a-complete-markdown-template-documentation-somewhere/4654

[^19]: https://davewasmer.github.io/docify/getting-started

[^20]: https://discourse.devontechnologies.com/t/markdown-template-style-like-read-the-docs-for-documents/78947

[^21]: https://opensource.com/article/20/7/docsify-github-pages

[^22]: https://www.markdownguide.org/getting-started/

[^23]: https://www.youtube.com/watch?v=TV88lp7egMw

[^24]: https://squidfunk.github.io/mkdocs-material/

[^25]: https://www.freecodecamp.org/news/how-to-write-good-documentation-with-docsify/

[^26]: https://markdown-it.github.io

