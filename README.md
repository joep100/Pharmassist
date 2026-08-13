# Pharmassist site

Four static pages. No build step, no framework — open `index.html` in a browser
and it works.

```
index.html          For pharmacies (home)
how-it-works.html   Operational detail
coverage.html       The 12km area + address checker
for-patients.html   Patient-facing
config.js           ← the only file you normally need to edit
checker.js          Address checker logic (shared by all pages)
styles.css          All styling
```

---

## Before this goes live

I wrote real copy rather than filler, which means some of it is a claim about
how you operate. **Check each of these and change or delete it.** Marketing
copy in a regulated space is not the place for a hopeful guess.

**Contact details** — phone and email are `01 000 0000` and
`hello@pharmassist.ie` throughout. They appear in `config.js` *and* hardcoded in
the footer and a few links on each page. Search-and-replace across all four
files.

**The 1pm cutoff** — invented. `how-it-works.html`, "Same-day cutoff".

**Proof of delivery** — I claim timestamped POD with recipient name and GPS
point, returned the same day. If you don't do the GPS part, cut it.

**Scanning bags in and out** — I claim a scan at collection creating a handover
record. If collections are currently a signature on a docket, say that instead.

**"If nobody's home we don't leave it"** — I made this the rule on two pages.
It's the right rule for medicines, but confirm it's yours.

**Flat rate per delivery** — stated on `how-it-works.html`. If pricing is
banded, that paragraph needs rewriting.

**Fridge lines and controlled drugs** — I deliberately wrote these as "talk to
us first, we may say no" rather than claiming a capability. That's the safe
version. Only strengthen it if you genuinely have the cold chain and the CD
procedures to back it up.

**"No setup cost"** — on `for-patients.html`, in the nudge section.

---

## Turning on the two forms

Both are wired but post nowhere until you fill in `config.js`:

```js
partnerFormUrl: "https://...",   // the pharmacy enquiry form
waitlistUrl:    "https://...",   // out-of-area email capture
```

Anything that accepts a JSON POST works — Formspree, Basin, or a Vercel function.
Until you set them, the forms still show a thank-you so nobody hits a dead end,
but nothing is sent. Don't ship it that way.

---

## The address checker

Your area is a 12km circle around O'Connell Bridge. A routing key (the first
three characters of an Eircode) is an *area*, not a point, so:

- **18 routing keys sit entirely inside** the circle → instant yes, free, from
  three characters typed.
- **Everything beyond the borderline list sits entirely outside** → instant no.
- **8 routing keys are cut through by the circle** (D13, D15, D18, D22, D24,
  A96, K67, K78) → these need the full Eircode and a real coordinate lookup.

Roughly 7 in 10 lookups never touch an API.

### Switching on the exact lookup

Put a Google Maps API key in `config.js`:

```js
googleApiKey: "AIza...",
```

Enable **Maps JavaScript API** and **Geocoding API** on the key. Google has
carried Eircode data since 2017, so Irish postcodes geocode accurately.

Without a key, borderline addresses are told to phone you. That's a working
fallback, not a broken state — but it's a worse experience for a chunk of
Tallaght, Blanchardstown and Clondalkin, which is a lot of people.

### Move the key server-side

A key in a web page can be scraped even with referrer restrictions. Since your
other sites are on Vercel, the clean fix is `api/geocode.js` holding the key,
with the browser calling your own endpoint. That also lets you cache each
Eircode you've already resolved, so repeat lookups cost nothing.

At minimum, restrict the key to your domain in Google Cloud Console:
*Application restrictions → HTTP referrers*.

---

## Deploying

Same route as Cyclone Archive — push to GitHub, import into Vercel, no build
command needed. It's plain static files.

Two things missing for launch: a real `favicon.ico`, and the logo PNG if you'd
rather use it than the type-set wordmark. The wordmark is currently set live in
Inter with `Pharm` in `#2F8F1C` and `assist` in `#708090`, with the cross drawn
as SVG — sharp at any size and nothing to load. To swap in the file, replace the
`<a class="mark">` block with:

```html
<a href="index.html"><img src="pharmassist.png" alt="Pharmassist" width="200"></a>
```
