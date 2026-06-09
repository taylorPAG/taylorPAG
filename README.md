# Hey, I'm Taylor Jones

Full-stack web developer building production e-commerce systems where the cost of getting it wrong is real customer orders and real money. Currently leading the web platform at Portable Air Group, a national industrial HVAC equipment supplier. I built the entire site from scratch over four months: 1000+ products, government procurement integration, an Airtable-synced used-equipment pipeline, and a custom deployment system designed so production can't be silently broken. Much of the tooling described in this README came out of real production incidents — what failed, what I changed, what's now site convention.

## What I'm Working On

**Lead Developer — Portable Air Group (Feb 2026 – present)**

Built the entire web platform from scratch for a national industrial HVAC equipment supplier. The site is a full product catalog with 1000+ SKUs, government procurement integration, and a used equipment pipeline synced from Airtable. Each product is complicated and a project of this scale could only be completed cost-effectively with the aid of AI — particularly Claude Code, which I've learned to treat as a real collaborator that needs real guardrails.

## Tech Stack

- **CMS:** WordPress + WooCommerce (custom theme, no page builders)
- **Data Layer:** ACF Pro (PHP-registered fields), FacetWP (faceted search)
- **Frontend:** Custom CSS design system, vanilla JS, responsive down to mobile
- **Automation:** n8n workflows, Airtable sync pipeline, WP REST API endpoints
- **Tooling:** WP-CLI, Python image pipeline, bash/PHP import scripts
- **Infrastructure:** Cloudflare, DreamPress, LocalWP dev environment

## What I Built

- **Custom renderer architecture** — hook-based PHP system that conditionally loads page-specific renderers, CSS, and JS. Zero page builder dependencies.
- **Product import pipeline** — self-improving system: bash scripts scrape manufacturer sites, extract specs to TSV, audit against ACF fields, and generate WP-CLI import scripts. Spec-name mapping dictionary grows with each brand.
- **Used equipment sync** — Airtable-to-WordPress pipeline. Used units link to master products, inherit specs/accessories, and flow through an available/pending/sold lifecycle synced with WooCommerce orders.
- **Variable product configurator** — FacetWP-styled filters with per-variation metadata (country of origin, amps, weight, CFM) driving badge logic and complex spec display.
- **Glass design system** — 4-tier Apple Liquid Glass-inspired CSS token system with spectral rim, shine, and tint effects. Backdrop-filter blur + layered backgrounds + mask-composite gradient borders.
- **Government procurement pages** — GSA Schedule, TAA compliance, and NAICS code pages with structured data (JSON-LD) for federal buyer discovery.
- **Image pipeline** — 6-stage Python pipeline: collect, convert to WebP, export WP data, match products, rename, bulk upload. Handles 1200×1200 standardization with Pillow.
- **SEO & security** — JSON-LD schema for products/org/FAQ, email/phone obfuscation system, CSP headers, cookie consent, robots optimization.
- **Archive system** — unified renderer for brand, category, and shop pages with knowledge tabs, spec range pills, brand logos, faceted filtering, and equipment-first sort.

## DevOps & Deployment Infrastructure

WordPress sites typically suffer from one of two failure modes: either nothing is environment-disciplined and changes get re-done by hand (and inconsistently) on each environment, or the database gets pushed wholesale to production and clobbers live customer orders. I built this pipeline to reject both. Everything below came out of a real pain point — most from a real incident.

- **Three-rail deployment pipeline.** WordPress has three fundamentally different kinds of state — code (PHP/CSS/JS), database content (settings, ACF values, taxonomy), and media (uploads). They have different lifecycles, ownership, and rollback semantics. Most WP DevOps tools treat them as one rail and either push everything or sync nothing. I separated them: code moves via git + `deploy-code`, content moves via a recipe journal + `replay-content`, media via incremental rsync. Production is push-when-ready by design — never auto-synced. The result: a content change can ship without touching code, a code change can ship without touching content, and live orders are never at risk from a routine deploy.

- **The recipe rail.** Doing the same content change three times (local → staging → prod) by hand creates drift you can't detect until something breaks. Every DB content change is authored *once* as an idempotent PHP recipe that resolves objects by business key (SKU, slug, title — never numeric IDs, which differ per environment). A single `replay-content` command snapshots prod, enforces staging-first ordering, and journals applied-state per environment. One artifact, three runs — the change either happened identically everywhere or it failed loudly. The "do it three times and pray" workflow is gone.

- **Image-asset propagation rail.** First time I tried to deploy a hero-image swap, I learned the hard way that WordPress attachment IDs are per-environment — the same image uploaded on three sites gets three different IDs. A recipe that says *"set page hero = 6558"* works locally and silently sets a wrong image (or none) elsewhere. Solved by treating image bytes as code: image files for managed assets live under the theme, ride the code rail via git, and the recipe calls `pag_rx_sideload_from_theme($path)` to md5-resolve to whatever the env-local attachment ID happens to be. One source of truth (the theme file), three env-local IDs, no hardcoding.

- **Hard-fail floor for replayable operations.** A recipe I shipped (the first image-propagation one) ran cleanly on staging and prod, got marked as applied in the journal, and did *nothing* — the bytes hadn't been deployed via `deploy-code` yet, so the recipe's "skip if source missing" branch fired and silently returned. We thought it was live for four days. The fix: any precondition a recipe can't satisfy must throw `WP_CLI::error()` so the journal can't be lied to. The principle generalized: *a soft-return in a replayable operation is a silent lie.* That rule is now site convention, and that incident is the reason the next bullet exists.

- **Drift inspection (`verify-images`).** After the silent-no-op above, there was no automated way to confirm dozens of pictures actually landed in production. Trust-but-verify needed a verifier. Read-only command that walks every image reference on a site (ACF root + repeater/group sub-fields + WooCommerce featured + WC gallery), captures attachment file md5/size, and diffs local against staging or prod. Five outcome buckets per record: `OK`, `MISSING_ON_{env}`, `MISSING_ON_LOCAL`, `FILE_MISSING`, `MD5_DRIFT`. Exits non-zero on mismatch so it slots into scripts as a gate. First run against staging surfaced 53 pre-existing drift records nobody had caught.

- **Production guardrails (38-rule deny-list).** AI coding agents — and humans tired at 2 AM — can run destructive commands by accident. The only environment with real customer data, real Stripe transactions, and real legal exposure can't depend on either of them being careful. The deny-list hard-blocks destructive operation classes (eval, db drop/import, search-replace, config writes, payment-option changes, content/user deletion) at the WP-CLI wrapper, not in policy. Staging stays an unrestricted sandbox; production stays sacred. Snapshots happen before any mutation; staging-first is enforced; no command bypasses the guardrail.

- **AI-collaboration hooks (PreToolUse).** Process discipline degrades under context pressure: the same rule that a coding agent (Claude Code) followed in message 5 gets forgotten by message 50. Documentation in a CLAUDE.md doesn't fix it — context drift means the doc gets summarized and lost. Solved by moving rule enforcement out of the conversation and into the harness: PreToolUse hooks intercept the agent's commands before they run and inject context-aware reminders the moment the agent is about to violate a convention (e.g. mutating the local DB without authoring a recipe, referencing an image attachment without using the helper). Same shape, narrowly-scoped per drift class, fires from outside the conversation. The rails hold even under heavy automation.

- **Airtable → WordPress sync.** The original sync ran on n8n and became unreliable — silent failures, sync lag, ambiguous state. Replaced it with a self-locking PHP poller (Windows Scheduled Task → curl → REST endpoint → upsert) that hash-detects changed records, sideloads photos with EXIF rotation applied before WebP conversion, and writes sync status back to Airtable on every operation. Steady-state ~5s per poll, zero changes — and now the sync state is observable, debuggable, and stops cleanly when something goes wrong. Airtable stays authoritative for used-inventory; WordPress is a read-only mirror of that data only.

- **Dynamic pricing engine.** Weeks-of-supply × condition pricing tiers for used inventory with floor-price enforcement, cooldowns, and an append-only pricing audit log. Inventory decisions are reproducible and auditable, not "what did we charge last Tuesday."

- **Reproducible toolchain.** Single-machine bus factor is one stolen laptop away from disaster. The entire pipeline (SSH aliases, WP-CLI config, deploy scripts, the deny-list, all reminder hooks) is versioned in a private `pag-ops` repo with a one-command installer. New machines bootstrap in under a minute. Tooling is backup-by-default; nothing important lives only on one workstation.

## Velocity with a Safety Net

The point of all the infrastructure above isn't engineering elegance — it's that we ship to production *every day*, sometimes multiple times on busy days, without giving up the local → staging → prod discipline. Typical "safe" pipelines are slow: CI queues, deploy windows, manual QA cycles measured in hours or days. Typical "fast" pipelines are reckless: yolo to prod, hope, fix forward.

This one is both. A routine content change goes from local idea to live in about ten minutes — recipe authored, replayed to staging, verified by the drift checker, replayed to prod with a snapshot taken first. Speed and safety here reinforce each other instead of trading off: the recipe rail is *faster* than redoing the work three times AND *safer* than diff-detection database sync; the verifier replaces a manual QA cycle that would itself slow things down; the hooks keep the conventions honest without anyone having to remember them at 2 AM.

That's how a solo developer ships at small-team velocity — not by cutting corners, but by removing the cost of the corners.

## By the Numbers

- 400+ commits over 4 months
- 1000+ products across 12 categories and 20+ brands
- ~15,000 lines of custom PHP, CSS, and JS
- 6 page-specific renderers, 30+ ACF field groups, 19 page templates
- 17 idempotent DB-content recipes (replayed identically across local, staging, prod)
- 1,880+ image references tracked and drift-checkable in one command
- 38-rule production deny-list (zero incidents from accidental command execution)
- ~10 minute cycle time from local idea to live in production (routine content change)

## Get in Touch

taylorwjones31@gmail.com
