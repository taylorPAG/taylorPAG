# Hey, I'm Taylor Jones

Full-stack web developer building production e-commerce and industrial catalog systems.

## What I'm Working On

**Lead Developer — Portable Air Group** (Feb 2026 – present)

Built the entire web platform from scratch for a national industrial HVAC equipment supplier. The site is a full product catalog with 1000+ SKUs, government procurement integration, and a used equipment pipeline synced from Airtable. Each product is complicated and a project of this scale could only be completed cost effectively with the aid of AI and especially Claude code. 

### Tech Stack
- **CMS:** WordPress + WooCommerce (custom theme, no page builders)
- **Data Layer:** ACF Pro (PHP-registered fields), FacetWP (faceted search)
- **Frontend:** Custom CSS design system, vanilla JS, responsive down to mobile
- **Automation:** n8n workflows, Airtable sync pipeline, WP REST API endpoints
- **Tooling:** WP-CLI, Python image pipeline, bash/PHP import scripts
- **Infrastructure:** Cloudflare, DreamPress, LocalWP dev environment

### What I Built
- **Custom renderer architecture** — hook-based PHP system that conditionally loads page-specific renderers, CSS, and JS. Zero page builder dependencies.
- **Product import pipeline** — self-improving system: bash scripts scrape manufacturer sites, extract specs to TSV, audit against ACF fields, and generate WP-CLI import scripts. Spec-name mapping dictionary grows with each brand.
- **Used equipment sync** — Airtable-to-WordPress pipeline via n8n. Used units link to master products, inherit specs/accessories, and flow through an available/pending/sold lifecycle synced with WooCommerce orders.
- **Variable product configurator** — FacetWP-styled filters with per-variation metadata (country of origin, amps, weight, CFM) driving badge logic and complex spec display.
- **Glass design system** — 4-tier Apple Liquid Glass-inspired CSS token system with spectral rim, shine, and tint effects. Backdrop-filter blur + layered backgrounds + mask-composite gradient borders.
- **Government procurement pages** — GSA Schedule, TAA compliance, and NAICS code pages with structured data (JSON-LD) for federal buyer discovery.
- **Image pipeline** — 6-stage Python pipeline: collect, convert to WebP, export WP data, match products, rename, bulk upload. Handles 1200x1200 standardization with Pillow.
- **SEO & security** — JSON-LD schema for products/org/FAQ, email/phone obfuscation system, CSP headers, cookie consent, robots optimization.
- **Archive system** — unified renderer for brand, category, and shop pages with knowledge tabs, spec range pills, brand logos, faceted filtering, and equipment-first sort.

### By the Numbers
- 400+ commits over 3 months
- 1000+ products across 12 categories and 20+ brands
- ~15,000 lines of custom PHP, CSS, and JS
- 6 page-specific renderers, 30+ ACF field groups, 19 page templates

## Get in Touch
- taylorwjones31@gmail.com
