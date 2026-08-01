# Test results

Fill this in as you go. Empty cells are honest; guesses are not.

Environment:

- Cookidoo market: `cookidoo.com.tr` (tr-TR)
- Subscription active: yes / no
- Tested from: desktop browser / Cookidoo mobile app / device
- Appliance: TM5 / TM6 / TM7
- Date:

## Step 1-2 — URL import

Paste each URL into Created Recipes → Import recipe.

| # | Variant | URL | Import accepted? | Title | Ingredients | Steps | Settings (temp/time/speed) | Notes |
|---|---------|-----|------------------|-------|-------------|-------|----------------------------|-------|
| 1 | B (microdata) | https://aenesbedir.github.io/recipe-poc/v1/b-microdata.html | | | | | | |
| 2 | A (JSON-LD) | https://aenesbedir.github.io/recipe-poc/v1/a-jsonld.html | | | | | | |
| 3 | C (plain) | https://aenesbedir.github.io/recipe-poc/v1/c-plain.html | | | | | | |

Column meanings:

- **Import accepted?** — did it create a recipe at all, or show an error? Record the
  exact error text.
- **Ingredients** — one per line, or all collapsed into a single blob?
- **Settings** — the decisive column. Did temperature / time / speed land in
  structured per-step fields, or stay as plain text inside the step description?

## Step 3 — partner widget endpoint

While logged in, open:

```
https://cookidoo.com.tr/created-recipes/tr-TR/add-to-cookidoo?recipeUrl=https%3A%2F%2Faenesbedir.github.io%2Frecipe-poc%2Fv1%2Fb-microdata.html
```

| Attempt | partnerId | Result |
|---------|-----------|--------|
| no partnerId | — | |
| invalid partnerId | `test-0000` | |

Record: does the page exist (a real import screen), 404, or a redirect somewhere
else? If it 404s on tr-TR, the widget path is simply not deployed for Turkey — which
matches the market list in `NOTES.md` §4.

## Step 4 — behaviour on the appliance

Only if something imported successfully.

| Question | Answer |
|----------|--------|
| Recipe visible under Created Recipes on the appliance? | |
| Steps shown one by one? | |
| Temperature/time/speed set automatically when advancing a step? | |
| Or does the user dial them in by hand? | |
| Timer starts on its own? | |

## Conclusion

Which of these happened:

- [ ] Import works **and** the appliance sets values automatically → transfer is a real feature, build it.
- [ ] Import works but the appliance needs manual entry → transfer is cosmetic, secondary at best.
- [ ] Import rejected for every variant → domain restriction. Transfer is out for now.
- [ ] Ambiguous → write down exactly what was ambiguous before deciding anything.
