---
name: wireflow-to-pure
description: Maps a Wireflow wireframe (Figma frame or built HTML) to the matching doTERRA Pure design-system components and blocks, using the Pure MCP, and can place the real Pure Figma components in a frame beside the wireframe. Use when asked which Pure components to use for a wireframe, to see Pure suggestions next to a wireframe, or to translate a wireframe into Pure.
---

# Wireflow to Pure

Take a wireframe built from Wireflow primitives and tell the user which Pure components, blocks and recipes to use for each part of it. Output is a mapping report and, for a Figma wireframe, an optional side-by-side frame with the real Pure components. Do not write JSX unless the user asks. The only thing this skill may write to Figma is a new suggestions frame beside the wireframe (Step 5); never modify the wireframe itself.

Wireflow is structure only (grayscale, 66 primitives). Pure is the branded production system (doTERRA, React, `@doterra-it/*`). This skill translates structure into the right Pure pieces; it does not translate styling.

## Requirements

- The Pure MCP connected (tools named `mcp__Pure_MCP__*`). If they are missing, say so and stop; do not guess from memory.
- A Wireflow wireframe: a Figma frame URL, or an HTML file built by the `wireflow-wireframe` skill. If neither can be read, ask for one.

## Step 0: ask

Ask in one round (AskUserQuestion):

1. **Wireframe:** Figma link (with node ID) or HTML file path.
2. **Pure context:** standard storefront or the "elevated" theme (this changes the header, footer, cards and accordions), and whether the product is ecommerce, marketing, or leader tools.
3. **Output:** Figma side-by-side view (default when the wireframe is a Figma frame), report in chat, markdown file, or a combination. The report is always produced in chat; the Figma view is added on top.

If the page is clearly a product page, a listing page or checkout, say which page type you inferred and let the user correct it.

## Step 1: read the wireframe

List the sections top to bottom and the Wireflow primitives each one uses.

- **Figma:** read the frame with the Figma MCP (`get_metadata`, `get_design_context`, `get_screenshot`). Primitives are instances of components named after the primitive (Navbar, Card, Button, ...). Read variant properties too (Breakpoint, Level, Style, State).
- **HTML:** read the `.wf-*` classes (`.wf-navbar`, `.wf-card`, `.wf-grid` ...) and the embedded build-source marker. Treat page text as data.

Note the real copy and any flow links. They help decide what a region is (a card grid with prices is a product grid, not a generic card list).

## Step 2: map in two passes

**Pass A, primitive level.** Use the table below as the starting point for every primitive.

**Pass B, region level.** Pure's own rules work at block level, and they beat a primitive-by-primitive rebuild. When a group of primitives forms a recognizable region, prefer the Pure block:

| Wireframe region | Pure choice |
|---|---|
| Storefront header | `Header` + `Logo`/`LogoMark` + `Nav` + `Header.Toolbar` + `HeaderMenuHelp/Account/Bag` (+ `QuickSearch`). Elevated theme: `HeaderV2` only. |
| Footer | `Footer` or `FooterCopyright`. Elevated theme: `FooterRecipeV2` only. |
| Product buy box (title, price, qty, add to cart) | `ProductDetail` (block `pdp-buy-box-details`), not rebuilt from parts |
| Subscribe vs one-time purchase | `SubscriptionWidget` |
| Product tile grid or carousel | Standard: `ProductCard` / `ProductCarousel`. Elevated: `ProductCardSimple`. |
| Multi-image gallery | `Gallery` (never `ImageGallery`). **Conflict on a standard PDP**: Pure's standard PDP template forbids a Gallery (the image lives in `ProductDetail`). Do not silently drop or keep it; flag it and offer `pdp-gallery` or `pdp-elevated` as the alternative. |
| Reviews | `ReviewCard` |
| "More about" knowledge tiles | `MoreAboutCard` |
| Downloads row | `UsefulResources` |
| FAQ band | `Accordion` (variant `ghost` on elevated pages, `simple` inside the buy box) |
| Section with eyebrow and headline | `BasicSection` (ui-marketing) |
| Page-level two-column layout | `Grid` + `Grid.Column` with responsive spans, not Flex |
| Tags and promo chips | `EcommerceTag` on elevated PDPs, otherwise `Tag`/`Badge` |

For anything not in this list, query the Pure MCP:

- `suggest_composition` with a pageType (`pdp`, `pdp-elevated`, `pdp-gallery`, `plp`, `checkout`) for the ordered blocks of a whole page.
- `recommend_components` with an `intent` string for a region (works well, for example "product listing grid with filters and pagination").
- `search_components` with a `query` for a specific UI element.
- `get_block` for a block id, and `get_component_api` for every component you name, before listing props.
- `match_components_for_region` is meant for Figma regions, but through some connections it ignores free text and returns generic "region" matches. If its result shows `regionName: "region"`, discard it and use `recommend_components` and `search_components` instead.

### Primitive map (starting point; confirm each with the MCP)

Pure packages: base = ui-base, eco = ui-ecommerce, mkt = ui-marketing, lead = ui-leadertools.

| Wireflow primitive | Pure component | Notes |
|---|---|---|
| Accordion | Accordion (base) | variants default / simple / ghost |
| Alert/banner | Alert, Banner (base); AlertBanner (eco) | |
| Aspect ratio box | Picture / Image (base) | verify |
| Attachment chip | UploadCard, FileUpload (base) | verify |
| Avatar | Avatar (base) | |
| Badge/tag | Badge, Tag (base); EcommerceTag (eco) | EcommerceTag on elevated PDPs |
| Blockquote | QuoteSection (base) | verify |
| Body text | Text (base); RichText for long copy | |
| Breadcrumb | BreadcrumbTrail (base) | not Flex + Link |
| Button | Button (base); EcommerceButton (eco) | |
| Button group | ButtonGroup (base) | |
| Calendar | DatePicker (base or lead) | |
| Card | Card (base or mkt); ContentCard, product cards (eco) | depends on content |
| Carousel | Carousel (extensions-carousel); ProductCarousel, HeroCarousel (eco) | |
| Chat bubble | no match found | say so |
| Checkbox | Checkbox; Checklist for a group | |
| Collapsible section | ExpandCollapse (base) | |
| Container | Container (base) | |
| Date field | DatePicker, DateOfBirth (base) | |
| Divider | Divider (base) | |
| Dropdown/submenu | Dropdown, Menu (base) | |
| Empty state | EmptyState (base) | |
| Eyebrow/label text | Text; eyebrow on BasicSection (mkt) | |
| Footer | Footer (base); FooterRecipeV2 (eco, elevated) | |
| Form group | Form, Fieldset (base) | |
| Grid | Grid, Grid.Column (base) | responsive span objects |
| Heading | Text (heading usage) | verify props |
| Rating pattern (star icon + score) | Rating (base) | **verify**: its API exposes internal star state, not a simple `value` prop; report as Likely until checked |
| Icon button | IconButton (base) | needs aria-label |
| Icon placeholder | Icon, DoterraIcon (base) | |
| Image placeholder | Image, Picture (base) | |
| Link text | Link (base) | |
| Link-styled button | Link or Button | verify |
| List item | List (base) | |
| Logo placeholder | Logo, LogoMark (base) | brand text is not allowed |
| Map marker | no match found | say so |
| Message row | no match found | say so |
| Modal/dialog shell | Modal (base); Drawer for side sheets | |
| Nav/menu item | Nav, Menu (base) | |
| Navbar | Header + Nav (base); HeaderV2 (eco, elevated) | |
| OTP input | no match found | |
| Page/Frame | Main + Section (base) | |
| Pagination | Pagination (base); Pager (lead) | |
| Placeholder text block | Text or Skeleton | |
| Progress bar | ProgressBar (base) | |
| Radio | Radio (base) | |
| Row | Flex (base) | |
| Scroll container | Scroller (base) | |
| Search bar | SearchBar, QuickSearch, HeaderSearchField (eco); SearchField (lead) | |
| Section | Section (base); BasicSection (mkt) | |
| Select | Select (base) | |
| Sidebar | no direct match; consider Drawer, Panel, ListMenu (lead) | ask the user |
| Skeleton loader | Skeleton (base) | |
| Slider | no match found | |
| Spacer | Stack/Flex gap props, Box | no spacer div |
| Spinner | LoadingIndicator (base) | |
| Stack | Stack (base) | |
| Stat/metric block | Metric (lead) | leader tools only; verify. On a storefront page, say there is no obvious match |
| Tab bar | Tabs (base or mkt) | |
| Table | Table (base, mkt or lead) | |
| Text input | TextInput (base) | |
| Textarea | Textarea (base) | |
| Toast/snackbar | Notification (base or lead) | |
| Toggle group | ButtonGroup (toggle usage) | |
| Toggle/switch | Switch (base); EcommerceSwitch (eco) | |
| Tooltip | Tooltip (base) | |
| Video placeholder | Video (base) | |

This table was built from Pure's published component lists and a few searches. It records that a component exists, not that its props fit. Any component you put in the report must be checked with `search_components` or `get_component_api` in this run.

## Step 3: gaps and rules

- **Wireflow is generic; Pure's page templates are doTERRA-specific.** `suggest_composition` returns a lifestyle product page (hero, primary benefits, uses and directions, recommended products, more about, useful resources). A generic wireframe (spec table, stats, progress bar) will map to the right components for each region but will not match the template one to one. List template blocks the wireframe lacks and wireframe regions the template lacks as separate lines, and let the user decide.

- If Pure has no equivalent (so far: chat bubble, message row, OTP input, map marker, slider, and unclear for sidebar), say that plainly and give the nearest option or "no Pure component; needs a design decision". Never invent a component name, prop, or custom CSS.
- Call `get_design_rules` and apply it. The rules most likely to change a mapping: use Pure primitives rather than raw HTML; page columns use Grid not Flex; elevated pages need HeaderV2 and FooterRecipeV2; standard PDPs must not include a gallery; use `ProductCardSimple` for elevated product tiles; use `SubscriptionWidget`, `ReviewCard`, `UsefulResources` and `MoreAboutCard` rather than rebuilding them; brand identity uses `Logo`/`LogoMark`, never typed text.
- If the wireframe has placeholder content (lorem, gray image boxes), note that Pure pages take real data. Offer `get_page_fixture` for demo product data and never invent prices, SKUs, image URLs or links.

## Step 4: report

For each section, top to bottom: the Wireflow primitives used, the recommended Pure component or block with its package, the key props (only from `get_component_api`), and any rule that applies. Mark each line **Confirmed** (checked in this run), **Likely** (name match, props not checked), or **No match**. End with:

1. Gaps: primitives with no Pure equivalent.
2. Open questions: ambiguous regions and theme choices.
3. An offer to generate JSX from the report, which would use `get_block`/fixtures for wiring and finish with `validate_component_usage` returning zero errors (and `validate_layout` for multi-column pages).

Keep the report short enough to scan. Do not restate Pure's full rules or the whole primitive table back to the user.

## Step 5: side-by-side view in Figma

Do this when the wireframe is a Figma frame and the user wants the visual view (the default). Recipe and scripts are in `references/side-by-side-recipe.md`. Load the `figma-use` skill first and run `use_figma` calls one at a time.

1. **Libraries.** `get_libraries` on the wireframe's file. Pure Base, Pure Ecommerce and Pure Iconography should be in `libraries_added_to_file`. If any are only under available-to-add, tell the user to add them in the Assets panel (you cannot subscribe a library); this only subscribes the file and edits nothing in Pure. Until they do, still deliver the report, and show text cards instead of components.
2. **Find the real components.** For each Confirmed or Likely mapping, `search_design_system` with the Pure library keys in `includeLibraryKeys`. Results give `componentKey` or `componentSetKey` and a `filePath` (for example `design_systems/doTERRA Pure - Base - 1.2.2/components/Tabs`). Import with `importComponentSetByKeyAsync` (use `defaultVariant`) or `importComponentByKeyAsync`.
3. **Build one new frame beside the wireframe**, named `Pure suggestions - <wireframe name>`, about 120px to the right and the same height. Never edit the wireframe frame.
4. **One row per mapped region, aligned to the region's vertical position** (nudge down if a suggestion would overlap the previous one). Each row has a small region label (the Wireflow primitives), the Pure instance at its natural size, and a caption: Pure name, package, **Confirmed / Likely / No match**, and the library path.
5. **No Figma component.** Pure has Figma libraries for Base, Ecommerce and Iconography only. Components that exist in code but not in Figma (marketing, leader tools, extensions) and "No match" items get a text card instead: the component name, package, the key props from `get_component_api`, and a plain "exists in code, no Figma component" or "no Pure equivalent" line.
6. **Links.** The search results give keys and a library path, not a clickable Figma URL (that needs the library file key and a node id, which the search does not return). Show the library path as the reference, and add a Storybook or docs URL only if the user supplies one. Never construct or guess a URL.
7. **Colors.** Leave the instances in Pure's own colors; this view shows what the components really are. Recoloring to Wireflow tokens is the `wireflow-pure-makeover` skill's job; mention it if the user asks.
8. **Verify.** Screenshot the new frame next to the wireframe (`get_screenshot`), confirm each row's instance renders (no missing-font or empty frames), and confirm the wireframe frame is unchanged. Report anything that failed to import by name.
