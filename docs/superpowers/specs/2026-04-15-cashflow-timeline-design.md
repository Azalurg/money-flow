# Cashflow Timeline App - Design Spec

Date: 2026-04-15
Status: Approved in conversation, ready for implementation planning

## 1) Goal and Scope

Build a complete, working web app for tracking and visualizing financial transactions as a timeline, delivered as a single HTML file with inline CSS and JavaScript.

In scope:
- Timeline configuration (start date, end date, initial balance)
- Transaction creation with validation
- Transaction editing (single transfer)
- Recurring transaction generation
- Chronological timeline rendering with running balance
- CSV import and export
- Live balance and statistics panel
- Responsive layout (mobile and desktop)

Out of scope:
- Any persistence mechanism (`localStorage`, backend, database)
- Multi-currency support
- Authentication/user accounts

## 2) Product Decisions Captured

- Timeline layout is always vertical.
- Desktop layout is two columns:
  - Left: forms + statistics panel
  - Right: timeline
- Left column stays visible (sticky) and can scroll independently if needed.
- On timeline axis: show date and running balance.
- Transaction details are shown in bubbles next to the axis.
- `in` transactions appear on the left side of the axis.
- `out` transactions appear on the right side of the axis.
- For `out` bubbles, display amount with explicit minus sign for clarity.
- If running balance at a point is negative, balance text is red.
- If any running balance is negative at least once, top bar shows warning info.
- CSV amount format uses decimal dot.
- Same-day ordering rule: `in` before `out`.
- Single transfer editing uses one shared modal.
- Edit action appears as a pencil icon in transaction bubble:
  - desktop: visible on hover/focus
  - mobile: always visible
- Editing allows changing all fields: date, amount, type, description.
- Edit save is blocked when edited date is outside timeline range.
- Transaction bubble size remains visually stable during hover and edit affordance display.
- Recurring transactions are generated in batch immediately (no persistent recurrence rule model).
- Recurrence supports:
  - every week
  - every N days
  - every month (same day-of-month with end-of-month fallback)
- Recurrence series length is defined by explicit repeat count (`X`).
- Series descriptions follow format `<Tytuł> i/X`.
- Occurrences outside timeline range are skipped and reported (`added/skipped`).

## 3) UX and Visual Design

### Theme and style
- Dark UI:
  - page background around `#0f0f0f` / `#111827`
  - surfaces around `#1c1c1e` / `#1f2937`
- Accent color: teal/emerald (`#10b981` family)
- Typography: Inter (Google Fonts CDN)
- Compact, clean controls; no gradient-heavy buttons; no decorative icon circles

### Top bar
- App name
- Current account balance (prominent)
- Negative-history badge if any negative running balance occurred
- CSV actions: Import and Export

### Main content
- Desktop: two-column layout
  - Left panel: configuration form, transaction form, statistics card
  - Right panel: vertical timeline from top to bottom
- Mobile (375px+): single-column stacked layout (forms first, timeline after)

### Timeline item composition
- Axis core shows:
  - date
  - running balance after operation
- Adjacent bubble shows:
  - transaction type context (visual color)
  - description (or fallback text)
  - signed display amount (`+` for in, `-` for out)
- Color coding:
  - `in`: green (`#4ade80`-like)
  - `out`: red (`#f87171`-like)
  - negative balance: red emphasis
- Bubble actions:
  - show pencil edit icon per transaction
  - icon appears without affecting bubble layout flow

### Edit modal
- One global modal shared by all transactions.
- Fields: date, amount, type, description.
- Actions: cancel and save.
- Validation errors shown inline in modal.
- Closing without save keeps original transaction data.

### Recurrence input mode
- Transaction form has two modes:
  - single transaction
  - recurring transaction
- In recurring mode, additional fields appear:
  - recurrence type (`weekly`, `monthly-same-day`, `every-n-days`)
  - N value (for `every-n-days`)
  - repeat count `X`
- Existing date/amount/type/description fields remain base inputs.

### Empty state
- Friendly message with guidance to add a transaction or import CSV.

## 4) Functional Requirements

### 4.1 Timeline configuration
Fields:
- Start date
- End date
- Initial balance (PLN, default `0`)

Action:
- `Apply` initializes/resets timeline state for given range and balance.

Validation:
- start and end required
- start must be <= end

### 4.2 Add transaction
Fields:
- Date (date picker, constrained to timeline range)
- Amount (number > 0, PLN)
- Type (`in`/`out`, required)
- Description (optional)

Action:
- `Add transaction` validates and inserts transaction into state.

Validation behavior:
- date must be within timeline range
- amount must be > 0
- type is required
- inline validation errors for invalid fields
- if date outside range, submission blocked and inline error shown

### 4.3 Edit transaction (single transfer)
- Trigger:
  - Click pencil icon on a transaction bubble.
- Behavior:
  - open modal prefilled with selected transaction data
  - allow editing date, amount, type, description
  - save updates existing transaction by `id`
  - cancel closes modal without changes
- Validation:
  - same rules as add transaction
  - edited date must be in configured timeline range
  - invalid input blocks save and shows inline modal errors
- Effects after successful save:
  - sort transactions by existing rules
  - recalculate running balances
  - refresh statistics and top-bar negative-history indicator
  - move bubble side if type changes (`in` left, `out` right)
  - keep bubble size constraints stable (no UI jump)

### 4.4 Add recurring transactions (batch generation)
- Trigger:
  - user selects recurring mode in add transaction form and submits.
- Inputs:
  - base fields: start date, amount, type, base description
  - recurrence fields: interval type, repeat count `X`, optional interval value `N`
- Generation behavior:
  - generate up to `X` occurrences from start date
  - for each occurrence, create standard transaction object
  - description format: `<baseTitle> i/X` (e.g., `Czynsz 3/12`)
- Interval rules:
  - weekly: +7 days each step
  - every-n-days: +N days each step (N > 0)
  - monthly-same-day:
    - keep same day-of-month where possible
    - if month is shorter, use last day of target month
- Range handling:
  - occurrences outside timeline range are skipped
  - in-range occurrences are added
  - user receives summary message (e.g., `Dodano 8/12, pominięto 4`).
- Failure behavior:
  - if no occurrences are in range, add nothing and show warning/error message.

### 4.5 Sorting and running balance
- Data model:
  - `transactions: Array<{ id, date, amount, type, description }>`
- Sorting order:
  - date ascending
  - within same date: all `in` before all `out`
  - stable order inside same date and same type (insertion/import order)
- Running balance:
  - starts at `initialBalance`
  - each `in` adds amount
  - each `out` subtracts amount
  - each rendered point stores computed running balance value

### 4.6 CSV export
- Output header required: `data,kwota,typ,opis`
- Export all current transactions
- Filename format: `cashflow_RRRR-MM-DD.csv`
- Amount exported as positive numeric value (semantic sign in `typ`)

### 4.7 CSV import
- Input must include header: `data,kwota,typ,opis`
- Each row validation:
  - valid date
  - amount numeric and > 0 (decimal dot format)
  - type in `{in, out}`
- Invalid rows are skipped and counted as errors
- If at least one valid row imported:
  - set `startDate = min(date)`
  - set `endDate = max(date)`
  - set `initialBalance = 0`
  - replace current transactions with imported valid rows
  - show message with counts: imported and skipped
- If zero valid rows:
  - show alert/error
  - do not modify current app state

## 5) Statistics Panel

Statistics card is always visible in left panel and updates after every state change.

Metrics:
- Number of operations
- Sum of deposits (`in`)
- Sum of withdrawals (`out`)
- Net money flow (`sum(in) - sum(out)`)
- Count of timeline points where running balance is negative
- Count of transitions into negative zone (`>=0` to `<0`)

## 6) State and Data Flow

In-memory state only:

```js
state = {
  timelineConfig: {
    startDate: null,
    endDate: null,
    initialBalance: 0,
  },
  transactions: [],
  nextId: 1,
  ui: {
    inlineErrors: {},
    editErrors: {},
    editingTransactionId: null,
    recurrenceErrors: {},
    flashMessage: "",
  },
};
```

Derived data pipeline on every update:
1. Read `transactions`
2. Sort by business rules
3. Compute running balances
4. Compute aggregate stats
5. Render top bar, forms constraints, stats card, timeline, empty state

Single render orchestrator:
- `renderAll()` is the only full refresh entry point after user actions.

## 7) Technical Structure (Single HTML)

### HTML blocks
- `head`: meta viewport + Inter font link + title
- `style`: all CSS tokens, layout, forms, cards, timeline styles
- `body`:
  - top bar
  - desktop/mobile main layout
  - left panel cards (config, add transaction, stats)
  - right panel timeline container
  - hidden file input for CSV import
  - edit modal markup
- `script`: all JS logic and event wiring

### JS module-style sections in one script
- `state` and constants
- `utils` (date, currency formatting, parsing)
- `validation`
- `compute` (sorting, running balance, stats)
- `csv` (serialize/parse/report)
- `render`
- `edit-modal` (open/close, prefill, save, validation)
- `recurrence` (interval date generation, batch add, recurrence validation)
- `events/init`

## 8) Error Handling Strategy

- Field-level errors shown inline for forms.
- Field-level errors shown inline for edit modal.
- Field-level errors shown inline for recurring fields in add form.
- Import-level errors shown as summary feedback.
- No crash on malformed CSV rows; skip safely.
- Clear messaging for empty timeline and invalid config ranges.

## 9) Responsiveness and Accessibility Baseline

- Mobile-first CSS with desktop enhancement breakpoint.
- Controls maintain usable touch targets at 375px width.
- Sufficient color contrast in dark theme.
- Inputs have labels; buttons have clear text labels.
- Edit icon remains keyboard reachable/focusable.
- Modal supports Escape and backdrop close patterns.

## 10) Acceptance Criteria

- App runs as a single HTML file with inline CSS/JS and only font CDN.
- Timeline is vertical from top to bottom across device sizes.
- Desktop shows two columns with forms/stats on left and timeline on right.
- Axis displays date and running balance; bubbles show transaction details.
- `in` renders left of axis; `out` renders right of axis with minus display.
- Single transfer can be edited from timeline bubble via modal.
- Edit validation blocks invalid save (including out-of-range date).
- Bubble size remains stable while showing edit affordance.
- Recurring mode can generate weekly, monthly, and every-N-days series.
- Recurring descriptions follow `<Tytuł> i/X` numbering.
- Out-of-range occurrences are skipped with `added/skipped` feedback.
- Negative balance values are red; top bar warns if negative occurred historically.
- Form validation works and prevents invalid transaction submission.
- CSV import/export works with required schema and reports skipped rows.
- Import with no valid rows does not alter existing state.
- Statistics panel updates correctly after every mutation.

## 11) Manual Verification Checklist

- Configure valid range and initial balance.
- Attempt invalid range and confirm validation.
- Add `in` and `out` transactions within range.
- Attempt date outside range and confirm inline error.
- Add same-day `in` and `out` and verify ordering (`in` then `out`).
- Confirm side placement (`in` left, `out` right).
- Confirm `out` amount displays minus sign.
- Open edit modal from bubble icon and confirm prefilled values.
- Edit each field and save; confirm timeline and stats update.
- Set edited date outside range and confirm inline modal error with blocked save.
- Change edited type (`in`/`out`) and confirm bubble side changes.
- Verify bubble size does not jump/collide when icon appears on hover.
- Add weekly recurring series and verify numbering `1/X..X/X`.
- Add every-N-days recurring series and verify date spacing.
- Add monthly recurring series from a long month day and verify end-of-month fallback.
- Verify partial out-of-range generation reports added/skipped counts.
- Verify full out-of-range generation adds nothing and shows warning.
- Verify generated records support edit/delete/CSV as standard transactions.
- Drive running balance below zero and verify red states + top warning.
- Import mixed-validity CSV and verify imported/skipped counts.
- Import fully invalid CSV and verify no state changes.
- Export CSV and confirm filename format and header.
