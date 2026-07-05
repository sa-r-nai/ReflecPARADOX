# AGENTS.md

## Scope

This file applies to the whole repository.

When implementing, editing, validating, or generating simai chart text, follow the
official simai notation summarized here. The source references are:

- https://w.atwiki.jp/simai/pages/1002.html
- https://w.atwiki.jp/simai/pages/1003.html

Do not invent simulator-specific extensions unless this repository already
documents or implements them explicitly.

The project-specific rules below override the general simai summary. The later
sections still describe official simai syntax so unsupported note types can be
recognized and rejected intentionally.

## Reflec Beat Project Rules

- This repository implements a Reflec Beat style rhythm game.
- Develop runtime and input behavior for Android and iOS.
- simai is used as the source chart notation, but this project supports only the
  note subset listed in this section.
- If a valid simai chart contains any unsupported note type, force a visible
  error instead of trying to play, downgrade, or silently ignore that note.
- Do not raise project-specific unsupported-note errors for valid non-note simai
  syntax. Timing controls, commas, whitespace, chart end markers, metadata,
  duration brackets, EACH separators, and pseudo-EACH timing separators should
  only error when their own syntax is actually malformed.

Supported note types:

- TAP
- HOLD
- EX TAP
- EX HOLD
- BREAK HOLD
- EX BREAK HOLD

Unsupported note examples that must raise visible errors:

- TOUCH and TOUCH HOLD
- SLIDE, BREAK SLIDE, chained SLIDE, and multiple SLIDE
- BREAK TAP
- star TAP or rotating star TAP using `$` or `$$`
- SLIDE-specific modifiers such as `@`, `?`, and `!`
- TOUCH firework notation using `f`
- any other official simai note type or note modifier not listed as supported

Reflec Beat note behavior:

- TAP becomes a normal tap note.
- HOLD becomes a normal hold note.
- EX TAP becomes a fixed-lane tap note. It must fall only on the authored lane.
- EX HOLD becomes a fixed-lane hold note. It must fall only on the authored lane.
- BREAK HOLD becomes a danger hold note. If the player is judging/holding while
  this note is active, the result is MISS.
- EX BREAK HOLD becomes a fixed-lane BREAK HOLD. It must fall only on the
  authored lane and use BREAK HOLD judgment behavior.

Lane and movement rules:

- The visible game should feel like it has no judgment line, but internally notes
  fall to seven judgment lanes.
- Authored simai button/lane `8` is invalid for this project. If even one note is
  authored on lane `8`, force a visible error.
- Valid authored note lanes are `1` through `7`.
- Every judgeable note for the local player starts from the opponent judgment
  line.
- By default, every note bounces off a wall one or two times before falling to
  the player side.
- Non-EX notes fall to random valid lanes by default. Their authored lanes are
  still parsed and validated, but must not lock the final lane.
- EX notes use the authored lane as their fixed final lane.
- The user must be able to configure the common fall-time setting from `100` to
  `1000` in steps of `50`.
- Interpret the fall-time setting as `setting / 100` seconds, so `100` is 1
  second and `1000` is 10 seconds.
- All notes use the same fall duration. Compute movement speed from each note's
  path distance, so longer paths move faster and shorter paths move slower.

## Mobile Unity Scene Rules

- The project is intended to be played by tapping on mobile devices.
- Author gameplay scenes around a portrait mobile Game View, preferably `9:16` or
  fixed `1080 x 1920` while editing.
- Prefer an orthographic 2D camera for gameplay scenes.
- Treat the whole phone screen as the play field.
- Map touch input to seven horizontal lanes across the screen width.
- Keep the internal judgment area near the player side, but do not require a
  visible judgment-line graphic.
- Use multi-touch input because HOLD notes and simultaneous notes can exist.
- Runtime input and layout code must account for Android and iOS aspect ratios
  instead of relying on editor-only pixel positions.

## General Simai Model

- simai notation is plain text for maimai charts.
- Use half-width ASCII symbols for syntax.
- Whitespace, line breaks, and tabs inside chart notation are formatting only and
  should be ignored by parsers.
- A chart is a sequence of note groups separated by commas.
- Each comma advances time by the current per-comma duration.
- A chart should end with `E`.
- The `first` metadata value, when present, offsets the first chart timing from
  audio playback start in seconds.

## Button And Sensor Coordinates

Buttons are numbered clockwise:

```text
8 1
7 2
6 3
5 4
```

- Button `1` is at roughly 1 o'clock, then numbering proceeds clockwise to `8`.
- Touch sensors use groups `A`, `B`, `C`, `D`, and `E`.
- `A`, `B`, `D`, and `E` have numbered areas `1` through `8`.
- `C` is the center. `C`, `C1`, and `C2` should be treated as equivalent for
  TOUCH placement.

## Timing Controls

- BPM is written as `(BPM)`, for example `(174)` or `(120.5)`.
- The per-comma note divider is written as `{divider}`, for example `{4}` for
  quarter-note commas and `{16}` for sixteenth-note commas.
- When BPM and divider are both written together, BPM must come first:
  `(174){16}`.
- Per-comma seconds can be set directly with `{#seconds}`, for example
  `{#0.35}`.
- For BPM `B` and divider `T`, per-comma duration is `240 / B / T` seconds.
- BPM and divider may change anywhere in the chart.

## Length Syntax

HOLD and SLIDE durations use bracketed length syntax.

- `[divider:count]` means `count` units of the note length `divider`.
- `[#seconds]` means an exact duration in seconds.
- `[bpm#divider:count]` means a duration based on the specified BPM.
- SLIDE also supports separate wait and travel duration forms:
  `[bpm#seconds]`, `[waitSeconds##travelSeconds]`,
  `[waitSeconds##divider:count]`, and `[waitSeconds##bpm#divider:count]`.

## TAP

- TAP: `1,`, `5,`
- BREAK TAP: add `b`, for example `1b,`
- EX TAP: add `x`, for example `1x,`
- BREAK and EX can combine, for example `5bx,`
- Project rule: BREAK TAP and EX BREAK TAP are unsupported and must raise visible
  errors.

## HOLD

- HOLD: button + `h` + length, for example `5h[2:1],`
- Exact seconds: `4h[#5.678],`
- Specific BPM duration: `4h[150#2:1],`
- BREAK HOLD: add `b`, for example `5hb[2:1],`
- EX HOLD: add `x`, for example `3hx[2:1],`
- EX BREAK HOLD combines `x`, `b`, and `h`, for example `7bxh[2:1],`
- When `h`, `b`, and `x` appear together, their order is not significant.
- A HOLD without a bracketed length, such as `3h,`, is a pseudo TAP-like HOLD
  and is treated internally like `[1280:1]`.

## SLIDE

Basic SLIDE form:

```text
start shape end [duration]
```

Example:

```text
1-4[8:3],
```

- Project rule: every SLIDE notation is unsupported and must raise a visible
  unsupported-note error when syntactically valid.
- The starting TAP becomes a star-shaped TAP by default.
- After the star-shaped TAP reaches the judgment line, the tracing star waits
  one beat at the current BPM before moving, unless explicit wait timing is
  specified.
- SLIDE travel speed is constant from start to end.
- BREAK SLIDE: add `b` after the duration bracket, for example
  `1-4[8:3]b,`.

Supported shapes:

- `-`: straight line
- `>` / `<` / `^`: outer arc. Use `>` for clockwise/rightward travel, `<` for
  counterclockwise/leftward travel, and `^` when the distance is less than half
  the circle and direction does not need to be distinguished.
- `v`: small V through the center
- `p` / `q`: p/q curved shapes around the center
- `s` / `z`: zigzag shapes
- `pp` / `qq`: larger p/q curves
- `V`: large V with an explicit middle button, for example `1V35`
- `w`: fan shape from one start to three ends

Some shape/start/end combinations are invalid in official simai. Validate them
against the shape relationship table from the source pages when implementing
strict validation.

## Multiple And Chained Slides

Multiple SLIDE from the same starting star:

```text
1-4[4:3]*-6[8:5],
```

- Project rule: multiple SLIDE and chained SLIDE are unsupported and must raise a
  visible unsupported-note error when syntactically valid.
- Write the starting button once.
- Use `*` before additional slide tracks.
- Each track may have different timing.
- The tracks are treated as EACH because they start moving at the same time.

Chained SLIDE:

```text
1-4q7-2[1:2],
```

- Write a sequence of shapes and button numbers, then one duration at the end.
- The whole chain is one slide path with constant speed.
- If per-segment durations are used, every segment must have its own duration:
  `1-4[2:1]q7[2:1]-2[1:1],`
- A chained BREAK SLIDE can only be entirely BREAK or entirely normal. Put `b`
  only after the final duration bracket.

## TOUCH And TOUCH HOLD

- Project rule: TOUCH and TOUCH HOLD are unsupported and must raise a visible
  unsupported-note error when syntactically valid.
- TOUCH: sensor name, for example `B1,`, `D4,`, or `C,`
- `C1,` and `C2,` should parse as center `C,`
- TOUCH HOLD: sensor + `h` + length, for example `Ch[4:3],` or `E1h[4:3],`
- TOUCH HOLD can be represented on any sensor in simai notation. Japanese source
  notes that non-center TOUCH HOLD was added officially from PRiSM PLUS.
- TOUCH HOLD without a bracketed length, such as `Ch,`, is a pseudo TOUCH-like
  TOUCH HOLD and is treated internally like `[1280:1]`.
- Firework effect: add `f`, for example `B7f,` or `Chf[1:2],`
- For TOUCH HOLD with firework, `hf` and `fh` are equivalent.

## EACH

- Simultaneous notes are written with `/`, for example `1/8h[2:1],`
- Three or more simultaneous notes are written the same way:
  `noteA/noteB/noteC,`
- Order usually does not affect behavior, but for simultaneous SLIDEs the earlier
  written slide is drawn in front.
- TAP-only EACH may omit `/`, for example `12,`
- If any element is not a normal TAP, or if BREAK is involved, keep `/`.
- SLIDEs count as EACH when their tracing stars start moving at the same time,
  even if their travel durations differ.
- Project rule: EACH syntax itself is not unsupported. Only unsupported note
  members inside an EACH group should raise unsupported-note errors.

## EX Notes

- TAP, HOLD, BREAK TAP, and BREAK HOLD can be EX in official simai.
- Add `x` similarly to `b`.
- Examples: `1x,`, `3hx[2:1],`, `5bx,`, `7bxh[2:1],`
- When `x`, `h`, and `b` are combined, their order is not significant.
- Project rule: EX TAP, EX HOLD, and EX BREAK HOLD are supported. EX BREAK TAP is
  unsupported because BREAK TAP is unsupported.

## Special Notations

- Force normal TAP into star-shaped TAP: add `$`, for example `1$,`
- Rotating star TAP: use `$$`, for example `1$$,`
- Force a SLIDE starting star back into a normal TAP: add `@`, for example
  `1@-5[8:1],`
- `$`, `@`, `b`, and `x` combinations are order-insensitive within the valid
  note type.
- Pseudo EACH: use backtick to create 1 ms offsets, for example ``1`2,``.
- In ``1`2`3/4,``, `2` is 1 ms after `1`, and `3/4` is another 1 ms later.
- Pseudo EACH notes are not true EACH and should not become yellow EACH notes or
  count as EACH in results.
- SLIDE without approaching star:
  - `1?-5[2:1],` removes the approaching star but fades in the tracing star.
  - `1!-5[2:1],` removes the approaching star and makes the tracing star appear
    only when movement begins.
  - In both forms, the slide arrow track still fades in.
- Project rule: star TAP, rotating star TAP, and SLIDE-specific special forms are
  unsupported note features and must raise visible errors when syntactically
  valid. Pseudo-EACH is timing syntax, not an unsupported note by itself.

## Parser Notes

- Treat note modifiers documented as order-insensitive as normalized sets.
- Keep the original order for rendering when order affects visual stacking,
  especially simultaneous SLIDEs.
- Preserve enough source-span information to report syntax errors near the
  original text after whitespace is ignored.
- Do not silently accept undocumented syntax in strict mode.
- Prefer explicit error messages for invalid shape/end combinations, missing
  durations, partially timed chained slides, and malformed BPM/divider blocks.
- In project validation, separate syntax errors from project unsupported-note
  errors. A syntactically invalid note should report syntax failure, while a
  syntactically valid but unsupported note should report unsupported note type.
- In project validation, lane `8` is a project error for supported notes and must
  be visible to the user.
