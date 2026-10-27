# Decal Lab

Color blank papercraft templates in your browser. Click a part to fill it, pick colors from your reference images, and print a colored PDF at the template's original size.

Made for Gundam papercraft, but it works with any line-art template: black cut lines, grey fold lines, white parts.

Your template is never altered. Lines stay pixel-for-pixel as in the original file, and colors go underneath them.

## Getting started

Decal Lab is a single file, `index.html`, with no build step and no install.

- **Open it directly:** double-click `index.html`.
- **Or through UniServer:** `http://localhost/wordpress/wp-content/themes/extensions/recolor/`

Then drop a template onto the page, or click **Open template**.

| Template type | Supported |
|---|---|
| PNG / JPG | Yes, used at their original size |
| PDF | Yes, all pages, drawn at 300 DPI |
| Pepakura (.pdo) | No. Export it to PDF from Pepakura first. |

PDF opening and PDF export load a library from a CDN the first time you use them, so they need an internet connection. PNG/JPG templates and PNG export work offline.

## Coloring

### Tools

| Tool | Key | What it does |
|---|---|---|
| Fill | `F` | Click inside a part to fill it with the current color |
| Clear part | `U` | Click a part to remove its color |
| Replace color | `R` | Click a colored part. Every part with that exact color changes to the current color. |
| Brush | `B` | Paint details the template doesn't outline, like vents, stripes or markings |
| Eraser | `E` | Erase paint with a brush |
| Pick color | `I` | Pick a color you already used on the template |
| Pan | `H` | Drag to move around. Holding `Space` does the same with any tool. |

### Fill options

- **Stop at fold lines** (on by default). With it on, a fill colors one face of a part. Turn it off to fill the whole part across its fold lines in one click.
- **Ctrl-click** does the opposite of the current fold-line setting, for a single click.
- **Shift-click** fills the white paper around the parts. A normal click there is blocked, so you don't color the background by accident.
- **Close gaps in outlines** (0–4 px). If a fill leaks into the next part through a break in a line, undo and raise this by 1 or 2.

### Brush options

- **Brush size:** a slider, or `[` and `]` on the keyboard.
- **Brush stays:**
  - **Inside the part** (default): paint is limited to the part where the stroke started.
  - **Inside the lines it touches:** paint never crosses a line. Use it for narrow corners and slivers the fill can't reach. While you paint, the stroke shows faded, then it's trimmed when you let go.
  - **Anywhere:** no limits.

The lines always sit on top of your colors. Painting over a black line doesn't change how it looks.

## Colors and references

- **Current color:** use the color box, type a hex code like `#1F4FA8`, or click a swatch.
- **Reference image:** drop or paste an image into the Reference panel, or use **Open reference**.
  - Scroll to zoom and drag to pan.
  - Click to pick a color. It averages a 3×3 pixel area, so photos and JPEG noise give a clean color.
  - **Extract palette from image** adds the image's main colors to your palette.
- **Palette:** click **+** or **Add** to save the current color. Right-click a swatch to remove it.
- **Saved palettes:** give a palette a name, for example `RX-78-2`, and click **Save**. Load it later for the next kit.
- **Recently used:** fills and brush strokes add their color here automatically.
- **Pasting an image (`Ctrl+V`):** if no template is open, it opens as the template. Otherwise it opens as the reference.

## Multi-page PDFs

- Every page loads as a thumbnail in the **Pages** panel.
- Click a thumbnail, or press `Page Up` / `Page Down`, to switch pages.
- Each page keeps its own colors and undo history.
- A blue dot marks pages you've colored.

## Exporting

Click **Export**.

| Option | Output |
|---|---|
| Colored PDF | All pages in one file, ready to print. PDF templates keep each page's original paper size. Image templates fit the chosen paper: A4, Letter or A3. |
| Colored PNG | The current page, at full template resolution |

When printing, choose **100% / Actual size**, not "Fit to page". Otherwise the parts won't be the right size to build.

## Keyboard shortcuts

| Key | Action |
|---|---|
| `F` `U` `R` `B` `E` `I` `H` | Switch tools (see the Tools table) |
| `Ctrl+Z` | Undo |
| `Ctrl+Y` or `Ctrl+Shift+Z` | Redo |
| `Space` + drag, or middle-mouse drag | Pan |
| Mouse wheel | Zoom at the cursor |
| `+` / `-` | Zoom in / out |
| `0` | Fit to screen |
| `[` / `]` | Smaller / bigger brush |
| `Page Up` / `Page Down` | Previous / next PDF page |
| `Alt`-click | Pick a color from the template |
| `Ctrl`-click | Fill with the opposite fold-line setting |
| `Shift`-click | Fill the background outside the parts |

## Good to know

- **Your work isn't saved when you close the tab.** Export before closing or refreshing. The page warns you if you have unexported changes.
- **Palettes and recent colors are saved in this browser** (`localStorage`). A different browser or a private window won't have them.
- **Print quality matches the template.** A low-resolution template, around 100 DPI, prints slightly soft. Decal Lab doesn't upscale or redraw it.
- **Fills reach to the line, not past it.** Where two lines actually meet in a sharp corner, the last hairline is line, so it stays uncolored. It's hidden by the line in the print.
- **Blurry PDFs:** in PDFs that contain a blurry, enlarged image, filling across fold lines (unticked or Ctrl-click) can spill into neighboring parts. Normal fills with "Stop at fold lines" on aren't affected.

## How it works

For developers:

- **One file:** HTML, CSS and plain JavaScript in `index.html`. No framework.
- **Libraries, loaded only when needed:**
  - [pdf.js 3.11.174](https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/) for reading PDFs
  - [jsPDF 2.5.1](https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/) for PDF export
- **Layers:**
  - A transparent color canvas sits under the original template.
  - The template is drawn with `mix-blend-mode: multiply`, so lines always show through the colors.
  - Exports combine the layers the same way.
- **Finding part edges:**
  - The template is turned into an "ink" map and blurred slightly.
  - The blurred peak height tells cut lines from fold lines. Outlines peak about twice as high as folds, and the split point is chosen automatically with Otsu's method.
  - A pixel only blocks a fill if it's both near a line and part of the line's dark core. This keeps thin strips fillable without letting fills leak across lines.
- **Fills:** a scanline flood fill, then grown a couple of pixels under the line so no white seam is left.
- **Undo:** stores only the changed rectangle, up to about 450 MB per page.
- **Saved data:** palettes and recent colors live in `localStorage` under the keys `recolor.palette`, `recolor.palettes` and `recolor.recent`.
