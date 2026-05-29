---
title: Mandelbrot Zoom
app_type: mandelbrot-zoom
wallet: 0x0A4028624000cC76A1DD706aE2078C57D0B911Fe
---

A small page that draws the Mandelbrot set and lets the visitor zoom into it without locking the main thread. The fractal compute runs inside a Web Worker on an OffscreenCanvas. The main thread only handles input and the small legend at the bottom of the page. Pan with a drag, zoom in with a scroll wheel or a pinch, pick a deeper color palette from a small strip, save the current view as a png. The page is the smallest interesting demonstration of offloading heavy compute off the main thread that this codebase can ship.

## [01] surface

The page is a single column at full viewport width, ink for the background, bone white for type. One hot magenta accent reserved for the active swatch in the palette legend and for the small crosshair under the cursor while zooming. Source Serif Pro at 20px for the single title line at the top reading mandelbrot zoom. JetBrains Mono at 12px for every other piece of text on the page, weights 400 and 500. The Iosevka monospace family is used in place of JetBrains Mono if it is available in the visitor's system, falling back cleanly. No icons except a small inline svg of a tiny pinwheel in the title line, fourteen pixels across, one path. The layout is one page, no scroll, the canvas region fills the available space minus the bottom legend strip which is sixty pixels tall.

## [02] canvas

The canvas region is one HTMLCanvasElement element pointer captured by the App. On mount, the canvas calls transferControlToOffscreen and posts the resulting OffscreenCanvas object to a single Web Worker created at start. The worker holds the canvas, the current viewport (a center complex coordinate and a width in complex units), and a render queue. The main thread never draws on the canvas directly after the initial transfer. The canvas resolution tracks the visitor's device pixel ratio. On window resize the main thread posts a resize message to the worker carrying the new pixel width and height, the worker repaints.

The fractal is rendered in tiles of 64 by 64 pixels. The worker keeps a list of dirty tiles, walks them in priority order from the center of the canvas outward, and posts a progress message back to the main thread after each tile. The progress message carries only the count of completed tiles, no image data, so the postMessage queue stays small. The legend at the bottom of the page shows a tiny progress hairline that grows from left to right as tiles complete. When a tile is finished the worker draws it directly on the OffscreenCanvas with putImageData. The viewport is dirty after a pan or a zoom, at which point the queue is cleared and refilled.

## [03] input

The visitor drags inside the canvas with the mouse to pan. The pan uses Pointer Events with setPointerCapture so the drag works across mouse, pen, and touch. The main thread converts the pointer delta from pixels to complex units using the current viewport scale and posts a pan message to the worker. The worker updates its viewport and clears the tile queue. Mouse wheel scrolling on the canvas zooms in or out, with the zoom centered on the cursor location, not on the canvas center. Pinch with two pointers also zooms, with the zoom centered between the two pointers. The smallest zoom is the standard view that frames the full set, and the largest zoom is bounded only by the precision of the double floating point math, around 1e15 before the visitor sees pixelation. A small hot magenta crosshair appears under the cursor during a pan or a pinch and disappears 400 milliseconds after the input stops.

A small text button in 12px mono in the top right of the canvas reads reset the view. Pressing it resets the viewport to the standard view, with the center at minus 0.6 and zero, and a width of 3.4.

## [04] palette

A horizontal legend strip sits at the very bottom of the page, sixty pixels tall. The left side of the strip is the active gradient, a single horizontal band twenty four pixels tall mapping iteration count from zero on the left to the maximum on the right, drawn from the same gradient the worker uses to color the fractal. Below the band a tiny mono line shows the current maximum iteration count, like 512 iterations, in muted bone. The maximum iteration count adjusts itself based on the current zoom depth, deeper zooms raise the iteration count up to 4096, shallow zooms lower it back to 256.

The right side of the strip is a row of eight small swatches, each twenty pixels wide and twenty pixels tall, each showing a different gradient. The eight gradients are viridis, magma, inferno, mako, rocket, twilight, parchment, ice. The active swatch is outlined in hot magenta two pixels wide. Tapping a swatch posts a palette message to the worker, which redraws every tile with the new gradient and reposts them. The active swatch is persisted to localStorage under the key mandelbrot.zoom.v1.

A small text button in 12px mono on the very right end of the strip reads save the current view as png. Pressing it asks the worker to copy the OffscreenCanvas content into an ImageBitmap, transfers it back to the main thread, paints it into a hidden temporary canvas at full resolution, and downloads it via a single anchor click with a filename containing the current center coordinates and the zoom level. The whole save round trip takes well under a second at typical canvas sizes.

## [05] worker protocol

The main thread and the worker exchange three message kinds. The shape is small, written here so the codegen does not invent its own.

```
to worker:
  { type: 'init',    canvas: OffscreenCanvas, width: number, height: number, dpr: number }
  { type: 'resize',  width: number, height: number, dpr: number }
  { type: 'pan',     dx: number, dy: number }
  { type: 'zoom',    factor: number, cx: number, cy: number }
  { type: 'palette', name: string }
  { type: 'reset' }
  { type: 'save' }

from worker:
  { type: 'progress', done: number, total: number }
  { type: 'viewport', center: { re: number, im: number }, width: number, maxIter: number }
  { type: 'saved',    bitmap: ImageBitmap }
```

The worker never sends back tile image data, only progress counts and the occasional viewport update for the legend display. The bitmap message is the only transfer that carries pixel data back, and it is transferred (not cloned) using the transferList argument.

## [06] persistence

The active palette name, the most recent viewport (center coordinates and width), and the canvas size at last close are saved to localStorage under the key mandelbrot.zoom.v1. On boot the App reads them, the worker init message carries the saved viewport into the worker so the first paint shows the same view the visitor left. If localStorage is unavailable, the page falls back to in memory and a small italic 11px line at the very bottom right of the legend reads this view is not being kept on this device. The save as png action does not persist anywhere, it simply downloads the file through a temporary anchor element.

## [07] edges

If OffscreenCanvas is not supported by the browser (older safari for example), the App falls back to a single threaded version that runs the same compute on the main thread inside a small idle loop using requestIdleCallback. The page paints, the visitor can still pan and zoom, the performance is slower, a small italic 11px line at the bottom of the legend reads your browser does not support offscreen canvas, falling back to single threaded mode. Everything else still works.

A pan that lands outside the standard view bounds is allowed, the visitor can move into regions outside the main cardioid. A zoom out below the standard view is clamped at the standard view, the legend hairline pulses faintly to indicate the clamp. A zoom in past the precision limit shows a single mono line in the bottom of the canvas reading the pixel grid has overtaken double precision, with no other consequence. Resize events are throttled to one repaint every 80 milliseconds during a drag of the window edge.

## [08] build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding ink, bone, bone muted, and the hot magenta accent. Vite as the build tool, with the worker imported through the standard new Worker(new URL('./worker.ts', import.meta.url), { type: 'module' }) pattern that Vite understands. State is plain useState and a single useReducer for the viewport mirror. No router, no global store, no context provider, no fractal library, no math library, no icon pack. The compute is hand written, the color stops for each palette are hardcoded constants in src/lib/palettes.ts.

Files. index.html with the Source Serif Pro and JetBrains Mono links in the head, plus a small style block declaring Iosevka as the preferred mono with the JetBrains Mono fallback. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the canvas ref, the worker, and the legend state. src/components/Title.tsx for the small header. src/components/CanvasHost.tsx for the canvas element and the pointer event handlers. src/components/Legend.tsx for the bottom strip. src/lib/worker.ts as the Web Worker entry, importing src/lib/compute.ts for the iteration loop and src/lib/palettes.ts for the eight gradients. src/lib/protocol.ts for the typed message shapes. src/lib/storage.ts for the localStorage try catch. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page on a laptop. Within half a second the standard view paints in viridis, the title sits at the top reading mandelbrot zoom, the bottom strip shows the active swatch and a small progress hairline that completes inside a second. The visitor drags into the seahorse valley, the worker repaints. They scroll wheel forward and the view zooms in centered on the cursor. They tap the inferno swatch, the fractal recolors. They press save the current view as png and a file lands in their downloads folder with the center and zoom in the name. They reload the page. The same view they were looking at is back, in inferno.
