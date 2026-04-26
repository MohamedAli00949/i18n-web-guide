# SEO & I18n

## Routing & SEO

### Language-Based URLs
- Use **different URLs** for each language version — do not rely on cookies or browser settings to switch language (Google recommendation).
- Use `hreflang` annotations and a sitemap to help Google link to the correct language version.
- It's fine to use localized words in URLs or IDNs (Internationalized Domain Names).
- Always use **UTF-8 encoding** in URLs. Arabic or non-Latin characters must be **converted to ASCII (punycode)** for DNS compatibility.

### Locale-Based URLs (Geotargeting)
Use a URL structure that makes geotargeting clear. Options by signal strength (see Google Search Central docs):
- ccTLD → strongest signal
- Subdomain → strong
- Subdirectory → moderate
- URL parameter → weakest (avoid for geotargeting)

## manifest.json

Include locale-specific `lang` and `dir` in the web app manifest:

```json
// English
{
  "name": "My Application",
  "lang": "en",
  "dir": "ltr"
}

// Arabic
{
  "name": "My Application",
  "description": "تطبيق بني بالحب من محمد علي",
  "lang": "ar",
  "dir": "rtl"
}
```

## Sitemap

Include `xhtml:link` alternate entries for every supported language on every URL:

```xml
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:xhtml="http://www.w3.org/1999/xhtml">
  <url>
    <loc>https://example.com/about</loc>
    <xhtml:link rel="alternate" hreflang="fr" href="https://example.com/fr/about"/>
    <xhtml:link rel="alternate" hreflang="ar" href="https://example.com/ar/about"/>
    <lastmod>2025-04-06T15:02:24.021Z</lastmod>
  </url>
</urlset>
```

## Page Metadata

Translate `<title>`, `<meta name="description">`, and all OpenGraph (`og:*`) tags per locale. This makes localized pages appear correctly in search results for the target audience.

## `<link rel="alternate">` Elements

Use when:
- Only the template/layout is translated (e.g., forums with user-generated content).
- There are small regional variations in a single language (e.g., `en-US`, `en-GB`, `en-IE`).
- The full content is translated into multiple languages.

Structure:
```html
<link rel="canonical" href="https://example.com/about"/>
<link rel="alternate" hreflang="fr" href="https://example.com/fr/about"/>
<link rel="alternate" hreflang="ar" href="https://example.com/ar/about"/>
<link rel="alternate" hreflang="x-default" href="https://example.com/about"/>
```

`x-default` = fallback for languages not listed.

> ⚠️ **Use only ONE method**: sitemap OR `<link>` elements OR HTTP headers. Don't mix them.

## HTTP Headers (Advanced / Non-HTML)

For non-HTML files (like PDFs), add an HTTP header:

```
Link: <url1>; rel="alternate"; hreflang="lang_code_1", <url2>; rel="alternate"; hreflang="lang_code_2"
```

This method is uncommon and not widely used in practice.

## Additional Resources
- [Google Search Central — International & Multilingual](https://developers.google.com/search/docs/specialty/international)
- [Webpage metadata — MDN](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Webpage_metadata)
- [Web App Manifest — W3C](https://w3c.github.io/manifest/)
- [Sitemaps — sitemaps.org](https://www.sitemaps.org/)
