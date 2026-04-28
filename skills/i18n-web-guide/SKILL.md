---
name: i18n-web-guide
description: 
  Comprehensive guidance for building internationalized (i18n) web applications using JavaScript, React, Next.js, and related frameworks.
  Use this skill whenever the user is working in JS/React/Next.js and asks about: internationalization, localization, i18n, l10n, multilingual apps, RTL/LTR support, translation management, hreflang, locale routing, ICU message format, Intl API, bidirectional text, or supporting multiple languages/regions.
  Also trigger for: URL routing for multiple languages in Next.js or React Router, SEO for multilingual Next.js sites, date/number formatting per locale in JS, translation file structure, RTL styling in React/Tailwind, Arabic support, or choosing between lingui vs next-intl vs react-intl vs i18next.
  Trigger when the user is working on a JS/React/Next.js codebase and asks anything related to: adding a new language, setting up translation files, formatting dates or currencies, supporting Arabic or RTL, fixing locale-related bugs, or scaffolding i18n from scratch.
  Apply even when the user doesn't say "i18n" explicitly — e.g. "how do I support Arabic in my Next.js app", "how do I format currency per country in React", "add RTL support to my Tailwind app", or "my Arabic text looks broken".
---

# I18n Web Apps — Complete Guideline

**Target stack:** JavaScript · React · Next.js · and any JS-based framework (Vite, Remix, Astro, etc.)

Based on Mohamed Ali's journey building i18n apps in Arabic, English, French, and German.

> **Reference files — read the relevant one(s) per task:**
>
> - Part 1: URL Routing → `references/01-routing.md`
> - Part 2: SEO & I18n → `references/02-seo.md`
> - Part 3: Text Content / Messages → `references/03-messages.md`
> - Part 4: Data Formatting → `references/04-data-formatting.md`
> - Part 5: Tools (Libraries, Plugins, Extensions) → `references/05-tools.md`
> - Part 6: UI & I18n → `references/06-ui.md`
> - Framework code patterns → `references/07-framework-patterns.md`
> - Master references → `references/00-master-references.md`

---

## Claude Code Behavior

When operating inside a codebase (Claude Code), go beyond answering — act:

1. **Detect the stack** — check `package.json` for the framework and existing i18n library before recommending anything.
2. **Check what exists** — look for existing locale files (`/messages`, `/locales`, `/i18n`), middleware, and routing config before scaffolding new ones.
3. **Write real code** — generate actual implementation files (message JSON, middleware, format utility functions, layout wrappers) based on the detected stack.
4. **Fix existing bugs** — when a user reports broken locale behavior, inspect the relevant files first, then apply the fix.
5. **Follow the project's conventions** — match existing file structure, naming, and import style.

Read `references/07-framework-patterns.md` for ready-to-use code patterns per framework.

---

## The I18n Process (Order Matters)

Always follow this sequence when starting an i18n project in JS/React/Next.js:

1. **Define URL routing pattern** — decide before writing any code. In Next.js App Router this means middleware + `[locale]` segment. In React Router this means a locale-aware wrapper route.
2. **Install and configure the i18n library** — for Next.js use `next-intl` or `lingui`; for React use `react-i18next` or `lingui`; see `references/05-tools.md` for the full comparison.
3. **Set up message files** — create `/messages/en.json` (or `/locales/en/common.json`) as the source of truth. All other languages mirror its structure.
4. **Configure layout direction** — set `dir` and `lang` on `<html>` dynamically from the active locale. In Next.js App Router, do this in `app/[locale]/layout.tsx`.
5. **Build data formatting utilities** — create a shared `lib/format.ts` (or similar) wrapping `Intl.*` objects configured for the active locale.
6. **Apply to shared components** — all design system components (buttons, inputs, cards) use the translation and formatting utilities.
7. **Use throughout the app** — never hard-code user-visible strings anywhere in the codebase.

---

## Quick Reference by Topic

### Routing

Read `references/01-routing.md` for:

- Sub-path slug (`/en/about`), domain routing (ccTLD, subdomain), URL parameters, default (cookie/header) routing, combined patterns
- When to use each pattern
- Server-side rendering configurations
- Navigation helpers (router, locale switching, redirects)

### SEO

Read `references/02-seo.md` for:

- `hreflang` annotations and sitemap alternate links
- `manifest.json` locale config (`lang`, `dir`)
- Page metadata (`<title>`, `<meta description>`, OpenGraph) per locale
- `<link rel="alternate">` and `<link rel="canonical">`
- HTTP header method (for non-HTML files like PDFs)

### Text Content / Messages

Read `references/03-messages.md` for:

- ICU message format: simple, arguments, `{select}`, `{plural}`, `{selectordinal}`, rich text, escaping
- File structure patterns (by page, by feature, combined)
- Content extraction (Lingui macros, next-intl experimental)
- Validation (TypeScript types, `i18n-check` CLI)
- Storage (JSON/YAML in codebase vs. Translation Management Systems: Transifex, Lokalise, Crowdin)

### Data Formatting

Read `references/04-data-formatting.md` for:

- `Intl.DateTimeFormat`, `Intl.DurationFormat`, `Intl.RelativeTimeFormat`
- `Intl.NumberFormat` (currency, percent, units) — known symbol limitations (EGP, SAR)
- `Intl.ListFormat` (conjunction, disjunction, unit)
- `Intl.Locale`, `Intl.DisplayNames`, `Intl.Collator`, `Intl.Segmenter`
- Polyfills via `format.js`

### Tools

Read `references/05-tools.md` for:

- Library comparison: lingui, formatJS/react-intl, i18next/react-i18next, next-intl, vue-i18n, @angular/localize, Transloco
- ESLint plugins: `eslint-plugin-i18n-checker`, `eslint-plugin-rtl-friendly`
- VSCode extensions: `i18n ally`
- Figma plugins: `i18n meta`

### UI & I18n

Read `references/06-ui.md` for:

- Layout direction: `dir` attribute, `lang` on `<html>`, Radix UI `DirectionProvider`, shadcn RTL config
- Writing modes: inline (LTR/RTL), block (top-to-bottom / vertical), `writing-mode`, `text-orientation`
- `unicode-bidi` CSS property values
- Typography: `font-family`, `font-weight`, `line-height`, `letter-spacing` (avoid on cursive scripts like Arabic)
- Logical properties (`margin-inline-start` etc.) vs. physical properties
- Form fields: range sliders, date pickers (locale + timezone + calendar system), text editors
- Translating HTML attributes: `title`, `alt`, `placeholder`, `aria-*`, `data-*`

---

## Quick Stack Reference

| Stack | Recommended library | Routing approach |
|---|---|---|
| Next.js App Router | `next-intl` or `lingui` | `app/[locale]/` + middleware |
| Next.js Pages Router | `next-intl` or `next-i18next` | `pages/[locale]/` |
| React (Vite/CRA) | `react-i18next` or `lingui` | React Router v6 locale wrapper |
| Remix | `remix-i18next` | Loader-based locale detection |
| Astro | `astro-i18n` or `i18next` | `src/pages/[locale]/` |

For full library comparison and setup details → `references/05-tools.md`
For framework-specific code patterns → `references/07-framework-patterns.md`

---

## Key Principles (Apply Always)

1. **Never hard-code user-visible strings** — all text goes into translation JSON/YAML files.
2. **Use ICU message format** — standard across all JS i18n libraries (`{name}`, `{count, plural, ...}`, `{gender, select, ...}`).
3. **Always validate translations** — missing keys in target languages cause silent runtime bugs. Use TypeScript types or `i18n-check` CLI.
4. **Prefer logical CSS properties** over physical ones (`margin-inline-start` not `margin-left`) for automatic RTL/LTR flipping — works with Tailwind `ps-*`/`pe-*` utilities too.
5. **Never use `letter-spacing` on Arabic or Persian text** — it breaks the connected letterforms visually.
6. **Review translations with language specialists** — LLM/machine translation alone is not sufficient for production. Always do human review.
7. **Use `Intl.*` objects** for all date, number, list, and currency formatting — never build manual format functions.
8. **Use `dir="auto"` on individual content blocks** — essential in text editors and any user-generated content with mixed languages.
9. **Only one hreflang strategy** — sitemap OR `<link>` elements OR HTTP headers. Never mix them.
10. **In Next.js App Router** — always handle locale in `middleware.ts` and pass it through `app/[locale]/layout.tsx`. Never read locale from the URL manually inside components.
