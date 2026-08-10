# Design Ops

The operating model for the design/development boundary across a multi-plane stack (Shopify · Webflow · Xano · Cloudflare): **teams exchange deterministic data shapes, not content edits.**

## The shift

| Old division of labor | Design Ops |
|---|---|
| Designers edit pages; devs re-implement them | Both sides author against a **data model** |
| Content lives in a CMS editor | The base shape lives in **git**, reviewed like code |
| Overrides are hand-edits that drift | Overrides **merge deterministically** — every field carries provenance for which layer won |
| Access is CMS logins | Writes pass a **capability gate** (`design:shape:write`) — held by role, or minted by purchase |
| Change history is memory | Every commit lands in a **hash-chained ledger** anyone can verify |

Four load-bearing rules:

1. **The shape is the contract.** A versioned JSON model is what designers, developers, and machines all read. Editing a surface is never the source of change — the surface follows the data.
2. **Override logic is deterministic.** Defaults → base (git) → design layer (published Webflow Collection) → override (edge KV), with per-field provenance in every read. Invalid input is dropped, never erred, so a bad layer can't poison the merge — the next layer down simply wins.
3. **Permissions gate the write.** Reads are public. Writes require a capability that arrives by role (designer entitlement, `design`/`content_edit` team tags) or by purchase (the SKU-grant rail: purchase = permission).
4. **Every change is on the record.** Commits and resets hash-chain into an append-only ledger; `chain_intact` is recomputed on every read.

## In this repo

- **[WHY-THE-STACK.md](WHY-THE-STACK.md)** — the design guideline, in plain designer speak: why Webflow/Shopify/Xano/Cloudflare, the six rules (designer first · non-destructive · many contributors · roles & permissions · timestamps & receipts · one name for everything), and what good looks like on the page.
- **[DEMO-SHAPE-5MIN.md](DEMO-SHAPE-5MIN.md)** — the live 0→1 proof: ten curl steps from reading the git-based shape to a capability-gated write, a re-rendering surface, and a verified chain.
- **[SETUP-WEBFLOW-SHAPES.md](SETUP-WEBFLOW-SHAPES.md)** — the designer setup, 1·2·3: author a shape layer from a Webflow Collection — create, publish, verify provenance. No code, no deploy.
- **[XANO-TERRAFORM-FOR-DESIGNERS.md](XANO-TERRAFORM-FOR-DESIGNERS.md)** — the two dev-side words translated for designers: Xano as the record book, Terraform as the component system for infrastructure, and why design authors *inside* the same discipline.
- **[OWNERSHIP.md](OWNERSHIP.md)** — who owns which resource (Terraform vs Wrangler vs the backend's native tooling), and the one drift rule that keeps AI-authored changes and human-owned infrastructure from fighting over the same resource.
- **[TERRAFORM-GITHUB-SETUP.md](TERRAFORM-GITHUB-SETUP.md)** — the 0→1 guideline for putting the infrastructure layer under GitHub-gated Terraform: least-privilege tokens, remote state, the PR plan/apply gate, progressive import, drift checks.
- **[diagrams/omni-directional-hub.svg](diagrams/omni-directional-hub.svg)** — the architecture in one picture: four caller types (chat UI, external agents, any frontend, the worker itself), one commerce brain, three stores — every call carrying its own identity, no sessions, no state held.

## The division of labor, restated

**AI authors upstream of the gate.** It proposes shapes, HCL, and worker code as reviewable diffs. **Humans own the gate** — the PR review, the plan, the approval environment, the secrets. **Machines enforce the rest** — deterministic merges, capability checks, ledgers, contract tests. Nobody edits production by hand, and nothing depends on anyone's memory.

---

*Docs only — this repository intentionally contains no application source code.*
