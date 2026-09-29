---
name: wireflow-update
description: "Resync a wireframe's HTML file with its Figma frame's CURRENT state after the Figma design has changed since the last build — detects what changed (primitives added, removed, reordered, or edited) and patches only those parts of the HTML, leaving everything else in the file untouched. Use when the user says the Figma wireframe changed and the HTML needs to catch up, or asks to 'resync', 'update the HTML to match Figma', 'refresh the wireframe', or similar. Companion to /wireflow-wireframe, which builds a wireframe fresh; this skill never builds from scratch and never writes back to Figma."
---

# Wireflow update — resync HTML with a changed Figma frame

**Direction of truth: Figma → HTML, one way only.** This skill exists for the moment after `/wireflow-wireframe` already built a matching Figma frame + HTML pair, and the Figma frame was then edited by hand (a primitive added, removed, reordered, or its content/variant changed) — the HTML file is now stale, and this skill's job is to bring it back into agreement with Figma. It never edits the Figma frame, never rebuilds a frame from scratch, and never invents new content beyond what Step 3 of `/wireflow-wireframe` already documents.

**This only works on an HTML file `/wireflow-wireframe` actually built** — specifically, one that still carries the build-source marker that skill's Step 5 embeds. If the file has no marker, or the marker's frame no longer resolves, stop and say so (Step 0) rather than guessing.

## Step 0 — find the file's Figma source, or stop

1. Get the HTML file to resync. If several are in play (a multi-screen flow), resync one file at a time — each screen's marker names its own frame(s), and a change to one screen's frame says nothing about a sibling screen's.
2. Read the file and find its build-source marker — the HTML comment `<!-- wireflow-source: fileKey=... frames="..." built=... -->` that `/wireflow-wireframe` Step 5 embeds right after `<meta charset>` in `<head>`. Parse `fileKey` and the `frames` list — a `;`-separated set of `tier=node-id` pairs (e.g. `desktop=123:45;tablet=123:67;mobile=123:89`), one entry per breakpoint that screen was actually built with. A marker from before the breakpoints feature may instead read `node-id=...` with no tier name — treat that bare id as `desktop=<that id>` and proceed the same way.
3. **Decide which frame(s) this resync is about.** If the marker lists only one tier, use it — nothing else to decide, same as before breakpoints existed. If it lists more than one tier:
   - If the user's request names a specific breakpoint ("the mobile frame changed," "resync the tablet version"), use that tier's frame only.
   - Otherwise, **default to the desktop tier's frame** (or, if desktop wasn't built for this screen, the widest tier that was) — this is the same frame `/wireflow-wireframe`'s Figma/HTML parity check already treats as the HTML's structural reference, so it's the correct default source of truth for "does the HTML's actual content match Figma" questions. Say which frame you're resyncing from in the final report (Step 5) so the user can correct you if they meant a different tier.
4. **If there's no marker at all:** this file either wasn't built by `/wireflow-wireframe`, or the marker was stripped out since. Tell the user plainly and offer two ways forward — they give you the Figma frame URL directly this one time (fine for a single resync, but the marker won't be there next time either), or they rebuild the screen fresh via `/wireflow-wireframe` so future resyncs work normally. Don't guess a frame from the filename or the HTML's own content.
5. **If the chosen tier's frame doesn't resolve** (`await figma.getNodeByIdAsync(nodeId)` returns `null` — load the `figma-use` skill first, mandatory, exactly as `/wireflow-wireframe` requires for any `use_figma` call): the frame may have been deleted, or duplicated into a new node with a different id. Tell the user what you found and ask how to proceed rather than assuming a same-named frame elsewhere in the file is the right replacement.

## Step 1 — snapshot both sides as ordered, nested primitive lists

This reuses the exact extraction method `/wireflow-wireframe`'s "Figma/HTML parity check" (its Step 4) already defines — read that section there before doing this if it's not already familiar; don't reinvent a different extraction approach.

1. **Figma side (current state):** in the same `use_figma` call, walk the frame's node tree (`frame.findAllWithCriteria({types: ['INSTANCE']})` or `frame.query('INSTANCE')`) in document order, mapping each instance to its `mainComponent`'s (or, for a variant instance, its parent set's) name, its resolved variant/content component properties, and the direct text content of any child `TEXT` node. Capture nesting/parent chain per entry, not just a flat list. **If the chosen frame is the tablet or mobile tier** (per Step 0.3), first strip out anything that's a documented reflow difference rather than a real content change, per `/wireflow-wireframe`'s "Responsive breakpoints" section — a mobile Navbar's missing nav-links Row and added menu-toggle icon, a Sidebar entirely absent from a mobile frame that also has a Navbar, and similar — before comparing against the HTML. Those are expected, correct differences baked into that tier's frame on purpose; treating them as "removed"/"added" primitives would patch the shared HTML into matching one breakpoint's reflowed layout instead of its real, tier-agnostic content.
2. **HTML side (as currently written on disk):** parse the file's actual DOM structure — a real parser (e.g. a short script using an HTML parser, or an equally careful structural read), never a naive regex over raw text, since nesting and attributes need to survive intact. Build the same shape: an ordered, nested list of `{ primitive, dataAttributes, textContent }`, mapping each top-level `.wf-*` class back to its primitive name via `../wireflow-wireframe/references/primitives.md` (the shared vocabulary table — read it fresh each run rather than assuming a name mapping from memory, since it's the same file `/wireflow-wireframe` maintains).

## Step 2 — align the two lists and classify every position

Figma is the reference sequence here — unlike the neutral "these two should already agree, flag if not" framing of the parity check, this skill exists specifically for when they don't, and Figma is which side moved.

Walk both lists with a sequence-alignment approach tolerant of insertions, deletions, and reordering — conceptually the same idea as a text-line diff (`diff`/`git diff`), applied to the list of primitive entries instead of lines, matching primarily on primitive identity + relative position, then on nesting/parent chain when a primitive name repeats (e.g. picking which of three `Card` entries lines up with which). Classify every position as one of:

- **Unchanged** — same primitive, same nesting, same text/attributes on both sides. Skip it.
- **Changed in place** — same primitive, same position, but different text content and/or different `data-*`/variant attributes on the Figma side now. Patch only that.
- **Added** — present in the current Figma list, absent from the HTML list. A new primitive to insert.
- **Removed** — present in the HTML list, absent from the current Figma list. An existing block to delete.
- **Reordered** — the same set of entries in a different relative sequence. Blocks to move, not delete-and-recreate.

## Step 3 — patch the HTML file directly, touching only matched primitive elements

This is a **targeted patch**, not a regeneration — the whole reason this skill exists separately from just re-running `/wireflow-wireframe` is to preserve anything in the file that isn't itself a recognized primitive element: custom comments, extra wrapper markup, or any attribute a person added by hand after the original build. Only touch what Step 2 classified as changed, added, removed, or reordered:

1. **Changed in place:** edit that element's text content and/or `data-*`/variant attributes only. Leave its surrounding markup, class list, and any wrapping structure untouched.
2. **Added:** build the new element's markup from `../wireflow-wireframe/references/primitives.md` — the exact pattern for that primitive, never paraphrased — and insert it at the position implied by its neighbors in the current Figma order. If the added primitive is an icon-swap slot, a variant set, or otherwise needs a rule beyond the bare markup pattern, check the relevant section of `../wireflow-wireframe/SKILL.md` ("Icons," "Variants," "Radius") rather than guessing.
3. **Removed:** delete only that element's own markup block. Don't cascade-delete a parent container that still holds other real content just because one child inside it was removed.
4. **Reordered:** move the existing block to its new position rather than deleting and recreating it — recreating would lose any manual attribute added to it since the original build.
5. **Never touch anything that isn't a recognized `.wf-*` primitive element** from the vocabulary table — this is the boundary that makes this a *targeted* patch rather than a rebuild. A hand-added comment, an extra wrapper `<div>`, an attribute the user added directly in the HTML: all left exactly as found, even if nothing on the Figma side explains them.
6. **Update the build-source marker** after patching: append a `synced=<ISO date>` field alongside the original `built=` field (don't remove or overwrite `built=`), so the file visibly shows it's been kept in sync and when. If Step 0 found the resolved frame's node-id had changed (e.g. the frame was duplicated rather than edited in place), update that one tier's entry in the `frames` list to the new id — leave every other listed tier's entry untouched, since this run only touched one tier's frame.

## Step 4 — re-run the Figma/HTML parity check

After patching, run `/wireflow-wireframe`'s Step 4 "Figma/HTML parity check" against the now-patched file, exactly as a fresh build does before being reported done. It should now report a full match. If it doesn't, fix the remaining gap before reporting — same bar as a first build, not a lower one because this was "just an update."

## Step 5 — report back

Tell the user specifically what changed — which primitives were added, removed, or reordered, and which had text/variant edits — not just "updated the HTML." Confirm the parity check passed. **State which breakpoint's frame this resync used** (per Step 0.3) whenever the file's marker lists more than one — so the user can say "actually, resync from the mobile frame instead" if the default guess wasn't what they meant. If the frame is part of a multi-screen flow and its navigation/prototype links also changed on the Figma side, call that out too, but remember this run only touched the one HTML file named in Step 0 — resync each affected screen separately rather than assuming one screen's changes.
