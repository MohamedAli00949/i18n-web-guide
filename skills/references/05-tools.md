# Tools

## Libraries

| Library | Best for | Notes |
|---|---|---|
| [lingui](https://lingui.dev) | JS, React, Vue | Message extraction via macros, lazy translation, AI translation support. Most developer-friendly i18n library. |
| [formatJS / react-intl](https://formatjs.github.io/) | React (enterprise) | Built by Yahoo. Strict ICU/web standards compliance. "Enterprise standard." |
| [i18next / react-i18next](https://www.i18next.com/) | Any JS framework | Full "batteries-included" framework. Highly popular. react-i18next for React. |
| [next-intl](https://next-intl.dev/) | Next.js (App Router) | Gold standard for Next.js. Works with both Server and Client Components. Handles localized routing out of the box. |
| [Vue I18n](https://vue-i18n.intlify.dev/) | Vue.js | Official Vue i18n plugin. |
| [@angular/localize](https://v17.angular.io/api/localize) | Angular | Built-in Angular solution. Translates at compile-time → zero runtime overhead. |
| [Transloco](https://github.com/jsverse/transloco) | Angular (alternative) | More flexible than @angular/localize. Supports runtime language switching without page reload. Better lazy-loading. |

## ESLint Plugins

| Plugin | Purpose |
|---|---|
| [eslint-plugin-i18n-checker](https://www.npmjs.com/package/eslint-plugin-i18n-checker) | Check for existence of i18n keys |
| [eslint-plugin-rtl-friendly](https://www.npmjs.com/package/eslint-plugin-rtl-friendly) | Enforce RTL-friendly CSS styles |

## VSCode Extensions

| Extension | Features |
|---|---|
| [i18n ally](https://marketplace.visualstudio.com/items?itemName=lokalise.i18n-ally) | Message extraction, inline annotations, inline message editing, machine translations |
| [Key Cooker](https://marketplace.visualstudio.com/items?itemName=MohamedAli.key-cooker) | Helps manage JSON/YAML i18n key structures (by Mohamed Ali) |

## Figma Plugins

| Plugin | Purpose |
|---|---|
| [i18n meta](https://www.figma.com/community/plugin/1584517127359156184/i18n-mate) | Generate messages from Figma designs, translate them, convert to variables and keys in the design |
