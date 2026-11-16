# AGENTS.md

## Project
Single-file wedding invitation: `Saurabh_Kajal_Wedding_Invitation.html` (~760 lines). No build step, no local assets, no tests. Git repo currently on `master` with no commits.

## Architecture
- Built on **ShaadiPath Template 11**. `<base href="https://assets.shaadipath.com/templates/template11/">` makes `style.css`, `script.js`, images, videos and music load from that CDN. Inline scripts at the bottom override/extend the template.
- `<script type="application/json" id="wedding-config">` holds all data, exposed as `window.__WEDDING_CONFIG__`. This is the source of truth for names, dates, invite text, RSVP, calendar URLs.
- Sections: Intro (click-to-start video) → Hero → Invite → Events → Couple → Things to Know → RSVP → Countdown → Footer. Floating music/nav buttons.
- Template `script.js` behaviors that matter:
  - Injects `invite.blessing` into `.invite-overline` (replaces the static placeholder text).
  - Builds names, dates, venues, RSVP link, calendar buttons from config.
  - Reveals `.reveal-item` via IntersectionObserver after the intro finishes.
  - Null-checks `#rsvpBtn` / `#rsvpHelper`, so those elements can be absent safely.

## Custom code (all inline, near end of file)
- `<style id="saurabh-kajal-event-styles">` + `<script id="saurabh-kajal-events">`: renders 3 day cards (3–5 Dec 2026) into `#eventsList` plus a "Baraat Route & Wedding Location" note. Guards via `dataset.customRendered`.
- `<script id="saurabh-kajal-event-reveal">`: IntersectionObserver fallback for event-card entrance animation.
- Third `<style>` block: invite-card text fitting rules + custom Things-to-Know grid styles (`custom-ttk-grid`, `custom-ttk-card`) and its renderer script (6 info cards into `#ttkGrid`).

## Changes made (do not regress)
1. **Event cards scroll animation** — native CSS scroll-driven entrance (`animation-timeline: view()`, staggered `animation-range`) for `.custom-day-card`; route note fades up. Fallback for browsers without `view()` uses `.is-in` + IntersectionObserver. Reduced-motion shows everything statically.
   - **Critical**: `.events-section` must use `overflow:clip`, not `overflow:hidden`. `overflow:hidden` makes the section the nearest scroll container, freezing the `view()` timeline (cards stay at end state). Override is in the custom style block.
2. **RSVP CTA removed** — the WhatsApp "Confirm" button (`#rsvpBtn`) and helper text (`#rsvpHelper`) were deleted from the markup. Save-the-Date calendar buttons remain and are populated from `calendarUrls`. Unused `rsvp.mode/btnText/whatsapp` config is harmless.
3. **Invite blessing fits the card** — the config blessing is long (~200 chars) and used to clip at the top/bottom of `.invite-card-text` (flex-centered, `overflow:hidden`). Fixed by:
   - Deleting the duplicate `.invite-blessing-note` paragraph (grandparents' names are already in the blessing).
   - Responsive rules: line-height/margins tightened globally; ≤520px, ≤340px, ≤310px progressively smaller type/padding/rules.
   - Verified no clipping from 320px to 1920px.

## Verification recipe
Tabbit CLI is unavailable here (`CLI_REGISTRATION_UNAVAILABLE`). Use headless Chrome via `puppeteer-core` (installed in `%TEMP%\pptr-check`):
```powershell
Set-Location "$env:TEMP\pptr-check"; node <script>.js
```
`executablePath: "C:\Program Files\Google\Chrome\Application\chrome.exe"`, args `--allow-file-access-from-files --autoplay-policy=no-user-gesture-required --mute-audio`. Skip the intro overlay by removing `#intro`, removing `body.intro-active`, adding `is-visible` to `#page`/`#heroContent`, and adding `is-in` to `#inviteCard`. Inspect computed styles / bounding boxes rather than screenshots (this model cannot view images). Cloudflare RUM CORS errors on `file://` are expected and unrelated.

## Gotchas
- Don't add `overflow:hidden` to `.events-section` again (breaks scroll animation).
- Don't re-add `.invite-blessing-note` unless the config blessing no longer contains the grandparents' names.
- `.reveal-item` elements are observed only after the intro; dynamic content needs its own observer (as the event reveal script does).
- Template CSS animation selectors target `.invite-card-text > *`; keep invite-card children as direct children if changing markup.
- Client-facing names: hero/invite use "Aayu. Kajal" / "Chi. Saurabh"; footer uses short "Kajal" / "Saurabh".
