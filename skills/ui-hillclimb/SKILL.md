---
name: "ui-hillclimb"
description: "Improve a web UI in small, scored, constrained rounds: screenshot every screen and UI part with one fast command, rate each part on an absolute scale, fix the worst parts first, and show before/after in one living artifact. Use when asked to polish, redesign, hill climb or rate a UI, or to make it look better without changing what it does."
---

Climb a UI's quality one round at a time. Each round makes a few changes, measures every affected part before and after, and leaves the app shippable. The user sets the limits. You work inside them.

## Ground rules

Ask once, then hold to these unless the user changes them:

- Button text and headings stay as they are. Users learn the words; renaming them reads as a regression even when the new label is "clearer". Styling is free.
- Section order stays as it is. Layout inside a section is free.
- Brand colours stay. Lighter or deeper shades of them are fine. No gradients unless asked.
- Skip information pages such as terms, privacy and refund policy unless asked.
- Phone first. Desktop comes after the phone parts are good.
- Know the audience. Older users need obvious buttons, pills over bare text links, 40px tap targets and nothing hidden behind a scroll.

## 1. Build the screenshot command first

Before any design work, make one command that captures every screen. The climb only works if a full capture takes seconds, so you re-run it after every change.

- Mock the API inside the browser with Playwright `page.route`. No backend, no database.
- Two datasets of made-up data. `demo` is ordinary. `worst` has long names, unbroken URLs, non-Latin script, emoji, empty fields, missing photos and unknown enum values. Never put real user data in fixtures, screenshots or artifacts, even if the user pastes production rows for you to study.
- Use stock portraits instead of drawn placeholders, matched to the profile's gender. Unsplash works; note the licence in a README next to the photos. Pull real dropdown lists from public GET endpoints so selects show real text lengths.
- Load the app once per login state and move between screens with client-side navigation: `history.pushState`, then dispatch a `popstate` event. Remount a route by navigating away and back.
- Wait for idle, not for a fixed time: no pending mocked requests, the DOM quiet for about 60ms, fonts and images loaded.
- Capture at 2x as JPEG quality 92. PNG encoding at 2x is twice as slow and looks the same for UI.
- Parallelism: 2 browsers and 4 tabs is the sweet spot. One Chromium encodes screenshots one at a time, so a second browser helps more than more tabs.
- Rebuild only when a source file is newer than the last build.
- Flags worth having: `--only`, `--data`, `--vp`, `--static`, `--parts`. Include a 320px viewport in `--vp`.

Measure the run and report it. A good target is under 10 seconds for phone screens with both datasets.

## 2. Capture parts, not just pages

Whole pages are too big to judge. Mark the parts of the UI with `data-shot="name"`: the navbar, the login form, each card, each section of a long page, each dialog. Add a `--parts` mode that crops each one with a 12px margin. Split anything taller than about two phone screens into sections.

Keep a list in the harness of which parts belong to which screen, so `--only login-form` captures one part in about a second.

## 3. Rate honestly

- Use an absolute scale: 10 is a best-in-class consumer app. Never rate against the previous version. A "4 called a 9" destroys trust fast.
- Look at each part alone at full 2x resolution. Never rate from a shrunken contact sheet; borders, spacing and type are invisible there.
- Write the baseline down per part with its main problems before changing anything.
- Target 7 on every part before pushing any part toward 8.

## 4. Climb

1. Fix the problems shared by many parts first. Examples that paid off: 13px sentence-case field labels instead of 12px uppercase, a neutral secondary button so the primary is the one main action, white dialog headers, a two-line brand box.
2. Then go part by part, worst first.
3. After each change, capture that part and put before and after side by side at full resolution. Keep the change only if the after is better.
4. Check worst-case data and a 320px phone for anything that touches text.

Rules that kept coming back:

- One border per edge. A bordered button inside a bordered card, or a tinted box inside a bordered card, looks doubled. Use a fill instead of a second border.
- Concentric corners. An inner radius is the outer radius minus the padding between them.
- One solid primary action per view. Destructive actions that open a confirmation are quiet; the confirmation step is solid red.
- Button labels never wrap on a phone. Stack dialog actions full width with the primary on top.
- Nothing is hidden behind a horizontal scroll. Five tabs on a phone become a 3+2 grid, not a scroller.
- Native selects cannot wrap. Give long options a full row; don't shrink text under 16px or iOS zooms on focus.
- Every text pair passes WCAG AA. Check the numbers, not your eyes.
- When a decoration looks unfinished, such as a short underline under a title, remove it rather than tune it.

## 5. Delegate rounds, verify on the main thread

Run each round in a background sub-agent with a tight brief. Give it the user's feedback in their own words, the files involved, the rules above and the checks to run. Ask it to report commits, a before and after score per part, and the paths of its comparison images. Keep the main thread free to talk to the user.

- Never edit files a running sub-agent is editing. Queue new feedback to it with a message instead.
- Capture "current state" screenshots from a separate git worktree at a fixed commit, so in-progress edits don't leak in. If the worktree shares `node_modules`, clear the shared build cache afterwards.
- When the sub-agent reports, open its comparison images yourself before telling the user it worked.
- Leave uncommitted files of a running sub-agent alone; it commits when its checks pass.

## 6. One living artifact

Publish one page and update it in place every round, so the user keeps one link.

- Switches for data, device and compare: demo or worst case, phone or desktop, now or the original baseline.
- Worst case shows only screens and parts that actually change with worst-case data. Compare images with a pixel diff; judge a dialog by the dialog itself, not the page dimmed behind it.
- A "Round N" badge and a log of rounds: number, commit, what changed.
- Any shot opens full size on tap.

Generate the page from a script and a rounds file, so a new round is "capture, add a line, regenerate, publish".

## 7. Turn feedback into rules

Every complaint is a rule that should never need repeating. Mismatched corner radii, a half-length underline, tabs hidden under a scroll, renamed buttons, two borders, a wrapped button label. Add each to the brief for the next round, and when the project grows a layout checker, encode them there.

## Constrained colour rounds

When the palette needs work, change values, not roles:

- Keep the brand anchors exact. Derive other shades in OKLCH within a few degrees of the anchor hue, with even lightness steps.
- Inject candidate palettes as CSS variable overrides from the screenshot tool, so no app code changes until the user picks one.
- Reject any candidate that fails AA contrast, and cap how much of each screen the strongest brand colour covers.
- Show the candidates with a palette switch on the living artifact, on identical screens.

## Commits

Small logical commits, one idea each. Before every commit, run the formatter, lint and type check, and both screenshot runs with zero errors. Follow the repo's commit style.
