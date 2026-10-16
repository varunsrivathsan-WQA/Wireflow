---
name: wireflow-pure-makeover
description: Builds a wireframe from real doTERRA Pure design-system components in Figma, then recolors every Pure fill, stroke and text color to Wireflow's grayscale tokens using instance-level overrides, without editing the Pure library. Use when asked to build a wireframe with Pure components in Wireflow colors, give Pure components a Wireflow or black-and-white makeover, or recolor a Pure-based frame to Wireflow tokens.
---

# Wireflow pure makeover

Build with real Pure components, then re-point their colors to Wireflow's variables so the result reads as a wireframe. Figma only. Pure's library is never edited: the recolor happens as overrides on instances inside the user's own file.

What this keeps: Pure's layout, spacing, radii, component structure. What it changes: color (and optionally images, shadows and font). Say this once to the user, because a recolored Pure page still looks more finished than a Wireflow page.

## Requirements

- Figma MCP connected. Load the `figma-use` skill before any `use_figma` call. Run `use_figma` calls one at a time.
- The target Figma file has the Pure libraries enabled (Base, and Ecommerce / Iconography if used). Check with `get_libraries(fileKey)`. A library listed under available-to-add must be added by the user in the Assets panel; you cannot subscribe it.
- Wireflow's color variables must be reachable from the target file, either as local variables or through the Wireflow UI Kit library (check `get_libraries`). Read local collections first. If there are none, search the library with `search_design_system` (entity `variable`) and import them with `figma.variables.importVariableByKeyAsync`; binding to imported library variables works. Only if neither exists, stop and point the user to the `wireflow` token setup or the `build-tokens` skill; do not invent colors.
- Pure MCP connected is optional but useful for choosing components (`suggest_composition`, `recommend_components`, `search_components`). If the page was already built with Pure, skip building.

## Hard rules

- Never edit a main component, variable, style or collection belonging to Pure. Refuse if the target file is the Pure library file itself.
- Never detach instances. Detaching breaks the link to Pure and is the opposite of the point.
- Only override layers inside the target frame. Do not touch anything outside it.
- Bind to Wireflow variables, never to raw hex values. A leftover hex fill is a defect.
- Do not add Pure components or tokens into the Wireflow library or file's local variables.
- Re-read node positions and sizes fresh on every call. The user may be editing the file at the same time; never rely on coordinates from an earlier call, and never move a frame the user placed.

## Step 0: ask (AskUserQuestion, one round)

1. Figma file link and the frame to build in or recolor (a new frame, or an existing Pure-based frame).
2. If building: the page type or a brief, or a Wireflow wireframe to follow (offer the `wireflow-to-pure` skill's mapping as input).
3. Options: images to gray placeholders (default yes), shadows removed (default yes, wireframes are flat), font left as Pure's or changed to Wireflow's system font (default leave).

## Step 1: preflight

1. `get_libraries` on the file. Confirm the Pure libraries are in `libraries_added_to_file`. If not, tell the user exactly which to add and wait.
2. Read the Wireflow color variables with `use_figma` (read-only): collections, modes, variable names, code syntax, resolved values. Names may have drifted, so never assume a name; build the role map from what is actually there.
3. Build the role map (see `references/recolor-script.md`, "Role map"): pick one Wireflow variable for each role, and show the user the role-to-variable table before proceeding. If a role has no obvious variable, ask.

## Step 2: build (skip if the frame already exists)

Find components with `search_design_system` scoped to the Pure library keys and import them with `figma.importComponentByKeyAsync`. Compose the page from Pure blocks, not from re-built parts. Follow Pure's rules (call `get_design_rules` if the Pure MCP is connected): Grid for page columns, Header/Footer variants matching the chosen theme, no Gallery on a standard product page. Use real component variants and set text through component properties where they exist.

## Step 3: test on one component first

Do not recolor the whole frame first. Pick two or three representative instances (for example the header and one product card), run the recolor on only those, screenshot before and after (`get_screenshot`), and show the user. Calibrate the classifier from what is wrong (a brand-colored button that went too light, text that lost contrast) and get a yes before continuing.

## Step 4: recolor the frame

Before recoloring, apply the component fixes in `references/recolor-script.md` ("Pure component quirks") to TextInput, Select and Link instances. Run the recolor script from `references/recolor-script.md` over the frame in batches (one section or about 150 nodes per call, because calls have a time limit). Order of work per call: collect paint nodes, clear style bindings that would block overrides, rebind fills, strokes and text fills, replace image and gradient fills, strip shadows if chosen. Record counts of mapped, unmapped and skipped paints.

## Step 5: verify

0. If the frame was copied at any point, recount its instances in a separate call afterwards; the first count after a copy can come back low.
1. Run the audit snippet (same reference file): list every paint in the frame that is not bound to a Wireflow variable. The target is zero. Anything left is reported with its node name, not hidden.
2. Screenshot each section and compare against the original. Look for lost contrast (text on filled buttons), invisible borders, and any color still visible that is not grayscale or the link blue.
3. Confirm nothing outside the frame changed and no instance was detached (count of instances before and after is equal).

## Step 6: report

Short summary: what was built, how many paints were rebound, what was left unmapped and why, and the known limits: overrides can reset when a variant is swapped (rerun the recolor), a Pure library update can introduce new colors (rerun), layout, spacing and radii are still Pure's, and CheckoutOption and PaymentMethods don't reflow at 375px wide (flag this when the frame is a mobile checkout). Offer a rerun command phrase ("rerun the Pure makeover on this frame").

Nothing here is verified against every Pure component. Treat the first run on a new component family as a test, and say so when something looks wrong.

## Step 7 (optional): dark copy

Once every color in the frame is bound to a Wireflow token, offer a dark version: clone the frame beside the original and switch the clone to the Dark mode of the Wireflow color collection (for example "Blankslate Tokens"). No recoloring is needed. See "Dark copy" in `references/recolor-script.md`. Check the result visually; anything still bound to a non-token color stays light and shows up in the audit.
