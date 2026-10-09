# Native Calculator App Specification

## 1. Stack and scope

- **Runtime:** CPython 3.14 (64-bit).
- **GUI:** `dearpygui==2.3.1` (`import dearpygui.dearpygui as dpg`). Desktop application using Dear PyGui's native viewport, **not** a browser, server, or web view.
- **Other dependencies:** Python standard library only (`decimal` for arithmetic). Install with `python -m pip install dearpygui==2.3.1`; start with `python calculator/main.py`.
- **Platforms:** Windows x86-64, Linux x86-64/ARM64, macOS ARM64, where the published Python 3.14 wheels are available. No network access, accounts, persistence, or scientific functions.

## 2. Window and coordinate system

- Title: `Calculator`. **Viewport client area: 360 × 480 logical pixels**, fixed size (no user resize), initially centered; OS title bar and borders are outside the specified client area.
- One primary Dear PyGui window at `(0, 0)`, size `(360, 480)`, with no title bar, scrolling, movement, resize handles, or content padding. Set it as the primary window and disable viewport resize. Ensure the drawable client area, not the outer OS frame, measures 360 × 480.
- All rectangles below use **client-relative** `(x, y, width, height)` coordinates, origin at top left, in logical pixels. Apply no implicit container spacing. For high-DPI displays preserve these logical dimensions and scale rendering as needed.

## 3. Exact layout

**Display region:** `(16, 20, 328, 124)`, background `#242932`, corner radius 10. Draw expression/history in the rectangle `(24, 36, 312, 22)`, 15 px muted text, right-aligned; draw current entry/result in `(24, 75, 312, 48)`, 34 px white text, right-aligned. Right text edge at `x=336`. Start with history empty and value `0`. Clip or shorten values that exceed their rectangle; never overlap the keypad.

**Keypad:** 4 columns × 5 rows; every button measures `76 × 52`, corner radius 8. Column `x` coordinates: **16, 100, 184, 268**. Row `y` coordinates: **164, 225, 286, 347, 408**. Thus horizontal gaps are 8 px, vertical gaps 9 px, and bottom margin is 20 px. Center each label; label font size 22 px.

| Row (`y`) | x=16 | x=100 | x=184 | x=268 |
|---|---|---|---|---|
| 164 | C | ± | % | ÷ |
| 225 | 7 | 8 | 9 | × |
| 286 | 4 | 5 | 6 | − |
| 347 | 1 | 2 | 3 | + |
| 408 | 0 | . | ⌫ | = |

**Palette:** window `#171A20`; display `#242932`; digit buttons `#303640`; function buttons (C, ±, %, ⌫) `#48515E`; operator buttons `#3E5878`; equals `#2679D9`; primary text `#F7F9FC`; history text `#A9B3C2`. Hover: lighten button fill by 10%; pressed: darken by 10%. Use a bundled or system font supporting `±`, `÷`, `×`, `−`, and `⌫`; fall back to an equivalent glyph if unavailable. No toolbars, menu bars, animations, or extra panels.

## 4. Behavior

- **Model:** maintain `entry` (display string), `stored_operand` (`Decimal | None`), `pending_operator` (`+, −, ×, ÷ | None`), `awaiting_entry` (bool), `last_operation` (operand/operator pair for repeated `=`), and `error` (bool). Never use `eval()` or binary floats for arithmetic.
- **Digits and decimal point:** overwrite initial `0` or a pending new entry; otherwise append. Allow one decimal point and up to 12 significant input digits. `0` cannot acquire redundant leading zeros. Preserve a trailing decimal point while editing.
- **Operators:** selecting `+`, `−`, `×`, or `÷` stores the current operand and waits for the next entry. If an operation is already pending and the second operand exists, compute it immediately, then store the chosen next operator (**left-to-right chaining, no precedence**). Selecting another operator before the next entry replaces the pending operator.
- **Equals:** evaluate the pending operation and display the result. Repeating `=` reapplies the last operator with the last right operand (e.g. `2 + 3 = =` gives `8`). If no operation is pending, `=` does nothing.
- **Unary keys:** `±` negates the current entry; `%` divides the current entry by 100 (e.g. `50 %` yields `0.5`); `⌫` removes the last entered character, returning to `0` if empty. `C` resets all state, including errors and repeated-equals memory.
- **Output:** use `decimal.Decimal` with precision 28 and format results with at most 12 significant digits, removing unnecessary trailing zeros. Use scientific notation for magnitudes that cannot fit the display. Normalize negative zero to `0`.
- **Errors:** division by zero or invalid/overflowing results show `Error`, clear pending operations, and block input except `C` or a digit (a digit starts a fresh calculation). The app must never terminate because of arithmetic input.
- **History:** when an operator is pending, show `<left operand> <operator>`; following `=`, show `<left operand> <operator> <right operand> =`; clear history on `C` or new standalone calculation.

## 5. Interaction and implementation

- Mouse: all 20 buttons are clickable; no keyboard focus is required after launch.
- Keyboard: `0`–`9`, `.`, `+`, `-`, `*`, `/`, `Enter`/`=`, `Backspace`, `Escape` (`C`), and `%` map to their buttons. Map `-` to subtraction; the `±` action is mouse-only. Ignore unsupported keystrokes.
- Construct the viewport, primary window, background/display drawing, and buttons explicitly. Use `pos=(x,y)`, `width=76`, and `height=52` for every button; do not depend on automatic layout. Use a draw layer or drawlist with measured text widths to right-align display text, then update it on state changes. Keep calculator logic independent of Dear PyGui so it can be unit-tested.
- Proposed files: `calculator/main.py` (GUI and event mappings), `calculator/calculator.py` (state and arithmetic), `tests/test_calculator.py` (logic tests), `requirements.txt` (pinned dependency).

## 6. Acceptance criteria

1. Launching `python calculator/main.py` opens one desktop window with the specified 360 × 480 client area and exact button coordinates; no browser or network service starts.
2. Mouse and keyboard produce identical arithmetic results for shared actions: `12.5 × 2 = 25`, `8 ÷ 0 = Error`, `2 + 3 = = 8`, `50 % = 0.5`, `9 ⌫ = 0`.
3. `2 + 3 × 4 = 20` confirms left-to-right chaining; `C` always restores the initial state.
4. All controls remain visible and clickable without clipping or overlap at normal display scaling, and text stays inside its assigned display rectangle.

**Dependency verification:** [Dear PyGui on PyPI](https://pypi.org/project/dearpygui/2.3.1/) lists CPython 3.14 wheels and the supported desktop platforms (checked October 2026).
