# Data Formatting

Use ECMAScript `Intl` objects instead of writing manual formatting functions. Configure them once during i18n initialization with numbering systems, date formats, and calendar settings.

> Polyfills for all `Intl` objects are available via [format.js](https://formatjs.github.io/docs/polyfills/).

## Date & Time

> ⚠️ Always identify the correct **calendar system** for the locale to avoid unexpected date formats. See [ICU supported calendars](https://unicode-org.github.io/icu/userguide/datetime/calendar/#calendar).

### `Intl.DateTimeFormat`

```js
// English (GB), full date + long time
new Intl.DateTimeFormat("en-GB", {
	dateStyle: "full",
	timeStyle: "long",
	timeZone: "Australia/Sydney",
}).format(date);
// "Sunday, 20 December 2020 at 14:23:16 GMT+11"

// Arabic (Egypt)
new Intl.DateTimeFormat("ar-EG", {
	dateStyle: "full",
	timeStyle: "short",
	timeZone: "Africa/Cairo",
}).format(date);
// "الأحد، ٢٠ ديسمبر ٢٠٢٠ في ٥:٢٣ ص"
```

Also supports `formatRange` for date ranges and `formatToParts`.

### `Intl.DurationFormat`

```js
const duration = { hours: 1, minutes: 46, seconds: 40 };

new Intl.DurationFormat("en", { style: "long" }).format(duration);
// "1 hour, 46 minutes, 40 seconds"

new Intl.DurationFormat("ar", { style: "long" }).format(duration);
// "ساعة، و46 دقيقة، و40 ثانية"
```

Duration object properties: `years`, `months`, `weeks`, `days`, `hours`, `minutes`, `seconds`, `milliseconds`, `microseconds`, `nanoseconds`.
Use `Temporal.Duration` or its [polyfill](https://www.npmjs.com/package/temporal-polyfill) to calculate durations.

### `Intl.RelativeTimeFormat`

```js
const rtfEn = new Intl.RelativeTimeFormat("en", {
	style: "short",
	numeric: "auto",
});
const rtfAr = new Intl.RelativeTimeFormat("ar", { numeric: "auto" });

rtfEn.format(0, "day"); // "today."
rtfAr.format(0, "day"); // "اليوم"
rtfEn.format(1, "day"); // "tomorrow."
rtfAr.format(1, "day"); // "غدًا."
```

> ⚠️ Relative time calculation methodology works best with the **Gregorian calendar**. May not be accurate for other calendar systems.

## Number Formatting

### `Intl.NumberFormat`

> ⚠️ Some currency symbols are not accurately rendered for all locales. For example, SAR prints `ر.س.` and EGP may show `E£` instead of the correct symbol. In these cases, build a custom formatting component.

```js
const amount = 654321.987;

// EGP in Arabic
new Intl.NumberFormat("ar-EG", { style: "currency", currency: "EGP" }).format(
	amount,
);
// "٦٥٤٬٣٢١٫٩٩ ج.م."

// USD in English
new Intl.NumberFormat("en-US", { style: "currency", currency: "USD" }).format(
	amount,
);
// "$654,321.99"

// Range formatting
new Intl.NumberFormat("en-US", {
	style: "currency",
	currency: "USD",
	maximumFractionDigits: 0,
}).formatRange(3, 5);
// "$3 – $5"
```

Number style options: `currency`, `percent`, `unit`, `decimal`.

## List Formatting

### `Intl.ListFormat`

Relationship types:

- `conjunction` → "A, B, and C"
- `disjunction` → "A, B, or C"
- `unit` → "A, B, C"

Style options: `long`, `short`, `narrow`

```js
const names = ["John", "Joe", "Martin"];

new Intl.ListFormat("en", { style: "short", type: "disjunction" }).format(
	names,
);
// "John, Joe, or Martin"

new Intl.ListFormat("de", { style: "short", type: "conjunction" }).format(
	names,
);
// "John, Joe und Martin"
```

> ⚠️ Validate locale support with `Intl.ListFormat.supportedLocalesOf()`. The default fallback is English.

## Locale & Constants

### `Intl.Locale`

Inspect locale properties: calendars, hour cycles, numbering systems, text direction, week info.

```js
const arEg = new Intl.Locale("ar-EG");
arEg.getCalendars(); // ["gregory", "coptic", "islamic", ...]
arEg.getTextInfo(); // { direction: "rtl" }
arEg.getWeekInfo(); // { firstDay: 6, weekend: [5, 6] }
arEg.getNumberingSystems(); // ["arab"]
```

### `Intl.DisplayNames`
Get localized names for regions, languages, currencies, calendars:
```js
new Intl.DisplayNames(["en"], { type: "region" }).of("US"); // "United States"
new Intl.DisplayNames(["ar"], { type: "region" }).of("US"); // "الولايات المتحدة"

new Intl.DisplayNames(["en"], { type: "currency" }).of("USD"); // "US Dollar"
new Intl.DisplayNames(["ar"], { type: "currency" }).of("USD"); // "دولار أمريكي"
```

## String Utilities

### `Intl.Collator`

Language-sensitive string sorting:

```js
["Z", "a", "z", "ä"].sort(new Intl.Collator("de").compare);
// ["a", "ä", "z", "Z"]
```

### `Intl.Segmenter`

Locale-sensitive text segmentation into graphemes, words, or sentences.

**Real-world use cases:**

- **Correct character limits** — chat apps, form validation, CMS fields, exam answers
- **Safe cursor movement & deletion** — rich text editors, mobile inputs, Arabic/Hindi/Thai fields
- **Word detection** for search & highlighting in RTL languages
- **Sentence splitting** for large text processing
- **Validation rules** that respect language

```js
// Count real graphemes in Arabic (important: Arabic string length !== grapheme count)
const segmenter = new Intl.Segmenter("ar", { granularity: "grapheme" });
const graphemes = [...segmenter.segment(arabicText)];
// graphemes.length will differ from arabicText.length
```

## Initialization Pattern

When setting up i18n, configure formatting objects once with:

- **Numbering systems** per supported language
- **Date formats** with specific calendar (prevents unexpected format changes)
- **Global format presets** — call with locale, get pre-configured formatter
