**Insurance Claim Manager**
A browser-based tool that turns a 110-page insurance claim PDF into a searchable, trackable inventory — with purchase logging and a one-click submission report.

Built in a single Claude session. No coding experience required.

Demo video: Download 'insurance_claim_manager_demo.mp4' 

**The Problem**
After a house fire, our insurance company (Allstate) gave us a 110-page PDF listing 914 lost items worth $166,088 in replacement value. For each item, we were allowed to shop for alternatives as long as the cost stayed within the net value (ACV). The process looked like this:

Scroll through 110 pages to find the item
Look up alternatives online
Track what we bought and how much we paid
Compile receipts and match them back to line items
Submit everything to the insurance company

Doing this manually for 914 items across 26 rooms would take hundreds of hours. So I built a tool to automate it.

**What It Does**
Feature	Description
Instant Search	Search all 914 items by keyword — no more scrolling
Room Filter	Filter by section (Kitchen, Basement, Master Bedroom, etc.)
Status Filter	View All, Pending, or Purchased items
Item Detail	See RCV, ACV, depreciation, condition, age, and a link to the insurer-suggested replacement
Purchase Log	Record what you bought, where, how much, and when
Budget Check	Automatically shows if you are within or over the ACV budget
Report Generator	One click produces a printable submission report for the insurance company
Local Persistence	Purchase logs save to browser localStorage — no account needed
How to Use It

This is a single self-contained HTML file. No installation, no server, no dependencies.

Download insurance_manager.html
Open it in any browser (Chrome, Firefox, Safari, Edge)
Search for items, log purchases, generate your report

That is it.

**The Numbers**
These are real numbers from the actual claim this tool was built for.

Metric	Value
Total items in claim	914
Total replacement cost (RCV)	$166,088
Total net value available (ACV)	$93,656
Items with direct purchase links	864
Rooms and sections	26
PDF pages	110
Estimated hours saved	~500

Labor value of 500 hours saved:

Rate	Savings
$50 / hr	$25,000
$100 / hr	$50,000
$150 / hr	$75,000
Tech Stack
Layer	Technology
Runtime	Browser — no server needed
Language	Vanilla HTML, CSS, JavaScript
PDF Extraction	Python (pdfplumber) — run once to generate the embedded JSON
Data Storage	JSON embedded in HTML + browser localStorage for purchases
Dependencies	None

See TECH-STACK.md for full details.

**How It Was Built**
This was built entirely through conversation with Claude (Anthropic's AI). No code editor was opened. No Stack Overflow. The entire build — PDF extraction, app design, data parsing, UI — happened in a single chat session.

See HOW-IT-WAS-BUILT.md for the full story.

**Limitations**
Purchase logs are saved in your browser's localStorage. Clearing browser data will erase them.
The inventory data is hardcoded into this specific HTML file. To use it for a different claim PDF, you would need to re-run the extraction script.
No multi-device sync — logs only exist on the browser where you entered them.

A full-stack version with a real database and PDF upload is documented separately for those who want to extend this.

**Who This Is For**
Anyone dealing with an insurance claim inventory. This was built for a specific Allstate claim but the concept applies to any insurer that provides a structured PDF inventory list.

If you lost your home and your insurance company gave you a document like this — you can adapt this tool for your own situation.

**Author**
Ryan Susanto
LinkedIn: https://www.linkedin.com/in/ryan-susanto-31161b303/ 

**Built with Claude by Anthropic**
MIT License. Use it, modify it, share it freely.
