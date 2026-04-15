# Cashflow Timeline Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file HTML cashflow tracker with a vertical timeline, CSV import/export, live balance, and statistics.

**Architecture:** Keep all product code in `index.html` (inline CSS + inline JS) with JS organized by sections: state, validation, compute, CSV, render, events. Use a single `renderAll()` entry point after any state change. Treat timeline rows as derived data from sorted transactions plus running balance.

**Tech Stack:** HTML5, CSS3, vanilla JavaScript (ES6), Inter font (Google Fonts CDN), browser File API + Blob API.

---

## File Structure

- Create: `index.html` — complete app (UI + styles + logic) in one file
- Modify: `docs/superpowers/specs/2026-04-15-cashflow-timeline-design.md` (only if implementation uncovers contradictions)
- Create: `docs/superpowers/plans/2026-04-15-cashflow-timeline-implementation.md` (this plan)

No runtime JS/CSS files outside `index.html`.

### Task 1: Scaffold Single-File App Shell

**Files:**
- Create: `index.html`
- Test: `index.html` (open in browser)

- [ ] **Step 1: Write static shell first (no business logic yet)**

```html
<!doctype html>
<html lang="pl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Cashflow Timeline</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    :root { --bg:#0f0f0f; --surface:#1f2937; --accent:#10b981; --ok:#4ade80; --bad:#f87171; --text:#e5e7eb; --muted:#9ca3af; }
  </style>
</head>
<body>
  <header class="topbar"></header>
  <main class="app-grid"></main>
  <script>
    // app bootstrap in next tasks
  </script>
</body>
</html>
```

- [ ] **Step 2: Add desktop/mobile layout and semantic containers**

```html
<main class="app-grid">
  <section class="left-panel">
    <article class="card" id="config-card"></article>
    <article class="card" id="transaction-card"></article>
    <article class="card" id="stats-card"></article>
  </section>
  <section class="timeline-panel">
    <div id="timeline-empty" class="empty-state"></div>
    <div id="timeline-list" class="timeline-list"></div>
  </section>
</main>
```

- [ ] **Step 3: Verify shell renders correctly**

Run: open `index.html` in browser  
Expected: top bar + 2-column desktop layout and stacked mobile layout

- [ ] **Step 4: Commit scaffold**

```bash
git add index.html
git commit -m "feat: scaffold single-file cashflow app layout"
```

### Task 2: Add Core State, Sorting, Running Balance, and Stats

**Files:**
- Modify: `index.html`
- Test: `index.html` (manual console checks)

- [ ] **Step 1: Add state model and pure compute helpers**

```js
const state = {
  timelineConfig: { startDate: "", endDate: "", initialBalance: 0 },
  transactions: [],
  nextId: 1,
  ui: { flashMessage: "", inlineErrors: {} }
};

function compareTransactions(a, b) {
  if (a.date !== b.date) return a.date < b.date ? -1 : 1;
  if (a.type !== b.type) return a.type === "in" ? -1 : 1; // in before out
  return a._seq - b._seq;
}

function buildTimelineRows(transactions, initialBalance) {
  const sorted = [...transactions].sort(compareTransactions);
  let running = Number(initialBalance) || 0;
  return sorted.map((tx) => {
    running += tx.type === "in" ? tx.amount : -tx.amount;
    return { ...tx, runningBalance: running };
  });
}
```

- [ ] **Step 2: Add statistics computation helper**

```js
function computeStats(rows) {
  let sumIn = 0;
  let sumOut = 0;
  let negativePoints = 0;
  let negativeEntries = 0;
  let prev = null;

  for (const row of rows) {
    if (row.type === "in") sumIn += row.amount;
    else sumOut += row.amount;

    if (row.runningBalance < 0) {
      negativePoints += 1;
      if (prev === null || prev >= 0) negativeEntries += 1;
    }
    prev = row.runningBalance;
  }

  return {
    operations: rows.length,
    sumIn,
    sumOut,
    netFlow: sumIn - sumOut,
    negativePoints,
    negativeEntries,
    hadNegative: negativePoints > 0
  };
}
```

- [ ] **Step 3: Manual failing/passing check in DevTools console**

Run in browser console:

```js
const demo = [
  { id:1, date:"2026-04-01", amount:100, type:"out", _seq:1 },
  { id:2, date:"2026-04-01", amount:50, type:"in", _seq:2 }
];
buildTimelineRows(demo, 0).map(r => r.type)
```

Expected: `['in','out']`

- [ ] **Step 4: Commit core compute layer**

```bash
git add index.html
git commit -m "feat: add state model with sorting and stats computation"
```

### Task 3: Implement Forms and Validation Rules

**Files:**
- Modify: `index.html`
- Test: `index.html` (form submission scenarios)

- [ ] **Step 1: Build timeline configuration form and handlers**

```js
function validateConfig(startDate, endDate) {
  if (!startDate || !endDate) return "Uzupełnij zakres dat.";
  if (startDate > endDate) return "Data startowa nie może być po końcowej.";
  return "";
}

function applyConfig(formData) {
  const startDate = formData.get("startDate");
  const endDate = formData.get("endDate");
  const initialBalance = Number(formData.get("initialBalance") || 0);
  const error = validateConfig(startDate, endDate);
  if (error) return { ok: false, error };

  state.timelineConfig = { startDate, endDate, initialBalance };
  state.transactions = [];
  state.nextId = 1;
  return { ok: true };
}
```

- [ ] **Step 2: Build transaction form validation and add logic**

```js
function validateTransaction(input) {
  const errors = {};
  if (!input.date) errors.date = "Wybierz datę.";
  if (input.date && (input.date < state.timelineConfig.startDate || input.date > state.timelineConfig.endDate)) {
    errors.date = "Data poza zakresem osi czasu.";
  }
  if (!(input.amount > 0)) errors.amount = "Kwota musi być większa od 0.";
  if (!input.type) errors.type = "Wybierz typ operacji.";
  return errors;
}

function addTransaction(input) {
  const errors = validateTransaction(input);
  if (Object.keys(errors).length) {
    state.ui.inlineErrors = errors;
    return false;
  }
  state.transactions.push({
    id: state.nextId++,
    date: input.date,
    amount: input.amount,
    type: input.type,
    description: (input.description || "").trim(),
    _seq: Date.now() + Math.random()
  });
  state.ui.inlineErrors = {};
  return true;
}
```

- [ ] **Step 3: Verify validation gates manually**

Run: try add transaction with date outside range and amount `0`  
Expected: inline errors shown, transaction not added

- [ ] **Step 4: Commit forms + validation**

```bash
git add index.html
git commit -m "feat: add timeline and transaction forms with validation"
```

### Task 4: Render Vertical Timeline, Top Bar Signals, and Empty State

**Files:**
- Modify: `index.html`
- Test: `index.html` (visual checks)

- [ ] **Step 1: Add render helpers and unified `renderAll()`**

```js
function formatPln(value) {
  return `${value.toFixed(2)} PLN`;
}

function renderAll() {
  const rows = buildTimelineRows(state.transactions, state.timelineConfig.initialBalance);
  const stats = computeStats(rows);
  renderTopbar(rows, stats);
  renderStats(stats);
  renderTimeline(rows);
  renderFormConstraints();
}
```

- [ ] **Step 2: Render timeline rows with axis date+balance and side bubbles**

```js
function renderTimeline(rows) {
  const list = document.getElementById("timeline-list");
  const empty = document.getElementById("timeline-empty");
  if (!rows.length) {
    list.innerHTML = "";
    empty.textContent = "Brak transakcji. Dodaj operację lub zaimportuj CSV.";
    return;
  }
  empty.textContent = "";
  list.innerHTML = rows.map((row) => {
    const sideClass = row.type === "in" ? "left" : "right";
    const signedAmount = row.type === "in" ? `+${row.amount.toFixed(2)} PLN` : `-${row.amount.toFixed(2)} PLN`;
    const desc = row.description || "Brak opisu";
    const negClass = row.runningBalance < 0 ? "negative" : "";
    return `<article class="timeline-row ${sideClass}">
      <div class="bubble ${row.type}"><p>${desc}</p><strong>${signedAmount}</strong></div>
      <div class="axis-node"><time>${row.date}</time><span class="${negClass}">${formatPln(row.runningBalance)}</span></div>
    </article>`;
  }).join("");
}
```

- [ ] **Step 3: Verify visual constraints**

Run: open `index.html` and add mixed transactions  
Expected: vertical axis top-to-bottom; `in` bubbles left; `out` right; negative balances in red

- [ ] **Step 4: Commit timeline rendering**

```bash
git add index.html
git commit -m "feat: render vertical timeline with side bubbles and running balance"
```

### Task 5: Implement CSV Import/Export and Result Reporting

**Files:**
- Modify: `index.html`
- Test: `index.html` (import/export scenarios)

- [ ] **Step 1: Add CSV export serializer and download flow**

```js
function serializeCsv(transactions) {
  const header = "data,kwota,typ,opis";
  const rows = transactions.map((tx) => {
    const safeDesc = (tx.description || "").replace(/"/g, '""');
    return `${tx.date},${tx.amount.toFixed(2)},${tx.type},"${safeDesc}"`;
  });
  return [header, ...rows].join("\n");
}

function exportCsv() {
  const content = serializeCsv(state.transactions);
  const blob = new Blob([content], { type: "text/csv;charset=utf-8" });
  const a = document.createElement("a");
  const today = new Date().toISOString().slice(0, 10);
  a.href = URL.createObjectURL(blob);
  a.download = `cashflow_${today}.csv`;
  a.click();
  URL.revokeObjectURL(a.href);
}
```

- [ ] **Step 2: Add CSV import parser with row-level validation and auto-range**

```js
function parseCsv(text) {
  const lines = text.split(/\r?\n/).filter(Boolean);
  if (!lines.length || lines[0].trim() !== "data,kwota,typ,opis") {
    return { ok: false, imported: [], errors: 1, headerError: true };
  }

  const imported = [];
  let errors = 0;
  for (let i = 1; i < lines.length; i += 1) {
    const [date, amountRaw, type, ...descParts] = lines[i].split(",");
    const amount = Number(amountRaw);
    const description = descParts.join(",").replace(/^"|"$/g, "").replace(/""/g, '"');
    if (!date || Number.isNaN(amount) || !(amount > 0) || !["in", "out"].includes(type)) {
      errors += 1;
      continue;
    }
    imported.push({ id: i, date, amount, type, description, _seq: i });
  }

  return { ok: imported.length > 0, imported, errors, headerError: false };
}
```

- [ ] **Step 3: Wire import flow and success/failure messaging**

Run: import mixed CSV  
Expected: valid rows loaded, invalid rows skipped, message shows imported/skipped counts

Run: import file with no valid row  
Expected: error alert/message; state remains unchanged

- [ ] **Step 4: Commit CSV features**

```bash
git add index.html
git commit -m "feat: add csv import export with validation and reporting"
```

### Task 6: Final Polish, Verification, and Delivery Commit

**Files:**
- Modify: `index.html`
- Test: `index.html`

- [ ] **Step 1: Add negative-history badge in top bar and ensure stats card is complete**

```js
function renderTopbar(rows, stats) {
  const current = rows.length ? rows[rows.length - 1].runningBalance : state.timelineConfig.initialBalance;
  const balanceEl = document.getElementById("current-balance");
  balanceEl.textContent = formatPln(current);
  balanceEl.classList.toggle("negative", current < 0);

  const warningEl = document.getElementById("negative-warning");
  warningEl.hidden = !stats.hadNegative;
  warningEl.textContent = "Uwaga: saldo było ujemne";
}
```

- [ ] **Step 2: Run full manual acceptance checklist from spec**

Run:
- open `index.html` on desktop and mobile width
- test form validation rules
- test same-day in/out ordering
- test minus display for `out`
- test negative history badge
- test import mixed and invalid CSV
- test export filename format

Expected: all acceptance criteria from spec section 10 pass

- [ ] **Step 3: Final commit**

```bash
git add index.html
git commit -m "feat: complete single-file cashflow timeline app"
```

## Plan Self-Review

- Spec coverage: each requirement maps to Tasks 1-6 (layout/theme/forms/timeline/csv/stats/negative states/responsive/empty state).
- Placeholder scan: no TODO/TBD placeholders remain.
- Type consistency: uses one transaction shape (`{ id, date, amount, type, description, _seq }`) and shared compute helpers across tasks.
