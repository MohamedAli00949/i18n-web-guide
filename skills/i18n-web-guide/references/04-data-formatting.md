# Data Formatting (Intl API)

Use `Intl.*` objects — configure once at init with numbering system, calendar, and date format.
Polyfills: [format.js](https://formatjs.github.io/docs/polyfills/)

## Dates

⚠️ Always set the calendar system explicitly — see [ICU calendars](https://unicode-org.github.io/icu/userguide/datetime/calendar/#calendar).

```js
new Intl.DateTimeFormat("en-GB", { dateStyle:"full", timeStyle:"long", timeZone:"Australia/Sydney" }).format(date)
// "Sunday, 20 December 2020 at 14:23:16 GMT+11"

new Intl.DateTimeFormat("ar-EG", { dateStyle:"full", timeStyle:"short", timeZone:"Africa/Cairo" }).format(date)
// "الأحد، ٢٠ ديسمبر ٢٠٢٠ في ٥:٢٣ ص"
```
Also: `formatRange()`, `formatToParts()`.

```js
new Intl.DurationFormat("en", { style:"long" }).format({ hours:1, minutes:46, seconds:40 })
// "1 hour, 46 minutes, 40 seconds"

new Intl.RelativeTimeFormat("en", { numeric:"auto" }).format(1, "day")  // "tomorrow"
new Intl.RelativeTimeFormat("ar", { numeric:"auto" }).format(0, "day")  // "اليوم"
```
⚠️ RelativeTimeFormat works best with Gregorian calendar.

## Numbers

⚠️ EGP/SAR symbols may not render correctly — build a custom component if needed.

```js
new Intl.NumberFormat("ar-EG", { style:"currency", currency:"EGP" }).format(654321.99)
// "٦٥٤٬٣٢١٫٩٩ ج.م."

new Intl.NumberFormat("en-US", { style:"currency", currency:"USD" }).format(654321.99)
// "$654,321.99"
```
Styles: `currency`, `percent`, `unit`, `decimal`. Also: `formatRange()`.

## Lists

```js
new Intl.ListFormat("en", { style:"short", type:"disjunction" }).format(["A","B","C"])
// "A, B, or C"
```
Types: `conjunction` (and), `disjunction` (or), `unit`. Styles: `long`, `short`, `narrow`.
⚠️ Check `Intl.ListFormat.supportedLocalesOf()` — fallback is English.

## Locale Utilities

```js
const arEg = new Intl.Locale("ar-EG");
arEg.getCalendars()       // ["gregory","coptic","islamic",...]
arEg.getTextInfo()        // { direction:"rtl" }
arEg.getWeekInfo()        // { firstDay:6, weekend:[5,6] }
arEg.getNumberingSystems() // ["arab"]

new Intl.DisplayNames(["ar"], { type:"region" }).of("US")    // "الولايات المتحدة"
new Intl.DisplayNames(["ar"], { type:"currency" }).of("USD") // "دولار أمريكي"

["Z","a","z","ä"].sort(new Intl.Collator("de").compare) // ["a","ä","z","Z"]
```

## Intl.Segmenter
Locale-aware split into graphemes / words / sentences.
Use for: character limits (Arabic grapheme ≠ string length), cursor movement in RTL, word search highlighting, text editors.
```js
const seg = new Intl.Segmenter("ar", { granularity:"grapheme" });
const count = [...seg.segment(text)].length; // correct char count for Arabic
```
