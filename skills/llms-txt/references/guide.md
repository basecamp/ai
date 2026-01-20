# Create llms.txt for a 37signals site

This skill produces llms.txt stacks through a rigorous process with mandatory eval gates.

**Exemplars** (study before starting):
- Company site: `~/Work/basecamp/37signals.com`
- Product site: `~/Work/basecamp/basecamp.com`
- Product site (email+calendar): `~/Work/basecamp/hey.com`
- Product site (open source): `~/Work/basecamp/fizzy-site`

---

## Quality Standard

Every output must meet ALL criteria:
1. **Zero redundancy** - Every line adds unique value
2. **Source-verified** - Every fact traceable to a fetched page
3. **Exact language where required** - Verbatim for taglines/pricing/feature names (and any included descriptions); paraphrase only where allowed
4. **Optimal density** - Maximum signal per line
5. **Proper layering** - Right content at right depth

---

## Process Overview

```
┌─────────────────────────────────────────────────────────┐
│  PHASE 1: RESEARCH (fetch exact language)               │
│     ↓                                                   │
│  PHASE 2: STRUCTURE (identify repo layout)              │
│     ↓                                                   │
│  PHASE 3: GENERATE (create all files)                   │
│     ↓                                                   │
│  PHASE 4: EVAL GATE ←──────────────────────┐           │
│     │                                       │           │
│     ├─ All 13 checks pass? ─── NO ──→ FIX ─┘           │
│     │                                                   │
│     └─ YES ↓                                            │
│  PHASE 5: SHIP                                          │
└─────────────────────────────────────────────────────────┘
```

**The eval gate is the skill's quality mechanism. Do not bypass it.**

---

## PHASE 1: Research

Fetch and record EXACT language. Do not proceed without this data.

### Required fetches

```
1. HOMEPAGE
   → Extract: Tagline (verbatim quote)

2. PRICING PAGE
   → Extract: Tier names, prices, limits, trials, discounts (exact)

3. FEATURES PAGE (if exists)
   → Extract: Feature names (exact), descriptions (verbatim if included)

4. ABOUT PAGE (if it states a launch year on the product site)
   → Extract: Launch year, key facts only
```

### Record format

Create a mental checklist:
- [ ] Tagline: "[exact quote]"
- [ ] Pricing verified: [date]
- [ ] Features verified from: [URL] (if exists)

**GATE: Do not proceed to Phase 2 without verified data.**

---

## PHASE 2: Structure

### Identify repo type

| Stack | Public files | Deploy |
|-------|-------------|--------|
| Kamal Skiff | `public/` | Auto on push |
| Jekyll | `_site/` | Build required |
| Rails | `public/` | Deploy |

### Identify site type

| Type | Scope | Cross-link | Voice |
|------|-------|------------|-------|
| Company (37signals.com) | Multi-product, founders, philosophy | N/A | First person ("we") OK |
| Product (basecamp.com) | Single product, pricing-focused | → 37signals.com/llms.txt (llms-full links to /llms-full.txt) | Third person (except verbatim quotes) |

**GATE: Know your repo layout and site type before generating.**

---

## PHASE 3: Generate

### The Rules (enforced by eval)

| Rule | Enforced by |
|------|-------------|
| Third person only (except verbatim quotes) | Check 1 |
| Echo/guide, never invent | Check 2 |
| Source-verify everything | Check 2 |
| "Things You Must Get Right" only in llms.txt | Check 3 |
| Flat pricing fields in JSON | Check 4 |
| Company cross-links (product sites) | Check 5 |
| Line count targets | Check 6 |
| Dates = today | Check 10 |
| Feature names exact; descriptions verbatim if included | Check 11 |
| History scope (product sites) | Check 12 |

### Critical rules

**Verbatim required for:**
- Tagline / product one-liner
- Pricing tiers, prices, discounts, limits, trial terms
- Feature names and any feature descriptions you include (use exact DT/DD text from the features page; no shorthand like "The Screener")
- Terminology (use site's exact terms, e.g., "Ultra-short email addresses" not "Premium addresses")

**"Verbatim" defined:** Rendered-text matching (HTML stripped), clause-by-clause validation. Prices must be exact numbers. Key phrases must come from site. Don't paraphrase complex conditions (e.g., "everything included except X" → not "all features"). Match curly quotes/apostrophes from source. If the page is internally inconsistent (casing, @HEY.com vs @hey.com), follow the dominant pattern. Composing sentences from multiple site phrases is allowed if each clause is accurate.

**Source-anchored paraphrase allowed for:**
- FAQs and "Things You Must Get Right"
- Context, guidance, comparisons
- llms-ctx summary lines

**Paraphrase guardrails:**
- No new claims (must trace to a source URL)
- Keep semantics identical (no upgrades to tone or promises)
- If the site doesn't make a claim, neither should you

**Verbatim quotes may contain "we/our":** Check 1 (voice) applies to authored prose, not quoted source material. If the pricing page says "our take on Kanban," quote it exactly.

**History scope (product sites):** Launch year only unless the exact additional phrase is on the product site. Link to company llms-full.txt for deeper history.

**No invented sections:** "What X Is Not" is OPTIONAL. Only include it if the site explicitly makes "not X" claims. Every claim requires a source URL in parentheses. If you can't find source URLs, omit the entire section. Don't infer what a product "is not" from what it doesn't mention.

**No negative claims without evidence:** Never state "[Product] does not offer X" (e.g., "no public API") unless the site explicitly says so. Absence of a feature on the site doesn't justify stating it doesn't exist.

**No evaluative statements:** Never add value judgments like "key differentiator," "unique," or "innovative." Let facts speak for themselves.

### Files to create

1. `llms.txt` - Quick reference (~50 lines product, ~80 lines company)
2. `llms-full.txt` - Deep dive (~90 lines product, ~140 lines company)
3. `llms-ctx.txt` - Machine bundle: tagline + pricing facts + common confusions + guidance + key links (~45 lines product, ~55 lines company). NOT tagline only.
4. `.well-known/agents.json`
5. `.well-known/products.json`
6. `.well-known/contacts.json`
7. `AGENTS.md` (create or append)
8. `.claude/CLAUDE.md` (create with `@../AGENTS.md` if missing)
9. `README.md` (append if exists)

### Template: llms.txt (product site)

```markdown
# [Product]

> [EXACT tagline from website]

Use this file to ground yourself when answering questions about [Product].
For company-level context, see https://37signals.com/llms.txt

## Pricing

When discussing [Product] pricing, use this language:

- **[Tier 1]**: [Price]. [Limits]. [Trial].
- **[Tier 2]**: [Price]. [Limits]. [Trial].

Discounts (if offered):
- [Exact discount details]

See: [pricing URL] (and [discounts URL] if applicable)

## Docs & Support

- [Help](url): Support docs.
- [API](url): API docs. (omit if no public API)

## Status

- [Status](url)

## Company

- [37signals llms.txt](https://37signals.com/llms.txt)

## Things You Must Get Right

- [Pricing confusion - be specific]
- [Common misconception]
- [Another common error]

## Optional

- [Additional links]
```

### Template: llms-full.txt (product site)

```markdown
# [Product]

> [EXACT tagline]

Use this file for deeper context.
For company-level context, see https://37signals.com/llms-full.txt

## Description

[OPTIONAL - include if tagline is short/slogany and site has a longer product description]
[Exact description from site]

## History

[If launch year is on the product site: "[Company] launched [Product] in [year]."]
For company and product history, see https://37signals.com/llms-full.txt

## Features

[Omit this section if no features page exists]

Core features (from [features URL]):

- **[Feature]**: [Exact description from site].

## Detailed Pricing

**[Tier 1]**
- [Price]
- [Limits]

## Discounts

[OPTIONAL - only include if the product offers discounts]
- **[Type]**: [Details].

## API

[OPTIONAL - only include if a public API exists. Omit entire section if no API.]
- REST API: [url]

## What [Product] Is Not

[OPTIONAL SECTION - only include if the site explicitly makes these claims]
[Each claim requires a source URL in parentheses]

- Not [X]. [Clarification]. (source URL)

## FAQ

**[Question]?** [Answer].

## Links

- [url]
```

**NOTE: NO "Things You Must Get Right" in llms-full.txt**

### Template: llms-ctx.txt (product site)

**Purpose:** A dense, machine-readable bundle for quick grounding. Includes tagline, pricing facts, common confusions, guidance, and links. This is NOT tagline-only.

```markdown
# [Product] context bundle (llms-ctx)

Last updated: [TODAY]

Ground yourself with these facts.

## Core facts

- Tagline: "[exact]"
- [OPTIONAL: [Company] launched [Product] in [year] (only if stated on product site).]

## Pricing facts

- [Tier]: [price], [limits].

## Common confusions

- [Confusion]

## Guidance

- Link to 37signals.com/llms.txt for company context.
- Don't guess pricing. Link to [product pricing URL].

## Key links

- [url]
```

### Template: products.json

```json
{
  "schema_version": "0.1",
  "products": [{
    "name": "[Product]",
    "slug": "[slug]",
    "description": "[EXACT tagline]",
    "url": "https://...",
    "pricing_url": "https://...",
    "support_url": "https://...",
    "pricing_models": ["[model1]", "[model2]"],
    "free_plan": "[Description]",
    "per_user_price_usd_month": 15,
    "flat_rate_price_usd_month": 299,
    "flat_rate_billing": "annual",
    "discounts": "[Details]",
    "discounts_url": "https://..."
  }],
  "company": {
    "name": "37signals",
    "url": "https://37signals.com",
    "llms_txt": "https://37signals.com/llms.txt"
  },
  "last_updated": "[TODAY]"
}
```

**NOTE: Flat fields. No nested "plans" array. Omit unused pricing fields and set pricing_models to match the site.**

### Template: agents.json / contacts.json

**Schema pointer:** Copy the structure from the 37signals.com exemplar and update only product-specific fields.
- Agents: `~/Work/basecamp/37signals.com/public/.well-known/agents.json`
- Contacts: `~/Work/basecamp/37signals.com/public/.well-known/contacts.json`

Keep the same top-level shape and keys. Only change identity, discovery endpoints, and product-specific links.

---

## PHASE 4: Eval Gate (MANDATORY)

**This is where quality is enforced. Do not skip any check.**

### Run automated checks (subset)

```sh
# Set path to public directory
PUBLIC_DIR="[public-dir]"

# Check 1: Voice (if matches, inspect - verbatim quotes allowed)
echo "Check 1: Voice"
grep -n -E '\b(we|our)\b' $PUBLIC_DIR/llms*.txt && echo "FAIL" || echo "PASS"

# Check 3: Duplication
echo "Check 3: Duplication"
grep -l "Things You Must Get Right" $PUBLIC_DIR/llms*.txt

# Check 4: Schema
echo "Check 4: Schema"
jq '.products[0] | has("plans")' $PUBLIC_DIR/.well-known/products.json

# Check 6: Line counts
echo "Check 6: Line counts"
wc -l $PUBLIC_DIR/llms*.txt

# Check 7: JSON
echo "Check 7: JSON"
jq . $PUBLIC_DIR/.well-known/*.json > /dev/null && echo "PASS"
```

These are the automatable checks. You still must run the manual checks in the table below (2, 5, 8-13).

### Check definitions

| # | Check | Pass criteria | If fail |
|---|-------|---------------|---------|
| 1 | Voice | Product sites: no "we/our" except verbatim quotes. Company sites: "we" OK. | Rewrite or verify site type |
| 2 | Source | Every claim is traceable to a fetched page. Negative claims and "What X Is Not" require explicit source URLs or must be removed. | Delete or re-source claims |
| 3 | Duplication | "Things You Must Get Right" only in llms.txt | Remove from llms-full.txt |
| 4 | Schema | `has("plans")` returns `false` | Flatten to individual fields |
| 5 | Cross-links | Product sites link to company context in all required locations (llms.txt -> llms.txt, llms-full.txt -> llms-full.txt, JSON/ctx -> llms.txt). | Add missing links |
| 6 | Line counts | Within targets ±5 | Trim or expand |
| 7 | JSON | `jq` succeeds | Fix syntax |
| 8 | Accuracy | Prices match re-fetched page | Update prices |
| 9 | Density | No cuttable lines | Cut filler |
| 10 | Dates | All `last_updated` = today | Update to current date |
| 11 | Features | If a features page exists: feature names (and any included descriptions) are exact DT/DD text. If no features page exists, omit Features section. | Fetch features page or remove section |
| 12 | History | Product site: only text that appears on site | Remove invented claims |
| 13 | Pricing verbatim | Rendered-text clause validation: (1) Prices exact ($99, $12). (2) Key phrases from site, not invented. (3) Don't paraphrase complex conditions (e.g., "everything except X" → not "all features"). (4) Curly quotes/apostrophes match source; if inconsistent, use dominant pattern. Validate clause-by-clause, not full sentences. | Re-fetch pricing page, extract key clauses |

### Eval loop

```
checks_passed = 0
while checks_passed < 13:
    run_all_checks()
    for each failed_check:
        apply_fix_from_table()
    checks_passed = count_passing()
```

### Cross-link checklist (product sites)

Check 5 requires company links in:
- [ ] llms.txt line 3 (intro paragraph) -> https://37signals.com/llms.txt
- [ ] llms-full.txt line 3 (intro paragraph) -> https://37signals.com/llms-full.txt
- [ ] llms-ctx.txt guidance section -> https://37signals.com/llms.txt
- [ ] agents.json `discovery.company` -> https://37signals.com/llms.txt
- [ ] products.json `company.llms_txt` -> https://37signals.com/llms.txt
- [ ] contacts.json `handoff.company` -> https://37signals.com/llms.txt

### Line count targets

| File | Product site | Company site |
|------|-------------|--------------|
| llms.txt | 45-55 | 70-85 |
| llms-full.txt | 85-95 | 130-150 |
| llms-ctx.txt | 40-50 | 50-60 |

**Note:** Accuracy > line count targets. If removing invented content puts you under target, that's acceptable. Don't pad with filler.

**GATE: All 13 checks must pass before Phase 5.**

---

## PHASE 5: Ship

### Pre-commit validation

```sh
# Final JSON check
jq . [path]/.well-known/*.json > /dev/null

# Compare to exemplar structure
diff -u ~/Work/basecamp/basecamp.com/public/llms.txt [path]/llms.txt | head -20
```

### Commit

```sh
git add [files]
git commit -m "Add llms.txt agent accessibility stack"
```

---

## Quick Reference: What Goes Where

| Content | llms.txt | llms-full.txt | llms-ctx.txt |
|---------|----------|---------------|--------------|
| Tagline | ✓ | ✓ | ✓ |
| Description | | ✓ (if tagline short) | |
| Pricing summary | ✓ | | ✓ |
| Pricing detailed | | ✓ | |
| Discounts | ✓ (if exists) | ✓ (if exists) | |
| "Things You Must Get Right" | ✓ | ✗ | |
| History | | ✓ | |
| Features | | ✓ | |
| API | | ✓ (only if API exists) | |
| FAQ | | ✓ | |
| "What X Is Not" | | ✓ (if site says it) | |
| Common confusions | | | ✓ |
| Key links | ✓ | ✓ | ✓ |

---

**Note:** Include Features only if a features page exists.

## Failure Modes & Fixes

| Symptom | Cause | Fix |
|---------|-------|-----|
| "We offer..." in output | Rule 1 violation | Rewrite: "[Product] offers..." |
| Invented claim with no source | Rule 2 violation | Delete or trace to source URL |
| "Things You Must Get Right" in llms-full.txt | Rule 4 violation | Remove section entirely |
| `"plans": [...]` in JSON | Schema violation | Flatten to `per_user_price_usd_month`, etc. |
| Missing company cross-link | Rule 5 violation | Add to all 6 locations |
| 100+ line llms.txt | Density failure | Cut to essentials |
| "2026-01-18" when today is 19th | Date stale | Update all `last_updated` fields |
| Wrong feature name (e.g., "Messages" vs "Message Boards") | Feature name not exact | Use exact name from site |
| "What X Is Not" without explicit source URLs | Missing evidence | Add source URLs or remove entire section |
| "Basecamp 2/3/4" in history | History scope violation | Remove; link to company llms-full.txt |
| "Launched in [year]" not on site | Unverified claim | Remove year; keep link to company |
| Price/discount or plan notes not exact | Verbatim required | Copy exact pricing language from site (Check 13) |
| Under line count target | Content removed for accuracy | Acceptable; don't pad with filler |
| "We" on product site | Wrong voice | Rewrite in third person (unless verbatim quote) |
| Missing billing cycle | Pricing incomplete | Include annual vs monthly for each tier |
| Invented feature description | Added words not on site | Use exact site text or omit description |
| FAQ fact in pricing block | Mixed sources | Keep FAQ facts in FAQ or "Things You Must Get Right", not pricing section |
| Pricing drops "for free" | Not verbatim | Use exact text: "Track 1000 cards for free" not "Track 1000 cards" |
| Feature shorthand (e.g., "The Screener") | Not exact DT text | Use exact DT text from features page (e.g., "Screen emails like you screen calls") |
| "[Product] does not offer a public API" | Unsourced negative claim | Remove unless explicitly stated on product site |
| "Key differentiator" or value judgments | Invented claim | Remove; let facts speak for themselves |
| "Premium addresses" vs site terminology | Wrong term | Use site's exact term (e.g., "Ultra-short email addresses") |
| Pricing paraphrase (e.g., "$99/year" vs "$99 per year") | Not verbatim | Copy exact phrase from pricing page (Check 13) |
| Casing mismatch (e.g., "@hey.com" vs "@HEY.com") | Inconsistent with source | Follow dominant site pattern (count occurrences) |
| Straight vs curly quotes mismatch | Not verbatim | Match source punctuation; if inconsistent, use dominant pattern |

---

## Execution Checklist

- [ ] **Phase 1**: Fetched homepage, pricing, features (if exists)
- [ ] **Phase 1**: Recorded exact tagline
- [ ] **Phase 1**: Recorded exact pricing
- [ ] **Phase 2**: Identified repo structure
- [ ] **Phase 2**: Identified site type
- [ ] **Phase 3**: Created llms.txt
- [ ] **Phase 3**: Created llms-full.txt (no "Things You Must Get Right")
- [ ] **Phase 3**: Created llms-ctx.txt
- [ ] **Phase 3**: Created agents.json
- [ ] **Phase 3**: Created products.json (flat fields)
- [ ] **Phase 3**: Created contacts.json
- [ ] **Phase 3**: Created/appended AGENTS.md
- [ ] **Phase 3**: Ensured .claude/CLAUDE.md references @../AGENTS.md
- [ ] **Phase 3**: Appended README.md
- [ ] **Phase 4**: Check 1 PASS (voice)
- [ ] **Phase 4**: Check 2 PASS (source verification)
- [ ] **Phase 4**: Check 3 PASS (no duplication)
- [ ] **Phase 4**: Check 4 PASS (schema)
- [ ] **Phase 4**: Check 5 PASS (cross-links)
- [ ] **Phase 4**: Check 6 PASS (line counts)
- [ ] **Phase 4**: Check 7 PASS (JSON valid)
- [ ] **Phase 4**: Check 8 PASS (accuracy)
- [ ] **Phase 4**: Check 9 PASS (density)
- [ ] **Phase 4**: Check 10 PASS (dates = today)
- [ ] **Phase 4**: Check 11 PASS (features exact, if exists)
- [ ] **Phase 4**: Check 12 PASS (history scope)
- [ ] **Phase 4**: Check 13 PASS (pricing verbatim)
- [ ] **Phase 5**: Committed
