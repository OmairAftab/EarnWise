# Earnwise

**Earned Value Analysis, made easier.**

Earnwise is a browser-based project cost analysis dashboard. Enter work package budgets, spending, and progress manually, or import an Excel/CSV file. The app calculates project performance, visualizes costs, flags unusual expenses with local machine learning, and generates downloadable reports.

Built with HTML, CSS, and JavaScript. No installation, backend, database, or API key is required to use the app.

## What can you do with it?

- **Monitor a project:** View the estimated budget, actual spending, cost variance, and forecast final cost.
- **Manage work packages:** Add, edit, delete, and search cost entries, and customize the project name.
- **Import spreadsheets:** Upload CSV, XLSX, or XLS files, append or replace data, and download an input template.
- **Compare costs visually:** View budget-versus-actual bars and cumulative Planned Value, Earned Value, and Actual Cost lines.
- **Identify unusual entries:** Run a local Isolation Forest model to flag unusual cost or progress patterns.
- **Understand the numbers:** Read a metric-based project summary and open the built-in EVA calculation guide.
- **Export results:** Download an HTML report with charts, calculations, AI findings, and work package details; export results as CSV; print or save a report as PDF.
- **Continue later:** Save project data in the current browser using localStorage.

## Quick start

1. Download or clone this repository.
2. Keep the files and `vendor/` folder together.
3. Open `index.html` in a modern browser.
4. Explore the included demonstration project, or clear the sample rows and enter your own work packages.

For an optional local web server, run this command inside the project folder:

```bash
python -m http.server 8000
```

Then open http://localhost:8000. Python is needed only for this optional server.

## A typical workflow

1. Set your project name and currency label.
2. Click **Add cost** to enter work packages, or **Import data** to upload a spreadsheet.
3. Review the project totals, charts, and each package's cost variance.
4. Open **AI insights** to review the summary and unusual entries.
5. Click **Download report**, **Export results CSV**, or **Print / Save PDF**.

The navigation links move between sections of one dashboard. **How EVA works** opens a calculation guide dialog.

## Spreadsheet format

The first row must contain these headers (case-insensitive):

```csv
name,category,budget,actual,planned,complete
Planning,Management,50000,40000,100,80
Development,Engineering,150000,70000,60,50
```

| Column | Meaning | Accepted values |
| --- | --- | --- |
| `name` | Work package name | 1-80 characters |
| `category` | Work category, such as Design or Testing | 1-40 characters |
| `budget` | Full approved budget for that work package | 0 to 1,000,000,000,000 |
| `actual` | Actual spending to date | 0 to 1,000,000,000,000 |
| `planned` | Completion expected by the reporting date | Number from 0 to 100 |
| `complete` | Actual completion by the reporting date | Number from 0 to 100 |

Use a separate row for each work package and a common reporting date. Use plain numbers for costs, without currency symbols or thousands separators. Progress is entered as `50` for 50%; Excel cells with a numeric percentage format are also supported.

The first worksheet of an Excel workbook is imported. Limits are **5 MiB per file** and **1,000 work packages** in total. Invalid rows reject the entire import without partially changing project data. CSV supports quoted commas, escaped quotes, multiline fields, and UTF-8 BOM.

Try [sample-data.csv](sample-data.csv), or download the CSV template from the import dialog. Clear the existing demo rows or select **Replace existing work packages** when importing a replacement project.

## How earned value is calculated

Earned Value Analysis compares the value of completed work with planned progress and actual spending.

| Metric | Formula | Interpretation |
| --- | --- | --- |
| BAC - Budget at Completion | Sum of work package budgets | Total approved project budget |
| AC - Actual Cost | Sum of actual spending | Spending to date |
| PV - Planned Value | Sum of `budget * planned / 100` | Budgeted value of work expected to be complete |
| EV - Earned Value | Sum of `budget * complete / 100` | Budgeted value of work actually completed |
| CV - Cost Variance | `EV - AC` | Positive is favorable; negative means over cost |
| CV % | `(EV - AC) / EV * 100` | Cost variance relative to earned value |
| SV - Schedule Variance | `EV - PV` | Positive means ahead of plan; negative means behind |
| CPI - Cost Performance Index | `EV / AC` | Below 1 indicates lower cost efficiency |
| SPI - Schedule Performance Index | `EV / PV` | Below 1 indicates progress behind plan |
| EAC - Estimate at Completion | `BAC / CPI` | Forecast final cost at current cost efficiency |
| ETC - Estimate to Complete | `EAC - AC` | Forecast remaining cost |
| VAC - Variance at Completion | `BAC - EAC` | Forecast surplus or shortfall against the budget |

The downloaded report also includes simple budget variance: `BAC - AC`, with percentage calculated against BAC. This is different from earned-value cost variance because it does not account for unfinished work.

Ratios with zero denominators display a dash. EAC is unavailable when CPI is zero or undefined. Calculations preserve precision; displayed monetary amounts are rounded to whole units. Project ratios use aggregate totals rather than averages of row ratios.

The cumulative chart follows work package entry order, **not calendar time**. When there are more than eight packages, the bar chart groups packages after the seventh into **Other**. Reports retain every work package. Currency selection changes the label without converting amounts.

## How the AI feature works

Earnwise implements an **Isolation Forest**, an unsupervised machine learning algorithm that isolates unusual entries using random decision trees.

- Runs entirely in the browser, using 80 trees and up to 128 sampled rows per tree.
- Requires at least five work packages.
- Examines log actual cost, actual-to-budget ratio, normalized cost variance, and the difference between planned and actual progress.
- Flags scores above 0.60 for review; a fixed random seed makes results reproducible for the same data.

A flagged entry is a prompt to investigate, not proof of fraud or an error. The written performance summary uses templates populated with calculated metrics; it is not generated by a large language model.

## Technology and project structure

```text
earnwise/
|-- index.html                  Dashboard and dialogs
|-- styles.css                  Responsive styling
|-- core.js                     EVA calculations, CSV validation, AI model
|-- app.js                      UI, imports, local storage, charts, exports
|-- vendor/
|   |-- xlsx.full.min.js         Bundled Excel parser
|   |-- SheetJS-LICENSE.txt      Required third-party license notice
|   `-- SOURCE.txt              Dependency version and origin
|-- sample-data.csv             Optional demonstration spreadsheet
|-- vercel.json                 Static deployment configuration
|-- .vercelignore               Deployment file allowlist
|-- .gitignore                  GitHub file allowlist
`-- README.md                   Project documentation
```

Charts are rendered with SVG. Excel import uses bundled **SheetJS Community Edition 0.20.3**. Google Fonts are optional and have system-font fallbacks. The main app features work without external services after the files are available locally; a hosted page must first be loaded or downloaded before using it offline. No service worker is included.

SheetJS is provided under the Apache License 2.0; its notice is in [vendor/SheetJS-LICENSE.txt](vendor/SheetJS-LICENSE.txt). This notice applies to that dependency and does not grant a license to the entire project.

## Data storage and limitations

- Project data is processed and saved in the current browser; there is no project upload API, account system, or shared cloud database.
- Different browsers, profiles, devices, and site origins have separate data. Moving from a local file to a Vercel URL does not automatically transfer your saved project. Export the results CSV and import it at the new URL if needed.
- Clearing browser storage can remove saved work. Export important data, and avoid leaving sensitive projects or downloaded reports on shared devices.
- A person controlling the browser can change saved data. This app does not provide authoritative financial records, approval workflows, or audit trails.
- Input validation, output escaping, and CSV formula neutralization are included. These controls do not replace authentication, server-side authorization, or a deployment security review if the app is extended into a shared system.