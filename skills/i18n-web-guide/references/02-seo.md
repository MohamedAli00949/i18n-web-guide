# SEO & I18n

## URL Rules
- Use different URLs per language — don't rely on cookies/browser to switch (Google recommendation)
- Fine to use localized words in URLs (IDNs). Arabic/non-Latin chars → convert to ASCII (punycode) for DNS
- Geotargeting signal strength: ccTLD > subdomain > subdirectory > URL param (avoid param for geotargeting)

## manifest.json
```json
{ "name": "App", "lang": "ar", "dir": "rtl" }
{ "name": "App", "lang": "en", "dir": "ltr" }
```

## Sitemap
```xml
<url>
  <loc>https://example.com/about</loc>
  <xhtml:link rel="alternate" hreflang="fr" href="https://example.com/fr/about"/>
  <xhtml:link rel="alternate" hreflang="ar" href="https://example.com/ar/about"/>
</url>
```

## Page Metadata
Translate `<title>`, `<meta name="description">`, and all `og:*` tags per locale.

## `<link rel="alternate">`
```html
<link rel="canonical" href="https://example.com/about"/>
<link rel="alternate" hreflang="fr" href="https://example.com/fr/about"/>
<link rel="alternate" hreflang="x-default" href="https://example.com/about"/>
```
`x-default` = fallback for unlisted languages.

⚠️ Use ONE method only: sitemap OR `<link>` elements OR HTTP headers. Never mix.

## HTTP Headers (non-HTML files like PDFs)
```
Link: <url1>; rel="alternate"; hreflang="en", <url2>; rel="alternate"; hreflang="ar"
```
