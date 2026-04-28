# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, Antigravity, etc.) when working with code in this repository.

A collection of skills for Claude.ai and Claude Code for senior software engineers. Skills are packaged instructions and scripts that extend Claude and your coding agents capabilities.

## Repository Overview

This repository contains AI skills for Claude. Each skill is a folder with a `SKILL.md` entry point and reference files under `references/`.

## Repo Structure

```
skills/
  i18n-web-apps/
    SKILL.md                          ← entry point, always read first
    references/
      00-master-references.md         ← all source links (lazy-load only)
      01-routing.md                   ← URL routing patterns
      02-seo.md                       ← hreflang, sitemap, metadata
      03-messages.md                  ← ICU format, translation files
      04-data-formatting.md           ← Intl.* API usage
      05-tools.md                     ← library + tool comparison
      06-ui.md                        ← RTL, direction, logical CSS
      07-framework-patterns.md        ← ready-to-use code per framework
```

## Editing Rules

- **SKILL.md** is the index and rules file — keep it under 600 tokens. No full code blocks here.
- **Reference files** are loaded on demand per topic — one topic per file, no cross-file duplication.
- **`00-master-references.md`** is lazy-loaded only. Never list it in SKILL.md's active index.
- Every code example must be specific to JS/React/Next.js. No generic pseudocode.
- All rules in `SKILL.md` must be actionable, not descriptive.

## Adding a New Framework or Library

1. Add a new section to `references/07-framework-patterns.md`.
2. If the library appears in the stack table in `SKILL.md`, update that row.
3. Update the relevant row in `references/05-tools.md`.
4. Fetch the official docs URL before writing — never write patterns from memory alone.

## Token Budget (Optimized Version)

| File | Target |
|---|---|
| `SKILL.md` | < 600 tokens |
| Each reference file | < 700 tokens |
| `07-framework-patterns.md` | < 2,000 tokens (code-heavy) |
| Total skill | < 6,500 tokens |

If a file exceeds its budget, split it or trim prose to bullet points.

## Content Rules

- **NEVER** use `letter-spacing` examples on Arabic/Persian — it's explicitly an anti-pattern in this skill.
- **NEVER** recommend `margin-left`/`margin-right` — always logical CSS equivalents.
- **ALWAYS** note EGP/SAR currency symbol limitation when writing `Intl.NumberFormat` examples.
- **ALWAYS** include the `setI18n` caveat when writing Lingui RSC patterns.
- Keep code blocks minimal — show critical lines only, use `// ...` for skippable boilerplate.

## Boundaries

- **NEVER** modify `00-master-references.md` links without verifying the URL still resolves.
- **NEVER** add Vue, Angular, or non-JS framework patterns to this skill — it targets JS/React/Next.js only.
- **NEVER** auto-generate translation content or recommend skipping human review.
- Do not rewrite the full skill to fix a single section — make targeted edits only.

## Author

[Mohamed Ali](https://github.com/MohamedAli00949) — [Substack series](https://mohamedali00949.substack.com/p/my-journey-with-i18n-for-web-apps)
