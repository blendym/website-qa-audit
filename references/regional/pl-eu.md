# Regional Rules: Poland & EU (`pl-eu`)

Apply this checklist automatically whenever auditing a Polish-language or EU-based website.

## 1. Registration Numbers & Checksums

Verify company identifiers in the footer, Contact page, and Privacy Policy / Terms using `Probe 5` from [../browser-probes.md](../browser-probes.md):

- **NIP**: Must be **10 digits** and pass the modulo-11 checksum (`weights [6,5,7,2,3,4,5,6,7]`). Flag even 10-digit NIPs that fail checksum validation as likely typos.
- **KRS**: Must be strictly **10 digits** (flag 9- or 11-digit typos such as extra leading zeros).
- **REGON**: Must be **9 or 14 digits** and pass the modulo-11 REGON checksum.

## 2. Legal Entity, Suffix & Privacy Policy (RODO)

- **Data Controller Match**: Confirm the entity named as *Administrator Danych Osobowych* in the Privacy Policy (full legal name, address, NIP, KRS, email, domain) matches the actual website operator in Contact/Footer — watch for policies copied from a sister company, related organization, or template placeholder.
- **Legal Suffix Spelling**: Watch for missing periods in Polish legal forms:
  - `sp. z o.o.` (both periods required, avoid `sp. z o.o` or `Sp.z.o.o.`)
  - `S.A.`, `sp. k.`, `sp. j.`
- **UODO & Supervisory Links**: Flag plain `http://` links to `uodo.gov.pl` (should be `https://uodo.gov.pl/`).
- **RODO Mandatory Elements**: Legal basis for processing, retention period, user rights (access, rectification, erasure, portability, objection, complaint to PUODO), and cookie/consent settings when analytics/marketing cookies exist.

## 3. Contact & Phone Formatting

- Prefer consistent international `tel:+48...` format in `href` attributes (flag mixed `tel:48...` vs `tel:+48...`).

## 4. Polish Copy & Localization Pitfalls

- **Missing Diacritics & Typos**: Watch for missing Polish diacritics (`ą, ć, ę, ł, ń, ó, ś, ź, ż`) and swapped letters in headings, buttons, and body copy.
- **Common Conjugation & Agreement Errors**:
  - 1st-person (`-ę`) vs 3rd-person (`-e`) singular verb endings in descriptive copy.
  - Grammatical gender agreement across clauses (especially neuter nouns such as *doświadczenie*, *przedsiębiorstwo*, *rozwiązanie*).
- **Voivodeship `<select>` Options & Casing**:
  - Verify spelling of all 16 Polish voivodeships and check for consistent capitalization across dropdown options (official Polish uses lowercase when standalone, or uniform Title Case inside UI lists).
- **Punctuation & Quotes**: Flag unclosed Polish quotation marks (`„...”`) and inconsistent dash/hyphen spacing.
- **Untranslated Plugin Strings**: Flag English CMS/plugin leftovers on Polish pages (e.g. `This field is required`, `Search...`, `Read more`, `No results found`, `Submit`).
