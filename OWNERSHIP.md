# Ownership Boundaries — Xano ↔ Cloudflare

One page. Read this before you touch anything. It exists so AI-authored changes and
human-owned infrastructure never fight over the same resource.

## The one drift rule

> **Every resource has exactly one source of truth. If a resource is Terraform-owned,
> you change it by editing HCL and letting CI `apply` — never in a dashboard, never via
> MCP/AI, never with `wrangler` by hand. If it's Wrangler-owned, you change it in
> `workers/` and CI deploys it. Never manage the same resource from two places.**

Breaking this rule is what causes `terraform plan` to show phantom diffs and revert
someone's out-of-band change on the next apply.

## Who owns what

| Resource                                   | Owned by            | You change it by                     |
|--------------------------------------------|---------------------|--------------------------------------|
| KV namespace (edge read cache)             | **Terraform**       | edit `terraform/main.tf` → CI apply  |
| Cloudflare Queue (write buffer)            | **Terraform**       | edit `terraform/main.tf` → CI apply  |
| Secrets Store secrets (Xano token, HMAC)   | **Terraform** (refs)*| put value in Secrets Store UI/CLI; reference in HCL |
| WAF / rate-limit ruleset                   | **Terraform**       | edit `terraform/main.tf` → CI apply  |
| DNS records, TLS                           | **Terraform**       | edit `terraform/main.tf` → CI apply  |
| Worker script + routes                     | **Wrangler**        | edit `workers/src/**` → CI deploy    |
| Worker bindings (to KV/Queue/Secret)       | **Wrangler**        | edit `workers/wrangler.toml`         |
| Gradual / canary rollout %                 | **Wrangler**        | `wrangler versions` in CI            |
| Data model, function stacks, RBAC          | **Xano** (native)   | Xano branch → Metadata API/UI        |
| The API contract (`openapi/xano.v1.json`)  | **Xano, exported**  | regenerate from Xano → commit → PR   |

\* Secret **values** live only in Cloudflare Secrets Store. Terraform/Wrangler reference
them by name; the plaintext is never in git and never in an AI context.

## Platforms vs. Terraform providers

Four platforms, **one Terraform provider**. A "provider" is a published plugin that
teaches Terraform to drive a platform's API — and across this stack only Cloudflare
has a real one. The other planes get the same discipline (a git-based, reviewable,
deterministic representation) from their own native tooling. That unevenness is the
honest architecture, not a gap to fill.

| Plane          | Terraform provider?                                  | Its deterministic path instead |
|----------------|------------------------------------------------------|--------------------------------|
| **Cloudflare** | ✓ Official, mature (`cloudflare/cloudflare`, pinned in `versions.tf`) | Terraform for the infra ring (KV, queues, WAF, DNS, secret refs) + Wrangler for the Worker itself |
| **Shopify**    | Partial, community-only — not a paved path           | Shopify CLI + theme/app code in git (`shopify app deploy`, theme push) |
| **Xano**       | None                                                 | Xano branches + the Metadata API scripted in CI; the exported OpenAPI spec is the git artifact |
| **Webflow**    | None                                                 | DevLink / Data API — plus the worker rails: fragments, variable ingest, published-Collection data layers |

Do **not** adopt the community Shopify providers to force symmetry: they cover
fragments of the Admin API, lag behind it, and would put a third manager on resources
the worker and the Shopify CLI already own — which violates the drift rule above.

### This stack's narrowing — structure as security, by design

In this stack, Terraform's only interaction surface is the **server-side Cloudflare
data plane — the KV namespaces — and it is locked**. This is not a limitation to
grow out of; it is the security posture. The boundary is enforced by the structure
itself — what each tool *cannot* reach — rather than by policy or trust:

- **Terraform-owned means locked.** The KV namespaces Terraform manages are changed
  only by editing HCL and letting CI apply. No dashboard edits, no wrangler by hand,
  no MCP.
- **Not available to AI.** AI's only relationship to this layer is authoring HCL diffs
  upstream of the gate. It never applies, never reads or writes the KV data itself,
  and the data is server-side only — never exposed to a client, never present in an
  AI context.
- Everything else Terraform *could* manage (WAF, DNS, queues) stays with its current
  owner until deliberately adopted per the table above — adoption is a decision,
  not a default.

## The AI / non-AI-dev handoff

```
 AI / MCP (upstream)              Deterministic gate (humans own)        Production
 ─────────────────────           ───────────────────────────────        ──────────
 author HCL + Worker TS  ──PR──►  CI: contract test                ──►   gradual deploy
 update openapi spec              CI: terraform plan                     (1% → 10% → 100%)
 commit                          CI: typecheck + lint                    instant rollback
                                  human review + approve
```

- **AI is always upstream of the gate.** It proposes diffs; it never applies, never
  holds a secret, never touches a dashboard.
- **Non-AI devs work entirely from committed code + the OpenAPI contract.** They review
  real diffs in a normal IDE. No MCP required to be productive here.
- **The gate is non-negotiable.** Nothing reaches production without passing contract
  tests, a reviewed `terraform plan`, and a canary rollout.

## Secrets: the two that matter

1. `XANO_SERVICE_TOKEN` — least-privilege Xano API token, per environment. Rotate on a
   schedule; Secrets Store makes rotation central.
2. `EDGE_HMAC_KEY` — shared signing key. The Worker signs each Xano request with it;
   Xano rejects any request whose signature is missing or invalid. This is what makes
   Xano trust *only* your edge, even though its URL is on the internet.

Neither value ever appears in `wrangler.toml`, in `.tf` files, in git, or in an AI
prompt. CI injects them at deploy from Secrets Store.

## Environments

`dev` / `staging` / `prod` are Xano branches. Each has its own Worker deployment
pointing at the matching Xano environment via `wrangler.toml` env vars. Promote together;
never point a prod Worker at a non-prod Xano branch.
