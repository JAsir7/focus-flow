# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Focus Flow — a Pomodoro timer web app (tasks, day streaks, custom durations, confetti). The entire app lives in **`focus-flow/index.html`**: HTML, CSS, and JS are deliberately inlined in that one file so it works when opened directly (file://, preview panels) with no server. Keep it self-contained — do not split the CSS/JS back out into separate files.

The root `index.html` is only a meta-refresh redirect into `focus-flow/` for GitHub Pages; don't put app code there.

`tic-tac-toe/` is a separate, unrelated git repository that happens to live in this folder. It is gitignored — never stage it or reference it from this project.

## Commands

There is no build step, linter, or test suite. Plain HTML/CSS/JS, no dependencies.

- Run locally: `npx serve focus-flow -l 5310` (this is what `.claude/launch.json`'s `focus-flow` config does — prefer starting it via the preview panel).
- Or just open `focus-flow/index.html` directly.

## Deployment

GitHub Pages serves the `main` branch root of `JAsir7/focus-flow` at https://jasir7.github.io/focus-flow/ — every push to `main` redeploys automatically (takes a minute or two). `gh` is at `/opt/homebrew/bin/gh` (not on the default PATH).

If the UI changes, regenerate the README screenshot with headless Chrome against the running dev server:

```sh
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu \
  --window-size=900,1000 --screenshot="focus-flow/screenshot.png" http://localhost:5310
```

## Architecture notes

All app logic is one IIFE in the `<script>` block of `focus-flow/index.html`:

- **State**: a single object persisted to `localStorage` under the `focusFlowState` key on every mutation via `saveState()`. `loadState()` handles day rollover: it zeroes `sessionsCompletedToday` on a new day and resets the streak unless the last completion was today or yesterday. Timers never resume as "running" across a reload.
- **Durations**: breaks come from the `DURATIONS` constant; the focus duration is user-adjustable (`state.workDuration`, set by the slider). Always go through `getDuration(mode)` rather than reading `DURATIONS` directly.
- **Session cycling**: work → short break, every 4th completed work session → long break (`nextModeAfterWork(sessionNumber)` — note it takes the session count *after* increment; passing the pre-increment count was a real bug once).
- **Rendering**: `renderTimer()` and `renderTasks()` rebuild the UI from state; there is no framework. The progress ring is an SVG circle driven by `stroke-dashoffset` against `RING_CIRCUMFERENCE`; per-mode theming works by reassigning the `--accent` CSS variable to `--work`/`--short`/`--long`.
