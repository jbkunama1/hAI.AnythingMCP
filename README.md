<p align="center">
  <img src="https://raw.githubusercontent.com/jbkunama1/hAI.AnythingMCP/main/Logo_I_AnyMCP.png" alt="hAI.AnythingMCP Logo" width="600">
</p>

# hAI.AnythingMCP

**Self-hosted AnythingMCP Gateway** — läuft als Portainer Stack im `highfishNetwork`

Basiert auf [HelpCode-ai/anythingmcp](https://github.com/HelpCode-ai/anythingmcp) · wandelt REST, SOAP, GraphQL & Datenbanken in MCP-Tools für Claude, ChatGPT, Gemini & Co. um

[![Upstream](https://img.shields.io/badge/upstream-HelpCode--ai%2Fanythingmcp-blue?logo=github)](https://github.com/HelpCode-ai/anythingmcp)
[![Docker](https://img.shields.io/badge/docker-helpcodeai%2Fanythingmcp-2496ED?logo=docker&logoColor=white)](https://hub.docker.com/r/helpcodeai/anythingmcp)
[![License](https://img.shields.io/badge/license-AGPL--3.0-green?logo=gnu)](https://github.com/HelpCode-ai/anythingmcp/blob/main/LICENSE)
[![Self-Hosted](https://img.shields.io/badge/deployment-self--hosted-orange?logo=portainer)](https://www.portainer.io/)
[![Network](https://img.shields.io/badge/network-highfishNetwork-blueviolet?logo=docker)]()
[![Stack](https://img.shields.io/badge/stack-Portainer-13BEF9?logo=portainer&logoColor=white)]()
[![Platform](https://img.shields.io/badge/platform-Debian%20%7C%20DietPi-red?logo=debian)]()
[![Tunnel](https://img.shields.io/badge/access-Cloudflare%20Tunnel-F38020?logo=cloudflare&logoColor=white)]()

---

## 📋 Inhaltsverzeichnis

- [Was ist AnythingMCP?](#was-ist-anythingmcp)
- [Features](#features)
- [Voraussetzungen](#voraussetzungen)
- [Schnellstart (Portainer Stack)](#schnellstart-portainer-stack)
- [Umgebungsvariablen](#umgebungsvariablen)
- [Services & Ports](#services--ports)
- [Öffentlicher Zugriff via Cloudflare Tunnel](#öffentlicher-zugriff-via-cloudflare-tunnel)
- [Netzwerk](#netzwerk)
- [Hinweise & Sicherheit](#hinweise--sicherheit)
- [Troubleshooting](#troubleshooting)
- [Upstream & Lizenz](#upstream--lizenz)

---

## Was ist AnythingMCP?

AnythingMCP ist ein selbst-gehostetes, KI-gestütztes **MCP-Gateway**. Es wandelt bestehende APIs und Datenbanken in [Model Context Protocol](https://modelcontextprotocol.io/) Tools um – ohne Code schreiben zu müssen.

- **REST, SOAP, GraphQL, SQL/NoSQL → MCP Tools**
- **175+ vorgefertigte Adapter** (Deutsche Bahn, DHL, weclapp, Shopware ...)
- **Kompatibel mit:** Claude, ChatGPT, Gemini, Copilot, Cursor
- **Voll self-hosted** – Credentials bleiben auf deiner Infrastruktur (AES-256-GCM)

---

## Features

| Feature | Details |
|---|---|
| 🔌 Connector-Typen | REST, SOAP/WSDL, GraphQL, Database, MCP-Bridge |
| 📥 Import-Formate | OpenAPI/Swagger, Postman, cURL, WSDL, GraphQL introspection |
| 🧩 Adapter | 175+ vorgefertigte (Deutsche Bahn, DHL, Etsy, Shopware …) |
| 🔐 Auth | OAuth2, Bearer, API Key, Basic, WS-Security, OAuth 1.0a |
| 📋 Audit Log | Jeder Tool-Call wird protokolliert |
| 👥 RBAC | Rollen & Tool-Whitelisting pro Nutzer |
| 🧠 Knowledge Graph | Beziehungen zwischen Connectors – optional, opt-in |
| 🐳 Docker-ready | `docker compose up` – läuft sofort inkl. Postgres & Redis |
| 🌐 Tunnel-ready | Läuft hinter Cloudflare Tunnel ohne Port-Forwarding am Router |

---

## Voraussetzungen

- Docker 24+
- Portainer (Stack-Deployment)
- Externes Netzwerk `highfishNetwork` muss vorhanden sein
- `.env` Datei auf Basis von `.env.example` anlegen
- (Optional, empfohlen) Laufender Cloudflare Tunnel für öffentlichen Zugriff ohne offene Ports

```bash
docker network create highfishNetwork
```

---

## Schnellstart (Portainer Stack)

1. **In Portainer:** `Stacks` → `Add Stack` → `Git Repository`
2. **Repository URL:** `https://github.com/jbkunama1/hAI.AnythingMCP.git`
3. **Repository reference:** `refs/heads/main`
4. **Compose path:** `docker-compose.yml`
5. **Environment-Variablen** eintragen (siehe unten)
6. **Deploy Stack**

Secrets vor dem Deploy generieren:

```bash
openssl rand -base64 48    # JWT_SECRET
openssl rand -hex 16       # ENCRYPTION_KEY (muss exakt 32 Zeichen sein)
openssl rand -base64 32    # NEXTAUTH_SECRET
openssl rand -hex 32       # MCP_BEARER_TOKEN
```

Nach dem Start:

- Web UI: `http://<host>:3003`
- MCP Endpoint: `http://<host>:4004/mcp/:serverId`
- API Docs: `http://<host>:4004/api/docs`
- ⚠️ **Sofort ersten Admin-Account anlegen!** Der erste registrierte User wird automatisch Admin.

---

## Umgebungsvariablen

Siehe [`.env.example`](./.env.example) für alle verfügbaren Variablen.

| Variable | Beschreibung | Beispiel |
|---|---|---|
| `POSTGRES_PASSWORD` | Postgres-Passwort | sicheres Passwort |
| `JWT_SECRET` | Signiert Session-/Auth-Tokens (Pflicht) | `openssl rand -base64 48` |
| `ENCRYPTION_KEY` | AES-256-GCM Schlüssel für Credentials (Pflicht, exakt 32 Zeichen) | `openssl rand -hex 16` |
| `NEXTAUTH_SECRET` | Next.js Auth Secret | `openssl rand -base64 32` |
| `MCP_BEARER_TOKEN` | Bearer-Token für MCP-Endpoint-Zugriff | `openssl rand -hex 32` |
| `MCP_AUTH_MODE` | Auth-Modus für MCP-Endpoint (`legacy`, u.a.) | `legacy` |
| `MCP_ALLOW_ANONYMOUS` | Anonymer Zugriff auf MCP-Endpoint erlauben (nicht empfohlen für öffentliche Domains) | `false` |
| `FRONTEND_URL` / `NEXTAUTH_URL` | Öffentliche URL der Web-UI | `https://haimcp.arbeitermili.eu` |
| `NEXT_PUBLIC_API_URL` / `SERVER_URL` | Öffentliche URL der API/MCP-Domain | `https://haimcpapi.arbeitermili.eu` |
| `CORS_ORIGIN` | Erlaubte Origin(s) für CORS | `https://haimcp.arbeitermili.eu` |
| `AMCP_PORT_UI` | Extern gemappter UI-Port | `3003` |
| `AMCP_PORT_MCP` | Extern gemappter MCP/API-Port | `4004` |
| `REDIS_URL` | Redis-URL (intern, automatisch gesetzt) | `redis://redis:6379` |

---

## Services & Ports

| Service | Intern | Extern (Standard) | Beschreibung |
|---|---|---|---|
| Web UI | 3000 | 3003 | Verwaltungsoberfläche |
| MCP Endpoint | 4000 | 4004 | `/mcp/:serverId` – Protokoll für AI-Clients |
| API Docs | 4000 | 4004 | `/api/docs` – Swagger REST-Dokumentation |
| Postgres | 5432 | – (nur intern) | Datenbank |
| Redis | 6379 | – (nur intern) | Cache (optional, App läuft auch ohne) |

---

## Öffentlicher Zugriff via Cloudflare Tunnel

Dieser Stack ist für den Betrieb hinter einem laufenden **Cloudflare Tunnel** ausgelegt – kein Port-Forwarding am Router nötig.

Beispiel-Setup mit zwei Subdomains:

| Subdomain | Zeigt auf (Host-Port) | Zweck |
|---|---|---|
| `haimcp.arbeitermili.eu` | `localhost:3003` | Web UI |
| `haimcpapi.arbeitermili.eu` | `localhost:4004` | MCP Endpoint + REST API |

**Ingress-Regel in der `config.yml` des Tunnels:**

```yaml
ingress:
  - hostname: haimcp.arbeitermili.eu
    service: http://localhost:3003
  - hostname: haimcpapi.arbeitermili.eu
    service: http://localhost:4004
  - service: http_status:404
```

**DNS-Routen anlegen:**

```bash
cloudflared tunnel route dns <tunnel-name> haimcp.arbeitermili.eu
cloudflared tunnel route dns <tunnel-name> haimcpapi.arbeitermili.eu
docker restart cloudflared
```

**MCP-Client-Verbindung:** Der vollständige MCP-Endpoint für einen Connector lautet:

```
https://haimcpapi.arbeitermili.eu/mcp/<serverId>
```

mit Header:

```
Authorization: Bearer <MCP_BEARER_TOKEN>
```

---

## Netzwerk

Dieser Stack nutzt das **externe Docker-Netzwerk `highfishNetwork`**, damit er sich mit anderen Diensten im Stack verbinden kann.

```yaml
networks:
  highfishNetwork:
    external: true
```

Falls das Netzwerk noch nicht existiert:

```bash
docker network create highfishNetwork
```

---

## Hinweise & Sicherheit

- ⚠️ **Erster Start:** Sofort nach dem Start den Admin-Account anlegen. Der erste registrierte User wird automatisch Admin.
- 🔒 Credentials werden AES-256-GCM verschlüsselt gespeichert (`ENCRYPTION_KEY`, exakt 32 Zeichen).
- 📋 Jeder Tool-Call wird im Audit-Log protokolliert.
- 🔑 `JWT_SECRET`, `ENCRYPTION_KEY`, `NEXTAUTH_SECRET` und `MCP_BEARER_TOKEN` regelmäßig rotieren.
- 🌐 Bei öffentlicher Domain (z. B. via Cloudflare Tunnel) immer `MCP_BEARER_TOKEN` setzen statt `MCP_ALLOW_ANONYMOUS=true`.
- 🗄️ Redis ist optional — läuft die App ohne Redis-Verbindung, wird Caching automatisch deaktiviert, ohne dass der Container abstürzt.

---

## Troubleshooting

| Fehler | Ursache | Lösung |
|---|---|---|
| `Cannot resolve environment variable: DATABASE_URL` | Kein Postgres-Service / Variable fehlt | Postgres-Service im Compose sicherstellen, `.env` prüfen |
| `[secrets] JWT_SECRET is not set` | Pflicht-Secret fehlt | `JWT_SECRET` in `.env` / Portainer-Stack setzen |
| `ECONNREFUSED ...:4000` | Backend crasht beim Start (meist fehlende Secrets) | Container-Logs prüfen, alle Pflicht-Secrets setzen |
| `401` am MCP-Endpoint, `www-authenticate: Bearer` | Kein/falscher `MCP_BEARER_TOKEN` gesendet | Echten Token-Wert (nicht Platzhalter!) im Client/Header verwenden |
| `Failed to fetch (check CORS?)` im Browser-Client | Falsche Domain/Endpoint-Pfad oder CORS_ORIGIN passt nicht zur Client-Origin | Richtige `haimcpapi`-Domain + Connector-ID nutzen, `CORS_ORIGIN` anpassen |
| `Redis connection error` (wiederholt) | Kein Redis-Service im Stack | Redis-Service ergänzen oder ignorieren (App läuft trotzdem, nur ohne Cache) |

---

## Upstream & Lizenz

- **Upstream:** [HelpCode-ai/anythingmcp](https://github.com/HelpCode-ai/anythingmcp)
- **Lizenz:** [AGPL-3.0](https://github.com/HelpCode-ai/anythingmcp/blob/main/LICENSE)
- **Maintainer dieses Stacks:** [jbkunama1](https://github.com/jbkunama1) · [realteacher.de](http://www.realteacher.de)

---

> ⭐ Wenn dir AnythingMCP gefällt, gib dem [Upstream-Repo](https://github.com/HelpCode-ai/anythingmcp) einen Star!
