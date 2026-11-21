# Optional Audit Modules

Run only the modules explicitly requested by the user. Pair manual inspection with the deterministic probes in [browser-probes.md](./browser-probes.md).

## Module 1: Performance (`performance`)

- **Core Web Vitals (run `Probe 7`)**:
  - **LCP (Largest Contentful Paint)**: Identify the LCP element (usually hero image or headline) and flag slow load (`>2500ms`) or lazy-loaded above-the-fold hero images (`loading="lazy"` reported by `Probe 7`).
  - **CLS (Cumulative Layout Shift)**: Flag `clsScore > 0.1` or visible layout jumps during load (images without `width`/`height`, late-loading web fonts, injected banners).
  - **INP / Responsiveness**: Note noticeable lag when opening menus, switching slides, or filtering lists.
- **Image Weights & Formats (run `Probe 3`)**:
  - Inspect network transfer sizes (`transferSizeKB`) for the heaviest images on the homepage and key subpages.
  - Flag oversized images (`>300–500 KB` for hero banners, `>150 KB` for cards/thumbnails), unresized high-res photos (`naturalWidth > 2x renderedWidth`), and legacy PNG/JPG photos where WebP/AVIF is appropriate.
- **Resource Hygiene**:
  - Flag excessive total page weight, unminified large JS/CSS bundles, or obvious render-blocking scripts in `<head>`.

## Module 2: Accessibility (`wcag`)

Evaluate against practical **WCAG 2.1 AA** criteria:
- **Color Contrast**: Check body text (`>=4.5:1`), large text/headings (`>=3:1`), and interactive UI controls/placeholders, especially over hero photos and tinted backgrounds.
- **Keyboard Navigation & Focus**:
  - Tab through header, dropdowns, hero controls, filters, and footer.
  - Verify visible focus outlines (no `outline: none` without a replacement) and no keyboard traps.
  - Check for a working "Skip to content" link.
- **Semantics & Landmarks (run `Probe 1`, `Probe 7`)**:
  - Verify `<html lang>`, `<header>`, `<nav>`, `<main>`, `<footer>`, and logical `<h1>`–`<h6>` hierarchy.
  - Check interactive elements: buttons should be `<button>`, links `<a href>`, not clickable `<div>`/`<span>` without keyboard/ARIA support.
- **Media & Forms (run `Probe 3`, `Probe 7`)**:
  - Meaningful images have descriptive `alt`; purely decorative images use `alt=""`.
  - Icon-only buttons (hamburger menu, slider arrows, close icons) have accessible `aria-label` or screen-reader text (`unlabeledButtons` in `Probe 7`).
  - Form inputs have associated `<label>` or `aria-label` (`unlabeledInputs` in `Probe 7`) and clear error messaging.

## Module 3: GDPR / RODO & Cookies (`gdpr`)

For EU/Polish websites, also apply [regional/pl-eu.md](./regional/pl-eu.md).
- **Pre-Consent Tracking (run `Probe 6` immediately on load)**:
  - Inspect `cookieNames` and `trackingRequests` on initial page load **before** clicking anything on the cookie banner.
  - Flag analytics/marketing scripts (GA4, GTM marketing tags, Meta Pixel, Hotjar, Clarity, TikTok, LinkedIn) firing before explicit consent.
- **Consent Banner & Symmetry**:
  - Check whether a cookie/consent banner appears on first visit when non-essential cookies or third-party trackers exist.
  - Check choice symmetry: is "Reject all" / "Essential only" just as accessible as "Accept all" (flag dark patterns)?
  - Verify whether cookie preferences can be reopened later from the footer.
- **Privacy Policy Completeness**:
  - Verify Data Controller identity (exact legal name, address, registration IDs, contact email) matches the website's actual operator.
  - Check stated legal basis, retention periods, user rights, and HTTPS links to supervisory authorities.

## Module 4: Passive Security (`security`)

Strictly passive, non-intrusive checks only — never run active scans, fuzzing, or exploitation attempts:
- **Transport & Mixed Content**:
  - Verify `https://` works cleanly and `http://` redirects to `https://`.
  - Flag any mixed-content warnings or plain `http://` external links/assets.
- **Response Headers (when visible in network inspection)**:
  - Check presence of `Strict-Transport-Security` (HSTS), `Content-Security-Policy` (CSP), `X-Frame-Options` or `frame-ancestors`, `X-Content-Type-Options: nosniff`, and `Referrer-Policy`.
- **Information Disclosure**:
  - Flag exposed CMS/generator meta tags (e.g. exact WordPress/plugin versions), verbose error messages/stack traces, or exposed source maps in production.
- **External Link Safety (run `Probe 7`)**:
  - Flag external `target="_blank"` links missing `rel="noopener"` or `rel="noreferrer"` (`unsafeBlankLinks` in `Probe 7`).
