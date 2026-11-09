---
name: "ui-hillclimb"
description: "Improve a web UI in small, scored, constrained rounds: screenshot every screen and UI part with one fast command, build a measuring harness before each new kind of round, enforce design tokens with checks, rate each part on an absolute scale, fix the worst parts first, and show before/after in one living artifact. Use when asked to polish, redesign, hill climb or rate a UI, or to make it look better without changing what it does."
---

Climb a UI's quality one round at a time. Each round makes a few changes, measures every affected part before and after, and leaves the app shippable. The user sets the limits. You work inside them.

## Ground rules

Ask once, then hold to these unless the user changes them:

- Wording stays as it is: button text, headings and dialog titles. Users learn the words; renaming them reads as a regression even when the new label is "clearer". Styling is free. If wording drifted, restore it from the main branch, which is the source of truth, and list any other copy changes for the user to decide on.
- Section order stays as it is. Layout inside a section is free.
- Brand colours stay. Lighter or deeper shades of them are fine. No gradients unless asked.
- Skip information pages such as terms, privacy and refund policy unless asked.
- Phone first. Desktop comes after the phone parts are good.
- Know the audience. Older users need controls that look like buttons rather than bare text links, 40px tap targets and nothing hidden behind a scroll.

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

## 2. Build the harness for a round before the round

Before a new kind of round, ask whether the screenshot tool can measure what the round is about. If it can't, build that first, verify it, commit it, and only then do the design work. Examples from practice:

- Colour: a `--palette file.json` flag that injects CSS variable overrides before first paint, so candidates need no app changes. A contrast checker that reads the real colour pairs from the code, not a guessed list. A `--measure` flag that reports how much of each screen each colour family covers.
- Corners: a static check that fails on any radius outside the token set, and a runtime check that reads the computed radius of every visible element on every screen.
- Comparison: an `--anchors` flag that saves the position of every short text on the page, so two versions of a screen can be lined up by content.

Prove each tool on a known answer before trusting it. An empty palette must give identical screenshots, and an obviously wrong colour must show up.

## 3. Capture parts, not just pages

Whole pages are too big to judge. Mark the parts of the UI with `data-shot="name"`: the navbar, the login form, each card, each section of a long page, each dialog. Add a `--parts` mode that crops each one with a 12px margin. Split anything taller than about two phone screens into sections.

Keep a list in the harness of which parts belong to which screen, so `--only login-form` captures one part in about a second.

## 4. Rate honestly

- Use an absolute scale: 10 is a best-in-class consumer app. Never rate against the previous version. A "4 called a 9" destroys trust fast.
- Look at each part alone at full 2x resolution. Never rate from a shrunken contact sheet; borders, spacing and type are invisible there.
- Write the baseline down per part with its main problems before changing anything.
- Target 7 on every part before pushing any part toward 8.

## 5. Climb

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
- One component per repeated pattern. Two styles of back button is a bug; route every instance through one component and keep each label.
- Rare or destructive actions appear once, in one place. A Delete Profile block repeated under every tab is noise.
- Locked fields must stay readable: full-strength text, a warm read-only fill, a lock cue instead of a dropdown arrow, and one short note saying why, addressed to the user as "your".
- A fill-only button on a tinted page can vanish. If its fill contrasts less than 3:1 with the background, give it an edge or a shadow. No palette tweak fixes that.
- Popups such as date pickers can run off a 320px screen. Keep them inside the viewport and check at 320px.

## Tokens, and checks that enforce them

Pick a handful of named tokens and ban everything else. For corner radius, three were enough: `control` 8px for buttons, fields, tiles and day cells, `surface` 12px for cards, dialogs and popups, and `inset` 4px for things nested tightly inside a control. Switch off the framework's default scale so stray values stop compiling, then back it up with a static check in CI and a runtime check in the screenshot tool. "Insane" pill and circle shapes crept in exactly where no token stopped them.

## 6. Delegate rounds, verify on the main thread

Run each round in a background sub-agent with a tight brief. Give it the user's feedback in their own words, the files involved, the rules above and the checks to run. Ask it to report commits, a before and after score per part, and the paths of its comparison images. Keep the main thread free to talk to the user.

- Never edit files a running sub-agent is editing. Queue new feedback to it with a message instead.
- Capture "current state" screenshots from a separate git worktree at a fixed commit, so in-progress edits don't leak in. If the worktree shares `node_modules`, clear the shared build cache afterwards.
- When the sub-agent reports, open its comparison images yourself before telling the user it worked.
- Ask sub-agents to report what they found that the user must decide, such as a check that fails for every candidate. Pass those on plainly instead of burying them.
- Leave uncommitted files of a running sub-agent alone; it commits when its checks pass.

## 7. One living artifact

Publish one page and update it in place every round, so the user keeps one link.

- Switches for data, device and compare: demo or worst case, phone or desktop, now or the original baseline.
- Worst case shows only screens and parts that actually change with worst-case data. Compare images with a pixel diff; judge a dialog by the dialog itself, not the page dimmed behind it.
- A "Round N" badge and a log of rounds: number, commit, what changed.
- Any shot opens full size on tap.
- Scroll compared shots together by content, not by percentage. Layouts differ in height between versions, so match the same text on both sides, keep only pairs that don't cross, and map the centre of one view to the other between those points.
- Section links in the toolbar only point at sections visible in the current mode, and they scroll with `scrollIntoView`. Hash jumps can fail inside an artifact frame, and a link to a hidden section looks broken.
- When the user has to choose between options such as palettes, make a separate compare page: pick any two, a role-by-role table with swatches and hex codes, then every element side by side.
- If the session's scratch folder moves, copy the page over and publish with the existing URL so the link stays the same.

Generate the page from a script and a rounds file, so a new round is "capture, add a line, regenerate, publish".

## 8. Turn feedback into rules

Every complaint is a rule that should never need repeating. Mismatched corner radii, a half-length underline, tabs hidden under a scroll, renamed buttons, two borders, a wrapped button label. Add each to the brief for the next round, and encode it as a check when you can.

When feedback is ambiguous, say how you read it, act on that reading, and flag it so the user can correct you in one line. A screenshot with one word under it usually names the problem shown; ask only if two readings lead to different work.

## Constrained colour rounds

When the palette needs work, change values, not roles:

- Keep the brand anchors exact. Derive other shades in OKLCH within a few degrees of the anchor hue, with even lightness steps.
- Inject candidate palettes as CSS variable overrides from the screenshot tool, so no app code changes until the user picks one.
- Reject any candidate that fails AA contrast, and cap how much of each screen the strongest brand colour covers.
- Show the candidates with a palette switch on the living artifact, on identical screens, with swatches and hex codes for each role next to the screenshots.
- Tight constraints make candidates nearly identical. A ±6° hue band around fixed anchors left five palettes that differed mostly in text darkness. Say so, and offer to loosen the rule for the lightest shades.
- How much of a screen the brand colour covers comes from layout, such as a full-width coloured footer, not from colour values. Measure it, but fix it in layout.

## Commits

Small logical commits, one idea each. Before every commit, run the formatter, lint and type check, and both screenshot runs with zero errors. Follow the repo's commit style.
