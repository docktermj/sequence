---
name: generate-website
description: Generate the GitHub Pages site in docs/ from source files in site/. Run this after adding or editing any content in site/.
user-invocable: true
allowed-tools: Read, Write, Glob, Bash
---

Generate the static website for the Thor Sequence RV repository at the current working directory. The source is in `site/`, the output goes to `docs/`. Follow these steps exactly.

## Step 1 — Load templates

Read both template files into memory:
- `site/_templates/page.html.tmpl`
- `site/_templates/card.html.tmpl`

## Step 2 — Collect customizations

Use Glob to find all files matching `site/customizations/*/meta.yaml`. For each:
- Read the YAML fields: `title`, `date`, `summary`, `status`, `tags`
- Read the corresponding `content.md`
- Derive the slug from the directory name (the `*` portion of the path)
- **Skip any entry where `status` is `draft`**
- Find the first image file (if any) in `site/customizations/<slug>/images/` — this is the thumbnail. Note the filename.

Sort the collected customizations by `date` descending (newest first).

## Step 3 — Clean and recreate docs/

Run this command to wipe the generated output and recreate the directory skeleton:

```bash
rm -rf docs && mkdir -p docs/assets docs/customizations
```

## Step 4 — Copy assets

Read each file in `site/_assets/` and write it to `docs/assets/<filename>`.

For each customization that has an `images/` directory (`site/customizations/<slug>/images/`), copy every file in it to `docs/customizations/<slug>/images/<filename>` using Bash. Create the target directory first if needed.

## Step 5 — Generate customization pages

For each published customization:

1. Convert `content.md` to semantic HTML. Translate Markdown directly:
   - `# Heading` → `<h1>`, `## Heading` → `<h2>`, etc.
   - `**text**` → `<strong>text</strong>`, `*text*` → `<em>text</em>`
   - Paragraphs separated by blank lines → `<p>...</p>`
   - `![alt](images/file.jpg)` → `<img src="images/file.jpg" alt="alt">`
   - Bulleted lists → `<ul><li>...</li></ul>`, numbered lists → `<ol>`
   - Inline code `` `text` `` → `<code>text</code>`
   - Plain URLs in list items → `<a href="URL">URL</a>`

2. Build the NAV breadcrumb: `<a href="../../index.html">Home</a> › <span>TITLE</span>` (always include `index.html` so file:// browsing works)

3. Substitute into `page.html.tmpl`:
   - `{{PAGE_TITLE}}` → the customization's `title`
   - `{{CONTENT}}` → the converted HTML (omit the leading `<h1>` if content.md starts with one — the template already renders it)
   - `{{NAV}}` → the breadcrumb HTML
   - `{{HERO}}` → empty string (detail pages have no hero)
   - `{{ROOT}}` → `../../` (directory prefix to docs root; the template appends `index.html` and `assets/...` where needed)

4. Write the result to `docs/customizations/<slug>/index.html`.

## Step 6 — Generate the index page

1. Read `site/index.md`. Convert the intro paragraphs to HTML (same Markdown rules as Step 5). This becomes `{{INTRO}}`.

2. For each published customization, render a card by substituting into `card.html.tmpl`:
   - `{{SLUG}}` → the slug
   - `{{TITLE}}` → the title
   - `{{DATE}}` → the raw date string (e.g., `2026-04-18`)
   - `{{DATE_FORMATTED}}` → human-readable date (e.g., `April 18, 2026`)
   - `{{SUMMARY}}` → the summary
   - `{{TAGS}}` → for each tag, render `<span class="tag">TAG</span>`, joined together; if no tags, empty string
   - `{{THUMBNAIL}}` → if the customization has a first image, render `<img class="card-thumb" src="customizations/SLUG/images/FILENAME" alt="TITLE">` (relative path; thumbnails point at images, not pages, so no `index.html` needed), otherwise empty string

3. Build the index page HTML using `page.html.tmpl`:
   - `{{PAGE_TITLE}}` → `Thor Sequence RV`
   - `{{NAV}}` → empty string
   - `{{HERO}}` → `<section class="hero"><p class="hero-tagline">A Class B+ motorhome, modified and documented.</p></section>`
   - `{{ROOT}}` → `./` (index is at the root of docs/)
   - `{{CONTENT}}` → `{{INTRO}}\n<div class="cards">\n` + all cards joined + `\n</div>`

4. Write to `docs/index.html`.

## Step 7 — Report

Run `find docs -type f | sort` and report the list of generated files and the total count. Confirm the site is ready to publish.
