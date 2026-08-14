# Pharmassist site

Four static pages. No build step, no framework. Open `index.html` in a browser
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

**Phone number.** Still a placeholder: `01 000 0000` throughout. It appears in
`config.js` and also hardcoded in the footer of all four pages, in the checker's
out-of-area message, and in a `tel:` link on `for-patients.html`. Search and
replace both the display form and the `tel:+35310000000` link form.

Email is settled as `hello@pharmassist.ie` and consistent across all four pages.

**The 1pm cutoff:** invented. `how-it-works.html`, "Same-day cutoff".

**Proof of delivery:** I claim timestamped POD with recipient name and GPS
point, returned the same day. If you don't do the GPS part, cut it.

**Scanning bags in and out:** I claim a scan at collection creating a handover
record. If collections are currently a signature on a docket, say that instead.

**"If nobody's home we don't leave it":** I made this the rule on two pages.
It's the right rule for medicines, but confirm it's yours.

**Flat rate per delivery:** stated on `how-it-works.html`. If pricing is
banded, that paragraph needs rewriting.

**Fridge lines and controlled drugs:** I deliberately wrote these as "talk to
us first, we may say no" rather than claiming a capability. That's the safe
version. Only strengthen it if you genuinely have the cold chain and the CD
procedures to back it up.

**"No setup cost":** on `for-patients.html`, in the nudge section.

---

## The NSAI certification marks

They appear on `index.html` and `how-it-works.html`. Three things to settle
before this is live:

**Certificate holder: done.** Both certificates are held by Flexi Logistics
Services Ltd., and the site says so in three places. `index.html` reads
*"Pharmassist is a trading name of Flexi Logistics Services Ltd., which holds
both certificates"*, `how-it-works.html` makes the same point, and the footer on
all four pages names the company with the copyright in its name rather than the
brand's.

One thing still open here: **the CRO number.** Adding it beside the company name
lets a pharmacist finish their due-diligence check in one go instead of guessing
which Flexi Logistics they've found on the register. Send it over and I'll put it
in the footer.

**Business name disclosure.** If Pharmassist is a registered business name of
Flexi Logistics Services Ltd., Irish rules generally require the underlying
company name on business documents, and the e-commerce regulations require clear
provider identification on the site. The footer now covers the name, but confirm
the exact wording and whether a registered office address is needed too. Worth
thirty seconds with whoever does your filings rather than taking my word for it.

**Check the certified scope.** ISO 9001 is certified against a defined scope of
activities. If Flexi Logistics Services Ltd.' certified scope covers warehousing and general
courier work but not pharmacy delivery, the copy claiming it covers "the way we
handle, track and hand over your bags" is a stretch. Read the certificate.

**Confirm you may use the marks this way.** NSAI sets conditions on how their
mark is displayed. I've placed both unaltered, at their own aspect ratio, with
no crops, tints or added borders, which is the conservative reading, but the
conditions attached to your certificate are the authority, not me.

### About the image files

The originals were **CMYK JPEGs with a 544KB embedded print colour profile**
each, a print format browsers handle inconsistently. I converted both to
greyscale RGB PNGs and stripped the profile: 431KB became 23KB and 13.6KB.

The 9001 original was only **200px wide**, against 831px for the 14001. I've
upscaled it to match so the pair look consistent, but it is doing more work than
the pixels really support and will be a touch softer on a high-resolution
screen. Worth asking NSAI for the EPS or SVG version of both, and then they're
sharp at any size and a fraction of the weight again.

---

## Turning on the two forms

Both are wired but post nowhere until you fill in `config.js`:

```js
partnerFormUrl: "https://...",   // the pharmacy enquiry form
waitlistUrl:    "https://...",   // out-of-area email capture
```

Anything that accepts a JSON POST works: Formspree, Basin, or a Vercel function.
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
fallback, not a broken state, but it's a worse experience for a chunk of
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

Same route as Cyclone Archive: push to GitHub, import into Vercel, no build
command needed. It's plain static files.

Two things missing for launch: a real `favicon.ico`, and the logo PNG if you'd
rather use it than the type-set wordmark.

The wordmark is currently set live in Inter, all caps per the current logo, with
`PHARM` in `#2F8F1C` and `ASSIST` in `#708090`, and the cross drawn as SVG. Sharp
at any size and nothing to load. The casing is done with CSS `text-transform`
rather than by typing `PHARMASSIST` into the HTML, which matters for
accessibility: screen readers announce it as the word "Pharmassist" instead of
spelling out eleven letters.

To swap in the image file instead, replace the `<a class="mark">` block with:

```html
<a href="index.html"><img src="pharmassist.png" alt="Pharmassist" width="200"></a>
```
