# 💰 Cashflow Dashboard

A personal finance dashboard built as a single self-contained HTML file — no server, no dependencies to install, no internet required after first load (Chart.js loaded from CDN).

---

## Getting Started

1. Open `index.html` in any modern browser (Chrome, Safari, Firefox, Edge)
2. The dashboard loads with embedded data from `cashflow 2.numbers`
3. No installation needed

---

## Features

### 📊 Summary Cards
Four cards at the top showing for the selected period:
- Average monthly income
- Average monthly expenses
- Average monthly balance (net)
- Total transaction count

### 📅 Period Selector
Select a custom date range using the **From / To** dropdowns, or click **"12 חודשים אחרונים"** for a quick last-12-months view.

### 🏷️ Category Filter
- 43 expense/income categories displayed as colored chips
- Click any chip to toggle it on/off
- Quick actions: **Select All**, **Clear All**, **Expenses Only**, **Income Only**
- All charts and the summary cards update instantly

### 📈 Charts

| Chart | Description |
|---|---|
| **Monthly Breakdown** | Stacked bar chart — each selected category is a separate color per month |
| **Category Donut** | Top 15 expense categories with amount + percentage in legend |
| **Monthly Balance** | Line chart showing net income/expense per month |
| **Year-over-Year** | Grouped bars comparing each month to the same month the prior year |

#### Year-over-Year category selector
The YoY chart has its own independent category filter — choose specific categories to compare without affecting the main dashboard filter. Use **"סנכרן עם הסינון הראשי"** to copy the main filter selection.

### 🔍 Business Search
- Type any business name — autocomplete dropdown shows matches ranked by transaction frequency
- Arrow keys ↑↓ to navigate, Enter to select
- Results show: total expenses, monthly average, active months, category, date range, full transaction list
- **+ הוסף לבחירה** — add the business to the multi-select export list

### 🗂️ Multi-Business Export
1. Search and add multiple businesses to the selection bar (shown as chips)
2. Click **"📄 ייצוא עסקים נבחרים"** to generate a printable report
3. The report opens in a new tab — use **"🖨️ הדפסה / שמור PDF"** to save as PDF

### 📋 Transactions Table
Full sortable list of transactions for the selected filters (shows latest 100). Columns: Month, Business, Category, Date, Amount.

### 📄 Export Report (Category-based)
Click **"📄 ייצוא דו"ח"** in the category filter section to generate a lawyer/accountant-friendly report with:
- Summary totals
- Average monthly expense per category
- Month-by-month breakdown table per category

---

## File Sidebar (Right Panel)

The sidebar shows all currently loaded data files.

| Action | How |
|---|---|
| **View loaded files** | Listed automatically with name and record count |
| **Add a file** | Drag & drop or click the **➕** area |
| **Remove a file** | Click **✕** next to any uploaded file |
| **Save all data** | **💾 שמור כ-JSON** — downloads a JSON snapshot |
| **Export filtered CSV** | **📊 ייצוא CSV מסונן** — exports only currently filtered records |

---

## Supported File Formats for Upload

### JSON
Files previously saved from this dashboard via **"שמור כ-JSON"**.

```json
[
  {
    "month": "2025-01",
    "business": "סופר-פארם",
    "category": "פארמה",
    "date": "15/01/2025",
    "amount": -234.5,
    "payment": "cal",
    "source": "creditCard",
    "excluded": false
  }
]
```

### CSV — Dashboard Export Format
Files exported via **"ייצוא CSV מסונן"**. Headers (English):

```
month,business,category,date,amount,payment,source
```

### CSV — Bank / Numbers Export Format
Files exported directly from the bank app or Numbers. Supported Hebrew headers:

| Hebrew Header | Field |
|---|---|
| שייך לתזרים חודש | month |
| שם העסק | business |
| קטגוריה בתזרים | category |
| תאריך התשלום | date |
| סכום | amount |
| אמצעי התשלום | payment |
| סוג מקור | source |
| האם מוחרג מהתזרים? | excluded |

> **Note:** `.numbers` files cannot be uploaded directly. Export as CSV from Numbers first: `File → Export To → CSV`.

---

## Data Structure

Each transaction record:

| Field | Type | Description |
|---|---|---|
| `month` | `"YYYY-MM"` | Billing month |
| `business` | string | Business/merchant name |
| `category` | string | Category label |
| `date` | `"DD/MM/YYYY"` | Transaction date |
| `amount` | number | Negative = expense, Positive = income |
| `payment` | string | Payment method (cal, discount, etc.) |
| `source` | string | `creditCard` or `checkingAccount` |
| `excluded` | boolean | If true, excluded from cashflow calculations |
| `_fileId` | number | Internal — which loaded file this record belongs to |

---

## Project Structure

```
cashflow-dashboard/
├── index.html        # The entire app — data + UI + logic in one file
└── README.md         # This file
```

The app is intentionally a **single HTML file** for portability — share it, open it offline, or email it.

---

## Tech Stack

| Library | Version | Usage |
|---|---|---|
| [Chart.js](https://www.chartjs.org/) | 4.4.0 | All charts (CDN) |
| Vanilla JS | ES2020 | All logic |
| CSS Variables | — | Dark theme |

No build step. No npm. No framework.

---

## Known Limitations

- Large files (>10,000 records) may slow down chart rendering
- `.numbers` files must be exported to CSV first
- The embedded dataset is baked into `index.html` — to update the base dataset, re-run the Python extraction script
- Browser's file access is sandboxed — saving to disk requires the download approach used here
