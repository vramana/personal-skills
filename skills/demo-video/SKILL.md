---
name: "demo-video"
description: "Create a demo video"
---
 
Create a single demo video of <FEATURE> in <APP> (<FRONTEND_PATH>). Record the real frontend, not a mockup, and follow the style spec below exactly so every demo video looks the same.
 
## What to demo
- Feature: <one sentence on what it does and who it is for>
- Story, 3 or 4 steps: <e.g. 1. turn on the setting, 2. configure it on one item, 3. show the result>
If the story isn't given, read the PR diff and pick the shortest path that shows the user-visible change and its result. Use the same domain data in every step, so the result screen shows what the earlier steps set up.
 
## How to record
1. Start the frontend dev server in the background: <DEV_SERVER_COMMAND> (e.g. `npx vite --port 5173 --host 127.0.0.1`).
2. Install `playwright-core@1.55.0` in a scratchpad folder (not the repo). Launch the preinstalled Chromium at <CHROMIUM_PATH>. Never run `playwright install`.
3. Fake every backend call with `context.route("**/*")`. Hold the state in memory, so mutations change what the next query returns.
   - Fake each API the frontend calls: <API_LIST, with URL patterns and response shapes>.
   - Fake authentication so the user is signed in as an admin with a token that does not expire.
   - Turn on any feature flags the feature needs.
   - Log any request you haven't faked.
   - Let the dev server origin and Google Fonts through. Answer everything else (analytics, third-party SDKs) with 204.
   - Set any session values the app needs in storage with `addInitScript`, before the page loads.
4. Make the data realistic:
   - a believable organization and location
   - people names and `@example.com` emails
   - IDs in the same format as production
   - names that match the story
   - no setup banners or empty-state warnings
5. Take a screenshot to check the page renders before you script the walkthrough.
## Video format
- Viewport and recording size 1280×800, `deviceScaleFactor: 1`, recorded with `recordVideo` on the browser context.
- Final file: H.264 MP4, `-pix_fmt yuv420p -crf 22 -preset slow -movflags +faststart -r 30`.
- Trim the start so the first frame is the fully faded-in title card. Log `Date.now()` relative to context creation just before the card shows, and use about that time plus 0.5s as `-ss`.
- Aim for 60–100 seconds.
## Overlay style
Inject one `<style>` and three fixed elements with `context.addInitScript` (they must survive navigation):
- Hide dev tools widgets (e.g. React Query devtools: `.tsqd-parent-container, .tsqd-open-btn-container { display: none !important; }`).
- **Cursor** (`#demo-cursor`):
  - 22px circle, `rgba(79,70,229,.35)` fill, `2px solid #4f46e5` border, z-index 100000, `pointer-events: none`
  - follows `mousemove`
  - scales to 0.7 with a darker fill on mousedown, with a 0.12s transition
- **Caption** (`#demo-caption`):
  - box: fixed, bottom 28px, centered, `max-width: 1000px`, background `rgba(17,20,45,.92)`, white text, `padding: 14px 24px`, `border-radius: 12px`, `box-shadow: 0 8px 30px rgba(0,0,0,.25)`, text centered
  - font: `500 20px/1.4 Montserrat`
  - fades in over 0.3s
  - a `.top` modifier moves it to `top: 56px`. Use it whenever the caption would cover what you're pointing at, such as table rows near the bottom.
- **Title card** (`#demo-card`):
  - full screen, background `#222647`, white Montserrat, z-index 100001, fades over 0.4s
  - `h1` 46px bold, `p` 24px at 0.9 opacity with `max-width: 900px`
- **Highlight** (`.demo-highlight`): `outline: 3px solid #f59e0b; outline-offset: 2px; border-radius: 6px`.
## Movement and timing
- `moveTo(locator)`: scroll it into view, then `page.mouse.move` to its center with `steps: 25`, then wait 250ms.
- `click(locator, pause = 700)`: moveTo, mouse down, wait 90ms, mouse up, then wait `pause`.
- Highlight the element you're talking about while its caption shows, and remove the highlight afterwards.
- Hold a caption about 3.5–5 seconds, longer for longer text. Leave about 1–2.5 seconds after a click that navigates or saves.
- Find elements by role or exact text (`getByRole`, `getByText(..., { exact: true })`, `getByLabel`). The caption text is in the DOM too, so loose text matches will hit it.
## Structure
1. **Title card** (about 4s): the feature name as h1, plus one sentence on the benefit.
2. **Context caption** on the starting screen: what's there today and what's wrong with it.
3. **Steps**: start each with a caption saying "Step N: <action>." Then, for every important control, highlight it and caption what it does.
4. **Result**: walk through the outcome one item at a time, with a highlight and caption for each. Include one item that shows unchanged behaviour still works. End the video on the result screen.
## Writing captions
- Write plain, short sentences in the present tense, one idea per caption, about 20 words at most.
- Describe what the user sees and why it matters. Avoid internal names such as function names, column names or flags.
- Use the names from the demo data, e.g. "Maria's booking with Alex Morgan opens only Room 1."
## Check before delivering
- Pull a contact sheet with `ffmpeg -vf "fps=1/4,scale=640:-1,tile=3x4"` and look at it. Check that:
  - no caption covers the element it describes
  - there are no blank or loading frames at the start
  - no error screens or dev widgets show
  - every step's result is visible
- Fix anything wrong and record again.
- Stop the dev server, keep everything out of the repo, and send the MP4 with a one-line caption.
- In the reply, say that the backend data was faked.
