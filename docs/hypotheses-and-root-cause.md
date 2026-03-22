# Hypotheses and Root Cause Analysis

← [Back to overview](../README.md)

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

**Boardview update:** The decoded boardview data **rules out main-board controller IC damage** as a cause of the observed column-bridging symptoms. The keyboard I²C bus routes through the IPD connector (component 3590), physically separate from JT200. The 25 raw DRIVE/SENSE matrix lines exist only within the keyboard module — the controller IC scans the matrix locally and reports key events over I²C. See the [Boardview Evidence vs. Hypotheses](keyboard-technology.md#boardview-evidence-vs-hypotheses) analysis in section 9 for full details.

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
| **1** | **H1: Conductive residue** (dried cola shorting column trace) | **★★★★★ Primary — quantitative proof** | ✅ Confirmed | Clean column pattern; **multi-key tests quantify bridge: ~30 kΩ forward, ~70 kΩ reverse, continuous 2D film**; parallel-path reverse ghosts (H1–H4) match voltage divider calculations exactly; scan-cycle persistence explains fast-typing glitch; 100% reproducible across 36+ tests |
| **2** | **H5: Insufficient initial cleaning** | **★★★★☆ Primary** | ✅ Active | N was already faulty at pickup; 2-hour surface clean cannot reach sub-0.3 mm gaps; **boardview shows ~0.5mm FPC pin pitch at JT200** |
| **3** | **H3: FPC trace corrosion** | **★★★☆☆ Contributing — reduced** | ⚠️ Slowed | Progressive worsening over days (historical); **but multi-key tests show perfectly stable bridge resistance — no drift across the entire testing session**; rest-period improvement; system has reached quasi-equilibrium (see corrosion status below) |
| **4** | **H6: Thermal cycling accelerator** | **★★☆☆☆ Contributing — diminished** | ⚠️ Slowed | Historical worsening during use; but current stability of bridge parameters suggests thermal cycling is no longer significantly changing the residue film |
| **5** | **H2: Connector misalignment** | **★★☆☆☆ Ruled out as sole cause** | ❌ Tested | Reseating connector did not resolve symptoms; **boardview shows GND shield pins at both ends would shift all signals — doesn't match clean column pattern** |
| **6** | **H4: Keyboard controller IC damage** | **☆☆☆☆☆ Ruled out** | ❌ Eliminated | **Boardview proves keyboard controller is inside the keyboard module, communicates only via I²C through IPD connector (3590) — matrix scanning happens locally, not on main board** |

#### What changed after multi-key testing (Groups E–H)

The multi-key tests produced **quantitative confirmation** of the residue bridge model that significantly sharpens the hypothesis ranking:

- **H1 upgraded from "structural proof" to "quantitative proof."** The single-key tests showed the bridge existed; the multi-key tests measured its electrical properties. The ~30/70 kΩ forward/reverse resistance values, the voltage divider thresholds (2.01 V forward → detected, 1.33 V reverse → not detected, 1.65 V parallel → detected), the scan-channel locking behavior (F7/G8), and the scan-cycle persistence in cross-row reverse ghosts (P+L→`phl`) all match a single continuous resistive film model. Every test — all 36+ combinations across Groups E through H — produced results exactly consistent with one contamination site. No anomalies, no inconsistencies.

- **H3 downgraded from ★★★★☆ to ★★★☆☆.** The bridge resistance is perfectly stable: identical behavior across tests performed over the entire session, with no measurable drift. If active corrosion were significantly progressing, we would expect changing resistance values (worsening ghost characters, or conversely improving as corrosion products build up insulating layers). The stability indicates the system has reached equilibrium — corrosion has largely stopped (see below).

- **H6 downgraded from ★★★☆☆ to ★★☆☆☆.** With the bridge parameters stabilized, thermal cycling is no longer a significant accelerator. Its role was historical (during the active worsening phase).

#### Corrosion status — has it stopped?

**Largely yes.** The evidence points to the corrosion process having reached a quasi-equilibrium:

1. **Bridge resistance is perfectly stable.** Across 36+ test combinations in Groups E–H (spanning different key pairs, hold durations, and activation patterns), the bridge produces identical, 100% reproducible results. If phosphoric acid were still actively dissolving copper, the resistance would be changing — either increasing (as corrosion products accumulate) or decreasing (as traces thin and leakage paths widen). Neither is happening.

2. **No new symptoms.** The affected column set (Cx/Cy/Cz + Space/') has not expanded. No previously unaffected keys have acquired ghost characters during the testing period. Active corrosion would eventually bridge additional traces.

3. **The acid has been consumed.** Phosphoric acid in dried cola residue is a finite reagent. At the concentrations present in a single spill droplet (~0.06% H₃PO₄ by weight in liquid, concentrated ~100–1000× during drying), the total amount of acid is small — on the order of micrograms. After several days of reacting with copper, tin, and nickel on the FPC pads, much of the acid has been neutralized by forming metal phosphate salts (which are themselves part of the conductive residue film, but the corrosive agent is depleted).

4. **Hygroscopic equilibrium.** The dried film has reached moisture equilibrium with ambient humidity. The electrochemical corrosion rate in a dried, equilibrated film is orders of magnitude slower than during the active drying phase (when acid concentration was peaking). The "dangerous period" described in the chemical timeline — the drying phase at 10 min – 2 hours — is long past.

**What this means for cleaning:** The stabilization is actually **good news**. It means:
- The damage is **not getting worse** — there is no urgency to clean immediately to prevent further corrosion.
- The residue film is **fixed and localized** — it won't migrate or spread to new pins.
- The bridge is **still conductive residue, not permanent trace damage** — the rest-period improvement and the stable (not increasing) resistance both confirm the traces are intact underneath. Cleaning should restore full function.

**Winner: H1 + H5**, with **H3** as historical contributor now largely spent. The primary cause is conductive cola residue forming a stable resistive film on the shared column C7 trace cluster, left behind by insufficient initial cleaning. The corrosion from phosphoric acid contributed to the historical worsening but has now reached equilibrium — the acid is largely consumed, the film is stable, and the traces appear intact. **The multi-key tests provide quantitative proof**: the actual bridge resistance values, voltage thresholds, parallel-path behavior, scan-channel locking, and scan-cycle persistence all match the single-contamination-site model with no anomalies across 36+ test combinations. Cleaning prognosis remains excellent.

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
    │        │JT200 (36pin)│ ◄── CONTAMINATION SITE #1   │
    │        │ ZIF connector│     DRIVE/SENSE interleaved │
    │        │@(4119,6111) │     3 bridging boundaries   │
    │        └──────┬──────┘                              │
    │               │ matrix lines stay in keyboard module│
    │        ┌──────┴──────┐                              │
    │        │  Keyboard    │ ◄── Controller inside module│
    │        │ Controller IC│     (NOT on main board)     │
    │        └──────┬──────┘                              │
    │               │ I²C bus only (SCL/SDA/INT)          │
    │        ┌──────┴──────┐                              │
    │        │IPD connector│ ◄── Component 3590           │
    │        │@(4104,5571) │     Separate from JT200      │
    │        └──────┬──────┘                              │
    │               │ I²C to SoC                          │
    │        ┌──────┴──────┐                              │
    │        │  M3 Pro SoC │                              │
    │        │   (U0600)   │                              │
    │        └─────────────┘                              │
    └─────────────────────────────────────────────────────┘

    CONTAMINATION SITE #2: Along the FPC ribbon cable itself,
    where cola wicked between the DRIVE/SENSE traces via
    capillary action in the ~0.5 mm pitch FPC gaps. Boardview
    confirms 25 matrix lines (12 DRIVE + 13 SENSE) routed
    through JT200, interleaved rather than grouped.

    CONTAMINATION SITE #3: Under the sealed scissor-switch key
    bodies for 6, Y, H, N — where cola entered through < 0.3 mm
    capillary gaps and cannot be manually extracted.
```

### Three specific contamination zones

> **Note:** The keyboard backlight is confirmed working — this rules out JT220 (backlight connector, ~2 mm below JT200) as a damage site and proves the contamination is spatially concentrated at the JT200 DRIVE/SENSE pin area, not spread across the entire connector neighborhood. Similarly, JT400 (IPD connector carrying the keyboard I²C bus) is ruled out because it would cause total keyboard failure, not selective column bridging.

1. **JT200 ZIF connector area** (most accessible) — dried cola residue on or around the DRIVE↔SENSE boundary pins of the 36-pin JT200 connector (boardview component #3588, at coordinates 4119–4253, 6111–6806). The decoded boardview confirms three specific DRIVE↔SENSE boundaries (pins 10↔11, 15↔16, 24↔25) where a conductive bridge produces exactly the observed one-column ghost-keypress pattern. This is the most likely primary site because: (a) the connector is an open junction point where liquid pools, (b) the interleaved DRIVE/SENSE topology creates bridging vulnerability at specific pin boundaries, and (c) the ~0.5mm pin pitch makes thorough cleaning very difficult.

2. **FPC ribbon cable traces** (moderately accessible) — residue wicked along the interleaved DRIVE/SENSE traces within the 36-conductor FPC ribbon. The boardview confirms 25 matrix lines (12 DRIVE + 13 SENSE) are routed through this cable, and their interleaved arrangement means DRIVE and SENSE traces run physically adjacent for the full length of the FPC — any cola wicking along the cable creates bridging potential at multiple points.

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

### Boardview-confirmed serviceability of spill areas

The decoded boardview data lets us **prove** — not just argue — that the contamination sites are in physically serviceable locations. Below is the complete disassembly depth analysis for each zone, including the exact number of components an engineer must detach to reach each site.

![Spill area serviceability map](../diagrams/boardview-820-02757-serviceability.svg)

#### Disassembly depth: what must be removed to reach each zone

The MacBook Pro 14" A2918 follows Apple's standard disassembly hierarchy. Every contamination zone is reached through the same initial steps:

| Step | Action | Tool | Time |
|------|--------|------|------|
| **0** | Power down, disconnect charger | — | 1 min |
| **1** | Remove 6× P5 Pentalobe bottom-case screws | P5 Pentalobe driver | 2 min |
| **2** | Release clips, slide bottom case forward, lift off | Suction cup + plastic spudger | 2 min |
| **3** | Disconnect battery (pull-tab connector near front-center) | Plastic spudger | 1 min |

**After step 3, the logic board top surface is fully exposed** — including all three keyboard/backlight connectors (JT200, JT220, JT400) along the top edge of the board. Total: **1 component detached** (bottom case), **~5 minutes**.

#### Detailed serviceability per zone

| Zone | Component | Board position | Disassembly depth | Components to detach | Could this be our problem area? | Serviceable? | Method | Time estimate |
|------|-----------|----------------|-------------------|---------------------|-------------------------------|--------------|--------|---------------|
| **1** | **JT200** (keyboard FPC ZIF, 36 pins) | Top edge of board, X 4119–4253, Y 6111–6806 | **Depth 1** — bottom case only | **1** (bottom case) | ✅ **Yes — most likely primary site.** Decoded pinout proves DRIVE/SENSE interleaving with 3 bridging boundaries at pins 10↔11, 15↔16, 24↔25. The connector is an open junction where liquid pools — cola flowing from the keyboard hits this point first. | ✅ **Yes — manually** | Lift ZIF latch, remove FPC, clean both the connector pads and FPC contact fingers with IPA + lint-free swab under magnification | 10–15 min |
| **2** | **FPC ribbon cable** (25 interleaved DRIVE/SENSE traces) | Runs from JT200 upward through top-case toward key matrix | **Depth 1** — bottom case only (cable visible along its run) | **1** (bottom case) | ⚠️ **Yes — possible secondary site.** If cola wicked along the FPC between JT200 and the key matrix, the interleaved DRIVE/SENSE traces (~0.5 mm pitch) could have residue bridges at any point along the cable. However, the column-wide symptom pattern is more consistent with a single-point bridge at the connector (Zone 1) than distributed contamination along the cable. | ⚠️ **Yes — ultrasonically** | FPC is enclosed between polyimide layers with ~0.1 mm gaps — surface wiping cannot reach internal traces. Requires ultrasonic bath in IPA/specialized solvent. | 3–7 day turnaround (ultrasonic service) |
| **3** | **Sealed key switch bodies** (6, Y, H, N keys) | Top-case interior, above FPC membrane | **Depth 1** — bottom case only (keys visible from underside through membrane) | **1** (bottom case) | ⚠️ **Unlikely as primary site.** Contamination inside individual key bodies cannot explain the entire column being affected simultaneously — that requires a shared trace-level short. However, residue under key switches could contribute to intermittent per-key issues and would explain why individual keys sometimes stick or feel gummy. | ⚠️ **Yes — ultrasonically only** | Scissor mechanisms have sub-0.3 mm capillary gaps; cannot be manually opened without breaking. Ultrasonic cavitation is specifically designed for this geometry. | 3–7 day turnaround (ultrasonic service) |
| **4** | **JT400** (IPD connector, 50 pins) | Top edge of board, X 4104–4225, Y 5571–5949 | **Depth 1** — bottom case only | **1** (bottom case) | ❌ **No — not a problem area.** JT400 carries the keyboard I²C bus (SCL/SDA/INT), trackpad signals, and IPD controller SPI bus. A fault here would cause **total keyboard failure** (no keys working at all) or trackpad issues — not the observed selective column bridging. The matrix DRIVE/SENSE lines never pass through JT400; they exist only within the keyboard module behind JT200. | ✅ **Yes — manually** | Standard ZIF connector, same procedure as JT200. | 10–15 min |
| — | **JT220** (backlight, 12 pins) | Top edge of board, X 4119–4253, Y 6899–7121; only ~2 mm below JT200 | **Depth 1** — bottom case only | **1** (bottom case) | ❌ **No — backlight is working.** The user confirms the keyboard backlight functions normally, which means JT220 and its signals (KBDLED_CATHODE1/2, PPVOUT_KBDLED_CONN) are not affected by contamination. JT220 would be cleaned as part of the same connector-area intervention regardless, since it's only ~2 mm from JT200. | ✅ **Yes — manually** | Same procedure as JT200; cleaned together in one pass. | Included in JT200 cleaning time |

#### Key finding: all contamination zones are at Disassembly Depth 1

Every identified spill zone is accessible after removing only the bottom case (6 screws + clips) and disconnecting the battery — a total of **1 component to detach** and **~5 minutes of disassembly time**. No zone requires:
- ❌ Logic board removal
- ❌ Battery removal (only disconnection)
- ❌ Display hinge disassembly
- ❌ Fan or heatsink removal
- ❌ Any soldering, reballing, or board-level rework

This is Apple's **shallowest possible service depth** — the same level required for routine operations like battery replacement or fan cleaning.

```
  MacBook Pro 14" A2918 — Disassembly depth map

  ╔═══════════════════════════════════════════════════════════╗
  ║  DEPTH 0: External (no disassembly)                      ║
  ║  └─ keyboard surface, ports, display                     ║
  ╠═══════════════════════════════════════════════════════════╣
  ║  DEPTH 1: Bottom case removed (6 screws + clips)    ◄──── ALL SPILL ZONES HERE
  ║  ├─ JT200 keyboard FPC connector  (Zone 1) ★ PRIMARY    ║
  ║  ├─ JT220 backlight connector     (backlight OK)         ║
  ║  ├─ JT400 IPD connector           (not a problem area)   ║
  ║  ├─ FPC ribbon cable              (Zone 2)               ║
  ║  ├─ Key switch bodies             (Zone 3)               ║
  ║  ├─ Battery connector                                    ║
  ║  └─ Fan, speakers, logic board (visible but not removed) ║
  ╠═══════════════════════════════════════════════════════════╣
  ║  DEPTH 2: Battery removed (3 pull-tabs + adhesive)       ║
  ║  └─ Under-battery area, trackpad cable routing           ║
  ╠═══════════════════════════════════════════════════════════╣
  ║  DEPTH 3: Logic board removed (10+ screws, 8+ connectors)║
  ║  └─ Board underside, BGA pads, shielded areas            ║
  ╚═══════════════════════════════════════════════════════════╝
```

#### Why the problem is almost certainly at JT200 (Zone 1)

The boardview data, combined with the symptom pattern, strongly localizes the fault to Zone 1:

1. **JT200 sits at the bottom edge of the keyboard area** (Y = 6111–6806), exactly where gravity-driven cola flow would pool first. Cola flowing down from the keyboard surface hits this open junction point before wicking into harder-to-reach FPC gaps and key bodies.

2. **The decoded JT200 pinout proves DRIVE/SENSE interleaving** with three specific bridging boundaries — a conductive residue bridge at any one of these creates exactly the observed one-column ghost-keypress pattern.

3. **The keyboard controller is inside the keyboard module** (behind JT200, communicating via I²C through JT400 at Y = 5571–5949). Matrix scanning happens locally — a fault at JT200 or within the FPC is the only way to get column-specific bridging.

4. **Backlight is working** — since JT220 sits only ~2 mm below JT200, the fact that backlight works means the contamination is spatially concentrated at JT200's DRIVE/SENSE pin area (Y = 6111–6806), not spread broadly across the entire connector neighborhood.

5. **No board-level trace repair is needed.** The 25 DRIVE/SENSE matrix lines exist only within the keyboard FPC — they never run as exposed traces on the main logic board.

**Conclusion: the spill areas are in serviceable places, all at the shallowest possible disassembly depth.** The primary damage site (JT200 ZIF connector) is a standard FPC connector accessible by removing 6 screws and lifting the bottom case — a routine 5-minute procedure. An engineer needs to detach only **1 component** (the bottom case) to inspect and clean the primary contamination site. The secondary sites (FPC traces, key bodies) are reachable by ultrasonic cleaning if manual ZIF cleaning alone does not fully resolve the symptoms.

#### Reachable = cleanable: why access depth determines serviceability

A critical point for the repair assessment: **if an engineer can physically reach a component, that component is also cleanable.** This is not a coincidence — it follows from the connector and cable design:

| Component | Why reachable = cleanable | Cleaning method |
|-----------|--------------------------|-----------------|
| **JT200 ZIF connector** | ZIF (Zero Insertion Force) connectors are **designed for repeated disconnect/reconnect**. The latch lifts, the FPC slides out freely, exposing both the connector pads and the FPC contact fingers. Both surfaces are flat, exposed copper/gold — directly accessible to a swab. | IPA-soaked lint-free swab under magnification. Wipe connector pads, wipe FPC fingers, dry, reseat. |
| **JT220 backlight connector** | Same ZIF design as JT200, same access procedure. Only ~2 mm away — cleaned in the same pass. | Same as JT200 (included in same cleaning step). |
| **JT400 IPD connector** | Same ZIF design, same edge of board. Accessible after bottom case removal. | Same IPA swab procedure, though this connector is not a problem area. |
| **FPC cable traces** | The FPC is a flat ribbon — once disconnected from JT200, the entire cable surface is exposed. However, **internal** traces (between polyimide layers) cannot be reached by surface wiping. | Surface: IPA wipe. Internal: requires ultrasonic bath (IPA or specialized solvent) to reach sub-0.1 mm gaps between polyimide layers. |
| **Key switch bodies** | Scissor mechanisms have sealed capillary gaps (sub-0.3 mm). Cannot be opened without breaking the mechanism. | Ultrasonic cavitation only — the sound waves create micro-bubbles that penetrate gaps too small for liquid flow. |

The key insight is that **ZIF connectors are field-serviceable by design** — they exist specifically so that cables can be disconnected, inspected, and reconnected during manufacturing and repair. The latch mechanism ensures no soldering is needed. This means the gap between "I can see the connector" and "I can clean the connector" is exactly **one latch flip** — there is no additional disassembly required to go from inspection to cleaning.

For the FPC cable and key bodies, the gap between "reachable" and "cleanable" is wider — surface contamination can be wiped, but internal contamination between layers requires ultrasonic cleaning. This is why the recommended service order is:
1. **First**: Clean JT200 connector (manual, 10–15 min) — fixes the problem if the bridge is at the connector pads
2. **Second**: Inspect and surface-clean the FPC cable (manual, 5 min) — catches visible residue on the cable surface
3. **Third**: Ultrasonic bath for FPC + key bodies (professional service, 3–7 day turnaround) — only if steps 1–2 don't resolve the issue

#### Detailed renders of contamination zones

The following detailed renders show each contamination zone at high magnification, with color-coded signal types and cleaning access annotations:

**JT200 Connector Zone — Close-up Detail:**

![JT200 connector detail render](../diagrams/boardview-820-02757-jt200-detail-render.svg)

This render shows the full 36-pin JT200 ZIF connector with each pin color-coded by signal type (SENSE = blue, DRIVE = red, control = green, GND = gray). The three DRIVE↔SENSE bridging risk boundaries are marked with warning indicators — these are the exact physical locations where a conductive residue bridge between adjacent ~0.5 mm-pitch traces produces the observed ghost keypresses. The JT220 backlight connector is shown below, confirmed working (green checkmark). The cola residue danger zone indicates where gravity-driven liquid pools at the connector junction.

**FPC Cable Run — Trace Interleaving and Cross-Section:**

![FPC cable render](../diagrams/boardview-820-02757-fpc-cable-render.svg)

This render shows the FPC ribbon cable running from JT200 to the keyboard module, with the 25 interleaved DRIVE/SENSE traces visible in cross-section. The polyimide-copper-polyimide sandwich structure has ~0.1 mm internal gaps where cola can wick by capillary action and become trapped — unreachable by surface wiping but accessible by ultrasonic cleaning.

**Cleaning Access Overview — Complete Board Service Map:**

![Cleaning access render](../diagrams/boardview-820-02757-cleaning-access-render.svg)

This render shows the full board from the bottom (case removed) with all three connector zones highlighted, the cleaning workflow for JT200, and the "REACHABLE = CLEANABLE" principle applied to each zone. The recommended service sequence follows the path of least resistance: start at JT200 (most likely fault site, easiest to clean), then inspect the FPC cable, then ultrasonic clean only if manual cleaning doesn't resolve the issue.

## Most Probable Root Cause

The most probable explanation is a **combination of Hypotheses 1 and 5**: the initial service-center cleaning was insufficient to remove cola residue from the keyboard FPC traces and sealed key switch bodies. Sticky, conductive residue (dried cola) remained on the keyboard flex cable, creating a stable resistive bridge across three adjacent keyboard matrix column pins (Cx/Cy/Cz, carrying columns for 6/Y/H/N, 9/O/L/., and 0/P/;/​/). Historical corrosion from phosphoric acid (H3) contributed to the initial worsening phase, but has now **largely stopped** — the bridge resistance is perfectly stable across 36+ multi-key tests with no measurable drift.

This accounts for:

1. The affected keys forming a clean column pattern.
2. Multiple symbols being generated by a single keypress (exactly 2 ghosts per key, matching the 3-pin bridge).
3. Spontaneous keypresses (now resolved as residue dried).
4. Symptoms worsening over time as residue dried and corrosion progressed (historical — now stabilized).
5. **Partial improvement after a 2-day rest period** — the fact that symptoms improved (correct characters returned) during a period of non-use strongly suggests the primary cause is conductive residue rather than irreversible trace damage. This is a positive indicator for cleaning.
6. **Bridge bidirectionality under parallel load** — pressing two ghost-column keys simultaneously (one from each bridged column) creates parallel reverse current paths that boost voltage above threshold, explaining the fast-typing glitch where ghost characters from the affected column appear during rapid typing of normally-unaffected keys.
7. **Scan-cycle persistence** — cross-row ghost-column key pairs (P+L, ;+.) also trigger reverse ghosts, because the residue film retains charge between consecutive row scans. This directly explains the originally reported fast-typing glitch.

The connector misalignment (Hypothesis 2) may have introduced additional artifacts but is unlikely to be the primary cause since reseating the connector did not resolve the issue.
