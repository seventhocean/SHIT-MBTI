# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SHITI (地狱笑话人格测试) is a single-page dark-humor personality quiz application. Deployed at https://sbti.unun.dev. Original concept by [B站@蛆肉儿串儿](https://www.bilibili.com/video/BV1LpDHByET6/).

This is a mirror with images and HTML separated. The site was refactored from SBTI (original MBTI-parody theme) to SHITI (dark humor/hell jokes theme).

## Project Structure

- `index.html` — Entire application: HTML structure, inline CSS, and inline JavaScript (no build step, no dependencies)
- `image/` — Personality result images (PNG/JPG), referenced by `TYPE_IMAGES` mapping in the JS (currently the mapping is empty — no images are wired up)

## Architecture

**Three screens** controlled by CSS class toggling (`screen` / `screen.active`):
1. `#intro` — Landing page with start button
2. `#test` — Quiz with progress bar, radio-button questions, submit validation
3. `#result` — Personality result with poster image, type description, 15-dimension breakdown

**Scoring logic (`computeResult`):**
- 30 regular questions shuffled at start, mapped to 15 dimensions (S1-S3, E1-E3, A1-A3, Ac1-Ac3, So1-So3)
- Each dimension has 2 questions, scored 1-3 per answer → raw score 2-6, mapped to levels L/M/H
- Results matched against 25 type patterns (e.g. `"HHH-HMH-MHH-HHH-MHM"`) using Manhattan distance + exact-match tiebreaker
- Special types: `DRUNK` (triggered by hidden `drink_gate_q1` → `drink_gate_q2` question gate), `HHHH` (fallback when no type matches >60%), `CHAOS` (also a fallback, but defined in NORMAL_TYPES)

**Question flow (`startTest`, `getVisibleQuestions`, `renderQuestions`):**
- 30 regular questions are shuffled; 1 special gate question (`drink_gate_q1`) is inserted at a random position
- If the user selects option 3 ("饮酒") on the gate question, a follow-up question (`drink_gate_q2`) is dynamically inserted
- Radio change events trigger `updateProgress()` which enables/disables the submit button

**Key JS objects** (in `index.html` `<script>`):
- `dimensionMeta` — dimension names and model groupings (5 models: 自我模型, 情感模型, 态度模型, 行动驱力模型, 社交模型)
- `questions` — 30 quiz questions
- `specialQuestions` — hidden gate/trigger questions (drink gate)
- `TYPE_LIBRARY` — 27 personality type definitions (code, Chinese name, intro, description)
- `TYPE_IMAGES` — mapping from type code to image path (empty object, no images wired up)
- `NORMAL_TYPES` — 25 type patterns for matching (excludes DRUNK and HHHH)
- `DIM_EXPLANATIONS` — per-dimension L/M/H level descriptions

**Event listeners:**
- `#startBtn` → `startTest(false)`
- `#backIntroBtn` → `showScreen('intro')`
- `#submitBtn` → `renderResult()`
- `#restartBtn` → `startTest(false)`
- `#toTopBtn` → `showScreen('intro')`

## Development Commands

There is no build system. The app is a static HTML file.

```bash
# Serve locally (any static server works)
python3 -m http.server 8080
# or
npx serve .

# Open index.html directly in a browser
open index.html
```

## Common Tasks

- **Add/edit questions:** Modify the `questions` array in `index.html`. Each question has `id`, `dim`, `text`, and `options` (with `label` and `value`).
- **Add/edit personality types:** Update `TYPE_LIBRARY`, `TYPE_IMAGES`, and `NORMAL_TYPES` in the script.
- **Change styling:** All CSS is in the `<style>` block in `<head>`. CSS variables in `:root` control the theme.
- **Add new images:** Place files in `image/` and add the mapping to `TYPE_IMAGES`.
- **Add special/hidden questions:** Add to `specialQuestions` array and wire up the trigger logic in `getVisibleQuestions()` and `computeResult()`.
