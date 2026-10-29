# Walmart Receipt Generator

A single-file, dependency-free web app that builds Walmart-style receipts in the browser — in-store register slips and walmart.com order confirmations — with live product import, real Code 128 barcodes, real QR codes, and full manual control over every field.

No build step. No framework. No server. Open the HTML file and it runs.


---

## Why

Real receipts are useful reference material: expense reports, return-window reconstruction, prop and set dressing, design mockups, teaching material, or just building a plausible paper trail for a project. Existing generators are usually paywalled, watermarked, or produce a generic blob of text that doesn't resemble anything.

This one recreates an actual Walmart register slip layout line for line — the survey header, the `ST#/OP#/TE#/TR#` register block, the `TC#` barcode, the EMV approval block, the customer-copy footer — and lets you drive every value.

## Features

### Two templates

- **In-store register receipt** — thermal paper styling, torn edge, dot-matrix-style type, register and terminal metadata, itemized lines with UPCs, tax and tender block, full card authorization block, barcode, and the QR promo footer.
- **Walmart.com order confirmation** — blue header, order number, delivery estimate, shipping address, itemized product list with thumbnails, totals, and payment line.

### Import from a URL

Paste any `walmart.com/ip/...` product URL and the app pulls the title, image, and current price and drops them onto a new line. Three CORS proxies are tried in sequence. If all fail (Walmart bot protection is aggressive), a paste-source fallback accepts the raw page HTML or the text from `r.jina.ai/<url>` and runs the same parser against it.

The parser reads `og:title`, `og:image`, `product:price:amount`, and JSON-LD `offers.price` from HTML, or markdown headings, image links, and price patterns from jina output.

### Real barcode, real QR

- The `TC#` field is encoded as a genuine **Code 128** symbol — subset C, with correct modulo-103 check character and stop pattern, drawn to canvas at 4× resolution with proper quiet zones. It's scannable, not decorative.
- The footer QR encodes a Walmart+ URL carrying the store number and is a real, scannable code.
- Both regenerate live as you edit.

### Full field control

Every printed value is editable: store number, phone, street, city, state, ZIP, manager name, date, time, `OP#`, `TE#`, `TR#`, feedback ID, `TC#`, copy type, item names, UPCs, quantities, prices, tax rate, tender amount, card issuer, network, last four, approval number, reference number, transaction ID, validation code, terminal number, AID, and AAC.

### Randomize

One click regenerates the transaction number, operator, terminal, approval, reference, transaction ID, validation, AID, AAC, feedback ID, `TC#`, card last four, store number, and stamps the current date and time — all in the correct digit shapes for each field.

### Export

- **Download PNG** at 3× scale.
- **Print** with a dedicated print stylesheet that hides the control panel and prints the paper at 1:1.


### Requirements

A modern browser with ES2020 support (Chrome, Edge, Firefox, Safari). The app loads three CDN libraries — `html2canvas`, `qrcodejs`, and the Courier Prime webfont — so an internet connection is needed on first load. Everything else is inline.

## Usage

1. Pick a template from the **Template** dropdown.
2. Either paste a product URL and hit **Fetch item**, or fill the fields in manually.
3. Set the store block. Use a real Supercenter address and phone for the store number you pick.
4. Set the tax rate for that store's actual state and county.
5. Add item lines. Keep names short and abbreviated — 13 to 16 characters reads most authentically.
6. Fill the card block. `Randomize transaction` will do most of it.
7. Export as PNG or print.

### Field notes

| Field | Notes |
|---|---|
| Store number | Printed zero-padded to five digits |
| UPC | 12 digits, printed in the item column |
| Qty / Price | Multiplied for the line amount; qty accepts decimals for weighed goods |
| Tax rate | Applied to subtotal; set `0` for tax-free states |
| Tendered | Blank or `0` defaults to the exact total, giving zero change |
| TC# | Space-separated 20 digits; feeds the barcode |
| Copy type | Customer, merchant, or store copy |

### Tips for realism

- Price endings of `.48`, `.97`, `.98` are Walmart's signature — avoid round numbers.
- Keep totals under roughly $150 so the tender amount stays plausible.
- Use one card issuer per receipt and keep the network consistent with it.
- Backdate the date and time together; a 3 a.m. grocery run with eight items looks off.
- Abbreviate item names the way a cashier's short description field would.

### Printing

For the closest result, print on a 3-inch thermal roll. Otherwise use **Save as PDF** with margins set to zero and print on plain paper, or print on white paper and trim.

## Project structure

```
index.html      # everything — markup, styles, logic, data
```

That's it. It's one file on purpose: easy to host on GitHub Pages, easy to fork, easy to drop on a USB stick.

## How the receipt is built

The layout mirrors a real Walmart slip in order: survey header with feedback ID, centered wordmark, store phone and manager line, address, register metadata, item count, `TC#` and barcode, itemized lines in a four-column grid (name / UPC / amount / `X`), right-aligned subtotal and tax, tender line, the full EMV authorization block, timestamp, change due, the "Low Prices You Can Trust" line, a second timestamp, the copy marker, and the centered QR promo.

Paper styling uses an off-white base, a repeating-linear-gradient overlay for thermal banding, Courier Prime at tight tracking for the dot-matrix feel, and a `clip-path` polygon generated in JavaScript for the torn edge.

## Known limitations

- **Symbology.** The barcode is valid Code 128. Walmart's POS may encode that field differently; it is visually and structurally correct but will not resolve to the original transaction.
- **Font.** Courier Prime is a typewriter face, not a true dot-matrix one. A pixel or dot-matrix webfont would be closer — swap the `font-family` on `.receipt` to change it.
- **URL import is best-effort.** Walmart's bot protection blocks a meaningful share of requests. The paste-source fallback exists for exactly that reason.
- **Prices are not validated.** The importer takes the first price it finds, which may be a comparison price, a rollback, or a unit price. Check it.
- **No persistence.** Refreshing the page resets to demo data. Nothing is written to storage.
- **PNG export** renders through `html2canvas`, which does not capture `clip-path` exactly — the torn edge is squared off in exports. Print preserves it.

## Customizing

- **Colors and type** — the `.receipt` rules near the top of the stylesheet.
- **Barcode height** — the `52` in `drawBarcode` and the matching `height` on `.barcode canvas`.
- **Paper width** — `.receipt.store` (340px) and the matching value in the `tear()` call.
- **Demo data** — the `base()` function returns the seed object; edit it to change what loads on start.
- **Adding a template** — write a `yourTemplateHTML()` function returning markup, add a case in `renderPaper()`, and add an option to the `#template` select.

## Contributing

Issues and pull requests welcome, particularly:

- Closer font matches for thermal print
- Additional store presets
- Fixes for proxy endpoints as they rot
- Retailer-specific barcode symbologies

## Disclaimer

This is an independent recreation for convenience and prop use. It is not produced, endorsed, or verified by Walmart Inc. Receipts generated here have no cryptographic or backend link to a real transaction — a store associate scanning the barcode will not find the sale. Use it as supporting documentation, not as proof.

## License

MIT. Do what you like with it.
