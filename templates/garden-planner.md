---
title: Garden Planner, Spring Bed Edition
app_type: garden-planner
wallet: 0xB29Dd0B83F78e609465D7FD86Ea7C752F0C91E95
---

A single page tool for sketching a vegetable bed and seeing whether the plants will actually fit. You drop crops onto a grid, the app draws their mature footprint, flags spacing conflicts, and prints a sowing calendar you can pin to the shed door. No accounts, no cloud, no AI suggestions. Just you, a grid, and the boring math that keeps tomatoes from shading basil.

The whole thing lives on one page with a working surface in the middle and two narrow rails on the sides. Background is the color of newsprint, ink is deep olive almost black, the plants are flat shapes in muted colors. It should read like a page from a small press field guide, not a SaaS dashboard.

## 01 Page layout

One page, three columns on desktop, stacked on mobile. Max width 1200px, centered, 24px gutter.

Left rail, 240px wide, is the crop drawer. A vertical list of 24 common spring and summer vegetables, each with an inline SVG glyph (top view of the mature plant, simplified to two or three shapes), the common name in 15px serif, the latin name in 12px italic muted, and a tiny spacing badge that reads the recommended in row by between row spacing in centimeters. Tap or click a crop to select it. The selected crop gets a 2px ring in clay and a faint background tint.

Center, fluid width, is the bed. A rectangular grid drawn with 30cm tiles. Default bed is 240cm by 120cm so the grid is 8 columns by 4 rows. The grid has a 1px hairline in dim ink. Tiles are not colored. Above the grid, a thin toolbar with the bed dimensions in cm (two number inputs separated by an X), a tile size selector (15cm, 30cm, 60cm), an orientation toggle (sun rises from this side), and a clear bed button on the right.

Right rail, 280px wide, is the calendar. A vertical timeline from February to October, one month per band, 32px tall. Inside each band, small horizontal pills mark the sow window, the transplant window, and the harvest window for every crop currently placed in the bed. Crops are stacked from top to bottom in placement order. The current month has a faint highlight band the full width of the rail.

## 02 Placing crops

Pick a crop from the left rail. The cursor over the grid shows a footprint preview, the same shape as in the drawer but sized to the crop's mature width and height in real centimeters, snapped to the tile grid. Click on a tile to plant. The crop sticks. Click again on an empty area to plant another of the same. Hit escape, or click the crop in the drawer a second time, to deselect.

A placed crop renders as the filled top view shape inside a soft circular footprint that shows the spacing zone. The footprint color matches the crop family color (see palette). If two footprints overlap by more than 5 percent of either area, both turn a faint warning red and the conflicting pair pulses for 800ms once. The warning persists. A small line under the toolbar reads, in italic, "two plants are too close" with a small jump to button that recenters the view on the conflict.

To move a placed crop, click and drag from inside its shape. The shape follows the cursor, snapped to tiles. Release to drop. To delete, hover the shape and click the small x glyph that appears in the top right of its footprint. Or focus the shape with tab and press delete or backspace. There is no multi select. There is no copy paste.

## 03 Crop drawer contents

The drawer is fixed. These 24 crops in this order. Each line has the english name, the latin name, the in row by between row spacing in cm, and the family group used for footprint color.

Tomato, Solanum lycopersicum, 60 by 90, solanum.
Pepper, Capsicum annuum, 45 by 60, solanum.
Eggplant, Solanum melongena, 60 by 75, solanum.
Cucumber, Cucumis sativus, 45 by 90, cucurbit.
Zucchini, Cucurbita pepo, 90 by 120, cucurbit.
Pumpkin, Cucurbita maxima, 120 by 180, cucurbit.
Bean bush, Phaseolus vulgaris, 15 by 45, legume.
Pea, Pisum sativum, 8 by 30, legume.
Carrot, Daucus carota, 8 by 25, umbellifer.
Parsnip, Pastinaca sativa, 10 by 30, umbellifer.
Beet, Beta vulgaris, 10 by 25, chenopod.
Chard, Beta vulgaris cicla, 25 by 40, chenopod.
Spinach, Spinacia oleracea, 15 by 25, chenopod.
Lettuce, Lactuca sativa, 25 by 30, aster.
Kale, Brassica oleracea acephala, 45 by 60, brassica.
Cabbage, Brassica oleracea capitata, 45 by 60, brassica.
Broccoli, Brassica oleracea italica, 45 by 60, brassica.
Radish, Raphanus sativus, 5 by 15, brassica.
Onion, Allium cepa, 10 by 25, allium.
Garlic, Allium sativum, 10 by 20, allium.
Leek, Allium ampeloprasum, 15 by 30, allium.
Basil, Ocimum basilicum, 25 by 30, herb.
Parsley, Petroselinum crispum, 20 by 30, herb.
Strawberry, Fragaria ananassa, 30 by 45, rose.

Family color is used only for the footprint ring and a 1px family tag in the calendar. The leaf shape inside is always the dim olive ink.

## 04 Sowing calendar

For each placed crop, derive three windows from a built in table keyed by family and crop. The table is hardcoded in the project for a temperate northern hemisphere zone, roughly USDA 7. Sow indoor window, transplant window, harvest window. Each window is a month range like Feb 15 to Apr 1.

In the right rail timeline, render each window as a horizontal capsule, 6px tall, color coded. Sow indoor is dim clay. Transplant is sage. Harvest is wheat. A small label sits to the right of each capsule with the crop name in 12px. If two capsules in the same band overlap horizontally, stack them on two rows inside the band.

Below the timeline, a single button reads "print this calendar". Pressing it opens window.print with a print stylesheet that hides the rails and grid and shows only the calendar plus the bed name and the date range. The print is intended for an A5 page, portrait.

## 05 Saving and loading

State lives in localStorage under the key garden.planner.bed.v1. Saved shape.

```json
{
  "bed": { "widthCm": 240, "heightCm": 120, "tileCm": 30, "sunFromSide": "south" },
  "name": "north bed",
  "crops": [
    { "id": "c1", "key": "tomato", "x": 30, "y": 30 },
    { "id": "c2", "key": "basil", "x": 90, "y": 30 }
  ]
}
```

x and y are the top left corner of the crop footprint in centimeters from the top left of the bed. On first load, seed with a small example bed named "north bed" containing two tomatoes, four basils, and a row of carrots. If localStorage already has data under that key, do not overwrite it.

A small text input above the bed shows the bed name and lets the user rename it. Renames save on blur. The clear bed button asks "remove all crops from this bed" with a small cancel.

There is no multi bed feature. There is one bed.

## 06 Interactions

Selecting a crop in the drawer with the keyboard. Up and down arrow move selection through the list, enter selects, escape deselects.

When a crop is selected and the grid has focus, arrow keys move the preview cursor across tiles, enter plants, escape clears the preview.

Dragging a placed crop snaps to tiles every 1cm during the drag, settles to the nearest tile on release.

The footprint pulse on conflict is one 800ms ease in out, then the warning red color stays until the conflict is resolved.

The print button respects the system print dialog and does not try to bypass it.

## 07 Edge cases

A crop placed off the edge of the bed snaps back inside on drop, with no error message.

A bed resized smaller than the placed crops keeps the crops where they were and shows a small line at the bottom of the toolbar that says "some plants are now outside the bed", with a button "fit them back in" that nudges every outside crop to the nearest legal tile.

Two crops at the exact same tile count as a conflict.

A bed dimension below 60cm in either direction is rejected with a tiny inline message under the input that says "minimum 60cm".

Tile sizes other than 15, 30, 60 are not allowed. Selector is a segmented control, not free input.

Loading is synchronous. No spinner. The first paint shows the seeded bed, the calendar with three crops worth of capsules, and the drawer ready.

If localStorage is unavailable, fall back to in memory state and show a small line in the footer "this bed is not being saved on this device".

## 08 Visual

Colors.

Page background, the paper color, #FAF7F0.
Grid lines, dim ink, #C9C2B4.
Primary ink for text, #2A2E26.
Muted ink for secondary text, #6E6F62.
Dim ink for placeholders and small labels, #9A9A8C.
Tile hover wash, #F0EBDC.
Conflict red, #B85440.

Family colors, used for the footprint ring at 60 percent opacity and the calendar family tag.
solanum, #C26A5A.
cucurbit, #D4A24C.
legume, #7FB069.
umbellifer, #C9B68C.
chenopod, #9B6B8E.
aster, #A8C28A.
brassica, #6F8B9F.
allium, #B59B6F.
herb, #7FB069 with 70 percent saturation.
rose, #C58B8B.

Type. Newsreader serif for the bed name, the toolbar dimensions, and the month band labels. Inter for everything else. IBM Plex Mono for the small spacing badges and any centimeter values. Tabular figures wherever there are numbers. Load all three from Google Fonts.

Spacing. Generous. 24px between the rails and the bed. 16px between drawer rows. 12px between toolbar controls. 32px above and below the timeline. The bed itself sits on a paper card with 24px padding.

Corners. 6px on cards. 4px on the small spacing badges. 0px on the grid tiles (square is the whole point).

Motion. The footprint pulse is 800ms ease in out. The drag follows the cursor in real time. The calendar capsules fade in over 200ms when their crop is first placed. Nothing else moves.

## 09 Stack

React 18 with TypeScript. Tailwind CSS for layout and color, with the paper and ink palette extended in tailwind.config.ts. Vite as the build tool. One default export from src/App.tsx. No state library, useState and useReducer only. No date library, native Date and Intl only. No drag and drop library, plain pointer events. No icon packs, every glyph is inline SVG written in the components that use it.

## Files to produce

index.html with Google Fonts links in the head.
src/main.tsx.
src/App.tsx.
src/components/CropDrawer.tsx.
src/components/BedGrid.tsx.
src/components/PlacedCrop.tsx.
src/components/Calendar.tsx.
src/components/Toolbar.tsx.
src/lib/storage.ts (localStorage with try catch).
src/lib/crops.ts (the 24 crops table with spacing and family).
src/lib/calendar.ts (the sow, transplant, harvest windows table and lookups).
src/lib/geometry.ts (overlap detection, snap to tile, fit inside bed).
src/lib/seed.ts (the example bed).
tailwind.config.ts.
package.json with react, react dom, vite, tailwind only.
README.md, two short paragraphs covering what it is and how to run it.

## Done when

A first time visitor opens the app and within ten seconds sees a paper colored page with a small bed already drawn, two tomato shapes, four basils, a line of carrots, and the right rail showing capsules for sow, transplant, and harvest windows for those three crops. They click a pepper in the drawer, click an empty tile, and a pepper footprint appears. They drop a second pepper too close, both pulse red, the toolbar shows the warning. They drag one away, the warning clears. They press print, the print dialog shows a clean A5 calendar.
