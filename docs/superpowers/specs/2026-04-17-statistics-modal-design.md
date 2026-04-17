# Cashflow Timeline - Statistics Modal and Extended Metrics Design

Date: 2026-04-17
Status: Draft approved in conversation, ready for implementation plan after spec review

## 1) Goal

Improve readability of statistics as the metric set grows by introducing a dedicated statistics modal, while keeping a compact snapshot in the existing left panel.

## 2) Scope

In scope:
- Keep exactly 4 quick-stat tiles in the existing left-panel statistics card.
- Add a statistics trigger icon/button in the top bar.
- Add a second statistics trigger in the statistics panel.
- Open one shared modal with full statistics.
- Replace existing negative-balance counters with more meaningful metrics.
- Add deposit/withdrawal averages and average interval metrics.

Out of scope:
- Backend or persistence changes.
- Changes to transaction/timeline core behavior unrelated to stats.
- CSV schema changes.

## 3) Product Decisions Captured

- Main panel keeps only 4 tiles:
  - `Ilość operacji`
  - `Bilans netto`
  - `Czas na minusie (dni)`
  - `Liczba okresów ujemnych`
- Full statistics move to modal grouped into 3 sections:
  - `Wolumen`
  - `Średnie i interwały`
  - `Saldo i ryzyko`
- Existing metrics removed:
  - `Punkty na minusie`
  - `Wejścia na minus`
- Keep top-bar warning badge for negative history.
- `Money flow` label becomes Polish: `Bilans netto`.

## 4) UX and Layout Design

### 4.1 Top bar trigger
- Add a `Statystyki` action in the top actions area.
- Desktop: icon + text label.
- Mobile: compact icon-first presentation while preserving accessible text/aria-label.

### 4.2 Panel trigger
- In statistics card add a secondary trigger (button/link-style control) opening the same modal.
- Both triggers call one open handler.

### 4.3 Statistics modal
- Shared modal component (same interaction model as current edit modal).
- Desktop: centered modal card with scrollable body.
- Mobile: fullscreen-style modal for easier reading.
- Sections and ordering:
  1. `Wolumen`
  2. `Średnie i interwały`
  3. `Saldo i ryzyko`
- Close interactions:
  - `Esc`
  - close button
  - backdrop click
- Accessibility/focus:
  - move focus into modal on open
  - trap focus while open
  - restore focus to opening trigger on close

## 5) Statistics Definitions

## 5.1 Wolumen
- `Ilość operacji` = all transactions count.
- `Liczba wpłat` = count where `type === "in"`.
- `Liczba wypłat` = count where `type === "out"`.
- `Suma wpłat` = sum of amounts for `in`.
- `Suma wypłat` = sum of amounts for `out`.

## 5.2 Średnie i interwały
- `Średnia wpłata` = `Suma wpłat / Liczba wpłat`.
- `Średnia wypłata` = `Suma wypłat / Liczba wypłat`.
- `Średni interwał wpłat (dni)`:
  - sort all `in` transaction dates ascending,
  - compute day differences for each consecutive pair,
  - average the differences,
  - same date is `0` days.
- `Średni interwał wypłat (dni)` is analogous for `out`.
- If there are fewer than 2 transactions of a given type, interval metric displays `-` (no data).

## 5.3 Saldo i ryzyko
- `Bilans netto` = `Suma wpłat - Suma wypłat`.
- `Czas na minusie (dni)`:
  - evaluate every calendar day in configured axis range (`startDate..endDate`, inclusive),
  - for each day use balance after all transactions on that day,
  - for days without transactions carry over previous day balance,
  - count days where daily balance `< 0`.
- `Liczba okresów ujemnych`:
  - count transitions from daily status `>= 0` to `< 0` across the axis-day sequence,
  - if initial balance on start day is `< 0`, first period starts immediately and is counted.
- Negative-history top warning remains active when `Czas na minusie (dni) > 0`.

## 6) UI Data Distribution

### 6.1 Left-panel quick stats (4 tiles only)
- `Ilość operacji`
- `Bilans netto`
- `Czas na minusie (dni)`
- `Liczba okresów ujemnych`

### 6.2 Modal full stats

`Wolumen`:
- Ilość operacji
- Liczba wpłat
- Liczba wypłat
- Suma wpłat
- Suma wypłat

`Średnie i interwały`:
- Średnia wpłata
- Średnia wypłata
- Średni interwał wpłat (dni)
- Średni interwał wypłat (dni)

`Saldo i ryzyko`:
- Bilans netto
- Czas na minusie (dni)
- Liczba okresów ujemnych

## 7) Technical Design Changes (Single File)

### HTML
- Add new top-bar statistics trigger button.
- Update statistics card to keep 4 quick tiles and add modal-open trigger.
- Add modal markup for full statistics.

### CSS
- Add styles for stats trigger icon/button variants.
- Add modal styles for statistics layout sections.
- Add mobile fullscreen modal rules.

### JavaScript
- Extend `computeStats(rows)` to include:
  - `countIn`, `countOut`
  - `avgIn`, `avgOut`
  - `avgIntervalInDays`, `avgIntervalOutDays`
  - `negativeDays`, `negativePeriods`
- Remove obsolete fields and rendering:
  - `negativePoints`, `negativeEntries`
- Add rendering split:
  - quick stats renderer (4 tiles)
  - full stats modal renderer
- Add modal open/close handlers with focus return.

## 8) Edge Cases and Error Handling

- If axis range is not configured, risk metrics dependent on range show zero or `-` according to current app fallback conventions.
- For interval averages with insufficient data (`< 2`), render `-` and avoid division by zero.
- For type averages with zero count, render `-`.
- Same-day multiple transactions are valid and produce `0` interval contribution.

## 9) Acceptance Criteria

- Top bar contains a working `Statystyki` trigger.
- Statistics card contains exactly 4 quick tiles and a working modal trigger.
- Both triggers open the same statistics modal.
- Modal contains grouped sections with full metric set.
- `Punkty na minusie` and `Wejścia na minus` are not visible and not computed.
- `Czas na minusie (dni)` is computed across all axis days, not only transaction days.
- `Liczba okresów ujemnych` matches transitions into negative daily status.
- Average amount and average interval metrics are correctly computed and displayed.
- Modal supports keyboard and backdrop close behavior with focus restoration.

## 10) Manual Verification Checklist

- Open stats modal from top bar trigger.
- Open stats modal from panel trigger.
- Verify modal sections and labels are in Polish.
- Verify left panel shows exactly 4 quick tiles.
- Add only `in` transactions and check averages/intervals behavior.
- Add only `out` transactions and check averages/intervals behavior.
- Add mixed transactions on same and different days; verify interval calculations.
- Configure timeline and create negative balance streaks; verify negative days and periods.
- Start with negative initial balance; verify first negative period is counted.
- Verify warning badge behavior follows negative-day existence.
