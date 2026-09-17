# OpenSEO MCP Server Integration Plan (Multi-CLI)

> **Stand:** 17.09.2026 | **Projekt:** [OpenSEO](https://github.com/every-app/open-seo) (`C:\Dev-Projekte\open-seo`)  
> **Status:** Container läuft (`http://localhost:3001`), MCP-Endpunkt verifiziert (46 Tools)

---

## 1. Ausgangslage & Verifikation

- **Lokaler Endpunkt:** `http://localhost:3001/mcp`
- **Authentifizierung:** `AUTH_MODE=local_noauth` (lokaler Admin `admin@localhost`, kein Auth-Token nötig)
- **DataForSEO Anbindung:** Aktiv via `DATAFORSEO_API_KEY` (Live-Verbindung verifiziert)
- **OpenRouter KI:** Aktiv via `OPENROUTER_API_KEY` (SAM-Agent bereit)
- **MCP Protokoll:** JSON-RPC 2.0 via HTTP SSE / POST (`Accept: application/json, text/event-stream`)
- **Verfügbare Tools (46 Stück):**
  - **Context & Management (kostenlos):** `whoami`, `list_projects`, `create_project`, `get_project_context`, `update_project_context`, `list_saved_keywords`, `save_keywords`
  - **Keywords & SERPs:** `research_keywords`, `get_keyword_metrics`, `get_serp_results`, `find_serp_competitors`, `get_ranked_keywords`
  - **Rank Tracking:** `create_rank_tracker`, `get_rank_tracker`, `run_rank_tracker`, `estimate_rank_tracker_cost`, `add_rank_tracking_keywords`, `remove_rank_tracking_keywords`
  - **Audits & OnPage:** `run_site_audit`, `get_audit_status`, `get_audit_issues`, `get_audit_pages`, `inspect_urls`
  - **Backlinks:** `get_backlinks_overview`, `get_backlinks_profile`
  - **Local SEO & Maps:** `search_local_businesses`, `get_business_profile`, `get_business_reviews`, `get_business_updates`, `get_google_business_questions`, `get_local_serp_results`, `get_local_rank_grid`, `list_business_categories`
  - **Analytics & GSC:** Google Analytics GA4 (8 Tools) & Google Search Console Performance/Opportunities

---

## 2. Architektur-Optionen für die globale Agenten-Anbindung

Da OpenSEO ein HTTP-basierter Server ist, gibt es zwei Integrationspfade:

### Option A: Lokaler MCP-Server (`http://localhost:3001/mcp`) — *Bedarfsgesteuert*
- **Vorteil:** Nutzt direkt deine eigene Docker-Instanz, deine lokalen Projekte, gespeicherten Keywords und SQLite/D1-Datenbank auf der Maschine.
- **Bedingung:** Der Docker-Container `open-seo-open-seo-1` muss laufen (`docker compose up -d`). Gemäss `DOCKER-RULES.md` läuft Docker lokal nur bei aktiver Arbeit. Wenn der Container gestoppt ist (`docker compose down`), antwortet der HTTP-Port nicht.
- **Konfiguration:**
  ```json
  "openseo-local": {
    "httpUrl": "http://localhost:3001/mcp"
  }
  ```

### Option B: Cloud/Hosted OpenSEO MCP (`https://app.openseo.so/mcp`) — *Immer verfügbar*
- **Vorteil:** Läuft auch dann, wenn Docker lokal gestoppt ist. Ideal für mobile Agenten oder leichtgewichtige Abfragen ohne laufenden Docker-Daemon.
- **Bedingung:** Erfordert einmaligen Login / API-Key unter `app.openseo.so`.

---

## 3. Empfohlener Rollout-Plan (3 Schritte)

### Schritt 1: Lokale Anbindung in Gemini CLI (`.gemini/settings.json`)
Eintrag unter `mcpServers`:
```json
"openseo": {
  "httpUrl": "http://localhost:3001/mcp"
}
```
*(Kann alternativ über `mcp-remote` oder Stdio-Proxy gekapselt werden, falls reine HTTP-Transports im spezifischen CLI-Client ein Stdio-Interface erwarten).*

### Schritt 2: Anbindung in Claude Code (`~/.claude.json` / Claude Config)
Für Claude Code:
```json
"openseo": {
  "type": "http",
  "url": "http://localhost:3001/mcp"
}
```

### Schritt 3: Docker-Workflow & Aliasse
Da Docker gemäss Regelwerk nicht permanent im Autostart laufen darf:
- Schnellstart-Befehl für SEO-Sitzungen: `cd C:\Dev-Projekte\open-seo && docker compose up -d`
- Feierabend-Befehl: `cd C:\Dev-Projekte\open-seo && docker compose down`
- Die bereits installierten globalen Skills (`openseo-seo-audit`, `openseo-keyword-research` etc.) greifen nahtlos auf die 46 Tools zu, sobald der Container aktiv ist.
