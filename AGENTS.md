# AGENTS.md — Palmetto Studios (Vitrin/Zid Theme)

## Project

**Vitrin/Zid theme** for Palmetto Studios fashion brand.  
Template language: **Jinja** (`.jinja`).  
Design: **Windows 95 / Netscape Navigator** retro aesthetic.  
Bilingual: English + Arabic (RTL, class `.ar`).

## Reference

- `example ui/Palmetto Studios.html` — the single source of truth for the design. Every new page/component must match this visual reference.

## Architecture

| File | Role |
|---|---|
| `layout.jinja` | Base layout. Defines CSS custom properties (Win95 design system), `{% block content %}`, includes header/footer/cart. |
| `header.jinja` | Hero, icon grid, link rows, alert ribbon, clock JS. All inline `<style>` + HTML. Also defines `.win`, `.titlebar`, `.btn`, `.statusbar`. |
| `footer.jinja` | Footer with links, copyright. |
| `cart-drawer.jinja` | Full cart DOM + vanilla JS (no framework). Open/close, add/remove/qty via `[data-add]` buttons. |
| `templates/home.jinja` | Uses `{% template_components %}` — sections are managed from Zid Theme Editor. No hardcoded content. |
| `sections/win95-catalog.jinja` | **Dynamic catalog section** — renders Zid products SSR via `safeget(settings, "products.results", [])`. Has `.catalog-head`, `.product-grid`, `.card`, `.badges`, `.seal`, `.statusbar`. |
| `sections/win95-home-content.jinja` | **Static content section** — all 7 info panels (welcome, about, palm collective, size guide, shipping, returns, contact). Same as original `home.jinja` minus the catalog. |
| `templates/product.jinja` | **Empty** — needs product detail page. |
| `templates/cart.jinja` | **Empty** — needs full cart page. |
| `sections/slider.jinja` | **Empty** — placeholder for hero slider section. |

## Sections (Theme Editor managed)

All sections in `sections/` have a paired `.schema.json`. Settings accessed via `settings.id`.

- `win95-catalog` — `"type": "products"` lets merchant pick which products to display. Uses `{{ url_for('product_details', slug=product.slug) }}` for links.
- `win95-home-content` — minimal schema with just show/hide. All content hardcoded.

**Product data pattern** (from Zid):
```
product.selected_product.formatted_price
product.selected_product.formatted_sale_price
product.selected_product.media[0].image.medium
product.selected_product.in_stock
product.name, product.slug, product.short_description
```

## How catalog renders products (SSR, no JS fetch)

Merchant picks products in Theme Editor → schema `"type": "products"` → `safeget(settings, "products.results", [])` → `{% for product in ... %}` → Win95-styled cards with `[data-add]` for cart.

## Cart wiring

Cart JS in `cart-drawer.jinja` reads `.name`, `.price`, `[data-product]` from the card DOM. Products from Zid include all these fields.

## Design system

CSS custom properties in `layout.jinja:12-32` define the Win95 palette: `--chrome`, `--hi`, `--lo`, `--tb-1`, `--paper`, `--ink`, `--desktop`, etc.

Key classes defined across layout/header/footer: `.win`, `.titlebar`, `.menubar`, `.statusbar`, `.btn`, `.raise`, `.inset`, `.field`, `.icon-grid`, `.card`, `.panel`, `.seal`, `.alert-ribbon`.

All styles are **inline in `<style>` tags** within each Jinja file. No external CSS files.

## Conventions

- Every text node needs both EN and AR versions. Arabic uses class `.ar` and `dir="rtl"` where needed.
- Font family: `"Times New Roman", serif` for body copy in panels, `"Courier New", monospace` for code/metrics, `"Tahoma", sans-serif` for UI chrome, `"Amiri"` for Arabic serif.
- JS is vanilla, no framework. Inline `<script>` inside templates. Cart uses event delegation via `[data-add]` on button click.
- Schema JSON files (`*.schema.json`) define CMS settings UI for editors. Section schemas use `settings.{id}` directly.
- Use `{% extends "layout.jinja" %}` and `{% block content %}` for all page templates.
- Use `{% include '...' %}` for includes.
- Use `{{ 'path' | asset_url }}` for asset URLs (handled by Zid).
- `{% vitrin_head %}` and `{% vitrin_body %}` are CMS injection points in layout.

## Empty / WIP files to complete

- `templates/product.jinja` — product detail page
- `templates/cart.jinja` — full cart page
- `sections/slider.jinja` + `sections/slider.schema.json` — hero slider section
- `components/pagination.jinja` — reusable pagination
- `locale/ar/LC_MESSAGES/messages.po` — empty translations file

## No build / test tooling

No `package.json`, no npm, no bundler, no linter, no test framework.  
Theme is deployed directly through Zid/Vitrin. No local dev server.
