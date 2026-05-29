---
title: ISS Tracker
app_type: iss-tracker
wallet: 0x9d355697e2A7b0AF45386684a24E434C9F3c1397
---

The International Space Station crosses the sky every ninety three minutes. The page paints a flat world map, drops a small marker where the station is right now, traces the next ninety minutes of ground track ahead of it, and writes the current altitude and velocity in a console green sidebar. Nothing on the page requires a network connection. The orbit is propagated from a pair of orbital elements baked into the source. The result is close enough for casual viewing. There is no login, no api key, no leaderboard, no settings menu.

1. The surface

The page background is a deep space color, the kind of dark you see in the corner of a control room at three in the morning. The primary text color is console green, the brightness of a vintage crt set just dim enough to read in the dark. Warm orange is reserved for the station marker and the ground track ahead. Faint cyan is reserved for the city labels that sit under the station as it crosses, and for the small lines that connect the sidebar readouts to the marker. IBM Plex Sans Condensed at 16px for the body and at 12px for the smaller readouts, weights 400 and 500. IBM Plex Mono at 12px for every number and every coordinate. No icons except a small inline svg of a four pointed star compass rose in the top right corner of the page, twelve pixels across, three paths. The layout is a single page, the world map fills 70 percent of the viewport width on desktop and stacks above the sidebar on phones.

2. The map

The world map is drawn on a Canvas2D context inside a single canvas element sized to the available width and a fixed sixteen by nine aspect. The map is an equirectangular projection, meaning longitude maps linearly to x and latitude maps linearly to y, the simplest projection there is. The coastlines are drawn from a small in source array of polygons, lightly simplified to a few hundred points per continent, hardcoded into src/lib/coastlines.ts as a flat array of latitude longitude pairs. The continents render as one pixel console green strokes at fifty percent opacity, no fill. The seas are simply the background. A 1px console green hairline at fifteen percent opacity draws the equator and the international date line, and four small numbered marks indicate ninety west, zero, ninety east, and one eighty along the equator. Latitude lines at thirty north and thirty south are drawn at five percent opacity. There are no other features on the map.

The canvas is redrawn on every animation frame through requestAnimationFrame. The redraw is cheap. The map background is cached into an offscreen canvas once on first paint, then blitted at the start of every frame, after which the live elements are drawn on top. The live elements are the marker, the trailing track from twenty minutes ago up to the current moment, and the forward track for the next ninety minutes drawn in dashed warm orange.

3. The marker

The current station position is drawn as a solid warm orange disc, six pixels across, with a thin one pixel console green outline at fifty percent opacity. A small label in IBM Plex Mono 11px warm orange sits to the right of the marker reading ISS, with the current altitude in kilometers and the current ground speed in kilometers per hour stacked on two lines below. The marker is repositioned on every frame as the orbit propagates forward. When the marker crosses the right edge of the map it wraps to the left edge in the same frame, no animation, no easing.

A small inline panel just above the marker shows the nearest named city, picked from a small in source list of about a hundred and twenty cities written into src/lib/cities.ts. The nearest city is computed by simple haversine distance and the label is drawn in faint cyan IBM Plex Mono 11px. If the nearest city is more than one thousand kilometers away (the station is over ocean), the label reads the ocean name from a small in source list of major oceans and seas, in italic.

4. The sidebar

To the right of the map on desktop, below the map on phones, a vertical console green panel padded 18px, the only piece of real chrome on the page. Inside the panel four short blocks stacked.

The first block, in IBM Plex Sans Condensed 14px 500, reads station now. Below it four small lines in IBM Plex Mono 12px. Latitude. Longitude. Altitude in kilometers. Velocity in kilometers per hour. Each value is rendered with two decimal places of precision, the unit suffix in faint cyan and the number in console green.

The second block reads next pass over you. Below it three lines. Rise. Peak. Set. Each line shows the local time in 24 hour format with the elevation above the horizon at peak in degrees. If the page does not know the visitor's location (the geolocation prompt was declined or unavailable), the next pass block instead shows the next ground track crossing of the equator with the longitude and the local time at that longitude in italic, and a small text button in 12px IBM Plex Mono cyan reads use my location.

The third block reads orbit. Below it three lines, inclination in degrees, orbital period in minutes, current revolution number since the epoch. The revolution number is a positive integer that increments as the station completes orbits, a tiny brass band of theatre, useful for nothing except looking cool in the corner.

The fourth block reads track, with two small text buttons in IBM Plex Mono 12px reading show fifteen minutes back, show ninety minutes ahead. Tapping either toggles the visibility of the corresponding part of the ground track on the map. By default both are on. A small text button below the two reads reset the view, which restores the defaults.

5. The orbit

The station position is computed from a pair of two line element style constants hardcoded into src/lib/elements.ts. The constants describe the orbit at a known reference epoch (a recent date, written into the file as a comment). On each frame the engine computes the time since the epoch, propagates the mean anomaly forward using the mean motion, converts to eccentric anomaly via a few Newton iterations, converts to true anomaly, and applies the inclination, right ascension of the ascending node, and argument of perigee to produce the latitude and longitude on the ground. This is a simplified SGP propagator. It is accurate to a few kilometers over a few weeks, which is more than enough for a page that exists to look pretty. The math lives in src/lib/propagate.ts as a single function that takes the elements and a date and returns the latitude longitude altitude and velocity. The file does not depend on any library.

A tiny block of constants used by the propagator, written into the file as a data block, looks roughly like this.

```
epoch:           2026-05-25T00:00:00Z
inclination:     51.6422
right ascension: 124.7831
eccentricity:    0.0006703
arg of perigee:  301.9904
mean anomaly:    58.0823
mean motion:     15.5008 rev per day
revolution num:  29455
```

The numbers are read at boot and converted into the right internal units, the propagator runs at frame rate without allocating new objects after the first frame.

6. The data dance

There is no fetch. There is no api call. The orbit is propagated from the constants, the coastlines are in source, the cities are in source. The visitor's location, when the visitor consents through the standard navigator.geolocation prompt, is used to compute the next pass over their position by stepping the propagator forward at one minute increments until the station's elevation above the visitor's horizon exceeds zero, then refining to the moments of rise, peak, and set. The next pass block updates once per minute, the marker updates every frame. localStorage stores the visitor's consent state and the most recent successful location under the key iss.tracker.place.v1. If localStorage is unavailable, the location lives in memory and a small italic 11px line at the bottom of the sidebar reads this location is not being kept on this device. There is no other persistence.

If the visitor declines location, the sidebar shows the equator crossing block, the map and the marker still work, the page is fully functional. The geolocation prompt is asked once on first load, never again automatically. The use my location button asks again on demand.

7. The shell

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding space, console, console muted, warm orange, faint cyan. Vite as the build tool. State is plain useState and one useReducer for the track and visibility flags. The render loop is a single requestAnimationFrame inside src/App.tsx. The canvas is drawn through a small wrapper hook in src/lib/canvas.ts. The propagator is pure math, no React inside. The coastlines and cities are constant arrays imported once. No router, no global store, no context provider, no map library, no astronomy library, no icon pack. The compass rose in the top right corner is inline svg, the small disc on the map is drawn directly on the canvas with arc.

Files. index.html with the IBM Plex Sans Condensed and IBM Plex Mono links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the canvas ref, the live state, and the sidebar. src/components/MapCanvas.tsx wrapping the canvas and the rAF loop. src/components/Sidebar.tsx for the four blocks. src/components/Compass.tsx for the corner glyph. src/lib/propagate.ts for the propagator. src/lib/elements.ts for the orbital constants. src/lib/coastlines.ts for the simplified continent polygons. src/lib/cities.ts for the city dataset. src/lib/oceans.ts for the ocean and sea names. src/lib/horizon.ts for the elevation and pass refinement. src/lib/storage.ts for the localStorage try catch. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page in a dark room. The world map paints, the equator and date line appear, the small console green coastlines settle, and within a second the orange disc lights up somewhere over the south pacific with a small label reading ISS, 412 km, 27 600 km/h. The dashed ground track stretches ahead toward chile. A small faint cyan label reads pacific ocean. The sidebar fills with the station now values, the orbit numbers, a small button asking for the visitor's location. They tap use my location, accept the browser prompt, the next pass block reads rise 21:14, peak 21:18 at 47 degrees, set 21:22. They watch the marker creep across the map for a minute. They close the tab. They open it the next morning. The location is remembered. The station is somewhere new.
