# Website Restructure — Design

Date: 2026-05-07
Status: Approved (pending user spec review)

## Context

The static website at `anyvoc.eu` has accumulated structural drift:

- **Root clutter:** 10 entries at the repo root (HTML pages, CSS, logo,
  three favicons, fonts folder, two content folders, CNAME).
- **`/legal/` is overloaded:** alongside the truly legal documents
  (`imprint.html`, `privacy.html`, `terms.html`) it hosts
  `attributions.html` (a credits / acknowledgements page driven by
  CC BY 4.0 and Apache 2.0 attribution duties — not a legal document
  per se) and `faq.html` (user-facing help — not legal at all).
- **Footer overgrowth:** the bottom navigation has reached six items
  (Imprint · Privacy · Terms · Acknowledgements · FAQ · Delete
  account), which crowds the site and dilutes the importance of the
  three actually legally mandated pages.

The Anyvoc app code (separate repo) hardcodes a handful of website
URLs in `constants/legal.ts` and ships them to the App Store / Play
listings; any URL change therefore requires an app release.

## Goals

1. Cut root clutter so only files that conventionally belong at the
   site root remain.
2. Restrict `/legal/` to the three documents German / EU law
   actually mandates: imprint, privacy policy, terms.
3. Trim the footer to four items (Imprint · Privacy · Terms · FAQ)
   and integrate the dropped two via natural body links from related
   legal pages.
4. Keep installed-app users on old binaries functional for a
   transition window via meta-refresh redirect stubs at the old
   URLs.

## Non-goals

- Visual / typographic redesign of the pages themselves.
- Translating any page back to German or any third language.
- Changing the page content beyond the footer reorg, the body link
  additions, and the file relocations.

## Constraints

- **App URL contract.** The following URLs are referenced from the
  app codebase (`constants/legal.ts`) and will be changed in
  lockstep with this restructure:
  - `https://anyvoc.eu/legal/faq.html` → moves
  - `https://anyvoc.eu/legal/attributions.html` → moves & renames
  - `https://anyvoc.eu/delete-account.html` → moves
  Pages that remain (privacy, terms, imprint) keep their URLs.
- **Hosting.** GitHub Pages, custom domain `anyvoc.eu` via CNAME.
  No server-side rewriting available — file-system layout *is* the
  URL layout.
- **Browser conventions.** `/favicon.ico` must remain at root for
  legacy lookup paths (RSS readers, link-preview bots, older
  browsers fetch `/favicon.ico` directly without parsing HTML).

## Final directory structure

```
/
├── CNAME                        unchanged (GitHub Pages)
├── index.html                   landing page
├── favicon.ico                  stays at root (browser convention)
├── favicon-32.png               stays at root (consistency with .ico)
├── apple-touch-icon.png         stays at root (consistency)
│
├── assets/                      NEW — static media bundle
│   ├── logo.png
│   ├── styles.css
│   └── fonts/
│       ├── inter-latin.woff2
│       └── inter-latin-ext.woff2
│
├── legal/                       legally-mandated documents only
│   ├── imprint.html
│   ├── privacy.html
│   └── terms.html
│
├── help/                        NEW — non-legal information
│   ├── faq.html
│   └── acknowledgements.html    (renamed from attributions.html)
│
└── account/                     NEW — account-management actions
    └── delete.html
```

## Footer + body link integration

The footer on every page reduces from six items to four:

```
Imprint · Privacy · Terms · FAQ
```

Acknowledgements and Delete Account drop out of the footer and
become reachable via in-page body links from the most semantically
adjacent legal page:

- **Acknowledgements.** A new short section at the bottom of
  `legal/imprint.html` titled *Third-party software* points to
  `../help/acknowledgements.html`. Rationale: the imprint already
  discloses entity / platform / hosting; software credits are
  another disclosure of the technical stack and fit naturally
  alongside.
- **Delete Account.** Two inline links inside `legal/privacy.html`:
  - In Section 5 (*Authentication via Supabase*), at the existing
    *Deletion* paragraph — links to `../account/delete.html`.
  - In Section 12 (*Your rights under GDPR*), appended to the
    bullet *Erasure of your data (Art. 17)* — links to
    `../account/delete.html`.
  Rationale: store reviewers (Apple / Google) typically open the
  privacy policy when auditing; placing the deletion link in the
  two GDPR-relevant paragraphs ensures it is found.

In-app discoverability is unaffected — the app code links to the
URLs directly via `ACKNOWLEDGEMENTS_URL` and `ACCOUNT_DELETION_URL`,
not through the website footer.

## File operations

Performed via `git mv` so history is preserved:

| From                                   | To                                     |
|----------------------------------------|----------------------------------------|
| `logo.png`                             | `assets/logo.png`                      |
| `styles.css`                           | `assets/styles.css`                    |
| `fonts/inter-latin.woff2`              | `assets/fonts/inter-latin.woff2`       |
| `fonts/inter-latin-ext.woff2`          | `assets/fonts/inter-latin-ext.woff2`   |
| `delete-account.html`                  | `account/delete.html`                  |
| `legal/faq.html`                       | `help/faq.html`                        |
| `legal/attributions.html`              | `help/acknowledgements.html`           |

The empty `fonts/` directory is removed after the moves.

## Path adjustments inside HTML files

Each HTML file's relative paths are rewritten according to its new
depth from the root:

| Page                          | Stylesheet                | Logo                  | Favicons          | Footer links to legal/ help/ account/ |
|-------------------------------|---------------------------|-----------------------|-------------------|---------------------------------------|
| `/index.html`                 | `assets/styles.css`       | `assets/logo.png`     | `favicon.ico` etc.| `legal/...`, `help/...`, `account/...` |
| `/legal/imprint.html`         | `../assets/styles.css`    | `../assets/logo.png`  | `../favicon.ico`  | siblings via bare name; `../help/...`, `../account/...` |
| `/legal/privacy.html`         | `../assets/styles.css`    | `../assets/logo.png`  | `../favicon.ico`  | same pattern |
| `/legal/terms.html`           | `../assets/styles.css`    | `../assets/logo.png`  | `../favicon.ico`  | same pattern |
| `/help/faq.html`              | `../assets/styles.css`    | `../assets/logo.png`  | `../favicon.ico`  | `../legal/...`, sibling; `../account/...` |
| `/help/acknowledgements.html` | `../assets/styles.css`    | `../assets/logo.png`  | `../favicon.ico`  | same pattern |
| `/account/delete.html`        | `../assets/styles.css`    | `../assets/logo.png`  | `../favicon.ico`  | `../legal/...`, `../help/...` |

`assets/styles.css` declares `@font-face src: url("fonts/inter-*.woff2")`
— the path is relative to the CSS file, and `fonts/` lives next to
`styles.css` inside `assets/`, so this stays valid without change.

The footer markup becomes identical in shape across all seven pages
(only the relative prefix differs):

```html
<nav class="footer-links" aria-label="Legal">
  <a href="<prefix>legal/imprint.html">Imprint</a>
  <a href="<prefix>legal/privacy.html">Privacy</a>
  <a href="<prefix>legal/terms.html">Terms</a>
  <a href="<prefix>help/faq.html">FAQ</a>
</nav>
```

## Backward-compatibility redirect stubs

Three meta-refresh stubs are placed at the now-vacated URLs so
in-the-wild app installs and any external bookmarks reach the new
locations:

- `/legal/faq.html` → redirects to `/help/faq.html`
- `/legal/attributions.html` → redirects to `/help/acknowledgements.html`
- `/delete-account.html` → redirects to `/account/delete.html`

Each stub is a tiny HTML file:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Moved — Anyvoc</title>
  <meta http-equiv="refresh" content="0; url=<new-url>" />
  <link rel="canonical" href="https://anyvoc.eu<new-url>" />
</head>
<body>
  <p>This page has moved to <a href="<new-url>"><new-url></a>.</p>
</body>
</html>
```

GitHub Pages cannot return HTTP 301; meta refresh + canonical link
is the standard substitute. The stubs can be deleted after a
review window of roughly 6–12 months, once telemetry shows app
installs on the legacy URL set are negligible.

## App-side changes (separate repo `Anyvoc/`)

`constants/legal.ts`:

```ts
const HOST = 'https://anyvoc.eu';

export const PRIVACY_URL          = `${HOST}/legal/privacy.html`;        // unchanged
export const TERMS_URL            = `${HOST}/legal/terms.html`;          // unchanged
export const IMPRINT_URL          = `${HOST}/legal/imprint.html`;        // unchanged
export const FAQ_URL              = `${HOST}/help/faq.html`;             // changed
export const ACKNOWLEDGEMENTS_URL = `${HOST}/help/acknowledgements.html`; // renamed + changed
export const ACCOUNT_DELETION_URL = `${HOST}/account/delete.html`;       // changed
```

The previous `LEGAL_HOST` constant collapses into a single `HOST`,
and the awkward `LEGAL_HOST.replace('/legal', '')` workaround for
the deletion URL goes away.

Call sites of the renamed `ATTRIBUTIONS_URL` constant — easy to
locate via TypeScript compile errors or a project-wide search —
are updated to `ACKNOWLEDGEMENTS_URL` in lockstep.

`docs/release-checklist-eu.md` is updated wherever it references
the old URLs (`legal/faq.html`, `legal/attributions.html`,
`/delete-account.html`).

The Play Console and App Store Connect listings carry the
deletion URL; both are updated to the new
`https://anyvoc.eu/account/delete.html` value before or
simultaneously with the next app submission.

## Verification

After the website changes are pushed:

1. Open each of the seven pages in a browser; confirm styles, logo,
   fonts, and favicons load (no 404s in network panel).
2. Click each of the four footer items from each page; confirm no
   404, correct target page reached.
3. Open `legal/imprint.html`; click the *Third-party software*
   link; confirm `help/acknowledgements.html` loads.
4. Open `legal/privacy.html`; click both inline deletion links
   (Section 5 and Section 12); confirm `account/delete.html` loads.
5. Request the three legacy URLs (`/legal/faq.html`,
   `/legal/attributions.html`, `/delete-account.html`); confirm
   each redirects to its new location within ~0 ms.
6. `curl -I` each canonical URL; expect HTTP 200.

After the app changes are merged (separate repo, separate sprint):

7. Build the app locally; tap *Settings → FAQ*, *Settings →
   Acknowledgements*, *Settings → Delete Account* — each opens its
   new URL.
8. Update the Play Console / App Store Connect deletion URL field.

## Risks and trade-offs

- **Two-repo coordination.** The website and app changes need to be
  released roughly together; if the website ships first, old app
  installs benefit from the redirect stubs, so the ordering is
  forgiving. The reverse (app first) is also fine because the new
  app constants only point at URLs that already exist as soon as
  the website is shipped.
- **Redirect stubs are not 301.** Search engines understand
  meta-refresh + canonical link as a soft redirect, not a permanent
  one. Fine for our scale; a non-issue for a coming-soon site.
- **Constant rename `ATTRIBUTIONS_URL` → `ACKNOWLEDGEMENTS_URL`.**
  Pure code churn, but keeps the website file name aligned with the
  user-facing label everywhere and avoids future confusion.
