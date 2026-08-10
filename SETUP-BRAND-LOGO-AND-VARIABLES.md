# Brand Logo & Variables — Webflow in, everywhere out (1·2·3)

Two short setups. Both follow the same law: **Webflow is where you author; one address is what every platform reads.** Shopify, AEM, Next, WordPress, Magento are linkers, not owners.

## The SVG logo, 1·2·3

1. **Upload the SVG in Webflow** (Assets panel). Open the asset and copy its URL.
2. **Register it to the brand** — the system fetches it from Webflow's CDN and vaults it:
   ```
   POST https://crm-sync.dev/brand/<your-brand>/assets
   { "kind": "logo", "url": "<webflow asset url>", "alt": "Your Brand — logo", "role": "primary" }
   ```
   The ALT text rides along — the machine-readable half of the asset.
3. **Every surface reads one address:** `https://crm-sync.dev/brand/<your-brand>/logos/<file>` — public, CORS-open, cached. This is also the whole answer to Shopify's SVG ban: Shopify never hosts the SVG; its theme links this copy.

## Variable naming, 1·2·3

1. **Name Webflow variables exactly as the canonical keys below** — a "SASS Overrides" variable collection, no `uk-` prefix, no invented names. Color type for colors; Size for radius/font-size. Display-name variants normalize (`Global Primary Background` ≡ `$global-primary-background` ≡ `global_primary_background`).
2. **Ingest translates names → the record:** `POST /brand/<slug>/ingest-webflow-vars` — dry-run diff by default, `?commit=1` applies.
3. **One emitted namespace for every platform.** Link `https://crm-sync.dev/brand/<slug>/theme.css` and style with `var(--brand-…)` / `var(--global-…)`. A new platform is a new `<link>`, not a new vocabulary.

## The canonical 16

| Webflow variable name | → brand field | emitted custom property |
|---|---|---|
| `global-primary-background` | primary_color | `--global-primary-background` · `--brand-primary` |
| `global-secondary-background` | secondary_color | `--global-secondary-background` · `--brand-secondary` |
| `global-background` | background_color | `--global-background` |
| `global-muted-background` | muted_background | `--global-muted-background` |
| `global-color` | text_color | `--global-color` |
| `global-emphasis-color` | emphasis_color | `--global-emphasis-color` |
| `global-muted-color` | muted_color | `--global-muted-color` |
| `global-inverse-color` | inverse_color | `--global-inverse-color` |
| `global-border` | border_color | `--global-border` |
| `global-link-color` | link_color | `--global-link-color` |
| `global-link-hover-color` | link_hover_color | `--global-link-hover-color` |
| `global-success-background` | success_color | `--global-success-background` |
| `global-warning-background` | warning_color | `--global-warning-background` |
| `global-danger-background` | danger_color | `--global-danger-background` |
| `global-font-size` | font_size | `--global-font-size` |
| `border-rounded-border-radius` | border_radius | `--brand-radius` |

Derived set also emitted by `theme.css`: `--brand-accent`, `--brand-font-heading`, `--brand-font-body`, `--brand-heading-weight`.

## The full icon set — favicon, manifest, OG (15 slots, 7 required)

The logo is one slot of a contracted set. **The filename IS the contract** — register each with the exact name below (`POST /brand/<slug>/assets` with `file`, `role`, `theme`, `alt`), and every consumer resolves it from the manifest. A missing slot is **never blank**: it falls back to your brand logo, then to a platform monochrome default — and every slot carries `alt` + `purpose` so screen readers, answer engines, and agents can read the set.

| File (the contract) | Role | Theme | Size | Purpose | MVP |
|---|---|---|---|---|---|
| `app-any.svg` | app | any | any | Scalable app icon | ✓ |
| `app-192.png` | app | any | 192×192 | Android home screen | ✓ |
| `app-512.png` | app | any | 512×512 | Install / splash | ✓ |
| `app-maskable-512.png` | maskable | any | 512×512 | Maskable (safe zone) | ✓ |
| `apple-touch-icon-180.png` | apple-touch | any | 180×180 | iOS home screen | ✓ |
| `favicon.svg` | favicon | any | any | Scalable tab icon | ✓ |
| `favicon-32.png` | favicon | light | 32×32 | Standard tab | ✓ |
| `favicon-16.png` | favicon | light | 16×16 | Small tab | — |
| `favicon-dark-32.png` | favicon | dark | 32×32 | Dark-mode tab | — |
| `favicon.ico` | favicon | any | 16–48 | Legacy favicon | — |
| `brand-light.svg` | brand | light | any | Logo on light bg | — |
| `brand-dark.svg` | brand | dark | any | Logo on dark bg | — |
| `brand-light.png` | brand | light | 512× | Raster logo (light) | — |
| `brand-dark.png` | brand | dark | 512× | Raster logo (dark) | — |
| `brand-social.jpg` | brand | light | 1200×630 | **Social / OG card** | — |

(`brand-light.svg` / `brand-dark.svg` are MVP in the live gate — the seven-slot MVP is: the three app icons, maskable, apple-touch, favicon.svg, favicon-32, plus the two brand SVGs.)

Two endpoints read it back:
- `GET /brand/<slug>/assets/manifest.json` — the **semantic manifest**: every slot with url, alt, purpose, and `status: provisioned | fallback | default` (machine-readable — AEO, agents, a11y)
- the PWA web-app manifest consumes the same slots for install icons

**The gate:** brand review is blocked until the MVP seven are `provisioned` — icon readiness is a release check, not a vibe.

## The set, generated and provisioned (2026-08-10)

Produced from one Illustrator SVG (3.6 MB of editor metadata stripped → **3.5 KB** of pure polygons), rasterized by supersampling, registered 15/15 — **MVP READY**. Source files live in [`brand-assets/`](brand-assets/); the live set serves from the brand:

| File | Size | Bytes | Serves from |
|---|---|---|---|
| [`app-any.svg`](brand-assets/app-any.svg) | any | 3,487 B | `crm-sync.dev/brand/test-brand/icons/app-any.svg` |
| [`app-192.png`](brand-assets/app-192.png) | 192×192 | 2,062 B | `crm-sync.dev/brand/test-brand/icons/app-192.png` |
| [`app-512.png`](brand-assets/app-512.png) | 512×512 | 10,425 B | `crm-sync.dev/brand/test-brand/icons/app-512.png` |
| [`app-maskable-512.png`](brand-assets/app-maskable-512.png) | 512×512 | 9,201 B | `crm-sync.dev/brand/test-brand/icons/app-maskable-512.png` |
| [`apple-touch-icon-180.png`](brand-assets/apple-touch-icon-180.png) | 180×180 | 1,927 B | `crm-sync.dev/brand/test-brand/icons/apple-touch-icon-180.png` |
| [`favicon.svg`](brand-assets/favicon.svg) | any | 3,487 B | `crm-sync.dev/brand/test-brand/icons/favicon.svg` |
| [`favicon-32.png`](brand-assets/favicon-32.png) | 32×32 | 1,124 B | `crm-sync.dev/brand/test-brand/icons/favicon-32.png` |
| [`favicon-16.png`](brand-assets/favicon-16.png) | 16×16 | 492 B | `crm-sync.dev/brand/test-brand/icons/favicon-16.png` |
| [`favicon-dark-32.png`](brand-assets/favicon-dark-32.png) | 32×32 | 1,209 B | `crm-sync.dev/brand/test-brand/icons/favicon-dark-32.png` |
| [`favicon.ico`](brand-assets/favicon.ico) | 16/32/48 | 3,405 B | `crm-sync.dev/brand/test-brand/icons/favicon.ico` |
| [`brand-light.svg`](brand-assets/brand-light.svg) | any (1277×686) | 3,487 B | `crm-sync.dev/brand/test-brand/logos/brand-light.svg` |
| [`brand-dark.svg`](brand-assets/brand-dark.svg) | any (1277×686) | 3,487 B | `crm-sync.dev/brand/test-brand/logos/brand-dark.svg` |
| [`brand-light.png`](brand-assets/brand-light.png) | 512×275 | 15,639 B | `crm-sync.dev/brand/test-brand/logos/brand-light.png` |
| [`brand-dark.png`](brand-assets/brand-dark.png) | 512×275 | 15,506 B | `crm-sync.dev/brand/test-brand/logos/brand-dark.png` |
| [`brand-social.jpg`](brand-assets/brand-social.jpg) | 1200×630 | 29,061 B | `crm-sync.dev/brand/test-brand/logos/brand-social.jpg` |

## OG / JSON-LD image naming — the page list

Same law as the icon set: **the filename is the contract.** One OG card per page, `1200×630 JPEG`, registered to the brand (`kind=logo`, `role=og`, `theme=light`, `name=og-<handle>`, alt = the page title) and served from `crm-sync.dev/brand/test-brand/logos/og-<handle>.jpg`. The same URL feeds BOTH `og:image` and the page's JSON-LD `image` field — one asset, two machine readers. Until a page's card exists, it falls back to the brand default **`brand-social.jpg`** — never blank, same ladder as the icons.

Products use `og-product-<product-handle>.jpg` on the identical contract.

| Page handle | Title | OG image (the contract) | Serves from |
|---|---|---|---|
| `home` | Home | `og-home.jpg` | `…/logos/og-home.jpg` |
| `difference` | The Difference | `og-difference.jpg` | `…/logos/og-difference.jpg` |
| `how-it-works` | How it works | `og-how-it-works.jpg` | `…/logos/og-how-it-works.jpg` |
| `mission` | Mission | `og-mission.jpg` | `…/logos/og-mission.jpg` |
| `products` | Products | `og-products.jpg` | `…/logos/og-products.jpg` |
| `knowledge-base` | Knowledge Base | `og-knowledge-base.jpg` | `…/logos/og-knowledge-base.jpg` |
| `firmware` | Protected Firmware | `og-firmware.jpg` | `…/logos/og-firmware.jpg` |
| `channel` | Channel Publish | `og-channel.jpg` | `…/logos/og-channel.jpg` |
| `build` | Build vs Buy | `og-build.jpg` | `…/logos/og-build.jpg` |
| `pim-sync` | PIM Sync — 3D Asset Mirror | `og-pim-sync.jpg` | `…/logos/og-pim-sync.jpg` |
| `omnibus` | Price Evidence — EU Omnibus | `og-omnibus.jpg` | `…/logos/og-omnibus.jpg` |
| `shopify-deadline` | The 26 August Deadline | `og-shopify-deadline.jpg` | `…/logos/og-shopify-deadline.jpg` |
| `signed` | For the Person Who Signed | `og-signed.jpg` | `…/logos/og-signed.jpg` |
| `returns` | Returns & Review Sessions | `og-returns.jpg` | `…/logos/og-returns.jpg` |
| `attio-alternative` | Attio Alternative | `og-attio-alternative.jpg` | `…/logos/og-attio-alternative.jpg` |
| `hubspot-alternative` | HubSpot Alternative | `og-hubspot-alternative.jpg` | `…/logos/og-hubspot-alternative.jpg` |
| `klaviyo-alternative` | Klaviyo Alternative | `og-klaviyo-alternative.jpg` | `…/logos/og-klaviyo-alternative.jpg` |
| `privacy` | Privacy Policy | `og-privacy.jpg` | `…/logos/og-privacy.jpg` |
| `terms` | Terms of Service | `og-terms.jpg` | `…/logos/og-terms.jpg` |
| `refund` | Refund Policy | `og-refund.jpg` | `…/logos/og-refund.jpg` |
| `data-requests` | Your Data Rights | `og-data-requests.jpg` | `…/logos/og-data-requests.jpg` |
| `data-sharing-opt-out` | Your Privacy Choices | `og-data-sharing-opt-out.jpg` | `…/logos/og-data-sharing-opt-out.jpg` |

(22 published pages as of 2026-08-10; the `event-time` draft joins the list when it publishes.)

**One boundary to remember:** runtime `var()` consumers update everywhere within minutes of a save. Surfaces linking the **compiled** `uikit.css` are a Sass build artifact — recompile and upload after token changes. And anything hand-hardcoded never updates at all; if a surface looks frozen, audit for a missing `<link>`.
