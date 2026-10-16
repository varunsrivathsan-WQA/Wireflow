# Side-by-side recipe

Run each snippet in its own `use_figma` call. Replace the REPLACE values. Do not edit the wireframe frame.

## A. Create the suggestions frame (write)

```js
const WIREFRAME_ID = 'REPLACE';
const wf = await figma.getNodeByIdAsync(WIREFRAME_ID);
if (!wf || wf.type !== 'FRAME') throw new Error('wireframe frame not found');
const f = figma.createFrame();
f.name = 'Pure suggestions - ' + wf.name;
f.resize(900, wf.height);
f.x = wf.x + wf.width + 120;
f.y = wf.y;
f.fills = [{ type: 'SOLID', color: { r: 1, g: 1, b: 1 } }];
f.clipsContent = false;
wf.parent.appendChild(f);
return { frameId: f.id, wireframeY: wf.y, wireframeHeight: wf.height };
```

## B. Add one row (write), repeat per mapped region

Inputs per row: `REGION_NODE_ID` (the wireframe region the row sits beside), `KIND` (`set` or `component`), `KEY` (componentSetKey or componentKey from `search_design_system`), `CAPTION` lines. Use `nextMinY` from the previous call so rows never overlap.

```js
const SUGGEST_ID = 'REPLACE';
const REGION_NODE_ID = 'REPLACE';
const KIND = 'set'; // 'set' | 'component'
const KEY = 'REPLACE';
const REGION_LABEL = 'REPLACE'; // e.g. 'Navbar + Search bar'
const CAPTION = ['REPLACE name - package', 'Confirmed', 'design_systems/.../components/NAME'];
const MIN_Y = 0; // nextMinY from the previous row

const frame = await figma.getNodeByIdAsync(SUGGEST_ID);
const wf = frame.parent.children.find(n => n.name === frame.name.replace('Pure suggestions - ', ''));
const region = await figma.getNodeByIdAsync(REGION_NODE_ID);
const wfBox = wf.absoluteBoundingBox, rBox = region.absoluteBoundingBox;
let y = Math.max(MIN_Y, rBox.y - wfBox.y);

await figma.loadFontAsync({ family: 'Inter', style: 'Regular' });
await figma.loadFontAsync({ family: 'Inter', style: 'Medium' });

let inst;
if (KIND === 'set') {
  const set = await figma.importComponentSetByKeyAsync(KEY);
  inst = set.defaultVariant.createInstance();
} else {
  const comp = await figma.importComponentByKeyAsync(KEY);
  inst = comp.createInstance();
}

function label(text, size, style, color) {
  const t = figma.createText();
  t.fontName = { family: 'Inter', style };
  t.fontSize = size;
  t.characters = text;
  t.fills = [{ type: 'SOLID', color }];
  return t;
}
const gray = { r: 0.4, g: 0.4, b: 0.4 };
const ink = { r: 0.1, g: 0.1, b: 0.1 };

const row = figma.createFrame();
row.name = 'Row - ' + REGION_LABEL;
row.layoutMode = 'VERTICAL';
row.itemSpacing = 8;
row.fills = [];
row.counterAxisSizingMode = 'AUTO';
row.primaryAxisSizingMode = 'AUTO';
row.appendChild(label(REGION_LABEL, 11, 'Medium', gray));
row.appendChild(inst);
CAPTION.forEach((line, i) => row.appendChild(label(line, i === 0 ? 13 : 11, i === 0 ? 'Medium' : 'Regular', i === 0 ? ink : gray)));
frame.appendChild(row);
row.x = 24;
row.y = y;
return { rowId: row.id, nextMinY: row.y + row.height + 24, instanceWidth: inst.width };
```

## C. Text card for items with no Figma component (write)

```js
const SUGGEST_ID = 'REPLACE';
const MIN_Y = 0;
const TITLE = 'REPLACE'; // e.g. 'ProductDetail (ui-ecommerce)'
const LINES = ['Exists in code, no Figma component', 'Key props: name, image, retailPrice, onAddToBag'];
const frame = await figma.getNodeByIdAsync(SUGGEST_ID);
await figma.loadFontAsync({ family: 'Inter', style: 'Regular' });
await figma.loadFontAsync({ family: 'Inter', style: 'Medium' });
const card = figma.createFrame();
card.name = 'Text card - ' + TITLE;
card.layoutMode = 'VERTICAL';
card.itemSpacing = 6;
card.paddingTop = card.paddingBottom = card.paddingLeft = card.paddingRight = 16;
card.counterAxisSizingMode = 'FIXED';
card.primaryAxisSizingMode = 'AUTO';
card.resize(420, 40);
card.fills = [{ type: 'SOLID', color: { r: 0.96, g: 0.96, b: 0.96 } }];
card.strokes = [{ type: 'SOLID', color: { r: 0.8, g: 0.8, b: 0.8 } }];
card.strokeWeight = 1;
card.cornerRadius = 6;
[TITLE, ...LINES].forEach((line, i) => {
  const t = figma.createText();
  t.fontName = { family: 'Inter', style: i === 0 ? 'Medium' : 'Regular' };
  t.fontSize = i === 0 ? 13 : 11;
  t.characters = line;
  t.fills = [{ type: 'SOLID', color: i === 0 ? { r: 0.1, g: 0.1, b: 0.1 } : { r: 0.4, g: 0.4, b: 0.4 } }];
  t.layoutAlign = 'STRETCH';
  card.appendChild(t);
});
frame.appendChild(card);
card.x = 24;
card.y = MIN_Y;
return { cardId: card.id, nextMinY: card.y + card.height + 24 };
```

## D. Verify (read-only)

```js
const frame = await figma.getNodeByIdAsync('REPLACE');
const rows = frame.children.map(c => ({ name: c.name, y: Math.round(c.y), h: Math.round(c.height), empty: c.findAll(n => n.type === 'INSTANCE').length === 0 && c.name.startsWith('Row') }));
return { count: rows.length, rows, emptyRows: rows.filter(r => r.empty).map(r => r.name) };
```

Notes:
- If an instance is wider than the 900px frame, widen the frame in step A (use the widest instance) rather than scaling the instance.
- Suggestion frames use Inter and neutral grays only; they are annotations, not part of the wireframe.
- If `importComponentSetByKeyAsync` throws, the library is not added to the file yet. Report it and fall back to a text card.
