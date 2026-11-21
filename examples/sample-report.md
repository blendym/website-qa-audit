# Sample QA Audit Report — `acme-hardware.example` *(Anonymized Example)*

- **Target URL**: `https://acme-hardware.example/` (fictional sample company: **Acme Hardware sp. z o.o.**)
- **Audit Date**: `2026-11-20`
- **Mode**: `Deep`
- **Viewports Checked**: Desktop (`1440x900`), Laptop (`1280x720`), Mobile (`390x844`)
- **Optional Modules**: `Performance`, `GDPR / Cookies`
- **Scope Covered**: `/`, `/products/`, `/pricing/`, `/docs/`, `/blog/`, `/contact/`, `/privacy/`, `/robots.txt`, `/qa-404-check-nonexistent`

## Executive Summary

| Priority / Category | Count | Key Highlight |
| :--- | :---: | :--- |
| Critical (Functional, Contact, Legal) | 5 | `noindex` left on `/docs/`, unstyled 404 server error, misspelled `mailto:`, invalid NIP checksum & 11-digit KRS, placeholder entity in Privacy Policy |
| UI/UX & Visual | 4 | Pricing table horizontal overflow on mobile, navbar wrapping at `1280x720`, low banner contrast, clipped newsletter button label |
| Content & Copy | 3 | `Lorem ipsum` leftover on `/blog/`, English validation text on localized form + missing period in `sp. z o.o.`, outdated footer year & price contradiction |
| Technical & SEO | 4 | Missing `<h1>` on `/pricing/`, `og:image` returning `404`, inconsistent `tel:` prefix, empty `<img src="">` tags on `/products/` |
| Optional Modules (`performance`, `gdpr`) | 2 | `2150 KB` hero PNG with `loading="lazy"` (`LCP = 3850 ms`), GA4 tracking active before cookie consent |

---

## 1. Critical (Functional, Contact, Legal)

1. **Accidental `noindex, nofollow` Left From Staging** (`/docs/`)
   - **Observed**: `Probe 1` detected `<meta name="robots" content="noindex, nofollow">` on the documentation landing page.
   - **Expected**: Remove `noindex, nofollow` before launch so search engines can index `/docs/`.

2. **Missing Custom 404 Page** (`/qa-404-check-nonexistent`)
   - **Observed**: Requesting a nonexistent route renders a raw, unstyled `404 Not Found` web-server error with no site header, search bar, or link back home.
   - **Expected**: Return HTTP `404` with a styled fallback template and a clear link back to `/`.

3. **Broken Support `mailto:` Link** (`/contact/`)
   - **Observed**: `Probe 2` flagged a mismatch on the Support card: visible text shows `support@acme-hardware.example`, while `href` points to `mailto:suport@acme-hardware.example` (missing `p`).
   - **Expected**: Update the `href` attribute to `mailto:support@acme-hardware.example`.

4. **Invalid Registration Numbers (NIP Checksum & 11-Digit KRS)** (`/contact/`)
   - **Observed**: `Probe 5` flagged:
     - `NIP 123-456-32-19` — fails modulo-11 checksum (`mod=8`, last digit `9`; valid check digit for prefix `123456321` is `8`).
     - `KRS 00001234567` — 11 digits (Polish KRS numbers must be strictly 10 digits).
   - **Expected**: Verify official registry records and fix both typos.

5. **Outdated Template Entity in Privacy Policy** (`/privacy/`)
   - **Observed**: The Data Controller section still lists **Example Holdings Ltd., 123 Main St** (`legal@template-demo.invalid`) instead of **Acme Hardware sp. z o.o.** (`privacy@acme-hardware.example`).
   - **Expected**: Replace the boilerplate entity with Acme Hardware sp. z o.o.'s verified legal details and ensure supervisory links use `https://`.

---

## 2. UI/UX & Visual Issues

1. **Horizontal Overflow on Pricing Comparison Table** (`/pricing/`, Mobile `390x844`)
   - **Observed**: `Probe 4` reported `hasHorizontalOverflow: true` (`overflowPx: 64`) caused by `table.pricing-matrix` extending past the right viewport edge (`right: 454`).
   - **Recommendation**: Wrap `table.pricing-matrix` in a scrollable container (`overflow-x: auto; max-width: 100%;`) or stack plan cards vertically under `768px`.

2. **Header Navigation Wraps Into Two Rows** (`/`, Laptop `1280x720`)
   - **Observed**: Top-bar links wrap onto a second line at `1280px` width, causing the sticky header to overlap the top `18px` of the subpage hero title.
   - **Recommendation**: Reduce horizontal nav gap between `1024px–1360px` or trigger the compact menu earlier.

3. **Low Contrast on Announcement Banner** (`/`, Desktop & Mobile)
   - **Observed**: Top announcement bar text renders in `#9ca3af` over a light `#e5e7eb` background (insufficient contrast ratio).
   - **Recommendation**: Darken announcement copy to `#1f2937` or use a darker bar background with `#ffffff` text.

4. **Clipped Newsletter Submit Button Label** (`/`, Mobile `390x844`)
   - **Observed**: `Probe 4` flagged `button.newsletter-submit` (`clientWidth: 96`, `scrollWidth: 138`), truncating the localized label inside the rounded button.
   - **Recommendation**: Allow button width to fit content (`width: auto; white-space: nowrap;`) and stack the input/button on narrow screens.

---

## 3. Content & Copy Issues

- **`/blog/`**:
  - Card 3 summary still displays placeholder text: `"Lorem ipsum dolor sit amet, consectetur adipiscing elit..."`.
- **`/contact/`**:
  - Submitting an empty form triggers an untranslated English validation error (`"Please fill out this field."`) on the Polish version of the page.
  - Footer legal name omits the trailing period: `"Acme Hardware sp. z o.o"` -> `"Acme Hardware sp. z o.o."`.
- **`/pricing/`**:
  - Footer displays an outdated copyright year (`© 2023 Acme Hardware`), and the Pro plan is listed as `$49/mo` in the hero card vs `$59/mo` in the FAQ below.

---

## 4. Technical & SEO

- **Missing `<h1>` Landmark**: `/pricing/` has `h1Count: 0` (`Probe 1` — headings begin at `<h2>`).
- **Broken `og:image` Asset (`/`)**: `<meta property="og:image">` points to `https://acme-hardware.example/assets/og-default.png`, which returns HTTP `404`.
- **Inconsistent `tel:` Format**: Header phone link uses `tel:48220000000` (missing `+`), while footer uses `tel:+48220000000` (`Probe 2`).
- **Empty `<img>` Tags**: `Probe 3` found 2 hidden `<img src="" alt="">` elements on `/products/`.

---

## 5. Optional Modules (`performance`, `gdpr`)

### Performance
- **Oversized & Lazy-Loaded Hero PNG (`Probe 3`, `Probe 7`)**: Homepage hero loads a `2150 KB` PNG (`3200x1800` scaled down to `1440x720`) with `loading="lazy"`, producing `LCP = 3850 ms`. Export a properly sized WebP/AVIF (`<300 KB`) and set `fetchpriority="high"` without `loading="lazy"`.

### GDPR / Cookies
- **Pre-Consent Tracking (`Probe 6`)**: Before clicking anything on the consent banner, `googletagmanager.com/gtag/js` has already executed and set `_ga` / `_ga_*` cookies. Block analytics until explicit consent and add a "Reject all" action at equal visual prominence to "Accept all".

---

## 6. Quick Wins / Developer Checklist

- [ ] Remove `<meta name="robots" content="noindex, nofollow">` from `/docs/`
- [ ] Add styled `/404` error template with navigation back to `/`
- [ ] Fix `mailto:suport@acme-hardware.example`, `NIP 123-456-32-19`, and 11-digit `KRS 00001234567` on `/contact/`
- [ ] Update Data Controller in `/privacy/` to `Acme Hardware sp. z o.o.`
- [ ] Wrap `table.pricing-matrix` in `overflow-x: auto` for mobile and fix header wrap at `1280x720`
- [ ] Replace `Lorem ipsum` on `/blog/`, unify `$49/mo` vs `$59/mo` on `/pricing/`, and localize form validation strings
- [ ] Add `<h1>` on `/pricing/`, restore `og-default.png` (`200 OK`), and convert the hero PNG (`2.15 MB`) to WebP (`<300 KB`)

## 7. What Works Well

- Primary storefront routes load with HTTP `200`, zero JavaScript console errors, and `/robots.txt` permits crawling.
- Mobile navigation drawer, product filters, and documentation search respond smoothly across desktop and mobile.
