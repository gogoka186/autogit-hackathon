---
title: Dice Tower
app_type: dice-tower
wallet: 0x98984e0130B6756AC5970799807A59185E0bb3DC
---

Roll dice. See the result. Keep a log. The page is a single column. The top is the input where you write a notation like 3d6+2 or 2d20kh1 advantage. The middle is the result, a row of small die faces with the total on the right. The bottom is the log, the last forty rolls written one per line. There is no character sheet, no campaign, no party, no account. The tower exists so that the people sitting around a table can keep their phone open and roll quietly when the lights are low.

## Build the surface

Obsidian for the page, the darker than coffee color of an unlit room. Parchment for the type, the color of paper held above a candle. Two accents only, blood for the natural maximum on a die and brass for the natural minimum. Crimson Pro serif at 16px for the body and the result total, weight 400 and 600 only. Roboto Mono at 14px for the notation input, the per die values, and every line of the log. No icons except a small inline svg of a six sided die in the top left corner, 16px, drawn with four paths. The column is at most 560px wide, centered, 24px gutter on small screens. Spacing is generous. Everything sits inside one card with 1px parchment hairline at twenty percent opacity, padded 28px, no rounded corners.

Roll the notation. The input is a single row at the top, a slash glyph in muted parchment to the left, a Roboto Mono input in the middle with a soft inset background that is a touch lighter than the page, no border, and a single text button to the right reading roll in 14px Crimson Pro 600 on a brass tinted background that fills the button at the moment of press for 140ms then returns. Pressing the enter key inside the input is identical to pressing the button. The input parses the notation the moment the user submits. The notation supports a count and a die size like 3d6, an optional sign and constant modifier like 3d6+2 or 1d20 minus 1, the standard keep modifiers kh and kl with a number like 2d20kh1 for advantage and 2d20kl1 for disadvantage, exploding dice marked with an exclamation like 3d6!, and reroll marked with a small r like 4d6r1 which rerolls every result of 1 once. Multiple terms can be combined with plus and minus like 1d20+1d4+3. Whitespace is allowed. Letters are case insensitive. A label can follow with a space, like 3d6 fireball, which then appears in the result and log next to the total.

## Wire the dice

The result region sits below the input separated by 16px of vertical space. The total is the largest element on the page, 44px Crimson Pro 600 in parchment, aligned left. To the right of the total on the same baseline sits a Roboto Mono 13px line in muted parchment showing the expression as it was parsed, like 3d6 plus 2, in case the user wants to verify the engine read what they meant. Below the total a single row of small die chips, one chip per individual die rolled, each chip a 36px square with the die face number in Roboto Mono 16px centered, with a 1px parchment hairline at thirty percent opacity. A natural maximum on a die (a 20 on a d20, a 6 on a d6, etc.) renders the chip border in blood and the number in blood. A natural minimum (a 1 on any die) renders in brass instead of blood. Dice that were dropped by a keep modifier render at thirty percent opacity with a single 1px line drawn diagonally across the chip. Dice produced by an explosion render with a small parchment dot in the top right of the chip, and a thin parchment line connects them to the chip that triggered them. Hovering or tapping a chip reveals a tiny tooltip in 11px Roboto Mono showing what the die was (its size and modifier) and the original roll before any reroll happened. The label, if present, sits below the chip row in 14px Crimson Pro italic, like fireball.

The roll button triggers a brief animation of the chips before the final values land. Each chip cycles through random faces for 300 to 450 milliseconds with a slight stagger across the row, the values settle one by one from left to right with a soft tick from WebAudio at each settle. The tick is a brief sawtooth at 800Hz with an 8ms attack and a 60ms release, gated by a single user gesture (the first roll). Audio is mixed at thirty percent gain, mono. If audio is blocked, the animation continues silently.

## Roll the log

Below the result, a thin horizontal parchment hairline at twenty percent opacity divides the page. Below the hairline, the log. It is a Roboto Mono 13px stack of lines, one per roll, the most recent at the top, padded 6px between lines, faded to seventy percent opacity for the older half of the list. Each line shows the time the roll happened in HH MM format on the left, the parsed expression in the middle, and the total on the right. If a label was provided, it sits in italic in muted parchment between the expression and the total. The log holds the last forty rolls. Older rolls drop off silently as new ones come in. A tiny text button in 12px Crimson Pro at the bottom of the log reads clear the log, with a confirm prompt that says forget every roll in this session, and a small cancel. Hovering over a log line for 400 milliseconds slides a tiny gloss to the right of the line in 11px Roboto Mono showing the per die values that made up the total.

The shape of a log entry is small and explicit, so the codegen does not have to guess.

```
{ id, time, expression, terms, dice, dropped, total, label }
```

Each die in the dice array stores its size, its rolled value, whether it exploded or was rerolled, and the chain of values if it was rerolled. The log array is saved to localStorage under the key dice.tower.log.v1 within 300ms of idle. On boot the saved log is restored. There is no export, there is no import, there is no share link. The log is private to the device and the session, and a tiny line of italic 11px parchment at the bottom of the page reads this log lives only in this browser on this device. If localStorage is unavailable, the log holds in memory and the same line reads this log is not being kept on this device. Clearing the log writes a single empty array.

A small text strip directly above the log holds quick chips for common rolls, each chip is a small Roboto Mono 12px pill in muted parchment with a 1px brass border at thirty percent opacity. The chips are 1d4, 1d6, 1d8, 1d10, 1d12, 1d20, 1d100, 2d20kh1 advantage, 2d20kl1 disadvantage, 4d6r1 stat. Tapping a chip drops the notation into the input and rolls it immediately. The chips are fixed, the user does not pick them.

A tiny text button in the top right of the card reads roll again, in 12px Crimson Pro brass. It rolls the previous notation with a fresh random outcome, useful for repeating attacks. Pressing the space key while the input is empty has the same effect.

Examples of input that should produce clean output.

```
3d6+2
2d20kh1 sneak attack
4d6r1 stat
1d20 plus 1d4 minus 1
```

## Done

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color only. Vite as the build tool. State is plain useState and one useReducer for the input parser and the log. No router, no global store, no context provider, no animation library, no dice library, no parser combinator library. The notation parser is a small hand written recursive descent function in src/lib/parse.ts that returns an abstract syntax tree of terms. The roller in src/lib/roll.ts walks the tree, calls a single rng that takes a min and a max integer inclusive, returns a result object that the UI renders. Every glyph including the small die in the corner and the diagonal drop line is inline svg in the component that uses it. The single audio context is created lazily on first roll inside src/lib/audio.ts. Persistence is in src/lib/storage.ts with a try catch.

Files. index.html with the Crimson Pro and Roboto Mono links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component that holds the current parsed expression, the current result, and the log. src/components/Header.tsx for the small die glyph and title line. src/components/Input.tsx for the notation input and the roll button. src/components/Result.tsx for the total and the chip row. src/components/Chip.tsx for one die chip. src/components/Quicks.tsx for the small row of quick chips. src/components/Log.tsx for the rolled log list. src/lib/parse.ts for the notation parser. src/lib/roll.ts for the roller and the result builder. src/lib/audio.ts for the tick. src/lib/storage.ts for localStorage. tailwind.config.ts extending the palette with obsidian, parchment, parchment muted, blood, brass. package.json with react, react dom, vite, and tailwind only. README.md with two short paragraphs.

A first time visitor opens the page and sees a dark card with an empty notation input and a small die in the corner. They type 1d20 and press enter, a single chip flickers and lands on 14, the total reads 14 in big serif, the log adds a line with the time. They press the 4d6r1 stat chip, four chips flicker, one was a 1 and rerolled into a 5, the dropped 1 is faded with a diagonal line, the total reads 18, the log writes the new line with the label stat in italic. They reload the page. The log is still there. They press clear the log, confirm, the log empties. They close the tab and reopen it tomorrow at a friend's table. The log is empty as they left it, the input is ready, the tick fires the first time they roll.
