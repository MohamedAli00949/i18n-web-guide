# Messages / ICU Format

## ICU Types

**Simple:** `{ "msg": "Hello" }`
**Variable:** `{ "greet": "Hello, {name}" }`
**Formatted:** `{ "n": "I have {count, number} cats." }`
**Select (switch):**
```json
{ "g": "{gender, select, male {He responds.} female {She responds.} other {They respond.}}" }
```
**Plural:**
```json
{ "items": "{n, plural, =0 {No items.} one {# item.} other {# items.}}" }
```
**Selectordinal:**
```json
{ "bday": "{year, selectordinal, one {#st} two {#nd} few {#rd} other {#th}} birthday!" }
```
**Rich text:**
```json
{ "price": "Price is <b>{amount}</b> with <link>{pct} off</link>" }
```
Rich text support: lingui (macros), react-i18next (`Trans`), react-intl/next-intl (`t.rich()`), vue-i18n.

**Escaping:** Use `'` to escape `{`: `"It '{isn''t}' a tag"`, `"'<notTag>'"`

## File Structure
Always include `common` namespace. Recommended for large apps (combined):
```json
{
  "common": { "save": "Save", "cancel": "Cancel" },
  "HomePage": { "title": "Welcome, {name}" },
  "Products": { "add": "Add Product" }
}
```

## Content Extraction
- **Lingui**: wrap with macros → `lingui extract` → translate `.po` → `lingui compile`
- **next-intl**: experimental `useExtracted` hook

## Validation
**TypeScript (small projects):**
```ts
type Equal<X,Y> = X extends Y ? (Y extends X ? true : false) : false;
type En = typeof import("../messages/en.json");
type Ar = typeof import("../messages/ar.json");
const _: Equal<Equal<En,Ar>, true> = true;
```
**i18n-check CLI** (recommended): finds missing/broken keys across all locales. Run in CI or pre-commit.

## Storage
- JSON/YAML in codebase — translate with LLMs then human review
- TMS for teams: Transifex, Lokalise, Crowdin
⚠️ Always have language specialists review — machine translation alone is not production-ready
