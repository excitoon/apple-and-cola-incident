# Repair Guide

← [Back to overview](../README.md)

## Recommended Next Steps

### Manual ZIF cleaning vs. ultrasonic cleaning

Since the test data confirms the contamination is at the **FPC/ZIF connector pin cluster** (not scattered along the FPC or inside individual key switches), a reasonable question is whether **simple manual cleaning** of the ZIF connector area would be sufficient, avoiding the cost and turnaround of ultrasonic cleaning.

**Manual ZIF connector cleaning** (what a service technician can do):
1. Open the ZIF latch, remove the FPC ribbon cable
2. Clean the exposed FPC contact pads and ZIF socket spring contacts with isopropyl alcohol (IPA ≥99%) and a lint-free swab or fine brush
3. Inspect under magnification for visible dried residue bridging adjacent pads
4. Reassemble and test

**This may be sufficient** if the contamination is limited to the exposed contact surfaces at the ZIF junction. The test data supports this being the primary site — the ZIF connector is an open junction point where spilled liquid pools and dries, and the clean column-specific pattern is consistent with adjacent pins being bridged at a single location.

**However, there are risks that manual cleaning alone may not be enough:**
- If residue has **wicked along the FPC traces** between the polyimide layers (~0.1 mm gap acts as a capillary channel), it cannot be reached by surface swabbing — only ultrasonic cavitation can dissolve residue inside these narrow channels.
- If residue has entered the **sealed key switch bodies** (sub-0.3 mm capillary spaces under keycaps), manual cleaning cannot reach it.
- The ZIF connector has **~0.5 mm pitch pins** — residue between adjacent pins can be difficult to fully remove with manual methods, especially if it has formed a thin conductive film in the gap between pads.

**Recommended approach: try manual ZIF cleaning first.** It is the simplest and cheapest intervention. If the service center can open and clean the ZIF connector area with IPA, re-run the Group A tests (`6`, `Y`, `H`, `N`, Space) immediately after reassembly. If the extra characters are gone, the problem is solved. If symptoms persist or partially improve, proceed to ultrasonic cleaning to address residue in less accessible locations (FPC trace gaps, under key switches).

### Step-by-step plan

1. **Ask the service center to try manual ZIF connector cleaning first** — open the ZIF latch, clean FPC pads and socket contacts with IPA (≥99%), inspect under magnification for visible residue on the cluster of adjacent pins carrying columns Cx/Cy/Cz and the Space/`'` pin pair.
2. **Re-run keyboard tests after manual cleaning** — test Group A keys (`6`, `Y`, `H`, `N`, Space) and compare against the pre-cleaning baseline. All keys should produce only their correct single character.
3. **If manual cleaning resolves the issue** — done. No ultrasonic cleaning needed.
4. **If symptoms persist** — proceed to **ultrasonic cleaning**. The service center offers this at approximately 1/3 the cost of keyboard replacement, with a 3–7 day turnaround. The contamination is then in the FPC trace gaps or under key switches, which only ultrasonic cavitation can reach.
5. **Re-run keyboard tests after ultrasonic cleaning** — compare against pre-cleaning baseline to objectively measure improvement.
6. **Visual inspection under magnification** of the FPC traces and ZIF connector pins after cleaning — look for any remaining dried residue.
7. **Resistance measurement** between the three identified column pins (Cx, Cy, Cz) and the Space/`'` pin pair on the ZIF connector to confirm the conductive bridges have been removed.
8. If neither cleaning method resolves the issue, **keyboard/top-case replacement** will be necessary. Corrosion that has fully etched through a copper trace is not reversible, but this is the fallback rather than the first resort.

## Note for the Ultrasonic Cleaning Lab

This section summarises the key technical details for the technicians performing ultrasonic cleaning on this device.

### Device

- **MacBook Pro 14" (M3 Pro, November 2023)**, model A2918, serial MWJPXQ4VC4
- **Logic board:** 820-02757
- **Keyboard type:** Scissor-switch (Magic Keyboard), integrated into top-case assembly
- **Spill substance:** Coca-Cola Zero (contains phosphoric acid, ionic salts/preservatives, artificial sweeteners, caramel-color residue)

### What to look for

Pre-cleaning keyboard testing (March 22) has confirmed the exact contamination location through **80+ individual tests** (single-key Groups A–D plus multi-key Groups E–H). The primary affected column carries keys **6, Y, H, N**, and it is shorted to **two adjacent FPC/ZIF pins** that carry the columns for **9/O/L/.** and **0/P/;/​/**. The Space bar column is also bridged to the `'` key column. The bridge measures approximately **30 kΩ forward / 70 kΩ reverse**, forming a continuous resistive film. See [Test Results and Analysis](testing.md#test-results-and-analysis-march-22) and [Simultaneous Multi-Key Press Test](testing.md#simultaneous-multi-key-press-test) for full data.

The contamination sites, in order of priority:

1. **ZIF connector area** — dried cola residue bridging **three adjacent column pins** (Cx, Cy, Cz) on the FPC/ZIF connector. These pins carry the columns for 6/Y/H/N, 9/O/L/., and 0/P/;/​/ respectively. The Space bar column pin nearby is also bridged to the `'` column pin. This is the most likely primary site because it is an open junction point where liquid pools.
2. **FPC ribbon cable** — residue wicked along the three column traces (Cx, Cy, Cz) where they run parallel within the ribbon. The ~0.1 mm gap between traces acts as a capillary channel.
3. **Under the sealed key switch bodies** for 6, Y, H, N — cola entered through sub-0.3 mm capillary gaps between the keycap, scissor arms, rubber dome, and FPC membrane.

### Why the "non-serviceable key blocks" diagnosis is incomplete

The service center correctly notes that individual scissor-switch key bodies are sealed and cannot be manually cleaned. However, the symptom pattern — **an entire column** (6/Y/H/N) affected simultaneously, not individual scattered keys — indicates the primary contamination is on the **shared column trace** in the FPC ribbon and/or ZIF connector, not solely inside individual key mechanisms. Contamination only inside key bodies would produce independent per-key failures, not a clean column pattern. Ultrasonic cleaning addresses all three contamination sites.

### Positive indicators for cleaning success

- **Ghost keypresses have stopped** — this means the residue has dried and stabilised, no longer actively migrating. Contamination is localised.
- **100% consistent test results** — every affected key produced identical output across repeated trials. The bridge is stable and well-defined, not intermittent. This means the residue forms a solid conductive film that should dissolve cleanly in ultrasonic bath solvent.
- **Bridge is unidirectional for single-key presses, bidirectional under parallel load** — pressing a single ghost-column key does NOT produce extras from the C7 column, confirming a moderate-resistance film (~30/70 kΩ forward/reverse). However, multi-key tests (Group H) proved that pressing two ghost-column keys simultaneously (one from Cy + one from Cz) creates parallel reverse paths that push voltage above the detection threshold, producing reverse ghosts. This means the bridge is a **dissolvable resistive film** (not a metallic short), and even partial cleaning that increases resistance by 2–3× should eliminate all ghost keypresses in both directions.
- **Shift modifier confirms matrix-level bridge** — Shift+6 produces `^()` (all three characters correctly shifted), confirming the bridge is in the column traces (pre-controller), not in the controller IC or firmware. The controller and IC are undamaged.
- **Partial symptom improvement after a 2-day rest period** — correct characters returned alongside incorrect ones when the keyboard was left unused for 2 days (powered off, internal keyboard disabled via Karabiner Elements). If traces were irreversibly corroded through, rest would not improve symptoms. This strongly suggests the primary mechanism is still **reversible conductive residue** rather than permanent copper trace damage.
- **Coca-Cola Zero residue** (dried acid/salt/organic film) is sufficiently soluble in water and isopropyl alcohol. Ultrasonic cavitation in an appropriate solvent should be able to dislodge and remove it even from sub-0.3 mm capillary spaces.

### Suggested cleaning focus areas

- Focus on the **ZIF connector pins** for the three bridged columns — specifically the cluster of adjacent pins carrying 6/Y/H/N, 9/O/L/., and 0/P/;/​/ columns, plus the nearby Space bar and `'` column pins. This is the primary contamination site.
- Thoroughly clean the corresponding section of the **FPC ribbon cable** where these three column traces run in parallel.
- Ensure ultrasonic bath exposure is sufficient to reach the **FPC trace gaps** (~0.1 mm between polyimide layers) and the **sealed key switch bodies** (sub-0.3 mm capillary spaces).
- After cleaning, a **resistance measurement** between the three adjacent column pins (Cx/Cy/Cz) and the Space/`'` pin pair on the ZIF connector would confirm whether the conductive bridges have been removed.

### Diagrams

See the [`diagrams/`](../diagrams/) directory for technical illustrations (available in both SVG and PNG formats):

- [Keyboard matrix with affected column C7](../diagrams/keyboard-matrix-affected-column.png)
- [Keyboard system block diagram](../diagrams/keyboard-system-block-diagram.png)
- [ZIF connector detail](../diagrams/zif-connector-detail.png)
- [FPC liquid damage before/after](../diagrams/fpc-liquid-damage.png)
- [Scissor-switch cross-section](../diagrams/scissor-switch-cross-section.png)
- [Bridge unidirectional circuit model](../diagrams/bridge-unidirectional-circuit.svg) — why the dried cola bridge appears unidirectional
- [Ohm's law voltage divider analysis](../diagrams/ohms-law-voltage-divider.svg) — forward vs reverse calculations
