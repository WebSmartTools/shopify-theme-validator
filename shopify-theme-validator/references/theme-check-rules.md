# Theme-Check Rules Reference

The full rule reference for the shopify-theme-validator skill. Read this before writing or fixing a section schema. Every rule here is from a real source: Shopify's theme-check 3.24.0 (installed in the shopify-liquid skill), the project's own validator (`validate.py` in the WebSmart theme v1.3.0 work folder), or the v1.2.1 upload defect.

## 1. Names: 25 Characters Max

Shopify enforces a 25-character limit on schema names. The limit counts every character, including spaces.

- **Section name.** The `name` field at the top of a section schema. theme-check catches this one (ValidSchemaName, error). A name over 25 characters is reported as: `Schema name '<name>' is too long (max 25 characters)`.
- **Block name.** The `name` field of each entry in a section schema's `blocks` array. theme-check does NOT check this. ValidSchemaName reads only the section's top-level `name` and its locale translation; it never looks at `blocks[].name`. This validator checks every block name. Shopify rejects a long block name on save with: `Invalid block '<type>': name is too long (max 25 characters)`.

The v1.2.1 defect, the reason this rule exists: two long block names in the theme made the upload drop `templates/product.json` and `sections/main-product.liquid` silently, with no error shown. The theme had passed theme-check. The fixed names were "Made to order" (13 characters) and "Personalization: text" (21 characters). Check every block name before delivery.

## 2. Setting Ids: Unique Inside Each Settings Array

A duplicate `id` inside one settings array breaks the schema.

- **What theme-check checks.** UniqueSettingsId targets `settings_schema.json` only. Its own description says: "This check is aimed at eliminating duplicate Ids in settings_schema.json".
- **What it misses.** Duplicate ids inside a section's `settings` array, or inside one block type's `settings` array, are not checked. A test theme with 28 sections, each carrying one deliberate mistake, showed theme-check flagging only 13 and missing 10, including duplicate setting `id`s inside sections and blocks. This validator checks every settings array: the section settings, each block type's settings, and `settings_schema.json`.
- **Scope.** A setting `id` may repeat across different scopes. A section setting and a block setting can both be `text`, and two different block types can both have a `text` setting. Duplicates only count inside one array.

**Fixing duplicates: `validate.py --fix`.** The validator can repair a duplicate instead of only reporting it. For each settings array with a duplicate, `--fix` keeps the first occurrence and renames the later ones to `<id>_2`, `<id>_3`, and so on, then rebinds the Liquid references that read them: the first reference to `section.settings.<id>` still reads the first setting, the second reference is rewritten to `section.settings.<id>_2`, and so on. The same rule applies to `block.settings.<id>` references, except when the id is duplicated in more than one block type, in which case the references are left alone and the report says to update them by hand. Bracket-notation references (`section.settings['<id>']`) are not rewritten; the report flags them. Fixed files are written under `fixed/` in the current directory, mirroring the theme structure, and the original files are never overwritten. Review the fixed files, then move them over the originals yourself. A renamed setting has no saved value in existing templates, because template JSON stores settings by id, so set it again in the editor after moving the file.

**Reserved ids: `size`, `first`, `last`.** Those three names are built-in Liquid filters, and settings are read with the same dot notation (`section.settings.size`, `block.settings.size`, `settings.size`). Shopify's own Liquid docs, on the metafield object page, document what happens: with dot notation, Liquid returns the value when the key exists, but applies the filter when it is missing, so a missing `size` returns a count instead of empty. The same failure applies to a setting id: the setting saves fine, so nothing refuses it, and a missing value silently returns the wrong value. The validator flags these three ids as warnings, in section settings, block settings, and `settings_schema.json`. Rename the id, or read it with bracket notation, for example `section.settings['size']`.

**Not reserved: `title`.** Shopify's section-schema docs list `heading`, `title`, and `text` as the ids the theme editor uses, in that order, for a block's title in the sidebar. A setting id of `title` is legitimate: the editor shows that setting's value as the block title. Do not flag it, and do not rename it.

## 3. Deprecated Image Filters

theme-check's DeprecatedFilter check flags four filters. Its fix message for all four is: replace the filter with `image_url`.

| Deprecated filter | Replace with |
| --- | --- |
| `img_url` | `image_url` |
| `article_img_url` | `image_url` (on the article image object) |
| `collection_img_url` | `image_url` (on the collection image object) |
| `product_img_url` | `image_url` (on the product featured image object) |

Not deprecated, and fine to use: `asset_img_url`, `file_img_url`, `payment_type_img_url`.

## 4. Prices: The Money Filter

Money comes from `shop.money_format`. Never hardcode a currency.

- A formatted price written literally in code, such as `$49.99`, is an error. Render prices with the money filter, for example `{{ product.price | money }}`.
- A whole-dollar amount in copy, such as `$75` in "Free shipping on orders over $75", is a warning. It is a hardcoded amount that is wrong for stores in another currency, but it is copy, not a rendered price.
- The project's own validator treated any literal `$` amount as a warning. This validator raises formatted prices to errors on purpose, because a formatted price is unambiguously a rendered price, not copy.

## 5. Structured Data: Wrap It

`structured_data` outputs bare JSON. Always wrap it:

```liquid
<script type="application/ld+json">{{ product | structured_data }}</script>
```

Same for `article`. Bare output renders as visible text on the page. This was a real defect in the WebSmart theme: the product and article structured data were bare, and the fix was the script wrapper.

## 6. Placeholder Copy

These exact phrases must not appear in theme code that shoppers can see:

- "Sample product"
- "$00.00"
- "replace with a real customer quote"
- "Demo text"
- `placeholder_svg_tag`
- "Sample review"
- "Sample customer"

Replace them with real copy before delivery. The validator reports the file and the phrase.

## 7. Templates and Sections Must Agree

- **A section file must exist before a template that references it saves.** Shopify refuses the template save otherwise. Create the section first, then the template. theme-check catches this one (MissingTemplate, error).
- **Every block type a template uses must be defined in that section's schema.** theme-check catches this one too (JsonMissingBlock, error). This validator checks it as well, because it is the most common template break after a rename.
- **Template `block_order` entries must exist.** Every id in a section's `block_order` must be a block in that section's `blocks` object. A stale id in `block_order` is an error.

## 8. Layout Files

Layout files (`layout/theme.liquid`, `layout/password.liquid`) must contain `{{ content_for_header }}` and `{{ content_for_layout }}`. A layout file missing either one fails to save. theme-check catches this one (RequiredLayoutThemeObject, error). A section file saved under `layout/` fails with `Missing {{content_for_header}} in the head section of the template`.

## 9. The Default Filter-Chain Trap

Do not chain `default` onto a whole tag. For example:

```liquid
alt: x | default: y, class: z
```

This applies `default` to the whole tag and drops the class. Assign the fallback to a variable first:

```liquid
{%- assign alt_text = x | default: y -%}
alt: alt_text, class: z
```

The validator flags the chained form as a warning.

## 10. What the Validator Does Not Check

- **Looks.** Validation is not a visual pass. A clean run means the code is sound. The theme still needs a preview on a real store and a human eye.
- **Dawn, Horizon, or Heritage references.** The project's own validator flagged those names, but that is a rule for one theme, not a general rule. A general validator must not flag a theme for mentioning Dawn, for example in migration notes. This validator does not check for those names.
- **JavaScript syntax when node is not installed.** The validator runs `node --check` on `assets/*.js` when node is available. When node is not installed, it skips the check and says so in a note instead of failing.
