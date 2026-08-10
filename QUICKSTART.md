# Quickstart — the shape demo for working devs

No theory. Six copy-paste blocks, what you'll see after each, one line of why. If you can edit a Liquid theme, you can run this. (The deeper version with all the reasoning: [DEMO-SHAPE-5MIN.md](DEMO-SHAPE-5MIN.md). The no-code version: the Designer Guide on the Miro board / [SETUP-WEBFLOW-SHAPES.md](SETUP-WEBFLOW-SHAPES.md).)

Three words used below, defined once:
- **shape** — a small JSON object (headline, message, color) that pages render from.
- **cap** — a permission flag on your account. `design:shape:write` is the one that lets you change shapes.
- **provenance** — per field, which layer supplied the value: the git file, the Webflow collection, or a saved override.

Setup (once):

```sh
BASE="https://<worker-host>"        # ask for the host, or read it off /stack/config
TOKEN="<your token>"                # admin key, tenant token, or your login JWT
```

## 1 · Read the shape

```sh
curl -s "$BASE/shape/promo"
```

You'll see the JSON: a `shape` object, a `_provenance` map (every field: `"github-base"`), and `_meta` telling you the merge order. **Why:** this is the whole model — pages render this, nothing else.

## 2 · Try to write without permission

```sh
curl -s -X POST "$BASE/shape/promo" -H 'Content-Type: application/json' -d '{"headline":"test"}'
```

You'll see `403` with `"cap": "design:shape:write"` and a purchase URL. **Why:** writes need the cap; the error names it and tells you how to get it. No silent failures.

## 3 · Preview a change (nothing saves)

```sh
curl -s -X POST "$BASE/shape/promo" -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' -d '{"headline":"Hello from the quickstart","accent":"#166534"}'
```

You'll see `"dry_run": true` and a `diff` array — each field: from, to, which layer it currently comes from. **Why:** default is preview; you can't fat-finger production.

## 4 · Commit it

```sh
curl -s -X POST "$BASE/shape/promo?commit=1" -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' -d '{"headline":"Hello from the quickstart","accent":"#166534"}'
```

You'll see `"ok": true` and a `ledger_hash`. **Why:** the save returns its own receipt. Now open `$BASE/demo/shape` in a browser — the banner shows your text. Re-run step 1: your fields now read `"kv-override"`.

## 5 · Check the record

```sh
curl -s "$BASE/shape/promo/ledger"
```

You'll see every commit and reset in order with `"chain_intact": true`. **Why:** the history is verifiable, not just logged.

## 6 · Reset

```sh
curl -s -X DELETE "$BASE/shape/promo" -H "Authorization: Bearer $TOKEN"
```

Back to the git base — and the reset itself lands in the ledger. Run step 1 to confirm.

---

That's the entire system: **read → refused → preview → commit-with-receipt → verify → reset.** Everything else in this repo is the same six moves with more words.

Two things that will save you a support thread:
- Invalid fields (`"accent":"red"`, a `javascript:` href) are **dropped silently by design** — the layer below wins. If your change "didn't take," check the field format, not the server.
- The git base caches for ~60s and the Webflow layer for 60s — a change to *those* layers takes up to a minute. Your KV override (step 4) is instant.
