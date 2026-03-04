# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the project

No build step. Open `index.html` directly in a browser:

```bash
start index.html          # Windows
open index.html           # macOS
xdg-open index.html       # Linux
```

For a local HTTP server (avoids any file:// restrictions):

```bash
python -m http.server 8080
# or
npx serve .
```

## Architecture

Single-page app in three files:

- **`index.html`** — all markup, JS logic, and translation strings in one file
- **`styles.css`** — all styles, imported by `index.html`
- **`ministers.csv`** — data reference only; its content is hard-coded into the HTML, not fetched at runtime

### Tab system

Four panels (`panelTimeline`, `panelMinisters`, `panelHolidays`, `panelQuiz`) are toggled via `setTab(name)`, which adds/removes the `.active` class on panels and the corresponding `.tab-btn` buttons. Only the active panel is visible (`display: block`).

Nav buttons are direct `<button class="tab-btn">` elements with IDs `tabTimeline`, `tabMinisters`, `tabHolidays`, `tabQuiz` — no dropdown.

### Quiz

Quiz logic lives entirely in `index.html`. Key pieces:

- `quizQuestions` — array of `{ event: {ru, el}, correctDate }` objects
- `generateWrongDates(correctDate)` — generates 3 plausible wrong date options
- `initializeQuiz()` — shuffles options, resets state; called lazily on first visit to the quiz tab
- State tracked in the `quiz` object: `currentQuestion`, `correctAnswers`, `checked`, `selectedOption`
- Quiz is re-initialized via `restartQuiz()` (two restart buttons: `quizRestartBtn` and `quizRestartBtn2`)

### i18n

All UI strings live in the `translations` object in `index.html` — two top-level keys: `ru` and `el`. Translation is applied by `applyTranslations()`:

- Elements with `data-i18n="key"` → `innerHTML` set from the translation object (supports embedded HTML tags like `<time>`, `<span>`)
- Elements with `data-ru="…" data-el="…"` → `textContent` switched directly (used for simple table cells)
- `data-i18n-aria="key"` → `aria-label` attribute

The current language is stored in `localStorage` under the key `lang`. Default is `ru`. Switching language also reformats quiz date options via `formatDate()`.

### Adding content

**New timeline event**: add an `<li>` inside the relevant `<article class="era">` in `index.html`, then add matching `ru` and `el` keys to both language objects in `translations`.

**New minister**: add a `<article class="minister-card">` in the appropriate `<section class="minister-group">` and add a row to `ministers.csv` for reference.

**New holiday**: add a `<tr>` to the relevant `<table>` in `panelHolidays`; use `data-ru` / `data-el` attributes on each `<td>` for bilingual support.

**New quiz question**: add an entry to `quizQuestions` with `event: { ru, el }` and `correctDate` in `YYYY-MM-DD` format.
