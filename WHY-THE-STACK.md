# Why This Stack — the Design Guideline

Webflow, Shopify, Xano, Cloudflare. Four tools, one reason: **so design leads, nothing gets destroyed, and nobody has to trust anybody's memory.**

Plain version of who does what:

- **Webflow** is where you draw. Pages, styles, and now data — your Shapes collection is a real input to the system, not a mockup of one.
- **Shopify** is where people buy. Checkout, orders, accounts — solved problems you never have to design from scratch.
- **Xano** is the record book. Names, orders, permissions, receipts. Your pages are windows onto it, not copies of it.
- **Cloudflare** is the in-between. It takes what everyone authored, merges it the same way every time, and serves it fast, everywhere.

## The six rules

### 1 · Designer first
Design isn't downstream. You draw it, you publish it, every surface follows — the store, the pages, the app. Nobody "implements" your change later; publishing *is* the change. If a surface can't inherit your work automatically, that's a bug in the system, not a ticket for you to file.

### 2 · Non-destructive
Nothing overwrites anything. The system works in layers, like your design tools do: a base layer underneath, your published layer on top, and every change can step back. Leave a field empty and the layer below shows through. Type something invalid and it's simply ignored — a bad value can't break the page, the layer below wins.

### 3 · Many contributors
Everyone works in their own layer — developers in code, designers in Webflow, operators in settings — and the layers stack in one fixed order. Nobody edits anyone else's layer, so nobody's work vanishes because someone else saved last. And every value on every surface can tell you which layer it came from. Not to assign blame — so nobody has to guess.

### 4 · Roles and permissions
Who may change what is a setting, not a habit. You hold the pencil for your layer; changing someone else's takes a permission you either have or don't. When you don't, the system says so and says how to get it — it never just fails quietly. And buying a product can carry a permission with it, which is why "access" here never means a shared password.

### 5 · Timestamps and receipts
Every save has a *when* and a *who*, written in one date format everyone and everything can read (ISO — `2026-08-10`, no ambiguity about months and days). Changes land in a running record that can be checked but not quietly edited. When someone asks "who changed this and when" — and someone always asks — the answer is a lookup, not a meeting.

### 6 · One name for everything
Every piece has one full, unambiguous name — the model has a name, its fields have names, permissions have names, even each customer's space has its own prefix. Two things can never collide by both being called "final," and searching always finds the one true copy. Same discipline as naming your layers and components — applied to everything.

## On the page

The same care, at the surface level — pulled from the [design board](https://miro.com/app/board/uXjVHzE2Qc4=/?share_link_id=425952194337) on Miro:

- **The headline shows up and machines can read it.** A visible headline in a real H1 — not a styled div, not baked into an image. If the title is absent or unreadable to a machine, the page doesn't exist to half its audience.
- **The CTA lands somewhere real.** The button leads to an actual action — a purchase event, a full-height frame that responds — never a scroll into nothing.
- **Media behaves.** An accessible Play button; fullscreen playback with close, volume, and scrubbing — on every platform, not just the one it was designed on.
- **Alternates are tested, not argued.** When two versions could work, both ship as an A/B — the data settles it.

## The lineage — none of this is new

This stack isn't a bet on new tools. It's the same ideas maturing for fifteen years, and we've been building on the API baseline for the last seven.

- **It starts with Ruby.** The era that gave us HBO GO-class applications: convention over configuration, background jobs doing the heavy work off-stage, templates as windows onto data. Those habits are the DNA of everything below.
- **Node came *from* Ruby.** Watch [Ryan Dahl's origin talk](https://youtu.be/EeYvFl7li9E?si=4UBevQ2PDdb54DzR) — Node was born from what Ruby's servers couldn't do: stay light, never block, handle everything as it arrives. There's a slide in that talk introducing `process.ENV` — the runtime handing your code its environment. Seventeen years later, the edge function in this stack receives its `env` the same way; the idea never changed, it just moved to every city on earth. The Cloudflare layer is Dahl's talk fully grown.
- **Business Catalyst and PhoneGap made a promise** — one build that reaches every surface, sites and apps from one source. They were early; the promise was right. AEM and edge functions are that promise finally kept.
- **Liquid is Ruby's gift to this stack.** Born at Shopify, adopted by GitHub — look at what GitHub and Liquid have shipped since 2020. When your Shopify theme renders, that's the Ruby lineage serving your design.
- **Background tasks never went away.** The Ruby-era pattern — queue the heavy work, run it on schedule, keep the page fast — is exactly [Xano's background tasks](https://docs.xano.com/building/logic/background-tasks) today: preload the data, sync the records, off the request path. The page stays instant because the work already happened.

The takeaway for a designer: the layers, the receipts, the publish-and-it-flows model — these aren't inventions of this project. They're the settled habits of the systems that won, assembled in one place.

## Where to go next

- [Author a shape from Webflow](SETUP-WEBFLOW-SHAPES.md) — the 1·2·3 setup.
- [Xano & Terraform, for designers](XANO-TERRAFORM-FOR-DESIGNERS.md) — the two dev-side words, translated.
- [The 5-minute demo](DEMO-SHAPE-5MIN.md) — watch all six rules run once, live.
