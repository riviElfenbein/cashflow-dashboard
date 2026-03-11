# Technical Documentation

## Architecture

The entire app is a single `index.html` file. It has three logical sections:

```
index.html
├── <style>          CSS — dark theme, layout, components, modal
├── <body>           HTML — layout structure
│   ├── .header      Top bar
│   ├── .main        Flex container
│   │   ├── .file-sidebar    Right panel — file management
│   │   └── .dashboard       Main content area
│   │       ├── .month-range         Period selector
│   │       ├── .cards               Summary KPI cards
│   │       ├── .filter-section      Category chips
│   │       ├── .charts-grid         4 Chart.js canvases
│   │       ├── .biz-search-section  Business search + multi-export
│   │       └── .table-section       Transactions table (sort/filter/edit)
│   └── #catEditOverlay   Category edit modal (fixed overlay)
└── <script>         JavaScript — all logic
    ├── DATA               RAW array (embedded JSON or empty)
    ├── FILE REGISTRY      Multi-file tracking
    ├── CSV PARSER         parseCSVToRecords() with auto-delimiter + parseAmount()
    ├── REBUILD            rebuildAfterDataChange()
    ├── LOCAL STORAGE      loadFromLocalStorage(), saveToLocalStorage()
    ├── FILTERING          getFiltered(), monthlyAgg(), catAgg()
    ├── CHARTS             initCharts(), render(), renderYoy()
    ├── TRANSACTIONS TABLE renderTable(), sortTx()
    ├── CATEGORY EDIT      openCatEditModal(), closeCatEdit(), applyCatEdit()
    ├── MONTH SELECTORS
    ├── CATEGORY FILTER
    ├── YOY CATEGORY FILTER
    ├── BUSINESS SEARCH
    ├── DATA TOOLS         Upload, save, CSV export
    ├── EXPORT REPORT      Category-based PDF report
    └── INIT               window.onload
```

---

## State Variables

```js
const RAW            // All transaction records (mutated on file add/remove/edit)
const FILES          // File registry: [{id, name, count}]
let ALL_CATS         // string[] — all unique categories (rebuilt dynamically)
let ALL_MONTHS       // string[] — all unique months (rebuilt dynamically)
let selectedCats     // Set<string> — active categories (main filter)
let yoyCats          // Set<string> — active categories (YoY chart only)
let fromMonth        // "YYYY-MM" — range start
let toMonth          // "YYYY-MM" — range end
let nextFileId       // Auto-increment for file IDs

// Table
let _txSort          // { col: string, dir: 1|-1 } — current sort state
let _txFilter        // string — live text filter on business/category
let _shownRecords    // Record[] — currently rendered rows (for onclick index lookup)

// Category edit
let _editingRecord   // Record | null — the record currently being edited in modal
```

---

## Key Functions

### Data Flow

```
render()
  └── getFiltered()          Filter RAW by selectedCats + date range + !excluded
        ├── monthlyAgg()     Group by month → {income, expense, net}
        └── catAgg()         Group by category → {cat: total}
  ├── Update cards (DOM)
  ├── Update monthlyChart    Per-category stacked bars
  ├── Update catChart        Doughnut
  ├── Update balanceChart    Line
  ├── renderYoy()            Uses yoyCats independently
  └── renderTable()          Transactions table (sort + filter applied here)
```

### Transactions Table

```
renderTable()
  ├── Updates sort icons on <th> elements
  ├── Calls getFiltered()
  ├── Applies _txFilter (text search on business OR category)
  ├── Sorts by _txSort.col / _txSort.dir
  ├── Slices to 200 rows → _shownRecords
  └── Renders #txBody innerHTML with inline onclick="openCatEditModal(_shownRecords[i])"

sortTx(col)
  ├── Toggles direction if same column, else resets to default dir
  └── Calls renderTable()
```

### Category Edit Modal

```
openCatEditModal(record)
  ├── Sets _editingRecord = record
  ├── Populates #catEditBizLabel with business name + total count
  ├── Populates #catEditSelect with ALL_CATS (current category pre-selected)
  ├── Clears #catEditNew free-text input
  └── Shows #catEditOverlay

applyCatEdit(allForBusiness)
  ├── Reads newCat = #catEditNew.value.trim() || #catEditSelect.value
  ├── allForBusiness=true  → RAW.forEach mutate all matching r.business
  ├── allForBusiness=false → mutate _editingRecord only
  ├── closeCatEdit()
  ├── saveToLocalStorage()
  └── rebuildAfterDataChange()
```

### File Registry

```js
addFile(name, records)    // Tag records with _fileId, push to RAW, rebuild + save
deleteFile(fileId)        // Remove records with matching _fileId, rebuild + save
renderFileList()          // Re-render sidebar file list DOM
```

### Upload Pipeline

```
User drops / selects file
  └── loadFileIntoRegistry(file)
        ├── ext === 'json'  → JSON.parse → records[]
        └── ext === 'csv'   → parseCSVToRecords() → records[]
              └── addFile(name, records)
                    └── rebuildAfterDataChange() → render()
```

### CSV Parser

`parseCSVToRecords(raw)`:
1. Auto-detects delimiter: counts `;` vs `,` in the header line
2. `parseLine(line)` — inner function respecting quoted fields (handles commas inside values)
3. Maps columns via `colMap` (supports both Hebrew and English headers)
4. `parseAmount(str)` — strips `₪ $ € £` symbols, RTL marks, handles:
   - `"1,234.56"` → 1234.56 (dot is decimal)
   - `"1.234,56"` → 1234.56 (comma is decimal, European format)
   - `"1,234"` → 1234 (thousands separator)
5. `excluded` normalised to boolean: `"true"/"True"/"1"` → `true`, everything else → `false`
6. Debug output to console: delimiter, first record, non-zero amount count

### Export Pipeline

```js
exportReport()          // Category-based report (selected cats × months)
exportBizReport()       // Business-based report (selectedBusinesses set)
exportFilteredCSV()     // Raw CSV of getFiltered() result
saveCurrentData()       // Full RAW array as JSON download
```

### Rebuild After Data Change

```
rebuildAfterDataChange()
  ├── Rebuilds ALL_CATS from RAW (unique, sorted)
  ├── Rebuilds ALL_MONTHS from RAW (unique, sorted)
  ├── Rebuilds EXPENSE_CATS (all cats not in INCOME_CATS)
  ├── Adds new cats to selectedCats and yoyCats (preserves existing selections)
  ├── Rebuilds #fromMonth / #toMonth <select> elements
  ├── Rebuilds category filter chips (#catFilters)
  ├── Rebuilds YoY chips (#yoyCatFilters)
  └── Calls render()
```

### localStorage Persistence

```js
saveToLocalStorage()        // Saves { files: FILES, records: RAW } under LS_KEY
loadFromLocalStorage()      // Restores on page load; returns true if data found
clearLocalStorage()         // Removes key, clears RAW + FILES, shows empty state
updateStorageStatus()       // Updates the status line in the sidebar
```

---

## Adding a New Chart

1. Add a `<canvas id="myChart">` inside `.charts-grid` in HTML
2. Declare `let myChart;` with the other chart vars
3. In `initCharts()`, initialize with `new Chart(ctx, config)`
4. In `render()`, update `myChart.data` and call `myChart.update()`

---

## Adding a New Category Filter Behavior

Categories are classified at startup and after every `rebuildAfterDataChange()`:

```js
const INCOME_CATS = new Set(['הכנסות קבועות','הכנסות משתנות','הכנסות לא תזרימיות']);
const EXPENSE_CATS = new Set();  // All others — rebuilt dynamically
```

The `selectedCats` Set drives `getFiltered()`. Toggle membership and call `render()`.

---

## Updating the Embedded Dataset (`index.local.html` only)

The `RAW` array is generated by a Python script:

```bash
pip install numbers-parser
python3 extract.py   # reads "cashflow 2.numbers", outputs cashflow_cleaned.json
```

Then the JSON is embedded inline. To update:

1. Export from Numbers as CSV (`File → Export To → CSV`)
2. Upload via the sidebar — it merges with existing data
3. Or re-run the Python script and replace `const RAW = [...]` in `index.local.html`

---

## CSS Custom Properties

```css
--bg         #0f1117   Page background
--surface    #1a1d27   Card background
--surface2   #22263a   Nested elements, inputs
--border     #2e3350   All borders
--accent     #6c63ff   Primary purple — buttons, highlights, active sort
--accent2    #00d4aa   Secondary teal
--red        #ff6b8a   Expenses, errors
--green      #00d4aa   Income, success
--yellow     #ffd166   Warnings, secondary values
--text       #e8eaf6   Primary text
--text2      #9099c8   Secondary / muted text
```

---

## Chart Color Palette

20 colors cycling for multi-dataset charts (stacked monthly, donut):

```js
const COLORS = [
  '#6c63ff','#00d4aa','#ff6b8a','#ffd166','#06d6a0',
  '#ef476f','#118ab2','#ffc43d','#1b998b','#e9c46a',
  '#f4a261','#e76f51','#264653','#2a9d8f','#457b9d',
  '#a8dadc','#f1faee','#e63946','#606c38','#dda15e'
];
```

---

## Browser Compatibility

| Browser | Status |
|---|---|
| Chrome 90+ | ✅ Full support |
| Safari 15+ | ✅ Full support |
| Firefox 90+ | ✅ Full support |
| Edge 90+ | ✅ Full support |

Requires: ES2020, CSS Custom Properties, FileReader API, Blob URL API.
