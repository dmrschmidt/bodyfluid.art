# TODO

The first production pass — copy de-faked (Lisbon → Berlin, water → fluid),
brand swapped to Bodyfluid/Art, all five plate masters wired in from
`photos/` via the `build_photos.py` pipeline, prints section wired for
Stripe via `data-*` attributes on all five plates, ported `stripe_sync.py`,
and the four core legal pages added — is done. What remains is the punch
list to take the site from "first pass" to "ready to sell". Ordered
roughly by impact / risk.

## 0. Real material — blockers for actually selling

- [x] **Photograph the five plates.** Masters live in `photos/` as
      `plate-I.jpg`…`plate-V.jpg`. `build_photos.py` (ported from
      photo-portfolio) builds optimised progressive JPEG (q≈82) + WebP
      (q≈80) variants into `photos-web/`, capped at 1500 px on the long
      edge, with intrinsic dimensions in `photos-web/manifest.json`.
      `index.html` serves them via `<picture>` everywhere — hero feature,
      the five work cards, and the five prints articles.
- [ ] **Confirm pricing.** Currently set at a flat €900 for all five plates.
      Photo-portfolio uses a similar anchor; revise per format / edition
      once decided and re-run `stripe_sync.py`.
- [x] **Five plates listed in prints.** Plate I (Periwinkle) and Plate IV
      (Mint) are now in `<section id="prints">` alongside II, III, V. All
      five carry `data-plate`/`data-img`/`data-price`/`data-remaining`
      attrs and matching entries in `DESCRIPTIONS` in `stripe_sync.py`.
- [x] **Run `stripe_sync.py`** — all five plates synced, each `<article
      class="ed">` carries a `data-stripe-url`, and the inline Acquire
      buttons are rewritten on page load to point at Stripe. The modal
      Acquire button reads the same source and falls back to a mailto
      enquiry if `data-stripe-url` is missing.

## 1. Legal — required for B2C sales from Germany

- [x] **PAngV price labels** — each plate card now shows "incl. VAT ·
      shipping at checkout" under the price in the prints grid.
- [x] **ODR notice in footer** — links to `ec.europa.eu/consumers/odr` plus a
      VSBG sentence are in the index footer and on every legal page.
- [x] **AGB / Terms of Sale page** — `terms.html` is in place (scope,
      conclusion of contract, prices, payment, delivery, passing of risk,
      retention of title, warranty, withdrawal, liability, applicable law).
- [x] **Widerrufsbelehrung + Muster-Widerrufsformular** — `withdrawal.html`
      includes the formal cancellation notice and the standard withdrawal
      form text.
- [x] **Shipping & delivery page** — `shipping.html` lists carriers, lead
      times, insurance handling, and customs notes by region.
- [x] **Privacy coverage** — `privacy.html` names the third parties actually
      involved (Bunny Fonts, Stripe). Controller is now identified by name
      and address in § 1. One placeholder remains: the **hoster** under § 4.
- [x] **Impressum.** `imprint.html` is served locally with both English and
      German sections under § 5 TMG (provider, responsibility, liability for
      content / links, copyright, ODR). All four other legal pages and the
      index footer link to it.
- [ ] **Self-host CSS** — `index.html` ships ~24 KB of inline CSS, which is
      fine. But the four legal pages each embed the same chrome inline.
      Either accept the duplication (zero build, easy diffs) or factor a
      shared `styles.css` like photo-portfolio does. Tailwind v4 is not
      pulled in at all here, so no third-country transfer to jsDelivr — good.

## 2. Accessibility — BFSG (German Accessibility Act, in force since June 2025)

- [x] **`prefers-reduced-motion`** — pauses the ticker, disables the reveal
      animations, and short-circuits transitions.
- [x] **Skip-to-content link** — keyboard-only `.skip-link` jumps to the
      work grid.
- [x] **Keyboard activation on plate cards** — every plate card (work,
      prints, and the hero feature) now has `role="button"`,
      `tabindex="0"`, `aria-haspopup="dialog"`, and Enter/Space handlers
      that open the plate modal. Inline `<a>` and `<button>` clicks inside
      the card pass through (the visible Acquire goes straight to Stripe).
- [x] **Modal a11y** — `#plate-modal` carries `role="dialog"`,
      `aria-modal="true"`, `aria-labelledby="modal-title"`; focus moves to
      the close button on open and returns to the triggering card on
      close; Tab/Shift-Tab is trapped inside the dialog; ESC and
      backdrop-click both close.
- [ ] **Color contrast audit** — `--mute` (#5a5a55) on `--paper` (#ededea)
      and white-on-coral combinations on the index need a contrast check at
      small sizes.
- [ ] **Decorative ticker** — currently marked `aria-hidden="true"` so SRs
      skip it. Confirm the marquee adds no information not present elsewhere.

## 3. Premium — what paying buyers notice

- [ ] **Schema.org JSON-LD** — `Product` per plate (price, availability,
      image), `Person` for the artist, `WebSite`. Drives rich results in
      search.
- [ ] **Image pipeline polish** — `build_photos.py` currently emits a
      single 1500 px JPEG + WebP per master. For phones, add an 800w
      variant and a `srcset` so smaller devices don't pull the desktop
      size; AVIF on top of WebP recovers another ~25%.
- [ ] **Designed OG share card** — 1200 × 630 with the wordmark.
      `og:image` is unset at the moment.
- [ ] **Certificate of authenticity sample** — image or PDF a buyer can see
      before committing €900.
- [ ] **Confirm `data-year` values** — every plate currently reads `2026`,
      matching the masters' provided dates. Overwrite if the actual shoot
      year differs (EXIF on the masters in `photos/` is the source of
      truth).
- [ ] **Journal pages** — `index.html` lists five journal teasers under
      `<section id="journal">`, but the entries link nowhere. Either build
      a `journal.html` like photo-portfolio or remove the section until
      there is real content.
- [ ] **Confirm Berlin address** — `index.html` contact panel and the
      Studio panel say "Neukölln, Berlin" without a street. If a real studio
      address exists, add it; otherwise this is the safer placeholder.

## 4. Lower priority

- [ ] `twitter:site` / `twitter:creator` handles in the social meta.
- [ ] Stripe success / cancel pages styled in the studio voice. Includes
      wiring `after_completion[type]=redirect` +
      `after_completion[redirect][url]` into the `payment_links` call in
      `stripe_sync.py`.
- [ ] Multi-currency display for non-EU buyers (USD / GBP indicative).
- [ ] Styled 404 page.
- [ ] `sitemap.xml` + `robots.txt` (photo-portfolio has both; mirror them
      here once the legal pages and journal are settled).
- [ ] Pre-commit hook — photo-portfolio's `.githooks/pre-commit` runs
      `build_photos.py` whenever a photo is staged. Mirror it here so the
      `photos-web/` build can't drift from `photos/`.
