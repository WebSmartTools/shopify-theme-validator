# Changelog

## v1.0.1 - 2026-11-16

- `validate.py --fix`: repairs duplicate setting ids. It keeps the first occurrence, renames the later ones to `<id>_2`, `<id>_3`, and so on, and rebinds the Liquid references that read them positionally (the first reference still reads the first setting, the second is rewritten to the renamed id). Fixed files are written under `fixed/` in the current directory, mirroring the theme structure, and the original files are never overwritten.
- Reserved setting ids: the validator now warns on setting ids named `size`, `first`, or `last`, which collide with Liquid's built-in filters of the same names. Shopify's own Liquid docs document the failure mode: with dot notation, a missing value silently returns the wrong value instead of empty. `title` is deliberately not flagged: Shopify's section-schema docs list `heading`, `title`, and `text` as the ids the theme editor uses for a block's title, so a setting id of `title` is legitimate.
- Two new premium reference files: `mega-menu.liquid` (a mega menu section with column and promo blocks) and `footer.liquid` (a footer section with link list and newsletter blocks and a `locales` object with English and German strings). Both pass the validator with no errors and no warnings.
- New premium doc: `cli-rescue-toolkit.md`, the rescue steps when the Shopify CLI misbehaves (ghost dev server on port 9292, per-theme `.shopify` cache, login and global cache reset, what logging out deletes).
- Price unchanged: $15, one-time payment, single-user licence.

## v1.0.0 - 2026-11-16

First release.

- Free skill: the shopify-theme-validator skill for Claude Code, built from the real rules in the shopify-liquid skill and the WebSmart theme v1.3.0 validator (block names 25 characters max, duplicate setting ids, deprecated img_url, price formatting through the money filter, bare structured_data, placeholder copy, layout content_for objects, the default filter-chain trap). Includes the theme-check rules reference (`references/theme-check-rules.md`), install steps for Claude Code, Codex, Cursor, and Claude.ai, a worked hero banner example, a template example, and a validator checklist.
- Premium pack: the free skill plus the pre-upload validator (`validate.py`, Python 3 standard library only), four fresh generic reference files (hero banner section, image with text section, template, settings schema), and the push-to-unpublished checklist.
- Price: $15, one-time payment, single-user licence.
