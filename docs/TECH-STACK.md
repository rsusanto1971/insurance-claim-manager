# Tech Stack

This project has two components: the extraction script that reads the PDF, and the browser app that runs the interface.

---

## PDF Extraction (Python)

Used once to pull all items out of the insurance PDF and generate the JSON data embedded in the HTML app.

| Tool | Version | Purpose |
|---|---|---|
| Python | 3.x | Runtime |
| pdfplumber | latest | Extract text from PDF with layout awareness |
| re (stdlib) | — | Regex parsing of item fields |
| json (stdlib) | — | Serialize extracted data |

**Why pdfplumber over pypdf?**

pdfplumber preserves spatial layout better than basic pypdf. Insurance claim PDFs are dense tables — character positioning matters when trying to split field values correctly.

**Extraction approach:**

The PDF did not have machine-readable tables (no `extract_tables()` hits). Everything was extracted from raw text using regex patterns that matched the fixed column structure:

```
(\d+\.\d+)\s*EA\s+([\d,]+\.?\d*)\s+([\d,]+\.?\d*)
\s+(\S+)\s+([\w\s.]+?)\s+(\d+\.?\d*%(?:\[M\])?|NA)
\s+\(([\d,]+\.?\d*)\)\s+([\d,]+\.?\d*)
```

A fallback regex handled items where the description wrapped across multiple lines before the numeric fields appeared.

---

## Browser App (HTML/CSS/JS)

A single self-contained `.html` file. No build step, no package manager, no server.

| Layer | Choice | Reason |
|---|---|---|
| Markup | HTML5 | Standard |
| Styling | Vanilla CSS | No framework needed for this scope |
| Logic | Vanilla JavaScript (ES6+) | No dependencies, runs anywhere |
| Data | JSON embedded in `<script>` tag | Self-contained, no fetch required |
| Persistence | Browser `localStorage` | Zero setup, survives page refresh |
| Fonts | System font stack | No external CDN calls |

**Data flow:**

```
PDF → pdfplumber → JSON → embedded in HTML → rendered by JS → localStorage for purchases
```

**Why a single HTML file?**

The goal was something anyone could open without installing anything. A single file can be emailed, shared via Google Drive, or dropped on a USB stick. There is no dependency to break, no server to run, no package to install.

---

## Data Structure

Each item stored in the embedded JSON:

```json
{
  "number": 25,
  "section": "BASEMENT",
  "description": "Keurig K-Classic Coffee Maker - Rhubarb",
  "quantity": "1.00",
  "unit": "EA",
  "rcv": "99.99",
  "age_life": "1/10 yrs",
  "condition": "Above Avg.",
  "dep_pct": "6%",
  "deprec": "6.00",
  "acv": "93.99",
  "orig_desc": "Keurig K10, Coffee maker",
  "url": "https://www.keurig.com/..."
}
```

Each purchase log stored in localStorage:

```json
{
  "desc": "Keurig K-Slim Coffee Maker",
  "store": "Amazon",
  "price": "79.99",
  "date": "2024-11-15",
  "notes": "Order #113-4829201-3847261"
}
```

localStorage key: `insurance_purchases`
localStorage value: object keyed by item number

---

## Performance

| Metric | Value |
|---|---|
| Total items in JSON | 914 |
| Raw JSON size | ~394 KB |
| Total HTML file size | ~391 KB |
| Search response time | Instant (client-side filter) |
| No. of network requests at runtime | 0 |

The app is fully offline after the initial file load.

---

## Browser Compatibility

Tested and works in:
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

Requires JavaScript enabled. No other dependencies.

---

## What Was Not Used

| Skipped | Why |
|---|---|
| React / Vue / Angular | Overkill for a single-user local file |
| Tailwind / Bootstrap | Unnecessary external dependency |
| IndexedDB | localStorage is sufficient for this data volume |
| Backend server | Nothing to serve — file runs locally |
| PDF.js | pdfplumber in Python gave better extraction accuracy |
| TypeScript | Not needed for a script this size |
