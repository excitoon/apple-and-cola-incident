# Apple and Cola Incident

## Incident Description

On March 18, a Coca-Cola Zero was spilled over an Apple MacBook Pro M3 Pro. The owner, unfamiliar with macOS, attempted to power it off but accidentally put it to sleep instead. The laptop was then gently wiped, a napkin was placed between the screen and keyboard, and it was positioned upside down for approximately 30 minutes to allow liquid to drain.

After waiting, the MacBook did not turn on, so it was taken to a service center. The service center was able to power it on, at which point it was immediately shut down again. The technicians spent about 2 hours cleaning the device.

## Device Identification

| Field | Value |
|---|---|
| Serial number | MWJPXQ4VC4 |
| Model | MacBook Pro 14-inch (M3 Pro, November 2023) |
| Model number | A2918 |
| EMC number | EMC 8304 |
| Logic board | 820-02757 |
| Processor | Apple M3 Pro (11-core CPU / 14-core GPU) |
| Display | 14.2" Liquid Retina XDR, 3024 × 1964 |
| Keyboard type | Scissor-switch (Magic Keyboard), integrated into top-case assembly |

The serial prefix **MWJ** identifies a 2024-batch MacBook Pro 14" with M3 Pro chip. The logic board number **820-02757** is the key identifier for locating component-level schematics and boardview files (see [Board-Level Schematic References](#board-level-schematic-references) below).

## Observed Symptoms

Upon receiving the device back from the service center:

- Key **N** did not work.
- The service center indicated keyboard replacement would be required, citing a "mechanical issue" — later clarified to mean that the individual key switch blocks are sealed units that cannot be manually cleaned; residue trapped inside them cannot be reached by conventional methods.
- After returning home, the entire column containing **H**, **Y**, **6**, and **N** was affected.
- **N** was producing incorrect characters (a **/** symbol and other unintended characters) instead of or in addition to `N`.
- Extra symbols were appearing spontaneously from the keyboard without any keys being pressed.
- Over the following days the picture evolved: **N** now produces 3 different symbols (including `N` itself), and similar multi-character behavior is present for the other keys in the affected column.
- **Ghost keypresses subsequently disappeared** — spontaneous phantom key events stopped after the residue dried and stabilised.
- **Partial improvement after rest period** (updated March 22): after 2 days of non-use (internal keyboard disabled via Karabiner Elements while using an external keyboard), the affected keys now produce **correct symbols plus two incorrect ones** (previously, correct symbols were intermittent or absent). This suggests the contamination may be partially dissipating or redistributing during periods without thermal cycling from laptop use.
- **Space bar also affected** — the Space key produces a space character followed by an apostrophe (`'`), confirming contamination extends beyond the original 6/Y/H/N column.

### Keyboard Diagnostic Output (raw `cat` test, March 22)

Each affected key was pressed twice using `cat` to capture the raw output. The results below show the exact characters produced per keypress:

| Key pressed | Expected | Actual output (per press) | Extra characters |
|---|---|---|---|
| **6** | `6` | `690` | `9`, `0` |
| **Y** | `y` | `yop` | `o`, `p` |
| **H** | `h` | `hl;` | `l`, `;` |
| **N** | `n` | `n./` | `.`, `/` |
| **Space** | ` ` | ` '` | `'` |

Raw `cat` transcript (each key pressed twice):
```
690
690
yop
yop
hl;
hl;
n./
n./
```

Space:
```
 '
 '
```

#### Pattern Analysis

Mapping the affected keys and their extra characters onto the physical keyboard layout reveals a consistent pattern — the two extra characters come from keys that are **3 and 4 physical positions to the right** in the same row:

```
  Physical keyboard (affected keys in [brackets], extra outputs in {braces})
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │  5  │ [6] │  7  │  8  │ {9} │ {0} │  -  │  =  │  ← 6 → 690
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  │  T  │ [Y] │  U  │  I  │ {O} │ {P} │  [  │  ]  │  ← Y → yop
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  │  G  │ [H] │  J  │  K  │ {L} │ {;} │ {'}*│Enter│  ← H → hl;
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  │  B  │ [N] │  M  │  ,  │ {.} │ {/} │     │     │  ← N → n./
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
                          Space → ' (apostrophe)
                          * ' is in the same column as 0, P, /
```

This means the affected column C7 (6/Y/H/N) is bridging to **two specific matrix columns**, not to the immediately adjacent columns C6/C8 as initially hypothesized. The bridged columns contain:
- **Bridged column A**: keys 9, O, L, `.` (period)
- **Bridged column B**: keys 0, P, `;`, `/`

The Space → `'` output is also consistent: the apostrophe `'` sits in the same physical column as `0`, `P`, and `/` — the same "bridged column B" identified above. This strongly suggests the **Space bar's matrix trace** also crosses or runs near the contamination site and is picking up a bridge to the same column B trace.

#### Revised Matrix Column Bridge Map

```
  FPC cross-section (schematic, updated with diagnostic data)

  ┌────────────────────────────────────────────────────────────────────┐
  │  Polyimide substrate                                               │
  ├──────────┬──── ··· ────┬──────────┬──────────┬──── ··· ────────────┤
  │  COL_7   │             │  COL_A   │  COL_B   │                     │
  │ (6,Y,H,N)│  ...cols... │ (9,O,L,.)│ (0,P,;,/)│                    │
  └──────────┼═════════════┼══════════┼══════════┤────────────────────-┘
             ║ dried cola  ║          ║          ║
             ║ residue     ╠══════════╣          ║
             ║ bridges     ║  bridge 1║          ║
             ╠═════════════╝          ║          ║
             ║            bridge 2    ║          ║
             ╚════════════════════════╝          ║
                                                 ║
                       ┌─────────────────────────╝
                       │ Space bar trace also bridges to COL_B
                       │ (produces ' in addition to space)
                       └──────────────────────────
```

**Key finding:** The conductive bridge spans **multiple columns**, not just one adjacent column. This indicates either:
1. The contamination site is at the **ZIF connector** where pins are packed at ~0.5 mm pitch — a large enough residue deposit could bridge across several pins simultaneously.
2. There is a **dendritic growth** (electrochemical migration) that has extended from the initial contamination point to reach non-adjacent pins, forming conductive "fingers" of tin/copper oxide.
3. The FPC trace routing is **non-sequential** — matrix columns C7, the column carrying 9/O/L/., and the column carrying 0/P/;/' may actually be routed on **adjacent or nearby FPC traces** even though they correspond to non-adjacent physical keyboard columns. This is common in keyboard FPC design where traces are reordered for routing efficiency.

Option 3 is the most likely explanation: the keyboard FPC does not route matrix columns in physical left-to-right order. The column traces for 6/Y/H/N, 9/O/L/., and 0/P/;/' are probably **physically adjacent on the FPC ribbon** even though they appear far apart on the keyboard surface. A single residue bridge at the connector or along the FPC can therefore affect non-adjacent physical columns.

This also explains why Space picks up `'` — the Space bar's matrix column trace likely runs near the same group on the FPC.

## Keyboard Mechanisms: Butterfly vs Scissor

Before diving into the electronics, it is worth clarifying the two keyboard mechanism types Apple has used, since the service center discussion references "non-serviceable key blocks":

### Butterfly mechanism (2015–2019, now discontinued)

The **butterfly mechanism** was Apple's ultra-thin key switch design used in the MacBook 12" (2015–2017) and MacBook Pro 13"/15" (2016–2019). It uses a **V-shaped pair of interlocking wings** (resembling a butterfly) that pivot at a single central point. When the keycap is pressed, the wings fold flat; when released, they spring back.

Key characteristics:
- **Extremely thin** — only ~0.55 mm key travel, enabling very slim laptop designs.
- **Fragile** — notorious for failures caused by dust, crumbs, or debris getting trapped under the mechanism, causing stuck or unresponsive keys.
- **Recalled** — Apple acknowledged the reliability problems and offered a free keyboard repair program for affected models.
- **Discontinued** — replaced by the scissor mechanism starting with the MacBook Pro 16" in late 2019.

> ⚠️ **Your MacBook (M3 Pro, 2023) does NOT use the butterfly mechanism.** It uses the scissor mechanism described below.

### Scissor mechanism (2019–present, your MacBook)

The **scissor mechanism** (marketed as "Magic Keyboard") is Apple's current keyboard design, used in all MacBooks since late 2019 including the M1, M2, and M3 generations. It uses an **X-shaped pair of interlocking plastic arms** (resembling scissors) that cross and pivot at the center. The arms clip into the keycap above and a base plate below, providing a stable, guided vertical motion.

Key characteristics:
- **More key travel** — approximately 1 mm, which feels more tactile than the butterfly design.
- **More reliable** — the X-shaped mechanism is less susceptible to dust and debris failures.
- **Still sealed** — the scissor arms, rubber dome, and FPC membrane layer form a sealed unit. Once liquid enters the sub-0.3 mm capillary gaps, it cannot be removed by manual cleaning — only by ultrasonic cavitation.
- **Integrated into the top-case** — the keyboard, battery, and palm rest are a single assembly. Replacing the keyboard means replacing the entire top-case.

See the [butterfly vs scissor comparison diagram](diagrams/butterfly-vs-scissor.svg) and [scissor-switch cross-section](diagrams/scissor-switch-cross-section.svg) for visual details.

## Keyboard Electronics: Schematics and Layout

### 1. Keyboard Matrix Principle

MacBook keyboards use a **row/column matrix** to detect keypresses. Each key sits at the intersection of one row wire and one column wire. The keyboard controller scans all rows one at a time and reads which column lines are active. This means:

- One **column** trace is shared by all keys in the same vertical column on the keyboard.
- One **row** trace is shared by all keys in the same horizontal row.
- A keypress is detected when a specific (row, column) pair is simultaneously active.

```
          COL_A   COL_B   COL_C   COL_D   COL_E   COL_F
           |       |       |       |       |       |
ROW_1 ---[Tab]---[Q]-----[W]-----[E]-----[R]-----[T]--- ...
           |       |       |       |       |       |
ROW_2 ---[CpsLk]-[A]-----[S]-----[D]-----[F]-----[G]--- ...
           |       |       |       |       |       |
ROW_3 ---[Shift]-[Z]-----[X]-----[C]-----[V]-----[B]--- ...
           |       |       |       |       |       |
          ...     ...     ...     ...     ...     ...
```

Each intersection square `[KEY]` contains a tiny rubber dome or scissor-switch membrane that, when pressed, electrically connects the row wire to the column wire at that point.

### 2. Affected Column in the Keyboard Matrix

The keys **6**, **Y**, **H**, and **N** all sit in the **same column** on the physical MacBook keyboard layout. They are connected to a single column trace that runs vertically through the keyboard membrane/FPC from top to bottom:

```
  Physical keyboard (excerpt of the affected area)
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │  5  │ [6] │  7  │  8  │  9  │  0  │  -  │  =  │  ← number row
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  │  T  │ [Y] │  U  │  I  │  O  │  P  │  [  │  ]  │  ← top letter row
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  │  G  │ [H] │  J  │  K  │  L  │  ;  │  '  │Enter│  ← home row
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  │  B  │ [N] │  M  │  ,  │  .  │  /  │     │     │  ← bottom row
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
           ▲
           │
     Affected column trace (shared by 6, Y, H, N)
```

A single conductive contamination or corrosion point anywhere along this column trace (or at the connector pin that carries it) will affect all four keys simultaneously.

### 3. Keyboard Flex Cable (FPC) and Connector

The MacBook keyboard is built into a large **Flexible Printed Circuit (FPC)** — a thin, ribbon-like PCB that carries all the row and column traces from the key matrix down to a connector at the bottom of the top-case assembly.

```
  Top-case assembly (schematic side view)
  ┌──────────────────────────────────────────────────┐
  │          K E Y B O A R D   K E Y S               │
  │   [key][key][key][key][key][key][key][key][key]   │
  │   [key][key][key][key][key][key][key][key][key]   │
  │   [key][key][key][key][key][key][key][key][key]   │
  │   [key][key][key][key][key][key][key][key][key]   │
  └────────────────────┬─────────────────────────────┘
                       │  FPC ribbon cable
                       │  (carries ~30–34 row/col traces)
                       │
              ┌────────┴────────┐
              │  ZIF Connector  │  ← Zero Insertion Force socket
              │  on logic board │    on top-case or directly on
              └────────┬────────┘    the logic board ribbon
                       │
              ┌────────┴────────┐
              │  Keyboard       │
              │  Controller IC  │  ← scans matrix, reports
              └─────────────────┘    keypresses over SPI/I2C
                                     to the Apple Silicon SoC
```

#### ZIF Connector Detail

A ZIF (Zero Insertion Force) connector has a row of gold-plated pads on the FPC that line up with spring contacts inside the socket. A locking latch clamps the cable in place. If the FPC is:

- **Inserted at a slight offset**: one or more pins shift, and a column or row trace connects to the wrong controller input pin — causing wrong characters.
- **Not fully inserted**: contact resistance is high or intermittent, causing missing or unreliable keypresses.
- **Inserted upside-down** (some cables are not keyed): the pin order is completely reversed, producing systematic wrong-character output across many keys.

```
  FPC edge (viewed from below, 34-pin example — simplified)

  ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
  │R1│R2│R3│R4│R5│R6│C1│C2│C3│C4│C5│C6│C7│C8│C9│..│Gnd│
  └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
   ▲                       ▲
   Row traces               Column traces
   (one per keyboard row)   (one per keyboard column)

  Each pad is ~0.5 mm wide with ~0.5 mm pitch — very easy to
  misalign by one position during re-insertion.
```

The following diagrams show what the physical connector looks like — both the socket on the logic board and the FPC ribbon end that plugs into it:

```
  ZIF socket on logic board (top view, latch open — ready to accept FPC)

        latch (open / raised)
        ╔═══════════════════════════════════════╗
        ║                                       ║
        ╠═══════════════════════════════════════╣  ← hinge line
        │ · · · · · · · · · · · · · · · · · · · │  ← spring contacts
        │_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _│  ← FPC insertion slot
        └───────────────────────────────────────┘
         ↑ FPC ribbon slides in here, flat side down

  ZIF socket on logic board (top view, latch closed — FPC clamped)

        latch (closed / rotated down, locks FPC)
        ┌───────────────────────────────────────┐
        │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│  ← latch pressed down
        │ · · · · · · · · · · · · · · · · · · · │  ← contacts gripping FPC pads
        │▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔│
        └───────────────────────────────────────┘

  Side cross-section (latch closed):

          FPC ribbon
         ┌────────────────────────────────────┐
         │ polyimide film │▐▌▐▌▐▌▐▌▐▌▐▌ pads  │  ← gold-plated copper pads
         └────────────────┴──────────────────-┘
                                 ↕ contact force from latch
         ┌───────────────────────────────────┐
         │  spring contacts inside socket    │  ← socket body, soldered to PCB
         └───────────────────────────────────┘
              ↑ logic board PCB
```

### 4. How Liquid Damage Causes the Observed Symptoms

The following diagram illustrates where cola can bridge traces and produce the observed multi-character and ghost-key behavior. Based on diagnostic data (see [Keyboard Diagnostic Output](#keyboard-diagnostic-output-raw-cat-test-march-22) above), pressing keys in the affected column produces characters from **two additional matrix columns** — the columns carrying keys 9/O/L/. and 0/P/;/'. These matrix columns are most likely routed on **physically adjacent FPC traces** despite corresponding to non-adjacent physical keyboard positions (a common FPC routing optimisation):

```
  FPC cross-section (schematic, updated with diagnostic data)

  ┌──────────────────────────────────────────────────────────────────────────┐
  │  Polyimide substrate (insulating base layer)                             │
  ├──────────────┬──────────────────┬──────────────────┬─────────── ··· ─────┤
  │  COL_C7      │   COL_Ca         │   COL_Cb         │   other cols       │
  │  (6,Y,H,N)   │  (9,O,L,.)       │  (0,P,;,/,')     │                   │  ← copper traces
  └──────────────┴──────────────────┴──────────────────┴─────────── ··· ─────┘
                   (physically adjacent on FPC despite being non-adjacent on keyboard)

  After cola spill and drying:

  ┌──────────────────────────────────────────────────────────────────────────┐
  │  Polyimide substrate                                                     │
  ├──────────────╬══════════════════╬══════════════════┬─────────── ··· ─────┤
  │  COL_C7      ║  dried cola +    ║   COL_Cb         │   other cols       │
  │  (6,Y,H,N)   ║  corrosion       ║  (0,P,;,/,')     │                   │
  │              ║  COL_Ca          ║                  │                    │
  │              ║  (9,O,L,.)       ║                  │                    │
  └──────────────╩══════════════════╩══════════════════┴─────────── ··· ─────┘
                 ▲                  ▲
                 Bridge 1: C7↔Ca    Bridge 2: C7↔Cb (or Ca↔Cb)
                 → pressing 6 also activates 9 and 0
                 → pressing Y also activates O and P
                 → pressing H also activates L and ;
                 → pressing N also activates . and /
                 → Space trace also bridges to Cb → produces '
```

### 5. Physical Location of the Connector on an M3 Pro MacBook

On the MacBook Pro M3, the keyboard FPC runs from the key area down to the **lower portion of the top-case**, where it connects to the logic board via a ZIF socket located roughly in the center-bottom of the top-case interior. Liquid spilled on the keyboard travels downward by gravity and capillary action, meaning the **connector and the lower portion of the FPC** are among the most likely sites for residue accumulation — consistent with all affected keys being in a single vertical column rather than a single horizontal row.

### 6. Affected Block Schematics (Model A2918 / Board 820-02757)

The following diagrams show the specific signal blocks affected in this incident on the MacBook Pro 14" M3 Pro (A2918), based on the keyboard system architecture for board 820-02757.

#### 6a. Keyboard System Block Diagram

```
  ┌─────────────────────────────────────────────────────────────────────┐
  │                    MacBook Pro 14" M3 Pro (A2918)                   │
  │                    Board: 820-02757 / EMC 8304                     │
  ├─────────────────────────────────────────────────────────────────────┤
  │                                                                     │
  │   ┌─────────────────────────────────────────────┐                  │
  │   │         KEYBOARD ASSEMBLY (top-case)        │                  │
  │   │                                             │                  │
  │   │  ┌─────────────────────────────────────┐   │                  │
  │   │  │  Key Matrix (~6 rows × 16 columns)  │   │                  │
  │   │  │  ┌───┬───┬───┬───┬───┬───┬───┐      │   │                  │
  │   │  │  │R1 │R2 │R3 │R4 │R5 │R6 │...│ rows │   │                  │
  │   │  │  ├───┼───┼───┼───┼───┼───┼───┤      │   │                  │
  │   │  │  │C1 │C2 │...│C7 │...│C15│C16│ cols │   │                  │
  │   │  │  └───┴───┴───┴─▲─┴───┴───┴───┘      │   │                  │
  │   │  │                │                     │   │                  │
  │   │  │         [AFFECTED COLUMN C7]         │   │                  │
  │   │  │          6 — Y — H — N               │   │                  │
  │   │  └──────────────────┬──────────────────┘   │                  │
  │   │                     │ FPC ribbon cable      │                  │
  │   │                     │ (polyimide flex,      │                  │
  │   │                     │  ~30-34 traces)       │                  │
  │   └─────────────────────┼───────────────────────┘                  │
  │                         │                                           │
  │                    ┌────┴─────┐                                     │
  │                    │   ZIF    │  ZIF connector on top-case          │
  │                    │ connector│  or logic board flex cable          │
  │                    └────┬─────┘                                     │
  │                         │                                           │
  │   ┌─────────────────────┴─────────────────────┐                    │
  │   │     KEYBOARD CONTROLLER IC                │                    │
  │   │     (dedicated MCU in top-case assembly)  │                    │
  │   │                                           │                    │
  │   │  ┌─────────────────────────────────────┐  │                    │
  │   │  │  Row driver outputs  (active scan)  │  │                    │
  │   │  │  Column sense inputs (read matrix)  │──┼── [C7 affected]   │
  │   │  │  Backlight LED PWM driver           │  │                    │
  │   │  │  Key debounce + ghost-key logic     │  │                    │
  │   │  └─────────────────────────────────────┘  │                    │
  │   └───────────────────┬───────────────────────┘                    │
  │                       │  SPI bus                                    │
  │                       │  (MOSI, MISO, SCK, CS)                     │
  │                       │                                             │
  │   ┌───────────────────┴───────────────────────┐                    │
  │   │        APPLE M3 PRO SoC                   │                    │
  │   │   (receives keypress data via SPI,         │                    │
  │   │    passes to macOS HID subsystem)          │                    │
  │   └───────────────────────────────────────────┘                    │
  │                                                                     │
  └─────────────────────────────────────────────────────────────────────┘
```

#### 6b. FPC Connector Pinout — Affected Columns Highlighted

The keyboard FPC carries all row and column signals to the controller IC. The affected column (C7, carrying keys 6/Y/H/N) is a single pin on the FPC/ZIF connector. Diagnostic data (March 22) confirms that C7 is bridged to **two** other column pins — the columns carrying 9/O/L/. and 0/P/;/'. FPC trace routing is non-sequential (matrix columns are reordered for routing efficiency), so these three columns occupy **adjacent pins** on the FPC even though their keys are 3–4 positions apart on the physical keyboard:

```
  FPC pinout (simplified 34-pin model, viewed from below)
  Actual pin assignments are illustrative — exact positions depend on board 820-02757 routing

  Pin:  1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16  17
       ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
       │R1│R2│R3│R4│R5│R6│C1│C2│..│Ca│C7│Cb│..│Cx│..│..│Gnd│
       └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
                                          ▲▲▲▲
                                          ││││
       Col carrying 9,O,L,.  (Ca) ─────┘│││
       Affected col 6,Y,H,N  (C7) ─────┘││
       Col carrying 0,P,;,/' (Cb) ──────┘│
       Space bar col trace    (Cx) ──────┘ (also bridges to Cb → outputs ')

       Dried cola bridges Ca ↔ C7 ↔ Cb:
       → pressing 6 also activates Ca (→ 9) and Cb (→ 0) → output "690"
       → pressing Y also activates Ca (→ O) and Cb (→ P) → output "yop"
       → pressing H also activates Ca (→ L) and Cb (→ ;) → output "hl;"
       → pressing N also activates Ca (→ .) and Cb (→ /) → output "n./"
       → Space bar trace bridges to Cb → outputs " '"
```

#### 6c. Signal Path from Affected Keys to SoC

```
  Affected key "N" pressed (diagnostic output: "n./"):
  ═══════════════════════════════════════════════════════════

  [N key]  (physical keypress)
      │
      ▼
  [Scissor switch collapses, rubber dome presses FPC membrane]
      │
      ▼
  [FPC contact pad closes ROW_4 × COL_7 intersection]
      │                    │                    │
      │                    │ dried cola residue  │
      │                    │ bridges C7↔Ca↔Cb   │
      │                    │                    │
      ▼                    ▼                    ▼
  COL_7 active         COL_Ca active       COL_Cb active
  (6,Y,H,N)            (9,O,L,.)           (0,P,;,/)
      │                    │                    │
      ▼                    ▼                    ▼
  [FPC traces → adjacent ZIF connector pins]
  (Ca, C7, Cb are adjacent on FPC despite non-adjacent physical keys)
      │                    │                    │
      ▼                    ▼                    ▼
  ┌──────────────────────────────────────────────────┐
  │   Keyboard Controller IC                         │
  │                                                  │
  │   Scans ROW_4 → reads COL_7  ──► reports "N"    │
  │                   reads COL_Ca ──► reports "."    │
  │                   reads COL_Cb ──► reports "/"    │
  │                                                  │
  │   Result: controller sends three keycodes        │
  │   for one physical keypress → "n./"              │
  └──────────────────────┬───────────────────────────┘
             │ SPI bus
             ▼
  ┌──────────────────────────────────────────────────┐
  │   Apple M3 Pro SoC                               │
  │   → macOS receives "N" + "." + "/"               │
  │   → all three appear on screen: "n./"            │
  └──────────────────────────────────────────────────┘
```
#### 6d. Logic Board Keyboard Area Layout (Board 820-02757)

```
  MacBook Pro 14" A2918 logic board (simplified top view)
  ┌──────────────────────────────────────────────────────────────────┐
  │                                                                  │
  │    ┌──────────────────────┐        ┌─────────────────┐          │
  │    │                      │        │  Thunderbolt /   │          │
  │    │    Apple M3 Pro      │        │  USB-C ports     │          │
  │    │    SoC (main chip)   │        └─────────────────┘          │
  │    │                      │                                      │
  │    │   ┌──────────────┐   │                                      │
  │    │   │ SPI keyboard │   │                                      │
  │    │   │ interface    │   │                                      │
  │    │   └──────┬───────┘   │                                      │
  │    └──────────┼───────────┘                                      │
  │               │ SPI traces on PCB                                │
  │               │                                                  │
  │    ┌──────────┴──────────┐                                       │
  │    │  Keyboard Controller│  ← dedicated IC near keyboard         │
  │    │  IC (MCU)           │    connector area                     │
  │    │  scans matrix,      │                                       │
  │    │  sends SPI to SoC   │                                       │
  │    └──────────┬──────────┘                                       │
  │               │ short PCB traces                                 │
  │               │                                                  │
  │    ╔══════════╧══════════╗  ← ════════════════════════════════   │
  │    ║  ZIF CONNECTOR      ║    CONTAMINATION LIKELY HERE          │
  │    ║  (keyboard FPC      ║    or along the FPC trace below       │
  │    ║   plugs in here)    ║  ← ════════════════════════════════   │
  │    ╚═════════════════════╝                                       │
  │               ▲                                                  │
  │               │ FPC ribbon cable                                 │
  │               │ (exits to top-case                               │
  │               │  keyboard assembly)                              │
  │                                                                  │
  │    ┌──────────────────┐   ┌───────────────────┐                 │
  │    │ Trackpad          │   │ Battery connector │                 │
  │    │ connector         │   │                   │                 │
  │    └──────────────────┘   └───────────────────┘                 │
  │                                                                  │
  └──────────────────────────────────────────────────────────────────┘
```

### 7. Board-Level Schematic References

For the MacBook Pro 14" A2918 (board **820-02757**, design **051-07754**), the following resources provide component-level schematics and boardview files that show the exact keyboard controller IC location, FPC connector pinout, and trace routing:

| Resource | URL | Contents |
|---|---|---|
| Apple Self-Service Repair Manual | https://support.apple.com/en-us/118617 | Exploded views, connector locations, part numbers (no circuit schematics) |
| NotebookSchematics (820-02757) | https://notebookschematics.com/macbook-pro-14-a2918-2023-m3-schematic-boardview-820-02757-schematic-boardview/ | PDF schematic + BRD boardview file |
| LaptopSchematic (820-02757) | https://www.laptopschematic.com/apple-macbook-pro-14-m3-a2918-2023-820-02757-schematic-boardview/ | PDF schematic + boardview |
| PCSchematics (051-07754) | https://pcschematics.com/apple-macbook-pro-14%E2%80%B3-a2918-2023-m3-820-02757-051-07754-schematic-boardview/ | Schematic and boardview download |
| RepairLap forum (EMC 8304) | https://www.repairlap.com/threads/apple-macbook-pro14-m3-a2918-2023-emc8304-820-02757-boardview-schematics.28192/ | Community-posted boardview + schematic files |
| LogiWiki board number index | https://logi.wiki/index.php/Board_Number_by_A_Number | Cross-reference A-number → board number |
| iFixit teardown (14" M3) | https://www.ifixit.com/Teardown/MacBook+Pro+14-Inch+2023+Teardown/169486 | High-res teardown photos of A2918 internals |

**Note:** The BRD boardview files require a viewer such as **OpenBoardView** (free, open-source) or **FlexBV** to navigate the component layout interactively. The PDF schematics show the full circuit including the keyboard controller IC (typically labeled as a "U"-prefixed component near the keyboard ZIF connector), SPI bus connections to the M3 Pro SoC, and individual column/row signal names.

### 8. Diagrams (uploaded as files)

The following diagrams are included in this repository in the [`diagrams/`](diagrams/) directory, available in both SVG (vector, scalable) and PNG (raster, 1200px wide) formats:

| Diagram | SVG | PNG | Description |
|---|---|---|---|
| Keyboard matrix with affected column | [SVG](diagrams/keyboard-matrix-affected-column.svg) | [PNG](diagrams/keyboard-matrix-affected-column.png) | Row/column matrix layout showing keys 6, Y, H, N sharing column C7 (highlighted in red) |
| Keyboard system block diagram | [SVG](diagrams/keyboard-system-block-diagram.svg) | [PNG](diagrams/keyboard-system-block-diagram.png) | Full signal chain: key matrix → FPC ribbon → ZIF connector → keyboard controller IC → SPI → M3 Pro SoC |
| ZIF connector detail (3 views) | [SVG](diagrams/zif-connector-detail.svg) | [PNG](diagrams/zif-connector-detail.png) | Top view (latch open), top view (latch closed), and side cross-section showing FPC pads contacting spring contacts |
| FPC liquid damage | [SVG](diagrams/fpc-liquid-damage.svg) | [PNG](diagrams/fpc-liquid-damage.png) | Before/after comparison showing how dried cola bridges column traces C7→C6/C8 |
| Scissor-switch cross-section | [SVG](diagrams/scissor-switch-cross-section.svg) | [PNG](diagrams/scissor-switch-cross-section.png) | Side view of the non-serviceable key block: keycap → scissor arms → rubber dome → FPC membrane |
| Butterfly vs scissor comparison | [SVG](diagrams/butterfly-vs-scissor.svg) | [PNG](diagrams/butterfly-vs-scissor.png) | Side-by-side comparison of the two Apple keyboard mechanisms |

### 9. Reference Photos of Real Hardware (external links)

The following links show actual teardown photos of MacBook Pro hardware similar to the M3 Pro model. These illustrate the real-world appearance of the components described in the ASCII schematics above:

**Keyboard FPC ribbon cable and ZIF connector:**
- iFixit MacBook Pro 14" M3 teardown — top-case interior showing the keyboard FPC ribbon routing and ZIF connector location: https://www.ifixit.com/Teardown/MacBook+Pro+14-Inch+2023+Teardown/169486
- iFixit MacBook Pro 16" keyboard replacement guide — close-up of the ZIF socket with latch open/closed and FPC insertion: https://www.ifixit.com/Guide/MacBook+Pro+16-Inch+2021+Keyboard+Replacement/148094

**Scissor-switch key mechanism (non-serviceable block):**
- iFixit MacBook Pro keyboard key replacement — photos of the scissor arms, rubber dome, and mounting clips from above and side angles: https://www.ifixit.com/Guide/MacBook+Pro+Retina+Keyboard+Key+Replacement/111942
- Apple scissor mechanism patent diagram (publicly available via Google Patents): https://patents.google.com/patent/US10490364B2 — shows the interlocking arm design and capillary-scale clearances

**Liquid damage on FPC traces (similar incidents):**
- iFixit liquid damage guide — photos showing corrosion and residue on logic board traces and connectors after a liquid spill: https://www.ifixit.com/Wiki/Liquid_Damage
- Rossmann Group YouTube channel frequently shows close-up microscope footage of cola/coffee damage on MacBook flex cables and connectors — search "Rossmann MacBook liquid damage keyboard" for representative examples

## Service Center's Position: "Non-Serviceable Key Blocks"

The service center has clarified what they mean by "mechanical issue": the individual key switch assemblies on modern MacBook keyboards are sealed units. Once liquid wicks into the tiny capillary space between the keycap, scissor-switch mechanism, and underlying membrane substrate, it cannot be removed by manual cleaning (swabbing, compressed air, or partial disassembly). This is technically accurate — the scissor-switch mechanism sits over a rubber dome on top of the FPC, and the tolerances are so tight that manual access is impossible without destroying the switch.

The diagram below shows what a single scissor-switch key block looks like in cross-section, illustrating why it is considered non-serviceable:

```
  Single MacBook scissor-switch key (cross-section, side view)

       ┌──────────────────────────────┐
       │         K E Y C A P          │  ← hard plastic cap, snaps onto scissor arms
       └────────┬──────────┬──────────┘
               /            \
  scissor arm /  (X-shaped   \ scissor arm
             /   pivot joint) \
  ┌─────────┴──────────────────┴─────────┐
  │   scissor mechanism (plastic arms)   │  ← pivot clips into keycap and base plate
  └──────────────────┬───────────────────┘
                     │  compresses
                     ▼
             ┌───────────┐
             │ rubber    │  ← silicone dome (~2 mm diameter), acts as spring
             │  dome     │    AND electrical actuator
             └─────┬─────┘
                   │  presses
                   ▼
  ┌────────────────────────────────────────┐
  │           FPC membrane layer           │  ← two conductive layers separated by
  │  ┌────────────────────────────────┐   │     a thin non-conductive spacer with
  │  │  row trace ── contact ── col   │   │     a hole; dome press brings them together
  │  └────────────────────────────────┘   │
  └────────────────────────────────────────┘

  Why it is "non-serviceable":
  ┌──────────────────────────────────────────────────────────────┐
  │  The scissor arms clip tightly into the base plate below and  │
  │  the keycap above. Capillary gaps between these parts are     │
  │  typically < 0.3 mm — too small for any swab or tool to       │
  │  reach. Disassembling the scissor mechanism requires          │
  │  unclipping the fragile plastic arms, which break easily      │
  │  and cannot be reassembled to factory spec.                   │
  │                                                               │
  │  Liquid path into the switch:                                 │
  │  spill → between keycap edge and switch body                  │
  │         → down scissor arm channels (capillary action)        │
  │         → onto rubber dome and FPC contact area               │
  │         ← cannot be reversed without ultrasonic cavitation    │
  └──────────────────────────────────────────────────────────────┘
```

However, this framing has an important nuance:

- **The root cause of the observed symptoms** (entire column affected, multi-character output, ghost keypresses) points to contamination on the **FPC column trace or connector**, not inside individual key switch bodies. A faulty individual key mechanism would typically affect only that one key, not all four keys sharing a column.
- **Ultrasonic cleaning** uses high-frequency sound waves in a liquid bath (often isopropyl alcohol or a dedicated electronics solvent) to cause cavitation — microscopic bubbles that implode and dislodge residue from surfaces unreachable by any manual method, including inside sealed key switch bodies and under FPC traces. This is the correct treatment for this type of contamination.

The service center offers ultrasonic cleaning at approximately **1/3 the cost of keyboard replacement**, with a turnaround of **3–7 days**. Given that:

1. Ghost keypresses have already resolved (the residue has stabilized and is no longer migrating).
2. The column symptoms remain consistent and localized, suggesting a single contamination site.
3. Replacement would still be an option if cleaning fails.

**Ultrasonic cleaning is the recommended first step** — it addresses the actual likely root cause (FPC trace contamination), costs significantly less than replacement, and carries low risk of worsening the situation.

## Chemical Analysis of Coca-Cola Zero Residue

The spill chemistry matters here because **Coca-Cola Zero is still chemically damaging to electronic circuits even without sugar**. A typical Coca-Cola Zero / Coke Zero Sugar formulation contains:

- **Water + dissolved CO₂** — the bulk liquid that quickly spreads by gravity and capillary action.
- **Phosphoric acid** — lowers pH and drives copper/tin corrosion.
- **Potassium/sodium salts** (such as potassium benzoate or citrate, depending on market formula) — create an ionic electrolyte that increases conductivity.
- **Artificial sweeteners** (such as aspartame and acesulfame K) — not the main conductor, but part of the organic residue left behind after drying.
- **Caramel color / flavor residues** — sticky organics that help the dried film adhere to pads, connector fingers, and membrane surfaces.

In other words, the dangerous part is **acidic electrolyte + hygroscopic residue**, not sucrose. Zero-sugar cola can still short adjacent conductors while wet and then leave behind a film that continues to attract moisture from the air and support leakage current after the visible liquid is gone.

### What Coca-Cola Zero does electrically and chemically

1. **While wet, it forms a conductive bridge** between adjacent keyboard-matrix traces or connector pins. That explains the initial ghost keypresses and multi-character output.
2. **As it dries, the liquid shrinks into a thinner, more localised film**. This often stops the random ghost presses but leaves a stable leakage path between one column trace and its neighbour.
3. **The acidic residue keeps attacking exposed metal** — especially copper, ENIG pads, tin finishes, and spring contacts — so the fault can worsen over hours or days even after the keyboard seems "dry."
4. **The residue is hygroscopic** enough to reabsorb a small amount of ambient moisture, so conductivity can vary with temperature, humidity, and usage.

This matches the observed progression: first unstable ghost presses, then a more repeatable "correct key plus extra keys" failure on a single column.

### Are the traces protected by any chemical cover?

**Partly — but not at the critical contact points.**

- **Most FPC copper traces are protected** by the flex-cable stack-up itself: the copper is laminated between **polyimide base film and polyimide coverlay**. This provides a protective barrier against casual abrasion and some chemical exposure.
- **However, the connector fingers, dome-switch contact lands, and mating pads are intentionally exposed** (often with nickel/gold or similar plating) so the keyboard can make electrical contact. Those exposed areas do **not** have a chemical cover over the active contact surface.
- **The scissor-switch assembly is mechanically sealed but not hermetic.** Liquid can still wick through sub-millimeter gaps and reach the exposed membrane contact area underneath.

So the accurate answer is:

- **Buried traces:** yes, they are somewhat protected by polyimide coverlay.
- **ZIF connector pads and switch contact points:** no, they must remain exposed, so they are vulnerable to cola residue and corrosion.

That is why a spill can leave the majority of the flex cable visually intact yet still produce a severe electrical fault at one connector pin or one membrane-contact region.

## Discussion

### Does "mechanical issue" accurately describe the problem?

The service center's characterisation of "mechanical issue" is imprecise and potentially misleading in its original sense. The symptoms are **electrical/chemical** in nature, not mechanical:

- **Ghost keypresses** (keys firing with nothing pressed) cannot result from mechanical damage — a broken spring or cracked keycap cannot generate electrical signals on its own. This is only possible if there is residual conductivity (dried cola bridging traces) or an active short circuit.
- **A single key producing 3 symbols** means the keyboard controller is receiving signals from multiple matrix intersections simultaneously — a purely electrical phenomenon caused by conductive contamination or shorted traces.
- **The entire column (6/Y/H/N) being affected at once** points to a shared column trace issue, not individual key mechanisms failing independently.

The service center later clarified they mean "mechanical" in the sense that the sealed key switch bodies are **non-serviceable** — residue trapped inside them cannot be reached by conventional manual cleaning. That narrower claim is technically valid, but the root cause of the column-wide electrical symptoms still lies at the FPC trace or connector level, not inside individual key bodies.

### Can symptom drift be explained by eventual drying?

Yes — but drying does **not** resolve the problem; it transforms it into a potentially worse one.

When Coca-Cola Zero dries, water evaporates but leaves behind a concentrated residue of acids, ionic salts/preservatives, sweeteners, and caramel-color organics. That residue:

1. **Concentrates the conductivity** — a dried film can produce a more stable and persistent short than the original liquid, because liquid may shift around while a dried film stays exactly where it is, bridging the same traces consistently.
2. **Continues to corrode** — phosphoric acid keeps attacking copper traces even after drying, progressing over days. This explains why symptoms *worsened* rather than stabilised: corrosion is an ongoing electrochemical process that does not stop when the liquid evaporates.
3. **Can wick and migrate** — as liquid evaporates it moves via capillary action, potentially spreading contamination to adjacent traces. This would explain why more keys became affected over time.

Drying therefore does explain the symptom drift — but by transforming a wet short into a chemically active residue that both maintains the short and progressively damages surrounding circuitry, not by healing the damage.

### What does the disappearance of ghost keypresses indicate?

Ghost keypresses require a *floating* or *intermittent* short — a conductive path with just enough resistance to occasionally trigger a false matrix intersection without a key being pressed. Once the residue fully dries and stabilises into a fixed film, it either:

- **Maintains a permanent short** (consistent with the column still producing wrong characters when a key is pressed), or
- **Loses enough conductivity** that it no longer bridges the trace spontaneously, only doing so when a key is mechanically pressed and completes the circuit more firmly.

The disappearance of ghost presses suggests the residue has settled into a stable state rather than continuing to spread. This is slightly better news for cleaning: contamination that is no longer migrating is more likely to be localised and addressable. The column symptoms (multi-character output from H/Y/6/N) persist because a stable conductive bridge remains between column traces, just no longer with enough free conductivity to cause floating triggers.

### Partial improvement observed after 2-day rest period

An important update (March 22): after the laptop was returned from the service center, ghost keypresses made it unusable, so an external keyboard was connected and the internal keyboard was disabled using **Karabiner Elements**. The laptop was used this way for approximately 2 days without the internal keyboard being active.

After re-enabling the internal keyboard, the affected keys now **produce the correct character plus two incorrect ones** — whereas previously the correct character was intermittent or absent entirely. This is a meaningful improvement.

This observation is diagnostically significant for several reasons:

1. **Reduced thermal cycling** — with the laptop still being used (so the SoC and battery were generating heat), but the keyboard controller not actively scanning the matrix, the thermal profile around the keyboard area would have been slightly lower. This may have slowed ongoing corrosion somewhat.
2. **No mechanical actuation** — without keys being pressed for 2 days, the physical pressure on the rubber dome → FPC membrane contact points was absent. This means no repeated compression of the contaminated contact area, which could otherwise spread or redistribute residue.
3. **Continued drying** — the 2-day rest period allowed further evaporation of any remaining moisture in the FPC/connector area. As moisture decreases, the conductive path weakens — consistent with the improvement from "no correct character" to "correct character plus extras."
4. **Positive signal for cleaning** — this improvement suggests the contamination is conductive residue (dried acid/salt/organic film) rather than irreversible copper corrosion through the trace. If the traces were physically etched through by phosphoric acid, a rest period would not improve the symptoms. The fact that it did improve suggests **ultrasonic cleaning has a good chance of resolving the problem**.

This supports the recommendation to pursue ultrasonic cleaning as the first step before considering top-case replacement.

## Hypotheses

### Hypothesis 1: Residual liquid causing short circuits (most likely)

Coca-Cola Zero contains water, acids, ionic salts/preservatives, sweeteners, and other residues. Even after the initial drying period, residual moisture or dried conductive film can remain in tight spaces — especially under the scissor-switch key mechanism or on the underlying flex-cable connector pads. When the keyboard is in use and the device warms up, residual conductivity between adjacent key traces can cause:

- A single key contact to trigger multiple key signals (explaining the 3-symbol behavior on **N**).
- Adjacent keys in the same matrix column (H, Y, 6, N) to be affected together, since they share a common column trace on the keyboard matrix PCB/flex cable.
- Spontaneous keypresses, as moisture bridges two traces that should not be connected.

The fact that the affected keys form a **single column** of the keyboard matrix is strong evidence of a column-trace short or contamination on the column wire, rather than individual key damage.

### Hypothesis 2: Incorrect reattachment of the keyboard flex-cable connector

The thin-film (ZIF/FFC) keyboard connector on MacBooks is fragile and directional. If the service center disconnected and reconnected it incorrectly — e.g., inserted at a slight angle, not fully seated, or with the locking latch not fully engaged — this can cause:

- Intermittent or missing contact on specific pins, producing missing or garbled keystrokes.
- A shifted contact alignment causing one column's signals to be read on the wrong row or column, producing wrong characters.

This was the owner's initial hypothesis and was tested by asking the service center to reseat the connector. However, the symptoms persisted, making this a less likely **sole** cause, though it could be a **contributing factor** if combined with liquid damage.

### Hypothesis 3: Damaged keyboard membrane / flex cable traces

If the liquid caused corrosion or a physical break on the flexible printed circuit (FPC) that runs under the keyboard, specific traces could be:

- Open (broken): key produces no output.
- Shorted to an adjacent trace: key produces multiple outputs or triggers neighboring keys.

Corrosion on flex-cable traces is a known consequence of cola spills due to the acidic, ionic composition of the liquid. This damage can worsen over time as oxidation progresses, which aligns with the observation that symptoms evolved and worsened over several days.

### Hypothesis 4: Damage to the keyboard controller IC

On Apple Silicon MacBooks (including M3 Pro), the keyboard controller is integrated into the top-case assembly as a dedicated IC that communicates with the SoC via SPI or I2C. If liquid reached the controller and caused partial damage, it could misinterpret column signals and generate phantom keypresses or incorrect key codes. This is less likely than trace-level damage but possible if liquid penetration was significant.

### Hypothesis 5: Insufficient or improper initial cleaning at the service center

The service center spent approximately 2 hours cleaning the device. However, standard service-center cleaning typically involves wiping visible liquid and using compressed air or isopropyl alcohol swabs on accessible surfaces. This does **not** adequately address cola residue because:

- Cola leaves a sticky, acidic film that requires thorough solvent flushing (not just wiping) to fully remove.
- The keyboard FPC and sealed key switch bodies have sub-millimeter gaps that surface-level cleaning cannot reach.
- If the device was powered on (even briefly) before all residue was removed, electrochemical corrosion may have been accelerated by the applied voltage across contaminated traces.

The fact that key N was already malfunctioning when the device was returned from cleaning suggests residue was already present on the FPC at that point. It is possible that the 2-hour cleaning was insufficient to remove cola from under the key mechanisms and along the FPC traces, leaving the contamination in place to worsen over subsequent days.

### Hypothesis 6: Thermal cycling accelerating damage

When the MacBook is in use, the internal temperature rises (CPU/GPU heat dissipates through the chassis). This thermal cycling can:

- Cause residual moisture trapped under key switches to evaporate and re-condense in slightly different positions, spreading contamination.
- Accelerate the electrochemical corrosion rate of phosphoric acid on copper traces (corrosion rate approximately doubles for every 10°C increase in temperature).
- Cause micro-expansion/contraction of the FPC substrate, potentially cracking traces already weakened by corrosion.

This could explain why symptoms worsened progressively during the days after the spill — each use session would have produced thermal cycling that accelerated trace degradation.

### Hypothesis Ranking Summary

| Rank | Hypothesis | Likelihood | Status | Key Evidence |
|------|-----------|------------|--------|-------------|
| **1** | **H1: Conductive residue** (dried cola shorting column trace) | **★★★★★ Primary** | ✅ Active | Clean column pattern (6/Y/H/N); multi-character output; improvement during rest period proves residue not permanent damage |
| **2** | **H5: Insufficient initial cleaning** | **★★★★☆ Primary** | ✅ Active | N was already faulty at pickup; 2-hour surface clean cannot reach sub-0.3 mm gaps; cola requires solvent flushing, not wiping |
| **3** | **H3: FPC trace corrosion** | **★★★★☆ Primary** | ⚠️ Partial | Progressive worsening over days; phosphoric acid attacks copper continuously; but rest-period improvement suggests traces not yet destroyed |
| **4** | **H6: Thermal cycling accelerator** | **★★★☆☆ Contributing** | ✅ Active | Worsening during use, improvement during 2-day powered-off rest; heat accelerates corrosion ~2× per 10°C |
| **5** | **H2: Connector misalignment** | **★★☆☆☆ Ruled out as sole cause** | ❌ Tested | Reseating connector did not resolve symptoms; may have been a minor contributor |
| **6** | **H4: Keyboard controller IC damage** | **★☆☆☆☆ Unlikely** | ❓ Not tested | Would affect more than one column; clean column pattern points to trace-level issue instead |

**Winner: H1 + H5 + H3**, with **H6** as accelerator. The primary cause is conductive cola residue on the shared column C7 trace, left behind by insufficient initial cleaning, with ongoing phosphoric acid corrosion worsening damage over time. The rest-period improvement confirms the dominant mechanism is still reversible contamination (H1) rather than irreversible corrosion (H3), making ultrasonic cleaning the correct first step.

## Physical Localization of the Damage

Based on the symptom pattern and electrical analysis, the contamination can be physically localized to a specific area:

### Where the damage is

```
    ┌─────────────────────────────────────────────────────┐
    │                   TOP CASE (interior view)          │
    │                                                     │
    │   Key matrix layer (FPC membrane)                   │
    │   ┌───────────────────────────────────────────┐     │
    │   │  ...  5   6   7  ...                      │     │
    │   │  ...  T  [Y]  U  ...     ← Affected keys  │     │
    │   │  ...  G  [H]  J  ...       are in column   │     │
    │   │  ...  B  [N]  M  ...       C7 (bracketed)  │     │
    │   └───────────┬───────────────────────────────┘     │
    │               │                                     │
    │    ══════════ FPC ribbon cable ══════════            │
    │               │                                     │
    │        ┌──────┴──────┐                              │
    │        │ ZIF connector│ ◄── CONTAMINATION SITE #1   │
    │        │  (30+ pins)  │     Cola residue on pin C7  │
    │        └──────┬──────┘     and bridges to C6/C8     │
    │               │                                     │
    │        ┌──────┴──────┐                              │
    │        │  Keyboard    │                              │
    │        │ Controller IC│ ◄── Unlikely damage site     │
    │        └──────┬──────┘                              │
    │               │ SPI bus                              │
    │        ┌──────┴──────┐                              │
    │        │  M3 Pro SoC │                              │
    │        └─────────────┘                              │
    └─────────────────────────────────────────────────────┘

    CONTAMINATION SITE #2: Along the FPC ribbon cable itself,
    where cola wicked between the column C7 trace and adjacent
    traces Ca (9/O/L/.) and Cb (0/P/;/') via capillary action
    in the ~0.1 mm gap between polyimide substrate layers.
    Space bar trace also runs near this group.

    CONTAMINATION SITE #3: Under the sealed scissor-switch key
    bodies for 6, Y, H, N — where cola entered through < 0.3 mm
    capillary gaps and cannot be manually extracted.
```

### Three specific contamination zones

1. **ZIF connector area** (most accessible) — dried cola residue bridging three adjacent FPC pins: C7 (6/Y/H/N), Ca (9/O/L/.), and Cb (0/P/;/'). The Space bar trace also appears to bridge to Cb at this location. This is the most likely primary site because: (a) the connector is an open junction point where liquid pools, (b) it explains the clean column pattern, and (c) it is the first component the service center would have accessed during cleaning (and may not have cleaned thoroughly enough).

2. **FPC ribbon cable traces** (moderately accessible) — residue wicked along the column C7 trace where it runs parallel to Ca and Cb within the ribbon. The ~0.1 mm gap between traces inside the FPC acts as a capillary channel that draws liquid in and is very difficult to clean without ultrasonic treatment.

3. **Under sealed key switch bodies** (non-serviceable) — the scissor-switch assemblies for 6, Y, H, N. The service center is correct that these cannot be manually opened or cleaned without breaking the mechanism. However, this is likely a **secondary** contamination site rather than the primary one, because contamination only inside individual key bodies would not explain the entire column being affected simultaneously.

### Are the service guys right about "mechanical" nature?

**Partly right, partly wrong:**

| Aspect | Service center says | Actual assessment |
|--------|-------------------|-------------------|
| Key blocks are non-serviceable | ✅ **Correct** — scissor mechanisms cannot be manually opened or cleaned | Confirmed: capillary gaps < 0.3 mm, arms break on disassembly |
| The issue is "mechanical" | ❌ **Inaccurate** — the cause is electrical/chemical (conductive residue + acid corrosion) | Ghost keypresses and multi-character output are purely electrical phenomena |
| Keyboard replacement is needed | ⚠️ **Premature** — ultrasonic cleaning should be tried first | Rest-period improvement proves contamination is still reversible |
| The primary damage location | ⚠️ **Incomplete** — they focus on sealed key blocks | Column pattern points to FPC/ZIF connector contamination, not just key bodies |

The service center is technically right that **some** contamination is in non-serviceable locations (under key switches), but they are wrong to characterize the issue as "mechanical" and may be overlooking the primary contamination site (ZIF connector and FPC traces). The column-wide symptom pattern cannot be explained by contamination inside individual key bodies alone — it requires a shared trace-level short, which is on the FPC or at the connector.

**Bottom line:** The issue is physically localized to the column C7 trace path — primarily at the ZIF connector junction and along the FPC ribbon, with secondary contamination under the key switch bodies. Ultrasonic cleaning can reach all three sites and should be attempted before committing to a full keyboard/top-case replacement.

## Most Probable Root Cause

The most probable explanation is a **combination of Hypotheses 1, 3, and 5**: the initial service-center cleaning was insufficient to remove cola residue from the keyboard FPC traces and sealed key switch bodies. Sticky, conductive residue (dried cola) remained on the keyboard flex cable, creating persistent shorts across a single keyboard matrix column (H, Y, 6, N). Ongoing corrosion from phosphoric acid, accelerated by thermal cycling during normal use (Hypothesis 6), caused progressive worsening. This accounts for:

1. The affected keys forming a clean column pattern.
2. Multiple symbols being generated by a single keypress.
3. Spontaneous keypresses (now resolved as residue dried).
4. Symptoms worsening over time as residue dried and corrosion progressed.
5. **Partial improvement after a 2-day rest period** — the fact that symptoms improved (correct characters returned) during a period of non-use strongly suggests the primary cause is conductive residue rather than irreversible trace damage. This is a positive indicator for cleaning.

The connector misalignment (Hypothesis 2) may have introduced additional artifacts but is unlikely to be the primary cause since reseating the connector did not resolve the issue.

## Recommended Next Steps

1. **Test remaining keys** — Before cleaning, run `cat` (or similar raw-input capture) on **every key** to map which additional keys may be affected and confirm the full extent of the bridging. This creates a baseline for verifying cleaning success. In particular, test keys in the bridged columns (9, O, L, `.` and 0, P, `;`, `/`, `'`) to see if pressing them also produces phantom characters from C7.
2. **Ultrasonic cleaning (recommended first step)** — The service center offers this at approximately 1/3 the cost of keyboard replacement, with a 3–7 day turnaround. This is the appropriate treatment: ultrasonic cavitation reaches inside sealed key switch bodies and under FPC traces where no manual cleaning can. The diagnostic data confirms three matrix columns are bridged (C7, Ca, Cb) plus the Space bar trace, so the cleaning lab should focus on the cluster of adjacent FPC pins carrying these columns.
3. **Visual inspection under magnification** of the FPC traces in the affected area for corrosion, dendritic growth, or residue bridges — ideally performed as part of or after the ultrasonic process.
4. **Resistance measurement** between pin C7 and adjacent pins (Ca, Cb) on the ZIF connector to confirm whether the conductive bridges have been removed after cleaning. Also measure the Space bar column pin.
5. **Post-cleaning verification** — Re-run the same `cat` test on all affected keys (6, Y, H, N, Space, and any others identified in step 1) to confirm each key produces only its intended character.
6. If ultrasonic cleaning does not resolve the issue, **keyboard/top-case replacement** will be necessary. Corrosion that has fully etched through a copper trace is not reversible, but this is the fallback rather than the first resort.

## Note for the Ultrasonic Cleaning Lab

This section summarises the key technical details for the technicians performing ultrasonic cleaning on this device.

### Device

- **MacBook Pro 14" (M3 Pro, November 2023)**, model A2918, serial MWJPXQ4VC4
- **Logic board:** 820-02757
- **Keyboard type:** Scissor-switch (Magic Keyboard), integrated into top-case assembly
- **Spill substance:** Coca-Cola Zero (contains phosphoric acid, ionic salts/preservatives, artificial sweeteners, caramel-color residue)

### What to look for

Diagnostic testing (March 22) has precisely mapped the contamination. **Three matrix columns** are bridged by dried cola residue, plus the Space bar trace:

| Affected column | Keys | Diagnostic output (per keypress) |
|---|---|---|
| **C7** (primary) | 6, Y, H, N | `690`, `yop`, `hl;`, `n./` |
| **Ca** (bridged) | 9, O, L, `.` | (activated when C7 keys pressed) |
| **Cb** (bridged) | 0, P, `;`, `/`, `'` | (activated when C7 keys pressed) |
| **Space bar trace** | Space | ` '` (space + apostrophe from Cb bridge) |

These three matrix columns (C7, Ca, Cb) are routed on **physically adjacent FPC traces** despite corresponding to non-adjacent physical keyboard positions. The most probable contamination sites, in order of priority:

1. **ZIF connector area** — dried cola residue bridging the cluster of adjacent pins carrying C7, Ca, and Cb. The Space bar column pin also runs near this group. This is an open junction point where liquid pools and is the most likely primary site.
2. **FPC ribbon cable** — residue wicked along the C7 trace where it runs parallel and close (~0.1 mm) to Ca and Cb. Capillary action draws liquid into this gap.
3. **Under the sealed key switch bodies** for 6, Y, H, N (and potentially Space) — cola entered through sub-0.3 mm capillary gaps between the keycap, scissor arms, rubber dome, and FPC membrane.

### Why the "non-serviceable key blocks" diagnosis is incomplete

The service center correctly notes that individual scissor-switch key bodies are sealed and cannot be manually cleaned. However, the symptom pattern — **an entire column** (6/Y/H/N) affected simultaneously, not individual scattered keys — indicates the primary contamination is on the **shared column trace** in the FPC ribbon and/or ZIF connector, not solely inside individual key mechanisms. Contamination only inside key bodies would produce independent per-key failures, not a clean column pattern. Ultrasonic cleaning addresses all three contamination sites.

### Positive indicators for cleaning success

- **Ghost keypresses have stopped** — this means the residue has dried and stabilised, no longer actively migrating. Contamination is localised.
- **Partial symptom improvement after a 2-day rest period** — correct characters returned alongside incorrect ones when the keyboard was left unused for 2 days (powered off, internal keyboard disabled via Karabiner Elements). If traces were irreversibly corroded through, rest would not improve symptoms. This strongly suggests the primary mechanism is still **reversible conductive residue** rather than permanent copper trace damage.
- **Coca-Cola Zero residue** (dried acid/salt/organic film) is sufficiently soluble in water and isopropyl alcohol. Ultrasonic cavitation in an appropriate solvent should be able to dislodge and remove it even from sub-0.3 mm capillary spaces.

### Suggested cleaning focus areas

- Thoroughly clean the **ZIF connector** and the corresponding section of the **FPC ribbon cable** — this is where the column traces C7, Ca, and Cb connect and are most accessible. The Space bar column pin is also in this area.
- Ensure ultrasonic bath exposure is sufficient to reach the **FPC trace gaps** (~0.1 mm between polyimide layers) and the **sealed key switch bodies** (sub-0.3 mm capillary spaces).
- After cleaning, a **resistance measurement** between pins C7, Ca, and Cb on the ZIF connector would confirm whether the conductive bridges have been removed. Also check the Space bar column pin isolation.
- **Verification test:** press each of 6, Y, H, N, and Space in a `cat` session — each should produce exactly one character with no extras.

### Diagrams

See the [`diagrams/`](diagrams/) directory for technical illustrations (available in both SVG and PNG formats):

- [Keyboard matrix with affected column C7](diagrams/keyboard-matrix-affected-column.png)
- [Keyboard system block diagram](diagrams/keyboard-system-block-diagram.png)
- [ZIF connector detail](diagrams/zif-connector-detail.png)
- [FPC liquid damage before/after](diagrams/fpc-liquid-damage.png)
- [Scissor-switch cross-section](diagrams/scissor-switch-cross-section.png)

## Summary

A Coca-Cola Zero spill on a MacBook Pro 14" M3 Pro (serial MWJPXQ4VC4, model A2918, board 820-02757) resulted in keyboard column C7 (keys 6, Y, H, N) plus the Space bar producing multiple incorrect characters per keypress, initial ghost keypresses (since resolved), and progressive symptom worsening over days.

Diagnostic testing (March 22) precisely mapped the fault: each affected key produces its correct character plus two extras from specific other columns. Key 6 outputs `690`, Y outputs `yop`, H outputs `hl;`, N outputs `n./`, and Space outputs ` '`. This reveals that **three matrix columns** (C7, Ca carrying 9/O/L/., and Cb carrying 0/P/;/') are bridged by a single contamination zone on the FPC, where these columns are routed on physically adjacent traces despite being non-adjacent on the keyboard surface.

The root cause is **dried conductive cola residue** bridging the adjacent FPC traces for columns C7, Ca, and Cb (plus the Space bar trace) at the ZIF connector area and/or along the FPC ribbon, combined with **ongoing phosphoric acid corrosion** accelerated by thermal cycling during use. The service center's characterisation of this as a "mechanical issue" is **inaccurate** — the symptoms are electrical/chemical in nature, and the multi-column bridging pattern points to shared trace contamination rather than individual key mechanism failures.

**Recommended action:** First, test all remaining keys with `cat` to establish a complete baseline. Then proceed with ultrasonic cleaning (already offered by the service center at 1/3 the cost of replacement, 3–7 days), focusing on the cluster of adjacent FPC pins carrying C7/Ca/Cb and the Space bar trace. The partial improvement observed during a 2-day rest period confirms the contamination is still predominantly reversible conductive residue rather than permanent trace damage, making cleaning the correct first step before considering keyboard/top-case replacement.
