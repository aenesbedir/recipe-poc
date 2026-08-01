# recipe-poc

> **Answered: no.** Cookidoo enforces a source allowlist. Importing from a
> third-party domain is rejected before any markup is read:
> *"Bu web sitesinden içeri aktarım şu anda onaylanmış değil. Yalnızca Cookidoo veya
> Vorwerk topluluk sitelerinden aktarıma izin verilmektedir."*
> Full results and product consequences in [`RESULTS.md`](RESULTS.md).

Throwaway test pages for one question:

> Can Cookidoo's "Created Recipes → Import recipe" flow import a recipe from a
> third-party website, and if so, how much of it survives?

This is not a product. It exists to kill or confirm the riskiest assumption behind a
Turkish Thermomix recipe-sharing platform before any of that platform gets built.

Background research and the reasoning behind each variant: [`NOTES.md`](NOTES.md).

## The pages

Live at <https://aenesbedir.github.io/recipe-poc/>

| Variant | Markup | Question it answers |
| --- | --- | --- |
| [`v1/b-microdata.html`](v1/b-microdata.html) | schema.org microdata | Mirrors the exact structure Vorwerk's own Recipe Community uses. Best case. **Test this first.** |
| [`v1/a-jsonld.html`](v1/a-jsonld.html) | schema.org JSON-LD | Is standard, widely-used recipe markup enough? |
| [`v1/c-plain.html`](v1/c-plain.html) | none | Control. If this imports, the parser reads prose and markup barely matters. |

The recipe content is byte-for-byte identical across all three. Only the markup
differs, so a failure can be attributed to format rather than content.

The recipe itself is original. No third-party recipe content is reproduced anywhere
in this repo; variant B mirrors Recipe Community's markup *structure* only.

## Test protocol

Prerequisite: an active Cookidoo subscription on the Turkish market
(`cookidoo.com.tr`). Without an active subscription the Import button does not
appear and the test cannot start. Verify this first — it is the cheapest possible
way to fail early.

### Step 0 — confirm the pages are actually live

Open each URL in a browser before pasting it anywhere. GitHub Pages can take a few
minutes on first deploy. A page that is not yet live produces a failed import that
looks exactly like a rejected format.

### Step 1 — the domain question

Log in to `cookidoo.com.tr`, go to Created Recipes → Import recipe, paste the
variant B URL.

If this fails, the format work is moot: either the importer requires a whitelisted
domain, or it cannot read our markup at all. Run variants A and C before concluding
which.

### Step 2 — the format question

Only if step 1 succeeds or fails ambiguously. Import A and C too, and compare what
each produces.

### Step 3 — the partner-widget question

The official "Add to Cookidoo" button resolves to a plain link (see `NOTES.md` §3):

```
https://<market>/created-recipes/<locale>/add-to-cookidoo?partnerId=<id>&recipeUrl=<url>
```

While logged in, try the Turkish equivalent with one of our pages:

```
https://cookidoo.com.tr/created-recipes/tr-TR/add-to-cookidoo?recipeUrl=<url-encoded page url>
```

Try it without a `partnerId` first. Whether the endpoint exists on the Turkish
market at all is unknown — anonymous probing cannot tell, because every path under
`/created-recipes/` redirects to login.

Do not build anything on top of borrowing another site's `partnerId`. If a partner
ID turns out to be required, the answer is to ask Vorwerk for one, not to reuse
someone else's.

### Step 4 — the question that actually decides the product

If a recipe imports, open it on the TM6/TM7 and record:

- Does it appear under Created Recipes on the device?
- Are temperature / time / speed set automatically per step, or does the user enter
  them by hand?

A recipe that imports but is not guided on the device is worth much less than it
looks on the web.

## What to record

For every attempt, note: variant, URL, whether the import was accepted, and then
field by field — title, ingredients (separate lines or one blob?), steps, and above
all whether temperature/time/speed landed in structured fields or stayed as text.

## Cache-busting

Retrying a corrected page under the same URL may hit a cached copy. Put each new
attempt under a fresh path — `v2/`, `v3/` — rather than editing files in `v1/`.
