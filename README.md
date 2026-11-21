# website-qa-audit

An AI agent skill for running thorough, real-browser QA, UX/UI, visual, copywriting, contact/legal, and technical/SEO audits of live websites across desktop, laptop/tablet, and mobile viewports.

## Audit Modes

| Mode | Viewports | Scope | Best For |
| :--- | :--- | :--- | :--- |
| **`quick`** | Desktop (`1440x900`), Mobile (`390x844`) | Homepage, Contact, Privacy Policy, `/robots.txt`, 404 check, top nav sanity check | Fast smoke tests & pre-meeting triage |
| **`deep`** *(default)* | Desktop (`1440x900`), Laptop/Tablet (`1280x720` / `768x1024`), Mobile (`390x844`) | Homepage, all primary nav pages, representative detail pages, 404, `/robots.txt`, interactive controls, form validation, full copy & legal review | Pre-launch QA sign-off & agency quality audits |
| **`re-audit`** | Same viewports as original audit | Prior audit findings re-verified against live URL (`FIXED` / `OPEN` / `PARTIAL` / `REGRESSION`) | Verifying developer fixes before deployment |

## Optional Modules

Enable any module on demand in your prompt:

| Module | What It Checks |
| :--- | :--- |
| **`performance`** | Core Web Vitals (`LCP`, `CLS`, responsiveness), image transfer weights & oversizing (`Probe 3`, `Probe 7`), lazy-loaded hero images, render-blocking assets |
| **`wcag`** | Practical WCAG 2.1 AA: text/UI contrast, keyboard tabbing & focus outlines, landmarks & headings, icon-only button labels, form labels, `alt` text |
| **`gdpr`** | Pre-consent cookies & tracking requests (`Probe 6`), consent banner choice symmetry (accept vs reject), Privacy Policy completeness & controller match |
| **`security`** | Strictly passive checks: HTTPS redirect, mixed content, observable security headers (`HSTS`, `CSP`, `X-Frame-Options`), exposed CMS versions/source maps, `target="_blank"` `rel="noopener"` |

## Key Capabilities

- **Deterministic Browser JS Probes** (`references/browser-probes.md`) — 7 copy-free JSON probes checking indexing blockers (`noindex` / `Disallow: /`), `mailto:`/`tel:` mismatches, dead buttons, oversized/empty images, mobile/tablet overflow & clipped CSS selectors, Polish NIP/REGON/KRS checksums, and pre-consent trackers.
- **Launch Readiness & Real-World Edge Cases** — verifies 404 fallback pages, working `og:image` & favicons, intermediate laptop/tablet breakpoints, multilingual consistency (`<html lang>`, untranslated plugin errors), and client-side form validation without sending spam.
- **Modular Regional Packs** (`references/regional/`) — region-specific legal and linguistic rules kept separate from the core checklist (currently includes `pl-eu.md` for Poland/EU; PRs welcome for other regions such as `de.md`, `uk.md`, `us.md`).
- **Shareable Report Templates** (`templates/`) — output concisely in chat or export as Markdown (`report-template.md`) or print-ready HTML (`report-template.html`).
- **Agent-Portable** — follows the open `SKILL.md` standard; works with Claude Code, Codex, Hermes Agent, Antigravity, Cursor, and any other agent with browser access (Playwright MCP, Chrome DevTools MCP, or built-in browser tools).

## Installation

### Using `skills` CLI

```bash
npx skills add blendym/website-qa-audit
```

### Manual Installation

Clone or copy this repository into the skills directory for your agent:

| Agent | Global Path | Workspace / Project Path |
| :--- | :--- | :--- |
| **Universal / Cross-Agent** | `~/.agents/skills/website-qa-audit` | `<project>/.agents/skills/website-qa-audit` |
| **Claude Code** | `~/.claude/skills/website-qa-audit` | `<project>/.claude/skills/website-qa-audit` |
| **Codex** | `~/.codex/skills/website-qa-audit` | `<project>/.codex/skills/website-qa-audit` |
| **Hermes Agent** | `~/.hermes/skills/website-qa-audit` | *or add `~/.agents/skills` to `skills.external_dirs` in `~/.hermes/config.yaml`* |
| **Antigravity** | `~/.gemini/config/skills/website-qa-audit` | `<project>/.agents/skills/website-qa-audit` |
| **Cursor** | `~/.cursor/skills/website-qa-audit` | `<project>/.cursor/skills/website-qa-audit` |

> **Prerequisite:** Your agent needs a live browser tool (e.g. Playwright MCP, Chrome DevTools MCP, or a built-in browser tool).

## Example Prompts

- **Deep audit (default)**:
  > Review `https://example.com/` and report any bugs, UX/visual defects, copy errors, or technical issues to fix before launch.
- **Quick audit**:
  > Run a quick QA audit on `https://example.com/`.
- **Deep audit with optional modules + HTML export**:
  > Run a deep audit on `https://example.com/` including `wcag`, `gdpr`, and `performance`, and save a client-ready HTML report.
- **Re-audit after fixes**:
  > Re-audit `https://example.com/` against `audit-example-com-2026-11-20.md` and tell me what is fixed or still open.
- **Polish prompt**:
  > Przejrzyj stronę `https://example.pl/` w trybie dogłębnym i sprawdź dodatkowo WCAG, RODO/cookies oraz wagi zdjęć.

## Sample Output

See [examples/sample-report.md](./examples/sample-report.md) for a full anonymized example audit report.

## Repository Structure

```text
website-qa-audit/
├── SKILL.md                          # Main skill instructions & workflow
├── references/
│   ├── browser-probes.md             # 7 deterministic ES2020 DOM/network JSON probes
│   ├── checklist.md                  # Core launch, QA, UX, copy & SEO checklist
│   ├── optional-modules.md           # Performance, WCAG, GDPR & Passive Security modules
│   └── regional/
│       └── pl-eu.md                  # Poland & EU legal/checksum/copy rules
├── templates/
│   ├── report-template.md            # Markdown report & re-audit template
│   └── report-template.html          # Standalone print-ready HTML client report
├── examples/
│   └── sample-report.md              # Full anonymized sample audit report
├── README.md
└── LICENSE
```

## Contributing

Issues and pull requests are welcome — especially new regional legal/language packs under `references/regional/` (e.g. `de.md`, `uk.md`, `us.md`) and additional deterministic browser probes.

## License

MIT
