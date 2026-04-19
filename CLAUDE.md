# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Shopify Liquid theme for the FFUJJI e-commerce store, built on the Baseline theme (v2.5.0) by Switch Themes. There is no build step or local dev server — all rendering happens on Shopify's platform.

## Shopify CLI Commands

```bash
shopify theme push        # Deploy changes to the connected Shopify store
shopify theme pull        # Pull the latest settings from Shopify
shopify theme dev         # Start a local development preview (proxies to Shopify)
shopify theme check       # Lint Liquid files
```

## Architecture

**Template → Section → Snippet hierarchy:**
- `templates/*.json` — Page builders. Define which sections appear on each page and their order.
- `sections/*.liquid` — Self-contained feature components, each with their own `{% schema %}` block defining admin-configurable settings and blocks.
- `snippets/*.liquid` — Reusable partials rendered via `{% render 'snippet-name', param: value %}`.
- `layout/theme.liquid` — Master HTML shell; includes GTM (GTM-PX8PVHSZ) and Hotjar (5023235).

**Styling:**
- `assets/theme.css` — Main stylesheet. Uses CSS custom properties generated at runtime by `snippets/css-variables.liquid` from theme settings.
- CSS variable naming: `--scheme-background`, `--scheme-text`, `--scheme-accent` — bound to the active color scheme (primary/secondary/tertiary).
- Custom utility classes mirror Tailwind (e.g., `grid-cols-1`, `lg:grid-cols-12`, `py-2`, `bg-scheme-background`).
- Borders are used as layout dividers (`border-grid-color`, `border-b-grid`).

**JavaScript:**
- `assets/theme.js` — Webpack bundle. Do not edit directly; it's compiled output.
- `assets/custom.min.js` — Custom JS for FFUJJI-specific behavior.
- Alpine.js for component reactivity (`x-data`, `x-init` attributes in sections).
- Global config at `window.theme` (strings, routes, cart state, money format).
- Components use unique Alpine namespaces: `ThemeSection_{{ section.id | handleize | replace: '-', '_' }}()`.

**Data flow:**
- All product/collection data is server-rendered via Liquid — no client-side API calls for initial page load.
- Cart mutations use AJAX against Shopify Cart API routes (`routes.cart_add_url`, `routes.cart_change_url`).

**Settings:**
- `config/settings_schema.json` — Defines all merchant-configurable theme options (colors, fonts, spacing, buttons).
- `config/settings_data.json` — Auto-generated active values. Do not edit manually; Shopify admin overwrites this file.
- Current brand colors: text `#030303`, background `#ffffff`, accent `#9b2e33`.

## Key Files to Know

| File | Purpose |
|------|---------|
| `snippets/css-variables.liquid` | Generates all CSS custom properties from settings |
| `snippets/product-form.liquid` | Variant selector + add-to-cart form |
| `snippets/cart-drawer.liquid` | Slide-out cart |
| `snippets/product-grid-item.liquid` | Product card used in all collection grids |
| `snippets/responsive-image.liquid` | Lazy-loaded image helper — use this for all images |
| `sections/header.liquid` | Navigation |
| `sections/main-product.liquid` | Product detail page |
| `sections/main-collection.liquid` | Collection listing page |

## Section Schema Pattern

Every section has a `{% schema %}` block at the bottom. Settings defined there appear in the Shopify theme editor. Blocks within a schema allow merchants to add/remove/reorder sub-components (e.g., product page tabs, feature blocks).

## Localization

Strings live in `locales/en.default.json` (primary). Reference them in Liquid via `{{ 'key' | t }}`. Eight locales supported (en, da, de, es, fr, nl, pt-BR, pt-PT).

## Custom Page Templates

Seasonal and brand-specific pages use custom template JSON files (e.g., `page.fw24-lookbook.json`, `page.ss25-lookbook.json`, `page.ffujjixforeign.json`). These map to a matching section for that layout.
