# POC-05-Real-Time-Payment-Map-----A-Sreebhala-Menon
# Real Rails — Real-Time Payments Map
### PoC #05 · Payments / Intelligence System · VAR Approved · All Fixes Applied

> **"You aren't just building a map. You are building a Global Competitiveness Index."**
> — Real Rails Master Manifesto

A production-grade intelligence dashboard that visualizes how money and data flow across the world's real-time payment infrastructure — 27 national schemes, 28 graph nodes, 44 directed edges, 6 system clusters, and 6 traceable transaction paths including the BIS NEXUS cross-border corridor.

---
## Preview
<img width="1912" height="918" alt="image" src="https://github.com/user-attachments/assets/e41e0420-72ef-4e9a-b91b-1ec5aa5f7f16" />
<img width="1886" height="921" alt="image" src="https://github.com/user-attachments/assets/2548eb90-63ad-466f-ae55-b6496dbfa5b0" />


---
## Project Status

| Item | Status |
|---|---|
| Phase 1 — World Map | ✅ Complete |
| Phase 2 — Connectivity Graph | ✅ Complete |
| VAR (Visualization Audit) | ✅ Final — 22 PASS · 0 PARTIAL · 1 NOTE · 0 FAIL |
| UAT Checklist | UAT Checklist ✅ Completed — 30 PASS · 0 FAIL · Tester Sign-off Done |
| Backend API | ✅ Live — 5 endpoints, 27 schemes |

---

## What This Builds

### Phase 1 — The Baseline
- **Interactive World Map** — Mercator dot-plot. Dot size scales with daily transaction volume. Click any country node for metadata.
- **Country Cards** — Scheme name, launch year, authority, P2P/P2B support, ISO 20022 status, transaction cap.
- **"Why This Matters" Panel** — How instant rails increase the velocity of money and drive GDP.
- **"Who Controls the Rail" Panel** — Central banks vs private consortia vs FinTech licensed participants.

### Phase 2 — Intelligence Layer
- **Relational Connectivity Graph** — 28 nodes, 44 directed edges, 6 cluster halos. Visualizes how money and data hop across Open Banking, RTP Rails, Card Networks, Correspondent Banking, and Supply Chain systems.
- **5 Edge Types** — payment (cyan) · data (violet) · auth (indigo) · settlement (emerald) · corridor (pink)
- **Cross-Border Corridor Edges** — NEXUS (UPI↔PayNow), PayNow↔FPS, EPC↔Pay.UK, SWIFT GPI — rendered as pink directed edges.
- **6 TX Paths** — End-to-end traceable flows with animated 900ms hop tracer.
- **Adoption Filters** — Filter by Leader / Established / Developing maturity tier.
- **Interactive Time-Slider** — Global spread of real-time rails 2004 → 2023.
- **Scheme Metadata Drill-Down** — API standard, protocol, interoperability per scheme.

---

## Repository Structure

```
poc-05-real-time-payments-map/
│
├── backend/
│   ├── main.py
│   ├── requirements.txt
│   └── README.md
│
├── frontend/
│   └── real-rails-connectivity-final.jsx
│
└── docs/
    ├── RealRails_PoC05_VAR_Final.docx
    └── RealRails_PoC05_UAT_Document.xlsx
```

---

## Backend — FastAPI

### Quick Start

```bash
python -m venv venv
source venv/bin/activate        # Mac/Linux
venv\Scripts\activate           # Windows

pip install -r requirements.txt
uvicorn main:app --reload --port 8000

# Test
open http://localhost:8000/docs
```

### API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/schemes` | All 27 RTP schemes + STATS meta |
| `GET` | `/api/schemes/{code}` | Single country (e.g. `IN`, `US`, `BR`) |
| `GET` | `/api/timeline` | Schemes grouped by launch year |
| `GET` | `/api/stats` | Aggregate counts |
| `GET` | `/api/download-sample` | First 5 records for demo |

### Zero Hallucination Policy

```bash
GET /api/schemes/XX
# → 404 { "error": "Country not found or status unknown" }
```

Unknown codes always return `404`. No fabricated scheme data is ever returned.

### STATS Response

```json
{
  "total_countries": 27,
  "leaders": 7,
  "established": 9,
  "developing": 11,
  "iso20022_adopted": 13,
  "earliest_launch": 2004,
  "latest_launch": 2023
}
```

---

## Frontend — Next.js

### Quick Start

```bash
npx create-next-app@latest real-rails-frontend
cd real-rails-frontend
# TypeScript ✅  Tailwind CSS ✅  App Router ✅

cp real-rails-connectivity-final.jsx src/components/
npm install && npm run dev
# → http://localhost:3000
```

### API Connection

The frontend ships with a complete static data fallback. The `useEffect` hook in `real-rails-connectivity-final.jsx` connects to the backend automatically:

```javascript
useEffect(() => {
  fetch("http://localhost:8000/api/schemes")
    .then(r => r.json())
    .then(d => { if (d?.data?.length) setApiStatus("live"); })
    .catch(() => setApiStatus("error"));
}, []);
```

Header indicator states:
- 🟢 **API LIVE** — Backend connected, live data
- 🔴 **API OFFLINE · STATIC** — Backend offline, static fallback active
- 🟡 **STATIC DATA** — Default pre-connection state

---

## Graph Data

| Metric | Value | Notes |
|---|---|---|
| Nodes (Entities) | 28 | Users + Banks + Rails + Networks + Intermediaries |
| Edges (Connections) | 44 | 40 flow edges + 4 corridor edges |
| Clusters | 6 | Initiators · Open Banking · RTP Rails · Card Networks · Correspondent · Supply Chain |
| Edge Types | 5 | payment · data · auth · settlement · corridor |
| TX Paths | 6 | See table below |
| Max Hops | 8 | Cross-Border SWIFT |

### Transaction Paths

| Path | Hops | Color | Settlement |
|---|---|---|---|
| Domestic RTP (UPI) | 7 | Cyan | 1.2 seconds |
| Card Payment | 8 | Indigo | 2–5 seconds |
| Cross-Border SWIFT | 8 | Rose | 1–3 days |
| Open Banking Data Flow | 5 | Violet | < 1 second |
| Supply Chain Finance | 6 | Amber | Same day |
| **NEXUS Cross-Border (UPI↔PayNow)** | **7** | **Pink** | **< 60 seconds** |

### Cross-Border Corridor Edges

| Edge | Type | Source |
|---|---|---|
| UPI → PayNow | corridor | BIS NEXUS pilot |
| PayNow → Faster Payments | corridor | MAS / Pay.UK interop |
| SEPA Instant → Faster Payments | corridor | EPC / Pay.UK |
| FedNow → SEPA Instant | corridor | SWIFT GPI |

---

## Visual DNA

| Token | Value | Usage |
|---|---|---|
| Background | `#030712` | Root div · SVG canvas · Header |
| Surface | `#0B1117` | Sidebar · Cards |
| Accent Primary | `#38BDF8` | Active states · Glow · CTA |
| Accent Secondary | `#818CF8` | Data edges · Auth overlays |
| Border | `#1F2937` | All containers · 1px |
| Emerald | `#34D399` | Settlement edges · Pass states |
| Corridor | `#F472B6` | Cross-border interop edges (NEXUS) |
| Layout | 70 / 30 | Main Stage / Intelligence Sidebar |

---

## Data Sources

| Source | Data Used | Schemes |
|---|---|---|
| **FedNow (Federal Reserve)** | Launch date · $500K cap · ISO 20022 · authority | US |
| **European Payments Council** | SCT Inst rulebook · €100K cap · RT1/TIPS | EU · DE |
| **World Bank GPSS** | Maturity tiers · authority names · launch years | All 27 |
| **BIS (NEXUS Project)** | Cross-border corridor pilot status · <60s settlement | UPI↔PayNow |
| **ISO 20022** | Message standard adoption per scheme | 13 schemes |

---

## Applied Fixes (Append-Only)

All changes applied following Discussion #10 append-only method — no truncation of existing code.

| Fix | What Changed | Result |
|---|---|---|
| FIX-01 | 4 corridor edges appended to EDGES array | Connections: 40 → 44 |
| FIX-02 | `corridor: "#F472B6"` added to EDGE_TYPE_COLORS | 5th edge type in header filter |
| FIX-04 | `flex-wrap` on cluster filter bar | Supply Chain button always visible |
| FIX-05 | NEXUS TX path appended to TX_PATHS | 6th path (7 hops, pink) |
| FIX-06 | SVG canvas `#040C18` → `C.bg` | Background chain aligned |
| FIX-07 | Sidebar `#08111A` → `C.surface` | DNA token compliance |
| API | `useEffect` fetch + `apiStatus` state | 3-state live/offline/static indicator |
| Legend | corridor added to INSIGHTS edge legend | All 5 types documented |
| Footer | BIS added to source attribution | All 5 sources cited |

---

## Governance

- Use **Zoho ID** for all repository contributions
- **Append-only rule** — never truncate existing code when making changes
- Always save a backup before any change
- **2-Hour Rule** — if visualization rendering fails, post to GitHub Discussion Forum immediately
- Never hardcode API keys — use `.env` for Mapbox or external credentials
- **Zero Hallucination** — unknown country code = `"Status Unknown"`, not fabricated data

---

*Real Rails Intelligence Library · PoC #05 · Sources: FedNow · EPC · World Bank · ISO 20022 · BIS (NEXUS) · Zero Hallucination Policy*
