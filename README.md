# Apple and Cola Incident

## Incident Description

On March 18, a Coca-Cola Zero was spilled over an Apple MacBook Pro M3 Pro. The owner, unfamiliar with macOS, attempted to power it off but accidentally put it to sleep instead. The laptop was then gently wiped, a napkin was placed between the screen and keyboard, and it was positioned upside down for approximately 30 minutes to allow liquid to drain.

After waiting, the MacBook did not turn on, so it was taken to a service center. The service center was able to power it on, at which point it was immediately shut down again. The technicians spent about 2 hours cleaning the device.

## Observed Symptoms

Upon receiving the device back from the service center:

- Key **N** did not work.
- The service center indicated keyboard replacement would be required, citing a "mechanical issue" — later clarified to mean that the individual key switch blocks are sealed units that cannot be manually cleaned; residue trapped inside them cannot be reached by conventional methods.
- After returning home, the entire column containing **H**, **Y**, **6**, and **N** was affected.
- **N** was producing incorrect characters (a **/** symbol and other unintended characters) instead of or in addition to `N`.
- Extra symbols were appearing spontaneously from the keyboard without any keys being pressed.
- Over the following days the picture evolved: **N** now produces 3 different symbols (including `N` itself), and similar multi-character behavior is present for the other keys in the affected column.

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

### 6. Reference Photos of Real Hardware

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

## Most Probable Root Cause

The most probable explanation is a **combination of Hypotheses 1, 3, and 5**: the initial service-center cleaning was insufficient to remove cola residue from the keyboard FPC traces and sealed key switch bodies. Sticky, conductive residue (dried cola) remained on the keyboard flex cable, creating persistent shorts across a single keyboard matrix column (H, Y, 6, N). Ongoing corrosion from phosphoric acid, accelerated by thermal cycling during normal use (Hypothesis 6), caused progressive worsening. This accounts for:

1. The affected keys forming a clean column pattern.
2. Multiple symbols being generated by a single keypress.
3. Spontaneous keypresses.
4. Symptoms worsening over time as residue dried and corrosion progressed.

The connector misalignment (Hypothesis 2) may have introduced additional artifacts but is unlikely to be the primary cause since reseating the connector did not resolve the issue.

## Recommended Next Steps

1. **Ultrasonic cleaning (recommended first step)** — The service center offers this at approximately 1/3 the cost of keyboard replacement, with a 3–7 day turnaround. This is the appropriate treatment: ultrasonic cavitation reaches inside sealed key switch bodies and under FPC traces where no manual cleaning can. With ghost presses already resolved and symptoms stabilized, the window for effective cleaning is still open.
2. **Visual inspection under magnification** of the FPC traces in the affected column for corrosion, cracks, or residue bridges — ideally performed as part of or after the ultrasonic process.
3. **Resistance measurement** between the column trace shared by H/Y/6/N and adjacent traces to confirm whether a short is still present after cleaning.
4. If ultrasonic cleaning does not resolve the issue, **keyboard/top-case replacement** will be necessary. Corrosion that has fully etched through a copper trace is not reversible, but this is the fallback rather than the first resort.
