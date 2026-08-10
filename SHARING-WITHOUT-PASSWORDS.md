# Sharing Without Passwords — the household problem, solved

Streaming taught the whole world one habit: when five people in a house want the same thing, you share the password. Everyone knows how that story goes — the password travels beyond the house, the company can't tell a family from a leak, and one day the crackdown locks out grandma.

The problem was never the sharing. The problem was the *instrument*. A password is a copy — once given, it's gone from your control, and nobody can say who holds it. So the platform's only responses are to trust everyone or punish everyone.

The Share panel replaces the copy with a **grant**:

- **You invite people, not distribute a secret.** Each address gets a single-use link that expires. The system stores a hash of the token, never the token itself, and redemption binds it to that person's account — a forwarded link cannot become a second grant.
- **Sharing onward is a setting, not a betrayal.** "Holders may pass this on" — with a declared peer count. Five household members? Set 5. The intent is written down, so passing it on inside that intent is *allowed*, not tolerated.
- **Fan-out beyond it is a finding, not a lockout.** The line on the panel is the whole philosophy: exceeding the declared count doesn't slam a door on the family — it surfaces in the record as a fact the publisher can see and act on. Enforcement with evidence, instead of punishment on suspicion.
- **Everything lands in the ledger.** Grants, denials, invites rejected, every unwrap — a running, tamper-evident record. When something looks off, the answer is a lookup, not an accusation.
- **Access can simply end.** An expiry date on the grant — "a grant with no end date is a grant nobody revisits." No password rotation, no "change it and re-text everyone."

The nice irony: the lineage of this stack runs back through the HBO GO era — the very generation of apps whose password-sharing defined the problem. Same household, same five people, same wish to share — but now the sharing is the *feature*, with names, limits, timestamps, and receipts.

**The one-line version: stop sharing the key; share the permission.**

---

*See it live: the Share panel on any gated asset — visibility (Public / Group / Private / Specific People), single-use invites, expiry, declared peers, and the access record underneath, chain-verified on every read.*
