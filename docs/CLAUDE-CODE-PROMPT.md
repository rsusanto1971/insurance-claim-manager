# Claude Code Prompt

Copy and paste the prompt below into a Claude Code session to build the full-stack version of this app.

---

```
Build a full-stack web application called "Insurance Claim Manager" that helps users manage insurance claim inventories after a house fire or similar loss event.

## Core Features

### 1. PDF Upload & AI Extraction
- A clean upload screen where the user can drag-and-drop or browse to upload an insurance claim PDF
- Once uploaded, send the PDF to the Anthropic Claude API (claude-sonnet-4-6) using the document content block to extract all inventory items
- The AI should extract every item with these fields: item number, description, quantity, unit, RCV (replacement cost value), age/life, condition, depreciation %, depreciation amount, ACV (actual cash value / net value), original description, and purchase URL
- Show a progress indicator while extraction is running
- After extraction, confirm how many items were found before saving

### 2. Database
- Use SQLite (via better-sqlite3) for the database
- Three tables:
  - claims — stores claim metadata: id, filename, claim number, insured name, insurance company, upload date, total items, total RCV, total ACV
  - items — stores every extracted item: id, claim_id (foreign key), item_number, section/room, description, quantity, unit, rcv, age_life, condition, dep_pct, deprec, acv, orig_desc, url
  - purchases — stores purchase logs: id, item_id, replacement_description, store, amount_paid, purchase_date, receipt_number, notes, created_at
- Support multiple claims (multiple PDFs uploaded over time)

### 3. Inventory Browser
- After a claim is loaded, show a searchable, filterable list of all items
- Search by keyword across description, original description, room/section
- Filter by room/section (dropdown)
- Filter by status: All / Pending / Purchased
- Each item card shows: item number, description, section, ACV value, condition, purchased status
- Click an item to open a detail view

### 4. Item Detail View
- Show all fields: description, RCV, ACV, depreciation, condition, age/life, quantity, room, original description
- Clickable link to the insurance-suggested replacement URL
- A form to log a purchase: replacement description, store/retailer, amount paid, purchase date, receipt/order number, notes
- Show whether the purchase is within or over the ACV budget
- Allow editing or clearing a logged purchase

### 5. Submission Report
- A "Generate Report" button that produces a printable HTML report
- Report includes: claim info header, summary stats (items purchased, total ACV, total paid, remaining balance), and a table of all purchased items with original item, replacement purchased, store, date, amount paid, ACV allowed
- Report should be print-ready with clean formatting

## Tech Stack
- Backend: Node.js with Express
- Database: SQLite via better-sqlite3
- Frontend: Single HTML file served by Express, using vanilla JS (no frontend framework)
- PDF parsing: Send PDF as base64 to Anthropic Claude API using the messages endpoint with a document content block
- Use the Anthropic Node.js SDK (@anthropic-ai/sdk)
- No TypeScript — plain JavaScript throughout

## Project Structure
insurance-claim-manager/
├── server.js          # Express server, all API routes
├── database.js        # SQLite setup and all DB queries
├── public/
│   └── index.html     # All frontend HTML, CSS, and JS in one file
├── uploads/           # Temp storage for uploaded PDFs
├── package.json
└── .env               # ANTHROPIC_API_KEY

## API Routes (server.js)
- POST /api/upload — accepts multipart PDF, extracts items via Claude API, saves to DB, returns claim object
- GET /api/claims — list all claims
- GET /api/claims/:id/items — get all items for a claim with optional search/filter query params
- GET /api/items/:id — get single item with its purchase log
- POST /api/items/:id/purchase — save or update a purchase log
- DELETE /api/items/:id/purchase — clear a purchase log
- GET /api/claims/:id/report — return report data (all purchased items) for a claim

## Claude API Extraction Prompt
When sending the PDF to Claude, use this system prompt approach:

Send the PDF as a base64 document block along with a user message instructing Claude to extract all inventory items and return ONLY a valid JSON array with no markdown, no explanation, just the raw JSON array. Each object in the array should have: number, section, description, quantity, unit, rcv, age_life, condition, dep_pct, deprec, acv, orig_desc, url.

Tell Claude to look for items listed in tables with columns like DESCRIPTION, QUANTITY, UNIT, RCV, AGE/LIFE, COND, DEP%, DEPREC, ACV. Items are numbered sequentially. Sections/rooms appear as headers above groups of items.

Use max_tokens of 8000 for the extraction call.

## UI Design
- Clean, professional design suitable for an insurance/legal context
- Dark navy header with claim info
- White card-based layout
- Green accent for ACV values and within-budget status
- Red for over-budget status
- Sidebar list panel + main detail panel layout (like an email client)
- Mobile-responsive
- Show a dashboard on first load if no claims exist, prompting the user to upload their first PDF

## Environment
- App runs on port 3000
- ANTHROPIC_API_KEY loaded from .env file
- Include a README.md with setup instructions: npm install, add API key to .env, node server.js, open localhost:3000

## Error Handling
- If Claude API extraction fails or returns malformed JSON, show a clear error message to the user
- If a PDF has no recognizable items, tell the user and allow them to try again
- Validate that amount paid is a positive number
- Handle large PDFs gracefully (the extraction may take 30-60 seconds — show a spinner with a message like "AI is reading your PDF...")

## Additional Notes
- The SQLite database file should be created automatically on first run if it does not exist
- Use multer for file upload handling
- Clean up uploaded PDF files from the uploads/ folder after extraction is complete
- The purchases table should use INSERT OR REPLACE so saving a purchase is idempotent
- Include sample CSS that makes the report look good when printed (hide nav, show full table)
```
