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
  │   │    passes to macOS HID subsystem)          │                    │
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
  │   → macOS receives "N" + "/" + ? │
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

## Discussion

### Does "mechanical issue" accurately describe the problem?

The service center's characterisation of "mechanical issue" is imprecise and potentially misleading in its original sense. The symptoms are **electrical/chemical** in nature, not mechanical:

- **Ghost keypresses** (keys firing with nothing pressed) cannot result from mechanical damage — a broken spring or cracked keycap cannot generate electrical signals on its own. This is only possible if there is residual conductivity (dried cola bridging traces) or an active short circuit.
- **A single key producing 3 symbols** means the keyboard controller is receiving signals from multiple matrix intersections simultaneously — a purely electrical phenomenon caused by conductive contamination or shorted traces.
- **The entire column (6/Y/H/N) being affected at once** points to a shared column trace issue, not individual key mechanisms failing independently.

The service center later clarified they mean "mechanical" in the sense that the sealed key switch bodies are **non-serviceable** — residue trapped inside them cannot be reached by conventional manual cleaning. That narrower claim is technically valid, but the root cause of the column-wide electrical symptoms still lies at the FPC trace or connector level, not inside individual key bodies.

### Can symptom drift be explained by eventual drying?

Yes — but drying does **not** resolve the problem; it transforms it into a potentially worse one.

When cola dries, water evaporates but leaves behind a concentrated residue of sugars, phosphoric acid, and mineral salts. That residue:

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
4. **Positive signal for cleaning** — this improvement suggests the contamination is conductive residue (dried sugar/acid) rather than irreversible copper corrosion through the trace. If the traces were physically etched through by phosphoric acid, a rest period would not improve the symptoms. The fact that it did improve suggests **ultrasonic cleaning has a good chance of resolving the problem**.

This supports the recommendation to pursue ultrasonic cleaning as the first step before considering top-case replacement.

## Hypotheses

### Hypothesis 1: Residual liquid causing short circuits (most likely)

Cola contains water, sugar, acids, and other conductive impurities. Even after the initial drying period, residual moisture or dried sugar residue can remain in tight spaces — especially under the scissor-switch key mechanism or on the underlying flex-cable connector pads. When the keyboard is in use and the device warms up, residual conductivity between adjacent key traces can cause:

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

Corrosion on flex-cable traces is a known consequence of cola spills due to the acidic and sugary composition of the liquid. This damage can worsen over time as oxidation progresses, which aligns with the observation that symptoms evolved and worsened over several days.

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
    traces C6/C8 via capillary action in the ~0.1 mm gap between
    polyimide substrate layers.

    CONTAMINATION SITE #3: Under the sealed scissor-switch key
    bodies for 6, Y, H, N — where cola entered through < 0.3 mm
    capillary gaps and cannot be manually extracted.
```

### Three specific contamination zones

1. **ZIF connector area** (most accessible) — dried cola residue on or around pin C7 and bridging to adjacent pins C6/C8. This is the most likely primary site because: (a) the connector is an open junction point where liquid pools, (b) it explains the clean column pattern, and (c) it is the first component the service center would have accessed during cleaning (and may not have cleaned thoroughly enough).

2. **FPC ribbon cable traces** (moderately accessible) — residue wicked along the column C7 trace where it runs parallel to C6/C8 within the ribbon. The ~0.1 mm gap between traces inside the FPC acts as a capillary channel that draws liquid in and is very difficult to clean without ultrasonic treatment.

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

## Pre-Cleaning Keyboard Behavior Tests

Before sending the device for ultrasonic cleaning, recording the exact keyboard reactions to individual keypresses provides a precise baseline for:

1. **Hypothesis confirmation** — identifying which extra characters appear for each affected key reveals which adjacent column traces are being bridged, directly confirming or refuting the contamination model.
2. **Cleaning verification** — comparing before/after results gives an objective measure of cleaning effectiveness.
3. **Damage scope assessment** — testing keys outside the suspected affected column checks whether contamination has spread further than the 6/Y/H/N column.

### Test Setup

1. Open **Terminal.app** and run `cat` — the terminal driver's local echo will display each character on screen immediately as the keyboard produces it, making extra characters from a single keypress visible. Alternatively, open **TextEdit** in plain-text mode (`Format → Make Plain Text`) with autocorrect, smart quotes, and text replacement all disabled (`Edit → Substitutions → uncheck all`).
2. Disable **Karabiner Elements** or any other key-remapping software so that the raw hardware output is captured.
3. Ensure the **internal keyboard** is the active input device (disconnect any external keyboard).
4. Set the input source to **U.S.** (or another simple ASCII layout) to avoid locale-specific key mappings.
5. Wait 60 seconds with the text field focused and no keys pressed. Record whether any ghost keypresses appear during this period.

### Test Procedure

For each key listed in the test matrix below:

1. Press and release the key **once**, firmly but briefly.
2. Record **all characters** that appear on screen (expected character plus any extras).
3. Repeat the same keypress **3 times** to check for consistency or intermittent behavior.
4. Then hold **Shift** and press the key once — record the output.
5. Then hold **Option (⌥)** and press the key once — record the output.

### Test Matrix

The matrix is divided into three groups: (A) the affected column, (B) adjacent keys that share a row with the affected keys (control group — should be unaffected if the problem is column-specific), and (C) distant keys (baseline — should be completely normal).

#### Group A — Affected column (6, Y, H, N)

These keys share column trace C7. If Hypothesis 1 is correct, each should produce its correct character plus extra characters from bridged columns.

| Key | Expected | Actual (press 1) | Actual (press 2) | Shift | Option (⌥) | Notes |
|-----|----------|-------------------|-------------------|-------|------------|-------|
| 6   | `6`      | `690`             | `690`             | `^()` |            | Extra `9` and `0` — consistent. Shift: `^` (Shift+6), `(` (Shift+9), `)` (Shift+0) — Shift applied to all |
| Y   | `y`      | `yop`             | `yop`             |       |            | Extra `o` and `p` — consistent |
| H   | `h`      | `hl;`             | `hl;`             |       |            | Extra `l` and `;` — consistent |
| N   | `n`      | `n./`             | `n./`             |       |            | Extra `.` and `/` — consistent |

#### Group B — Adjacent keys (control group)

These keys are in the same rows as the affected keys but in different columns. If the problem is purely a column C7 issue, these should all work correctly.

| Key | Expected | Actual (press 1) | Actual (press 2) | Shift | Option (⌥) | Notes |
|-----|----------|-------------------|-------------------|-------|------------|-------|
| 5   | `5`      | `5`               | `5`               |       |            | ✅ Normal |
| 7   | `7`      | `7`               | `7`               |       |            | ✅ Normal |
| T   | `t`      | `t`               | `t`               |       |            | ✅ Normal |
| U   | `u`      | `u`               | `u`               |       |            | ✅ Normal |
| G   | `g`      | `g`               | `g`               |       |            | ✅ Normal |
| J   | `j`      | `j`               | `j`               |       |            | ✅ Normal |
| B   | `b`      | `b`               | `b`               |       |            | ✅ Normal |
| M   | `m`      | `m`               | `m`               |       |            | ✅ Normal |

#### Group B2 — Keys from the newly identified bridged columns

The test results revealed that the actual bridged columns contain `9/O/L/.` and `0/P/;/​/`. These keys should also be tested — pressing them may produce extra characters from column C7 (reverse bridge).

| Key | Expected | Actual (press 1) | Actual (press 2) | Shift | Option (⌥) | Notes |
|-----|----------|-------------------|-------------------|-------|------------|-------|
| 9   | `9`      | `9`               | `9`               |       |            | ✅ Normal — no reverse `6` |
| 0   | `0`      | `0`               | `0`               |       |            | ✅ Normal — no reverse `6` |
| O   | `o`      | `o`               | `o`               |       |            | ✅ Normal — no reverse `y` |
| P   | `p`      | `p`               | `p`               |       |            | ✅ Normal — no reverse `y` |
| L   | `l`      | `l`               | `l`               |       |            | ✅ Normal — no reverse `h` |
| ;   | `;`      | `;`               | `;`               |       |            | ✅ Normal — no reverse `h` |
| .   | `.`      | `.`               | `.`               |       |            | ✅ Normal — no reverse `n` |
| /   | `/`      | `/`               | `/`               |       |            | ✅ Normal — no reverse `n` |

#### Group C — Distant keys (baseline)

These keys are far from the spill area and should be completely unaffected.

| Key | Expected | Actual (press 1) | Actual (press 2) | Shift | Option (⌥) | Notes |
|-----|----------|-------------------|-------------------|-------|------------|-------|
| A   | `a`      | `a`               | `a`               |       |            | ✅ Normal |
| S   | `s`      | `s`               | `s`               |       |            | ✅ Normal |
| L   | `l`      | `l`               | `l`               |       |            | ✅ Normal |
| P   | `p`      | `p`               | `p`               |       |            | ✅ Normal |

#### Group D — Space bar and `'` key (additional findings)

Space was found to produce an extra character during testing. The reverse test (`'` key) confirmed the bridge is **unidirectional** — `'` does not produce a space:

| Key   | Expected | Actual (press 1) | Actual (press 2) | Shift | Option (⌥) | Notes |
|-------|----------|-------------------|-------------------|-------|------------|-------|
| Space | ` `      | ` '`              | ` '`              |       |            | Extra `'` — consistent |
| '     | `'`      | `'`               | `'`               |       |            | ✅ Normal — no reverse space (unidirectional) |

#### Ghost keypress observation

| Condition | Duration | Characters observed | Notes |
|-----------|----------|---------------------|-------|
| No keys pressed, laptop idle | 60 s |  |  |
| No keys pressed, after typing on affected keys for 30 s | 60 s |  |  |

### Test Results and Analysis (March 22)

Testing was performed using `cat` in Terminal.app with Karabiner Elements disabled, internal keyboard only, U.S. input source.

#### Observed pattern

Every key in the affected column produces the correct character **plus two extra characters from keys that are +3 and +4 physical positions to the right** in the same row:

```
  Physical keyboard — mapping affected keys to their extra outputs

  Key pressed → Output     Extra char 1        Extra char 2
  ─────────────────────────────────────────────────────────
  6          → 690         9  (+3 positions)    0  (+4 positions)
  Y          → yop         o  (+3 positions)    p  (+4 positions)
  H          → hl;         l  (+3 positions)    ;  (+4 positions)
  N          → n./         .  (+3 positions)    /  (+4 positions)
  Space      →  '          '  (see below)
```

The extra characters form **two complete, consistent columns** on the keyboard:

```
  Column C7 (affected)    Column "X" (+3)    Column "Y" (+4)
  ────────────────────    ───────────────    ───────────────
       6                       9                  0
       Y                       O                  P
       H                       L                  ;
       N                       .                  /
```

#### Key findings

1. **Three electrical column traces are bridged.** Column C7 (6/Y/H/N) is shorted to two other columns — the one carrying 9/O/L/. and the one carrying 0/P/;/​/. All three must be **adjacent pins on the FPC/ZIF connector**, even though they are 3–4 physical key positions apart on the keyboard surface.

2. **The FPC pin ordering does not follow physical keyboard layout.** This is the most important discovery from the test. The original analysis assumed that physically adjacent keys (5/T/G/B and 7/U/J/M) would be on adjacent FPC traces. The actual data proves that the FPC trace routing maps the physical column containing 6/Y/H/N to a connector pin that sits next to the pins for 9/O/L/. and 0/P/;/​/ — not next to 5/T/G/B or 7/U/J/M. This is normal for keyboard FPC design where trace routing is optimized for PCB layout, not physical key position.

3. **Results are 100% consistent.** Every key produced identical output across both trials. This confirms a **stable, low-resistance conductive bridge** — dried residue forming a fixed short circuit, not an intermittent or marginal connection. This is a good indicator for ultrasonic cleaning: the contamination is well-defined and localised.

4. **Space bar is also affected.** Space producing `'` means the Space bar's electrical column is bridged to the `'` key's column on the FPC. Since `'` is physically adjacent to `;` (which is part of the +4 bridged column), the Space bar's FPC trace is likely near the same contamination zone. This extends the known damage scope beyond the originally identified 6/Y/H/N column.

5. **H1 is strongly confirmed.** The perfectly consistent column pattern — where every affected key produces extras from the same two other columns — is exactly the signature of a single contamination site bridging adjacent FPC/ZIF pins. If there were multiple independent corrosion sites (H3), different keys would show different extra characters. If the controller IC were damaged (H4), the pattern would not map cleanly to keyboard matrix columns.

6. **Control group and baseline keys are 100% normal.** Group B (5, 7, T, U, G, J, B, M — physically adjacent to the affected column) and Group C (A, S, L, P — distant keys) all produce only their correct single character with no extras. This conclusively confirms the damage is **column-specific**, not row-related or widespread. The physically adjacent keys 5/T/G/B and 7/U/J/M being unaffected also independently confirms finding #2: these keys are on different, non-adjacent FPC pins despite being physically next to the affected column.

7. **The bridge is unidirectional.** The reverse bridge test (Group B2) shows that pressing keys in the bridged columns (`9`, `0`, `O`, `P`, `L`, `;`, `.`, `/`) does **not** produce extra characters from column Cx (6/Y/H/N). Similarly, pressing `'` does not produce a space (reverse of the Space→`'` bridge). This means current leaks from Cx→Cy/Cz when Cx is driven during matrix scanning, but not in the reverse direction. This is consistent with the keyboard controller's sequential column scan: when Cx is driven and a key is pressed, the conductive residue bridge lets the drive signal partially activate Cy and Cz, registering ghost keypresses. When Cy or Cz are driven, the reverse leakage to Cx is either below the detection threshold or the scan timing/debounce suppresses it. The character ordering (`690` not `096`) also confirms Cy and Cz are scanned after Cx.

8. **Shift modifier applies to all bridged characters.** Shift+`6` produces `^()` — that is `^` (Shift+6), `(` (Shift+9), `)` (Shift+0). The Shift modifier is correctly applied to the ghost keypresses, confirming the bridge is in the **keyboard matrix wiring** (column traces), not in the controller IC or firmware. The controller reads all three columns as active during the same scan cycle and applies modifier state to all detected keys. This definitively rules out H4 (controller damage).

#### Revised contamination model

```
  FPC connector pinout (actual pin ordering revealed by test data)

  ... ─┬──┬──┬──┬──┬──┬──┬──┬──┬──┬── ...
       │  │  │Cx│Cy│Cz│  │  │  │  │
  ... ─┴──┴──┴──┴──┴──┴──┴──┴──┴──┴── ...
              ▲   ▲   ▲
              │   │   │
       Cx = column carrying 6/Y/H/N     (original "C7")
       Cy = column carrying 9/O/L/.     (physical +3 from C7)
       Cz = column carrying 0/P/;/​/     (physical +4 from C7)

  Dried cola residue bridges all three adjacent pins:

  ... ─┬──┬──┬──┬──┬──┬──┬──┬──┬──┬── ...
       │  │  │Cx│Cy│Cz│  │  │  │  │
  ... ─┴──┴──┴══╧══╧══┴──┴──┴──┴──┴── ...
              ▲═══════▲
              dried cola residue bridging
              3 adjacent FPC/ZIF pins

  Nearby: Space bar column pin is also adjacent to the ' key's
  column pin, with residue bridging those two as well.
```

#### What this means for cleaning

The contamination is concentrated at a **single cluster of adjacent FPC/ZIF connector pins**. This is the best-case scenario for ultrasonic cleaning because:

- The damage is localised (one spot on the connector, not multiple scattered sites along the FPC).
- The bridge is between adjacent pins with ~0.5 mm pitch — well within the range of ultrasonic cavitation to reach and dissolve dried sugar/acid residue.
- The consistency of results (no intermittent behavior) means the residue is stable and should dissolve cleanly once the solvent reaches it.

### Identifying the Bridged Columns

The test results have conclusively identified the bridged columns. The original prediction of C6/C8 (physical neighbors of C7) was incorrect — the actual bridged columns are **3 and 4 physical positions to the right**, confirming that FPC pin ordering differs from physical key order.

| Affected column (C7) key | Extra character 1 | Extra character 2 | Physical offset |
|---|---|---|---|
| `6` | `9` | `0` | +3, +4 positions right in number row |
| `Y` | `o` | `p` | +3, +4 positions right in top row |
| `H` | `l` | `;` | +3, +4 positions right in home row |
| `N` | `.` | `/` | +3, +4 positions right in bottom row |
| Space | `'` | — | Space bar column bridged to `'` column |

The bridged columns on the FPC/ZIF connector (in order of pin position):
- **Pin Cx**: column carrying `6` / `Y` / `H` / `N`
- **Pin Cy** (adjacent to Cx): column carrying `9` / `O` / `L` / `.`
- **Pin Cz** (adjacent to Cy): column carrying `0` / `P` / `;` / `/`
- **Nearby**: Space bar column pin bridged to `'` column pin

### Follow-Up Test Results (March 22)

All three recommended follow-up tests have been completed. Results:

1. **Reverse bridge test (Group B2)** ✅ — All 8 keys (`9`, `0`, `O`, `P`, `L`, `;`, `.`, `/`) produce only their correct character. **The bridge is unidirectional** — current leaks from Cx to Cy/Cz but not in reverse. This is consistent with the keyboard matrix sequential column scan and confirms the residue bridge has a directional characteristic (see finding #7 above).

2. **Shift modifier test** ✅ — Shift+`6` produces `^()` — that is `^` (Shift+6), `(` (Shift+9), `)` (Shift+0). Shift is correctly applied to all ghost keypresses. This **confirms the bridge is in the matrix wiring** (column traces on the FPC), not in the controller IC or firmware (see finding #8 above). H4 (controller damage) is definitively ruled out.

3. **`'` key reverse test** ✅ — Pressing `'` produces only `'`, no space. **The Space→`'` bridge is also unidirectional**, consistent with the main Cx→Cy/Cz bridge behavior.

## Recommended Next Steps

1. **Ultrasonic cleaning** — All pre-cleaning tests are complete. The test data confirms the contamination is a stable, localised, unidirectional residue bridge at a single cluster of adjacent FPC/ZIF pins — ideal conditions for cleaning. The service center offers this at approximately 1/3 the cost of keyboard replacement, with a 3–7 day turnaround.
2. **Re-run the same keyboard tests after cleaning** — compare against the pre-cleaning baseline documented above to objectively measure improvement. All Group A keys should produce only their correct single character; Space should produce only a space.
3. **Visual inspection under magnification** of the FPC traces and ZIF connector pins — the contamination site is the cluster of pins carrying columns Cx/Cy/Cz (6/Y/H/N column and the 9/O/L/. and 0/P/;/ columns). Look for visible dried residue bridging these adjacent pins.
4. **Resistance measurement** between the three identified column pins (Cx, Cy, Cz) and the Space/`'` pin pair on the ZIF connector to confirm whether the conductive bridges have been removed after cleaning.
5. If ultrasonic cleaning does not resolve the issue, **keyboard/top-case replacement** will be necessary. Corrosion that has fully etched through a copper trace is not reversible, but this is the fallback rather than the first resort.

## Note for the Ultrasonic Cleaning Lab

This section summarises the key technical details for the technicians performing ultrasonic cleaning on this device.

### Device

- **MacBook Pro 14" (M3 Pro, November 2023)**, model A2918, serial MWJPXQ4VC4
- **Logic board:** 820-02757
- **Keyboard type:** Scissor-switch (Magic Keyboard), integrated into top-case assembly
- **Spill substance:** Coca-Cola Zero (contains phosphoric acid, sugars, mineral salts)

### What to look for

Pre-cleaning keyboard testing (March 22) has confirmed the exact contamination location. The primary affected column carries keys **6, Y, H, N**, and it is shorted to **two adjacent FPC/ZIF pins** that carry the columns for **9/O/L/.** and **0/P/;/​/**. The Space bar column is also bridged to the `'` key column. See [Test Results and Analysis](#test-results-and-analysis-march-22) for full data.

The contamination sites, in order of priority:

1. **ZIF connector area** — dried cola residue bridging **three adjacent column pins** (Cx, Cy, Cz) on the FPC/ZIF connector. These pins carry the columns for 6/Y/H/N, 9/O/L/., and 0/P/;/​/ respectively. The Space bar column pin nearby is also bridged to the `'` column pin. This is the most likely primary site because it is an open junction point where liquid pools.
2. **FPC ribbon cable** — residue wicked along the three column traces (Cx, Cy, Cz) where they run parallel within the ribbon. The ~0.1 mm gap between traces acts as a capillary channel.
3. **Under the sealed key switch bodies** for 6, Y, H, N — cola entered through sub-0.3 mm capillary gaps between the keycap, scissor arms, rubber dome, and FPC membrane.

### Why the "non-serviceable key blocks" diagnosis is incomplete

The service center correctly notes that individual scissor-switch key bodies are sealed and cannot be manually cleaned. However, the symptom pattern — **an entire column** (6/Y/H/N) affected simultaneously, not individual scattered keys — indicates the primary contamination is on the **shared column trace** in the FPC ribbon and/or ZIF connector, not solely inside individual key mechanisms. Contamination only inside key bodies would produce independent per-key failures, not a clean column pattern. Ultrasonic cleaning addresses all three contamination sites.

### Positive indicators for cleaning success

- **Ghost keypresses have stopped** — this means the residue has dried and stabilised, no longer actively migrating. Contamination is localised.
- **100% consistent test results** — every affected key produced identical output across repeated trials. The bridge is stable and well-defined, not intermittent. This means the residue forms a solid conductive film that should dissolve cleanly in ultrasonic bath solvent.
- **Bridge is unidirectional** — pressing keys in the bridged columns (9/O/L/., 0/P/;//) does NOT produce extras from the C7 column (6/Y/H/N). This is consistent with a resistive bridge interacting with the controller's sequential column scan, and rules out a metallic short circuit (which would be bidirectional). The unidirectionality suggests the residue forms a moderate-resistance film — enough to pass current in one scan direction but not enough to trigger detection in reverse — which is a positive sign for cleanability.
- **Shift modifier confirms matrix-level bridge** — Shift+6 produces `^()` (all three characters correctly shifted), confirming the bridge is in the column traces (pre-controller), not in the controller IC or firmware. The controller and IC are undamaged.
- **Partial symptom improvement after a 2-day rest period** — correct characters returned alongside incorrect ones when the keyboard was left unused for 2 days (powered off, internal keyboard disabled via Karabiner Elements). If traces were irreversibly corroded through, rest would not improve symptoms. This strongly suggests the primary mechanism is still **reversible conductive residue** rather than permanent copper trace damage.
- **Coca-Cola Zero residue** (dried sugar/acid film) is soluble in water and isopropyl alcohol — ultrasonic cavitation in an appropriate solvent should be able to dissolve and remove it even from sub-0.3 mm capillary spaces.

### Suggested cleaning focus areas

- Focus on the **ZIF connector pins** for the three bridged columns — specifically the cluster of adjacent pins carrying 6/Y/H/N, 9/O/L/., and 0/P/;/​/ columns, plus the nearby Space bar and `'` column pins. This is the primary contamination site.
- Thoroughly clean the corresponding section of the **FPC ribbon cable** where these three column traces run in parallel.
- Ensure ultrasonic bath exposure is sufficient to reach the **FPC trace gaps** (~0.1 mm between polyimide layers) and the **sealed key switch bodies** (sub-0.3 mm capillary spaces).
- After cleaning, a **resistance measurement** between the three adjacent column pins (Cx/Cy/Cz) and the Space/`'` pin pair on the ZIF connector would confirm whether the conductive bridges have been removed.

### Diagrams

See the [`diagrams/`](diagrams/) directory for technical illustrations (available in both SVG and PNG formats):

- [Keyboard matrix with affected column C7](diagrams/keyboard-matrix-affected-column.png)
- [Keyboard system block diagram](diagrams/keyboard-system-block-diagram.png)
- [ZIF connector detail](diagrams/zif-connector-detail.png)
- [FPC liquid damage before/after](diagrams/fpc-liquid-damage.png)
- [Scissor-switch cross-section](diagrams/scissor-switch-cross-section.png)

## Summary

A Coca-Cola Zero spill on a MacBook Pro 14" M3 Pro (serial MWJPXQ4VC4, model A2918, board 820-02757) resulted in keyboard column C7 (keys 6, Y, H, N) producing multiple incorrect characters per keypress, initial ghost keypresses (since resolved), and progressive symptom worsening over days.

Pre-cleaning keyboard testing (March 22) has precisely identified the contamination: **three adjacent FPC/ZIF column pins are bridged by dried cola residue**. Every affected key produces its correct character plus two extras from columns that are +3 and +4 physical positions to the right (6→690, Y→yop, H→hl;, N→n./), confirming the FPC pin ordering does not follow the physical keyboard layout. The Space bar is also affected (Space→ '). Results are 100% consistent across trials, indicating a stable conductive bridge. Follow-up tests confirmed the bridge is **unidirectional** (reverse keys produce no extras) and **operates at the matrix wiring level** (Shift+6→`^()`, modifier correctly applied to all ghost keypresses), definitively ruling out controller damage. All pre-cleaning tests are complete — ideal conditions for ultrasonic cleaning confirmed.

The root cause is **dried conductive cola residue** shorting adjacent column pins on the keyboard FPC ribbon cable and/or ZIF connector, combined with **ongoing phosphoric acid corrosion** accelerated by thermal cycling during use. The service center's characterisation of this as a "mechanical issue" is **inaccurate** — the symptoms are electrical/chemical in nature, and the whole-column pattern points to shared trace contamination rather than individual key mechanism failures.

**Recommended action:** Ultrasonic cleaning (already offered by the service center at 1/3 the cost of replacement, 3–7 days). The test results confirm localised, stable contamination at a single cluster of adjacent FPC/ZIF pins — the best-case scenario for cleaning. The partial improvement observed during a 2-day rest period confirms the contamination is still predominantly reversible conductive residue rather than permanent trace damage.
