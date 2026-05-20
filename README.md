# Bodyfluid / Art

Static one-page catalogue for the editions of Bodyfluid.
Deployed at <https://bodyfluid.art/>.

## Layout

```
index.html              — single-page catalogue (hero, work, statement,
                          studio, prints, journal teasers, contact)
terms.html              — AGB / conditions of sale
withdrawal.html         — Widerrufsbelehrung + Muster-Widerrufsformular
shipping.html           — shipping & delivery
privacy.html            — privacy policy
originals/              — raw plate masters straight from camera
photos/                 — six plate masters (plate-I.jpg … plate-V.jpg)
photos-web/             — built artifact: optimised progressive JPEG +
                          WebP variants of each plate (capped at 1500 px
                          on the long edge), plus manifest.json with
                          intrinsic dimensions for CLS-safe markup.
build_photos.py         — rebuilds photos-web/ from photos/ (idempotent)
stripe_sync.py          — pushes available plates to Stripe (Products →
                          Prices → Payment Links) and writes
                          `data-stripe-url` back into `index.html`.
TODO.md                 — running production gaps
.gitignore
```

The catalogue is rendered from `<article ...>` attributes on each print card
under `<section id="prints">` (`data-plate`, `data-title`, `data-img`,
`data-year`, `data-edition`, `data-remaining`, `data-price`, `data-desc`).
Everything else (hero, work grid, journal teasers) is static markup.

## Setup

One build step:

```sh
python3 build_photos.py   # photos/ → photos-web/
```

After that, `index.html` references `photos-web/` directly. CSS is inline
in each HTML file; fonts come from Bunny Fonts. Open `index.html` to
preview locally.

`photos-web/` is committed on purpose so the static host has nothing to
build at deploy time.

## Stripe sync

`stripe_sync.py` is ported verbatim from photo-portfolio and is idempotent
and ledger-backed:

```sh
echo 'STRIPE_SECRET_KEY=sk_test_…' > .env   # sk_live_… in live mode
python3 stripe_sync.py
```

For each available plate it:

1. Uploads the plate image to Stripe Files (cached in the ledger).
2. Creates a Stripe Product + Price + Payment Link the first time it sees
   the plate, or rotates the Price + Link if the EUR amount changed.
3. Deactivates Price + Link when `data-remaining="0"`.
4. Rewrites the matching `<article>` with `data-stripe-url`, so the
   `Acquire` button deep-links to Stripe. Until `data-stripe-url` is
   present the button falls back to a `mailto:` enquiry.

Ledgers are scoped per mode:

- `.stripe_products.test.json` — test keys
- `.stripe_products.live.json` — live keys (sync prompts for confirmation)

Both ledgers are gitignored. The shipping rate and country allow-list are
shared with photo-portfolio (`SHIPPING_RATE`, `ALLOWED_COUNTRIES` in
`stripe_sync.py`).

Per-plate Stripe descriptions live in `DESCRIPTIONS` in `stripe_sync.py`,
keyed by the same image path used in the HTML's `data-img`. Re-running with
a changed description does NOT update Stripe — to force a refresh, delete
the plate's entry from the active ledger.

## Conventions

- Print dimensions are written as `long × short` in cm and reflect the
  source image's actual aspect ratio at a 70 cm long edge.
- Pigment giclée on Hahnemühle Photo Rag Baryta, 315 gsm. Hand-numbered,
  signed in graphite, shipped flat with a letterpress certificate of
  authenticity. Same paper, sizes, carrier, and country allow-list as
  photo-portfolio.

## Known production gaps

See [`TODO.md`](TODO.md).
