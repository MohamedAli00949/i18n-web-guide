# Framework Code Patterns

Ready-to-use implementation patterns for JS/React/Next.js stacks.

---

## Next.js App Router + next-intl (Recommended)

### 1. Install
```bash
npm install next-intl
```

### 2. File structure
```
app/
  [locale]/
    layout.tsx       ← sets <html lang dir>
    page.tsx
messages/
  en.json
  ar.json
  fr.json
middleware.ts        ← locale detection + redirect
next.config.ts
```

### 3. `middleware.ts`
```ts
import createMiddleware from 'next-intl/middleware';
import { routing } from './i18n/routing';

export default createMiddleware(routing);

export const config = {
  matcher: ['/((?!api|_next|.*\\..*).*)']
};
```

### 4. `i18n/routing.ts`
```ts
import { defineRouting } from 'next-intl/routing';

export const routing = defineRouting({
  locales: ['en', 'ar', 'fr', 'de'],
  defaultLocale: 'en'
});
```

### 5. `app/[locale]/layout.tsx`
```tsx
import { notFound } from 'next/navigation';
import { NextIntlClientProvider } from 'next-intl';
import { getMessages } from 'next-intl/server';
import { routing } from '@/i18n/routing';

type Props = { children: React.ReactNode; params: { locale: string } };

export default async function LocaleLayout({ children, params: { locale } }: Props) {
  if (!routing.locales.includes(locale as any)) notFound();

  const messages = await getMessages();
  const dir = locale === 'ar' ? 'rtl' : 'ltr';

  return (
    <html lang={locale} dir={dir}>
      <body>
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  );
}
```

### 6. Using translations

**Server component:**
```tsx
import { getTranslations } from 'next-intl/server';

export default async function Page() {
  const t = await getTranslations('HomePage');
  return <h1>{t('title')}</h1>;
}
```

**Client component:**
```tsx
'use client';
import { useTranslations } from 'next-intl';

export function Button() {
  const t = useTranslations('Common');
  return <button>{t('save')}</button>;
}
```

**Rich text:**
```tsx
t.rich('terms', {
  link: (chunks) => <a href="/terms">{chunks}</a>
})
```

### 7. Locale-aware `Link` and `useRouter`
```tsx
import { Link, useRouter, usePathname } from '@/i18n/navigation';
// next-intl wraps Next.js navigation to prepend the locale automatically
```

## Next.js App Router + Lingui (RSC)

Lingui supports React Server Components as of v4.10.0. Unlike next-intl which uses React context for client components, Lingui uses React `cache()` to maintain an I18n instance per request on the server — enabling the same `Trans` and `useLingui` API to work identically in both server and client components.

### 1. Install
```bash
npm install @lingui/react @lingui/core
npm install -D @lingui/swc-plugin @lingui/cli
```

### 2. `next.config.js`
```js
/** @type {import('next').NextConfig} */
module.exports = {
  experimental: {
    swcPlugins: [["@lingui/swc-plugin", {}]]
  }
};
```

### 3. `lingui.config.ts`
```ts
import { defineConfig } from '@lingui/conf';

export default defineConfig({
  sourceLocale: 'en',
  locales: ['en', 'ar', 'fr'],
  catalogs: [{ path: 'src/locales/{locale}/messages', include: ['src'] }],
  format: 'po'
});
```

### 4. `src/appRouterI18n.ts` (server-only)

Holds one I18n instance per locale for the entire app. Uses React `cache()` so each request gets an isolated instance.

```ts
import { setupI18n } from '@lingui/core';
import { cache } from 'react';

const localeImports = {
  en: () => import('../locales/en/messages'),
  ar: () => import('../locales/ar/messages'),
  fr: () => import('../locales/fr/messages'),
};

export const getI18nInstance = cache(async (locale: string) => {
  const { messages } = await localeImports[locale]();
  return setupI18n({ locale, messages: { [locale]: messages } });
});
```

### 5. `LinguiClientProvider.tsx` (client bridge)

The I18n instance is not serializable and cannot be passed directly from server to client. Instead, pass `initialLocale` and `initialMessages` (plain serializable values) and reconstruct the instance on the client.

```tsx
"use client";

import { I18nProvider } from "@lingui/react";
import { type Messages, setupI18n } from "@lingui/core";
import { useState } from "react";

export function LinguiClientProvider({
  children,
  initialLocale,
  initialMessages,
}: {
  children: React.ReactNode;
  initialLocale: string;
  initialMessages: Messages;
}) {
  const [i18n] = useState(() =>
    setupI18n({
      locale: initialLocale,
      messages: { [initialLocale]: initialMessages },
    })
  );
  return <I18nProvider i18n={i18n}>{children}</I18nProvider>;
}
```

### 6. `app/[lang]/layout.tsx`

```tsx
import { setI18n } from "@lingui/react/server";
import { getI18nInstance } from "../appRouterI18n";
import { LinguiClientProvider } from "../LinguiClientProvider";

type Props = { params: { lang: string }; children: React.ReactNode };

export default async function RootLayout({ params: { lang }, children }: Props) {
  const i18n = await getI18nInstance(lang);
  setI18n(i18n); // makes i18n available to all server components in this request

  const dir = lang === 'ar' ? 'rtl' : 'ltr';

  return (
    <html lang={lang} dir={dir}>
      <body>
        <LinguiClientProvider initialLocale={lang} initialMessages={i18n.messages}>
          {children}
        </LinguiClientProvider>
      </body>
    </html>
  );
}
```

### 7. Using translations

The same `Trans` and `useLingui` API works in **both server and client components** — no need to split translation logic by component type.

```tsx
import { Trans, useLingui } from "@lingui/react/macro";

export function SomeComponent() {
  const { t } = useLingui();
  // In RSC, useLingui() is not a hook — it reads from React cache()
  return (
    <div>
      <Trans>Some Item</Trans>
      <p>{t`Other Item`}</p>
    </div>
  );
}
```

### 8. Extract + compile workflow
```bash
npx lingui extract   # scans source files, updates .po catalogs
# → send .po files to translators or TMS
npx lingui compile   # compiles .po → .js message catalogs
```

### Key caveats

- **Call `setI18n` in every page and layout** — not just the root layout. Because Next.js App Router layouts preserve state and don't re-render on navigation, the `setI18n` call in the root layout won't run again on page transitions. Each nested page and layout must call it independently.
- **Language change = URL redirect** — redirect users to the new locale URL rather than switching dynamically. Server-rendered locale-dependent content would go stale with dynamic switching.
- **Static rendering** — use `generateStaticParams` to build pages for all locales at build time. Never define translated strings at module level (outside components), as they'll be frozen to whichever locale was active when the module was first imported:

```tsx
// ❌ Frozen to build-time locale
const greeting = t(i18n)`Hello World`;

// ✅ Rendered per locale via generateStaticParams
export default function Page() {
  return <Trans>Hello world</Trans>;
}
```

Reference: [lingui.dev/tutorials/react-rsc](https://lingui.dev/tutorials/react-rsc)

---

## React + react-i18next (Vite / CRA)

### 1. Install
```bash
npm install i18next react-i18next i18next-browser-languagedetector
```

### 2. `src/i18n.ts`
```ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

import en from './locales/en/common.json';
import ar from './locales/ar/common.json';

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: { en: { common: en }, ar: { common: ar } },
    fallbackLng: 'en',
    defaultNS: 'common',
    interpolation: { escapeValue: false }
  });

export default i18n;
```

### 3. `main.tsx`
```tsx
import './i18n'; // must import before App
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(<App />);
```

### 4. Using translations
```tsx
import { useTranslation } from 'react-i18next';

export function Greeting() {
  const { t, i18n } = useTranslation();
  return (
    <div dir={i18n.language === 'ar' ? 'rtl' : 'ltr'}>
      <p>{t('greeting', { name: 'Mohamed' })}</p>
      <button onClick={() => i18n.changeLanguage('ar')}>عربي</button>
    </div>
  );
}
```

### 5. Plural example
```json
// en/common.json
{ "items": "{{count}} item", "items_other": "{{count}} items" }
```
```tsx
t('items', { count: 5 }) // "5 items"
```

---

## React + Lingui (Vite / CRA)

### 1. Install + configure
```bash
npm install @lingui/react @lingui/core
npm install -D @lingui/cli @lingui/macro @lingui/vite-plugin
```

### 2. `lingui.config.ts`
```ts
import { defineConfig } from '@lingui/conf';

export default defineConfig({
  sourceLocale: 'en',
  locales: ['en', 'ar', 'fr'],
  catalogs: [{ path: 'src/locales/{locale}/messages', include: ['src'] }],
  format: 'po'
});
```

### 3. Using macros (extractable)
```tsx
import { Trans, useLingui } from '@lingui/react/macro';

export function Welcome({ name }: { name: string }) {
  const { t } = useLingui();
  return (
    <div>
      <Trans>Welcome to our app</Trans>
      <p>{t`Hello, ${name}`}</p>
    </div>
  );
}
```

### 4. Extract + compile workflow
```bash
npx lingui extract   # scans source, updates .po files
# → send .po files to translators
npx lingui compile   # compiles .po → .js catalogs
```

---

## Data Formatting Utility (`lib/format.ts`)

Create once, use everywhere across any framework:

```ts
type Locale = 'en' | 'ar' | 'fr' | 'de';

export function formatCurrency(value: number, currency: string, locale: Locale) {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency,
    // Note: EGP and SAR symbols may not render correctly in all locales.
    // Wrap in a custom display component if accuracy is required.
  }).format(value);
}

export function formatDate(date: Date, locale: Locale, options?: Intl.DateTimeFormatOptions) {
  return new Intl.DateTimeFormat(locale, {
    dateStyle: 'medium',
    ...options
  }).format(date);
}

export function formatRelativeTime(date: Date, locale: Locale): string {
  const rtf = new Intl.RelativeTimeFormat(locale, { numeric: 'auto' });
  const diff = date.getTime() - Date.now();
  const seconds = Math.round(diff / 1000);
  const minutes = Math.round(seconds / 60);
  const hours = Math.round(minutes / 60);
  const days = Math.round(hours / 24);

  if (Math.abs(days) > 0) return rtf.format(days, 'day');
  if (Math.abs(hours) > 0) return rtf.format(hours, 'hour');
  if (Math.abs(minutes) > 0) return rtf.format(minutes, 'minute');
  return rtf.format(seconds, 'second');
}

export function formatList(items: string[], locale: Locale, type: Intl.ListFormatType = 'conjunction') {
  return new Intl.ListFormat(locale, { style: 'long', type }).format(items);
}

export function getDirection(locale: Locale): 'ltr' | 'rtl' {
  const rtlLocales = ['ar', 'he', 'fa', 'ur'];
  return rtlLocales.includes(locale) ? 'rtl' : 'ltr';
}
```

---

## RTL-aware Tailwind CSS

### 1. Enable logical properties
```js
// tailwind.config.js — no extra config needed in Tailwind v3.3+
// Use ps-* (padding-inline-start) and pe-* (padding-inline-end)
// Use ms-* and me-* for margin-inline-start/end
```

### 2. Usage
```tsx
// ✅ Correct — flips automatically in RTL
<div className="ps-4 pe-2 text-start border-s-2">

// ❌ Avoid — fixed direction
<div className="pl-4 pr-2 text-left border-l-2">
```

### 3. RTL-specific overrides (if needed)
```tsx
<div className="ltr:rotate-0 rtl:rotate-180"> {/* for icons that need flipping */}
```

---

## Translation File Structure (Recommended for Next.js)

```
messages/
  en.json    ← source of truth
  ar.json    ← mirrors en.json structure exactly
  fr.json
  de.json
```

```json
// messages/en.json
{
  "common": {
    "save": "Save",
    "cancel": "Cancel",
    "loading": "Loading..."
  },
  "HomePage": {
    "title": "Welcome, {name}",
    "subtitle": "You have {count, plural, =0 {no messages} one {# message} other {# messages}}"
  },
  "ProductsPage": {
    "actions": {
      "addProduct": "Add Product",
      "deleteProduct": "Delete Product"
    }
  }
}
```

### TypeScript validation
```ts
// types/i18n.ts — ensures all locales match English structure
import type en from '../messages/en.json';
type Messages = typeof en;

declare global {
  interface IntlMessages extends Messages {}
}
```

---

## Common Next.js i18n Bugs & Fixes

| Bug | Cause | Fix |
|---|---|---|
| `locale` is undefined in component | Reading from URL manually | Use `useLocale()` from next-intl |
| Page not redirecting to default locale | Missing `matcher` in middleware config | Add `/((?!api\|_next\|.*\\..*).*)`|
| Arabic date showing Gregorian numbers | No `numberingSystem` config | Pass `{ numberingSystem: 'arab' }` to `Intl.DateTimeFormat` |
| RTL layout not flipping | `dir` set on wrong element | Set `dir` on `<html>`, not `<body>` or a wrapper div |
| Missing translation key warning | Key exists in `en.json` but not `ar.json` | Run `i18n-check` or TypeScript type validation |
| `next-intl` server/client mismatch | Using `useTranslations` in a server component | Use `getTranslations` (async) in server components |
| Lingui translations frozen to one locale | Module-level `t()` call at build time | Move translation inside the component or use lazy translation |
| Lingui RSC — wrong locale in nested page | `setI18n` only called in root layout | Call `setI18n` in every page and layout independently |
