# Author Data Shapes from Webflow — Designer Setup, 1·2·3

You can drive a live data shape from a Webflow Collection — no code, no deploy, no ticket. Create the collection once, and every published item flows into the deterministic merge as its own layer: `default → github-base → webflow-cms → kv-override`. Every read of the shape shows `webflow-cms` provenance on the fields you authored, so it is always visible that design set them.

## 1 · Create the Collection

In your Webflow site: **CMS → Create Collection**, name it **Shapes** (the slug must be `shapes` — that's how it's discovered automatically).

Add **plain-text fields** with exactly these field slugs (Webflow derives the slug from the field name — check it in field settings):

| Field name | Field slug | What it drives |
|---|---|---|
| Headline | `headline` | The banner headline |
| Message | `message` | The supporting line |
| CTA Label | `cta-label` | Button text |
| CTA Href | `cta-href` | Button link — must start with `https://`, `/`, or `#` |
| Accent | `accent` | A hex color, e.g. `#166534` |
| Updated By | `updated-by` | Your name or team — it shows in the provenance trail |

(Name and Slug exist on every collection automatically — leave them.)

## 2 · Add the item and publish

Create one item: **Name** `Promo`, **Slug** `promo`. Fill in the fields you want to set — you don't have to fill all of them; **fields you leave empty fall through to the git base**. Then **Publish the site** — the layer reads *published* items only, so Publish is your commit button.

## 3 · Verify

Within a minute (the edge caches this layer for 60 seconds), open the live shape:

```
https://<your-worker-host>/shape/promo
```

Your fields now read `"webflow-cms"` in `_provenance`, and `_meta.webflow_layer` says `"active"`. The demo surface at `/demo/shape` re-renders on its own. That's the whole loop: **edit in Webflow → Publish → every surface follows.**

## Rules worth knowing

- **A developer override still wins.** The `kv-override` layer (written through the `design:shape:write` capability) sits above yours — deliberate, so an incident override can never be undone by a site publish. When no override is set, design owns the values.
- **Invalid input falls through, never breaks.** A malformed color or an unsafe link is dropped and the layer below wins — publishing can't poison the shape.
- **Nothing here depends on memory.** The base lives in git, your layer is your published collection, overrides are ledgered. Read [Xano & Terraform, for Designers](XANO-TERRAFORM-FOR-DESIGNERS.md) for why the whole stack works this way.
