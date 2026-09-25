# How This Was Built

This tool was built entirely through a conversation with Claude — Anthropic's AI assistant. No prior coding experience was needed. The entire process, from problem description to working app, happened in a single session.

Here is the exact sequence of how it came together.

---

## Step 1: Describe the Problem

The starting point was a plain English description of the situation:

> "After my house fire, the insurance company made an inventory of all the items we lost in the fire and put this in a text list that they share with us. The list contains more than 500 items in a PDF of 100 pages. Each item contains the item description, part number, brand, model, price, depreciation value, and the net value, including the URL to where we can buy the item. We are allowed to shop and find similar items as long as the cost is below the net value. We usually have to scan through these long PDF documents for the item we are looking for, and then shop for alternatives. After we purchase the item and receive it, we submit the receipt to the insurance company along with the item we replace in the list. Is there a way for me to automate this?"

That was it. No technical requirements. No wireframes. Just a clear description of a painful manual process.

---

## Step 2: Clarify the Priorities

Before building anything, Claude asked three questions:

1. What parts of the process would help you most?
2. Can you upload the PDF?
3. How comfortable are you with tech tools?

The answers:
- Search the PDF easily + generate submission reports
- Yes, I can upload the PDF
- Moderate — I can follow instructions

This shaped the scope: a browser-based tool, no installation required, focused on search and reporting.

---

## Step 3: Upload the PDF

The actual Allstate insurance claim PDF was uploaded directly into the Claude conversation. The file was 110 pages and contained 914 line items organized by room across 26 sections.

The PDF structure per item:

```
DESCRIPTION | QUANTITY | UNIT | RCV | AGE/LIFE | COND. | DEP % | DEPREC. | ACV
```

Each major item also included:
- An original description field (the item as originally described before the insurer found a replacement)
- A URL to the insurer-suggested replacement product

---

## Step 4: Extract the Data

Claude used Python with the `pdfplumber` library to extract all items from the PDF.

**The extraction process:**

```python
import pdfplumber
import re
import json

items = []

with pdfplumber.open('claim.pdf') as pdf:
    full_text = ""
    for page in pdf.pages[1:]:  # pages 2 onward
        text = page.extract_text()
        if text:
            full_text += text + "\n"

# Parse item blocks using regex
# Each item starts with a number followed by a period
# e.g. "25. Keurig K-Classic Coffee Maker..."
```

The parser handled:
- Multi-line item descriptions (long product names that wrap across lines)
- Section/room headers (KITCHEN, BASEMENT, MASTER BEDROOM, etc.)
- Items with and without URLs
- Numbers formatted with commas (e.g. `2,699.99`)
- Depreciation displayed with brackets: `(462.86)`

**Final extraction results:**

| Metric | Count |
|---|---|
| Total items extracted | 914 |
| Items with ACV parsed | 910 |
| Items with purchase URLs | 864 |
| Sections/rooms identified | 26 |

---

## Step 5: Build the App

Once the data was extracted and confirmed, Claude wrote the full single-file HTML application with the JSON data embedded directly inside it.

The app was designed and coded in one pass, including:
- Search and filter logic
- Item detail panel
- Purchase logging form
- Budget comparison (paid vs. ACV)
- Report generator
- localStorage persistence

The entire HTML file is self-contained — no external libraries, no CDN calls, no backend.

---

## Step 6: Verify and Refine

After the first version was generated, a few issues were caught and fixed in follow-up messages:

- Section names were all showing as "General" — fixed by improving the section header detection regex
- Some items with long descriptions or URL fragments in the text were not parsing correctly — fixed with a fallback regex and a cleanup pass on description fields
- 4 items out of 914 still had no ACV parsed — these had unusual formatting in the PDF and were left as-is since the description was still captured

The final accuracy: **910 of 914 items fully parsed (99.6%)**.

---

## Total Time

The entire session — problem description, PDF upload, data extraction, app build, testing, and refinement — took **under 90 minutes**.

---

## What Made This Work

Three things:

**1. A clear problem description.** The initial prompt described the pain precisely — what the document contained, what the process required, and what outcome was needed. Claude did not need to guess.

**2. Real data.** Uploading the actual PDF allowed Claude to see exactly how the data was structured rather than guessing at a format. This made the extraction code accurate on the first attempt.

**3. Iterating in plain English.** Every correction was described conversationally ("the sections are all showing as General") rather than as a code change. Claude handled the debugging.

---

## Replicating This for Your Own Claim

If you want to build this for a different insurance PDF:

1. Upload your PDF to Claude at [claude.ai](https://claude.ai)
2. Describe the column structure of your document (DESCRIPTION, QUANTITY, UNIT, RCV, ACV, etc.)
3. Ask Claude to extract all items and build a searchable HTML app
4. Download the resulting HTML file

The extraction logic may need minor adjustments depending on your insurer's PDF format, but the approach is the same.

---

## Next Steps (Full-Stack Version)

The HTML file version has one limitation: purchase logs are stored in browser localStorage and do not sync across devices.

A full-stack version with Node.js, Express, SQLite, and a Claude API-powered PDF upload feature is documented in [FULL-STACK-VERSION.md](FULL-STACK-VERSION.md).
