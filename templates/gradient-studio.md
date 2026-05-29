---
title: Gradient Studio
app_type: gradient-studio
wallet: 0x82dCfC0B392F9f2EdeAdb81bfD2543fB6880Ee1f
---

Build a tool to compose layered CSS gradients. One page. Top half is the preview, bottom half is the editor. The user adds stops, drags them along the bar, swaps between linear, radial, conic. Every change shows in the preview within a frame. The user copies the CSS or the Tailwind class. Done.

## Frame

White page. Ink for text. One fuchsia accent reserved for the active stop handle and the copy button. Inter weight 500 for all interface text. JetBrains Mono for hex values and copied output only. No serifs, no other weights, no other colors.

#### preview

The preview fills the upper half of the viewport, never less than 320 pixels tall. The composed gradient is painted as the background of this region. A 1px ink hairline borders it on all four sides. In the top right corner of the preview, four small text buttons in mono 12px read png, jpg, css, tailwind. The first two save the preview as a rasterised file at 1200 by 800. The last two copy the snippet to the clipboard and a small toast slides from the top reading copied to clipboard for 1400 milliseconds.

#### type picker

Below the preview, a row of three text buttons in mono 13px reading linear, radial, conic. The active one has a fuchsia underline 2px thick. Clicking changes the gradient kind and the bar below reshapes accordingly.

## Editing

The bar sits below the type picker. The bar is 32 pixels tall, the full width of the column, rendered with the current gradient flowing across it left to right regardless of the actual angle or shape. Stops are shown as small circles 18 pixels across, sitting on top of the bar at the percent positions, color matching the stop. Dragging a stop horizontally updates its percent in real time. Clicking the empty bar between two stops inserts a new stop at that exact percent, color interpolated from the neighbors. Right clicking or long pressing a stop removes it. The active stop has a fuchsia ring 2px thick around the circle.

#### stop fields

Directly under the bar, four narrow input fields in a single row. Hex, alpha, percent, ease. Hex accepts a 6 character or 8 character hex value, validated on blur, invalid input reverts. Alpha is a slider from 0 to 100. Percent is a number input from 0 to 100, also reflected by the position of the active stop on the bar. Ease is a dropdown of linear, smooth, hard, where smooth applies an extra interpolation point at the midpoint, and hard inserts a duplicate of the next stop at percent minus 0.1.

#### angle

For linear, a number input labelled angle, in degrees, from 0 to 359, with a tiny inline svg compass that rotates to match. For radial, a dropdown of circle, ellipse and a second dropdown of closest side, closest corner, farthest side, farthest corner. For conic, a number input for the from angle, plus a position input written as two percent values separated by a space.

#### stops list

Right of the angle controls, a vertical list of all stops in order, each row showing a color swatch 16 by 16, the hex in mono 12px, the percent in mono 12px aligned right. Click the row to make that stop the active one. The list is reorderable by dragging a row up or down, which renumbers the percents proportionally to keep visual order intact.

## Presets and sampling

Below the controls, a horizontal scrollable row of preset gradient swatches, each 96 wide by 56 tall, with a 1px ink hairline border. The presets are sunrise, ocean, magma, mint, dusk, raincloud, candy, citrus, plum, bone. Clicking a preset overwrites every stop and type with the preset values. There is no save preset feature. Custom work lives only in the current session.

A small button in mono 13px reading sample image sits to the right of the presets row. Clicking it opens a file picker for a local image, decodes it onto an offscreen canvas, and shows a 240 wide preview underneath with a small reticle that follows the cursor. Clicking inside the preview picks the pixel under the reticle and assigns its hex to the currently active stop. The image never leaves the page. There is no upload anywhere.

## Output

At the very bottom of the editor, a single block of mono 13px text on a 1px ink hairline card with rounded 4 corners. The block shows the CSS in a tight format like

```
background: linear-gradient(135deg, #FFD166 0%, #EF476F 60%, #06D6A0 100%);
```

Clicking the block copies the text and shows the same toast. To the right of the block, a tab labelled tailwind shows the same gradient as a Tailwind arbitrary value class

```
bg-[linear-gradient(135deg,_#FFD166_0%,_#EF476F_60%,_#06D6A0_100%)]
```

Clicking the tailwind tab swaps the block content. The output updates every time any control or field changes.

## Behavior

Arrow left and right while a stop is active nudge its percent by 1, hold shift for 5. Delete or backspace removes the active stop. Plus adds a new stop at 50 percent. Letters l, r, c switch the gradient type. Letter t toggles between css and tailwind output. Cmd or ctrl plus c copies the current output.

The full editor state, a flat object with type, angle, shape, position, fromAngle, and a stops array, is mirrored into the url hash on every change, encoded as a short base64url string. Loading a page with that hash decodes and restores the state. This is the only sharing mechanism. There is no server, no account, no save list. localStorage stores only the most recent state under the key gradient.studio.last.v1 as a fallback for when no hash is present.

On boot, if the url hash is present and valid, decode it. Otherwise, if localStorage has a saved state, load it. Otherwise, seed a sunrise default with three stops at 0, 60, 100 with hex FFD166, EF476F, 06D6A0 and a linear angle of 135. The preview paints once on the first frame. There is no spinner.

A hex value that fails validation reverts to the last good value, the input briefly tints fuchsia for 200 milliseconds. The bar never collapses below 240 pixels wide, the layout switches to stacked controls when the viewport drops under 520 pixels. The conic gradient angle wraps at 360. Two stops cannot occupy the exact same percent, an attempt offsets the new one by 0.1. The eyedrop tool falls back silently if the image fails to decode, with a small mono 11px line below the picker reading could not read this image, with no other consequence.

There is no sound. The page does not produce audio. Audio buttons are not present.

## Build

React 18 with TypeScript. Tailwind CSS for layout, color, and the responsive stack. Vite as the build tool. The default export from src/App.tsx is the only component used as the entry point. State is held in a single useReducer at the top, the actions are addStop, removeStop, updateStop, setType, setAngle, setShape, setPosition, setFromAngle, loadState. No router. No global store. No context. No animation library. No icon pack. Every glyph including the compass and the eyedrop reticle is inline svg in the component that uses it.

Files. index.html with the Inter and JetBrains Mono links in the head. src/main.tsx as the React entry. src/App.tsx exporting the single default component. src/components/Preview.tsx. src/components/Bar.tsx. src/components/StopFields.tsx. src/components/AngleControls.tsx. src/components/StopList.tsx. src/components/Presets.tsx. src/components/Eyedrop.tsx. src/components/Output.tsx. src/lib/gradient.ts holding the css string builder, the tailwind class builder, and the interpolation helper. src/lib/hash.ts holding the encode and decode of the state into the url hash. src/lib/canvas.ts holding the rasterise helper for png and jpg export. src/lib/storage.ts wrapping localStorage in a try catch. tailwind.config.ts extending the palette with ink and the fuchsia accent. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs covering what it is and how to run it.
