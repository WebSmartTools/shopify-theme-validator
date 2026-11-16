# Validator Checklist

Run through this list before every theme upload. Every item is checked by `validate.py` in the premium pack, except the last two, which need a human eye.

## Before you run the validator

- [ ] Every section file you changed is saved, and every new section has a `{% schema %}` block.
- [ ] Every template you changed references only sections that exist as files.

## The validator run

- [ ] Run `python3 validate.py /path/to/theme` from the theme root.
- [ ] Zero errors. Warnings are notes to check by hand, not blockers.

## What the errors mean

- [ ] Block name over 25 characters: shorten the name in the section schema. Count spaces too.
- [ ] Section name over 25 characters: shorten the name in the schema.
- [ ] Duplicate setting id: rename one of the two settings inside the same settings array.
- [ ] Deprecated filter (`img_url`, `article_img_url`, `collection_img_url`, `product_img_url`): replace it with `image_url`.
- [ ] Bare structured_data: wrap it in `<script type="application/ld+json">`.
- [ ] Placeholder copy: replace the phrase with real copy.
- [ ] Hardcoded formatted price, such as $49.99: render it with the money filter.
- [ ] Template references a missing section: create the section file first.
- [ ] Template uses a block type the section does not define: add the block to the section schema, or fix the type in the template.
- [ ] Layout missing content_for_header or content_for_layout: add the missing object to the layout file.

## Before you publish

- [ ] Push to an unpublished theme first and preview it on a real store.
- [ ] Check the routes: home, a product page, a collection page, the cart.
- [ ] A human eye has looked at the preview. A clean validation is not a visual pass.
