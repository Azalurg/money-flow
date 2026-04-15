# Tokyo Night Palette Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Apply the Tokyo Night palette consistently across the cashflow app UI, including interactive and semantic states.

**Architecture:** Keep the existing layout and component structure unchanged, and remap all color tokens and hardcoded color usages to semantic variables based on Tokyo Night values. Ensure hover/focus/warning/undo/action states inherit from the same token set to avoid visual drift.

**Tech Stack:** HTML5, CSS3 custom properties, vanilla JavaScript (no logic changes required except visual state hooks already present).

---

## File Structure

- Modify: `index.html` — CSS token remapping and component state color updates

### Task 1: Remap Global Color Tokens to Tokyo Night

**Files:**
- Modify: `index.html`
- Test: `index.html` (visual inspection)

- [ ] **Step 1: Update `:root` palette tokens**

```css
:root {
  --bg: #1a1b26;
  --surface: #24283b;
  --surface-2: #1f2335;
  --border: #414868;
  --text: #c0caf5;
  --muted: #9aa5ce;
  --accent: #7aa2f7;
  --in: #9ece6a;
  --out: #f7768e;
  --danger: #f7768e;
}
```

- [ ] **Step 2: Verify no legacy root colors remain**

Run: search for previous palette hexes (`#10b981`, `#4ade80`, `#f87171`, `#0f1115`)  
Expected: only deliberate, remapped usage remains

- [ ] **Step 3: Commit token remap**

```bash
git add index.html
git commit -m "style: remap global theme tokens to tokyo night palette"
```

### Task 2: Apply Tokyo Night to Core Surfaces and Typography States

**Files:**
- Modify: `index.html`
- Test: `index.html`

- [ ] **Step 1: Update major containers and separators**

```css
body, .topbar, .card, .timeline-shell, .modal-card {
  /* use token-based colors only */
}
```

- [ ] **Step 2: Update text hierarchy and muted labels**

```css
.brand p, .balance-label, .stat-label, .axis-date {
  color: var(--muted);
}
```

- [ ] **Step 3: Verify contrast manually**

Run: open app and inspect topbar/forms/timeline on desktop and mobile width  
Expected: clear readability with Tokyo Night tones

- [ ] **Step 4: Commit surface/typography updates**

```bash
git add index.html
git commit -m "style: apply tokyo night colors to surfaces and text hierarchy"
```

### Task 3: Update Interactive States (Buttons, Hover, Focus, Inputs)

**Files:**
- Modify: `index.html`
- Test: `index.html`

- [ ] **Step 1: Update button states to Tokyo Night semantics**

```css
.btn { /* bg/border/text from tokens */ }
.btn:hover { /* border in #565f89 range */ }
.btn-primary { /* accent-based styling using var(--accent) */ }
```

- [ ] **Step 2: Update focus-visible ring to accent blue**

```css
input:focus-visible,
select:focus-visible,
.btn:focus-visible {
  outline: 2px solid rgba(122, 162, 247, 0.45);
}
```

- [ ] **Step 3: Verify form controls in all states**

Run: hover/focus on inputs/selects/buttons/type toggles  
Expected: consistent Tokyo Night interaction feedback

- [ ] **Step 4: Commit interaction color updates**

```bash
git add index.html
git commit -m "style: align hover and focus states with tokyo night accents"
```

### Task 4: Update Semantic States (In/Out, Warning, Undo, Action Icons)

**Files:**
- Modify: `index.html`
- Test: `index.html`

- [ ] **Step 1: Map semantic positives/negatives to Tokyo Night**

```css
.tx-bubble.in .tx-amount { color: var(--in); }
.tx-bubble.out .tx-amount { color: var(--out); }
.balance-value.negative, .axis-balance.negative { color: var(--danger); }
```

- [ ] **Step 2: Update warning badge, undo button, edit/delete icon states**

```css
.history-warning { /* red semantic derived from #f7768e */ }
.undo-btn { /* blue semantic derived from #7aa2f7 */ }
.tx-edit-btn:hover { /* accent blue */ }
.tx-delete-btn:hover { /* danger red */ }
```

- [ ] **Step 3: Verify functional states manually**

Run:
- create negative balance
- delete item to show undo
- hover edit/delete icons

Expected: all semantic states match Tokyo Night and remain readable

- [ ] **Step 4: Commit semantic state updates**

```bash
git add index.html
git commit -m "style: map semantic warning and action states to tokyo night"
```

### Task 5: Final Visual Sweep and Delivery Commit

**Files:**
- Modify: `index.html` (if minor fixes needed)
- Test: `index.html`

- [ ] **Step 1: Run full visual checklist from spec**

Run manually:
- topbar and cards
- transaction form + recurring fields
- timeline bubbles and axis
- edit modal
- delete/undo states
- desktop + mobile widths

Expected: no out-of-palette artifacts; color consistency across states

- [ ] **Step 2: Final cleanup pass**

Check:
- no hardcoded legacy green/red/emerald values outside intended semantics
- component states still discoverable
- no regressions in readability

- [ ] **Step 3: Final commit**

```bash
git add index.html
git commit -m "style: complete tokyo night palette integration"
```

## Plan Self-Review

- Spec coverage: token mapping, full component state mapping, and Tokyo Night acceptance criteria are covered.
- Placeholder scan: no placeholders present.
- Type consistency: uses existing CSS variable strategy and state classes already in code.
