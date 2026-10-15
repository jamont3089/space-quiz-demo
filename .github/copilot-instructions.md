# Space quiz

## Running and checking the app

Open `index.html` directly in a browser using a `file://` URL. In PowerShell from the project folder:

```powershell
Start-Process .\index.html
```

For integrated-browser work, open a Browser canvas pointed at the file's absolute URL. Use `?scoutTheme=light` and `?scoutTheme=dark` to check each theme independently of the operating system.

Browser smoke checks:
- For a single-question check, choose Neil Armstrong on question 1: score becomes 1, progress becomes 10%, the answer turns green, an explanation appears, and Next becomes enabled. Further clicks must not add points.
- Choose an incorrect answer: score stays unchanged, progress still advances, the selection shakes red, and the correct answer is revealed.
- Complete all ten questions with all-correct, all-wrong, and mixed answers. Verify score, correct/missed counts, percentage, emoji reaction, and completed mission telemetry.
- Replay must reset score, progress, sector, answer controls, and focus. Check real Tab, Shift+Tab, Enter, and Space input when browser automation supports it; programmatic focus and `.click()` do not verify keyboard input.
- Measure contrast in both themes and feedback states. Wait for CSS transitions to settle before sampling computed colors; immediately sampled borders can still be the previous state.

## Architecture

The runtime deliverable is a single self-contained `index.html`: inline CSS, inline SVG artwork, and plain JavaScript. Preserve direct file loading and offline operation without servers, dependencies, external fonts, or asset requests.

The head script sets `data-theme` before rendering. The final script owns the fixed ten-question array and in-memory `index`, `score`, and `answered` state. Reloading starts a new quiz.

`renderQuestion()` creates answer buttons and resets feedback; `selectAnswer()` locks the current question, applies feedback, updates score and progress, and enables Next. Next advances only after an answer. `showResults()` switches from the quiz to results; the restart handler resets the full session.

Progress measures answered questions, while sector measures the currently displayed question. `updateProgress()` synchronizes the ARIA progress value, fill width, completed count, percentage, score, and uplink status. `renderQuestion()` separately updates the sector when advancing.

## Project conventions

- Questions have `category`, `question`, four `options`, a zero-based `answer` index, and explanatory `fact`. Ten-question totals also appear in markup, question numbering, SVG progress segments, and result thresholds; keep these surfaces consistent if changing quiz length.
- DOM IDs are shared contracts between markup and JavaScript via `get(id)`. Quiz/results visibility uses `hidden`; preserve the `[hidden]` CSS rule.
- Use `textContent` for question, option, and feedback copy. Existing `innerHTML` uses are limited to controlled structural templates.
- Colors use the `--cp-*` theme variables. Preserve the base light/dark palette and use the derived `--cp-readable-*` and `--cp-control-border` variables for accessible text and control boundaries rather than hardcoded component colors.
- Theme defaults to `prefers-color-scheme`; `scoutTheme` overrides it. Operating-system theme changes update the page only without that override.
- Keep the centered 504px maximum-width layout, 15px sans-serif body, and responsive command-center wrapping. Telemetry uses Consolas and a 12px size.
- Preserve green pop/red shake feedback, the hex-grid mission panel, orbit animation, and segmented progress. All animation and transition rules must honor `prefers-reduced-motion`.
- Feedback communicates outcomes with text and symbols as well as color. Preserve its polite, atomic live region, answer labels, progressbar attributes, decorative SVG `aria-hidden`, and labeled result emoji.
- Answering moves focus to Next; advancing/replaying moves focus to the question heading; results moves focus to the result heading. These headings have `tabindex="-1"` for programmatic focus, and keyboard focus has a visible outline.
- Maintain WCAG AA contrast: 4.5:1 for normal text, 3:1 for qualifying large text and essential UI indicators. Measure rendered blended backgrounds and animated scan colors, not just raw palette tokens.
