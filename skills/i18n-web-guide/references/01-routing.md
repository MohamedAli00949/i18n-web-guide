# URL Routing

## Routing Patterns

**1. Sub-path slug** — `/en/about` (most popular)
Best for: blogs, landing pages, marketing sites.

**2. Domain routing**
- ccTLD: `example.fr`, `example.de` — strongest geotargeting signal
- Subdomain by language: `ar.example.com`, `fr.example.com`
- Subdomain by locale: `ca.example.com`, `eg.example.com`

**3. URL parameter** — `/?lang=en`
⚠️ Avoid for geotargeting — search engines can't crawl location links via params. Use only for language switching within an already-localized domain (e.g. indeed.com).

**4. Default routing** (no URL locale)
For: dashboards, SaaS, social apps. Detects from browser/user preferences, stores in cookie.

**5. Combined**
- Subdomain (region) + sub-path (lang): `eg.example.com/en/`
- Subdomain (region) + param (lang): `eg.example.com/?lang=en`
Use when app has region-specific third-party dependencies (payments, delivery).

## Decision Guide

| Use case | Pattern |
|---|---|
| Blog / landing page | Sub-path slug |
| Country SEO targeting | ccTLD or subdomain |
| SaaS / dashboard | Default (cookie/header) |
| Region + language | Combined |

## SSR Checklist
- Cache translated pages correctly (cookies/headers)
- Add i18n response headers
- Middleware for detection + redirect
- Generate alternate links for sitemap + `<link>` elements
- Generate page metadata per locale
