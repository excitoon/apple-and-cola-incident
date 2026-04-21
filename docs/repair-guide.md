# Repair Guide

← [Back to overview](../README.md)

## Recommended Next Steps

### Update: second service cleaned the connector — issue persists

A second service center has cleaned the JT200 ZIF connector (IPA clean of the connector pads and FPC contact fingers) and **the keyboard issue remained unchanged** after cleaning. This is a definitive diagnostic result: it rules out the connector contact surfaces as the contamination site, because cleaning them had no effect. The residue bridge must be located in the less accessible zones that manual connector cleaning cannot reach:

- **Zone 2 — FPC ribbon cable traces** (contamination wicked between the polyimide layers, ~0.1 mm gap, unreachable by surface wiping)
- **Zone 3 — Sealed key switch bodies** (sub-0.3 mm capillary gaps under the 6/Y/H/N scissor mechanisms)

The recommended service order has therefore changed: **manual ZIF connector cleaning has been tried and did not resolve the issue**. The next intervention is **ultrasonic cleaning**, which is the only method capable of reaching residue trapped inside the FPC trace layers and under sealed key switches.

### Manual ZIF cleaning vs. ultrasonic cleaning

~~Since the test data confirms the contamination is at the **FPC/ZIF connector pin cluster** (not scattered along the FPC or inside individual key switches), a reasonable question is whether **simple manual cleaning** of the ZIF connector area would be sufficient, avoiding the cost and turnaround of ultrasonic cleaning.~~

**Manual ZIF connector cleaning has been attempted by a second service** — the connector pads and FPC contact fingers were cleaned with IPA, and the keyboard issue remained unchanged afterward. This step is complete and did not resolve the problem.

The remaining contamination is in locations that manual cleaning cannot reach:
- **FPC ribbon cable traces** — residue wicked between the polyimide layers (~0.1 mm gap acts as a capillary channel) cannot be reached by surface swabbing; only ultrasonic cavitation can dissolve residue inside these narrow channels.
- **Sealed key switch bodies** — sub-0.3 mm capillary spaces under keycaps cannot be reached by manual cleaning.

**Recommended approach: proceed to ultrasonic cleaning.** Manual ZIF connector cleaning has been tried and did not resolve the issue, confirming the contamination is not at the connector contact surfaces. The fault is localised to the FPC trace gaps or under the sealed key switches, which only ultrasonic cavitation can reach.

### Step-by-step plan

1. ~~**Ask the service center to try manual ZIF connector cleaning first**~~ — **already attempted by second service; issue remained.** The connector is not the contamination site.
2. **Proceed directly to ultrasonic cleaning** — the contamination is in the FPC trace gaps or under key switches, which only ultrasonic cavitation can reach. The second service center has also stated they cannot ultrasonically clean the keyboard because the keyboard is integrated into the top case; this claim is inaccurate — the top case with keyboard can be cleaned as a unit, and at minimum the FPC ribbon can be cleaned independently. See [Second Service Center assessment](service-center-assessment.md#second-service-center-keyboard-not-detachable--cannot-ultrasonically-clean) for full analysis.
3. **Re-run keyboard tests after ultrasonic cleaning** — test Group A keys (`6`, `Y`, `H`, `N`, Space) and compare against the pre-cleaning baseline. All keys should produce only their correct single character.
4. **Visual inspection under magnification** of the FPC traces after cleaning — look for any remaining dried residue.
5. **Resistance measurement** between the three identified column pins (Cx, Cy, Cz) and the Space/`'` pin pair on the ZIF connector to confirm the conductive bridges have been removed.
6. If ultrasonic cleaning does not fully resolve the issue, **keyboard/top-case replacement** will be necessary. Corrosion that has fully etched through a copper trace is not reversible, but this is the fallback rather than the first resort. Note: the second service center has proposed tearing off the keyboard and replacing it with screws — this is a non-standard approach that may not address the electrical fault and is not preferred; if replacement is needed, a full OEM top-case assembly replacement is the correct procedure. Indicative pricing for a genuine Apple A2918 top-case assembly: ~$378 (third-party / eBay) or ~$500–$700+ at an AASP/IRP (including labour); reduced or covered if under AppleCare+. See [Second Service Center assessment — Cost-to-benefit ratio](service-center-assessment.md#assessment-of-claim-2-tear-off-keyboard--replace-on-screws) for the full breakdown.

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

1. ~~**ZIF connector area**~~ — **cleaned by second service; issue persisted.** Manual IPA cleaning of the connector pads and FPC contact fingers did not resolve the problem, ruling out the connector contact surfaces as the contamination site.
2. **FPC ribbon cable** — residue wicked along the three column traces (Cx, Cy, Cz) where they run parallel within the ribbon. The ~0.1 mm gap between traces acts as a capillary channel. This is now the **primary contamination site**.
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

**Important update:** A second service center has **cleaned the JT200 ZIF connector** (IPA clean of connector pads and FPC contact fingers) and the **issue remained after cleaning**. This definitively rules out the connector contact surfaces as the contamination site. The contamination is therefore located deeper inside the keyboard assembly, in the less accessible zones:

- **Primary focus: FPC ribbon cable traces** — the three bridged column traces (Cx, Cy, Cz) run in parallel within the FPC ribbon between the ZIF connector and the key matrix. Cola residue has wicked into the ~0.1 mm gap between the polyimide layers. This is unreachable by surface wiping but directly addressable by ultrasonic cavitation. Thoroughly clean the full length of the FPC ribbon, paying particular attention to the section where the Cx/Cy/Cz traces run in parallel.
- **Secondary focus: sealed key switch bodies** for 6/Y/H/N — ultrasonic bath exposure should be sufficient to reach the sub-0.3 mm capillary gaps within the scissor mechanisms.
- ~~ZIF connector pins~~ — **manual cleaning attempted by second service; issue persisted. Not the contamination site.**
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
