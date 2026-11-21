# Core Website QA Audit Checklist

Use every category below for **Deep Mode** audits. For **Quick Mode**, focus on items marked **`[Quick]`** plus core console/network checks.

For Polish / EU legal and language specifics, also load [regional/pl-eu.md](./regional/pl-eu.md).

## 1. Critical: Launch Readiness, Functional, Contact & Legal

- **Indexing & Staging Leftovers `[Quick]`**:
  - Run `Probe 1`: verify `<meta name="robots">` and `googlebot` do **not** contain accidental `noindex` or `nofollow`.
  - Check `/robots.txt` for accidental `Disallow: /` or links to staging domains.
- **404 Error Page `[Quick]`**:
  - Visit a guaranteed nonexistent path (e.g. `/qa-404-check-nonexistent`). Verify it returns HTTP `404` and renders a styled error page with working navigation back home (not a raw server error or blank screen).
- **Hero & Primary CTAs `[Quick]`**:
  - Click every hero button/link across **all** carousel slides. Verify `<button>` elements actually navigate, scroll, or open a modal (watch for dead buttons found by `Probe 2`).
- **Navigation & Links `[Quick]`**:
  - Verify top navigation, dropdowns, footer links, social links, and legal links load cleanly (HTTP `200`) and match their destination.
- **Contact Details `[Quick]`**:
  - Run `Probe 2`: compare every `mailto:` href against visible text and domain spelling.
  - Check every `tel:` href for consistent international format (e.g. `+48...` vs missing `+`).
  - Flag mismatches between team member names and email addresses for confirmation.
- **Company & Legal Data `[Quick]`**:
  - Open Privacy Policy / Terms and confirm the **Data Controller** (company name, street address, registration IDs, contact email, domain) matches the entity operating the website (not a sister company, NGO, or template placeholder).
  - Verify registration IDs using regional rules (e.g. [regional/pl-eu.md](./regional/pl-eu.md) + `Probe 5` for NIP/REGON/KRS).
  - Flag outdated `http://` links to external institutions or resources.
- **Interactive Controls & Form Validation UX**:
  - Test filters, search inputs, tabs, accordions, maps, and pagination.
  - Trigger client-side form validation safely (empty submit or obviously invalid email format, without submitting real inquiries/spam) to verify error messages are clear, styled, and written in the site's language.

## 2. UI / UX & Visual Polish

- **Contrast & Legibility `[Quick]`**:
  - Inspect text over photos/overlays on **every** hero slide across desktop and mobile (slide 1 vs slide 2 often differ).
- **Multi-Viewport Responsiveness**:
  - **Desktop (`1440x900`) `[Quick]`**: layout alignment, spacing, sticky headers, footer integrity.
  - **Laptop / Tablet (`1280x720` or `768x1024`)**: check for wrapping navigation items, colliding sticky headers, cramped grids, or cut-off modals.
  - **Mobile (`390x844`) `[Quick]`**: run `Probe 4` to detect horizontal overflow (`scrollWidth > innerWidth`) and clipped filter pills, tabs, buttons, or placeholders; open and test the mobile drawer menu.
- **Imagery & Brand Assets**:
  - Inspect client/partner logo strips for opaque white rectangular boxes on tinted/colored backgrounds.
  - Check for broken, stretched, pixelated, or awkwardly cropped images (`Probe 3`).
  - Check for duplicated photos in adjacent sections (e.g. the same portrait repeated back-to-back).
- **Conversion & UX Coherence**:
  - Verify whether CTA wording (*"Tell us about your project"*, *"Request a quote"*) matches what the destination page provides (e.g. a static contact card with zero `<form>` elements).

## 3. Content, Copy & Localization

- **Typos & Grammar `[Quick]`**: Scan headings, hero slides, intro copy, service pages, `<select>` options, and legal text for typos, missing diacritics, wrong inflection/gender agreement, missing verbs, broken punctuation, or unclosed quotes.
- **Factual Consistency**: Cross-check repeated claims across pages (e.g. years in business vs founding year, consistent legal suffix).
- **Formatting Consistency**: Check capitalization in lists, `<select>` options, team bios, and portfolio titles.
- **Multilingual & Locale Hygiene**:
  - Verify `<html lang>` (`Probe 1`) matches the active page language.
  - Test language switcher links (if present) and check `hreflang` tags.
  - Flag untranslated UI strings, form validation errors, search placeholders, or CMS plugin messages (e.g. English *"Read more"* / *"No results"* on a Polish page).
- **Placeholders & Alt Text**: Flag `Lorem ipsum`, template remnants, and typos in image `alt` attributes.

## 4. Technical & SEO Hygiene

- **Console & Network `[Quick]`**: Record JS console errors, failed requests (`4xx`/`5xx`), broken JSON/API endpoints, missing fonts, or mixed content.
- **Heading Hierarchy `[Quick]`**: Check `<h1>` count on every audited page via `Probe 1` (flag pages with `0` `<h1>` elements or broken heading order).
- **Meta, Social Share & Icons `[Quick]`**:
  - Inspect `<title>`, `<meta name="description">`, `og:title`, `og:description`, and `og:image` (`Probe 1`). Watch for CMS plugins auto-scraping hero buttons or slide captions into descriptions.
  - Verify `og:image` exists and loads cleanly (HTTP `200`).
  - Check for `favicon` and `apple-touch-icon` links.
- **DOM Hygiene**: Flag hidden/empty `<img src="">` tags (`Probe 3`), missing `alt` attributes on meaningful images, and broken accessibility toolbars.
