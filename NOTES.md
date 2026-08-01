# Research notes — Cookidoo recipe import

Everything below was gathered on 2026-08-01 from public sources. Nothing here is
official documentation of an integration contract; Vorwerk publishes none.

## 1. What the official docs actually say

The Cookidoo import tutorial states, verbatim:

> Copy the URL from a Cookidoo® recipe or from other recipe websites

Source: <https://cookidoo.thermomix.com/foundation/tutorials/en-US/courses/how-to-import/import-followthesesteps>

So "other websites" is explicitly mentioned. However, the same tutorial page lists
only Vorwerk-owned recipe communities as examples: recipecommunity.com.au,
rezeptwelt.de, recetario.es, ricettario-bimby.it, przepisownia.pl,
mundodereceitasbimby.com.pt, svetreceptu.cz, espace-recettes.fr.

The help-centre article is narrower and names only two sources (Cookidoo itself and
the Vorwerk Recipe Community):
<https://cookidoo.thermomix.com/foundation/en-US/articles/learn-about-importing-recipes>

No page documents which HTML or structured-data format an external site must provide.
That gap is the reason for this PoC.

## 2. How the Recipe Community page is actually marked up

Inspected: <https://www.recipecommunity.com.au/basics-recipes/basic-rice/dzm204s6-053f3-541039-beea0-0vynldmv>

Findings:

- **No JSON-LD at all.** Zero `application/ld+json` blocks on the page.
- Structured data is **microdata**: `itemscope itemtype="https://schema.org/Recipe"`
  wrapping the whole recipe.
- Fields observed: `name`, `url`, `image`, `description`, `datePublished`,
  `dateModified`, `recipeCategory`, `recipeCuisine`, `keywords`, `recipeYield`,
  `prepTime`, `cookTime`, `totalTime`, `recipeHint`, `author`, `tool`.
- Ingredients: `<li itemprop="recipeIngredient">` with separate `<span>`s for
  amount, unit, name and note.
- Instructions: `recipeInstructions` is a `HowToSection` containing
  `HowToStep` items, each with `<meta itemprop="position">`,
  `<meta itemprop="name">` (a truncated preview of the text) and
  `<span itemprop="text">`.
- **Machine settings are free text inside the step text**, wrapped in `<strong>`,
  e.g. `18-22min/Varoma/speed3`. There is no structured temperature/speed/time field
  anywhere in the markup.

That last point matters: whatever consumes this data must parse the settings out of
prose. There is no structured settings contract to conform to.

## 3. The "Add to Cookidoo" button is a partner widget

The page embeds a custom element:

```html
<add-to-cookidoo
    partner-id="rezeptwelt-b20807"
    market="cookidoo.com.au"
    lang="en"
    type="single-line"
    theme="cookidoo"></add-to-cookidoo>
<script type="module" async src="//assets.cookidoo.io/a2c/a2c.js"></script>
```

The widget (Stencil component, chunk `p-a5aa613a.entry.js`) accepts
`partnerId`, `market`, `recipeUrl`, `type`, `size`, `theme`, `width`, `height`,
`justifyContent`.

Its only job is to build a link:

```js
buildUrl() {
  const e = this.resolveMarket(),
        o = e.key.startsWith("dev-") ? "customer-recipes" : "created-recipes",
        i = new l(`${e.url}/${o}/${this.marketLocale}/add-to-cookidoo`);
  return i.addParam("partnerId", this.partnerId),
         i.addParam("recipeUrl", this.resolvePageUrl()),
         i.buildUrl();
}
```

Resulting shape:

```
https://<market>/created-recipes/<locale>/add-to-cookidoo?partnerId=<id>&recipeUrl=<url>
```

Implications:

- There **is** a `partnerId`. The one-click button is a partner programme with issued
  identifiers, not an open endpoint. Whether the backend validates `partnerId`
  against `recipeUrl`'s domain is unknown and is exactly what the PoC should probe.
- The endpoint requires authentication. Anonymous requests to
  `/created-recipes/...` on any market redirect to `eu.login.vorwerk.com`.
  A nonexistent path under `/created-recipes/` redirects identically, so an
  anonymous probe cannot tell whether a path exists. **Testing must be done
  logged in.**

## 4. Turkey is not in the widget's market list

The widget's market table (production entries only):

| key | url | locales |
| --- | --- | --- |
| cookidoo.de | https://cookidoo.de | de-DE |
| cookidoo.ch | https://cookidoo.ch | de-CH, it-CH, fr-CH, en |
| cookidoo.at | https://cookidoo.at | de-AT |
| cookidoo.co.uk | https://cookidoo.co.uk | en-GB |
| cookidoo.fr | https://cookidoo.fr | fr-FR |
| cookidoo.it | https://cookidoo.it | it-IT |
| cookidoo.es | https://cookidoo.es | es-ES |
| cookidoo.pt | https://cookidoo.pt | pt-PT |
| cookidoo.pl | https://cookidoo.pl | pl-PL |
| cookidoo.cz | https://cookidoo.cz | cs |
| cookidoo.com.au | https://cookidoo.com.au | en-AU, en-NZ |
| cookidoo.us | https://cookidoo.thermomix.com | en-US |
| cookidoo.international | https://cookidoo.international | en, fr, es, ms, da, el, id, is, kk, no, ro, ru, sv, uk, zh-Hans, zh-Hans-CN, pt, hu, vi |
| cookidoo.thermomix.com | https://cookidoo.thermomix.com | en-US |

Turkey's Cookidoo exists and is live — `https://cookidoo.com.tr` redirects to
`/foundation/tr-TR/explore` — but **no market entry maps to it**, and `tr` is not
among `cookidoo.international`'s locales.

Interestingly, the widget **does** ship Turkish UI strings
(`prefix: "buna ekle:"`, `"Oluşturulmuş Tarifler"`), so the locale is prepared but
the market is not wired up.

Consequence: even with a partner ID, the official one-click widget cannot currently
target Turkish users. That is a separate blocker from the URL-import question and
it affects the product plan directly.

## 5. What is still unknown

- Does the logged-in "Import recipe" box accept a URL from an arbitrary domain?
- If yes, which markup does it read — microdata, JSON-LD, or prose?
- Are the Thermomix settings (temperature / time / speed / reverse) extracted into
  structured fields, or kept as plain text?
- How does an imported Created Recipe behave on TM6/TM7 — guided, or a static
  step list?

Only the last two determine whether the import is worth building a product around.
