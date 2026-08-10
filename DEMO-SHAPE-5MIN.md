# The 5-Minute Demo — Deterministic Data Shape

**What it proves:** the Design/Dev handoff moves from *editing content* to *exchanging data models*. The shape is versioned in git, merged deterministically at the edge with per-field provenance, written only under a capability, rendered live on an edge-served surface, and every commit is hash-chained.

**Planes exercised:** GitHub (base model in a public spec repo) · Cloudflare (merge + gate + ledger at the edge) · commerce (a purchase mints the write capability — purchase = permission).

Set once:

```sh
BASE="https://<your-worker-host>"     # the edge worker serving the /shape plane
KEY="<your admin, designer, or tenant token>"
```

## The sequence

**1 · Discovery** — the stack advertises the shape plane:
```sh
curl -s "$BASE/stack/config" | python3 -c "import json,sys; print(json.load(sys.stdin)['shape'])"
```

**2 · Read the base** — every field's provenance is `github-base`; the model file is a plain JSON committed to the public spec repo:
```sh
curl -s "$BASE/shape/promo" | python3 -m json.tool
```

**3 · Roles gate the write** — no token → 403 that names the cap AND the purchase that mints it:
```sh
curl -s -X POST "$BASE/shape/promo" -H 'Content-Type: application/json' -d '{"headline":"nope"}'
# → {"error":"…needs the design:shape:write capability","cap":"design:shape:write","purchase":"…"}
```
Buying the named product grants `caps.design:shape:write` through the SKU-grant rail — **purchase = permission**. Designer entitlements and `design`/`content_edit` team role-tags also hold the cap.

**4 · Dry-run diff** — authorized, but nothing written (non-destructive by default):
```sh
curl -s -X POST "$BASE/shape/promo" -H "Authorization: Bearer $KEY" -H 'Content-Type: application/json' \
  -d '{"headline":"Designers ship shapes now.","accent":"#166534","updated_by":"demo:designer"}' | python3 -m json.tool
```

**5 · Commit** — override written, ledger hash returned:
```sh
curl -s -X POST "$BASE/shape/promo?commit=1" -H "Authorization: Bearer $KEY" -H 'Content-Type: application/json' \
  -d '{"headline":"Designers ship shapes now.","accent":"#166534","updated_by":"demo:designer"}'
```

**6 · Re-read** — the changed fields now say `kv-override`; untouched fields still say `github-base`. Determinism made visible:
```sh
curl -s "$BASE/shape/promo" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['_provenance'])"
```

**7 · The surface follows the data** — open **`$BASE/demo/shape`** in a browser: the banner re-renders from the model (no page was edited; 10s auto-poll or the Refresh button).

**8 · Verify the chain:**
```sh
curl -s "$BASE/shape/promo/ledger" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['count'], d['chain_intact'])"
```

**9 · The closer — git flows to the edge with no deploy.** Edit the model JSON in the GitHub UI, commit, and re-run step 2: the `github-base` fields update within ~1–2 minutes (raw CDN upstream, 60s edge cache). *Pre-warm this once before presenting; upstream can take up to ~5 min.*

**10 · Reset** (repeatable — the reset itself is ledgered):
```sh
curl -s -X DELETE "$BASE/shape/promo" -H "Authorization: Bearer $KEY"
```

## Guardrails worth pointing at mid-demo

- Invalid fields are **dropped, not erred** — `{"accent":"red","cta_href":"javascript:…"}` → 400 `no valid fields`; a bad layer can never poison the merge, the next layer down simply wins.
- Read is public + `Cache-Control: no-store`; the write needs `design:shape:write`; the ledger read is public (`chain_intact` is recomputed per request).
- If GitHub is unreachable the worker falls back to a bundled base copy (`base_layer:"assets-fallback"` makes that visible too).
- A published Webflow "Shapes" Collection joins the merge as its own layer (`webflow-cms`, between the git base and the KV override) — see [SETUP-WEBFLOW-SHAPES.md](SETUP-WEBFLOW-SHAPES.md); `_meta.merge_order` states the full ladder on every read.

## The point, in one line

The discipline already applied to firmware and licenses — *permission as data, provenance on every read, a ledger under every write* — applied to the design plane: **shapes, not page edits.**
