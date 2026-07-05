# Ddokdan Ilbo — Design System (`ddokdan-ds`)

The CSS-only design system for **Ddokdan Ilbo (똑단일보)**. It carries the
tokens (colour, type, spacing), the shared `.cdn-*` component visuals, the A4
print contract, and the web layer as one versioned package. This same package
is the **single source of truth** for every Ddokdan Ilbo output — the print/CD
issues and the online site both consume it, at the same version tag, so a
printed page and a web page read as the same newspaper.

## Entry points

| File | Consumer | How it is consumed |
|---|---|---|
| `styles.css` | Print / CD pipeline | Linked at **runtime** via a pinned jsDelivr URL. The single public API for print — do not link the tier files individually. |
| `web.css` | Website (Next.js) | Imported at **build time** as a pinned git dependency: `import 'ddokdan-ds/web.css'`. Does not link the CDN at runtime. |
| `core.css` | Either / tooling | The medium-neutral token foundation only (no print contract, no web overlay, no components). For a consumer that supplies its own composition. |

## Consuming the package

**Print / CD — jsDelivr, runtime link, pinned tag:**

```html
<link rel="stylesheet"
      crossorigin="anonymous"
      href="https://cdn.jsdelivr.net/gh/vvoohyeon/ddokdan-ds@v1.0.0/styles.css">
```

**Website — git dependency, build-time import, pinned tag:**

```jsonc
// package.json
"dependencies": {
  "ddokdan-ds": "github:vvoohyeon/ddokdan-ds#v1.0.0"
}
```

```js
// app entry
import 'ddokdan-ds/web.css';
```

## Versioning

- **Pin to a version tag, always** — e.g. `@v1.0.0`. Both consumers resolve the
  identical tag; that shared pin is what keeps print and web visually in sync.
- **Never** consume `@latest`, a branch, or an unpinned range. An unpinned
  reference lets the two consumers drift onto different bytes and breaks the
  cross-media guarantee below.

## Token architecture (three tiers)

- **`tokens/core/`** — fonts, colours, typography, spacing, element defaults.
  Medium-neutral; the frozen foundation both media share.
- **`tokens/print/print-core.css`** — the CSS-native A4 page contract (page
  fidelity, unbreakable blocks, uncropped photos). Print entry only.
- **`tokens/web/`** — `web-layer.css` (web-only variables: interaction states,
  fluid type, breakpoints, responsive knobs) and `web-components.css` (layout /
  interaction overlays on the shared elements + web-only furniture).
- **`tokens/components.css`** — the shared `.cdn-*` component visuals. Consumed
  **by both** `styles.css` and `web.css`; this is the structural guarantee of
  cross-media consistency.

**Type sizing — fixed content, fluid chrome.** Article content is **fixed
pixels = print**: shared `.cdn-*` elements carry their own pixel sizes in
`components.css` (body 18px, headline 31px, masthead 60px, hook 21px) and use
them identically at every viewport, so print and web read the same. The
`--fs-*` scale in `tokens/core/typography.css` mirrors those render sizes; where
a component sets its own size, that hardcode is the source of truth. Fluid type
(`--fs-*-fluid`) is reserved for **web-only chrome** (shell, issue-head) and
must never be applied to shared content — doing so would break print/web parity.

## Cross-media consistency principle

The shared content elements — body copy, headings, kickers, figures, the
알고 있었나요? box, 어휘 카드 vocabulary cards, 오늘의 질문 callouts, and the
quiz — have exactly **one** visual source of truth: `tokens/components.css`.
Print and web import that same file, so those elements are identical across
media by construction. The web layer (`web-components.css`) may only add
**layout mechanics** (grid columns, widths, outer margins) and **interaction**
(hover, focus, reveal affordances) on top; it must never restate a shared
element's colour, border, radius, shadow, typography, inner padding, or label.
Contributors extending the web layer must keep to that boundary — any web
override of a shared element belongs to layout or interaction only, and the
resting appearance must match print.
