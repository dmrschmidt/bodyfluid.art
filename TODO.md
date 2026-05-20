# TODO

The first production pass — copy de-faked (Lisbon → Berlin, water → fluid),
brand swapped to Bodyfluid/Art, all five plate masters wired in from
`photos/` via the `build_photos.py` pipeline, prints section wired for
Stripe via `data-*` attributes on all five plates, ported `stripe_sync.py`,
and the four core legal pages added — is done. What remains is the punch
list to take the site from "first pass" to "ready to sell". Ordered
roughly by impact / risk.

## 0. Real material — blockers for actually selling

- [ ] **Confirm pricing.** Currently set at a flat €900 for all five plates.
      Photo-portfolio uses a similar anchor; revise per format / edition
      once decided and re-run `stripe_sync.py`.
- [ ] **Re-run `stripe_sync.py` for Plate VI** — Plate VI · Vermilion has
      `DESCRIPTIONS` entry and a full `<article data-*>` in the prints
      section, but no `data-stripe-url` yet. Running sync (in live mode)
      will upload the image, create the Product + Price + Payment Link,
      and write `data-stripe-url` back into `index.html`.

## 1. Legal — required for B2C sales from Germany

- [ ] **Self-host CSS** — `index.html` ships ~24 KB of inline CSS, which is
      fine. But the four legal pages each embed the same chrome inline.
      Either accept the duplication (zero build, easy diffs) or factor a
      shared `styles.css` like photo-portfolio does. Tailwind v4 is not
      pulled in at all here, so no third-country transfer to jsDelivr — good.

## 2. Accessibility — BFSG (German Accessibility Act, in force since June 2025)

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
- [ ] **Certificate of authenticity sample** — image or PDF a buyer can see
      before committing €900.

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
