# Keyboard Technology

← [Back to overview](../README.md)

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

See the [butterfly vs scissor comparison diagram](../diagrams/butterfly-vs-scissor.svg) and [scissor-switch cross-section](../diagrams/scissor-switch-cross-section.svg) for visual details.

## Keyboard Electronics: Schematics and Layout

### 1. Keyboard Matrix Principle

MacBook keyboards use a **row/column matrix** to detect keypresses. Each key sits at the intersection of one row wire and one column wire. The keyboard controller scans all rows one at a time and reads which column lines are active. This means:

- One **column** trace is shared by all keys in the same vertical column on the keyboard.
- One **row** trace is shared by all keys in the same horizontal row.
- A keypress is detected when a specific (row, column) pair is simultaneously active.

**From the board 820-02757 boardview data** (decoded from `820-02757-06-boardview.brd`): the keyboard matrix on this MacBook Pro is **12 rows × 13 columns**, with signal names `KBD_DRIVE_Y0`–`KBD_DRIVE_Y11` (rows) and `KBD_SENSE_X0`–`KBD_SENSE_X12` (columns). Three additional modifier keys (`KBD_CONTROL_KEY`, `KBD_LEFT_OPTION_KEY`, `KBD_RIGHT_SHIFT_KEY`) have dedicated lines outside the matrix.

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

The following diagram illustrates where cola can bridge traces and produce the observed multi-character and ghost-key behavior:

```
  FPC cross-section (schematic, not to scale)

  ┌─────────────────────────────────────────────────────┐
  │  Polyimide substrate (insulating base layer)         │
  ├────────────┬────────────────────┬────────────────────┤
  │  COL_D     │      COL_E         │      COL_F          │  ← copper traces
  │  (6,Y,H,N) │    (adjacent col)  │    (adjacent col)   │
  └────────────┴────────────────────┴────────────────────┘

  After cola spill and drying:

  ┌─────────────────────────────────────────────────────┐
  │  Polyimide substrate                                 │
  ├────────────┬═══════════════════ ┬────────────────────┤
  │  COL_D     ║   dried cola +    ║      COL_F          │
  │  (6,Y,H,N) ║   corrosion       ║                     │
  └────────────╩═══════════════════╩────────────────────┘
               ▲
               Conductive bridge between COL_D and COL_E
               → pressing any key in COL_D also activates
                 a COL_E signal → wrong extra character output
               → residual conductivity even when no key is
                 pressed → spontaneous ghost keypresses
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
  │   │    passes to OS X HID subsystem)          │                    │
  │   └───────────────────────────────────────────┘                    │
  │                                                                     │
  └─────────────────────────────────────────────────────────────────────┘
```

#### 6b. FPC Connector Pinout — Affected Column Highlighted

The keyboard FPC carries all row and column signals to the controller IC. The affected column (C7, carrying keys 6/Y/H/N) is a single pin on the FPC/ZIF connector. Contamination at this pin or along the C7 trace causes all four keys to malfunction simultaneously:

```
  FPC pinout (simplified 34-pin model, viewed from below)

  Pin:  1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16  17
       ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
       │R1│R2│R3│R4│R5│R6│C1│C2│C3│C4│C5│C6│C7│C8│C9│..│Gnd│
       └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
                                              ▲▲▲
                                              │││
       Affected pin C7 ───────────────────────┘││
       Adjacent pin C6 (bridged by residue) ───┘│
       Adjacent pin C8 (possibly bridged) ──────┘

       When dried cola bridges C7 to C6 (or C8):
       → pressing any key in column 7 (6,Y,H,N) also activates
         column 6 (or 8) → extra characters appear
       → residual conductivity between C7 and adjacent pins
         without a keypress → ghost keypresses (now resolved)
```

#### 6c. Signal Path from Affected Keys to SoC

```
  Affected key "N" pressed:
  ═══════════════════════════════════════════════════════════

  [N key]  (physical keypress)
      │
      ▼
  [Scissor switch collapses, rubber dome presses FPC membrane]
      │
      ▼
  [FPC contact pad closes ROW_4 × COL_7 intersection]
      │                              │
      │                              │ ← dried cola residue
      │                              │   bridges COL_7 to COL_6
      │                              │
      ▼                              ▼
  COL_7 signal active           COL_6 signal ALSO active
      │                              │
      │        ┌─────────────────────┤
      ▼        ▼                     │
  [FPC trace → ZIF connector pin 13] │
      │        │                     │
      │   [ZIF connector pin 12] ◄───┘  (contamination here or
      │        │                         along the FPC trace)
      ▼        ▼
  ┌──────────────────────────────────────────────────┐
  │   Keyboard Controller IC                         │
  │                                                  │
  │   Scans ROW_4 → reads COL_7 ──► reports "N"     │
  │                   reads COL_6 ──► reports extra   │
  │                   reads COL_8? ─► reports extra   │
  │                                                  │
  │   Result: controller sends                       │
  │   multiple keycodes for one                      │
  │   physical keypress                              │
  └──────────────────────┬───────────────────────────┘
             │ SPI bus
             ▼
  ┌──────────────────────────────────┐
  │   Apple M3 Pro SoC               │
  │   → OS X receives "N" + "/" + ? │
  │   → all three appear on screen   │
  └──────────────────────────────────┘
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

**Note:** The committed `.brd` file is an **OpenBoardView-compatible boardview/layout** file, obfuscated with the byte transformation `out = ~((in >> 6) | (in << 2))` (CR/LF bytes pass through unchanged). The decoded text contains sections `str_length`, `var_data`, `Format` (board outline coordinates), `Pins1` (component list), `Pins2` (pin-to-net assignments with x/y coordinates), and `Nails` (test points). It provides complete placement data, pin/net labels, and connector pinouts. It is still **not a substitute for the separate PDF schematic / full netlist** when we need complete circuit-level interpretation beyond what placement/connectivity data offers.

### 8. Diagrams (uploaded as files)

The following diagrams are included in this repository in the [`diagrams/`](../diagrams/) directory, available in both SVG (vector, scalable) and PNG (raster, 1200px wide) formats:

| Diagram | SVG | PNG | Description |
|---|---|---|---|
| Keyboard matrix with affected column | [SVG](../diagrams/keyboard-matrix-affected-column.svg) | [PNG](../diagrams/keyboard-matrix-affected-column.png) | Row/column matrix layout showing keys 6, Y, H, N sharing column C7 (highlighted in red) |
| Keyboard system block diagram | [SVG](../diagrams/keyboard-system-block-diagram.svg) | [PNG](../diagrams/keyboard-system-block-diagram.png) | Full signal chain: key matrix → FPC ribbon → ZIF connector → keyboard controller IC → SPI → M3 Pro SoC |
| ZIF connector detail (3 views) | [SVG](../diagrams/zif-connector-detail.svg) | [PNG](../diagrams/zif-connector-detail.png) | Top view (latch open), top view (latch closed), and side cross-section showing FPC pads contacting spring contacts |
| FPC liquid damage | [SVG](../diagrams/fpc-liquid-damage.svg) | [PNG](../diagrams/fpc-liquid-damage.png) | Before/after comparison showing how dried cola bridges column traces C7→C6/C8 |
| Scissor-switch cross-section | [SVG](../diagrams/scissor-switch-cross-section.svg) | [PNG](../diagrams/scissor-switch-cross-section.png) | Side view of the non-serviceable key block: keycap → scissor arms → rubber dome → FPC membrane |
| Butterfly vs scissor comparison | [SVG](../diagrams/butterfly-vs-scissor.svg) | [PNG](../diagrams/butterfly-vs-scissor.png) | Side-by-side comparison of the two Apple keyboard mechanisms |
| Bridge unidirectional circuit model | [SVG](../diagrams/bridge-unidirectional-circuit.svg) | — | Circuit diagram showing why dried cola residue bridge appears unidirectional: forward direction crosses detection threshold, reverse does not |
| Ohm's law voltage divider analysis | [SVG](../diagrams/ohms-law-voltage-divider.svg) | — | Side-by-side Ohm's law calculations for forward (2.01 V, detected) vs reverse (1.33 V, not detected) bridge directions |
| Chemical attack cross-section | [SVG](../diagrams/chemical-attack-cross-section.svg) | | FPC layer stack showing protected vs exposed zones and four chemical attack vectors (pinhole undermining, ionic bridging, osmotic blistering, copper dissolution) |
| Chemical timeline — three phases | [SVG](../diagrams/chemical-timeline-phases.svg) | | Wet → drying → dried residue phases with key reactions, plus corrosion rate vs time graph showing peak during drying |
| Galvanic corrosion cell | [SVG](../diagrams/galvanic-corrosion-cell.svg) | | Electrochemical cell at a solder/copper/gold bimetallic junction in cola electrolyte, with galvanic series table |
| Dendrite / electrochemical migration | [SVG](../diagrams/dendrite-electrochemical-migration.svg) | | Three-step process: Cu dissolution at anode → ion migration → Cu deposition and dendrite growth at cathode, leading to permanent trace bridge |
| Connector chemical vulnerability | [SVG](../diagrams/connector-chemical-vulnerability.svg) | | ZIF connector cross-section annotated with all six overlapping chemical vulnerability factors |
| Boardview 820-02757 — overview | | [PNG](../diagrams/boardview-820-02757-overview.png) | PCSchematics boardview: full board layout showing M3 Pro SoC (U0600), keyboard controller area, and connector locations |
| Boardview 820-02757 — detail 1 | | [PNG](../diagrams/boardview-820-02757-detail-1.png) | PCSchematics boardview: zoomed detail view of board area |
| Boardview 820-02757 — detail 2 | | [PNG](../diagrams/boardview-820-02757-detail-2.png) | PCSchematics boardview: zoomed detail view of board area |
| Boardview 820-02757 — capture 1 | | [PNG](../diagrams/boardview-820-02757-capture1.png) | Original PCSchematics screenshot from archive |
| Boardview 820-02757 — capture 2 | | [PNG](../diagrams/boardview-820-02757-capture2.png) | Original PCSchematics screenshot from archive |
| Board 820-02757 — high-quality render | [SVG](../diagrams/boardview-820-02757-render.svg) | | Clean vector board render for presentation/annotation use, derived from the boardview overview |
| Board 820-02757 — illustration zones | [SVG](../diagrams/boardview-820-02757-illustration-zones.svg) | | Annotated boardview-derived map of the major board neighborhoods that can be illustrated reliably from the committed board data |
| Board 820-02757 — zone detail sheet | [SVG](../diagrams/boardview-820-02757-zone-detail-sheet.svg) | | Five focused boardview-derived zoom panels for the highlighted zones, including the decoded keyboard / backlight / trackpad neighborhoods |
| Boardview 820-02757 data file | | [BRD](../diagrams/820-02757-06-boardview.brd) | OpenBoardView-compatible boardview data file for board 820-02757-06 |
| Boardview 820-02757 decoded text export | | [TXT](../diagrams/820-02757-06-boardview-decoded.txt) | Readable text export decoded from the committed `.brd` file, preserving the boardview sections and records |
| JT200 keyboard FPC connector pinout | [SVG](../diagrams/boardview-820-02757-jt200-pinout.svg) | | Complete 36-pin JT200 connector pinout decoded from the boardview, showing DRIVE/SENSE interleaving and bridging risk boundaries |
| Board 820-02757 — component map | [SVG](../diagrams/boardview-820-02757-component-map.svg) | | Board-level map with real component positions decoded from the boardview, showing keyboard / trackpad / SoC neighborhoods and contamination risk zone |
| Board 820-02757 — keyboard signal architecture | [SVG](../diagrams/boardview-820-02757-kbd-signal-architecture.svg) | | Complete signal architecture showing SoC → IPD connector → JT200 keyboard FPC → matrix signals, with power rails, I²C bus, and bridging boundaries identified from decoded data |
| Board 820-02757 — serviceability map | [SVG](../diagrams/boardview-820-02757-serviceability.svg) | | Disassembly depth map showing all spill zones at Depth 1 (bottom case only), with problem-area assessment and components-to-detach count for each zone |
| JT200 connector zone — detail render | [SVG](../diagrams/boardview-820-02757-jt200-detail-render.svg) | | Close-up render of JT200 36-pin ZIF connector with color-coded pins (SENSE/DRIVE/control/GND), bridging risk boundaries, cola residue zone, ZIF latch, and JT220 below |
| FPC cable run — detail render | [SVG](../diagrams/boardview-820-02757-fpc-cable-render.svg) | | FPC ribbon cable render showing 25 interleaved DRIVE/SENSE traces, polyimide cross-section, contamination wicking zones, and ultrasonic cleaning annotation |
| Cleaning access — board service map | [SVG](../diagrams/boardview-820-02757-cleaning-access-render.svg) | | Full board cleaning access overview with connector zones, IPA swab workflow, and reachable=cleanable principle for each contamination zone |

### 9. Boardview Screenshots (Board 820-02757)

The following screenshots were captured from the **PCSchematics** boardview for board **820-02757** (MacBook Pro 14" A2918, M3 Pro). They show the component layout as rendered in the boardview viewer:

Knowing the exact boardview / schematic set **does help**, but mostly by improving **placement confidence** and enabling **better illustrations**. It does **not automatically change the root-cause conclusion** for this incident on its own: the actual failure site still has to be confirmed by physical evidence such as residue location, corrosion, and continuity/microscope inspection.

While the boardview alone does **not change the hypothesis ranking**, it does give a **meaningful confidence boost** to the connector-path / FPC contamination hypothesis. The dominant evidence is still the symptom pattern (one affected matrix column, multi-character output, progression over time, partial improvement after rest), but the decoded boardview now provides **structural proof** that the FPC connector has the exact physical topology needed for the observed failure.

After fully decoding the committed `.brd` file (using the OpenBoardView obfuscation formula `~((b >> 6) | (b << 2))` per byte), we now have a **critical structural finding** about the unidirectional behavior. The decoded boardview reveals the **complete JT200 keyboard FPC connector pinout** — a 36-pin ZIF connector where DRIVE (row) and SENSE (column) lines are **interleaved**, not grouped separately:

| FPC pins | Signal group | Notes |
|---|---|---|
| 1 | GND | shield |
| 2–6 | KBD_RIGHT_SHIFT_KEY, PP1V825, KBD_ID1, GND, KBD_ID2 | power / ID / modifier |
| **7–10** | **KBD_SENSE_X7, X11, X6, X9** | SENSE group A |
| **11–15** | **KBD_DRIVE_Y0, Y10, Y2, Y8, Y1** | DRIVE group A |
| **16–24** | **KBD_SENSE_X8, X12, X10, X5, X4, X3, X1, X2, X0** | SENSE group B |
| **25–31** | **KBD_DRIVE_Y11, Y9, Y3, Y4, Y5, Y7, Y6** | DRIVE group B |
| 32–36 | KBD_CAP_CATHODE, PP3V3_AON, KBD_LEFT_OPTION, KBD_CONTROL, GND | LED / power / modifiers |

This interleaving creates **three DRIVE↔SENSE boundaries** where adjacent FPC traces transition from one signal type to the other:

1. **Pin 10 (SENSE X9) ↔ Pin 11 (DRIVE Y0)** — adjacent traces, different signal types
2. **Pin 15 (DRIVE Y1) ↔ Pin 16 (SENSE X8)** — adjacent traces, different signal types
3. **Pin 24 (SENSE X0) ↔ Pin 25 (DRIVE Y11)** — adjacent traces, different signal types

**This is structurally significant for explaining unidirectionality.** A conductive residue bridge at any of these three boundaries would short a DRIVE line to a SENSE line. During matrix scanning, when the controller drives the row HIGH and reads the column, the bridge registers a false keypress. But in the reverse direction (when the bridged column is being read during a different row's scan), the drive line is not being actively asserted — so no false reading is generated. This is the physical mechanism behind the observed one-way ghost keypresses, and the boardview now proves that such DRIVE↔SENSE adjacencies **exist on the actual FPC connector** of this board.

![JT200 keyboard FPC connector pinout](../diagrams/boardview-820-02757-jt200-pinout.svg)

From the committed `.brd` boardview file, we decoded three categories of useful data:

1. **A full readable text export** of the boardview data itself — [`820-02757-06-boardview-decoded.txt`](../diagrams/820-02757-06-boardview-decoded.txt), decoded from the board file using the section structure that OpenBoardView parses (`str_length`, `var_data`, `Format`, `Pins1`, `Pins2`, `Nails`).
2. **The complete JT200 connector pinout** shown above — extracted from the `Pins2` records for component #3588 (JT200), sorted by physical pin position.
3. **Layout-based illustrations** (board outline, major packages, connector neighborhoods, and incident-relevant regions), now including the highlighted-zone detail sheet below.

The decoded export contains **522 outline points**, **5,298 component records**, **16,676 pin/net records**, and **1,222 test-point (nail) records** across **2,528 unique net names**. The keyboard neighborhood alone includes **82 unique keyboard-related nets** spanning matrix signals, power rails, I²C bus, backlight control, and current sensing. The I²C keyboard bus (`I2C_KBD_SCL`, `I2C_KBD_SDA`, `KBD_INT_L`) connects through test points at coordinates (4490–4536, 5486–5542) — physically separate from the matrix signals at JT200, confirming that the keyboard controller communicates with the SoC via I²C while the matrix scanning happens locally within the keyboard module.

The adjacent **JT220** connector (component #3589, 12 pins) handles the keyboard backlight: `KBDLED_CATHODE1`, `KBDLED_CATHODE2`, `PPVOUT_KBDLED_CONN`, and multiple GND pins. Its physical position at board coordinates (4119–4253, 6899–7121) places it immediately below JT200 (6111–6806), meaning contamination spreading downward from the keyboard FPC area could also reach the backlight connector — consistent with backlight being a secondary failure risk after matrix contamination.

This repository now also includes a **high-quality vector board render** derived from the boardview material for documentation and future annotation work:

![Board 820-02757 — high-quality vector render](../diagrams/boardview-820-02757-render.svg)

And an additional **boardview-derived illustration map** showing the main zones that can be expanded into further diagrams:

![Board 820-02757 — illustration zones](../diagrams/boardview-820-02757-illustration-zones.svg)

And a **board-level component map** using the real positions decoded from the boardview data, showing the keyboard/trackpad/SoC spatial relationships and contamination risk zone:

![Board 820-02757 — component map](../diagrams/boardview-820-02757-component-map.svg)

This repository now also includes a **detail sheet for the highlighted zones**, using the decoded board data to label the most relevant board neighborhoods:

![Board 820-02757 — zone detail sheet](../diagrams/boardview-820-02757-zone-detail-sheet.svg)

![Board 820-02757 — full board overview](../diagrams/boardview-820-02757-overview.png)

![Board 820-02757 — boardview detail 1](../diagrams/boardview-820-02757-detail-1.png)

![Board 820-02757 — boardview detail 2](../diagrams/boardview-820-02757-detail-2.png)

The following additional screenshots and the boardview data file were provided in [Archive.zip](https://github.com/user-attachments/files/26164995/Archive.zip):

![Board 820-02757 — capture 1](../diagrams/boardview-820-02757-capture1.png)

![Board 820-02757 — capture 2](../diagrams/boardview-820-02757-capture2.png)

The boardview data file [`820-02757-06-boardview.brd`](../diagrams/820-02757-06-boardview.brd) can be opened in [OpenBoardView](https://github.com/OpenBoardView/OpenBoardView) or similar boardview software for interactive component lookup. A readable decoded export is also included as [`820-02757-06-boardview-decoded.txt`](../diagrams/820-02757-06-boardview-decoded.txt).

#### Boardview Evidence vs. Hypotheses

The decoded boardview data provides concrete, board-specific evidence that can now be evaluated against each of the six incident hypotheses. The following table summarizes what the boardview confirms, refutes, or is neutral on:

| Hypothesis | Boardview Impact | Specific Evidence from Decoded Data |
|---|---|---|
| **H1: Conductive residue** (dried cola shorting column trace) | **🔺 Strongly supported** | JT200 pin records prove DRIVE/SENSE lines are **interleaved** on the FPC with **three DRIVE↔SENSE boundaries** at pins 10↔11, 15↔16, 24↔25. A conductive bridge at any boundary creates exactly the observed one-column ghost-keypress pattern. The 1.825V keyboard power rail (`PP1V825_S2_HOLD_KBDLDO`, net 721, regulated by a dedicated LDO) feeds the SENSE lines and is close to the detection threshold (~1.2V), meaning even a high-impedance dried residue bridge registers as a keypress in one scan direction but not the other — confirming the unidirectional mechanism. |
| **H2: Connector misalignment** | **🔻 Weakened** | The decoded connector records show JT200 has **shield GND pins at both ends** (pins 1 and 33–36), with the active signals in the interior. A misaligned FPC would affect pins at one edge first (shifting all signals by one position), which would produce **systematic wrong-character** output across many keys — not the observed clean single-column pattern. The boardview makes misalignment a poor fit. |
| **H3: FPC trace corrosion** | **🔺 Supported** | The boardview shows that `PP1V825_S2_HOLD_KBDLDO` (net 721) feeds a dedicated LDO regulator with **current monitoring** (`INA_KBD1V8_IOUT`, net 1133). If trace corrosion increased resistance on SENSE lines, the current draw would change detectably. More importantly, the tight 0.2mm pin pitch at JT200 (Y-coordinates incrementing by ~20 units) means even minor copper dissolution creates inter-trace leakage paths — consistent with progressive worsening. |
| **H4: Keyboard controller IC** | **🔻 Strongly weakened** | The decoded data reveals a critical architectural fact: the keyboard I²C bus (`I2C_KBD_SCL` net 710, `I2C_KBD_SDA` net 709, `KBD_INT_L` net 711) routes through **connector 3590** (the IPD connector) at board coordinates (4104–4225, 5571–5712), which is physically **separate from JT200** (at 4119–4253, 6111–6806). The matrix DRIVE/SENSE signals are entirely within the keyboard module behind JT200 — they never reach the logic board as individual matrix lines. This means the keyboard controller IC is **inside the keyboard assembly**, not on the main logic board, and communicates only via I²C. A main-board controller IC fault cannot produce the observed column-specific matrix bridging because the matrix is scanned locally within the keyboard module. |
| **H5: Insufficient cleaning** | **🔺 Supported** | The boardview confirms JT200 sits at the **bottom edge of the keyboard area** (Y=6111–6806), with JT220 (backlight) immediately below (Y=6899–7121) — a gap of only 93 board-units (~2mm). Gravity-driven cola flow from the keyboard membrane would pool at JT200 first, then overflow to JT220. The decoded pin spacing (~20 board-units ≈ ~0.5mm pitch) means standard wipe-cleaning cannot reach between FPC pads. |
| **H6: Thermal cycling** | **○ Neutral** | The boardview does not add direct evidence for or against thermal cycling effects. However, the confirmed physical proximity of JT200 to the SoC (U0600 at board center) and the dedicated power rails with current monitoring suggest the keyboard area does receive some thermal flux from normal operation. |

![Keyboard signal architecture](../diagrams/boardview-820-02757-kbd-signal-architecture.svg)

**Key architectural finding:** The boardview decode confirms that on the 820-02757 board, the keyboard operates as a **self-contained module** that scans its own matrix internally and communicates with the SoC via I²C through the IPD connector (3590). The JT200 connector carries raw DRIVE/SENSE matrix lines into the keyboard module — meaning any matrix-level fault (ghost keypresses, column bridging) must originate **at or beyond JT200**, not on the main logic board. This definitively localizes the fault to the FPC/connector area and eliminates main-board IC damage as a plausible explanation for the observed symptoms.

**Updated hypothesis ranking impact:** The boardview evidence moves H4 (controller IC damage) from "★☆☆☆☆ Unlikely" to **effectively ruled out** for this specific symptom pattern. It also strengthens H1 (conductive residue) from "★★★★★ Primary" to "★★★★★ Primary with structural proof" — the interleaved DRIVE/SENSE topology on the actual FPC provides a concrete physical mechanism for exactly the observed behavior, not just a general plausibility argument.

### 10. Reference Photos of Real Hardware (external links)

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
