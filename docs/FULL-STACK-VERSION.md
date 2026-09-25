# Full-Stack Version

The single HTML file version works well for one person on one device. This document covers the full-stack version — a Node.js app with a real database and AI-powered PDF upload — for those who want to extend this into a shareable tool.

---

## Why a Full-Stack Version

The HTML file has three limitations:

| Limitation | Impact |
|---|---|
| Inventory is hardcoded | Cannot be used with a different claim PDF without rebuilding the file |
| localStorage only | Purchase logs do not sync across devices or family members |
| No upload feature | Every new claim requires a developer to re-run the extraction script |

The full-stack version solves all three.

---

## What Changes

| Feature | HTML Version | Full-Stack Version |
|---|---|---|
| PDF processing | Python script, run once | Upload in the browser, AI extracts on the fly |
| Data storage | Embedded JSON + localStorage | SQLite database |
| Multi-device | No | Yes |
| Multi-claim | No | Yes — upload multiple PDFs |
| Family sharing | No | Possible (shared server) |
| Setup required | None | Node.js + API key |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js |
| Framework | Express |
| Database | SQLite (via better-sqlite3) |
| PDF upload | multer |
| AI extraction | Anthropic Claude API (@anthropic-ai/sdk) |
| Frontend | Single HTML file served by Express, vanilla JS |

---

## Architecture

```
Browser
  |
  | HTTP
  v
Express Server (server.js)
  |           |
  |           | Claude API (PDF extraction)
  |           v
  |     Anthropic API
  |
  | SQLite
  v
database.js (better-sqlite3)
  |
  v
claims.db
```

---

## Database Schema

```sql
-- One row per uploaded PDF
CREATE TABLE claims (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  filename TEXT,
  claim_number TEXT,
  insured_name TEXT,
  insurance_company TEXT,
  upload_date TEXT,
  total_items INTEGER,
  total_rcv REAL,
  total_acv REAL
);

-- One row per extracted item
CREATE TABLE items (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  claim_id INTEGER REFERENCES claims(id),
  item_number INTEGER,
  section TEXT,
  description TEXT,
  quantity TEXT,
  unit TEXT,
  rcv TEXT,
  age_life TEXT,
  condition TEXT,
  dep_pct TEXT,
  deprec TEXT,
  acv TEXT,
  orig_desc TEXT,
  url TEXT
);

-- One row per logged purchase
CREATE TABLE purchases (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  item_id INTEGER REFERENCES items(id),
  replacement_description TEXT,
  store TEXT,
  amount_paid REAL,
  purchase_date TEXT,
  receipt_number TEXT,
  notes TEXT,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(item_id)
);
```

---

## API Routes

| Method | Route | Description |
|---|---|---|
| POST | /api/upload | Upload PDF, extract items via Claude, save to DB |
| GET | /api/claims | List all uploaded claims |
| GET | /api/claims/:id/items | Get all items for a claim (supports search and filter) |
| GET | /api/items/:id | Get single item with purchase log |
| POST | /api/items/:id/purchase | Save or update a purchase log |
| DELETE | /api/items/:id/purchase | Clear a purchase log |
| GET | /api/claims/:id/report | Get all purchased items for a claim (for report) |

---

## How to Build It

This version was designed to be built with Claude Code. Copy and paste the full prompt from the repository's [CLAUDE-CODE-PROMPT.md](CLAUDE-CODE-PROMPT.md) into a Claude Code session.

**Prerequisites:**
- Node.js installed (nodejs.org)
- An Anthropic API key (console.anthropic.com)

**Setup after Claude Code generates the files:**

```bash
cd insurance-claim-manager
npm install
cp .env.example .env
# Add your ANTHROPIC_API_KEY to .env
node server.js
# Open http://localhost:3000
```

---

## Cost Estimate

The Claude API is used once per PDF upload for extraction.

| PDF Size | Estimated Cost |
|---|---|
| 50 pages / ~300 items | ~$0.10 |
| 110 pages / ~900 items | ~$0.20 to $0.30 |
| 200 pages / ~1,500 items | ~$0.50 |

All estimates based on claude-sonnet-4-6 pricing. New Anthropic accounts receive free credits sufficient for several extractions.

---

## Deployment (Optional)

To share the app with family members or friends, deploy to a free hosting service:

| Platform | Free Tier | Notes |
|---|---|---|
| Railway | Yes | Easiest — connects to GitHub, auto-deploys |
| Render | Yes | Similar to Railway, slightly slower cold start |
| Fly.io | Yes | More control, slightly more setup |

Once deployed, anyone with the URL can upload their own PDF and use the full app. No installation required on their end.
