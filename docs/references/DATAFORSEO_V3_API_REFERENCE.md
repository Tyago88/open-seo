# DataForSEO v3 API Reference & Implementation Guide

> **Stand:** 17.09.2026 | **Projekt:** [OpenSEO](https://github.com/every-app/open-seo) (`C:\Dev-Projekte\open-seo`)  
> **Kanonische Referenz-Collection:** `C:\Users\yairo\Downloads\dataforseo_xmpl_v3_postman\DataForSEO v3.postman_collection.json` (518 Requests, 29 MB)

---

## 1. Architektur-Überblick

DataForSEO dient als primärer Daten-Layer für SEO- und Keyword-Abfragen in OpenSEO und weiteren Solopreneur-Tools. Statt teurer All-in-One-Abonnements (Semrush/Ahrefs) basiert das Modell auf reiner **Pay-as-you-go API-Nutzung** mit Credit-Guthaben.

| Komponente | Rolle / Funktion |
|---|---|
| **DataForSEO API v3** | Datenlieferant für SERP, Labs, Keywords, Backlinks, OnPage und Business-Daten |
| **OpenSEO** | Lokale Open-Source-UI (PWA / Docker), Workflow-Orchestrator und Agent-Skill-Host |
| **Lokale Agent-Skills** | 9 automatisierte Workflows (`openseo-seo-audit`, `openseo-keyword-research`, etc.) |

---

## 2. Authentifizierung & Credentials

* **Dashboard & API-Schlüssel:** [`https://app.dataforseo.com/api-access`](https://app.dataforseo.com/api-access)
* **Verfahren:** Standard **HTTP Basic Authentication**
* **Kombination:** `api_login:api_password` codiert als Base64-String:
  $$\text{Base64}(\text{api\_login} : \text{api\_password})$$
* **HTTP-Header:**
  ```http
  Authorization: Basic <Base64-String>
  Content-Type: application/json
  Accept-Encoding: gzip
  ```
* **Ablage in OpenSEO:** In der lokalen Datei [`C:\Dev-Projekte\open-seo\.env`](file:///C:/Dev-Projekte/open-seo/.env) unter:
  ```env
  DATAFORSEO_API_KEY=dein_base64_schluessel
  ```

---

## 3. Die wichtigsten API-Familien (Postman-Mapping)

Aus der 518 Requests umfassenden v3-Postman-Collection sind folgende Module für Solopreneur- & Schweizer KMU-Projekte am relevantesten:

### A. DataForSEO Labs API (`/v3/dataforseo_labs/`)
Voraggregierte Datenbank mit über 4.8 Mrd. Keywords und historischen SERPs. **Kein Live-Crawl-Wartezeit, synchrone Antworten innert ~1–2 Sekunden.**

* **`keyword_overview/live`**: Basisprofil für Keywords (Suchvolumen, CPC, Intent, SERP-Features).
* **`ranked_keywords/live`**: Rankende Suchbegriffe einer Domain/URL.
* **`competitors_domain/live`**: Erkennung organischer Mitbewerber nach Schnittmengen.
* **`page_intersection/live`**: Keywords, bei denen mehrere Wettbewerber-Seiten gemeinsam ranken.

### B. SERP API (`/v3/serp/google/`)
Liefert vollständige Echtzeit-Suchergebnisseiten inkl. aller Google SERP-Features:
* Organische Positionen 1–100
* Local Pack (Maps-Kartenbox, z. B. Bärenplatz Bern)
* People Also Ask (PAA)
* Featured Snippets & AI Overviews
* News, Jobs, Shopping

### C. BusinessData API (`/v3/business_data/google/`)
* Google Business Profiles (Maps-Rankings, Adresse, Öffnungszeiten, Rezensionen, Google Q&A).
* Essenziell für lokale SEO-Audits (z. B. Bern / Schweizer Regionen).

### D. OnPage API (`/v3/on_page/`)
* Automatisierte Crawler-Prüfungen (Status-Codes, Meta-Tags, Canonical, Duplicate Content).
* Integrierter **Lighthouse-Audit** für Core Web Vitals und Mobile Performance.

---

## 4. Schweizer Geotargeting & Spezifikationen

Für präzise Resultate im Schweizer Markt müssen Anfragen immer explizit lokalisiert werden:

```json
[
  {
    "language_name": "German",
    "location_name": "Bern,Bern,Switzerland",
    "keyword": "wochenmarkt bern bio gemüse"
  }
]
```

* **ISO Location Code Schweiz:** `2756`
* **ISO Language Codes:** `de` (Deutsch), `fr` (Französisch), `it` (Italienisch)
* **Encoding:** Strikt **UTF-8** (insbesondere bei Umlauten: `ä`, `ö`, `ü`).

---

## 5. Payloads & Endpunkt-Beispiele

### 1. SERP Google Live Advanced
* **URL:** `POST https://api.dataforseo.com/v3/serp/google/organic/live/advanced`
```json
[
  {
    "language_name": "German",
    "location_name": "Switzerland",
    "keyword": "bio gemüse bern",
    "device": "desktop",
    "os": "windows"
  }
]
```

### 2. Labs Domain Ranked Keywords
* **URL:** `POST https://api.dataforseo.com/v3/dataforseo_labs/google/ranked_keywords/live`
```json
[
  {
    "target": "märitkorb.ch",
    "location_name": "Switzerland",
    "language_name": "German",
    "load_rank_absolute": true,
    "limit": 20
  }
]
```

### 3. OnPage Lighthouse Task
* **URL:** `POST https://api.dataforseo.com/v3/on_page/lighthouse/task_post`
```json
[
  {
    "url": "https://märitkorb.ch",
    "for_mobile": true,
    "categories": ["seo", "performance", "pwa"]
  }
]
```

---

## 6. Kosten-Guardrails & Solopreneur Best Practices

1. **Credit-Wallet Modell:** Kleines monatliches Testbudget ($10–$25) im Account limitieren.
2. **Caching-Strategie:**
   * SERP-Snapshots: 7–30 Tage cachen.
   * Keyword-Metriken (Volumen/CPC): 30–90 Tage cachen.
   * Backlink-Daten: monatlich/quartalsweise.
3. **Array-Batching:** DataForSEO berechnet Task-Pauschalen plus Item-Gebühren. Einzelabfragen vermeiden und Keywords in Arrays bündeln.
4. **Sandbox-Modus:** Schema- und Logikprüfungen können gebührenfrei gegen `https://sandbox.dataforseo.com/v3/` getestet werden.
