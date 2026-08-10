# Xano & Terraform, for Designers

Two words you'll hear from the dev side of the stack, translated. Neither is your tool — but both explain why your work behaves the way it does, and why nothing you publish depends on anyone's memory.

## Xano — the record book

Xano is where the **facts** live: customers, orders, permissions, consent, receipts. When a surface you designed shows a price, a name, or a "you have access" state, that value was not typed into the page — it was read from the record book at that moment.

Why you care: **your designs are windows, not copies.** A window can be redesigned, swapped, or deleted without the facts changing — and the facts can change without anyone re-editing your design. That's the same contract as the Shapes collection: you author a layer of the data, and every surface that looks at the data follows. Delete the dashboard, the permission still answers.

## Terraform — the component system for infrastructure

You already believe in this idea; you call it a **component**. Define something once, and every instance follows the definition — change the master, every copy updates; nobody hand-edits an instance and hopes.

Terraform is exactly that, for infrastructure. The servers, caches, security rules, and domains behind the stack are written down as plain-text definitions in version control, reviewed like any other change, and applied by machine. Nobody "just tweaks a setting" in a dashboard — because a hand-tweak is the infrastructure version of a detached style: invisible, unrepeatable, and destined to fight the master definition later. There's even the same rule you know from design systems: **every resource has exactly one source of truth** ([the ownership map](OWNERSHIP.md) lists who owns what).

## Why this reaches your work

The whole stack — your Shapes collection, the git-based models, the capability that gates overrides, the infrastructure underneath — follows one discipline:

1. **Everything is written down** — a shape in git, a collection you publish, an HCL file, a record in Xano. No tribal knowledge, no "ask the person who set it up."
2. **Everything merges deterministically** — layers with a fixed order, where the same inputs always produce the same result, and every value can tell you where it came from.
3. **Every change has a gate and a trail** — a Publish button, a capability, a reviewed pull request, a ledger row. Change is cheap; *silent* change is impossible.

Design isn't downstream of this system. The Shapes collection makes you an author *inside* it — same rules, same guarantees, same provenance as everyone else.

## Deeper reads

- [Ownership map & the drift rule](OWNERSHIP.md) — who owns which resource, and why the same resource is never managed from two places.
- [Terraform × GitHub setup](TERRAFORM-GITHUB-SETUP.md) — the dev-side runbook, if you want to see what the "component system for infrastructure" looks like in practice.
- [The 5-minute demo](DEMO-SHAPE-5MIN.md) — watch the whole loop run: git base, your layer, a gated override, a verified ledger.
