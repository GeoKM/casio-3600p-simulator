# SPEC: CASIO fx-3600P Programmable Scientific Calculator

## 1. Project Overview

**Name:** CasioPy — fx-3600P Simulator
**Type:** Interactive web app (single HTML file)
**Summary:** A faithful recreation of the CASIO fx-3600P programmable scientific calculator from the 1980s, running in the browser. Includes authentic keystroke-level programmability, LCD display simulation, and the distinctive beige/grey livery of the era.
**Target users:** Retro computing enthusiasts, students, calculator collectors

---

## 2. Visual & Rendering Specification

### Calculator Body

- **Outer shell:** Beige/grey plastic texture resembling the original fx-3600P
  - Colour: `#D4D0C4` (main body), `#B8B4A8` (darker panels)
  - Subtle gradient/shadow to simulate curved plastic
  - Rounded corners (~12px) matching the original
  - Embossed "CASIO" text in the top-left of the shell
  - "fx-3600P" model label below brand name
  - "SCIENTIFIC CALCULATOR" sub-label

### Display

- **Type:** Custom CSS LCD simulation (no canvas needed)
- **Screen background:** Faint greenish-grey LCD colour `#B8D0A0` with a subtle grid of pixels
- **Display segments:** Dark green-grey `#4A5A3A` for active digits, faint ghost `#7A8A6A` for segments in "off" state
- **Layout:**
  - Main readout: 10-digit mantissa + 2-digit exponent (same as original)
  - Mode indicators on the left: `DEG`, `RAD`, `GRAD`, `LRN` (program mode), `STAT`, `SD`
  - Small floating point indicator `·` between mantissa and exponent when in scientific notation
- **Display angle:** Slightly recessed behind a dark plastic bezel

### Keyboard Layout

**SHELL LAYOUT — TOP ROW:**
`[  SHIFT  ][  MODE  ][  2ndF  ][  K   ][    ]`

**KEYBOARD GRID — 5 columns × 8 rows of keys:**

Row 1:  `AC`  `C`  `(-)`  `ENG`  `OFF`
Row 2:  `7`   `8`  `9`   `÷`   `DEL`
Row 3:  `4`   `5`  `6`   `×`   `INS`
Row 4:  `1`   `2`  `3`   `−`   `HLT`
Row 5:  `0`   `.`  `EXP` `+`   `ENT`
Row 6:  `1/x` `x²` `√x` `↑`   `(`
Row 7:  `π`   `x!` `%`   `DRG`  `)`
Row 8:  `sin` `cos` `tan` `log` `ln`

**SECONDARY FUNCTIONS (orange text, activated by SHIFT):**
- `sin⁻¹` `cos⁻¹` `tan⁻¹` `10ˣ` `eˣ`

**THIRD FUNCTIONS (green text, activated by 2ndF):**
- `nCr` `nPr` `←` `→` `[  ]`

**MODE BUTTON SUB-MENU (cycling through modes):**
- Mode A: `DEG` (degrees)
- Mode B: `RAD` (radians)
- Mode C: `GRAD` (gradians)
- Mode D: `STAT` (statistics)
- Mode E: `LRN` (program learning — keystroke recording)
- Mode F: `SD` (single-variable statistics)

**BOTTOM ROW — PROGRAM KEYS:**
`[  P1  ][  P2  ][  P3  ][  x→M  ][  x↔M  ]`

### Button Styling

- **Shape:** Rectangular with slightly rounded top, as per the original
- **Size:** Approximately 42×28px each
- **Font:** Bold sans-serif for primary labels
- **Primary label:** Dark grey `#2A2A2A` centre-aligned
- **Orange SHIFT functions:** Orange `#C84B00` in top-left corner of key
- **Green 2ndF functions:** Dark green `#2A5A2A` in top-right corner of key
- **Pressed state:** Button sinks ~2px, slightly darker shade
- **Hover state:** Subtle brightness increase
- **Click sound:** Short, crisp beep (Web Audio API oscillator, ~800Hz, 30ms, square wave)

### Color Palette

| Element | Colour |
|---------|--------|
| Calculator body | `#D4D0C4` |
| Darker trim | `#B8B4A8` |
| Display bezel | `#3A3A38` |
| LCD background | `#B8D0A0` |
| LCD active digit | `#4A5A3A` |
| LCD ghost digit | `#7A8A6A` |
| Key face | `#E8E8E0` |
| Key pressed | `#D0D0C8` |
| Key text primary | `#2A2A2A` |
| Key text SHIFT | `#C84B00` |
| Key text 2ndF | `#2A5A2A` |
| Brand label | `#4A4A48` |

### Animations

- Button press: CSS `transform: translateY(2px)` + box-shadow reduction, 60ms
- Mode indicator transitions: 150ms fade
- Power-on: LCD segments fade in over 300ms
- Power-off: LCD fade out over 200ms

---

## 3. Simulation Specification

### Display States

- **Power on:** Shows `0.` (resting state)
- **Power off:** All segments dark (no display update)
- **Normal calculation:** Up to 10 digits mantissa, exponent shown when |x| ≥ 1e10 or |x| < 1e-4
- **Error state:** Display reads `Error` with blinking effect

### Calculation Model

- **Precision:** JavaScript `number` (IEEE 754 double), 15 significant digits
- **Arithmetic:** Standard binary operations with operator precedence (left-to-right for same precedence, except exponentiation which is right-associative)
- **Functions:** sin, cos, tan, sin⁻¹, cos⁻¹, tan⁻¹, log₁₀, ln, eˣ, 10ˣ, √x, x², 1/x, x!, nCr, nPr, ENG notation
- **Constants:** π (= Math.PI), e (= Math.E)
- **Angle modes:** DEG (°), RAD, GRAD — trig functions respect mode setting

### Memory

- **M register:** One independent storage register
- **K register:** Last intermediate result (constant memory)
- **Operations:** `x→M` (store), `x↔M` (swap), `MR` (recall), `MC` (clear memory)
- **Display memory:** Auto-display after each operation

### Programmability (fx-3600P Keystroke Level)

**Program Slots:** P1, P2, P3 — each holds up to 38 steps

**Step Types (each keypress = 1 step):**
- Digits and decimal point
- Operations: `+`, `−`, `×`, `÷`, `=`
- Functions: sin, cos, tan, log, ln, √x, x², 1/x, etc.
- ENT (input) — pauses program, prompts user for a value
- HLT (halt) — pauses program, waits for user to press `=` to continue
- DEL (delete last step in program edit)
- INS (insert mode for program edit)

**Program Control:**
- `LBL 0`–`LBL 9` — set a label at a program step
- Conditional jumps: `x=0`, `x≥0`, `x>0`, `x<0`, `x≤0` — jump to label if true
- Unconditional jump: can use `x=0` with accumulator = 0, or use INV to invert condition

**Recording (LRN mode):**
1. Press `MODE E` to enter LRN
2. Press `P1`/`P2`/`P3` to select program slot
3. Type the sequence of keys
4. Press `AC` to exit LRN and save program

**Playback:**
- Press `P1`/`P2`/`P3` to execute the program
- ENT prompts for input (display shows `0.` and blinks cursor)
- HLT pauses; press `=` to continue
- After completion, display shows result

**Program Memory Display:**
- In LRN mode, a 2-digit step counter shows `00`–`38`
- Steps used shown as small dots or a progress bar below the main display

**Programming Aids:**
- `INV` key (SHIFT + `HLT`): Inverts the next conditional (e.g., `INV` `x=0` = `x≠0`)
- `HLT` (Halt): Pause and wait for user `=`
- `ENT` (Enter): Pause for keyboard input

---

## 4. Interaction Specification

### Mouse / Touch

- All keys clickable via mouse
- Keys respond to `mousedown` (press) and `mouseup` (release)
- Touch-friendly: keys are large enough for finger taps on mobile

### Keyboard Mapping

Map calculator keys to physical keyboard:

| Calculator Key | Keyboard |
|---------------|----------|
| `0`–`9` | `0`–`9` |
| `.` | `.` |
| `+` | `+` |
| `−` | `-` |
| `×` | `*` |
| `÷` | `/` |
| `=` | `Enter` |
| `AC` | `Escape` |
| `C` | `Backspace` |
| `EXP` | `E` |
| `(-)` | `)` |
| `ENT` | `Enter` (in program) |
| `HLT` | `H` |
| `DEL` | `Delete` |
| `INS` | `Insert` |
| `sin` | `S` |
| `cos` | `C` (conflict — use Shift+S) |
| `tan` | `T` |
| `log` | `L` |
| `ln` | `N` |
| `SHIFT` | `Shift` |
| `2ndF` | `Ctrl` |
| `MODE` | `M` |
| `P1` | `F1` |
| `P2` | `F2` |
| `P3` | `F3` |
| `x→M` | `M` (when not in program) |
| `x↔M` | `X` |
| `DEG` | `D` |
| `RAD` | `R` |
| `GRAD` | `G` |

### Audio

- **Key click:** Square wave oscillator, 800Hz, 30ms, very short decay envelope. Volume: low
- **Error beep:** 200Hz, 150ms, sawtooth wave
- **Power-on chime:** Two-tone ascending beep (400Hz → 600Hz)
- Audio can be muted via small speaker icon in corner

---

## 5. Acceptance Criteria

1. ✅ Calculator powers on showing `0.` on the LCD display
2. ✅ All digit keys (0–9) and decimal point work correctly
3. ✅ All four basic operations (+, −, ×, ÷) function with correct precedence
4. ✅ Scientific functions work: sin, cos, tan, log, ln, √x, x², 1/x, x!, π, e
5. ✅ Inverse trig functions (SHIFT + trig) work
6. ✅ ENG notation (engineering exponent) toggles correctly
7. ✅ Angle mode cycles DEG → RAD → GRAD → DEG and affects trig calculations
8. ✅ Memory operations (x→M, x↔M, MR, MC) work correctly
9. ✅ STAT mode activates single-variable statistics (mean, std dev, etc.)
10. ✅ LRN mode allows recording a program into P1, P2, or P3
11. ✅ Programs execute correctly when P1/P2/P3 pressed
12. ✅ ENT pauses program for input, HLT pauses for continue
13. ✅ Conditional jumps (x=0, x≥0, etc.) work in programs
14. ✅ DEL and INS edit functions work in LRN mode
15. ✅ LCD display renders with authentic LCD green colour and ghost segments
16. ✅ Button press animations and audio feedback work
17. ✅ Keyboard shortcuts map correctly
18. ✅ Display shows error state appropriately
19. ✅ Display shows scientific notation (EXP) when numbers are large/small
20. ✅ Power off (OFF key) blanks the display
