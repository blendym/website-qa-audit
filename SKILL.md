---
name: website-qa-audit
description: >-
  Runs a live QA, UX/UI, visual, copywriting, contact/legal, and technical/SEO
  audit of any website in a real browser across desktop, laptop/tablet, and
  mobile viewports. Supports three modes (quick, deep, re-audit), deterministic
  browser JS probes, regional packs (e.g. PL/EU), four optional modules (Core
  Web Vitals / image weights, WCAG accessibility, GDPR/cookies, passive
  security), and Markdown/HTML report export. Use whenever the user asks to
  review a website, find bugs on a URL, run a pre-launch QA check, re-audit
  fixes, or inspect UX, copy, and technical health (e.g. "review this site",
  "find bugs on this URL", "re-audit fixes", "przejrzyj tę stronę", "wyłap
  błędy", "sprawdź poprawki").
---

# Website QA & UX Audit

Inspect a live website in a real browser across desktop, intermediate laptop/tablet, and mobile viewports using deterministic DOM probes and manual visual checks, then deliver an evidence-backed report grouped by priority.

## Modes

Choose the mode based on the user's request. Default to **`deep`** for full review requests unless the user asks for a fast triage or supplies a prior report for re-verification.

### 1. Quick Mode (`quick`)
Fast triage for smoke checks or rapid reviews:
- **Viewports**: Desktop (`1440x900`) and Mobile (`390x844`).
- **Pages**: Homepage (full scroll + hero slides), Contact page, Privacy Policy / legal page, `/robots.txt`, a guaranteed nonexistent URL for 404 check, and a sanity check of top navigation links.
- **Checks**: Broken CTAs/buttons, console/network errors, accidental `noindex` / `Disallow: /`, major visual/contrast defects, mobile horizontal overflow & menu usability, contact details (`mailto:`/`tel:`), company registration & Privacy Policy entity match, obvious copy errors, basic `<h1>` and meta/OG tags, plus core JS probes from [references/browser-probes.md](./references/browser-probes.md).

### 2. Deep Mode (`deep`) — *Default*
Thorough pre-launch or quality audit:
- **Viewports**: Desktop (`1440x900`), Laptop/Tablet (`1280x720` or `768x1024`), and Mobile (`390x844`).
- **Pages**: Homepage, every primary navigation subpage (About, Services/Offer + 1-2 representative detail pages per template, Portfolio/Projects/Map, Team, Contact), all legal pages (Privacy Policy, Cookies, Terms), `/robots.txt`, and a 404 test URL. For very large sites (50+ pages), sample 1-2 representative pages per template type and state the sampled scope clearly.
- **Checks**: Full checklist in [references/checklist.md](./references/checklist.md), deterministic JS probes in [references/browser-probes.md](./references/browser-probes.md), regional checks when applicable (e.g. [references/regional/pl-eu.md](./references/regional/pl-eu.md)), every hero/carousel slide, interactive controls (filters, search, dropdowns, tabs, maps), client-side form validation (tested with empty/invalid input without submitting real data or spam), multilingual consistency, full copy/grammar review, and SEO/social/icon hygiene.

### 3. Re-Audit Mode (`re-audit`)
Fix verification after a previous audit:
- **Inputs**: A prior audit report (pasted inline or referenced by file path) + live URL.
- **Workflow**: Re-verify each previously reported finding in the live browser using the same viewports and probes, plus a quick regression scan of surrounding areas.
- **Output**: A status table (`FIXED`, `OPEN`, `PARTIAL`, `REGRESSION`) with verification notes for every original item, followed by any newly discovered issues.

## Optional Modules

Run these only when requested by the user (e.g. "deep audit + WCAG and performance", "sprawdź też RODO i wagi zdjęć"). Full criteria are in [references/optional-modules.md](./references/optional-modules.md):

- **Performance (`performance`)** — Core Web Vitals (LCP, CLS, INP/responsiveness), image transfer weights & oversizing (`Probe 3`, `Probe 7`), lazy-loaded hero images, render-blocking resources.
- **Accessibility (`wcag`)** — WCAG 2.1 AA essentials: text/UI contrast, keyboard navigation & visible focus states, landmarks/headings hierarchy, ARIA labels, icon-only button labels, form labels, alt text (`Probe 1`, `Probe 7` + manual tabbing).
- **GDPR / Cookies (`gdpr`)** — Consent banner presence & choice symmetry, pre-consent cookies & tracking requests (`Probe 6`), Privacy Policy completeness and controller match.
- **Passive Security (`security`)** — HTTPS redirect & mixed content, observable security headers, exposed CMS versions/stack traces/source maps, `target="_blank"` `rel="noopener"` (`Probe 7`). Strictly passive and non-intrusive.

## Execution Workflow

1. **Use a Real Browser**:
   - Inspect the live rendered site using your browser tool or browser subagent (e.g. Playwright MCP, Chrome DevTools MCP, or your agent's built-in browser tool/subagent).
   - Do **not** substitute `curl`, `wget`, or raw HTML scraping for live rendering and visual inspection.
   - Never bypass CAPTCHAs, login walls, or origin blocks; if blocked or unreachable, report it clearly and stop.
2. **Run Deterministic JS Probes**:
   - Execute the relevant snippets from [references/browser-probes.md](./references/browser-probes.md) in the browser context to gather exact JSON facts (SEO/robots, `mailto:`/`tel:` mismatches, image weights/broken tags, overflow/clipped selectors, NIP/REGON/KRS checksums, pre-consent trackers) instead of guessing from visual scans alone.
3. **Apply Regional Rules When Applicable**:
   - When auditing Polish or EU websites, automatically apply [references/regional/pl-eu.md](./references/regional/pl-eu.md).
4. **Capture Evidence Screenshots**:
   - Capture screenshots of notable visual/UX defects (e.g. unreadable hero contrast, clipped mobile filter pills, wrapping intermediate-viewport navbars, opaque logo boxes) and cite only screenshot paths verified by your runtime environment.
5. **Never Create Spam or Mutate Production Data**:
   - Trigger client-side form validation safely (empty or malformed test values, or inspect validation rules) without submitting real contact, newsletter, or checkout forms.

## Report Format & Export

- **Default Output**: Respond concisely in the **user's language**, grouped by priority (`Critical`, `UI/UX & Visual`, `Content & Copy`, `Technical & SEO`, requested `Optional Modules`, `Quick Wins / Developer Checklist`, and `What Works Well`). Quote exact faulty text, URLs, viewports, and CSS selectors where useful.
- **File Export (on request)**: When the user asks for a shareable file, save either:
  - Markdown report (`audit-<domain>-<yyyy-mm-dd>.md`) using [templates/report-template.md](./templates/report-template.md), or
  - Standalone, print-ready HTML client report (`audit-<domain>-<yyyy-mm-dd>.html`) using [templates/report-template.html](./templates/report-template.html).
- **Reference Output**: See [examples/sample-report.md](./examples/sample-report.md) for expected depth, specificity, and formatting.
