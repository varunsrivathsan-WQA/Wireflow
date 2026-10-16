# Recolor script and role map

All snippets run inside `use_figma`. Read-only first, then write. Keep each call to one section of the frame.

## Role map

Pick one Wireflow COLOR variable per role by reading the file's variables. Typical roles (names will differ; resolve by what exists, then confirm with the user):

| Role | Used for |
|---|---|
| surface | page and card backgrounds (lightest) |
| surfaceAlt | subtle section and input backgrounds |
| surfaceMuted | mid-gray fills, disabled, tags |
| border | default strokes and dividers |
| borderStrong | emphasized strokes |
| textPrimary | body and headings |
| textSecondary | secondary and helper text |
| textOnDark | text on filled dark surfaces |
| action | filled primary buttons and dark fills |
| link | link text and link-colored actions |
| imagePlaceholder | replaces photos, gradients, video |

## Step A: resolve roles to variables (read-only)

```js
// Fill in the NAMES after reading the variables. Do not guess.
const ROLE_NAMES = {
  surface: 'REPLACE', surfaceAlt: 'REPLACE', surfaceMuted: 'REPLACE',
  border: 'REPLACE', borderStrong: 'REPLACE',
  textPrimary: 'REPLACE', textSecondary: 'REPLACE', textOnDark: 'REPLACE',
  action: 'REPLACE', link: 'REPLACE', imagePlaceholder: 'REPLACE',
};
const vars = await figma.variables.getLocalVariablesAsync('COLOR');
const byName = new Map(vars.map(v => [v.name, v]));
const missing = [];
const roles = {};
for (const [role, name] of Object.entries(ROLE_NAMES)) {
  const v = byName.get(name);
  if (!v) missing.push(role + ' -> ' + name); else roles[role] = v.id;
}
return { missing, roles, available: vars.map(v => v.name) };
```

## Step B: recolor (write), run per batch

Paste the resolved `roles` ids into `ROLE_IDS`. `FRAME_ID` is the target frame, `SECTION_IDS` limits a batch (or leave empty for the whole frame on small pages).

```js
const FRAME_ID = 'REPLACE';
const SECTION_IDS = []; // optional: node ids of sections to process this call
const ROLE_IDS = {}; // paste from Step A
const OPTS = { images: true, shadows: true };

const frame = await figma.getNodeByIdAsync(FRAME_ID);
if (!frame) throw new Error('frame not found');
// Guard: never edit a main component
let p = frame;
while (p) { if (p.type === 'COMPONENT' || p.type === 'COMPONENT_SET') throw new Error('target is inside a component'); p = p.parent; }

const roleVars = {};
for (const [r, id] of Object.entries(ROLE_IDS)) roleVars[r] = await figma.variables.getVariableByIdAsync(id);

function lum(c) { return 0.2126 * c.r + 0.7152 * c.g + 0.0722 * c.b; }
function sat(c) { const mx = Math.max(c.r, c.g, c.b), mn = Math.min(c.r, c.g, c.b); return mx === 0 ? 0 : (mx - mn) / mx; }

async function pureName(paint) {
  const a = paint.boundVariables && paint.boundVariables.color;
  if (!a) return '';
  try { const v = await figma.variables.getVariableByIdAsync(a.id); return v ? v.name.toLowerCase() : ''; } catch (e) { return ''; }
}

// ctx: 'fill' | 'stroke' | 'text'
function classify(c, name, ctx) {
  if (ctx === 'text') {
    if (/link|anchor/.test(name)) return 'link';
    if (/inverse|on-dark|on-primary|on-action/.test(name)) return 'textOnDark';
    if (/muted|secondary|subtle|helper|placeholder/.test(name)) return 'textSecondary';
    const L = lum(c);
    if (L > 0.85) return 'textOnDark';
    if (L < 0.3) return 'textPrimary';
    return 'textSecondary';
  }
  if (ctx === 'stroke') {
    if (/strong|emphasis|focus/.test(name)) return 'borderStrong';
    return lum(c) < 0.35 ? 'borderStrong' : 'border';
  }
  if (/surface|background|bg|canvas/.test(name) && lum(c) > 0.85) return lum(c) > 0.95 ? 'surface' : 'surfaceAlt';
  if (/primary|brand|action|accent|cta/.test(name)) return 'action';
  const L = lum(c), S = sat(c);
  if (L > 0.95) return 'surface';
  if (L > 0.85) return 'surfaceAlt';
  if (S > 0.25 && L < 0.8) return 'action';
  if (L < 0.3) return 'action';
  return 'surfaceMuted';
}

async function remapPaint(paint, ctx, counts) {
  if (paint.type === 'IMAGE' || paint.type === 'VIDEO' || paint.type.startsWith('GRADIENT')) {
    if (!OPTS.images) return paint;
    counts.replaced++;
    return figma.variables.setBoundVariableForPaint({ type: 'SOLID', color: { r: 0.85, g: 0.85, b: 0.85 } }, 'color', roleVars.imagePlaceholder);
  }
  if (paint.type !== 'SOLID') return paint;
  const name = await pureName(paint);
  const role = classify(paint.color, name, ctx);
  const v = roleVars[role];
  if (!v) { counts.unmapped.push(role); return paint; }
  counts.mapped++;
  const np = figma.variables.setBoundVariableForPaint({ type: 'SOLID', color: paint.color, opacity: paint.opacity }, 'color', v);
  return np;
}

const roots = SECTION_IDS.length ? await Promise.all(SECTION_IDS.map(id => figma.getNodeByIdAsync(id))) : [frame];
const counts = { mapped: 0, replaced: 0, unmapped: [], skipped: 0, shadowsRemoved: 0, nodes: 0 };

for (const root of roots) {
  const nodes = [root, ...root.findAll(() => true)];
  for (const n of nodes) {
    if (n.removed || n.visible === false) { counts.skipped++; continue; }
    counts.nodes++;
    if ('fills' in n && n.fills !== figma.mixed && Array.isArray(n.fills) && n.fills.length) {
      if (n.fillStyleId) await n.setFillStyleIdAsync('');
      const ctx = n.type === 'TEXT' ? 'text' : 'fill';
      n.fills = await Promise.all(n.fills.map(f => remapPaint(f, ctx, counts)));
    } else if (n.type === 'TEXT' && n.fills === figma.mixed) {
      const segs = n.getStyledTextSegments(['fills']);
      for (const s of segs) {
        const nf = await Promise.all(s.fills.map(f => remapPaint(f, 'text', counts)));
        n.setRangeFills(s.start, s.end, nf);
      }
    }
    if ('strokes' in n && Array.isArray(n.strokes) && n.strokes.length) {
      if (n.strokeStyleId) await n.setStrokeStyleIdAsync('');
      n.strokes = await Promise.all(n.strokes.map(s => remapPaint(s, 'stroke', counts)));
    }
    if (OPTS.shadows && 'effects' in n && n.effects.some(e => e.type === 'DROP_SHADOW' || e.type === 'INNER_SHADOW')) {
      if (n.effectStyleId) await n.setEffectStyleIdAsync('');
      n.effects = n.effects.filter(e => e.type !== 'DROP_SHADOW' && e.type !== 'INNER_SHADOW');
      counts.shadowsRemoved++;
    }
  }
}
return counts;
```

Notes:
- Text nodes may need their fonts loaded before `setRangeFills`. If it throws about fonts, call `await figma.loadFontAsync(f)` for each of `n.getRangeAllFontNames(0, n.characters.length)` first.
- Rebinding a paint inside an instance creates an override on that instance only. The Pure main component is untouched.
- Do not call `detachInstance`.

## Step C: audit (read-only)

Lists paints in the frame that are not bound to a color variable. The goal is an empty list.

```js
const frame = await figma.getNodeByIdAsync('REPLACE');
const bad = [];
for (const n of [frame, ...frame.findAll(() => true)]) {
  if (n.visible === false) continue;
  for (const key of ['fills', 'strokes']) {
    if (!(key in n) || !Array.isArray(n[key])) continue;
    for (const p of n[key]) {
      if (p.visible === false) continue;
      const bound = p.boundVariables && p.boundVariables.color;
      if (!bound) bad.push({ id: n.id, name: n.name, kind: key, type: p.type });
    }
  }
}
return { count: bad.length, sample: bad.slice(0, 40) };
```

Also count instances before and after (`frame.findAll(n => n.type === 'INSTANCE').length`) to prove nothing was detached.
