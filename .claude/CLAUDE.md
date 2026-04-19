# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This repository contains customizations and documentation for the **Thor Sequence RV**. It is published via GitHub Pages at `https://hub.dockter.com/sequence` — all generated HTML lives in `docs/`.

## Directory Structure

```text
site/                        ← Edit here
  _templates/                ← HTML layout templates (page.html.tmpl, card.html.tmpl)
  _assets/                   ← Shared CSS and static files
  index.md                   ← Home page intro text
  customizations/
    <slug>/
      meta.yaml              ← Metadata (title, date, summary, status, tags)
      content.md             ← Full write-up in Markdown
      images/                ← Photos for this customization

docs/                        ← GENERATED — never edit by hand
```

## Generating the Website

Run the skill:

```text
/generate-website
```

No toolchain required — Claude Code reads source files and writes `docs/`. Run it after any change to `site/`.

## Adding a New Customization

1. Create `site/customizations/<slug>/` (use a short, lowercase, hyphenated name)
2. Add `meta.yaml` with the fields below
3. Write `content.md` in Markdown
4. Drop photos in `images/` (optional)
5. Run `/generate-website`

### meta.yaml fields

```yaml
title: "Solar Panel Installation"
date: "2026-04-18"
summary: "Added 400W of roof solar and a 100Ah lithium battery."
status: complete        # draft | complete  ("draft" hides it from the site)
tags: [electrical, solar]
```

## Publishing

After running `/generate-website`, commit the `docs/` changes and push. GitHub Pages auto-deploys.

```bash
git add docs/
git commit -m "feat: regenerate site"
git push
```

## Repository Conventions

- Branch naming: `<issue-number>-<github-username>-<monotonically-increasing-integer>` (e.g., `1-docktermj-1`)
- Commits use Conventional Commits format: `<type>(<scope>): <summary>` with `Co-Authored-By` trailer
- Asset URLs in templates use the absolute base path `/sequence/` (required for GitHub Pages subfolder deployment)
