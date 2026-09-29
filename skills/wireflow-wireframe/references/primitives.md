# Wireflow — the 66 primitives (exact markup)

**This is the single source of truth for primitive-to-markup mapping.** Both `/wireflow-wireframe` (building fresh) and `/wireflow-update` (resyncing existing HTML to a changed Figma frame) read this same file — never let a paraphrased or partially-copied version of this table exist anywhere else. If a primitive's markup ever needs to change, change it here once.

Copy these patterns, don't paraphrase them. Each category below is the full blended, alphabetized set — the shadcn/ui-parity extension's members sit inline with the original 50, marked **(ext)** on first mention only as a note for this table; there is no separate section for them anywhere else (not in Figma's page list, not in the catalog, not in `wireflow.css`'s category structure as seen by anyone but the stylesheet's own internal comments).

**Layout & structure (9)**
- Container — `<div class="wf-container">...</div>` (centered max-width column, horizontal padding only)
- Divider — `<hr class="wf-divider">`
- Grid — `<div class="wf-grid"><div class="wf-grid-cell">...</div>...</div>`
- Page/Frame — `<div class="wf-page">...</div>`
- Row — `<div class="wf-row">...</div>` (horizontal rhythm)
- Scroll container (ext) — `<div class="wf-scroll">…long content…</div>` (fixed height, scrolls)
- Section — `<section class="wf-section">...</section>` (full-width band, vertical padding only)
- Spacer — `<div class="wf-spacer" data-size="5"></div>` (data-size: 1–8, default 5; Figma side is a variant set, see "Variants" in the wireflow-wireframe SKILL.md)
- Stack — `<div class="wf-stack">...</div>` (vertical rhythm)

**Navigation (8)**
- Breadcrumb — `<nav class="wf-breadcrumb"><a>Home</a><svg class="wf-icon">…chevron-right…</svg><span data-current="true">Current</span></nav>` (real chevron separator, not a "/" character)
- Dropdown/submenu — `<div class="wf-dropdown"><a class="wf-nav-item">…</a>…</div>` (radius-30)
- Footer — `<footer class="wf-footer"><div class="wf-row wf-footer__columns"><nav class="wf-stack"><h3>…</h3><a class="wf-footer__link">…</a>…</nav></div><hr class="wf-divider"><div class="wf-row"><small>© …</small><div class="wf-row"><svg class="wf-icon">…globe…</svg><svg class="wf-icon">…mail…</svg><svg class="wf-icon">…message-circle…</svg></div></div></footer>` (container stays flat — only the social-icon row changed from gray boxes to real icons)
- Nav/menu item — `<a class="wf-nav-item">Menu item</a>` and `<a class="wf-nav-item" data-active="true">Active item</a>` (reused inside Navbar/Sidebar/Dropdown; active state is radius-30; Figma side is a variant set)
- Navbar — `<nav class="wf-navbar"><div class="wf-navbar__logo"></div><div class="wf-row">…nav items…</div><button class="wf-button">…</button><button class="wf-navbar__menu-toggle" aria-label="Menu"><svg class="wf-icon">…menu…</svg></button></nav>` (the menu-toggle button is always present in the markup — it's a static hamburger icon, hidden above the mobile breakpoint, that stands in for the nav-links row and CTA at mobile; no JS, matching the kit's "one static visual state" precedent)
- Pagination — `<nav class="wf-pagination"><button class="wf-page-number" aria-label="Previous"><svg class="wf-icon">…chevron-left…</svg></button><a class="wf-page-number">1</a><a class="wf-page-number" data-active="true">2</a>…<button class="wf-page-number" aria-label="Next"><svg class="wf-icon">…chevron-right…</svg></button></nav>` (chips are radius-30; real Previous/Next chevron controls flank the numbers)
- Sidebar — `<nav class="wf-sidebar"><a class="wf-nav-item">…</a><a class="wf-nav-item" data-active="true">…</a>…</nav>`
- Tab bar — `<div class="wf-tab-bar"><button class="wf-tab" data-active="true">…</button><button class="wf-tab">…</button></div>`

**Typography (6)**
- Blockquote — `<blockquote class="wf-blockquote"><p>"…"</p><cite>— Attribution</cite></blockquote>`
- Body text — `<p class="wf-body">…</p>`
- Eyebrow/label — `<p class="wf-eyebrow">Eyebrow label</p>`
- Heading — `<h2 class="wf-heading" data-level="2">…</h2>` (data-level 1–6, independent of the semantic tag)
- Link text — `<a class="wf-link" href="#">Link text</a>` (the ONLY place blue appears)
- Placeholder text block — `<p class="wf-placeholder-text">Lorem ipsum…</p>`

**Actions (5)**
- Button — `<button class="wf-button" data-variant="primary">Button</button>` (variant: primary | secondary; radius-30)
- Button group — `<div class="wf-button-group"><button class="wf-button wf-button-group__item" data-variant="secondary">One</button>…</div>` (seamless segmented control — every child button also gets the `wf-button-group__item` class)
- Icon button — `<button class="wf-icon-button"><svg class="wf-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"/><path d="M12 5v14"/></svg></button>` (radius-30; holds a real icon — swappable via the `"Icon"` instance-swap property on the Figma side; default is a plus glyph)
- Link-styled button — `<button class="wf-link-button">Link-styled button</button>` (an action, not navigation — don't use Link text for this)
- Toggle group (ext) — `<div class="wf-toggle-group"><button data-active="true">Left</button><button>Center</button><button>Right</button></div>` (segmented, multi-option — distinct from Toggle/switch's single on/off)

**Media & branding (8)**
- Aspect ratio box (ext) — `<div class="wf-aspect-ratio" data-ratio="16-9"><div class="wf-image-placeholder" style="width:100%;height:100%">Image</div></div>` (data-ratio: 16-9 | 4-3 | 1-1; Figma side is a variant set)
- Avatar — `<span class="wf-avatar">AB</span>` (circular, `border-radius: 50%` exception)
- Carousel (ext) — `<div class="wf-carousel"><div class="wf-carousel__viewport"><div class="wf-carousel__slide">…</div></div><button class="wf-carousel__control" data-direction="prev">…</button><button class="wf-carousel__control" data-direction="next">…</button></div>`
- Icon placeholder — `<span class="wf-icon-placeholder"><svg class="wf-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><path d="M12 16v-4"/><path d="M12 8h.01"/></svg></span>` (outer box stays flat/radius-10; holds a real icon — swappable via the `"Icon"` instance-swap property on the Figma side; default is an info glyph)
- Image placeholder — `<div class="wf-image-placeholder">Image</div>`
- Logo placeholder — `<div class="wf-logo-placeholder">Logo</div>` (radius-40)
- Map marker (ext) — `<span class="wf-marker"><svg class="wf-icon">…map-pin…</svg></span>`
- Video placeholder — `<div class="wf-video-placeholder"><svg class="wf-icon" data-size="md" ...>…play…</svg>Video</div>`

**Forms (13)**
- Attachment chip (ext) — `<div class="wf-attachment"><svg class="wf-icon">…paperclip…</svg><span class="wf-attachment__name">file.pdf</span><button class="wf-attachment__remove"><svg class="wf-icon">…×…</svg></button></div>`
- Calendar (ext) — `<div class="wf-calendar"><div class="wf-calendar__header"><button><svg class="wf-icon">…prev…</svg></button><span class="wf-calendar__label">September 2026</span><button><svg class="wf-icon">…next…</svg></button></div><div class="wf-calendar__weekdays">…</div><div class="wf-calendar__grid"><span class="wf-calendar__day" data-muted="true">31</span>…<span class="wf-calendar__day" data-selected="true">15</span>…</div></div>` (full month grid — every visible day 1–30 plus muted leading/trailing days from the adjacent months, not a partial/truncated grid; exactly one day shown selected, matching the documented example)
- Checkbox — `<label class="wf-checkbox"><input type="checkbox">Checkbox label</label>` (radius-20)
- Date field (ext) — `<div class="wf-date-field"><input type="text" placeholder="MM/DD/YYYY"><svg class="wf-icon">…calendar…</svg></div>`
- Form group — `<div class="wf-form-group"><label>Label</label><input class="wf-input"><small>Helper text</small></div>` (error state: `<small data-variant="error">This field is required.</small>`; Figma side is a variant set)
- OTP input (ext) — `<div class="wf-otp"><input class="wf-otp__slot" maxlength="1">…</div>`
- Radio — `<label class="wf-radio"><input type="radio" name="…">Radio label</label>`
- Search bar — `<div class="wf-search-bar"><svg class="wf-icon" ...>…</svg><input type="search" placeholder="Search"></div>` (radius-30)
- Select — `<div class="wf-select"><select><option>Select an option</option></select><svg class="wf-icon">…chevron-down…</svg></div>` (radius-30; real chevron, swappable via the `"Icon"` instance-swap property — not a CSS-drawn triangle)
- Slider (ext) — `<div class="wf-slider"><div class="wf-slider__track"><div class="wf-slider__fill" style="width:62%"></div><span class="wf-slider__thumb" style="left:62%"></span></div></div>` (pill exception; track/fill/thumb are CSS `position:relative`/`absolute` — stays a plain Figma frame, not auto-layout)
- Text input — `<input class="wf-input" type="text" placeholder="Placeholder">` (radius-30)
- Textarea — `<textarea class="wf-textarea" placeholder="Placeholder"></textarea>` (radius-30)
- Toggle/switch — `<label class="wf-toggle" data-checked="true"><span class="wf-toggle__track"><span class="wf-toggle__knob"></span></span>Toggle label</label>` (pill exception; Figma side is a variant set — the track/knob stay a plain absolutely-positioned frame inside each variant, same reasoning as Slider)

**Data & content display (9)**
- Accordion item — `<div class="wf-accordion-item"><div class="wf-accordion-item__header"><h4 class="wf-heading" data-level="4">…</h4><svg class="wf-icon" ...>…chevron…</svg></div><hr class="wf-divider"><p class="wf-accordion-item__body">…</p></div>` (radius-40; renders open/static — no JS in v1)
- Badge/tag — `<span class="wf-badge">Badge</span>` (radius-30, matching Button — not a smaller radius)
- Card — `<div class="wf-card"><div class="wf-image-placeholder">…</div><h3 class="wf-heading" data-level="3">Card title</h3><p class="wf-body">…</p><button class="wf-button">Learn more</button></div>` (radius-40)
- Chat bubble (ext) — `<div class="wf-bubble" data-role="assistant">…</div>` (data-role: assistant | user)
- Collapsible section (ext) — `<div class="wf-collapsible"><div class="wf-collapsible__trigger"><span>…</span><svg class="wf-icon">…chevron…</svg></div><div class="wf-collapsible__content">…</div></div>` (renders open/static)
- List item — `<div class="wf-list-item"><div class="wf-row"><svg class="wf-list-item__icon wf-icon" data-size="lg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><path d="M12 16v-4"/><path d="M12 8h.01"/></svg><span>Label</span></div><span>Detail</span></div>` (icon slot holds a real icon — swappable via the `"Icon"` instance-swap property on the Figma side)
- Message row (ext) — `<div class="wf-message" data-role="assistant"><span class="wf-avatar">…</span><div class="wf-message__content"><div class="wf-bubble">…</div><span class="wf-message__time">…</span></div></div>`
- Stat/metric block — `<div class="wf-stat"><strong>1,234</strong><span>Metric label</span></div>`
- Table — `<table class="wf-table"><thead><tr><th>Name</th>…</tr></thead><tbody><tr><td>Item</td>…</tr></tbody></table>` (sortable header upgrade: `<th data-sortable="true">Name <svg class="wf-icon">…chevron-down…</svg></th>`; a native table grid — stays a plain Figma frame per row/cell, not auto-layout)

**Feedback & overlays (8)**
- Alert/banner — `<div class="wf-alert"><svg class="wf-icon" data-size="md" ...>…</svg><div class="wf-alert__content"><strong class="wf-alert__title">Heads up!</strong><span class="wf-alert__description">Message.</span></div></div>` (icon + title + description anatomy, matching real shadcn Alert; radius-40; urgency via icon/title wording, e.g. "Error" — never via color)
- Empty state (ext) — `<div class="wf-empty"><span class="wf-icon-placeholder"><svg class="wf-icon">…</svg></span><h3 class="wf-empty__title">No results</h3><p class="wf-empty__body">…</p><button class="wf-button" data-variant="primary">Reset filters</button></div>`
- Modal/dialog shell — `<div class="wf-modal"><div class="wf-modal__header"><h2 class="wf-heading" data-level="3">Title</h2><button><svg class="wf-icon">…×…</svg></button></div><hr class="wf-divider"><p class="wf-modal__body">…</p><div class="wf-modal__footer"><button class="wf-button" data-variant="secondary">Cancel</button><button class="wf-button" data-variant="primary">Confirm</button></div></div>` (radius-40; Sheet/Drawer upgrade: add `data-position="right"|"left"|"bottom"` to slide from an edge instead of centering — those edge variants stay flat on purpose)
- Progress bar — `<div class="wf-progress"><div class="wf-progress__fill" style="width:62%"></div></div>` (pill exception; no flex container in the CSS at all — stays a plain Figma frame)
- Skeleton loader (ext) — `<span class="wf-skeleton" data-shape="text"></span>` (data-shape: text | circle; each shape is a single leaf shimmer shape with nothing to lay out)
- Spinner (ext) — `<svg class="wf-spinner" viewBox="0 0 24 24">…</svg>` (its own rotating class, not `.wf-icon`; the component is just the icon instance filling the frame — nothing to lay out)
- Toast/snackbar — `<div class="wf-toast"><span>Message.</span><button><svg class="wf-icon">…×…</svg></button></div>` (radius-40)
- Tooltip — `<div class="wf-tooltip"><span class="wf-tooltip__bubble">Tooltip text</span><span class="wf-tooltip__pointer"></span></div>` (simple bubble: radius-20; rich variant/Hover Card upgrade: radius-40, and its bubble is a vertical auto-layout stack of a heading + body paragraph with a real gap token between them — never two text nodes placed side by side; Figma side is a variant set)
