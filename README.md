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

The serial prefix **MWJ** identifies a 2024-batch MacBook Pro 14" with M3 Pro chip. The logic board number **820-02757** is the key identifier for locating component-level schematics and boardview files (see [Board-Level Schematic References](docs/keyboard-technology.md#7-board-level-schematic-references)).

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

## Table of Contents

The investigation is organized into the following documents:

| Document | Description |
|---|---|
| **[Keyboard Technology](docs/keyboard-technology.md)** | Butterfly vs scissor mechanisms, keyboard matrix electronics, FPC/ZIF connector details, boardview analysis, and board-level schematic references |
| **[Chemical Analysis](docs/chemical-analysis.md)** | Coca-Cola Zero residue chemistry, corrosion mechanisms, chemical timeline phases, and why the FPC connector area is the most critical site |
| **[Service Center Assessment](docs/service-center-assessment.md)** | Service center's "non-serviceable key blocks" position, and drying analysis showing inadequate cleaning/drying of the ZIF connector |
| **[Hypotheses and Root Cause](docs/hypotheses-and-root-cause.md)** | Discussion of symptom patterns, six hypotheses with ranking, physical damage localization, boardview-confirmed serviceability, and the most probable root cause |
| **[Pre-Cleaning Tests](docs/testing.md)** | Complete test methodology and results (80+ tests across Groups A–H), including multi-key press tests, bridge unidirectionality analysis, and voltage divider model |
| **[Repair Guide](docs/repair-guide.md)** | Recommended next steps (manual ZIF cleaning → ultrasonic → replacement), step-by-step plan, and technical notes for the ultrasonic cleaning lab |

## Summary

A Coca-Cola Zero spill on a MacBook Pro 14" M3 Pro (serial MWJPXQ4VC4, model A2918, board 820-02757) resulted in keyboard column C7 (keys 6, Y, H, N) producing multiple incorrect characters per keypress, initial ghost keypresses (since resolved), and progressive symptom worsening over days.

Pre-cleaning keyboard testing (March 22) has precisely identified the contamination through **80+ individual tests**: **three adjacent FPC/ZIF column pins are bridged by dried cola residue** forming a continuous resistive film (~30 kΩ forward, ~70 kΩ reverse). Every affected key produces its correct character plus two extras from columns that are +3 and +4 physical positions to the right (6→690, Y→yop, H→hl;, N→n./), confirming the FPC pin ordering does not follow the physical keyboard layout. The Space bar is also affected (Space→ '). Results are 100% consistent across all trials, indicating a stable conductive bridge. Follow-up tests confirmed the bridge is **unidirectional for single-key presses** but **bidirectional under parallel load** (pressing two ghost-column keys simultaneously produces reverse ghosts — Group H), and **operates at the matrix wiring level** (Shift+6→`^()`, modifier correctly applied to all ghost keypresses), definitively ruling out controller damage.

Multi-key testing (Groups E–H, 36+ combinations) provided **quantitative proof** of the residue bridge model: bridge dominance (first-held affected key captures the scan channel — F7/G8), universal auto-repeat blocking by the controller's anti-ghosting algorithm (G3–G7, G10), ghost-suppression-via-O (cross-column suppression from the ghost-column side — G7 discovery), and critically, **reverse ghost generation** through parallel paths (H1–H4) and **scan-cycle persistence** (cross-row Cy+Cz pairs produce Cx ghosts — P+L→`phl`). The scan-cycle persistence finding directly explains the originally reported fast-typing glitch: any two ghost-column keys pressed in quick succession during fast typing can inject affected-column characters.

**Corrosion status: largely stopped.** Bridge resistance is perfectly stable across all 36+ multi-key tests (no drift), no new symptoms have appeared, the phosphoric acid reagent is largely consumed, and the dried film has reached hygroscopic equilibrium. The damage is not getting worse.

The root cause is **dried conductive cola residue** shorting adjacent column pins on the keyboard FPC ribbon cable and/or ZIF connector, with **historical phosphoric acid corrosion** (now stabilized) that contributed to the initial worsening. The service center's characterisation of this as a "mechanical issue" is **inaccurate** — the symptoms are electrical/chemical in nature, and the whole-column pattern points to shared trace contamination rather than individual key mechanism failures.

**Drying analysis conclusion:** The evidence confirms that the service center's 2-hour cleaning intervention failed in two compounding ways: **(1) the ZIF connector was not properly cleaned** — cola residue was left on the exposed connector pads (key N was already malfunctioning at pickup, and the ~30–70 kΩ bridge resistance indicates dissolvable organic/acid film that a targeted IPA cleaning would have removed), and **(2) the connector area was not properly dried** — residual liquid trapped by capillary forces in the ZIF slot underwent acid concentration during natural drying, the device was powered on while the area was still wet (applying 3.3 V scanning bias that drove electrochemical migration), and the resulting residue film hardened before the owner could intervene. Proper procedure — opening the ZIF latch, flushing pads with ≥99% IPA, inspecting under magnification, force-drying, and only then powering on — takes 10–15 minutes and would have prevented the persistent contamination.

**Recommended action:** Try **manual ZIF connector cleaning first** (open ZIF latch, clean FPC pads and socket contacts with IPA ≥99%, inspect under magnification) — this is the simplest and cheapest intervention for contamination confirmed at the connector pin cluster. If symptoms persist after manual cleaning, proceed to **ultrasonic cleaning** (offered by the service center at ~1/3 the cost of replacement, 3–7 day turnaround) to address residue that may have wicked into FPC trace gaps or under sealed key switches. The test results confirm localised, stable contamination at a single cluster of adjacent FPC/ZIF pins — the best-case scenario for cleaning. The partial improvement observed during a 2-day rest period confirms the contamination is still predominantly reversible conductive residue rather than permanent trace damage.
