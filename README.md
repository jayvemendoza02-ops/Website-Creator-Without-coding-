# Openframe

A visual website builder that runs entirely in the browser and exports plain,
readable HTML, CSS and JavaScript. No account, no server, no build step, no
proprietary file format.

![Openframe editor](docs/preview-editor.png)

## Quick start

1. Open `index.html` in a modern browser (Chrome, Edge, Firefox or Safari).
   Double-clicking the file works — there is nothing to install or compile.
2. The **Lumen** demo site loads on first run so you have something real to
   take apart.
3. Click any element on the canvas, change it in the inspector on the right, and
   press **Export code** when you are happy.

The editor autosaves to `localStorage` as you work. The exported site is a
separate, static site you can host anywhere.

## What you can do

### Build with blocks
The **Add** tab has fifteen ready-made blocks: navbar, hero (centered and split),
feature cards, logos, stats, gallery, testimonial, pricing, log in, contact form,
FAQ, comparison table, call to action and footer. Click or drag one onto the
canvas. Blocks are ordinary elements afterwards — nothing is locked.

The **Log in** and **Contact form** blocks are built from real form elements
(`<form>`, `<label>`, `<input>`, `<textarea>`, `<button type="submit">`), so the
exported site has a working form without a line of hand-written code.

### Edit on the canvas
- Click to select, drag the blue handles to reorder, drop blocks between
  existing sections.
- Double-click any heading, text, button, link or icon to edit its copy inline.
- The **Layers** tab is a live tree of the page: rename, hide, lock, reorder and
  delete from there.
- The toolbar switches between **Desktop**, **Tablet (≤900px)** and
  **Mobile (≤600px)**. Style changes made in a smaller breakpoint are saved as
  overrides and exported as real media queries.

### Style with the inspector
The inspector is generated from a schema, so every element gets the same
predictable controls:

- **Layout** — display, direction, wrap, alignment, gap, grid columns, width,
  max width, min height, self align, push to bottom.
- **Typography** — font, size, weight, line height, tracking, alignment, case,
  color.
- **Spacing** — padding and margin with a link-all-sides toggle.
- **Background** — color, image URL, size, overlay tint.
- **Border & shadow** — radius, border, shadow presets.
- **Effects** — opacity, hover behavior, on-scroll animation.
- **Position** — position, inset, z-index, float, clear.
- **Flex & grid** — grow, shrink, basis, order, grid template, auto flow,
  grid placement, row/column gap.
- **Box & background** — height, aspect ratio, box sizing, overflow, object and
  background position/repeat/attach, blend mode, accent color.
- **Border sides** — per-side border widths and colors, outline.
- **Text detail** — italic, indent, word spacing, white space, decoration,
  vertical align, overflow, wrap, hyphens, list style.
- **Advanced effects** — transform, filter, backdrop filter, blend mode,
  transition, cursor, selection, pointer events, visibility, resize.
- **Custom CSS** — a free-text box for any declaration the controls do not
  cover (`letter-spacing:.02em`, `clip-path:…`, `content:"→"`).

### Every element HTML has
37 element types are on the **Add** tab, grouped so they stay findable:

| Group | Elements |
| --- | --- |
| Structure | section, container, grid |
| Text | heading, text, blockquote, code, label, summary, figcaption |
| Actions | button, link, icon, divider, spacer |
| Media | image, video, audio, embed frame, figure |
| Forms | form, input, textarea, select, option, checkbox, radio |
| Data | list, list item, table, table row, table cell, header cell, progress, meter |
| Advanced | embed (raw HTML) |

Content fields are element-specific: an input exposes type, id, name,
placeholder, value and required; a select exposes id, name and multiple; a
video exposes source, poster, controls, autoplay, loop and muted.

### Use design tokens
The **Theme** tab edits eight colors and two font roles. Elements reference
tokens (`$primary`, `$heading`), so changing a token updates the whole site.
On export, tokens become CSS custom properties:

```css
:root {
  --color-primary: #4f46e5;
  --font-heading: 'Space Grotesk', ui-sans-serif, system-ui, sans-serif;
}
```

### Work with pages
The **Pages** tab adds, duplicates, renames and deletes pages, and sets each
page's URL path, browser title and meta description. The first page always
exports as `index.html`; `/about` exports as `about.html`. The demo ships with
four pages: Home, About, Log in and Contact.

### Preview and export
- **Preview** opens a standalone rendered copy of the current page in a new tab.
- **Export code** shows every generated file with syntax-free but readable
  source. Download a single file or the whole site as a `.zip`.

Exported bundle:

| File | Contents |
| --- | --- |
| `index.html`, `about.html`, … | Semantic HTML with real links between pages |
| `styles.css` | One rule per element, plus `@media` blocks for overrides |
| `script.js` | ~1.5 KB: mobile nav, scroll reveal, current year |
| `README.md` | Short hand-off notes for whoever hosts the site |

The CSS is intentionally class-based and flat: one readable rule per element
(`.hero-title { … }`), no CSS-in-JS, no generated hashes, no runtime framework.
Delete the JS file and the site still works — only the menu and reveal
animations need it.

## Project files

Openframe is dependency-free and uses classic scripts on purpose, so `file://`
works without CORS or a dev server.

```
index.html            editor shell and SVG icon sprite
assets/editor.css     editor chrome (not exported)
assets/js/
  util.js             DOM helpers, escaping, download, slugify
  schema.js           element specs, style schema, tokens, fonts, breakpoints
  model.js            document model, tree helpers, style → CSS mapping
  blocks.js           the eleven insertable blocks
  demo.js             the Lumen demo site
  render.js           document → canvas DOM
  store.js            state, undo/redo, localStorage autosave
  ui.js               buttons, fields, popovers, toasts
  inspector.js        schema-driven inspector panels
  panels.js           Add / Layers / Pages / Theme sidebars
  dnd.js              canvas and layer drag-and-drop
  zip.js              minimal ZIP writer (STORE + CRC32)
  exporter.js         HTML/CSS/JS generation and download
  app.js              boot, selection, overlay, inline editing, files, keys
```

The document is plain JSON:

```json
{
  "version": 1,
  "name": "Lumen",
  "theme": { "colors": { "primary": "#4f46e5" }, "fonts": { "heading": "space-grotesk" } },
  "pages": [{ "id": "pg_1", "name": "Home", "path": "/", "nodes": [] }]
}
```

`Save project (.json)` writes exactly this shape; `Open project…` reads it back.
The format is documented by that file, not by a spec you have to trust.

## Keyboard shortcuts

| Keys | Action |
| --- | --- |
| `Ctrl/Cmd + Z` | Undo |
| `Ctrl/Cmd + Shift + Z`, `Ctrl + Y` | Redo |
| `Ctrl/Cmd + D` | Duplicate selection |
| `Delete` / `Backspace` | Delete selection |
| `Enter` | Edit selected text inline |
| `Esc` | Deselect / close dialog |
| `Ctrl/Cmd + S` | Save to local autosave now |

## Notes and limits

- The editor and the exported site are static. There is no CMS, form handling,
  authentication or database; add those with your own backend or a service.
- Remote images (the demo uses `picsum.photos`) and Google Fonts need a network
  connection. Use your own hosted assets for production.
- The mobile navbar is deliberately minimal: links collapse behind a **Menu**
  button and open as a second row.
- `localStorage` is per browser and per origin. Opening the file from a
  different path or browser gives you a different autosave slot. Export a
  project JSON if you want a durable copy.
- No license file is included. Add one before publishing if you plan to
  distribute the editor itself.
#   W e b s i t e - C r e a t o r - W i t h o u t - c o d i n g -  
 