# Terraform × GitHub — 0→1 Setup Guideline

How to take this scaffold from a folder on disk to a governed, GitHub-gated Terraform pipeline. Companion to `OWNERSHIP.md` (read that first — it defines who owns what; this page is the *how*). Time budget: ~30 minutes once, then every change is a normal PR.

---

## 1 · Create the repo and push the scaffold

```sh
cd "xano-cloudflare-handoff 2"
git init -b main
git add -A && git commit -m "handoff scaffold: terraform + worker proxy + contract gate"
gh repo create xano-cloudflare-handoff --private --source=. --push
```

Private repo. The scaffold contains no secrets by design — keep it that way (`.gitignore` already excludes `*.tfvars`, `.terraform/`, `terraform.tfstate*`).

## 2 · Mint the least-privilege Cloudflare API token

Cloudflare dash → My Profile → API Tokens → Create Token (custom). Scope it to exactly what `terraform/main.tf` manages:

| Permission | Level |
|---|---|
| Account · Workers KV Storage | Edit |
| Account · Queues | Edit |
| Account · Secrets Store | Edit |
| Account · Account Rulesets (WAF/rate-limit) | Edit |
| Zone · DNS (your zone only) | Edit |
| Zone · Zone WAF | Edit |

One token per environment when you split dev/staging/prod. This token is **CI's identity** — a human applying locally uses their own, so the audit log distinguishes the two.

## 3 · Remote state on R2 (stay in one cloud)

State maps HCL to real resource IDs — it must be shared, locked, and never in git. You're all-in on Cloudflare, so use R2's S3-compatible API instead of adding AWS/Terraform Cloud:

```sh
npx wrangler r2 bucket create tfstate   # once
```

In `terraform/versions.tf`, replace the backend comment with:

```hcl
backend "s3" {
  bucket                      = "tfstate"
  key                         = "xano-cloudflare-handoff/prod.tfstate"
  region                      = "auto"
  endpoints                   = { s3 = "https://<ACCOUNT_ID>.r2.cloudflarestorage.com" }
  skip_credentials_validation = true
  skip_region_validation      = true
  skip_requesting_account_id  = true
  skip_metadata_api_check     = true
  skip_s3_checksum            = true
  use_path_style              = true
}
```

Create an R2 API token (Object Read & Write, `tfstate` bucket only) and export its pair as `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` — locally in your shell, and in GitHub as repo secrets. (Names are the S3-protocol convention; the values are R2 credentials.)

> R2's S3 API does not support state *locking*. With CI as the only applier (next section) that's acceptable — serialize applies through the merge queue. If you later want real locking, Terraform Cloud's free tier is the drop-in alternative backend.

## 4 · GitHub repo secrets + environment gate

```sh
gh secret set CLOUDFLARE_API_TOKEN        # the token from step 2
gh secret set CLOUDFLARE_ACCOUNT_ID
gh secret set AWS_ACCESS_KEY_ID           # R2 state credentials
gh secret set AWS_SECRET_ACCESS_KEY
```

Then two settings in the GitHub UI:

1. **Settings → Environments → New environment `production`** → check *Required reviewers*, add yourself. The scaffold's `deploy` job already declares `environment: production`, so merge-to-main pauses for a human click before anything ships. This is the human gate from `OWNERSHIP.md`, enforced by GitHub.
2. **Settings → Branches → protect `main`**: require PRs, require the `verify` and `plan` status checks to pass.

## 5 · First plan/apply (the 0→1 moment)

```sh
cd terraform
cp terraform.tfvars.example terraform.tfvars   # fill account_id, zone_id, environment — stays local, gitignored
terraform init                                  # connects the R2 backend, downloads the pinned provider
terraform plan                                  # read the diff — everything should be a create
terraform apply                                 # the only hand-run apply; after this, CI owns it
```

Then put the *values* of the two runtime secrets in Secrets Store (never in HCL/git):

```sh
npx wrangler secrets-store secret create XANO_SERVICE_TOKEN
npx wrangler secrets-store secret create EDGE_HMAC_KEY
```

Commit the (value-free) tfvars example + any HCL adjustments and push. From here forward: **edit HCL → PR → CI runs `terraform plan` on the PR → human reads the diff → merge → approval gate → apply/deploy.** Nobody — human or AI — runs `apply` from a laptop again.

## 6 · Importing what already exists

Your zone, DNS records, and existing KV namespaces predate Terraform. Two rules:

- **Don't rush to import everything.** A resource stays dashboard/MCP-owned until it's deliberately adopted. Un-imported ≠ broken; it's just not Terraform-owned yet (`OWNERSHIP.md` table is the registry of what has been adopted).
- When adopting, write the HCL first, then bind it to the live resource:

```sh
terraform import cloudflare_workers_kv_namespace.edge_cache <account_id>/<namespace_id>
terraform plan   # must show NO changes — that proves the HCL matches reality before CI ever applies
```

Adopt in this order: net-new resources (queue, secrets refs, rate-limit rules) → DNS records → WAF. Leave the existing production Worker with Wrangler forever (that's its owner per the table).

## 7 · The drift rule, operationalized

- A **weekly drift check** keeps everyone honest — add a scheduled job that runs `terraform plan -detailed-exitcode` and opens an issue on exit code 2 (drift found). Drift means someone changed a Terraform-owned resource out-of-band; the fix is a PR that either codifies or reverts it — never a silent re-apply.
- **MCP/AI stops touching a resource the moment it's imported.** Upstream of the gate, AI edits HCL like any contributor; the PR is where its work becomes reviewable.

## Day-2 cheat sheet

| You want to… | Do |
|---|---|
| Change a rate limit / WAF rule / KV binding target | Edit `terraform/main.tf` → PR |
| Change Worker code or bindings | Edit `workers/src/**` / `wrangler.toml` → PR (CI deploys, canary-first) |
| Rotate `XANO_SERVICE_TOKEN` | New value in Secrets Store; nothing in git changes |
| Bump the provider | Edit `versions.tf` pin → PR (lockfile updates like package-lock) |
| See why prod differs from HCL | `terraform plan` output on the drift issue |
| Emergency revert | `wrangler rollback` for the Worker; `git revert` + merge for infra |
