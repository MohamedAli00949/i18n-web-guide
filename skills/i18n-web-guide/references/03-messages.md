# Text Content / Messages

Move all visible text into JSON/YAML files and use ICU message format — the standard across all major i18n frameworks (JS, Java, C, PHP).

## 1. ICU Message Types

### Simple

```json
{ "message": "Message Content" }
```

### Simple Arguments (dynamic variables)

```json
{ "greeting": "Hello, {name}" }
```

### Formatted Arguments `{key, type, format}`

```json
{ "msg-number": "I have {numCats, number} cats." }
{ "black-number": "Almost {pctBlack, number, ::percent} of them are black." }
```

### `{select}` — switch-like branching

```json
{
	"gender-message": "{gender, select, male {He will respond shortly.} female {She will respond shortly.} other {They will respond shortly.}}"
}
```

Values: `{ "gender": "male" }` → **He will respond shortly.**

### `{plural}` — locale-aware pluralization

```json
{
	"items": "{itemCount, plural, =0 {You have no items.} one {You have # item.} other {You have # items.}}"
}
```

> Use `Intl.PluralRules` or `format.js` polyfill to understand plural rules per language.

### `{selectordinal}` — ordinal numbers (1st, 2nd, 3rd)

```json
{
	"cat-birthday": "It's my cat's {year, selectordinal, one {#st} two {#nd} few {#rd} other {#th}} birthday!"
}
```

### Rich Text Formatting (HTML/JSX combined with text)

```json
{
	"rich-msg": "Our price is <boldThis>{price}</boldThis> with <link>{pct} discount</link>"
}
```

**Library support:**

- **lingui**: uses component-based macros
- **react-i18next**: `Trans` component
- **react-intl**: `t.rich()` function + component
- **next-intl**: `t.rich()` for rich text, `t.markup()` for HTML
- **vue-i18n**: HTML element combination support

### Quoting / Escaping

Use single quotes `'` to escape curly braces or apostrophes:

```json
{ "message": "Escape curly braces with single quotes (e.g., '{name})" }
{ "message": "This '{isn''t}' obvious" }
{ "message": "'<notATag>hello</notATag>'" }
```

## 2. File Structure Patterns

Always include a `common` namespace for shared messages.

### By Routing Page

```json
{
	"common": { "save": "Save", "cancel": "Cancel" },
	"AboutPage": { "title": "About page" }
}
```

### By Feature

```json
{
	"common": { "save": "Save", "cancel": "Cancel" },
	"Products": {
		"actions": {
			"add-product": "Add Product",
			"delete-product": "Delete Product"
		}
	}
}
```

### Combined (Recommended for large apps)

```json
{
	"common": { "save": "Save", "cancel": "Cancel" },
	"Products": {
		"new-product": { "title": "Add Product", "description": "..." },
		"all-products": { "header": { "title": "Welcome to the products page" } }
	}
}
```

## 3. Content Extraction (for existing hard-coded content)

For apps with hard-coded strings (single-language apps, AI-generated content):

### Lingui

Uses **macros** to extract messages from JSX/JS at compile time. Full workflow:

1. Wrap strings with Lingui macros.
2. Run `lingui extract` to generate `.po`/`.json` files.
3. Translate the extracted files.
4. Compile back into the app.

### next-intl

Experimental `useExtracted` hook for content extraction.

## 4. Validation

### TypeScript Types (simple, for small projects)

```ts
export type Equal<X, Y> = X extends Y ? (Y extends X ? true : false) : false;

type MessagesEn = typeof import("../../messages/en.json");
type MessagesFr = typeof import("../../messages/fr.json");
type MessagesAr = typeof import("../../messages/ar.json");

type MatchEnAr = Equal<MessagesEn, MessagesAr>;
type MatchEnFr = Equal<MessagesEn, MessagesFr>;

const _check: Equal<MatchEnAr, MatchEnFr> = true;
```

### i18n-check CLI (recommended for scaling)

Validates ICU and i18next translation files. Finds **missing** and **broken** translations by comparing source language to all target files.

- Run as a pre-commit hook or in CI.
- GitHub: [lingualdev/i18n-check](https://github.com/lingualdev/i18n-check)

## 5. Storing Translations
### Traditional
Store JSON/YAML files in the codebase. Translate manually using LLMs or translation sites.
> ⚠️ **Always have language specialists review machine translations.** In production projects, relying solely on automated translation never reaches 100% accuracy.
### Translation Management Systems (TMS)
For larger teams and scalable workflows:
- [Transifex](https://www.transifex.com/)
- [Lokalise](https://lokalise.com/)
- [Crowdin](https://crowdin.com/)

## Additional Resources

- [ICU Message Format](https://unicode-org.github.io/icu/userguide/format_parse/messages/)
- [Lingui](https://lingui.dev)
- [FormatJS](https://formatjs.github.io/)
- [i18next](https://www.i18next.com/)
- [vue-i18n](https://vue-i18n.intlify.dev/)
- [i18n-check](https://github.com/lingualdev/i18n-check)
- [Key Cooker VSCode Extension](https://marketplace.visualstudio.com/items?itemName=MohamedAli.key-cooker) — helps manage JSON/YAML translation keys
