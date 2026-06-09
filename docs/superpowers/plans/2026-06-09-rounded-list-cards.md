# Rounded List Cards Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add subtle rounded border frames to list surfaces while keeping the Yinwang-inspired reading style intact.

**Architecture:** This is a CSS-only change. Existing Hugo templates already expose stable selectors for article lists, category lists, and archive rows, so the work stays in `assets/css/main.css` and does not change rendering logic.

**Tech Stack:** Hugo, CSS

---

### Task 1: Style Rounded List Frames

**Files:**
- Modify: `assets/css/main.css`

- [ ] **Step 1: Verify the current build passes**

Run: `hugo --gc --minify`
Expected: PASS

- [ ] **Step 2: Add rounded frame styling to list surfaces**

Update selectors for:
- `.post-list-item`
- `.post-card`
- `.category-terms-item`
- `.archive-year-item`

Apply:
- rounded corners
- subtle border
- light background fill
- card spacing instead of separator-only spacing
- minimal hover state

- [ ] **Step 3: Re-run the build**

Run: `hugo --gc --minify`
Expected: PASS

- [ ] **Step 4: Commit**

```bash
git add assets/css/main.css docs/superpowers/specs/2026-06-09-rounded-list-cards-design.md docs/superpowers/plans/2026-06-09-rounded-list-cards.md
git commit -m "style: add rounded frames to list items"
```
