# Tradespace launch checklist

Review of the website as a cold visitor from outreach or ads. Launch: **14 September**.

How to use: go through item by item and check them off. **P0 first.** Then blockers before ad spend. Everything else is ordered by impact, not by how it appears on the page.

---

## P0: waitlist must convert

Do these before anything else. A filled form that does not land in Brevo, or a signup with no thank-you, is a lost lead.

- [x] **Connect the bottom waitlist form to Brevo**
  - `#beta-cta-form` now POSTs for real to the same Brevo endpoint as the hero: `fetch(action + '?isAjax=1')` with `new FormData(form)` — identical to what Brevo's own `main.js` does
  - Fields match the hero exactly: `EMAIL`, `OPT_IN=1`, `locale=sv`, honeypot `email_address_check`, `g-recaptcha-response`
  - **Brevo's script only binds one form** (`document.querySelector("#sib-form")`), so a duplicated Brevo block would get no behaviour. The bottom form does its own POST instead
  - **Brevo enforces reCAPTCHA server-side**: a POST without a token returns `400 {"success":false,"errors":{"g-recaptcha-response":…}}`. The bottom form therefore has its own reCAPTCHA widget, rendered explicitly by JS and revealed once the visitor starts typing (same pattern as the hero). Site key is read from the hero widget at runtime so the two cannot drift apart
  - Consent notice with a link to the privacy policy now sits under the field (the old fake form had none)
  - Errors are mapped to Swedish: bad address, expired captcha, network failure. The captcha resets after every failure
  - Progressive enhancement: without `fetch`/`FormData` the form posts natively to Brevo; a `<noscript>` line points at info@tradespace.se
  - Not verified end-to-end yet: a real signup needs a human-solved captcha from an allowed origin. See "Verify after deploy" below

- [x] **Capture UTM into hidden Brevo fields** — code done, needs 5 attributes created in Brevo
  - All five (`utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`) are read from the URL on landing, stored in `sessionStorage` and attached as hidden `UTM_*` inputs to **both** forms. The hero picks them up automatically because Brevo builds its own `FormData` from the form
  - Latest campaign wins; values survive in-session navigation and are capped at 200 chars
  - **Precondition: create `UTM_SOURCE` / `UTM_MEDIUM` / `UTM_CAMPAIGN` / `UTM_CONTENT` / `UTM_TERM` as text attributes in Brevo (Settings → Contact attributes) before the first ad click.** Brevo maps form fields to existing contact attributes; fields with no matching attribute are not guaranteed to be accepted
  - Fallback if they are missing: the bottom form retries the POST automatically without the UTM fields, so the signup still lands (contact without attribution beats a lost lead). The hero form cannot auto-retry — Brevo resets the captcha on error — so the UTM fields are stripped after the first failure and the visitor's next attempt goes through
  - Probed: adding unknown fields (even `ZZ_DEFINITELY_NOT_AN_ATTRIBUTE`) does not change Brevo's response — it still fails on the captcha check, so unknown fields at least do not fail earlier in validation

- [x] **Thank-you confirmation** — built as a popup instead of a `/tack` page
  - Native `<dialog id="ts-thanks-dialog">` shown on success from **both** forms. Short formal Swedish copy plus a single OK button
  - Not dismissible by clicking beside it (native `<dialog>` backdrop does not close). Escape still works as the keyboard equivalent of OK
  - Scroll is locked while it is open and unlocked in the OK handler itself — not only on the `close` event, so the page can never be left unscrollable
  - `generate_lead` fires once per page load, from whichever form converted (guarded by `window.__tsWaitlistLeadFired`)
  - Hero path listens only for Brevo's own `sib-form-message-panel--active` class. A broader visibility check fired falsely when `sib-styles.css` failed to load, which would have shown the thank-you on page load
  - After conversion `html.ts-waitlist-joined` hides the sticky CTA for the rest of the session
  - Deliberately left out (was in the original `/tack` spec): the second ask (industry / buy-vs-sell) and the “dela med en kollega” moment. Both need Brevo fields, so they belong with the UTM work
  - No extra fields were added to the hero form

- [x] **Remove dead Google Apps Script waitlist handler**
  - Removed the `#notify-form` handler, the Apps Script endpoint and the hardcoded `SECRET` from `index.html`

---

## Blockers: fix before any ad spend

- [x] **Install remaining ad pixels** — all three installed behind consent
  - [x] GA4 (`G-30SEFBVGXZ`)
  - [x] LinkedIn Insight Tag (partner `10681985`) + `<noscript>` pixel
  - [x] Meta pixel (`1561336405691270`) with `PageView` + `<noscript>` pixel
  - [x] Conversion on signup: `lintrk('track', { conversion_id: 30047233 })` and `fbq('track', 'Lead')` fire alongside `generate_lead`, from the same guarded `fireWaitlistLead()`. Each call is behind a `typeof` check, so a blocked pixel is silent rather than a crash
  - The Meta `Lead` event was not in the brief; it is the counterpart to the LinkedIn conversion and without it Meta cannot optimise for signups. Remove the three lines if you do not want it

- [x] **CookieFirst consent banner**
  - `consent.js` is the first script in `<head>`, before every tracker, so it can block them. `<meta charset>` stays first (it must land inside the page's first 1024 bytes)
  - All five tracking scripts are `type="text/plain"` with `data-cookiefirst-category`: GA4 → `performance`, LinkedIn and Meta → `advertising`. Verified against the live config: `CookieFirst.consent` returns exactly `necessary` / `functional` / `performance` / `advertising`, so the category names match
  - **Verified blocked before consent**: on a cleared profile with `consent === null`, `lintrk` and `fbq` are `undefined`, all five tags are still inert and no tracker cookie is set. The `gtag` that exists at that point is CookieFirst's own Consent Mode shim (`function gtag(){dataLayer.push(arguments);}`), which only queues into `dataLayer` — not our GA4 snippet
  - Cookie policy lives in its own popup reusing the privacy-policy styling, filled by `<div id="cookiefirst-policy-page">`, reachable from a new footer link. Open/close/Escape/scroll-lock verified
  - **Could not be verified locally:** `acceptAllCategories()` writes the consent cookie but leaves every category `false` and activates nothing — CookieFirst is bound to `tradespace.se` and runs restricted on localhost. Activation must be checked after deploy, see below
  - The two `<noscript>` pixels fire without consent by definition. With JavaScript off the CMP cannot run either, so this is unavoidable; delete them if you want to be strict
  - [x] Conversion event `generate_lead` on successful waitlist signup (fires from the thank-you popup, which replaced the planned `/tack` page — nothing left to move)
  - [ ] Meta pixel
  - [ ] LinkedIn Insight Tag
  - Without the remaining pixels we cannot retarget non-converters or let Meta/LinkedIn optimise delivery.

- [x] **Add meta description and Open Graph tags**
  - [x] `og:title`
  - [x] `og:description`
  - [x] `og:image` (`assets/images/og-image.png`, 1200×630)
  - [x] Meta description
  - Also added canonical, `og:url`, `og:locale`, and Twitter `summary_large_image` so pasted links get a preview.

- [x] **Cut page weight (currently ~5.7 MB)**
  - [x] Map PNG (1.35 MB) no longer loaded. Dots were sampled from the original texture and shipped as `assets/images/hero-map-dots.json` (~90 KB, ~26 KB gzipped). Same positions and intensity, so the canvas looks identical.
  - [x] Dropped italic variable font (579 KB). It was only used on the hidden privacy-popup date.
  - [x] Cleaned `TS_Logo_White_SVG.svg` from 114 KB to 6.1 KB. Wordmark and mark unchanged; dropped Figma stroke-expansion paths and the tagline that was illegible at header size.
  - [x] reCAPTCHA loads only after the visitor types an email.
  - [x] **Images converted to WebP with JPEG fallback** (`<picture>` + `<source type="image/webp">`, so nothing breaks on an old browser). Loaded image weight measured over the wire: **1,724 KB → 605 KB, a 1,119 KB saving (−65 %)**
    - 6 ad cards: 900×506 kept, re-encoded at q78 (107–123 KB → 33–42 KB each)
    - 3 product shots: 1920×1080 → 1600×900 at q80 (331–381 KB → 115–139 KB each). 1600 px still covers the fullscreen viewer's `min(1100px, 95vw)` and the 595 px inline shot on a 2× display
    - `picture { display: contents }` so the wrapper adds no box; verified card and image geometry is pixel-identical to before
    - The fullscreen viewer's `data-image` points at the WebP too
  - Not touched: the Inter Tight variable font is still 581 KB, the largest single asset. Subsetting it to Latin is the next real win

---

## Offer and hero

- [x] **Move the offer into the hero**
  - Hero tagline is now: “Publik beta öppnar 14 september. Säkra din plats och få 20 gratis annonser samt 25% rabatt året ut.”
  - Same offer still repeats in the bottom CTA. Meta/OG descriptions updated so pasted links carry it too.

- [ ] **Make the hero concrete in three seconds**: height done, copy still open
  - “Handla med förtroende” is a brand line, not a product explanation. **Copy is untouched** — the reverted first pass is not back
  - Done: the hero is no longer enormous. `min-height: max(1420px, 155vh)` → `max(1240px, 140vh)`, card padding `clamp(172px, 23vh, 252px) 0 18vh` → `clamp(96px, 13vh, 170px) 0 12vh`, ad feed `top: clamp(720px, 86vh, 880px)` → `clamp(704px, 81vh, 810px)`, bottom fade `min(30vh, 340px)` → `min(15vh, 220px)`
  - Measured at 1280×720: hero 1420 → 1240 px. The visible consent notice added back ~50 px of copy, so the ad feed could not come as far up as the first pass: 16 px of card 1 above the fold at 720 px, ~145 px at 900 px (was 126). The real win is the 180 px of dead hero height
  - The ad feed's `top` must stay below the hero copy. If the copy's height changes, re-measure: at 1280×720 the consent text ends at 687 px and the cards start at 704 px
  - **The ad feed follows the captcha.** The cards are absolutely positioned, so they used to stay put while the captcha reveal pushed the consent text down on top of them. `.hero-section` now carries a `--hero-captcha-space` variable that both the ad feed's `top` and the hero's `min-height` add in; JS keeps it equal to the reveal's real height via a `ResizeObserver`, with re-measurements at 120/600/1400/3000/6000 ms because the reCAPTCHA widget renders asynchronously (a single measurement at click time catches only padding). Verified: cards and hero move down by exactly 116 px and the 17 px clearance to the consent text is preserved. Mobile and tablet are unaffected — there the feed is in normal flow
  - Card 1 still clears the bottom fade and card 2 still peeks under it, so the intended look is unchanged. Mobile and tablet-portrait are untouched (both already set `min-height: auto`)
  - Still open: the copy question — a cold visitor is told “trust”, not what is bought and sold here

- [x] **Shift copy out of future tense**
  - Hero: “Tradespace är…” (was “Vi bygger…”)
  - Network: “Tradespace är en verifierad marknadsplats…”
  - Why: “Det här är plattformen som svensk B2B har saknat.”
  - Success/toast: concrete “Vi hör av oss när betan öppnar den 14 september.” (dropped “revolutionera”)
  - Footer: “* Annonskorten är exempel”: product screenshots are no longer labelled as mockups
  - Kept “Publik beta öppnar 14 september”: that is the launch date, not hedging

- [ ] **Add social proof**: skipped for now
  - Waitlist count in the hero (e.g. “X svenska företag står redan i kön”)
  - And/or founder names, photos, LinkedIn links
  - Nothing on the page currently proves another human wants this

- [ ] **Clarify the 25 % rabatt**: skipped for now
  - There is no pricing anywhere
  - Either publish indicative pricing (“från X kr/mån”) or reframe in absolute terms (“20 gratis annonser och ett halvår utan avgift”)

- [ ] **Add competitive framing and self-identification**: skipped for now
  - Why not Blocket Företag, Alibaba, LinkedIn, or the supplier they already use?
  - “Svenska företag” is 1.2 million companies: say who this is for (tillverkning, logistik, energi, bygg, medtech)

- [x] **Reframe or cut the “1,2 miljoner / 200 nya per dag” section**
  - Reframed as visitor benefit, not investor slide
  - Title: “Alla på ett ställe.”
  - Lead: “1,2 miljoner potentiella kunder och leverantörer. Verifierade, på ett ställe.”
  - Stats: “att göra affärer med” / “att nå varje dag”

- [x] **Add a sticky CTA after the hero**
  - Floating “Säkra er plats” pill, bottom-right on desktop and a full-width bar under 600 px
  - Appears when the hero form leaves the viewport, hides again as the bottom CTA comes into view (`IntersectionObserver` on `#sib-form` and `#beta-cta`), and disappears for good once the visitor has signed up
  - Click scrolls to the bottom form and focuses the field (`preventScroll`, so the smooth scroll is not interrupted). Honours `prefers-reduced-motion`
  - `aria-hidden` and `tabindex="-1"` while hidden, so it is not a focus trap for keyboard or screen readers

---

## Mechanics and polish

- [x] **Hero email field: `type="text"` → `type="email"`**
  - Also set `autocomplete="email"`. Honeypot field left as `type="text"`.

- [x] **Fix the GDPR consent** — the pre-ticked box is gone
  - The old box was worse than pre-ticked: it was also `display: none` inside a `height: 0` block, so consent was collected silently and the visitor never saw the terms
  - Now consent is given by submitting the form, with the notice visible right under the field: “Genom att anmäla er godkänner ni att ta emot nyheter från Tradespace Sweden AB enligt **integritetspolicyn**. Ni kan avbryta när som helst via länken i våra mejl.” The policy word is a real link that opens the existing popup — same wording on both forms
  - `OPT_IN=1` is now a hidden input. Kept inside its `.form__entry` wrapper so Brevo's error handling still finds a target for the field
  - **Landmine found and fixed while testing:** with the checkbox gone, Brevo's client-side validation flagged `sib-optin` as “Detta fält kan inte vara tomt” and blocked every hero submit — and the message rendered inside the `height: 0` block, so it was invisible. Removing `data-required="true"` from the optin wrapper fixes it. Verified: only the captcha blocks now, and Brevo's script reaches the POST with the full payload
  - If a lawyer prefers an explicitly ticked box instead of consent-by-submit, that is a visible unticked checkbox in the hero — one extra click on the primary conversion path

- [x] **Fix “Tradespace Användarvillkor” footer link**
  - Removed. It was a `mailto:` that opened the mail client instead of terms; the terms do not exist yet. Put the link back when they are published

- [x] **Fix “Vill ni annonsera här?” overlapping the ad cards**
  - Mobile and tablet-portrait: the indicator is now in normal flow **below** the ad feed instead of absolutely positioned over it, so it cannot land on an image. This is layout-guaranteed, not tuned
  - Desktop: geometrically unavoidable — three card columns fill the width, so there is no free area to move a centred 220 px block to. Added a soft radial scrim behind it so the text and button read cleanly and the overlap looks deliberate
  - Desktop scrim needs your eyes; see "Verify after deploy"

- [x] **Fix tone inconsistencies**
  - Error: “Det gick inte att skicka. Försök igen.” (was “Din ansökan…”)
  - Success: “Tack! Ni har fått en bekräftelse på er e-post…”
  - Address is *ni* in marketing copy: “Säkra er plats”, “Ange er e-postadress”, “Bygg ert nätverk”
  - Buttons also say “Säkra er plats” instead of “Meddela mig”
  - Privacy policy stays *du*: it addresses the individual as data subject

- [x] **Rewrite CTA label “Meddela mig”**
  - All four CTAs now: “Säkra er plats”

- [x] **Trust scroll section: pacing rebalanced, length deliberately kept**
  - The original goal was ~one viewport of scroll. That was dropped on purpose: the BankID/Bolagsverket approach read as too fast, and shortening the section is exactly what would make it faster still
  - What was actually wasted was the *mapping*, not the height: the last 22 % of the scroll range animated nothing. That dead tail is gone and the reclaimed scroll went into the approach
  - `mergeTravel` now spans 6–70 % of the range instead of 10–50 %. Measured at 1280×720: the logos collide after **403 px of scrolling instead of 288 px (+40 %)**, the resolution beat keeps its ~156 px, and the dead tail is down from 127 px to 17 px
  - `checkProgress` and `logoTint` are expressed in `mergeTravel` space, so they followed the new pacing without being touched
  - Section height is unchanged at `180vh`. The knob is one line: dropping it to `170vh` would still leave the approach ~23 % longer than before, if you want some length back

- [x] **Check declared vs rendered font weights**
  - Visible hero `h1` was already 600; the global `h1 { font-weight: 100 }` was leftover and not what painted
  - Global `h2 { 70 }` and `h3 { 25 }` are invalid/below the font axis (100–900), so browsers could ignore them and fall back to UA bold
  - `@font-face` now uses `format("truetype")` instead of deprecated `truetype-variations`
  - Removed `font-variation-settings: normal` on `body` (it can block the `wght` axis) and set `font-synthesis: none`
  - Global headings and the privacy popup title now use `--font-weight-semibold` (600)

---

## Lighthouse (localhost, 17 Aug 2026)

Same two a11y fails on desktop and phone. Performance 92 → 63 is simulated Slow 4G (150 ms RTT, ~1.6 Mbps) plus 4× CPU, not a new class of bugs. Touch targets pass on phone.

| | Desktop | Phone (simulated) |
|---|---|---|
| Performance | **92** | **63** |
| Accessibility | **93** | **93** |
| Best Practices | **100** | **100** |
| SEO | **100** | **100** |
| FCP | 0.9s | 5.0s |
| LCP | **1.8s** | **8.3s** |
| Speed Index | 0.9s | 5.0s |
| TBT | 0ms | 50ms |
| CLS | 0.001 | 0.008 |
| TTI | 1.8s | 8.3s |

Observed (unthrottled) LCP breakdown is the same on both: TTFB ~2 ms, **element render delay ~2.0 s**. LCP element is the hero **h2**. Phone 8.3 s is that delay plus Slow 4G + 4× CPU on inline JS.

### Fix in the page (these drop Accessibility)

- [x] **Hidden fullscreen overlay still has a focusable close button**
  - `#image-fullscreen` is now `inert` and `aria-hidden="true"` when closed; close button has `tabindex="-1"` until opened

- [x] **White text on `#2093c6` fails WCAG AA (3.47:1, needs 4.5:1)**
  - Filled CTA buttons now use `--ts-primary-600: #0f74a0` (~5.25:1). Brand accent `#2093C6` unchanged on links and highlights

- [x] **Fullscreen image has no `width` / `height`**
  - `#image-fullscreen-img` now has `width="1100"` `height="700"` to match the overlay max size

### Performance: worth knowing, not launch blockers

- Inter Tight variable TTF is **581 KB**, largest first-party asset (~1.4 MB total page)
- Ad card JPGs are 900×503. Phone only flags the two near-viewport cards (Nordkomponent + Klarbäck, ~148 KB). Desktop flagged more (~500 KB if resized / WebP)
- Inline JS on `index.html` is the real phone cost: ~4.8 s script evaluation under 4× CPU (hero animation + map). Desktop hid this
- Brevo `sib-styles.css` is render-blocking (phone: ~600 ms, desktop: ~120 ms)
- Brevo Roboto `font-display` (not ours): ~40 ms
- Most “unused JS” is Bitwarden + GTM + Brevo. Legacy JS (11 KB) is Brevo `main.js`

### Ignore on this run (localhost / extensions / hosting)

- Cache TTL 0, no gzip, HTTP/1.0: Python `localhost:8765`
- Bitwarden long task (210 ms) and unused JS
- No CSP / HSTS / COOP / X-Frame-Options: set on deploy, not in `index.html`
- HTTPS N/A on localhost

---

## If we only do P0

- [x] Connect the bottom form to Brevo
- [x] Capture UTM into Brevo (needs the 5 attributes created in Brevo)
- [x] Thank-you confirmation (popup, not a `/tack` page) and the dead Apps Script handler removed

---

## Verify after deploy

Local verification could not cover these. Only the first item is a prerequisite for ad spend; the rest is ~15 minutes of clicking.

- [ ] **Create the UTM attributes in Brevo** (Settings → Contact attributes, text): `UTM_SOURCE`, `UTM_MEDIUM`, `UTM_CAMPAIGN`, `UTM_CONTENT`, `UTM_TERM`. Do this before the first ad click, then load `?utm_source=test&utm_medium=test` and sign up to confirm the values land on the contact
- [ ] **Real signup through the bottom form on tradespace.se.** A valid submit needs a human-solved reCAPTCHA, so it cannot be automated. Expect: captcha appears when you start typing → tick it → “Säkra er plats” → thank-you popup → contact in Brevo → confirmation email
- [ ] **Same for the hero form**, to confirm the popup replaced the inline panel and that the new hidden `OPT_IN` submits cleanly
- [ ] **Look at three things in a real browser.** `IntersectionObserver`, `requestAnimationFrame` and CSS transitions were all frozen in the local test harness, so these were verified by code and geometry only: the sticky CTA's show/hide trigger, the bottom captcha's reveal animation, and the scrim behind “Vill ni annonsera här?” on desktop
- [ ] **Confirm the pixels actually activate after consent.** This is the one thing local testing could not cover, and getting it wrong means spending on ads with no tracking. On tradespace.se: accept all in the banner, then in the console check `typeof window.lintrk` and `typeof window.fbq` — both must say `function` — and watch the network tab for `snap.licdn.com` and `connect.facebook.net`. Then sign up and confirm the conversion lands in LinkedIn Campaign Manager and Meta Events Manager. If they stay blocked, the fix is either to register the scripts in the CookieFirst dashboard or to drop the `type="text/plain"` gating and rely on CookieFirst's automatic blocking
- [ ] **Point the CookieFirst banner's cookie-policy link at the landing page.** The policy is a popup here, not a URL. Either give it a real page or set the banner's link to the site the policy already lives on
- [ ] **Check Brevo for junk contacts** from endpoint probing on 17 Aug: `not-a-valid-email` and `tradespace-form-test@example.com`. Every browser request was rejected with `400`, and the `curl` probes that answered `{"success":true}` never reached field validation (identical response for valid and invalid input alike), so most likely nothing was created — worth one look
- [ ] **Consider double opt-in in Brevo.** The success copy promises a confirmation email; confirm that is actually switched on
- [ ] **Unused files still ship in the deploy: ~31 MB.** Not a page-speed problem — nothing loads them — but they bloat the repo and every deploy. The 8 source PNGs behind the ad cards (~14.8 MB), `tradespace video.mov` (14 MB, only named inside a commented-out block), `tradespace-dotted-world-map.png` (1.3 MB), the italic font (0.57 MB), `ExampleScreenshot.PNG`, `TS_Logo_SVG.svg`, `logo.svg` and 2 unused card JPGs. Most are masters worth keeping — but out of the deployed folder, not in it. Say the word and I will move them
- [ ] **Note on the hero's inline error panel.** Brevo's big `#error-message` panel adds ~76 px inside the hero when it appears, which now reaches into the ad cards. It does *not* appear for the common cases (missing captcha and empty fields use small inline labels and push nothing), so it was left alone rather than repositioned blind

---

## Keep: do not break these

These are working. Protect them while changing everything else.

- [x] BankID + Bolagsverket scroll animation (clearest differentiator)
- [x] Example ad feed (concrete, credible Swedish companies and needs)
- [x] Product rows, especially unit pricing (vikt, tid, längd)
- [x] Dark navy visual system (serious, trust-led)
- [x] Real product screenshots (underused today, but they exist)
- [x] Honest “* Exempelbilder” labelling
- [x] Org number, Lund address, real GDPR policy in footer
