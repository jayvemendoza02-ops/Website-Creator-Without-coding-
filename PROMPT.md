# Improved prompt

Copy everything below the line into a fresh chat with a capable coding agent.
It is the original request rewritten as an executable brief: concrete scope,
hard constraints, a data model, and acceptance criteria.

---

## Build "Openframe", a browser-based visual website builder that exports open code

Design and implement a working visual website builder that runs entirely in the
browser. The user composes pages by dragging and dropping blocks, styles
everything through a point-and-click inspector, previews responsive layouts, and
exports a plain static website (HTML, CSS and a little JS) that a human can read,
edit, and host anywhere.

The output must be **open**: no proprietary runtime, no proprietary file format,
no build step required to view the exported site. If the user unzips the export
and opens `index.html`, it works.

### Hard constraints

- **No frameworks and no build step.** Vanilla HTML, CSS and JavaScript only.
  Classic browser scripts (not ES modules) so that double-clicking the editor's
  `index.html` works from `file://` without a local server.
- **No network dependency for the editor itself.** The editor must boot offline.
  Google Fonts and remote demo photos may load from the network but must degrade
  gracefully.
- **No backend.** Projects live in `localStorage` and can be saved/opened as JSON
  files. All export happens client-side, including the ZIP (implement a minimal
  ZIP writer with STORE entries and CRC32 rather than pulling a library).
- **Readable output.** One CSS rule per element, semantic class names derived from
  element names, `@media` blocks for responsive overrides, no hashed class names,
  no inline style soup, no runtime hydration.
- **Keep editor chrome separate from exported styles.** The editor is allowed to
  be opinionated; the exported site must contain only the website.
- Ship as a folder the user can open directly. No package manager, no bundler, no
  `npm install`.

### Coverage: everything HTML and CSS can express

The point of the tool is that the user never has to write code. That only holds
if the element catalog and the style schema cover the whole language:

- **Elements:** cover the everyday HTML vocabulary, not just marketing blocks —
  `form`, `label`, `input`, `textarea`, `select`, `option`, `checkbox`, `radio`,
  `ul`/`ol` + `li`, `table` + `tr` + `td` + `th`, `details` + `summary`,
  `figure` + `figcaption`, `blockquote`, `pre`, `video`, `audio`, `iframe`,
  `progress`, `meter`. Keep a raw-HTML escape hatch for anything exotic.
- **Element-specific content fields:** an input must expose type, id, name,
  placeholder, value and required; a video must expose source, poster, controls,
  autoplay, loop and muted. Generic fields are not enough.
- **CSS:** ship a single property table that drives both the inspector and the
  stylesheet generator, so the two can never drift. It must cover position and
  insets, z-index, float/clear, flex grow/shrink/basis/order, grid template,
  auto flow and placement, row/column gap, height, min/max, aspect ratio, box
  sizing, overflow, object/background position, size, repeat and attachment,
  per-side borders, outline, font style, indent, word spacing, white space,
  decoration, vertical align, text overflow and wrap, hyphens, list style,
  transform, filter, backdrop filter, blend mode, transition, cursor, user
  select, pointer events, visibility and resize.
- **Escape hatch:** one free-text "custom CSS" field per element for
  declarations the controls do not cover. It must apply to the canvas preview
  and to the export identically.
- **Forms must be real:** a login or contact form built from native elements
  must export as `<form action method>`, `<label for>`, `<input type required>`
  and `<button type="submit">` — not as a blob of escaped HTML.

### Editor experience

Three-column layout, light UI, calm and precise (think a small design tool, not a
dashboard):

1. **Left sidebar** with four tabs:
   - **Add** — searchable palette of ready-made blocks (navbar, hero centered,
     hero split, feature cards, logo row, stats, gallery, testimonial, pricing,
     call to action, footer). Click or drag to insert.
   - **Layers** — live tree of the current page: select, rename, reorder by drag,
     hide, lock, delete.
   - **Pages** — add, duplicate, rename, delete pages; set URL path, browser
     title and meta description.
   - **Theme** — edit eight color tokens and two font roles; everything that uses
     tokens updates live.
2. **Center canvas** — a real rendered page inside a device-sized viewport, with
   zoom-to-fit, selection outline, drag-to-reorder, and drop indicators. Double-
   click headings, text, buttons, links and icons to edit copy inline. Show
   floating duplicate / hide / delete actions and resize handles for supported
   elements.
3. **Right inspector** — schema-driven, not hand-written per element. Two tabs:
   **Design** and **Content**.
   - Design groups: Layout, Typography, Spacing, Background, Border & shadow,
     Effects.
   - Layout: display, direction, wrap, align items, justify, gap, grid columns,
     self align, push to bottom, width, max width, min height, image fit.
   - Typography: font, size, weight, line height, tracking, alignment, case, color.
   - Spacing: padding and margin with a link-all-sides toggle.
   - Background: color, image URL, size, overlay strength and overlay color.
   - Border & shadow: radius, width, style, color, shadow presets.
   - Effects: opacity, hover behavior, on-scroll animation.
   - Reset buttons for breakpoint overrides.

Top bar: editable site name, undo/redo, Desktop / Tablet (≤900px) / Mobile
(≤600px) switch, autosave state, Preview, Export code, and a File menu (new blank
site, load demo, save project JSON, open project JSON, export).

### Responsive editing model

Each element stores `styles.base`, `styles.tablet` and `styles.mobile`. Editing
in Tablet or Mobile writes only the changed properties as overrides and shows a
reset affordance. The canvas renders the merged style for the active breakpoint;
export emits `@media (max-width: 900px)` and `@media (max-width: 600px)` blocks
containing only overrides. Do not duplicate entire rule sets into media queries.

### Design tokens

Ship a default theme:

```js
colors: primary, primaryDark, accent, ink, muted, surface, canvas, border
fonts:  heading, body   // ids that map to a font stack
```

Elements reference tokens as `$primary`, `$heading`, and so on. In the editor,
resolve tokens to concrete values. On export, emit them once as CSS custom
properties (`--color-primary`, `--font-heading`) and reference
`var(--color-primary)` throughout the stylesheet so the exported site can be
re-themed from one place. Handle alpha/tint values with `color-mix()` and keep
hex fallbacks sane.

### Document model

Keep one plain-JSON document as the single source of truth:

```js
{
  version: 1,
  name: "Lumen",
  theme: { colors: {...}, fonts: { heading: "space-grotesk", body: "inter" } },
  pages: [{
    id, name, path, title, description,
    nodes: [{
      id, type, name, hidden, locked,
      props: { text, level, href, src, alt, anchor, attrs, ... },
      styles: { base: {...}, tablet: {...}, mobile: {...} },
      children: [...]
    }]
  }]
}
```

Element types: section, container, grid, heading, text, button, link, image,
icon, divider, spacer, embed, form, label, input, textarea, select, option,
checkbox, radio, list, listitem, table, row, cell, headercell, video, audio,
iframe, details, summary, figure, figcaption, blockquote, code, progress, meter.
Containers accept children; the root accepts only sections and other top-level
nodes. All mutations go through a store that validates the document, snapshots
for undo/redo, notifies listeners, and autosaves with debounce. Editing a style
key on desktop deletes the override; editing on mobile stores it only if it
differs from the inherited value.

### Export requirements

Generate these files from the document:

- One `.html` per page (`/` → `index.html`, `/about` → `about.html`), with
  semantic tags, real inter-page links, `alt` text, page titles and meta
  descriptions, and scroll-reveal `data-*` attributes where used.
- `styles.css` with a readable reset, token variables, one rule per element,
  hover rules, animation keyframes, and the two media-query blocks.
- `script.js` (~1–2 KB): mobile nav toggle, IntersectionObserver reveal, current
  year. The exported site must still render and link correctly with JS disabled.
- `README.md` with hand-off notes.
- A ZIP download containing the bundle, written without libraries.

Escape user content correctly: HTML text, attribute values, CSS url() quotes.
Never emit `undefined`, `NaN`, `[object Object]` or `nullpx`.

### Quality bar

- The demo site ("Lumen", Home + About) must look designed, not like a wireframe:
  real hierarchy, generous spacing, consistent type scale, restrained color,
  a highlighted pricing tier, a testimonial band, a strong footer.
- Keyboard: `Ctrl/Cmd+Z` undo, `Ctrl/Cmd+Shift+Z`/`Ctrl+Y` redo, `Ctrl/Cmd+D`
  duplicate, `Delete` remove, `Enter` edit text, `Esc` deselect, `Ctrl/Cmd+S`
  save.
- Guard the inspector and render pipeline against null selection, empty pages,
  locked/hidden nodes and deleted parents.
- Use SVG icons (sprite + `document.createElementNS` when building icons in JS).
- Accessible basics: focus states, labels on icon buttons, alt text, sensible
  heading order.

### Acceptance criteria

Verify these before calling it done:

1. Double-clicking `index.html` opens the editor with the demo site and no
   console errors.
2. Selecting a top-level section and a deeply nested element both render the
   inspector; inline text editing commits to the document.
3. Dragging a block from Add inserts it at the drop position; layer drag reorder
   and delete work; undo/redo restores every step.
4. Switching to Mobile and changing a style creates an override that disappears
   on Desktop and exports inside `@media (max-width: 600px)`.
5. The mobile navbar hides its links and the Menu button opens them as a second
   row in the exported site.
6. Export produces balanced braces, two media blocks, valid tag nesting, no
   suspicious tokens, and a ZIP that standard tools can open.
7. Screenshots of the exported site at 1440px and at 390px (use an iframe for
   true 390px emulation if the headless browser enforces a wider minimum) show
   no overflow, clipping or broken sections.
8. A form built in the editor exports as native HTML: the exported file
   contains `<form`, `<label for=`, `<input type="email" required>` and
   `<button type="submit">`, and the field ids match their labels.
9. Every element type renders on the canvas and in the export, including
   `details`/`summary`, `table`/`tr`/`td`/`th`, `video`, `iframe`, `progress`
   and `meter`.
10. A new CSS property set in the inspector (for example `position: absolute`
    plus `z-index`) appears in the exported rule for that element, and a custom
    CSS declaration is written verbatim into the same rule.

### Deliverables

- `index.html` and `assets/` for the editor.
- `README.md` explaining how to run it, the feature set, the file map, keyboard
  shortcuts and limits.
- The demo project embedded so the first run is not an empty canvas.

Do not describe what you would build; build it, test it in a real browser,
screenshot the result, fix what the screenshots reveal, and only then report
back with the file list and the checked acceptance criteria.
