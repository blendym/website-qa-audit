# Website QA Audit Report — `<DOMAIN>`

- **Target URL**: `<URL>`
- **Audit Date**: `<YYYY-MM-DD>`
- **Mode**: `<Quick | Deep | Re-Audit>`
- **Viewports Checked**: `Desktop (1440x900)`, `Laptop/Tablet (1280x720 / 768x1024)`, `Mobile (390x844)`
- **Optional Modules**: `<None | Performance, WCAG, GDPR, Passive Security>`
- **Scope Covered**: `<List of audited routes/templates>`

## Executive Summary

| Priority / Category | Count | Key Highlight |
| :--- | :---: | :--- |
| Critical (Functional, Contact, Legal) | `0` | ... |
| UI/UX & Visual | `0` | ... |
| Content & Copy | `0` | ... |
| Technical & SEO | `0` | ... |
| Optional Modules | `0` | ... |

---

## 1. Critical (Functional, Contact, Legal)

1. **[Issue Title]** (`<URL>`, `<Viewport / Section>`)
   - **Observed**: Exact behavior, broken selector, or faulty data.
   - **Expected**: What should happen or exact corrected value.

## 2. UI/UX & Visual Issues

1. **[Issue Title]** (`<URL>`, `<Viewport / Selector>`)
   - **Observed**: Visual defect, contrast issue, clipping, or layout break.
   - **Evidence**: `<Verified screenshot path if captured>`
   - **Recommendation**: Concrete CSS/UX fix.

## 3. Content & Copy Issues

- **`<URL>`**
  - Exact quote: `"..."` -> Suggested fix: `"..."`

## 4. Technical & SEO

- **Indexing / 404 / Meta / Headings / DOM**: Concrete findings from `Probe 1`, `Probe 3`, and console/network checks.

## 5. Optional Modules *(include only requested modules)*

### Performance / Core Web Vitals
### WCAG Accessibility
### GDPR / Cookies
### Passive Security

---

## 6. Quick Wins / Developer Checklist

- [ ] `<High-priority fix 1 — URL / selector>`
- [ ] `<High-priority fix 2 — URL / selector>`

## 7. What Works Well

- Clean HTTP `200` responses, zero console errors, responsive checks, working interactive controls.

---

## Re-Audit Status Table *(use only in `re-audit` mode)*

| ID | Original Finding | Status | Verification Note |
| :--- | :--- | :---: | :--- |
| `C-1` | ... | `FIXED` / `OPEN` / `PARTIAL` / `REGRESSION` | Verified at `<URL>` on `<YYYY-MM-DD>` |
