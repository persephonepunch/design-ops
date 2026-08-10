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

**One boundary to remember:** runtime `var()` consumers update everywhere within minutes of a save. Surfaces linking the **compiled** `uikit.css` are a Sass build artifact — recompile and upload after token changes. And anything hand-hardcoded never updates at all; if a surface looks frozen, audit for a missing `<link>`.
