# AGENTS.md — Palmetto Studios (Vitrin/Zid Theme)

## Project

**Vitrin/Zid theme** for Palmetto Studios fashion brand.  
Template language: **Jinja** (`.jinja`).  
Design: **Windows 95 / Netscape Navigator** retro aesthetic.  
Bilingual: English + Arabic (RTL, class `.ar`).  
No build tooling. No npm, bundler, linter, or tests. Deployed directly to Zid/Vitrin.

## Reference

- `example ui/Palmetto Studios v2.html` — single source of truth for the design. Match this.
- `example ui/Palmetto Studios v1.html` — earlier design revision.

## Architecture

| File | Role |
|---|---|
| `layout.jinja` | Base layout. CSS custom properties (Win95 palette), `{% block content %}`, `window.i18n` JS strings, includes header/footer/cart components. |
| `header.jinja` | Hero, icon grid, link rows, alert ribbon, clock JS. Also defines `.raise`, `.inset`, `.field`, `.btn`, `.win`, `.titlebar`, `.statusbar`. |
| `footer.jinja` | Footer with links, copyright. |
| `components/cart-sidebar.jinja` | Cart drawer DOM + CSS (styled Win95 window). Overlay, drawer, item layout, totals, checkout link. |
| `components/cart-script.jinja` | Cart JS (vanilla). `loadCart()`, `addToCart()`, `updateCartItem()`, `removeCartItem()`, `openCart()`, `closeCart()`. Global functions — callable from inline `onclick`. Event delegation via `[data-add]` and `[data-add-variant]`. |
| `components/product-card.jinja` | Shared product card component (image, name, price, desc, Add/Details buttons). Used by `templates/products.jinja` and `sections/win95-catalog.jinja`. |
| `components/pagination.jinja` | Simple prev/next pagination via `?page=X`. Expects `products` object with `page`, `pages_count`, `count`. |
| `templates/home.jinja` | `{% template_components %}` — sections managed from Zid Theme Editor. |
| `templates/product.jinja` | **Product detail page** — fully implemented. Gallery, variants, bundles, qty selector, add-to-cart, related products. |
| `templates/products.jinja` | **Product listing page** — grid of product cards + pagination. Iterates `products.results`. |
| `templates/cart.jinja` | **Empty** — full cart page not yet implemented. |
| `sections/win95-catalog.jinja` | Dynamic catalog section — SSR products via `safeget(settings, "products.results", [])`. |
| `sections/win95-home-content.jinja` | Static content section — 7 info panels (welcome, about, size guide, etc.). |
| `sections/slider.jinja` + `slider.schema.json` | **Empty** — placeholder for hero slider section. |

## Two cart pathways (critical)

| Context | Mechanism | Notes |
|---|---|---|
| Listing pages (catalog, products grid) | Global `addToCart(pid, qty, btn)` — `POST /api/v1/cart/items` | Simple product → direct API call. Works for products **without** variants or bundles. |
| Product detail page | `window.zid.cart.addProduct(options, { showErrorNotification: true })` — Zid SDK | Uses form `#product-form`. Required for variants, bundles. Callback: `window.productOptionsChanged()`. |

Do **not** use `addToCart()` on product detail pages — it bypasses variant selection and bundle validation.

## Cart API

All cart functions in `cart-script.jinja` are **global**:

| Function | Method | Endpoint |
|---|---|---|
| `loadCart()` | GET | `/api/v1/cart` |
| `addToCart(pid, qty, btn)` | POST | `/api/v1/cart/items` `{ product_id, quantity }` |
| `updateCartItem(id, qty, btn)` | PATCH | `/api/v1/cart/items/{id}` (form-urlencoded) |
| `removeCartItem(id, btn)` | DELETE | `/api/v1/cart/items/{id}` |

Cart drawer opens on add; badge count updates from API response. Cart close triggers: Escape key, overlay click, ✕ button, "Continue Shopping".

## Product data pattern (from Zid)

```
product.selected_product.formatted_price
product.selected_product.formatted_sale_price
product.selected_product.media[0].image.full_size (or .medium)
product.selected_product.in_stock
product.selected_product.sku
product.name, product.slug, product.id, product.short_description
product.has_variants / product.structure == 'parent'
product.product_class == 'dynamic_bundle' (bundles)
product.related_products[] (on product detail page)
product.categories[0].name
product.badge.body.en (promo badge)
product.rating.average, product.rating.total_count
product.discount_percentage
product.quantity (stock level; 0 = not infinite stock)
```

## Product listing data (for `templates/products.jinja`)

```jinja
products.results[] — array of product objects
products.count — total products
products.page — current page number
products.pages_count — total pages
```

## Design system

CSS custom properties in `layout.jinja:12-32`: `--chrome`, `--hi`, `--hi-2`, `--lo`, `--lo-2`, `--tb-1`, `--tb-2`, `--paper`, `--ink`, `--desktop`, `--warn-yellow`, `--red`, `--green`.

Key classes: `.win`, `.titlebar`, `.menubar`, `.statusbar`, `.btn`, `.raise`, `.inset`, `.field`, `.icon-grid`, `.card`, `.panel`, `.seal`, `.alert-ribbon`.

All styles are **inline in `<style>` tags** within each Jinja file. No external CSS files (except Google Fonts CDN for Amiri).

Font families: `"Times New Roman", serif` for body copy, `"Courier New", monospace` for code/metrics, `"Tahoma", sans-serif` for UI chrome, `"Amiri"` for Arabic.

## Conventions

- Every text node needs both EN and AR. Arabic uses class `.ar` and `dir="rtl"`.
- Arabic font size is larger (18px default, 15px for small).
- JS is **vanilla**, no framework. Inline `<script>` in templates.
- Schema JSON files (`*.schema.json`) define CMS settings UI. Section settings accessed via `settings.id`.
- All page templates use `{% extends "layout.jinja" %}` and `{% block content %}`.
- Use `{% include '...' %}` for includes; `{{ 'path' | asset_url }}` for asset URLs.
- `{% vitrin_head %}` and `{% vitrin_body %}` are CMS injection points.
- `_('text')` is the server-side i18n function.
- `window.i18n.{key}` stores JS translation strings (defined in `layout.jinja`).
- `components/` directory contains reusable partials (product-card, pagination, cart). Include them with path relative to theme root.
- `main.js` is referenced in layout but loaded from Zid's CDN — it does **not** exist in this repo.
- `locale/ar/LC_MESSAGES/messages.po` is empty — Arabic translations are inline in templates.
