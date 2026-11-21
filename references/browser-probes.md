# Deterministic Browser JS Probes

Run these self-contained ES2020 JavaScript IIFE snippets directly inside the live browser context (e.g. Playwright `page.evaluate`, Chrome DevTools console/evaluate, or browser subagent scripts) to collect exact, reproducible JSON facts.

---

## Probe 1 — SEO, Indexing, Social Share & Icons

Checks `<h1>` hierarchy, `<meta name="robots">` (`noindex`/`nofollow` launch blockers), `<html lang>`, Open Graph tags, canonical/hreflang links, and favicons. Also manually fetch `/robots.txt` (verify no accidental `Disallow: /`) and check that `og:image` returns HTTP 200.

```js
(() => {
  const getMeta = (attr, val) =>
    document.querySelector(`meta[${attr}="${val}"]`)?.getAttribute('content')?.trim() || null;

  const h1s = [...document.querySelectorAll('h1')].map((el) => el.innerText.trim());
  const countsByLevel = {};
  for (let i = 1; i <= 6; i++) {
    countsByLevel[`h${i}`] = document.querySelectorAll(`h${i}`).length;
  }

  const icons = [...document.querySelectorAll('link[rel*="icon"]')].map((el) => ({
    rel: el.getAttribute('rel'),
    href: el.href || el.getAttribute('href'),
    sizes: el.getAttribute('sizes') || null,
  }));

  const hreflangs = [...document.querySelectorAll('link[rel="alternate"][hreflang]')].map((el) => ({
    hreflang: el.getAttribute('hreflang'),
    href: el.href || el.getAttribute('href'),
  }));

  const metaDescription = getMeta('name', 'description');
  const title = document.title?.trim() || null;

  return {
    url: location.href,
    lang: document.documentElement.getAttribute('lang') || null,
    title,
    titleLength: title ? title.length : 0,
    metaDescription,
    metaDescriptionLength: metaDescription ? metaDescription.length : 0,
    metaRobots: getMeta('name', 'robots'),
    googlebot: getMeta('name', 'googlebot'),
    canonical: document.querySelector('link[rel="canonical"]')?.href || null,
    hreflangs,
    og: {
      title: getMeta('property', 'og:title'),
      description: getMeta('property', 'og:description'),
      image: getMeta('property', 'og:image'),
      url: getMeta('property', 'og:url'),
    },
    icons,
    headings: {
      h1Count: h1s.length,
      h1Texts: h1s,
      countsByLevel,
    },
  };
})();
```

---

## Probe 2 — Contact Links (`mailto:` / `tel:`) & Suspicious Buttons/Links

Audits `mailto:` and `tel:` consistency, flags dead links (`href="#"`, `href=""`, `javascript:`), and lists non-submit `<button>` elements so every candidate interactive button can be click-tested.

```js
(() => {
  const mailtoLinks = [...document.querySelectorAll('a[href^="mailto:"]')].map((a) => {
    const hrefEmail = a.getAttribute('href').replace(/^mailto:/i, '').split('?')[0].trim();
    const visibleText = a.innerText.trim();
    const looksLikeEmail = visibleText.includes('@');
    return {
      visibleText,
      hrefEmail,
      textMatchesHref: !looksLikeEmail || visibleText.toLowerCase() === hrefEmail.toLowerCase(),
    };
  });

  const telLinks = [...document.querySelectorAll('a[href^="tel:"]')].map((a) => {
    const rawHref = a.getAttribute('href').trim();
    const normalized = rawHref.replace(/^tel:/i, '').replace(/[\s()-]/g, '');
    return {
      visibleText: a.innerText.trim(),
      href: rawHref,
      hasPlusPrefix: normalized.startsWith('+'),
    };
  });

  const deadLinks = [...document.querySelectorAll('a')].filter((a) => {
    const href = (a.getAttribute('href') || '').trim();
    return href === '' || href === '#' || href.toLowerCase().startsWith('javascript:');
  }).map((a) => ({
    text: a.innerText.trim(),
    href: a.getAttribute('href'),
    ariaLabel: a.getAttribute('aria-label'),
  }));

  const interactiveButtons = [...document.querySelectorAll('button')].filter((btn) => {
    const type = (btn.getAttribute('type') || '').toLowerCase();
    return type !== 'submit' && type !== 'reset';
  }).map((btn) => ({
    text: btn.innerText.trim(),
    ariaLabel: btn.getAttribute('aria-label'),
    ariaControls: btn.getAttribute('aria-controls'),
    disabled: btn.disabled,
    classes: btn.className,
  }));

  return {
    mailtoCount: mailtoLinks.length,
    mailtoMismatches: mailtoLinks.filter((m) => !m.textMatchesHref),
    mailtoLinks,
    telCount: telLinks.length,
    telWithoutPlus: telLinks.filter((t) => !t.hasPlusPrefix),
    deadLinks,
    interactiveButtons,
  };
})();
```

---

## Probe 3 — Images, Broken/Empty `src`, Sizing & Network Transfer Weights

Scans DOM images for empty `src`, broken loads, missing `alt`, and oversized dimensions (`naturalWidth > 2 * renderedWidth`), and joins with `PerformanceResourceTiming` to surface the heaviest image transfers.

```js
(() => {
  const perfByUrl = new Map();
  try {
    for (const entry of performance.getEntriesByType('resource')) {
      if (entry.initiatorType === 'img' || /\.(png|jpe?g|webp|avif|gif|svg)(\?|$)/i.test(entry.name)) {
        perfByUrl.set(entry.name, {
          transferSizeKB: entry.transferSize ? Math.round((entry.transferSize / 1024) * 10) / 10 : null,
          decodedBodySizeKB: entry.decodedBodySize ? Math.round((entry.decodedBodySize / 1024) * 10) / 10 : null,
        });
      }
    }
  } catch (_) {}

  const emptySrc = [];
  const broken = [];
  const missingAlt = [];
  const oversized = [];
  const weights = [];

  for (const img of document.querySelectorAll('img')) {
    const rawSrc = img.getAttribute('src');
    const resolvedSrc = img.currentSrc || img.src || '';
    const alt = img.getAttribute('alt');
    const rect = img.getBoundingClientRect();

    if (rawSrc === null || rawSrc.trim() === '') {
      emptySrc.push({ alt, classes: img.className });
      continue;
    }

    if (img.complete && img.naturalWidth === 0) {
      broken.push({ src: resolvedSrc, alt });
    }

    if (alt === null) {
      missingAlt.push({ src: resolvedSrc });
    }

    if (rect.width >= 100 && img.naturalWidth > rect.width * 2) {
      oversized.push({
        src: resolvedSrc,
        naturalWidth: img.naturalWidth,
        naturalHeight: img.naturalHeight,
        renderedWidth: Math.round(rect.width),
        renderedHeight: Math.round(rect.height),
      });
    }

    const perf = perfByUrl.get(resolvedSrc);
    if (perf && perf.transferSizeKB !== null) {
      weights.push({
        src: resolvedSrc,
        transferSizeKB: perf.transferSizeKB,
        decodedBodySizeKB: perf.decodedBodySizeKB,
        naturalWidth: img.naturalWidth,
        renderedWidth: Math.round(rect.width),
      });
    }
  }

  weights.sort((a, b) => (b.transferSizeKB || 0) - (a.transferSizeKB || 0));

  return {
    totalImages: document.querySelectorAll('img').length,
    emptySrcCount: emptySrc.length,
    emptySrc,
    brokenCount: broken.length,
    broken,
    missingAltCount: missingAlt.length,
    missingAlt: missingAlt.slice(0, 20),
    oversizedCount: oversized.length,
    oversized: oversized.slice(0, 15),
    heaviestImages: weights.slice(0, 15),
  };
})();
```

---

## Probe 4 — Viewport Overflow & Clipped Controls (Mobile / Tablet)

Run at `390x844` (mobile) and `1280x720` or `768x1024` (laptop/tablet). Reports exact horizontal overflow elements and clipped buttons/pills/tabs/placeholders (`scrollWidth > clientWidth`).

```js
(() => {
  const describe = (el) => {
    const id = el.id ? `#${el.id}` : '';
    const cls = typeof el.className === 'string' && el.className.trim()
      ? '.' + el.className.trim().split(/\s+/).slice(0, 3).join('.')
      : '';
    return `${el.tagName.toLowerCase()}${id}${cls}`;
  };

  const vw = window.innerWidth;
  const docScrollW = document.documentElement.scrollWidth;
  const overflowElements = [];

  for (const el of document.querySelectorAll('body *')) {
    const rect = el.getBoundingClientRect();
    if (rect.width === 0 || rect.height === 0) continue;
    if (rect.right > vw + 1 || rect.left < -1) {
      overflowElements.push({
        selector: describe(el),
        left: Math.round(rect.left),
        right: Math.round(rect.right),
        viewportWidth: vw,
        text: (el.innerText || '').trim().slice(0, 50),
      });
      if (overflowElements.length >= 20) break;
    }
  }

  const clippedControls = [];
  for (const el of document.querySelectorAll('a, button, [role="tab"], input, select')) {
    const rect = el.getBoundingClientRect();
    if (rect.width === 0 || rect.height === 0) continue;
    const style = getComputedStyle(el);
    const clipsX = style.overflowX === 'hidden' || style.overflowX === 'clip' || style.textOverflow === 'ellipsis';
    if (clipsX && el.clientWidth > 0 && el.scrollWidth > el.clientWidth + 2) {
      clippedControls.push({
        selector: describe(el),
        text: (el.innerText || el.getAttribute('placeholder') || el.value || '').trim().slice(0, 60),
        clientWidth: el.clientWidth,
        scrollWidth: el.scrollWidth,
      });
    }
  }

  return {
    viewportWidth: vw,
    documentScrollWidth: docScrollW,
    hasHorizontalOverflow: docScrollW > vw,
    overflowPx: Math.max(0, docScrollW - vw),
    overflowElements,
    clippedControls: clippedControls.slice(0, 20),
  };
})();
```

---

## Probe 5 — Polish Registration Checksum Validator (NIP, REGON, KRS)

Extracts NIP, REGON, and KRS candidates from the current page text and validates exact digit lengths and modulo-11 checksums (NIP and 9-/14-digit REGON).

```js
(() => {
  const isValidNIP = (raw) => {
    const d = raw.replace(/^PL/i, '').replace(/[\s-]/g, '');
    if (!/^\d{10}$/.test(d)) return { valid: false, normalized: d, reason: `expected 10 digits, got ${d.length}` };
    const weights = [6, 5, 7, 2, 3, 4, 5, 6, 7];
    const sum = weights.reduce((acc, w, i) => acc + w * Number(d[i]), 0);
    const mod = sum % 11;
    const valid = mod !== 10 && mod === Number(d[9]);
    return { valid, normalized: d, reason: valid ? 'ok' : `invalid NIP checksum (mod=${mod}, last=${d[9]})` };
  };

  const isValidREGON9 = (d) => {
    const weights = [8, 9, 2, 3, 4, 5, 6, 7];
    const sum = weights.reduce((acc, w, i) => acc + w * Number(d[i]), 0);
    const check = (sum % 11) % 10;
    return check === Number(d[8]);
  };

  const isValidREGON = (raw) => {
    const d = raw.replace(/[\s-]/g, '');
    if (d.length === 9 && /^\d{9}$/.test(d)) {
      const valid = isValidREGON9(d);
      return { valid, normalized: d, reason: valid ? 'ok' : 'invalid 9-digit REGON checksum' };
    }
    if (d.length === 14 && /^\d{14}$/.test(d)) {
      if (!isValidREGON9(d.slice(0, 9))) {
        return { valid: false, normalized: d, reason: 'invalid base 9-digit REGON checksum' };
      }
      const weights = [2, 4, 8, 5, 0, 9, 7, 3, 6, 1, 2, 4, 8];
      const sum = weights.reduce((acc, w, i) => acc + w * Number(d[i]), 0);
      const check = (sum % 11) % 10;
      const valid = check === Number(d[13]);
      return { valid, normalized: d, reason: valid ? 'ok' : 'invalid 14-digit REGON checksum' };
    }
    return { valid: false, normalized: d, reason: `expected 9 or 14 digits, got ${d.length}` };
  };

  const isValidKRS = (raw) => {
    const d = raw.replace(/[\s-]/g, '');
    const valid = /^\d{10}$/.test(d);
    return { valid, normalized: d, reason: valid ? 'ok' : `expected 10 digits, got ${d.length}` };
  };

  const text = document.body?.innerText || '';
  const collect = (regex, validator) => {
    const seen = new Set();
    const out = [];
    for (const match of text.matchAll(regex)) {
      const raw = match[1].trim();
      if (seen.has(raw)) continue;
      seen.add(raw);
      out.push({ raw, ...validator(raw) });
    }
    return out;
  };

  return {
    nip: collect(/\bNIP[^\S\r\n]*[:.]?[^\S\r\n]*(?:nr\.?[^\S\r\n]*)?((?:PL[^\S\r\n]*)?(?:\d[ -]?){7,14}\d)/gi, isValidNIP),
    regon: collect(/\bREGON[^\S\r\n]*[:.]?[^\S\r\n]*(?:nr\.?[^\S\r\n]*)?((?:\d[ -]?){7,15}\d)/gi, isValidREGON),
    krs: collect(/\bKRS[^\S\r\n]*[:.]?[^\S\r\n]*(?:nr\.?[^\S\r\n]*)?((?:\d[ -]?){7,14}\d)/gi, isValidKRS),
  };
})();
```

---

## Probe 6 — GDPR Pre-Consent Cookies & Trackers

Run **immediately after first page load**, before clicking any cookie banner button. Detects first-party/third-party JS cookies and tracking requests already fired prior to user consent.

```js
(() => {
  const trackerPatterns = [
    'google-analytics.com',
    'googletagmanager.com',
    'doubleclick.net',
    'connect.facebook.net',
    'facebook.com/tr',
    'static.hotjar.com',
    'clarity.ms',
    'analytics.tiktok.com',
    'snap.licdn.com',
    'mc.yandex.ru',
    'matomo',
  ];

  const cookieNames = document.cookie
    ? document.cookie.split(';').map((c) => c.split('=')[0].trim()).filter(Boolean)
    : [];

  let trackingRequests = [];
  try {
    trackingRequests = performance
      .getEntriesByType('resource')
      .map((e) => e.name)
      .filter((url) => trackerPatterns.some((pattern) => url.toLowerCase().includes(pattern)));
  } catch (_) {}

  return {
    cookieCount: cookieNames.length,
    cookieNames,
    trackingRequestCount: trackingRequests.length,
    trackingRequests: [...new Set(trackingRequests)],
  };
})();
```

---

## Probe 7 — Performance, WCAG & Link Safety Snapshot

Captures navigation timing, buffered LCP/CLS (where exposed by the browser), above-the-fold lazy-loaded images, icon-only buttons without accessible names, unlabeled visible form controls, and external `target="_blank"` links missing `noopener`.

```js
(() => {
  let navTiming = null;
  try {
    const nav = performance.getEntriesByType('navigation')[0];
    if (nav) {
      navTiming = {
        ttfbMs: Math.round(nav.responseStart - nav.requestStart),
        domContentLoadedMs: Math.round(nav.domContentLoadedEventEnd),
        loadEventMs: Math.round(nav.loadEventEnd),
      };
    }
  } catch (_) {}

  let lcpMs = null;
  let lcpElement = null;
  try {
    const lcps = performance.getEntriesByType('largest-contentful-paint');
    const last = lcps[lcps.length - 1];
    if (last) {
      lcpMs = Math.round(last.startTime);
      lcpElement = last.element ? last.element.tagName.toLowerCase() : null;
    }
  } catch (_) {}

  let clsScore = null;
  try {
    const shifts = performance.getEntriesByType('layout-shift');
    if (shifts.length > 0) {
      clsScore = Math.round(
        shifts.filter((s) => !s.hadRecentInput).reduce((sum, s) => sum + s.value, 0) * 1000
      ) / 1000;
    }
  } catch (_) {}

  const lazyAboveFoldImages = [...document.querySelectorAll('img[loading="lazy"]')]
    .filter((img) => {
      const r = img.getBoundingClientRect();
      return r.width > 0 && r.height > 0 && r.top < window.innerHeight;
    })
    .map((img) => ({ src: img.currentSrc || img.getAttribute('src'), top: Math.round(img.getBoundingClientRect().top) }));

  const unlabeledButtons = [...document.querySelectorAll('button')]
    .filter((btn) => {
      const text = btn.innerText.trim();
      const aria = (btn.getAttribute('aria-label') || '').trim();
      const labelledBy = btn.getAttribute('aria-labelledby');
      const title = (btn.getAttribute('title') || '').trim();
      return !text && !aria && !labelledBy && !title;
    })
    .map((btn) => ({ classes: btn.className, html: btn.outerHTML.slice(0, 120) }));

  const unlabeledInputs = [...document.querySelectorAll('input, select, textarea')]
    .filter((el) => {
      const type = (el.getAttribute('type') || '').toLowerCase();
      if (['hidden', 'submit', 'reset', 'button'].includes(type)) return false;
      const r = el.getBoundingClientRect();
      if (r.width === 0 || r.height === 0) return false;
      const id = el.id;
      const hasForLabel = id ? Boolean(document.querySelector(`label[for="${CSS.escape(id)}"]`)) : false;
      const wrappedInLabel = Boolean(el.closest('label'));
      const aria = Boolean((el.getAttribute('aria-label') || '').trim() || el.getAttribute('aria-labelledby'));
      return !hasForLabel && !wrappedInLabel && !aria;
    })
    .map((el) => ({ tag: el.tagName.toLowerCase(), type: el.getAttribute('type'), name: el.getAttribute('name'), placeholder: el.getAttribute('placeholder') }));

  const unsafeBlankLinks = [...document.querySelectorAll('a[target="_blank"]')]
    .filter((a) => {
      try {
        const u = new URL(a.href, location.href);
        if (u.origin === location.origin) return false;
      } catch (_) {
        return false;
      }
      const rel = (a.getAttribute('rel') || '').toLowerCase().split(/\s+/);
      return !rel.includes('noopener') && !rel.includes('noreferrer');
    })
    .map((a) => ({ text: a.innerText.trim().slice(0, 40), href: a.href }));

  return {
    navTiming,
    lcpMs,
    lcpElement,
    clsScore,
    lazyAboveFoldImages,
    unlabeledButtonsCount: unlabeledButtons.length,
    unlabeledButtons: unlabeledButtons.slice(0, 15),
    unlabeledInputsCount: unlabeledInputs.length,
    unlabeledInputs: unlabeledInputs.slice(0, 15),
    unsafeBlankLinksCount: unsafeBlankLinks.length,
    unsafeBlankLinks: unsafeBlankLinks.slice(0, 15),
  };
})();
```
