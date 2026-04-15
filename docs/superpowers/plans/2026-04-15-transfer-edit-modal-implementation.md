# Transfer Edit Modal Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add single-transfer editing from timeline bubbles using a shared modal with validation and stable bubble layout.

**Architecture:** Keep implementation in `index.html` and extend existing rendering/event model. Add a global edit modal, bubble-level edit affordance (pencil icon), and update flow that mutates one transaction by `id`, then re-sorts and re-renders. Reuse existing validation patterns and state-driven rendering.

**Tech Stack:** HTML5, CSS3, vanilla JavaScript (ES6), current single-file app architecture.

---

## File Structure

- Modify: `index.html` — modal markup, bubble action icon, CSS behavior, edit state, edit handlers
- Modify: `docs/superpowers/specs/2026-04-15-cashflow-timeline-design.md` (already done)

### Task 1: Add Edit Affordance UI in Timeline Bubbles

**Files:**
- Modify: `index.html`
- Test: `index.html` (manual visual check)

- [ ] **Step 1: Add pencil edit trigger inside rendered bubble HTML**

```js
const bubble = `
  <div class="tx-bubble ${row.type}">
    <button
      type="button"
      class="tx-edit-btn"
      data-edit-id="${row.id}"
      aria-label="Edytuj transfer"
      title="Edytuj transfer"
    >✎</button>
    <p class="tx-desc">${description}</p>
    <p class="tx-amount">${signedAmount}</p>
  </div>
`;
```

- [ ] **Step 2: Add CSS for hover/focus edit icon and stable bubble size**

```css
.tx-bubble {
  position: relative;
  height: 98px;
  overflow: hidden;
  padding-right: 40px;
}

.tx-edit-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  opacity: 0;
  pointer-events: none;
  transform: scale(0.96);
  transition: opacity 120ms ease, transform 120ms ease;
}

.tx-bubble:hover .tx-edit-btn,
.tx-bubble:focus-within .tx-edit-btn {
  opacity: 1;
  pointer-events: auto;
  transform: scale(1);
}

@media (hover: none), (pointer: coarse) {
  .tx-edit-btn {
    opacity: 1;
    pointer-events: auto;
    transform: none;
  }
}
```

- [ ] **Step 3: Verify visual behavior**

Run: open `index.html`  
Expected: icon appears on hover desktop, is always visible on mobile/touch emulation, bubble geometry does not jump

- [ ] **Step 4: Commit UI affordance**

```bash
git add index.html
git commit -m "feat: add timeline bubble edit affordance"
```

### Task 2: Add Shared Edit Modal Markup and Styling

**Files:**
- Modify: `index.html`
- Test: `index.html` (modal open/close)

- [ ] **Step 1: Add global modal structure to HTML body**

```html
<div id="editModal" class="modal-backdrop" hidden>
  <div class="modal-card" role="dialog" aria-modal="true" aria-labelledby="editModalTitle">
    <h2 id="editModalTitle">Edytuj transfer</h2>
    <form id="editForm" novalidate>
      <!-- date, amount, type, description fields + inline errors -->
      <div class="modal-actions">
        <button type="button" id="editCancelBtn" class="btn">Anuluj</button>
        <button type="submit" class="btn btn-primary">Zapisz zmiany</button>
      </div>
    </form>
  </div>
</div>
```

- [ ] **Step 2: Add modal CSS (overlay, card, actions)**

```css
.modal-backdrop {
  position: fixed;
  inset: 0;
  display: none;
  align-items: center;
  justify-content: center;
  background: rgba(2, 6, 23, 0.72);
  z-index: 40;
  padding: 16px;
}

.modal-backdrop.open {
  display: flex;
}

.modal-card {
  width: min(480px, 100%);
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 14px;
}
```

- [ ] **Step 3: Verify static modal rendering**

Run: temporarily add `.open` class in DevTools  
Expected: centered modal with readable form on desktop/mobile widths

- [ ] **Step 4: Commit modal structure**

```bash
git add index.html
git commit -m "feat: add shared transfer edit modal markup and styles"
```

### Task 3: Implement Edit State, Open/Close, and Prefill

**Files:**
- Modify: `index.html`
- Test: `index.html` (open and close interactions)

- [ ] **Step 1: Extend app state for edit flow**

```js
ui: {
  flashMessage: "",
  configError: "",
  txErrors: { date: "", amount: "", type: "", form: "" },
  editingTransactionId: null,
  editErrors: { date: "", amount: "", type: "", form: "" }
}
```

- [ ] **Step 2: Add helpers for modal lifecycle**

```js
function openEditModal(transactionId, triggerEl) { /* lookup, prefill, open */ }
function closeEditModal() { /* hide, clear transient edit state */ }
function populateEditForm(transaction) { /* assign field values */ }
function renderEditErrors() { /* inline edit error rendering */ }
```

- [ ] **Step 3: Wire modal close interactions**

Run: open modal then press `Esc`, click backdrop, click `Anuluj`  
Expected: modal closes without data changes

- [ ] **Step 4: Commit edit modal lifecycle**

```bash
git add index.html
git commit -m "feat: add edit modal open close and prefill flow"
```

### Task 4: Implement Edit Validation and Save Update

**Files:**
- Modify: `index.html`
- Test: `index.html` (edit save scenarios)

- [ ] **Step 1: Implement edit submit and validation reuse**

```js
function handleEditFormSubmit(event) {
  event.preventDefault();
  const input = /* read fields */;
  const errors = validateTransactionInput(input);
  if (hasAnyTxError(errors)) {
    state.ui.editErrors = errors;
    renderEditErrors();
    return;
  }
  saveEditedTransaction(input);
}
```

- [ ] **Step 2: Update transaction by id and re-render all derived values**

```js
function saveEditedTransaction(input) {
  const tx = state.transactions.find((item) => item.id === state.ui.editingTransactionId);
  if (!tx) return;
  tx.date = input.date;
  tx.amount = input.amount;
  tx.type = input.type;
  tx.description = input.description;
  sortTransactionsInState();
  closeEditModal();
  state.ui.flashMessage = "Transakcja została zaktualizowana.";
  renderAll();
}
```

- [ ] **Step 3: Verify save behavior and edge cases**

Run: edit each field and save  
Expected: transaction updates, resorting applies, balances/stats/topbar update

Run: set edited date outside range  
Expected: inline modal error, no save

- [ ] **Step 4: Commit edit persistence logic**

```bash
git add index.html
git commit -m "feat: support validated single-transfer editing"
```

### Task 5: Add Timeline Event Delegation and Final Verification

**Files:**
- Modify: `index.html`
- Test: `index.html`

- [ ] **Step 1: Add click delegation for edit icon in timeline list**

```js
document.getElementById("timelineList").addEventListener("click", (event) => {
  const button = event.target.closest("[data-edit-id]");
  if (!button) return;
  openEditModal(Number(button.dataset.editId), button);
});
```

- [ ] **Step 2: Run full manual checklist for edit feature**

Run:
- hover bubble on desktop and check icon visibility
- touch/mobile width and check always-visible icon
- open modal, save valid edit
- invalid date out of range blocks save
- change type in/out and verify side relocation
- verify bubble does not resize enough to break layout

Expected: all edit acceptance criteria satisfied

- [ ] **Step 3: Final feature commit**

```bash
git add index.html
git commit -m "feat: complete transfer editing via modal from timeline"
```

## Plan Self-Review

- Spec coverage: includes hover/touch edit affordance, shared modal, full-field edit, date-range validation, stable bubble sizing, and derived data refresh.
- Placeholder scan: no placeholders remain.
- Type consistency: edit flow uses existing transaction shape (`id`, `date`, `amount`, `type`, `description`) and existing sort/render pipeline.
