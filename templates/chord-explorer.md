---
title: Chord Explorer
app_type: chord-explorer
wallet: 0x2f5D4F1C3d84F1EdDb5c84c7A43820080a141fcb
---

Pick a root, pick a quality, hear the chord. Look at the keyboard, see the notes light up. Look at the fretboard, see the same notes on guitar. Swap the inversion, the keys shift. Click a scale tab, the notes that belong to the chord stay lit and the rest of the scale glows faint. That is the whole app. One page, two instruments, a thin row of controls.

### Surface

Charcoal page, off white type, one amber accent. Layout is a vertical stack at most 720px wide, centered, 28px gutter on small screens. From top to bottom, the regions are, the title line, the controls strip, the keyboard, the fretboard, the readout block, the scale picker. Nothing else exists.

### Title

A single line, 14px mono, color dim. Reads "chord explorer" in lowercase. To the right, in the same line, the current chord name in 22px mono, color off white. Example "C maj7 / E". The slash and the bass note appear only on non root inversions.

### Controls

Three groups in one row. Group one, the root, twelve buttons in a single row, each 36px wide, mono 14px, reading C, C#, D, D#, E, F, F#, G, G#, A, A#, B. Selected root has a 2px amber underline, no fill. Group two, the quality, a horizontal scroll row of pills, each pill is the quality short name in 13px mono, examples maj, min, dim, aug, sus2, sus4, maj7, min7, dom7, dim7, m7b5, maj9, min9, add9, 6, m6, 7sus4. Selected pill has amber fill and charcoal text. Group three, the inversion, four small numbered buttons 0, 1, 2, 3. Zero is the root position. The number greys out if the chord does not have that many notes (a triad has 0, 1, 2 only).

### Keyboard

Three octave piano, C3 to C6. Drawn entirely in inline SVG. White keys are 32px wide and 140px tall with a 1px dim divider. Black keys are 20px wide and 88px tall, drawn over the white keys at the correct offsets. Default state, all keys are unlit. When a chord is selected, every key matching a note in the chord turns amber. The bass note (the lowest sounding pitch given the inversion) gets a darker amber fill plus a tiny dot below its key label. Hovering a lit key shows a tiny floating label one line above the keyboard, mono 12px, reading the note plus its octave example "E4". Clicking any key, lit or unlit, plays that single note through a WebAudio oscillator for 600ms.

### Fretboard

Six strings, fifteen frets, low E on the bottom. SVG. Each string is 1px in dim, each fret is 1px in dim, inlays are tiny circles on frets 3, 5, 7, 9 and a double inlay on 12. Default state, every fret is dark. When a chord is selected, the lowest practical voicing for that chord and inversion is computed and the four to six fingered positions light up as amber circles 10px wide with the note name inside in 9px mono charcoal. If a string is muted in that voicing, the nut for that string shows a small x in dim. Tapping or clicking any lit circle plays that single note for 600ms. Double click the fretboard background to play the full voicing as an arpeggio, twelve milliseconds between notes, bottom to top.

### Readout

A block of four short mono lines, 14px, color off white. Line one, the chord name expanded to its long form, example "C dominant seventh, first inversion". Line two, the notes from bottom to top with octave numbers, example "E4 G4 Bb4 C5". Line three, the intervals from the root, example "3 5 b7 1". Line four, two small button shapes, 24px tall, mono 12px, reading "play piano" and "play guitar". Each plays the chord through WebAudio. Piano uses a triangle wave envelope, guitar uses a sawtooth with a short pluck envelope, both at 60 percent gain mixed to mono.

### Sound

WebAudio context lazy created on first user gesture. One shared AudioContext. The piano envelope is attack 5ms, decay 120ms, sustain 0.4, release 350ms. The guitar envelope is attack 2ms, decay 90ms, sustain 0.15, release 220ms with a one pole lowpass at 3500 Hz. Notes mapped from name plus octave to frequency via standard equal temperament with A4 at 440 Hz. If the browser blocks audio until interaction, the first click anywhere on the page resumes the context, and the very next sound triggers normally.

### Scale picker

Below the readout, a tight tab row, 32px tall, mono 13px. Tabs are major, natural minor, dorian, mixolydian, harmonic minor, melodic minor, pentatonic major, pentatonic minor, blues. Selecting a scale overlays a thin amber dot on every note in the scale across both the keyboard (just above each matching white key, just below each matching black key) and the fretboard (small open circle at every matching position, not a filled circle). Chord tones keep their full amber fill. Scale tones that are not in the chord render as the dotted outline only. Click a scale tab a second time to clear the overlay. Only one scale can be active at a time.

### Defaults

On first load the app shows C major in root position with no scale overlay. The keyboard shows C4, E4, G4 lit, the fretboard shows the open C voicing on the second through fifth strings. The readout reads accordingly. The amber color used everywhere is the single hex value #E5A53A.

### Persistence

localStorage key chord.explorer.state.v1. Saves the last selected root, quality, inversion, and scale. On boot, if present, restores them. If absent, uses the defaults above. If localStorage throws, the app uses in memory state and writes a tiny dim line at the very bottom of the page, mono 11px, reading "this session is not being saved".

### Keys

Letter A through G on the keyboard sets the root to that note. Holding shift while pressing the letter sets the root to the sharp variant when one exists, otherwise it stays. Number keys 1 through 9 cycle through qualities in the order shown in the pill row. Arrow up and down move through inversions. Spacebar plays the current chord on the last used instrument (piano if none used yet). Escape clears any active scale overlay.

### Math

Note names internally map to integers 0 through 11 where C is 0. Chord recipes are integer offsets from the root. maj is [0, 4, 7], min [0, 3, 7], dim [0, 3, 6], aug [0, 4, 8], sus2 [0, 2, 7], sus4 [0, 5, 7], maj7 [0, 4, 7, 11], min7 [0, 3, 7, 10], dom7 [0, 4, 7, 10], dim7 [0, 3, 6, 9], m7b5 [0, 3, 6, 10], maj9 [0, 4, 7, 11, 14], min9 [0, 3, 7, 10, 14], add9 [0, 4, 7, 14], 6 [0, 4, 7, 9], m6 [0, 3, 7, 9], 7sus4 [0, 5, 7, 10]. An inversion of n rotates the first n notes up by an octave. Voicings on guitar pick the lowest fret position where every chord tone appears within a four fret span on adjacent strings, muting any string that does not contribute. If no such voicing exists for a given chord, the fretboard shows a small dim line that reads "no clean voicing under fret 12" and the play guitar button is greyed.

### Edges

Resizing the window below 380px stacks the controls into two rows. Below 320px the fretboard shrinks proportionally and the note labels inside the lit circles disappear. The fretboard never scrolls horizontally. The keyboard never scrolls horizontally. The page never has a horizontal scrollbar. If WebAudio is unavailable, audio buttons render as visible but disabled and a tiny dim line at the top of the readout reads "audio is not available in this browser". The visual app still works.

### Type

JetBrains Mono everywhere. Loaded from Google Fonts. Weights 400 and 600 only. No serif, no sans, no other font on the page. Numbers use tabular figures by default through the font feature setting.

### Files

The build is one Vite project. index.html with the JetBrains Mono link in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component that holds all top level state. src/components/Controls.tsx for the three button groups. src/components/Keyboard.tsx for the SVG piano. src/components/Fretboard.tsx for the SVG guitar. src/components/Readout.tsx for the four line block and play buttons. src/components/ScaleTabs.tsx for the bottom tab row. src/lib/theory.ts for note names, chord recipes, scale tables, interval naming, and the voicing solver. src/lib/audio.ts for the AudioContext singleton and the two envelopes. src/lib/storage.ts for localStorage with a try catch wrapper. tailwind.config.ts extending the palette with charcoal, off white, dim, and amber. package.json with react, react dom, vite, and tailwind only. README.md with two short paragraphs describing the app and how to run it.

### Stack

React 18, TypeScript, Tailwind CSS, Vite. Single default App export. No external UI libraries. No icon packs. No audio library. WebAudio used directly. Every shape on the page including the piano, the guitar, the inversion buttons, and the small play glyphs is inline SVG written in the component that uses it. State is plain useState and useReducer. No router, no context provider, no global store.

### Acceptance

The page opens in under a second on a fresh browser. The first paint shows C major lit on both instruments, the readout shows "C major, root position", the four line block reads cleanly. Clicking the dom7 pill switches the chord, the keyboard relights instantly, the fretboard recomputes the voicing, the readout updates. Pressing the play piano button produces a soft triangle wave chord through the speakers within 50 milliseconds of the click. Switching to mixolydian shows scale dots layered behind the chord tones. Reloading the page restores everything. Resizing down to phone width keeps the app readable, the fretboard intact, no horizontal scroll.
