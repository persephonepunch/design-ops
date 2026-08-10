# The Design System — fast-load core, fallback ladder, tokens, native-chrome kill list

> Published from the working design-system skill (2026-08-10). This is the enforcement-grade companion to [WHY-THE-STACK.md](WHY-THE-STACK.md): the same six rules, as executable doctrine — load order, the L1–L5 fallback ladder, token consumption with the canonical SASS-Overrides names, component scoping, the custom-select and native-chrome normalization rules, content canon, and the load-order TDD that enforces all of it.


# Skill: Design System (fast-load core · fallback · tokens)

## The one rule
Every UI surface is built in **three stacked layers that fail downward, never upward**. Each upper
layer only *enhances* the one below; remove any upper layer and the surface degrades to a still-valid
state — never blank, never thrown.

```
Layer 0  Machine-readable HTML core   ← paints FIRST, works with ZERO CSS / no worker / no JS
Layer 1  Core fallback CSS (literals) ← hardcoded brand: "Gotham Book", Arial, literal radius/colors
Layer 2  Design-token variables       ← var(--_apps---*) wins when present; updates on Webflow publish
```

The whole system collapses to one authoring habit — **one line per themed property**:
```css
font-family: var(--_apps---typography--button-font, "Gotham Book", Arial, sans-serif);
                 └────────── Layer 2 (token) ──────────┘  └──── Layer 1 (literal fallback) ────┘
```
Token present → Layer 2 wins (brand updates on publish, multi-tenant). Token absent/renamed → Layer 1
literal. CSS stripped entirely → Layer 0 markup still conveys content/structure to humans, agents, and
assistive tech. **No `var()` is ever written without a literal fallback.**

---

## 1. Execution order (what runs, in time)
1. **Webflow static markup paints** — the page's own semantic header/footer/form HTML renders before any
   network call. This is Layer 0, the fast-load core. First meaningful paint never waits on a worker.
2. **Inline loader fetches the worker embed** (source of truth — never the `.html` design file):
   - Header → `cf-worker-webflow-sync` `/embed/header` (`HEADER_HTML` constant) · `Cache-Control: max-age=60`
   - Footer → `cf-worker-crm-sync` `/embed/footer` (`CRM_FOOTER_HTML`) · `Cache-Control: no-cache` (live on deploy)
   - Other embeds (`collection-grid`, `collection-grid-inline.js`, `kb-search.js`) → served from
     `WEBFLOW_SYNC_STATE` KV as `embed:<name>` (namespace `577959a8…`); 404 if the blob is missing.
3. **Injected HTML overrides the static baseline.** If the fetch 404s/errors, step 1's markup remains =
   the L1 asset fallback (degrade, don't blank).
4. **Dependencies settle** — injected scripts wait for readiness (`Webflow.push` / `DOMContentLoaded` /
   `customElements.whenDefined`) before touching the DOM. Never a bare top-level `$(...)`/`getElementById`.
5. **Styling resolves fallback-first, then tokens** (Layer 1 literals, then Layer 2 `var()` overrides).

**Cache asymmetry (spec it):** footer is instant on deploy; header + KV embeds lag up to **60s** behind a
publish/deploy. Don't conclude "the fix didn't work" inside that window — cache-bust with `?cb=` or wait 60s.

---

## 2. Fast-load core (Layer 0)
- The surface must be **meaningful with CSS disabled**: real headings, labels, lists, buttons, links —
  not `div` soup hydrated only by JS. Agents (AEO), screen readers, and a failed worker all read this.
- First paint comes from inline Webflow markup; worker embeds + tokens are **progressive enhancement**
  layered async. Never block first paint on a `fetch`.
- A surface with an empty mount and a pending `fetch` must show a **skeleton/placeholder**, never blank.

---

## 3. Fallback ladder (degrade at every load stage)
| Layer | Trigger | Required behavior |
|---|---|---|
| **L1 Asset** | KV blob / embed missing (404) | host shows skeleton/placeholder + `console.error`, never blank |
| **L2 Dependency** | jQuery / Webflow / web-component absent | component **no-ops gracefully** (guarded), logs, never throws |
| **L3 Mount** | target element missing | abort cleanly (no error storm); optionally a labeled fallback box |
| **L4 Token** | design token absent / renamed | literal brand fallback in the `var()` |
| **L5 Data** | API empty / 500 (`/collection`, etc.) | empty-state UI (e.g. `#pim-empty`), never blank |

---

## 4. Design tokens — consume Webflow Variables, don't hardcode
The published site exposes **Webflow Variables as CSS custom properties on `:root`** (~81 of them). The
embed renders in the same document, so it reads them with `var()`. Semantic set (purpose-built for apps):

| Token | Current value |
|---|---|
| `--_apps---typography--button-font` | `"Gotham Book", Impact, sans-serif` |
| `--_apps---typography--heading-font` | `"Gotham Book", Impact, sans-serif` |
| `--_apps---typography--body-font` | `Roboto, Arial, sans-serif` |
| `--_fonts---gotham` / `--_fonts---gotham-bold` | Gotham Book / Gothamultra |

Map: body→`body-font`, titles/headings→`heading-font`, buttons/CTAs→`button-font`. Benefits: changes flow
on **Webflow publish** (no KV/worker edit), and the *same* embed blob is **multi-tenant brand-portable**
(each store publishes its own `:root`). **Caveats:** Webflow auto-generates/hashes the names — stable
unless the variable or its collection is renamed (one already showed a `<deleted|variable…>` artifact), so
**always keep the literal fallback**. Variable **modes** (per-brand/dark) switch values automatically.

### 4b. UIkit Global naming — the "SASS Overrides" alignment
A second Webflow Variable collection ("**SASS Overrides**") names variables **exactly after UIkit's
`$global-*` Sass variables, minus the `$`** — so **one name spans three layers**: Webflow Variable
(`--global-primary-background`) = UIkit Sass override (`$global-primary-background`) = the worker brand
token. Edit the Webflow Variable → it flows to the compile and (via the bridge) to live UIkit.

**Names are universal, values are per-brand.** Every brand's site creates the *same* names; only the
values differ. The bridge CSS is brand-agnostic (`var(--global-primary-background, …)`); each site/registry
supplies its value. So the SAME setup serves every brand by slug (`/brand/<slug>/…`).

**The full Global set (set these; everything derives):** `global-primary-background`,
`global-secondary-background`, `global-background`, `global-muted-background`, `global-color`,
`global-emphasis-color`, `global-muted-color`, **`global-inverse-color`** (the white text/buttons on dark
sections — the built-in inverse), `global-border`, `global-link-color`, `global-link-hover-color`,
`global-success-background`, `global-warning-background`, `global-danger-background`, `global-font-size`,
`border-rounded-border-radius`. **Buttons, sections (default/muted/primary/secondary), inverse
(uk-light/uk-dark), status colors, and `uk-svg` icons all infer from these** — "set the Globals, everything
follows." UIkit has NO CSS variables, so the live link is the worker's `.uk-*` bridge in `theme.css`; the
authoritative path is the Sass compile (`/brand/<slug>/uikit.scss?full=1` → `sass site.scss > site.css`).
Don't prefix with `uk-`. Full detail: the `uikit` skill + `docs/DESIGN-SYNC.md`.

### 4c. Derived states — compute from the token, never hand-pick

Hover/active/pressed shades are **computed from the base token, not stored as separate hexes** —
one rule serves every brand, and a brand change on Webflow publish re-derives its states for free.

**The standard hover rule: 30% darker = keep 70%, mix 30% black, in oklch** (perceptual — no
muddy/hue-shifted srgb mixing):
```css
.card:hover { background: color-mix(in oklch, var(--brand, #1a1a1a) 70%, black); }
```
- Dark surfaces invert: hover goes **lighter** — `color-mix(in oklch, var(--brand) 70%, white)`.
- **Dosage by surface type** — 70/30 is for **filled controls** (black CTAs, buttons); large
  **surface washes** (cards, rows, panels) take **~95/5** — 30% black on a pale grey card is a
  slab, not a hover (store System Grid precedent: `#fff → #f4f4f4` ≈ 4%). Pair every derived
  state with a base-state `transition: background-color .12s ease` so the shade eases, not snaps.
- **Webflow authoring:** the `=` custom-value field takes the **value only** (no `background:`
  prefix, no `;`), the token must be a variable that exists on the site (SASS Overrides names,
  e.g. `var(--global-muted-background, <its current hex>)` — insert via the variable picker so
  renames stay wired), and set it on the class's **Hover state**.
- Enhancement tier (literal "×0.7 lightness", keeps full chroma — newer browsers):
  `oklch(from var(--brand, #1a1a1a) calc(l * 0.7) c h)`.
- Ladder it like everything else (L1 literal → computed):
  ```css
  .card:hover { background: #141414; }                    /* L1 literal */
  @supports (color: color-mix(in oklch, red 70%, black)) {
    .card:hover { background: color-mix(in oklch, var(--brand, #1a1a1a) 70%, black); }
  }
  ```
- **Banned:** a hand-picked darker hex per brand/per component for a state — that's a second
  source of truth that drifts the moment the token changes.

---

## 5. Component CSS scoping (prevents the bleed bugs)
- **Scope every rule under the component root** (`#pim-collection …`, `.crm-pp-* …`). No bare element
  selectors (`img {}`, `button {}`) — they get clobbered by, or clobber, the host page. (The squished
  thumbnail bug was the embed's own `.pim-card img { width:100% }` beating `.pim-variant-thumb`.)
- Win the **global no-radius reset** (`*{border-radius:0 !important}`) for specific CTAs with a more
  specific `!important` rule (`#pim-collection .pim-card-btn { border-radius: var(--…--radius, .375rem) !important }`)
  rather than deleting the reset.
- Mounting into a host element that carries host classes (e.g. `.collectiongrid`) drags the host's
  descendant CSS into your component — neutralize (`display:block`, strip class) or mount in a neutral node.

---

## 6. Mount contract
- Each hydrator declares **one mount id** and renders into the *intended* one. The collection grid mounts
  in **`#collection-div`** (main area), NOT the stray footer `#collection-grid`. A wrong mount renders the
  surface offscreen/below the fold = looks blank even when the DOM is full.
- `innerHTML` does not execute `<script>` — re-create script nodes when injecting an HTML embed.

---

## 7. UI Load-order TDD (enforcement)
Lives in `tests/harness/suites/ui-load-order.ts`, same pattern as `embed-integrity.ts`
(`testAsync`, RED/GREEN phases). Run: `npx tsx tests/harness/runner.ts --live --suite=ui-load-order`.
**Data-driven** off a component/token registry (per surface: mount id, worker asset URL(s), required deps,
required `--_apps---*` tokens, L1–L5 fallback). Stage assertions:

1. **Asset resolves** — every referenced worker asset → 200 + non-empty (catches the missing-KV-blob blank page).
2. **Mount contract** — declared mount id exists and is the intended one.
3. **Dependency readiness** — loader guards on `Webflow.push`/`DOMContentLoaded`/`customElements.whenDefined`.
4. **CSS scoping** — no bare `img{}`/`button{}`; rules namespaced under the component root.
5. **Token + fallback** — no `var()` without a literal fallback.
6. **Cache/TTL sanity** — fast-update surfaces declare the right `cache-control`.
7. **No secret leak** — (already in `embed-integrity.ts`).

---

## 8. Authoring checklist (any new modal / form / embed)
- [ ] Layer 0 markup is meaningful with CSS off (semantic, labeled, machine-readable).
- [ ] Every themed property is `var(--token, <literal>)` — never a bare `var()`.
- [ ] All CSS scoped under the component root; no bare element selectors.
- [ ] One declared mount id; renders into the intended node; scripts re-executed on inject.
- [ ] Deps guarded (no top-level DOM access); empty-state + skeleton present (L1/L5).
- [ ] Registered in the `ui-load-order` registry so the suite covers it.
- [ ] Knows its cache TTL (footer no-cache vs header/KV 60s).

## 9. Component rules — native-chrome kill list (baseline)

Source of record: the design-rule cards in `~/Documents/00ysl/portfolio/00designrules/*.jpg`
(`selectskill`, `alertskill`, …). The cards are being folded into this skill; each card → one rule
below. **Baseline set — extend as more cards are captured** (each new rule keeps the same shape:
*what's banned · what to build · monochrome + Layer-0 fallback*). The throughline: **kill OS chrome,
repaint in the monochrome system** (greys + black/white, color only when significant — see §4 tokens).

### 9.1 Selects / dropdowns — `selectskill.jpg`
- **Banned:** a raw native `<select>` on any branded surface. The OS renders its own popup (system-blue
  highlight, system font, detached rounded chrome, drop shadow) that ignores the design system and looks
  different on every OS/browser.
- **Build a custom dropdown:**
  - *Trigger* — full-width, **square corners**, hairline border (`var(--_apps---color--border, var(--core-border, #d0d0d0))`),
    body font, selected value left-aligned, chevron right. On open, border → focus/brand (the input focus blue).
  - *Panel* — **attached directly under the trigger, same width**, white bg, square, hairline border. No
    detached OS popup.
  - *Options* — full-width rows, comfortable padding, body font. Hover → `--core-surface` grey.
  - *Highlighted / selected option* — **black bg / white text** (`--core-text` fill), **never OS blue**.
  - *A11y* — `role="listbox"`/`role="option"`, arrow-key nav, Enter/Esc, visible focus ring (native parity).
  - *Layer-0 fallback* — if JS is absent, degrade to a plain native `<select>` (functional, unstyled) — never a dead `div`.

### 9.2 Buttons — `alertskill.jpg`
- **Primary** — black filled (`var(--_apps---color--text, var(--core-text, #1a1a1a))` bg, white text),
  **square corners** (win the global no-radius reset with a specific `!important`), medium weight.
- **Secondary / Cancel** — ghost: transparent bg, text-only (or hairline border), monochrome.
- **Destructive** (Delete, etc.) — uses the **same black-filled primary, NOT red**. Color only when
  significant; the monochrome rule holds.
- **Banned:** native browser button chrome / default OS button styling.

### 9.3 Dialogs — `alertskill.jpg`
- **Banned:** `alert()` / `confirm()` / `prompt()`. They render OS chrome ("<site> says…"), block the main
  thread, can't be styled, and read as untrusted.
- **Build a custom modal:** centered card on a dimmed backdrop — title, one-line body ("This cannot be
  undone"), right-aligned action row with **ghost Cancel + filled primary** (§9.2). Square corners, white
  surface, hairline border.
- **Behavior:** Esc + backdrop-click close, focus trap while open, return focus to the trigger on close.

### 9.4 Links on modal-served surfaces + the trigger-attribute namespaces
- **External links on any page served inside the site modal iframe** (worker pages like
  `/wrong-shape`, KB doc renders) **MUST be `target="_blank" rel="noopener"`.** Third-party
  sites send `X-Frame-Options`/`frame-ancestors` and an in-frame navigation renders as a
  blocked-frame error (Rithum precedent, 2026-07-14). Same-origin links may stay in-frame —
  the modal's Back button covers the return trip.
- **Namespace registry** — what an authored element's id/attributes may mean; never mix them:
  - `data-crm-nav="<key>"` (attr) or `id="crm-nav-<key>"` (id form) — **app-surface triggers.**
    Keys from the worker's `navLinks()`: `design-sync · brand-studio · get-the-app · teams ·
    keys · dashboard · configure · docs-chat`. Behavior: login gate where the surface is
    authed, click-time token, opens in the site modal. The element ALWAYS keeps a real
    `href` (the worker URL or a store page) as the no-JS / logged-out fallback, set to open
    in **This tab** — the handler preventDefaults when it runs.
  - `data-crm-action="<action>"` — privacy/consent + brand-studio contract
    (`reset-consent`, `open-brand-studio`, …) handled by the footer/brand loaders.
  - `data-auth-state="logged-in" | "logged-out"` — session-scoped visibility; `updateAuthUI`
    toggles these, never hand-roll show/hide.
  - **Cart ids — two surfaces, two pairs, never interchangeable:**
    - **Webflow sites** (master header embed, PIM cart): trigger `id="nav-cart"` (hookNavCart —
      strips the href, injects the badge, click opens the cart drawer) + injected badge
      `id="nav-cart-badge"`. There is a text-contains-"CART" fallback, but it breaks under
      relabeling/translation — always set the explicit id. Author a real href
      (`https://www.crm-sync.dev/cart`) as the no-JS fallback; the hook removes it at runtime.
    - **Shopify theme** (crm-header.liquid, Shopify cart): link `id="crm-nav-cart"` + count
      `id="crm-cart-count"`, SSR'd from Liquid and kept live via `/cart.js`.
    - A section pushed between surfaces carrying the other surface's cart id silently no-ops.
  - **Semantic section ids** (`segment-hero`, `consent-first`, …) — kebab-case, unique per
    page, stable forever: they are simultaneously deep-link anchors, JSON-LD `@id`s, and
    #ID-push section keys. Non-trigger elements must NEVER take a `crm-nav-*` id.

### 9.5 Card-as-link hover — kill the inherited underline
- **Symptom:** a whole card wrapped in a single `<a>` (product card, tile, list row) underlines
  **every line of text** on hover — eyebrow, title, body, price — because the site/theme ships a
  global `a:hover { text-decoration: underline }` and the card is one link, so the decoration
  propagates to all descendant text. Looks broken; reads as a mis-styled hyperlink, not a card.
- **Rule:** a card/tile/row whose ROOT is the link is a *surface*, not a text link — its hover
  affordance is the whole surface (background/color/elevation shift), never text-decoration. Set
  `text-decoration: none` on **both** the base and the `:hover` state, and win the global rule:
  ```css
  .card, .card:hover { text-decoration: none !important; }
  .card:hover { background: var(--_hover, #0a0a0a); color: #fff; }  /* surface feedback */
  ```
  `.card:hover` (0,2,0) already outranks `a:hover` (0,1,1); the `!important` is belt-and-suspenders
  against a themed `a:hover { …!important }`. Precedent: the "Our products" scroll card (`.ops-card`),
  worker + `crm-products-scroll.liquid`, 2026-07-20.
- **Inline text links are the opposite** — a real `<a>` *inside* running copy keeps its underline
  (`text-underline-offset:.15em`); the no-underline rule is ONLY for surfaces whose root is the link.
- **Enforcement:** flag any element that is an `<a>` (or `[href]`) with block/flex/grid display and
  more than one text child, lacking `text-decoration:none` on `:hover`.

### 9.6 _(reserved — more cards to add)_
`securitylayers.jpg`, `ratio-scale.jpg`, `widthratio.png`, `meta/social/publishing/livestream` not yet
distilled. Add each as a `9.x` subsection in the same shape when captured.

> **Enforcement hook:** the `ui-load-order` suite (§7) should grow a "no native chrome" assertion —
> flag any `<select>` not behind the custom-dropdown component, and any `alert(`/`confirm(`/`prompt(` in
> embed/modal source — once these baseline rules are live in code.

---

## 10. Content canon — no duplicate pages (AI-first data governance)

**Core ethos: every piece of content has exactly ONE canonical page. Duplicate pages with the same
content confuse the user — and they confuse agents, answer engines (AEO), and search the same way.**
A product that sells governed, deduplicated data cannot ship a duplicated content surface.

- **One canon per topic.** E.g. "How it works" (the 5-step setup: Install the app → Connect Shopify →
  Interactive Key Ceremony → Set tenant config → Webflow Data Designer) lives ONLY at
  `crm-sync.webflow.io/product#setup`. Every other surface **links there** — it never restates the steps.
- **Link, don't restate.** A page that needs to reference another topic gets a pointer
  (`See how it works — the 5-step setup →`), not a copy. Precedent: `crm-sync.dev/sell` (2026-07-03) —
  its "How it works" nav/CTA now deep-link to `/product#setup`, and its own section was retitled to
  what it uniquely owns ("Book the work").
- **Sections own ONE job.** If a section's content duplicates another page, the section is either
  retitled to its unique job or reduced to a link.
- **Kill or draft the clones.** Copy-pages (`*-old`, `*-copy`, duplicated slugs) are drafted/archived
  before publish — a published clone is a bug, not a backup. Backups live in git/Webflow versions.
- **Why AI-first:** agents and LLMs ingest every published surface as ground truth. Two pages saying
  the same thing in drifting versions = two conflicting answers. Canon + links is how the site stays
  a clean retrieval corpus (one URL per fact, hash-deep-linkable like `#setup`).

---

## 11. Typographic rag — widows & line endings (the typography term, NOT Agentic RAG)

**Disambiguation first: in this section "rag" = the uneven right edge of left-aligned text**
(rag-right composition). It is unrelated to Retrieval-Augmented Generation — this stack runs
agentic RAG (Vectorize KB, `/kb/search`), so the two senses coexist in this repo; a "rag rule"
in a design context always means typography.

**The rule: no widows on any authored surface.** A paragraph's last line must never be a single
word or a stub (`it.`, `too.`, a bare number). Headlines must never strand one word on their
own line. Applies to worker-served HTML pages, embeds, modals, decks, and PDFs we generate
going forward (no retroactive PDF regeneration — fix at the source, let the next render pick
it up).

- **Paragraph endings — glue the last 2–3 words** with `&nbsp;`:
  `…structurally can't&nbsp;carry&nbsp;it.` If the tail can't fit, all glued words wrap together —
  a one-word last line becomes impossible at any viewport width. Precedent: `/wrong-shape`
  closing paragraph (worker `index.ts`, 2026-07-14).
- **Markdown/download twins stay clean.** Don't put literal NBSPs in `.md` files served as
  downloads — the fix belongs only in rendered HTML/CSS surfaces.
- **Headlines** — `text-wrap: pretty` on `h1–h3` wherever we own the CSS. As of Safari 18.4
  (WebKit, 2025-03) `pretty` evaluates the WHOLE paragraph — rag, hyphens, AND short last
  lines — so it supersedes `balance` as the house default for headings (webkit.org/blog/16547).
  `balance` remains acceptable for short centered display heads (≤3 lines) where equal line
  lengths read better than a tight rag. Progressive enhancement — safe no-op in old engines.
  Applied 2026-07-17: crm-chrome.css h1–h6, .crm-board h1/h2 (KB/docs boards), /app hero,
  /compare h2, /start + /get h1. Break-where-the-sense-breaks (`&nbsp;` at phrase
  boundaries) is still correct for load-bearing lines — CSS is the default, glue is the
  guarantee.
- **Body copy** — where we own the stylesheet (doc.css shell, embeds), `p { text-wrap: pretty }`
  is the zero-markup baseline; `&nbsp;` glue remains the guaranteed fix for load-bearing copy.
- **Never `text-align: justify`** on web surfaces — rivers and bad word spacing; rag-right is
  the system's alignment.
- **Keep the rag calm.** No successive lines stepping in/out sharply; if a line ends on a weak
  word (a, an, the, of, to — or a dangling `—`), glue it to the next word.

---

## 12. Button-group alignment on mobile — gap, never sibling margins

**The rule: a group of buttons/CTAs is ALWAYS a flex row with `gap` — never siblings spaced by
`margin-left`.** Sibling margins (`.cta + .cta { margin-left: .6rem }`) look identical on desktop
but break the moment the group wraps on a narrow viewport: every wrapped button keeps its
margin-left and renders **indented instead of flush left** — a staircase of misaligned black
buttons. Because the bug only exists at mobile widths, it ships silently from desktop previews.

**House pattern** (precedents: `.fw-cta-row` /pages/firmware, `.pim-cta-row` /pages/pim-sync —
both 2026-07-21, worker `index.ts`):

```css
.x-cta-row { display: flex; flex-wrap: wrap; gap: .6rem; margin-top: 1.4rem; }
.x-cta-row .x-cta { margin: 0; }   /* the row owns ALL spacing; buttons own none */
```
```html
<div class="x-cta-row">
  <a class="x-cta" href="…">Primary action →</a>
  <a class="x-cta" href="…">Secondary →</a>
</div>
```

- **Spacing lives on the container, not the children.** `gap` applies both horizontally
  (side-by-side) and vertically (wrapped stack), so one declaration is correct at every width.
  Any `margin` on the buttons themselves is zeroed inside the row.
- **Vertical rhythm too**: the group's `margin-top` moves onto the row container — a button's own
  top margin double-spaces once it wraps under a sibling.
- **`flex-wrap: wrap` is mandatory.** A non-wrapping row overflows the viewport; the wrap into a
  flush-left stack IS the mobile design, not a fallback.
- **Full-width mobile stack (optional)**: when stacked buttons should span the container, add
  `@media (max-width: 40rem) { .x-cta-row .x-cta { flex: 1 1 100%; text-align: center; } }` —
  still no margins.
- **Review trigger**: any `+`-combinator margin between interactive controls
  (`.a + .a { margin-left }`) is a defect — rewrite as a gap row on sight, same treatment as a
  bare `img{}`/`button{}` selector (§5).
- **Verify at mobile width** before shipping: 390px viewport (or DevTools iPhone preset) — left
  edges of all buttons in the group must align with the body text above them.

---

## References
- Worker source of truth + deploy: `header-footer-automation` skill.
- Live brand tokens, fallback ladder, suite plan: memory `ui-design-system-load-order-tdd`.
- Embeds served from KV (`embed:<name>`, namespace `577959a8…` / `WEBFLOW_SYNC_STATE`) on `cf-worker-webflow-sync`.
- Closest existing suite to copy: `tests/harness/suites/embed-integrity.ts`.
