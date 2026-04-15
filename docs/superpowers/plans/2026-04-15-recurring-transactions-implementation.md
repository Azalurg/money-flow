# Recurring Transactions Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add recurring transaction creation with interval options and auto-numbered titles in the existing single-file cashflow app.

**Architecture:** Extend the existing add-transaction form with a mode toggle and recurrence-only fields. Generate recurring occurrences in batch at submit time, convert each generated occurrence into a regular transaction object, and keep existing sorting/render/statistics flow unchanged. Reuse existing validation patterns and add recurrence-specific validation/messages.

**Tech Stack:** HTML5, CSS3, vanilla JavaScript (ES6), existing single-file app architecture (`index.html`).

---

## File Structure

- Modify: `index.html` — form UI, recurrence styles, recurrence generation logic, handlers
- Modify: `docs/superpowers/specs/2026-04-15-cashflow-timeline-design.md` (already done)

### Task 1: Add Recurrence Mode UI to Add-Transaction Form

**Files:**
- Modify: `index.html`
- Test: `index.html` (manual UI toggling)

- [ ] **Step 1: Add transaction mode control in form markup**

```html
<div class="form-row">
  <label>Tryb dodawania</label>
  <div class="type-row">
    <label class="type-option" for="txModeSingle">
      <input id="txModeSingle" type="radio" name="txMode" value="single" checked>
      Pojedyncza
    </label>
    <label class="type-option" for="txModeRecurring">
      <input id="txModeRecurring" type="radio" name="txMode" value="recurring">
      Cykliczna
    </label>
  </div>
</div>
```

- [ ] **Step 2: Add recurrence-only fields container**

```html
<div id="recurrenceFields" class="recurrence-fields" hidden>
  <div class="form-row">
    <label for="recurrenceType">Interwał</label>
    <select id="recurrenceType" name="recurrenceType">
      <option value="weekly">Co tydzień</option>
      <option value="monthly">Co miesiąc</option>
      <option value="every-n-days">Co N dni</option>
    </select>
  </div>
  <div class="form-row" id="recurrenceEveryRow" hidden>
    <label for="recurrenceEvery">N (dni)</label>
    <input id="recurrenceEvery" name="recurrenceEvery" type="number" min="1" step="1">
  </div>
  <div class="form-row">
    <label for="recurrenceCount">Liczba powtórzeń (X)</label>
    <input id="recurrenceCount" name="recurrenceCount" type="number" min="1" step="1">
    <div id="recurrenceError" class="field-error" aria-live="polite"></div>
  </div>
</div>
```

- [ ] **Step 3: Add minimal CSS visibility and spacing helpers**

```css
.recurrence-fields {
  display: grid;
  gap: 8px;
  margin-top: 4px;
}

.recurrence-fields[hidden] {
  display: none;
}
```

- [ ] **Step 4: Verify UI mode switching manually**

Run: open `index.html`, switch single/recurring  
Expected: recurrence fields appear only in recurring mode

- [ ] **Step 5: Commit form UI**

```bash
git add index.html
git commit -m "feat: add recurring mode fields to transaction form"
```

### Task 2: Implement Recurrence Validation and Date Generators

**Files:**
- Modify: `index.html`
- Test: `index.html` (manual console checks + form validation behavior)

- [ ] **Step 1: Add helper functions for recurrence date math**

```js
function addDaysUtc(isoDate, days) { /* returns YYYY-MM-DD */ }
function addMonthsSameDayWithFallback(isoDate, monthsToAdd) { /* same day or month end */ }
```

- [ ] **Step 2: Add recurrence validation function**

```js
function validateRecurrenceInput(input) {
  const errors = { recurrence: "" };
  if (!Number.isInteger(input.recurrenceCount) || input.recurrenceCount <= 0) {
    errors.recurrence = "Liczba powtórzeń musi być większa od 0.";
  }
  if (input.recurrenceType === "every-n-days" && (!Number.isInteger(input.recurrenceEvery) || input.recurrenceEvery <= 0)) {
    errors.recurrence = "Dla interwału co N dni podaj N większe od 0.";
  }
  return errors;
}
```

- [ ] **Step 3: Add recurrence occurrence generator**

```js
function generateRecurringDates(startDate, recurrenceType, recurrenceEvery, recurrenceCount) {
  const dates = [];
  for (let i = 0; i < recurrenceCount; i += 1) {
    if (recurrenceType === "weekly") dates.push(addDaysUtc(startDate, i * 7));
    else if (recurrenceType === "monthly") dates.push(addMonthsSameDayWithFallback(startDate, i));
    else dates.push(addDaysUtc(startDate, i * recurrenceEvery));
  }
  return dates;
}
```

- [ ] **Step 4: Verify helper behavior quickly in browser console**

Run in console:

```js
addMonthsSameDayWithFallback("2026-01-31", 1)
```

Expected: valid month-end fallback date in February

- [ ] **Step 5: Commit recurrence core helpers**

```bash
git add index.html
git commit -m "feat: add recurring date generation and validation helpers"
```

### Task 3: Integrate Recurring Submit Flow into Add Transaction Handler

**Files:**
- Modify: `index.html`
- Test: `index.html` (single and recurring submit behavior)

- [ ] **Step 1: Add UI state fields for recurrence errors and mode**

```js
ui: {
  ...,
  recurrenceError: ""
}
```

- [ ] **Step 2: Add function to create recurring transactions batch**

```js
function addRecurringTransactions(input) {
  // validate recurrence fields
  // generate dates
  // filter by timeline range
  // create transactions with description `${baseTitle} ${i}/${X}`
  // push added transactions, sort, set flash message with added/skipped
}
```

- [ ] **Step 3: Branch submit logic by mode in `handleTransactionSubmit`**

```js
const mode = String(formData.get("txMode") || "single");
if (mode === "recurring") {
  const ok = addRecurringTransactions(input);
  if (ok) event.currentTarget.reset();
} else {
  const ok = addTransaction(input);
  if (ok) event.currentTarget.reset();
}
```

- [ ] **Step 4: Ensure recurring fields are ignored in single mode**

Run: submit single transaction with recurrence fields present in DOM  
Expected: one standard transaction added, no recurrence behavior triggered

- [ ] **Step 5: Commit submit integration**

```bash
git add index.html
git commit -m "feat: integrate recurring transaction batch creation on submit"
```

### Task 4: Render/Events Polish for Recurrence Controls and Errors

**Files:**
- Modify: `index.html`
- Test: `index.html` (visual/error behavior)

- [ ] **Step 1: Add render function for recurrence fields visibility**

```js
function renderRecurrenceControls() {
  const recurring = document.getElementById("txModeRecurring").checked;
  const recurrenceFields = document.getElementById("recurrenceFields");
  recurrenceFields.hidden = !recurring;

  const everyNDays = recurring && document.getElementById("recurrenceType").value === "every-n-days";
  document.getElementById("recurrenceEveryRow").hidden = !everyNDays;
  document.getElementById("recurrenceError").textContent = state.ui.recurrenceError;
}
```

- [ ] **Step 2: Add event listeners for recurrence controls**

```js
document.getElementById("txModeSingle").addEventListener("change", renderAll);
document.getElementById("txModeRecurring").addEventListener("change", renderAll);
document.getElementById("recurrenceType").addEventListener("change", renderAll);
```

- [ ] **Step 3: Hook recurrence rendering into `renderAll()`**

Run: toggle modes and recurrence types  
Expected: correct fields and errors shown/hidden at all times

- [ ] **Step 4: Commit recurrence UX polish**

```bash
git add index.html
git commit -m "feat: add recurrence control rendering and inline errors"
```

### Task 5: Manual Verification Pass and Final Commit

**Files:**
- Modify: `index.html`
- Test: `index.html`

- [ ] **Step 1: Run full recurring feature manual checklist**

Run manually:
- weekly recurrence with X=4
- every-n-days with N=10, X=6
- monthly recurrence from day 31 (month-end fallback)
- partial out-of-range summary
- full out-of-range rejection
- verify generated entries support edit/delete/export

Expected: all recurring acceptance criteria pass

- [ ] **Step 2: Final cleanup and consistency pass**

Check:
- no stale recurrence errors after successful submit
- flash messages are user-friendly and specific
- sorting and stats remain correct

- [ ] **Step 3: Final feature commit**

```bash
git add index.html
git commit -m "feat: support recurring transactions with interval generation"
```

## Plan Self-Review

- Spec coverage: recurrence UI mode, interval generation, title numbering, out-of-range skip reporting, and compatibility with existing edit/delete/CSV flow are covered.
- Placeholder scan: no placeholders remain.
- Type consistency: recurring records use existing transaction schema and existing sort/render pipeline.
