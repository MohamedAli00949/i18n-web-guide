---
name: i18n-web-apps
description: >
  I18n for JS/React/Next.js — routing, SEO, ICU messages, Intl formatting, RTL/LTR, Arabic support. Trigger for: multilingual apps, hreflang, locale URLs, date/number/currency, RTL styling, Tailwind logical props, next-intl/lingui/react-i18next, translation files, Arabic/cursive bugs, dir attribute, writing modes. Claude Code: trigger when working on JS/React/Next.js codebase with any i18n task.
---

# I18n Web Apps

Stack: JS · React · Next.js · Vite · Remix · Astro

Reference files (read relevant one per task):
- Routing → `references/01-routing.md`
- SEO → `references/02-seo.md`
- Messages/ICU → `references/03-messages.md`
- Data formatting → `references/04-data-formatting.md`
- Tools/libraries → `references/05-tools.md`
- UI/direction/CSS → `references/06-ui.md`
- Framework code → `references/07-framework-patterns.md`
- Sources/links → `references/00-master-references.md` (only when user asks)

## Claude Code
Check `package.json` + existing `/messages`, `/locales`, `/i18n` before acting. Write real files, inspect before fixing bugs, match project conventions.

## Setup Order
1. URL routing — Next.js: middleware + `[locale]`; React: locale wrapper route
2. i18n library — Next.js: `next-intl`; React: `react-i18next` or `lingui`
3. Message files — `/messages/en.json` source of truth; all locales mirror structure
4. Direction — `dir` + `lang` on `<html>` from active locale
5. Format utils — `lib/format.ts` wrapping `Intl.*`
6. Apply to shared components — never hard-code user-visible strings

## Stack
| Stack | Library | Routing |
|---|---|---|
| Next.js App Router | `next-intl` or `lingui` | `app/[locale]/` + middleware |
| Next.js Pages Router | `next-intl` / `next-i18next` | `pages/[locale]/` |
| React Vite/CRA | `react-i18next` / `lingui` | React Router v6 wrapper |
| Remix | `remix-i18next` | Loader detection |
| Astro | `astro-i18n` / `i18next` | `src/pages/[locale]/` |

## Rules
1. All text in JSON/YAML — never hard-code strings
2. ICU format — `{name}`, `{n, plural, ...}`, `{g, select, ...}`
3. Validate — TypeScript types or `i18n-check` CLI
4. Logical CSS — `margin-inline-start` not `margin-left`; Tailwind: `ps-*`/`pe-*`
5. No `letter-spacing` on Arabic/Persian — breaks letterforms
6. Human review required — machine translation not production-ready
7. `Intl.*` for all formatting — never manual functions
8. `dir="auto"` on content blocks for mixed-direction UGC
9. One hreflang method — sitemap OR `<link>` OR HTTP headers
10. Next.js: locale via `middleware.ts` → layout, never read from URL in components
