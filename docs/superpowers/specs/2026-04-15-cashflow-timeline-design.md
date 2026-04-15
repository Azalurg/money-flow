# Cashflow Timeline App - Design Spec

Date: 2026-04-15
Status: Approved in conversation, ready for implementation planning

## 1) Goal and Scope

Build a complete, working web app for tracking and visualizing financial transactions as a timeline, delivered as a single HTML file with inline CSS and JavaScript.

In scope:
- Timeline configuration (start date, end date, initial balance)
- Transaction creation with validation
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

### 4.3 Sorting and running balance
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

### 4.4 CSV export
- Output header required: `data,kwota,typ,opis`
- Export all current transactions
- Filename format: `cashflow_RRRR-MM-DD.csv`
- Amount exported as positive numeric value (semantic sign in `typ`)

### 4.5 CSV import
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
- `script`: all JS logic and event wiring

### JS module-style sections in one script
- `state` and constants
- `utils` (date, currency formatting, parsing)
- `validation`
- `compute` (sorting, running balance, stats)
- `csv` (serialize/parse/report)
- `render`
- `events/init`

## 8) Error Handling Strategy

- Field-level errors shown inline for forms.
- Import-level errors shown as summary feedback.
- No crash on malformed CSV rows; skip safely.
- Clear messaging for empty timeline and invalid config ranges.

## 9) Responsiveness and Accessibility Baseline

- Mobile-first CSS with desktop enhancement breakpoint.
- Controls maintain usable touch targets at 375px width.
- Sufficient color contrast in dark theme.
- Inputs have labels; buttons have clear text labels.

## 10) Acceptance Criteria

- App runs as a single HTML file with inline CSS/JS and only font CDN.
- Timeline is vertical from top to bottom across device sizes.
- Desktop shows two columns with forms/stats on left and timeline on right.
- Axis displays date and running balance; bubbles show transaction details.
- `in` renders left of axis; `out` renders right of axis with minus display.
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
- Drive running balance below zero and verify red states + top warning.
- Import mixed-validity CSV and verify imported/skipped counts.
- Import fully invalid CSV and verify no state changes.
- Export CSV and confirm filename format and header.
