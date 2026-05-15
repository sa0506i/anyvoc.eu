# anyvoc.eu — project conventions for Claude

Static marketing + legal site for the Anyvoc mobile app. Deployed via
GitHub Pages from `main`. No build step.

## "Last updated" date — MANDATORY bump on every content change

Every content page (legal, help, account) carries a visible
last-updated date near the top (or, in the case of `imprint.html`,
near the bottom):

```html
<p class="lead">Last updated: 2026-05-15.</p>
```

**Rule:** any commit that changes the substantive content of such a
page **must** bump that date to the day of the commit, in the same
commit. This is non-negotiable — visitors and regulators rely on the
date to know when they last looked at a current version, and Apple /
Google store reviews check it.

What counts as a substantive change:

- adding, removing, or rewording a sentence (yes — typo fixes too)
- adding, removing, or renumbering a section
- changing a license attribution, a price, a feature flag, a tier name
- changing a fact about a processor, a jurisdiction, a retention period
- correcting a stale cross-reference (e.g. "section 5" → "section 6")

What does **not** require a date bump:

- pure styling changes in `assets/styles.css` only
- whitespace / indentation reflows with no content change
- changes to `index.html` (it has no date)
- changes outside the content pages (e.g. `CNAME`, build assets, `.claude/`)

### Where the date lives in each file

| File                          | Location of date            |
| ----------------------------- | --------------------------- |
| `help/faq.html`               | top, after `<h1>`           |
| `help/acknowledgements.html`  | top, after `<h1>`           |
| `legal/privacy.html`          | top, after `<h1>`           |
| `legal/terms.html`            | top, after `<h1>`           |
| `legal/imprint.html`          | bottom, before `</article>` |
| `account/delete.html`         | top, after `<h1>` (if present) |

### Date format

ISO 8601 calendar date: `YYYY-MM-DD.` (with the trailing full stop, to
match existing copy). No times, no locale variants.

## Other conventions

- **Internal links** are relative and must round-trip from any starting
  page. Pages under `legal/` and `help/` reach each other via
  `../legal/foo.html` or `../help/bar.html`, **not** `foo.html` (that
  resolves inside the wrong directory). Past bugs: FAQ linking
  `privacy.html` instead of `../legal/privacy.html`.
- **Tier wording**: the free tier is called **Basic** (matching the
  in-app wording). Do not use "Free" as a tier name in user-facing
  copy. "Pro" is "Pro". Guests are "Guest".
- **FAQ ≠ implementation manual**: the FAQ may describe product-level
  behaviour (tiers, modes, limits, standard terms like OCR / CEFR /
  Leitner) but must **not** name third-party processors (Supabase,
  Mistral, Fly.io, ML Kit), storage engines, internal grid constants
  (puzzle char caps, tile counts), accuracy benchmarks, or fallback
  strategies. Those belong in the privacy policy where GDPR Art. 13
  requires processor names, or in the acknowledgements page where
  license obligations require attribution.
- **Acknowledgements** must keep the Leipzig Corpora (CC BY 4.0),
  Kuperman et al. AoA, Google ML Kit (Apache 2.0), Mozilla Readability
  (Apache 2.0), and Apple / Google trademark notices. Don't strip
  these on cleanup passes.
- **Jurisdictional coverage**: privacy + terms must explicitly cover
  the EU member states (GDPR), the EEA non-EU states (Norway, Iceland,
  Liechtenstein), the United Kingdom (UK GDPR / DPA 2018), and
  Switzerland (FADP). New territory? Add a section, don't quietly
  expand a generic clause.
- **Commit style**: one scope per commit, `Area: short summary` in the
  subject (`FAQ: …`, `Terms: …`, `Privacy: …`, `Acknowledgements: …`).
  Body explains the why, not the what.
