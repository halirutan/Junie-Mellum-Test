# Implementation Plan — Native Calculator App

## Milestone 1: Project Setup and Dependency Installation

**Goal:** Install `dearpygui==2.3.1` and verify the project runs.

**Commands:**
```bash
uv add dearpygui==2.3.1
uv run python calculator/main.py
```

**Expected outcome:** `pyproject.toml` updated with `dearpygui` dependency; the stub `calculator/main.py` runs without error.

**Status:** Not Started

---

## Milestone 2: Calculator Logic Module (`calculator/calculator.py`)

**Goal:** Implement pure Python calculator logic, independent of any GUI framework.

**File to create:** `calculator/calculator.py`

**Class:** `Calculator`

**State variables:**
- `entry` (str): Current display string, starts as `"0"`.
- `stored_operand` (Decimal | None): First operand for pending operation.
- `pending_operator` (str | None): One of `"+", "−", "×", "÷"`.
- `awaiting_entry` (bool): Whether the next digit starts a new entry.
- `last_operation` (tuple | None): `(operator, right_operand)` for repeated `=`.
- `error` (bool): Whether an error state is active.

**Methods:**
- `digit(d: str)` — Append digit; overwrite initial `0` or pending entry.
- `decimal()` — Add decimal point; allow only one.
- `operator(op: str)` — Store operand, set pending operator; left-to-right chaining.
- `equals()` — Evaluate pending operation; support repeated `=`.
- `clear()` — Reset all state.
- `negate()` — Negate current entry.
- `percent()` — Divide current entry by 100.
- `backspace()` — Remove last character; return to `0` if empty.

**Arithmetic rules:**
- Use `decimal.Decimal` with precision 28.
- Format results: at most 12 significant digits, remove trailing zeros.
- Scientific notation for overflow beyond 12 digits.
- Normalize negative zero to `0`.
- Division by zero shows `"Error"`, blocks input except `C` or digit.
- Left-to-right chaining, no operator precedence.

**Delegation:** Team-lead delegates creation of `calculator/calculator.py` to worker with full method signatures and behavior spec.

**Status:** Not Started

---

## Milestone 3: Unit Tests (`tests/test_calculator.py`)

**Goal:** Verify calculator logic covers all acceptance criteria.

**Files to create:** `tests/` directory, `tests/test_calculator.py`

**Test cases:**
1. Basic arithmetic: `12.5 × 2 = 25`
2. Division by zero: `8 ÷ 0 = Error`
3. Repeated equals: `2 + 3 = = 8`
4. Percent: `50 % = 0.5`
5. Backspace: `9 ⌫ = 0`
6. Left-to-right chaining: `2 + 3 × 4 = 20`
7. Clear resets all state
8. Negate flips sign
9. Decimal point handling (single point, no redundant zeros)
10. 12-digit limit enforcement

**Command:**
```bash
uv run pytest tests/test_calculator.py -v
```

**Status:** Not Started

---

## Milestone 4: GUI — Viewport, Display, and Keypad Layout (`calculator/main.py`)

**Goal:** Build the Dear PyGui desktop window with exact pixel-perfect layout.

**File to modify:** `calculator/main.py`

**Viewport:**
- Client area: 360×480 logical pixels, fixed size, centered.
- Title: `Calculator`.

**Primary window:**
- Position `(0, 0)`, size `(360, 480)`.
- No title bar, no scrolling, no padding.

**Display region:**
- Rectangle `(16, 20, 328, 124)`, background `#242932`, corner radius 10.
- History text: `(24, 36, 312, 22)`, 15px, muted (`#A9B3C2`), right-aligned.
- Entry text: `(24, 75, 312, 48)`, 34px, white (`#F7F9FC`), right-aligned.

**Keypad — 20 buttons, 76×52 each, corner radius 8, font 22px:**

| Row (y) | x=16 | x=100 | x=184 | x=268 |
|---------|------|-------|-------|-------|
| 164     | C    | ±     | %     | ÷     |
| 225     | 7    | 8     | 9     | ×     |
| 286     | 4    | 5     | 6     | −     |
| 347     | 1    | 2     | 3     | +     |
| 408     | 0    | .     | ⌫     | =     |

**Palette:**
- Window: `#171A20`
- Digit buttons: `#303640`
- Function buttons (C, ±, %, ⌫): `#48515E`
- Operator buttons (+, −, ×, ÷): `#3E5878`
- Equals: `#2679D9`
- Primary text: `#F7F9FC`

**Delegation:** Team-lead delegates GUI layout to worker with exact coordinates and colors.

**Status:** Not Started

---

## Milestone 5: GUI — Event Handling, Keyboard Mapping, and Integration

**Goal:** Wire all buttons and keyboard to the `Calculator` class, update display, handle visual feedback.

**File to modify:** `calculator/main.py`

**Button wiring:**
- All 20 buttons call corresponding `Calculator` methods.
- After each call, update history and entry display text.

**Keyboard mapping:**
- `0`–`9` → digit buttons
- `.` → decimal
- `+`, `-`, `*`, `/` → operators
- `Enter`/`=` → equals
- `Backspace` → backspace
- `Escape` → clear
- `%` → percent
- `±` is mouse-only

**Display update:**
- On every state change, refresh history and entry text.
- Right-align text within display rectangles.
- Clip text that exceeds rectangle bounds.

**Visual feedback:**
- Hover: lighten button fill by 10%.
- Pressed: darken button fill by 10%.

**Verification:**
- Launch app: `uv run python calculator/main.py`
- Test all acceptance criteria: `12.5 × 2 = 25`, `8 ÷ 0 = Error`, `2 + 3 = = 8`, `50 % = 0.5`, `9 ⌫ = 0`, `2 + 3 × 4 = 20`.

**Status:** Not Started
