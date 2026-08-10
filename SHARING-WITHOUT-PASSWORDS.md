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

## Where this goes next — screens without pages

The reason this model beats the password isn't just the audit trail. It's that **a grant never needed a login page** — so it works on things that don't have one.

**IoT, without a DOM.** The household's devices — the TV, the thermostat, the console — have no browser and no form to type into. That's exactly where passwords fail hardest (typing a password with a remote is the punishment for the whole model). Here, a device holds a grant the same way a person does: paired once with a short, human-typeable code — an alphabet with no O/0 or I/1, because someone reads it off a screen across the room — and from then on it acts inside its mandate: what it may fetch, until when, for which household. Every request it makes lands in the same ledger as everyone else's. No DOM, no password, no shared secret sitting in a smart TV's settings screen.

**Gaming, with peers.** A game world is just another surface with no page — and "the household" generalizes to "the party." Five peers in a session is the same shape as five household members: a declared count on a grant, passes-on allowed inside it, fan-out beyond it a finding. Items, access, and unlocks travel as grants bound to accounts — a shared item can't be duplicated by forwarding, for the same reason a forwarded invite link can't become a second grant.

**And both run on data shaping.** A device and a game don't read your page — they read the model. The same deterministic shape that renders your banner renders a console lobby or a device's little status screen: one JSON contract, merged the same way every time, with provenance saying which layer set each field. Design once at the model level, and the surfaces without DOMs inherit it exactly like the ones with them.

**Dark warehouses — governance with nobody watching.** A lights-out warehouse is machines acting all day with no human in the room — which is exactly where ungoverned automation becomes a black box. Run it without agents under mandates and without server-side governance, and the building works fine right up until nobody can say what it did or why. The alternative is already on this page: every robot, scanner, and pick-arm acts inside a mandate — what it may move, fetch, or ship, with a cap and an expiry — and every scan lands as a row in the ledger. The dark warehouse doesn't need a supervisor on the floor; it needs what the household got. Governance isn't a person watching. It's a record being written.

**Packages with evergreen QR data — the surface with no screen at all.** A printed code is the most permanent thing you ship: fixed the moment ink meets board, scanned for years by systems that don't exist yet. Evergreen means the *ink* never changes while the *data* never goes stale — the code resolves to a domain you own, and behind it sits the living model: today's price, today's manual, today's recall status, this buyer's language. Repoint the data, never reprint the box. It's data shaping for paper — the package is just one more surface reading the shape, and the scan it invites is one more ledger row in the same chain the warehouse and the household write to.

The household was never the edge case. It was the preview — for the party, the living room, the warehouse floor, and the box on the doorstep.

---

*See it live: the Share panel on any gated asset — visibility (Public / Group / Private / Specific People), single-use invites, expiry, declared peers, and the access record underneath, chain-verified on every read.*
