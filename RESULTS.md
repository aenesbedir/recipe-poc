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

| # | Variant | URL | Import accepted? | Error |
|---|---------|-----|------------------|-------|
| 1 | B (microdata) | https://aenesbedir.github.io/recipe-poc/v1/b-microdata.html | no | "Tarif içe aktarılamadı — İçe aktarım esnasında bazı şeyler ters gitti. Buna alternatif olarak, yeni bir tarif oluşturabilir ve her bir tarif ögesini kaynak sayfadan manuel olarak kopyalayıp, yapıştırabilirsiniz." |
| 2 | A (JSON-LD) | https://aenesbedir.github.io/recipe-poc/v1/a-jsonld.html | no | "Tarif içe aktarılamadı — Bu web sitesinden içeri aktarım şu anda onaylanmış değil. Yalnızca Cookidoo veya Vorwerk topluluk sitelerinden aktarıma izin verilmektedir." |
| 3 | C (plain) | https://aenesbedir.github.io/recipe-poc/v1/c-plain.html | not run | Skipped — the domain check runs before any markup parsing, so markup cannot change the outcome. |

The variant A message is decisive: **the importer enforces a source allowlist.**
Only Cookidoo itself and Vorwerk community sites are accepted. Markup format is
irrelevant for a non-approved domain.

### Two entry points behave differently

The same domain produced two different messages, which turned out to be an entry
point difference, not a source difference:

| Entry point | Source | Result |
|---|---|---|
| Created Recipes → `+` → Import recipe | approved (recipecommunity.com.au) | **imports successfully** |
| Created Recipes → `+` → Import recipe | ours (github.io) | rejected, allowlist message |
| `add-to-cookidoo?recipeUrl=` deep link | anything, including an approved source | generic failure |

Conclusions:

- The **import box** is the real code path. It performs the source check and
  produces the specific allowlist message.
- The **deep link** is broken on the Turkish market. It fails generically even for
  an approved Vorwerk source, which matches `NOTES.md` §4: the a2c widget has no
  market entry for Turkey, so the route it points at was never wired up here.
- Turkey's market **does** accept a recipe from a foreign approved market (an
  Australian Recipe Community URL imported fine into a `tr-TR` account). The gate is
  the domain alone — not language, not market.

Column meanings:

- **Import accepted?** — did it create a recipe at all, or show an error? Record the
  exact error text.
- **Ingredients** — one per line, or all collapsed into a single blob?
- **Settings** — the decisive column. Did temperature / time / speed land in
  structured per-step fields, or stay as plain text inside the step description?

## Step 3 — partner widget endpoint

```
https://cookidoo.com.tr/created-recipes/tr-TR/add-to-cookidoo?recipeUrl=<url>
```

| Question | Answer |
|----------|--------|
| Does the path exist on tr-TR? | It responds, but never succeeds — an approved Vorwerk source fails through it too. Effectively non-functional on this market. |
| Is `partnerId` required to reach the flow? | No. Its absence is not what causes the failure. |
| Usable at all? | No. Use the Created Recipes import box instead. |

## Step 4 — behaviour on the appliance

**Now testable.** The Recipe Community recipe imported through the import box is
sitting in Created Recipes, so the remaining product-critical question can be
answered with it — no third-party site required.

| Question | Answer |
|----------|--------|
| Recipe visible under Created Recipes on the appliance? | |
| Steps shown one by one? | |
| Temperature/time/speed set automatically when advancing a step? | |
| Or does the user dial them in by hand? | |
| Timer starts on its own? | |

## Conclusion

- [x] **Import rejected for every variant → source allowlist. Transfer is out for now.**

Cookidoo restricts recipe import to Cookidoo and Vorwerk community sites. A
third-party site cannot become an import source by publishing better markup — the
gate is the domain, not the format. The tutorial wording "or from other recipe
websites" means "other *approved* recipe websites" in practice.

### What this means for the product

1. A "Send to Cookidoo" button cannot be built. Do not put one in the MVP.
2. The realistic fallback is the one Cookidoo's own error message suggests:
   help the user copy each recipe element into a manually created recipe.
3. The primary value has to live outside Cookidoo — a step-by-step cooking mode in
   the web app itself, which also serves TM5/TM31 owners with no subscription.
4. If the transfer is ever wanted, the ask to Vorwerk is now concrete: get the
   domain added to the approved-source list, and get a `partnerId` issued for the
   `add-to-cookidoo` widget. That is a partnership conversation, not an
   engineering task, and it needs an existing user base first.
