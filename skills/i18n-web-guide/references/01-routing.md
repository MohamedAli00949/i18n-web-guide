# URL Routing

## Routing Patterns

### 1. Sub-path Slug (Most Popular)
Also called **subdirectories** or **prefix-based routing**.
Best for: static content — landing pages, blogs, marketing sites.

```
https://example.com/[lang|locale]
https://example.com/[lang|locale]/about
```

### 2. Domain Routing

#### a. ccTLD (Country-code top-level domain)
Improves geotargeting for single-country audiences.
Structure: `website-name.[country-code]` (e.g., `example.fr`, `example.de`)

#### b. Subdomain with gTLD

**Based on language:**
`ar.example.com`, `fr.example.com`

**Based on locale (geotargeting):**
`ca.example.com`, `eg.example.com`

Best for geotargeting. Most powerful for SEO.

### 3. URL Parameter Routing
Structure: `website-name.com/?lang=en` or `?locale=ca`

⚠️ Not recommended for geotargeting — search engines struggle to crawl location-specific links via URL params.

**Use case:** Sites already localized by domain that want to support multiple languages within a region (e.g., `indeed.com` language switcher).

### 4. Default Routing (No URL locale)
Used for: dashboards, SaaS platforms, social media apps with dynamic content.

- Detects language from browser settings or user preferences.
- Stores selected lang/locale in cookies or similar.

### 5. Combined Patterns
For apps with region-specific third-party dependencies (payment gateways, delivery providers):

**a. Subdomain (location) + Sub-path slug (language):**
```
[locale].yourdomain.com/[lang]/
eg.example.com/en/
ca.example.com/fr/about/
```

**b. Subdomain (location) + URL parameter (language):**
```
[locale].yourdomain.com/?lang=[lang]
eg.example.com/?lang=en
ca.example.com/?lang=fr
```


## Best Practices

### Server-Side Rendering Configurations
- Handle **caching** for server-side translated pages (cookies, headers).
- Add **i18n headers** to page responses.
- Add **proxy/middleware** for request handling and routing redirection.
- Generate **alternate links** for sitemap and `<link>` elements.
- Detect locale from cookies or HTTP headers.
- Generate **page metadata** per locale.

### Navigation Functions to Provide
- **Router functions** for getting native localized page links.
- **State management layer** for messages, format configurations (DRY principle) per locale.
- Helpers for pathname, redirection, navigation link, and locale switching.

## Choosing a Pattern — Decision Guide

| Use case | Recommended pattern |
|---|---|
| Blog / landing page | Sub-path slug |
| Targeting specific countries (SEO) | ccTLD or Subdomain |
| SaaS / dashboard | Default (cookie/header) |
| Mixed region + language | Combined (subdomain + sub-path) |
| Single domain, multiple languages | Sub-path slug |
