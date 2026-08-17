# Tradespace launch checklist

Review of the website as a cold visitor from outreach or ads. Launch: **14 September**.

How to use: go through item by item and check them off. **P0 first.** Then blockers before ad spend. Everything else is ordered by impact, not by how it appears on the page.

---

## P0: waitlist must convert

Do these before anything else. A filled form that does not land in Brevo, or a signup with no thank-you, is a lost lead.

- [ ] **Connect the bottom waitlist form to Brevo**
  - Hero `#sib-form` is the real Brevo form: POSTs to `cbebf07e.sibforms.com`, fields `EMAIL`, `OPT_IN`, hidden `locale=sv`, honeypot `email_address_check`, plus reCAPTCHA
  - Bottom `#beta-cta-form` is a fake: `action="#sib-form"` GET, field named `email` (not `EMAIL`). Submit copies the value into the hero field, scrolls up, and focuses it. No POST, no consent, no captcha
  - Visitor who fills the bottom form has not joined the list until they submit the hero form again
  - Fix: submit the bottom email through the same Brevo form (hidden clone or JS POST), or replace the bottom form with a button that only scrolls to the hero

- [ ] **Capture UTM into hidden Brevo fields**
  - `utm_source` / `utm_medium` / `utm_campaign` (and `utm_content` / `utm_term` if present) are not sent today
  - Without this, waitlist contacts cannot be attributed to ads or outreach

- [ ] **Add a `/tack` thank-you page**
  - Redirect here after a successful Brevo submit (replace the inline success panel as the primary confirmation)
  - Fire `generate_lead` on this page (today it fires when the inline Brevo success panel appears)
  - Second ask after conversion, not before: industry / buy-vs-sell
  - “Dela med en kollega” moment
  - Must not add extra fields to the hero form

- [ ] **Remove dead Google Apps Script waitlist handler**
  - JS still posts to a Apps Script endpoint if `#notify-form` exists
  - That form is gone; only Brevo `#sib-form` is live. Dead fetch plus a hardcoded secret in `index.html`

---

## Blockers: fix before any ad spend

- [ ] **Install remaining ad pixels**
  - [x] GA4 (`G-30SEFBVGXZ`)
  - [x] Conversion event `generate_lead` on successful waitlist signup (move this to `/tack` when that page exists)
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

---

## Offer and hero

- [x] **Move the offer into the hero**
  - Hero tagline is now: “Publik beta öppnar 14 september. Säkra din plats och få 20 gratis annonser samt 25% rabatt året ut.”
  - Same offer still repeats in the bottom CTA. Meta/OG descriptions updated so pasted links carry it too.

- [ ] **Make the hero concrete in three seconds**: tried, reverted
  - “Handla med förtroende” is a brand line, not a product explanation
  - Cold visitor needs: what do I buy/sell here, and who else is on it
  - Ad cards already answer this, but they sit below the fold on a 1440×900 laptop
  - Shorten the hero (`min-height: max(1420px, 155vh)` is enormous) and/or pull 1-2 ad cards above the fold
  - First pass (concrete subhead + pulled-up cards) was reverted. Leave as-is unless we revisit.

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

- [ ] **Add a sticky CTA after the hero**
  - ~4,600 px between hero form and bottom CTA with no visible call to action
  - Nav “Meddela mig” is only visible at the top, where the form is already in view

---

## Mechanics and polish

- [x] **Hero email field: `type="text"` → `type="email"`**
  - Also set `autocomplete="email"`. Honeypot field left as `type="text"`.

- [ ] **Un-check the GDPR consent box**
  - `<input id="OPT_IN" required checked>` is not valid consent (Planet49 / GDPR)
  - Credibility problem for a trust/verification product
  - Also: consent text mentions integritetspolicyn without linking to it

- [ ] **Fix “Tradespace Användarvillkor” footer link**
  - Currently a `mailto:`: Mail.app opens instead of terms
  - Publish the terms, or remove the link until they exist

- [ ] **Fix “Vill ni annonsera här?” overlapping the ad cards**
  - Desktop: sits on Voltverk card text
  - Mobile: lands in the middle of the Klarbäck image

- [x] **Fix tone inconsistencies**
  - Error: “Det gick inte att skicka. Försök igen.” (was “Din ansökan…”)
  - Success: “Tack! Ni har fått en bekräftelse på er e-post…”
  - Address is *ni* in marketing copy: “Säkra er plats”, “Ange er e-postadress”, “Bygg ert nätverk”
  - Buttons also say “Säkra er plats” instead of “Meddela mig”
  - Privacy policy stays *du*: it addresses the individual as data subject

- [x] **Rewrite CTA label “Meddela mig”**
  - All four CTAs now: “Säkra er plats”

- [ ] **Tighten the trust scroll section**
  - 1,620 px of scroll-jacking for one sentence
  - Best asset on the page: keep it, but aim for ~one viewport of scroll

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

- [ ] Connect the bottom form to Brevo
- [ ] Capture UTM into Brevo
- [ ] Add `/tack` and remove the dead Apps Script handler

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
