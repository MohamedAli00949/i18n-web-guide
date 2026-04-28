# Framework Code Patterns

## Next.js App Router + next-intl

```bash
npm install next-intl
```

**File structure:**
```
app/[locale]/layout.tsx   ← sets <html lang dir>
messages/en.json ar.json fr.json
middleware.ts
i18n/routing.ts
```

**`middleware.ts`:**
```ts
import createMiddleware from 'next-intl/middleware';
import { routing } from './i18n/routing';
export default createMiddleware(routing);
export const config = { matcher: ['/((?!api|_next|.*\\..*).*)')] };
```

**`i18n/routing.ts`:**
```ts
import { defineRouting } from 'next-intl/routing';
export const routing = defineRouting({ locales: ['en','ar','fr','de'], defaultLocale: 'en' });
```

**`app/[locale]/layout.tsx`:**
```tsx
import { notFound } from 'next/navigation';
import { NextIntlClientProvider, getMessages } from 'next-intl';
import { routing } from '@/i18n/routing';

export default async function Layout({ children, params: { locale } }) {
  if (!routing.locales.includes(locale)) notFound();
  return (
    <html lang={locale} dir={locale === 'ar' ? 'rtl' : 'ltr'}>
      <body><NextIntlClientProvider messages={await getMessages()}>{children}</NextIntlClientProvider></body>
    </html>
  );
}
```

**Using translations:**
```ts
// Server component
const t = await getTranslations('HomePage');
// Client component
const t = useTranslations('Common');
// Rich text
t.rich('terms', { link: (c) => <a href="/terms">{c}</a> })
// Locale-aware navigation
import { Link, useRouter } from '@/i18n/navigation';
```

---

## Next.js App Router + Lingui (RSC)

Supported since Lingui v4.10.0. Uses React `cache` instead of context — same `Trans`/`useLingui` API works in both server and client components.

```bash
npm install @lingui/react @lingui/core
npm install -D @lingui/swc-plugin @lingui/cli
```

**`next.config.js`:**
```js
module.exports = { experimental: { swcPlugins: [["@lingui/swc-plugin", {}]] } };
```

**`lingui.config.ts`:**
```ts
export default defineConfig({
  sourceLocale: 'en', locales: ['en','ar','fr'],
  catalogs: [{ path: 'src/locales/{locale}/messages', include: ['src'] }],
  format: 'po'
});
```

**`src/appRouterI18n.ts`** — one I18n instance per locale, server-only:
```ts
import { setupI18n } from '@lingui/core';
import { cache } from 'react';
const locales = { en: () => import('../locales/en/messages'), ar: () => import('../locales/ar/messages') };
export const getI18nInstance = cache(async (locale: string) => {
  const { messages } = await locales[locale]();
  return setupI18n({ locale, messages: { [locale]: messages } });
});
```

**`LinguiClientProvider.tsx`** — bridges server → client (I18n instance is not serializable):
```tsx
"use client";
import { I18nProvider } from '@lingui/react';
import { setupI18n, type Messages } from '@lingui/core';
import { useState } from 'react';
export function LinguiClientProvider({ children, initialLocale, initialMessages }:
  { children: React.ReactNode; initialLocale: string; initialMessages: Messages }) {
  const [i18n] = useState(() => setupI18n({ locale: initialLocale, messages: { [initialLocale]: initialMessages } }));
  return <I18nProvider i18n={i18n}>{children}</I18nProvider>;
}
```

**`app/[lang]/layout.tsx`:**
```tsx
import { setI18n } from '@lingui/react/server';
import { getI18nInstance } from '../appRouterI18n';
import { LinguiClientProvider } from '../LinguiClientProvider';
export default async function RootLayout({ params: { lang }, children }) {
  const i18n = await getI18nInstance(lang);
  setI18n(i18n);
  return (
    <html lang={lang} dir={lang === 'ar' ? 'rtl' : 'ltr'}>
      <body>
        <LinguiClientProvider initialLocale={lang} initialMessages={i18n.messages}>
          {children}
        </LinguiClientProvider>
      </body>
    </html>
  );
}
```

**Using translations** — identical API in server and client components:
```tsx
import { Trans, useLingui } from '@lingui/react/macro';
export function MyComponent() {
  const { t } = useLingui(); // reads from cache() in RSC, hook in client
  return <div><Trans>Some Item</Trans><p>{t`Hello`}</p></div>;
}
```

**⚠️ Caveats:**
- Call `setI18n` in **every** page + layout — root layout doesn't re-run on navigation
- Language change = URL redirect, not dynamic switch (server content goes stale)
- Static rendering: use `generateStaticParams`; never define `t()` at module level (frozen at build time)

```bash
npx lingui extract && npx lingui compile
```

---

## React + react-i18next (Vite/CRA)

```bash
npm install i18next react-i18next i18next-browser-languagedetector
```

**`src/i18n.ts`:**
```ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';
import en from './locales/en/common.json';
import ar from './locales/ar/common.json';

i18n.use(LanguageDetector).use(initReactI18next).init({
  resources: { en: { common: en }, ar: { common: ar } },
  fallbackLng: 'en', defaultNS: 'common',
  interpolation: { escapeValue: false }
});
```

**Usage:**
```tsx
const { t, i18n } = useTranslation();
<div dir={i18n.language === 'ar' ? 'rtl' : 'ltr'}>
  {t('greeting', { name: 'Mohamed' })}
</div>
// Plurals: { "items": "{{count}} item", "items_other": "{{count}} items" }
```

---

## React + Lingui

```bash
npm install @lingui/react @lingui/core
npm install -D @lingui/cli @lingui/macro @lingui/vite-plugin
```

**`lingui.config.ts`:**
```ts
export default defineConfig({
  sourceLocale: 'en', locales: ['en','ar','fr'],
  catalogs: [{ path: 'src/locales/{locale}/messages', include: ['src'] }],
  format: 'po'
});
```

**Usage + workflow:**
```tsx
const { t } = useLingui();
<Trans>Welcome</Trans>  {t`Hello, ${name}`}
```
```bash
npx lingui extract   # → .po files for translators
npx lingui compile   # → .js catalogs
```

---

## Shared Format Utility (`lib/format.ts`)

```ts
type Locale = 'en' | 'ar' | 'fr' | 'de';

export const formatCurrency = (v: number, currency: string, locale: Locale) =>
  new Intl.NumberFormat(locale, { style: 'currency', currency }).format(v);
// ⚠️ EGP/SAR symbols may be wrong — use custom component if needed

export const formatDate = (d: Date, locale: Locale, opts?: Intl.DateTimeFormatOptions) =>
  new Intl.DateTimeFormat(locale, { dateStyle: 'medium', ...opts }).format(d);

export function formatRelativeTime(date: Date, locale: Locale) {
  const rtf = new Intl.RelativeTimeFormat(locale, { numeric: 'auto' });
  const s = Math.round((date.getTime() - Date.now()) / 1000);
  if (Math.abs(s) >= 86400) return rtf.format(Math.round(s/86400), 'day');
  if (Math.abs(s) >= 3600)  return rtf.format(Math.round(s/3600), 'hour');
  if (Math.abs(s) >= 60)    return rtf.format(Math.round(s/60), 'minute');
  return rtf.format(s, 'second');
}

export const formatList = (items: string[], locale: Locale, type: Intl.ListFormatType = 'conjunction') =>
  new Intl.ListFormat(locale, { style: 'long', type }).format(items);

export const getDirection = (locale: Locale) =>
  ['ar','he','fa','ur'].includes(locale) ? 'rtl' : 'ltr';
```

---

## RTL Tailwind
```tsx
// ✅ flips in RTL
<div className="ps-4 pe-2 text-start border-s-2">
// ❌ avoid
<div className="pl-4 pr-2 text-left border-l-2">
// flip icons
<div className="ltr:rotate-0 rtl:rotate-180">
```

---

## Translation File Structure
```json
// messages/en.json — source of truth; all locales mirror exactly
{
  "common": { "save": "Save", "cancel": "Cancel" },
  "HomePage": {
    "title": "Welcome, {name}",
    "subtitle": "{n, plural, =0 {No messages} one {# message} other {# messages}}"
  }
}
```
```ts
// TypeScript validation — types/i18n.ts
import type en from '../messages/en.json';
declare global { interface IntlMessages extends typeof en {} }
```

---

## Common Bugs & Fixes
| Bug | Cause | Fix |
|---|---|---|
| `locale` undefined in component | Reading URL manually | Use `useLocale()` |
| No default locale redirect | Missing middleware `matcher` | Add `/((?!api\|_next\|.*\\..*).*)`|
| Arabic dates show Latin numbers | No numberingSystem | Pass `{ numberingSystem: 'arab' }` |
| RTL not flipping | `dir` on wrong element | Set on `<html>`, not `<body>` |
| Missing key warnings | Key missing in target locale | Run `i18n-check` |
| Server/client mismatch | `useTranslations` in server component | Use `getTranslations` (async) |
