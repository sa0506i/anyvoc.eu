# Website Restructure Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure the static site so `/legal/` only holds legally-mandated documents, non-legal pages live under `/help/` and `/account/`, static media moves to `/assets/`, and the footer reduces to four items — without breaking installed-app users on hardcoded URLs.

**Architecture:** Sequential file moves + path rewrites + footer simplification + meta-refresh redirect stubs at the legacy URLs for backward compat. Each task ends in a coherent committable state where every HTML page in the repo loads, every footer link resolves, and (after Task 7) every legacy URL redirects.

**Tech Stack:** Static HTML, plain CSS (self-hosted Inter font), GitHub Pages (custom domain `anyvoc.eu` via CNAME). No build pipeline, no test runner. Verification is via `grep` for residual paths, `ls` for file existence, and visual browser preview.

**Spec:** `docs/superpowers/specs/2026-05-07-website-restructure-design.md`

---

## File-Level Map

| File                               | Operation | Notes |
|------------------------------------|-----------|-------|
| `logo.png`                         | move      | → `assets/logo.png` |
| `styles.css`                       | move      | → `assets/styles.css` |
| `fonts/inter-latin.woff2`          | move      | → `assets/fonts/inter-latin.woff2` |
| `fonts/inter-latin-ext.woff2`      | move      | → `assets/fonts/inter-latin-ext.woff2` |
| `fonts/`                           | delete    | empty after the moves |
| `legal/faq.html`                   | move      | → `help/faq.html` |
| `legal/attributions.html`          | move + rename | → `help/acknowledgements.html` |
| `delete-account.html`              | move      | → `account/delete.html` |
| `index.html`                       | edit      | path updates + footer (Task 1, 2, 3, 4, 6) |
| `legal/imprint.html`               | edit      | path updates + body link + footer (Task 1, 2, 3, 4, 5, 6) |
| `legal/privacy.html`               | edit      | path updates + body links + footer (Task 1, 2, 3, 4, 5, 6) |
| `legal/terms.html`                 | edit      | path updates + footer (Task 1, 2, 3, 4, 6) |
| `help/faq.html` (after Task 2)     | edit      | path updates + footer (Task 3, 4, 6) |
| `help/acknowledgements.html` (after Task 3) | edit | path updates + footer (Task 4, 6) |
| `account/delete.html` (after Task 4) | edit    | path updates + footer (Task 6) |
| `legal/faq.html`                   | recreate  | Task 7: redirect stub |
| `legal/attributions.html`          | recreate  | Task 7: redirect stub |
| `delete-account.html`              | recreate  | Task 7: redirect stub |

---

## Task 1: Move static assets to `/assets/`

**Files:**
- Move: `logo.png`, `styles.css`, `fonts/inter-latin.woff2`, `fonts/inter-latin-ext.woff2`
- Modify: all 7 HTML files (`index.html`, `delete-account.html`, `legal/imprint.html`, `legal/privacy.html`, `legal/terms.html`, `legal/faq.html`, `legal/attributions.html`)
- Delete: `fonts/` directory (empty after moves)

- [ ] **Step 1: Snapshot current state**

```
ls *.png *.css fonts/
grep -l 'href="styles.css"\|src="logo.png"\|href="../styles.css"\|src="../logo.png"' index.html legal/*.html delete-account.html
```

Expected: `logo.png`, `styles.css` exist at root; `fonts/` has 2 woff2 files; grep lists all 7 HTML files.

- [ ] **Step 2: Move the files via git**

```
mkdir -p assets
git mv logo.png assets/logo.png
git mv styles.css assets/styles.css
git mv fonts assets/fonts
```

- [ ] **Step 3: Verify the moves succeeded**

```
ls assets/
ls assets/fonts/
ls fonts 2>&1 || echo "fonts/ removed (expected)"
```

Expected: `assets/` contains `logo.png`, `styles.css`, `fonts/`; `fonts/` at root no longer exists.

- [ ] **Step 4: Update `index.html` paths**

Find:
```html
    <link rel="stylesheet" href="styles.css" />
```
Replace with:
```html
    <link rel="stylesheet" href="assets/styles.css" />
```

Find:
```html
        <img src="logo.png" alt="Anyvoc" width="200" height="200" />
```
Replace with:
```html
        <img src="assets/logo.png" alt="Anyvoc" width="200" height="200" />
```

- [ ] **Step 5: Update `delete-account.html` paths**

Find:
```html
    <link rel="stylesheet" href="styles.css" />
```
Replace with:
```html
    <link rel="stylesheet" href="assets/styles.css" />
```

Find:
```html
        <img src="logo.png" alt="" />
```
Replace with:
```html
        <img src="assets/logo.png" alt="" />
```

- [ ] **Step 6: Update all four `legal/*.html` files (path prefix `../`)**

For each of `legal/imprint.html`, `legal/privacy.html`, `legal/terms.html`, `legal/faq.html`, `legal/attributions.html`:

Find:
```html
    <link rel="stylesheet" href="../styles.css" />
```
Replace with:
```html
    <link rel="stylesheet" href="../assets/styles.css" />
```

Find:
```html
        <img src="../logo.png" alt="" />
```
Replace with:
```html
        <img src="../assets/logo.png" alt="" />
```

(Five files — repeat the same two replacements in each.)

- [ ] **Step 7: Verify no residual references to old paths**

```
grep -rn 'href="styles.css"\|href="../styles.css"\|src="logo.png"\|src="../logo.png"\|"fonts/' index.html legal/ delete-account.html
```

Expected: no matches. (The `@font-face` rules inside `assets/styles.css` still reference `fonts/inter-*.woff2`, which is correct because `fonts/` now lives next to `styles.css` inside `assets/`.)

- [ ] **Step 8: Commit**

```
git add assets/ index.html delete-account.html legal/imprint.html legal/privacy.html legal/terms.html legal/faq.html legal/attributions.html
git commit -m "Move static assets to /assets/ (logo, styles, fonts)"
```

---

## Task 2: Move `legal/faq.html` → `help/faq.html`

**Files:**
- Move: `legal/faq.html` → `help/faq.html`
- Modify: `index.html`, `delete-account.html`, `legal/imprint.html`, `legal/privacy.html`, `legal/terms.html`, `legal/attributions.html`, and the moved `help/faq.html` itself (footer)

- [ ] **Step 1: Snapshot current FAQ link occurrences**

```
grep -rn '"legal/faq.html"\|"faq.html"\|"../legal/faq.html"' index.html legal/ delete-account.html
```

Expected: 7 hits (one per HTML file's footer). Note: in `legal/*.html` footers, the link reads `"faq.html"` (sibling); in `index.html` and `delete-account.html` it reads `"legal/faq.html"`.

- [ ] **Step 2: Create `help/` and move the file**

```
mkdir -p help
git mv legal/faq.html help/faq.html
```

- [ ] **Step 3: Verify the move**

```
ls help/
ls legal/ | grep -v faq
```

Expected: `help/faq.html` exists; `legal/faq.html` is gone.

- [ ] **Step 4: Update incoming links in `index.html`**

Find:
```html
          <a href="legal/faq.html">FAQ</a>
```
Replace with:
```html
          <a href="help/faq.html">FAQ</a>
```

- [ ] **Step 5: Update incoming links in `delete-account.html`**

Find:
```html
          <a href="legal/faq.html">FAQ</a>
```
Replace with:
```html
          <a href="help/faq.html">FAQ</a>
```

- [ ] **Step 6: Update incoming links in `legal/imprint.html`**

Find:
```html
          <a href="faq.html">FAQ</a>
```
Replace with:
```html
          <a href="../help/faq.html">FAQ</a>
```

- [ ] **Step 7: Update incoming links in `legal/privacy.html`**

Find:
```html
          <a href="faq.html">FAQ</a>
```
Replace with:
```html
          <a href="../help/faq.html">FAQ</a>
```

- [ ] **Step 8: Update incoming links in `legal/terms.html`**

Find:
```html
          <a href="faq.html">FAQ</a>
```
Replace with:
```html
          <a href="../help/faq.html">FAQ</a>
```

- [ ] **Step 9: Update incoming links in `legal/attributions.html`**

Find:
```html
          <a href="faq.html">FAQ</a>
```
Replace with:
```html
          <a href="../help/faq.html">FAQ</a>
```

- [ ] **Step 10: Update outgoing links inside `help/faq.html`**

The moved file currently links to its `legal/` siblings via bare names; those still resolve since the file is now in `help/`, which is also depth 1, but the targets are in `legal/`. Update:

For each of these in `help/faq.html`:

Find:
```html
          <a href="imprint.html">Imprint</a>
          <a href="privacy.html">Privacy</a>
          <a href="terms.html">Terms</a>
          <a href="attributions.html">Acknowledgements</a>
```
Replace with:
```html
          <a href="../legal/imprint.html">Imprint</a>
          <a href="../legal/privacy.html">Privacy</a>
          <a href="../legal/terms.html">Terms</a>
          <a href="../legal/attributions.html">Acknowledgements</a>
```

(The self-link `<a href="faq.html">FAQ</a>` stays as-is — it's a same-directory link.)

- [ ] **Step 11: Verify no residual old FAQ paths**

```
grep -rn '"legal/faq.html"' .
grep -rn '"faq.html"' legal/
```

Expected: no matches in either grep (Task 7 will create a stub at `legal/faq.html` later, but right now the path should be entirely vacated).

- [ ] **Step 12: Commit**

```
git add help/faq.html index.html delete-account.html legal/imprint.html legal/privacy.html legal/terms.html legal/attributions.html
git commit -m "Move FAQ from /legal/ to /help/"
```

---

## Task 3: Move + rename `legal/attributions.html` → `help/acknowledgements.html`

**Files:**
- Move + rename: `legal/attributions.html` → `help/acknowledgements.html`
- Modify: `index.html`, `delete-account.html`, `legal/imprint.html`, `legal/privacy.html`, `legal/terms.html`, `help/faq.html`, and the renamed `help/acknowledgements.html` itself (footer)

- [ ] **Step 1: Snapshot current attributions link occurrences**

```
grep -rn 'attributions.html' .
```

Expected: hits in all 6 other HTML files' footers (`legal/imprint.html`, `legal/privacy.html`, `legal/terms.html`, `index.html`, `delete-account.html`, `help/faq.html`) plus the file's self-link. Patterns will be `"attributions.html"` (legal-sibling), `"legal/attributions.html"` (root), and `"../legal/attributions.html"` (after Task 2's update of help/faq).

- [ ] **Step 2: Move + rename the file**

```
git mv legal/attributions.html help/acknowledgements.html
```

- [ ] **Step 3: Verify**

```
ls help/
ls legal/
```

Expected: `help/` contains `faq.html` and `acknowledgements.html`; `legal/` contains only `imprint.html`, `privacy.html`, `terms.html`.

- [ ] **Step 4: Update `index.html`**

Find:
```html
          <a href="legal/attributions.html">Acknowledgements</a>
```
Replace with:
```html
          <a href="help/acknowledgements.html">Acknowledgements</a>
```

- [ ] **Step 5: Update `delete-account.html`**

Find:
```html
          <a href="legal/attributions.html">Acknowledgements</a>
```
Replace with:
```html
          <a href="help/acknowledgements.html">Acknowledgements</a>
```

- [ ] **Step 6: Update `legal/imprint.html`**

Find:
```html
          <a href="attributions.html">Acknowledgements</a>
```
Replace with:
```html
          <a href="../help/acknowledgements.html">Acknowledgements</a>
```

- [ ] **Step 7: Update `legal/privacy.html`**

Find:
```html
          <a href="attributions.html">Acknowledgements</a>
```
Replace with:
```html
          <a href="../help/acknowledgements.html">Acknowledgements</a>
```

- [ ] **Step 8: Update `legal/terms.html`**

Find:
```html
          <a href="attributions.html">Acknowledgements</a>
```
Replace with:
```html
          <a href="../help/acknowledgements.html">Acknowledgements</a>
```

- [ ] **Step 9: Update `help/faq.html`**

Find:
```html
          <a href="../legal/attributions.html">Acknowledgements</a>
```
Replace with:
```html
          <a href="acknowledgements.html">Acknowledgements</a>
```

- [ ] **Step 10: Update outgoing links inside `help/acknowledgements.html`**

The file inherited footer entries from when it was in `legal/`. They reference siblings via bare names, but its siblings are now in `legal/` at depth 1 from `help/`.

For these in `help/acknowledgements.html`:

Find:
```html
          <a href="imprint.html">Imprint</a>
          <a href="privacy.html">Privacy</a>
          <a href="terms.html">Terms</a>
          <a href="attributions.html">Acknowledgements</a>
          <a href="faq.html">FAQ</a>
```
Replace with:
```html
          <a href="../legal/imprint.html">Imprint</a>
          <a href="../legal/privacy.html">Privacy</a>
          <a href="../legal/terms.html">Terms</a>
          <a href="acknowledgements.html">Acknowledgements</a>
          <a href="faq.html">FAQ</a>
```

- [ ] **Step 11: Verify no residual old attribution paths**

```
grep -rn 'attributions.html' .
```

Expected: no matches.

- [ ] **Step 12: Commit**

```
git add help/acknowledgements.html index.html delete-account.html legal/imprint.html legal/privacy.html legal/terms.html help/faq.html
git commit -m "Move attributions to /help/acknowledgements.html"
```

---

## Task 4: Move `delete-account.html` → `account/delete.html`

**Files:**
- Move: `delete-account.html` → `account/delete.html`
- Modify: `index.html`, `legal/imprint.html`, `legal/privacy.html`, `legal/terms.html`, `help/faq.html`, `help/acknowledgements.html`, and the moved `account/delete.html` itself (paths + footer)

- [ ] **Step 1: Snapshot occurrences**

```
grep -rn 'delete-account.html' .
```

Expected: hits in all 6 other HTML files' footers plus the file's self-link.

- [ ] **Step 2: Move the file**

```
mkdir -p account
git mv delete-account.html account/delete.html
```

- [ ] **Step 3: Verify the move**

```
ls account/
ls -la delete-account.html 2>&1 || echo "delete-account.html removed (expected)"
```

- [ ] **Step 4: Update `index.html`**

Find:
```html
          <a href="delete-account.html">Delete account</a>
```
Replace with:
```html
          <a href="account/delete.html">Delete account</a>
```

- [ ] **Step 5: Update `legal/imprint.html`**

Find:
```html
          <a href="../delete-account.html">Delete account</a>
```
Replace with:
```html
          <a href="../account/delete.html">Delete account</a>
```

- [ ] **Step 6: Update `legal/privacy.html`**

Find:
```html
          <a href="../delete-account.html">Delete account</a>
```
Replace with:
```html
          <a href="../account/delete.html">Delete account</a>
```

- [ ] **Step 7: Update `legal/terms.html`**

Find:
```html
          <a href="../delete-account.html">Delete account</a>
```
Replace with:
```html
          <a href="../account/delete.html">Delete account</a>
```

- [ ] **Step 8: Update `help/faq.html`**

Find:
```html
          <a href="../delete-account.html">Delete account</a>
```
Replace with:
```html
          <a href="../account/delete.html">Delete account</a>
```

- [ ] **Step 9: Update `help/acknowledgements.html`**

Find:
```html
          <a href="../delete-account.html">Delete account</a>
```
Replace with:
```html
          <a href="../account/delete.html">Delete account</a>
```

- [ ] **Step 10: Update outgoing paths inside `account/delete.html`**

The moved file currently has `assets/`, `legal/`, etc. paths assuming root depth. Update to depth-1 prefixes.

Find:
```html
    <link rel="icon" href="favicon.ico" sizes="any" />
    <link rel="icon" type="image/png" sizes="32x32" href="favicon-32.png" />
    <link rel="apple-touch-icon" href="apple-touch-icon.png" />

    <link rel="stylesheet" href="assets/styles.css" />
```
Replace with:
```html
    <link rel="icon" href="../favicon.ico" sizes="any" />
    <link rel="icon" type="image/png" sizes="32x32" href="../favicon-32.png" />
    <link rel="apple-touch-icon" href="../apple-touch-icon.png" />

    <link rel="stylesheet" href="../assets/styles.css" />
```

Find:
```html
      <a href="./" class="brand">
        <img src="assets/logo.png" alt="" />
```
Replace with:
```html
      <a href="../" class="brand">
        <img src="../assets/logo.png" alt="" />
```

Find:
```html
      <a href="./" class="back-link">← Home</a>
```
Replace with:
```html
      <a href="../" class="back-link">← Home</a>
```

Find (the body link to privacy):
```html
        <a href="legal/privacy.html">privacy policy</a>
```
Replace with:
```html
        <a href="../legal/privacy.html">privacy policy</a>
```

Find (the bottom "see also" line):
```html
        See also: <a href="legal/imprint.html">imprint</a> ·
        <a href="legal/privacy.html">privacy policy</a>.
```
Replace with:
```html
        See also: <a href="../legal/imprint.html">imprint</a> ·
        <a href="../legal/privacy.html">privacy policy</a>.
```

Find (the footer block, currently with root-relative paths):
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="legal/imprint.html">Imprint</a>
          <a href="legal/privacy.html">Privacy</a>
          <a href="legal/terms.html">Terms</a>
          <a href="help/acknowledgements.html">Acknowledgements</a>
          <a href="help/faq.html">FAQ</a>
          <a href="delete-account.html">Delete account</a>
        </nav>
```
Replace with:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="../legal/imprint.html">Imprint</a>
          <a href="../legal/privacy.html">Privacy</a>
          <a href="../legal/terms.html">Terms</a>
          <a href="../help/acknowledgements.html">Acknowledgements</a>
          <a href="../help/faq.html">FAQ</a>
          <a href="delete.html">Delete account</a>
        </nav>
```

- [ ] **Step 11: Verify no residual root-pointing links from `account/delete.html`**

```
grep -n 'href="legal/\|href="help/\|href="assets/\|href="favicon\|src="assets/\|href="./" class="brand"\|href="./" class="back-link"' account/delete.html
```

Expected: no matches.

- [ ] **Step 12: Verify no residual old delete-account paths anywhere**

```
grep -rn 'delete-account.html' .
```

Expected: no matches (Task 7 will recreate `delete-account.html` as a stub).

- [ ] **Step 13: Commit**

```
git add account/delete.html index.html legal/imprint.html legal/privacy.html legal/terms.html help/faq.html help/acknowledgements.html
git commit -m "Move delete-account to /account/delete.html"
```

---

## Task 5: Add body-link integrations (imprint → acknowledgements, privacy → delete)

**Files:**
- Modify: `legal/imprint.html`, `legal/privacy.html`

- [ ] **Step 1: Add the *Third-party software* section to `legal/imprint.html`**

Find:
```html
      <p class="lead">Last updated: 2026-05-05.</p>
    </article>
```
Replace with:
```html
      <h2>Third-party software</h2>
      <p>
        Anyvoc bundles open-source software and reference data under
        various licences. The complete credit notice is available on
        the <a href="../help/acknowledgements.html">acknowledgements page</a>.
      </p>

      <p class="lead">Last updated: 2026-05-05.</p>
    </article>
```

- [ ] **Step 2: Verify the imprint addition**

```
grep -n 'Third-party software\|acknowledgements page' legal/imprint.html
```

Expected: 2 hits — the heading and the link text.

- [ ] **Step 3: Add the deletion link to `legal/privacy.html` Section 5**

Open `legal/privacy.html` and locate the *Deletion* paragraph inside Section 5 (Authentication via Supabase). It currently reads similar to:

```
<strong>Deletion:</strong> You can delete your account at any
time in the app under <em>Settings → Delete Account</em>. ...
```

Find:
```html
        <strong>Deletion:</strong> You can delete your account at any
        time in the app under <em>Settings → Delete Account</em>. All
        account data at Supabase is then irrevocably deleted. Local
        vocabulary data is removed from the device in the same step.
      </p>
```
Replace with:
```html
        <strong>Deletion:</strong> You can delete your account at any
        time in the app under <em>Settings → Delete Account</em>. All
        account data at Supabase is then irrevocably deleted. Local
        vocabulary data is removed from the device in the same step.
        If you cannot reach the in-app flow, you can also
        <a href="../account/delete.html">request account deletion via the web</a>.
      </p>
```

- [ ] **Step 4: Add the deletion link to `legal/privacy.html` Section 12**

Locate the GDPR rights bullet list inside Section 12. Find:
```html
        <li>Erasure of your data (Art. 17)</li>
```
Replace with:
```html
        <li>Erasure of your data (Art. 17) — see also <a href="../account/delete.html">request account deletion</a></li>
```

- [ ] **Step 5: Verify the privacy additions**

```
grep -n 'request account deletion via the web\|request account deletion</a>' legal/privacy.html
```

Expected: 2 hits.

- [ ] **Step 6: Commit**

```
git add legal/imprint.html legal/privacy.html
git commit -m "Add body links: imprint → acknowledgements, privacy → delete"
```

---

## Task 6: Reduce the footer to four items across all seven pages

**Files:**
- Modify: `index.html`, `account/delete.html`, `legal/imprint.html`, `legal/privacy.html`, `legal/terms.html`, `help/faq.html`, `help/acknowledgements.html`

The target footer keeps Imprint · Privacy · Terms · FAQ and drops Acknowledgements + Delete account. The relative prefix differs per page depth.

- [ ] **Step 1: Update footer in `index.html`**

Find:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="legal/imprint.html">Imprint</a>
          <a href="legal/privacy.html">Privacy</a>
          <a href="legal/terms.html">Terms</a>
          <a href="help/acknowledgements.html">Acknowledgements</a>
          <a href="help/faq.html">FAQ</a>
          <a href="account/delete.html">Delete account</a>
        </nav>
```
Replace with:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="legal/imprint.html">Imprint</a>
          <a href="legal/privacy.html">Privacy</a>
          <a href="legal/terms.html">Terms</a>
          <a href="help/faq.html">FAQ</a>
        </nav>
```

- [ ] **Step 2: Update footer in `legal/imprint.html`**

Find:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="imprint.html">Imprint</a>
          <a href="privacy.html">Privacy</a>
          <a href="terms.html">Terms</a>
          <a href="../help/acknowledgements.html">Acknowledgements</a>
          <a href="../help/faq.html">FAQ</a>
          <a href="../account/delete.html">Delete account</a>
        </nav>
```
Replace with:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="imprint.html">Imprint</a>
          <a href="privacy.html">Privacy</a>
          <a href="terms.html">Terms</a>
          <a href="../help/faq.html">FAQ</a>
        </nav>
```

- [ ] **Step 3: Update footer in `legal/privacy.html`**

Find:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="imprint.html">Imprint</a>
          <a href="privacy.html">Privacy</a>
          <a href="terms.html">Terms</a>
          <a href="../help/acknowledgements.html">Acknowledgements</a>
          <a href="../help/faq.html">FAQ</a>
          <a href="../account/delete.html">Delete account</a>
        </nav>
```
Replace with:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="imprint.html">Imprint</a>
          <a href="privacy.html">Privacy</a>
          <a href="terms.html">Terms</a>
          <a href="../help/faq.html">FAQ</a>
        </nav>
```

- [ ] **Step 4: Update footer in `legal/terms.html`**

Find:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="imprint.html">Imprint</a>
          <a href="privacy.html">Privacy</a>
          <a href="terms.html">Terms</a>
          <a href="../help/acknowledgements.html">Acknowledgements</a>
          <a href="../help/faq.html">FAQ</a>
          <a href="../account/delete.html">Delete account</a>
        </nav>
```
Replace with:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="imprint.html">Imprint</a>
          <a href="privacy.html">Privacy</a>
          <a href="terms.html">Terms</a>
          <a href="../help/faq.html">FAQ</a>
        </nav>
```

- [ ] **Step 5: Update footer in `help/faq.html`**

Find:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="../legal/imprint.html">Imprint</a>
          <a href="../legal/privacy.html">Privacy</a>
          <a href="../legal/terms.html">Terms</a>
          <a href="acknowledgements.html">Acknowledgements</a>
          <a href="faq.html">FAQ</a>
          <a href="../account/delete.html">Delete account</a>
        </nav>
```
Replace with:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="../legal/imprint.html">Imprint</a>
          <a href="../legal/privacy.html">Privacy</a>
          <a href="../legal/terms.html">Terms</a>
          <a href="faq.html">FAQ</a>
        </nav>
```

- [ ] **Step 6: Update footer in `help/acknowledgements.html`**

Find:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="../legal/imprint.html">Imprint</a>
          <a href="../legal/privacy.html">Privacy</a>
          <a href="../legal/terms.html">Terms</a>
          <a href="acknowledgements.html">Acknowledgements</a>
          <a href="faq.html">FAQ</a>
          <a href="../account/delete.html">Delete account</a>
        </nav>
```
Replace with:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="../legal/imprint.html">Imprint</a>
          <a href="../legal/privacy.html">Privacy</a>
          <a href="../legal/terms.html">Terms</a>
          <a href="faq.html">FAQ</a>
        </nav>
```

- [ ] **Step 7: Update footer in `account/delete.html`**

Find:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="../legal/imprint.html">Imprint</a>
          <a href="../legal/privacy.html">Privacy</a>
          <a href="../legal/terms.html">Terms</a>
          <a href="../help/acknowledgements.html">Acknowledgements</a>
          <a href="../help/faq.html">FAQ</a>
          <a href="delete.html">Delete account</a>
        </nav>
```
Replace with:
```html
        <nav class="footer-links" aria-label="Legal">
          <a href="../legal/imprint.html">Imprint</a>
          <a href="../legal/privacy.html">Privacy</a>
          <a href="../legal/terms.html">Terms</a>
          <a href="../help/faq.html">FAQ</a>
        </nav>
```

- [ ] **Step 8: Verify**

```
grep -rn 'Acknowledgements\|Delete account' --include='*.html' . | grep -v 'docs/'
```

Expected: only the body-link mentions inside `legal/imprint.html` (the *Third-party software* section's link text uses lowercase "acknowledgements page", so no Acknowledgements-with-capital remains in any footer) and the `<h1>Delete your Anyvoc account</h1>` and `<h1>Acknowledgements</h1>` heads of the dedicated pages. No footer hits.

- [ ] **Step 9: Commit**

```
git add index.html account/delete.html legal/imprint.html legal/privacy.html legal/terms.html help/faq.html help/acknowledgements.html
git commit -m "Reduce footer to 4 items (Imprint · Privacy · Terms · FAQ)"
```

---

## Task 7: Create backward-compatibility redirect stubs

**Files:**
- Create: `legal/faq.html` (redirect stub)
- Create: `legal/attributions.html` (redirect stub)
- Create: `delete-account.html` (redirect stub)

These three files are recreated at their pre-restructure URLs so installed-app users on old binaries — which still hit `https://anyvoc.eu/legal/faq.html` etc. — get auto-forwarded to the new locations.

- [ ] **Step 1: Create `legal/faq.html` stub**

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Moved — Anyvoc</title>
    <meta http-equiv="refresh" content="0; url=/help/faq.html" />
    <link rel="canonical" href="https://anyvoc.eu/help/faq.html" />
  </head>
  <body>
    <p>This page has moved to <a href="/help/faq.html">/help/faq.html</a>.</p>
  </body>
</html>
```

- [ ] **Step 2: Create `legal/attributions.html` stub**

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Moved — Anyvoc</title>
    <meta http-equiv="refresh" content="0; url=/help/acknowledgements.html" />
    <link rel="canonical" href="https://anyvoc.eu/help/acknowledgements.html" />
  </head>
  <body>
    <p>This page has moved to <a href="/help/acknowledgements.html">/help/acknowledgements.html</a>.</p>
  </body>
</html>
```

- [ ] **Step 3: Create `delete-account.html` stub**

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Moved — Anyvoc</title>
    <meta http-equiv="refresh" content="0; url=/account/delete.html" />
    <link rel="canonical" href="https://anyvoc.eu/account/delete.html" />
  </head>
  <body>
    <p>This page has moved to <a href="/account/delete.html">/account/delete.html</a>.</p>
  </body>
</html>
```

- [ ] **Step 4: Verify all three stubs exist and contain a refresh directive**

```
ls legal/faq.html legal/attributions.html delete-account.html
grep -l 'http-equiv="refresh"' legal/faq.html legal/attributions.html delete-account.html
```

Expected: 3 files listed twice (once by `ls`, once by `grep -l`).

- [ ] **Step 5: Commit**

```
git add legal/faq.html legal/attributions.html delete-account.html
git commit -m "Add backward-compat redirect stubs for moved URLs"
```

---

## Task 8: Push to `origin/main`

- [ ] **Step 1: Sanity-check the working tree**

```
git status
git log --oneline origin/main..HEAD
```

Expected: clean working tree; 7 new commits ahead of `origin/main` (Tasks 1–7).

- [ ] **Step 2: Pull-rebase to absorb any concurrent CNAME or other tweaks**

```
git pull --rebase origin main
```

- [ ] **Step 3: Push**

```
git push origin main
```

- [ ] **Step 4: Verify GitHub Pages deployment after a few minutes**

```
curl -sI https://anyvoc.eu/index.html | head -1
curl -sI https://anyvoc.eu/legal/imprint.html | head -1
curl -sI https://anyvoc.eu/help/faq.html | head -1
curl -sI https://anyvoc.eu/help/acknowledgements.html | head -1
curl -sI https://anyvoc.eu/account/delete.html | head -1
curl -sI https://anyvoc.eu/legal/faq.html | head -1
curl -sI https://anyvoc.eu/legal/attributions.html | head -1
curl -sI https://anyvoc.eu/delete-account.html | head -1
curl -sI https://anyvoc.eu/assets/styles.css | head -1
curl -sI https://anyvoc.eu/assets/logo.png | head -1
curl -sI https://anyvoc.eu/assets/fonts/inter-latin.woff2 | head -1
```

Expected: every line `HTTP/2 200`.

- [ ] **Step 5: Visual smoke test**

Open in a browser:
- `https://anyvoc.eu/`
- `https://anyvoc.eu/legal/imprint.html`
- `https://anyvoc.eu/legal/privacy.html`
- `https://anyvoc.eu/legal/terms.html`
- `https://anyvoc.eu/help/faq.html`
- `https://anyvoc.eu/help/acknowledgements.html`
- `https://anyvoc.eu/account/delete.html`

For each: confirm logo renders, fonts loaded (Inter is visible, not the system fallback), footer shows exactly four items, all four footer links resolve to 200.

Open `https://anyvoc.eu/legal/imprint.html` and click the *acknowledgements page* link in the *Third-party software* section. Confirm `help/acknowledgements.html` loads.

Open `https://anyvoc.eu/legal/privacy.html` and verify both inline deletion links (Section 5 *Deletion* paragraph and Section 12 *Erasure* bullet) resolve to `account/delete.html`.

Visit each redirect stub URL — `/legal/faq.html`, `/legal/attributions.html`, `/delete-account.html` — and confirm the browser address bar lands on the new location within ~0 ms.

---

## App-Side Follow-up (separate repo `Anyvoc/`, separate sprint)

These changes happen in `E:\dev\Claude-React\Anyvoc\`, not this worktree. Documented here so coordination is clear; execute as part of the next app release prep.

### Update `constants/legal.ts`

Replace the entire file contents with:

```ts
// External URLs for legal + support documents, hosted on anyvoc.eu as
// static HTML so they can be updated without an app release. Both stores
// require these URLs to be present during submission and to remain
// reachable for the lifetime of the app. Pages live in three folders:
// /legal/ (mandated documents), /help/ (FAQ + acknowledgements), and
// /account/ (account-management actions).

const HOST = 'https://anyvoc.eu';

export const PRIVACY_URL          = `${HOST}/legal/privacy.html`;
export const TERMS_URL            = `${HOST}/legal/terms.html`;
export const IMPRINT_URL          = `${HOST}/legal/imprint.html`;
export const FAQ_URL              = `${HOST}/help/faq.html`;
export const ACKNOWLEDGEMENTS_URL = `${HOST}/help/acknowledgements.html`;
export const SUPPORT_URL = 'mailto:feedback@anyvoc.eu';
export const FEEDBACK_EMAIL = 'feedback@anyvoc.eu';

// EU Online Dispute Resolution platform — required link in the imprint
// for any online business with consumer contracts inside the EU. Static
// URL maintained by the European Commission. Referenced from the
// imprint.html page on anyvoc.eu and surfaced from in-app links where
// useful (currently informational only).
export const ODR_URL = 'https://ec.europa.eu/consumers/odr/';

// Web URL for account-deletion requests. Required by Google Play store
// listings since 2024 (alongside the in-app delete flow handled by the
// Supabase Edge Function — see app/settings.tsx handleDeleteAccount).
// The page itself lives on anyvoc.eu (separate codebase) and offers a
// mailto / form route; the in-app flow remains the primary path.
export const ACCOUNT_DELETION_URL = `${HOST}/account/delete.html`;
```

### Rename `ATTRIBUTIONS_URL` → `ACKNOWLEDGEMENTS_URL`

Use the TypeScript compile step to find every call site:

```
cd /e/dev/Claude-React/Anyvoc
npx tsc --noEmit 2>&1 | grep ATTRIBUTIONS_URL
```

For each reported call site, rename the import and the usage. Re-run `npx tsc --noEmit` until clean.

### Update `docs/release-checklist-eu.md`

Search for the old URLs and replace:
- `legal/faq.html` → `help/faq.html`
- `legal/attributions.html` → `help/acknowledgements.html`
- `/delete-account.html` → `/account/delete.html`

Also update the `### 3.4 New page: attributions.html` heading and surrounding prose to reflect the new path under `/help/`.

### Update Play Console + App Store Connect

In the next app submission cycle:
- Play Console → App Content → Account Deletion → URL: change to `https://anyvoc.eu/account/delete.html`.
- App Store Connect: no dedicated field for the deletion URL, but the privacy policy already links to it; no separate action required.

Test in-app: build, tap Settings → FAQ / Settings → Acknowledgements / Settings → Delete Account; each opens the correct new URL.
