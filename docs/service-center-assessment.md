# Service Center Assessment

← [Back to overview](../README.md)

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


## Drying Analysis: Inadequate Cleaning and Drying of the ZIF Connector

The accumulated evidence — keyboard test results, chemical process timeline, physical contamination localisation, and observed symptom evolution — is sufficient to conclude that the service center's 2-hour cleaning intervention failed in two specific ways at the ZIF connector area:

1. **The ZIF connector was not properly cleaned.**
2. **The ZIF connector area was not properly dried.**

These are not separate problems — they compound each other, and together they explain why the keyboard symptoms persisted and worsened after the device was returned from service.

### Evidence that the ZIF connector was not properly cleaned

| # | Evidence | Source |
|---|----------|--------|
| 1 | **Key N was already malfunctioning at pickup from service.** The device was returned with N producing incorrect characters — meaning conductive cola residue was already present on the FPC/ZIF connector at the time the service center completed its work. | [Observed Symptoms](../README.md#observed-symptoms) |
| 2 | **Three adjacent FPC/ZIF pins are bridged by stable dried cola residue.** Testing confirmed that columns Cx (6/Y/H/N), Cy (9/O/L/.), and Cz (0/P/;//) are shorted through a ~30–70 kΩ conductive film. This residue sits directly on the exposed connector pads — the most physically accessible part of the keyboard assembly. | [Test Results and Analysis](testing.md#test-results-and-analysis-march-22) |
| 3 | **Bridge resistance indicates organic/acid residue, not metallic corrosion.** The 30–70 kΩ range is characteristic of dried sugar/acid/salt film. A copper dendrite or solder bridge would be <1 kΩ. This type of residue is easily visible as a brown or amber film under magnification and is soluble in ≥99% IPA. | [What this tells us about the residue](testing.md#what-this-tells-us-about-the-residue) |
| 4 | **Cleaning the ZIF connector is a straightforward procedure.** It requires opening the ZIF latch, removing the FPC ribbon, and swabbing the exposed pads and socket contacts with IPA — a routine operation that takes minutes. The connector is the most accessible component in the keyboard assembly. | [Manual ZIF cleaning vs. ultrasonic cleaning](repair-guide.md#manual-zif-cleaning-vs-ultrasonic-cleaning) |
| 5 | **The ZIF connector area is the most chemically critical site.** It concentrates every vulnerability: exposed copper pads, ENIG plating with pinholes, tight pitch (~0.5 mm), bimetallic junction with socket spring contacts, and capillary retention of liquid. Any cleaning procedure that does not specifically address this area is incomplete. | [Why the FPC connector area is the most chemically critical site](chemical-analysis.md#why-the-fpc-connector-area-is-the-most-chemically-critical-site) |
| 6 | **The service center focused on "non-serviceable key blocks."** Their diagnosis centred on sealed scissor-switch mechanisms rather than the FPC/ZIF connector. This framing suggests the connector pads may not have been specifically inspected or cleaned — the technicians were looking at the wrong part of the problem. | [Service Center's Position](#service-centers-position-non-serviceable-key-blocks) |
| 7 | **Glitch drift confirms residue was left on the connector.** If the connector had been properly cleaned, there would be no conductive residue to undergo the drying-phase concentration process, and the fault pattern would have been static (either working or not). Instead, the symptoms evolved qualitatively over days — N-only at first, then the full column (6/Y/H/N), then three-column bridging (Cx/Cy/Cz), with ghost keypresses appearing and later disappearing as the residue transitioned from wet electrolyte to stable dried film. This progressive, multi-stage glitch drift is the signature of a contamination site that was never cleaned: the residue was still undergoing physical and chemical changes (drying, concentrating, migrating, hardening) for days after service. A properly cleaned connector would show either no fault or a fixed, unchanging fault — not a drifting one. | [Observed Symptoms](../README.md#observed-symptoms), [Can symptom drift be explained by eventual drying?](hypotheses-and-root-cause.md#can-symptom-drift-be-explained-by-eventual-drying) |

**Conclusion:** The cola residue on the ZIF connector pads was either not identified during the 2-hour cleaning, or was inadequately removed by surface-level wiping that failed to dissolve the thin conductive film between adjacent 0.5 mm-pitch pins. A targeted IPA cleaning of the connector area — the standard procedure for liquid-damaged FPC connectors — was either not performed or not performed thoroughly enough.

### Evidence that the ZIF connector area was not properly dried

| # | Evidence | Source |
|---|----------|--------|
| 1 | **The drying phase is the most damaging period.** As liquid evaporates from the connector area, the acid concentration increases 100–1000×. A droplet that was 0.06% H₃PO₄ by weight becomes an aggressive etchant as water leaves. The corrosion rate **increases** during drying, not decreases. If the service center rinsed the area but did not force-dry it, the residual liquid underwent this concentration process in place. | [Drying phase](chemical-analysis.md#2-drying-phase-minutes-to-hours--concentration-and-film-formation) |
| 2 | **Capillary retention in the ZIF slot delays drying by hours.** The narrow gap of the ZIF socket (< 0.3 mm) retains liquid for 1–4 hours by capillary forces. Even if external surfaces were wiped dry, liquid trapped inside the ZIF slot would have continued to concentrate and deposit residue on the contact pads. Proper drying requires opening the ZIF latch and allowing the slot to ventilate, or using forced air/vacuum. | [Timing estimates](chemical-analysis.md#5-timing-estimates-for-key-chemical-processes) — "Capillary retention in tight gaps: liquid persists 1–4 h" |
| 3 | **The device was powered on at the service center while still wet.** The service center powered on the MacBook to verify it worked. If the ZIF connector area was not yet fully dry, the keyboard controller's 3.3 V scanning voltage would have been applied across the wet electrolyte bridging adjacent pins. Under bias, electrochemical migration (dendrite growth) nucleates within 30 minutes to 2 hours. | [Timing estimates](chemical-analysis.md#5-timing-estimates-for-key-chemical-processes) — "Dendrite nucleation: 30 min – 2 h under bias" |
| 4 | **Symptom worsening over days indicates ongoing electrochemical activity.** If the connector area had been properly dried (no residual moisture or dissolved residue), the conductive bridge would not have formed or strengthened post-service. The progressive worsening — from single-key (N) to full-column (6/Y/H/N) to three-column bridging (Cx/Cy/Cz) — is the signature of residue that dried in place, concentrating and spreading via capillary and hygroscopic mechanisms. | [Can symptom drift be explained by eventual drying?](hypotheses-and-root-cause.md#can-symptom-drift-be-explained-by-eventual-drying) |
| 5 | **Residue film becomes progressively harder to remove after drying.** The caramel color compounds and melanoidin cross-linking products in the residue harden over 6–12 hours and become increasingly adherent over days. If the area was improperly dried, the window for easy IPA-based cleaning was missed — by the time the device was returned to the owner, the residue had hardened into a film that requires ultrasonic cavitation to remove. | [Dried residue phase](chemical-analysis.md#3-dried-residue-phase-hours-to-days-and-beyond) — "Residue hardening: ~6–12 h onset, progressive over days to weeks" |
| 6 | **The unidirectional bridge pattern indicates liquid that flowed and dried in place.** The asymmetric residue thickness (thick near Cx, thin near Cz) is consistent with cola that pooled at one pin and spread outward by capillary action during drying. This directional deposition pattern would not form if the area had been properly cleaned and dried — it is the fingerprint of undisturbed, naturally dried liquid. | [Does the "thickness gradient" create a diode? No.](testing.md#does-the-thickness-gradient-create-a-diode-no) |
| 7 | **Glitch drift tracks the drying-phase chemical timeline.** The observed symptom evolution — (a) initial single-key fault (N), (b) expansion to full column (6/Y/H/N), (c) ghost keypresses appearing, (d) ghost keypresses disappearing, (e) three-column bridging stabilising, (f) partial improvement during 2-day rest — maps directly onto the drying-phase stages: acid concentration → capillary migration spreading the bridge → wet electrolyte causing intermittent floating shorts (ghosts) → residue drying into stable film (ghosts stop, fixed bridge remains) → hygroscopic film slowly losing moisture during rest (bridge resistance increases, partial improvement). Each stage of the glitch drift corresponds to a specific phase of improper drying. If the area had been force-dried, the contamination would have been arrested at whatever state existed at the moment of drying — not allowed to progress through the full multi-day chemical timeline. | [Observed Symptoms](../README.md#observed-symptoms), [Timing estimates](chemical-analysis.md#5-timing-estimates-for-key-chemical-processes), [Partial improvement observed after 2-day rest period](hypotheses-and-root-cause.md#partial-improvement-observed-after-2-day-rest-period) |

**Conclusion:** The ZIF connector area was not properly dried after cleaning. Residual liquid — either original cola or a dilute mixture of cola residue and cleaning solvent — was left in the capillary spaces of the ZIF slot. This liquid underwent the drying-phase concentration process, depositing a thin, concentrated, hygroscopic acid/salt/organic film on the connector pads. The device was then powered on while this process was still ongoing, applying bias voltage that accelerated electrochemical damage.

### How improper cleaning and drying compounded each other

The two failures are synergistic — each one makes the consequences of the other worse:

```
  Timeline of what happened at the service center (reconstructed):

  ┌─────────────────────────────────────────────────────────────────────┐
  │ 1. Device received with cola inside                                │
  │    → Cola present on ZIF connector, FPC traces, under key switches │
  ├─────────────────────────────────────────────────────────────────────┤
  │ 2. Service center cleaning (~2 hours)                              │
  │    → Visible liquid wiped from surfaces                            │
  │    → Compressed air / IPA swabbing on accessible areas             │
  │    → ZIF connector pads NOT specifically cleaned (or insufficiently │
  │      cleaned — thin residue film left on 0.5 mm-pitch pads)       │
  │    → ZIF slot NOT force-dried (capillary-trapped liquid remains)   │
  ├─────────────────────────────────────────────────────────────────────┤
  │ 3. Device powered on to verify function                            │
  │    → 3.3 V scanning voltage applied to still-wet connector area   │
  │    → Electrochemical migration begins (Cu²⁺ ion transport)        │
  │    → Device shows key N malfunction → powered off again            │
  ├─────────────────────────────────────────────────────────────────────┤
  │ 4. Device returned to owner                                        │
  │    → ZIF connector area still drying (capillary retention: 1–4 h) │
  │    → Acid concentration peaking as liquid evaporates               │
  │    → Residue film depositing on pads (onset ~15–45 min)           │
  ├─────────────────────────────────────────────────────────────────────┤
  │ 5. Following days — progressive worsening                          │
  │    → Single-key fault (N) expands to full column (6/Y/H/N)        │
  │    → Ghost keypresses appear as residue bridges stabilise          │
  │    → Eventually: three adjacent columns bridged (Cx/Cy/Cz)        │
  │    → Residue hardens, becomes difficult to remove                  │
  └─────────────────────────────────────────────────────────────────────┘
```

The compounding effect is:

1. **Incomplete cleaning left cola residue on the connector pads.** Even a small amount of residue is enough to seed the drying-phase concentration process.
2. **Incomplete drying allowed the residual liquid to undergo acid concentration in place.** A droplet of dilute cola that would have been easily wiped away instead became a concentrated, aggressive etchant as it dried inside the ZIF slot.
3. **Powering on the device before full drying accelerated the damage.** The scanning voltage drove electrochemical migration (Cu²⁺ ion transport between adjacent pads), potentially nucleating dendrites that would not have formed without bias.
4. **The residue hardened before the owner could intervene.** By the time the laptop was returned and symptoms were observed, the residue film had cross-linked and become adherent (onset ~6–12 hours), making simple cleaning insufficient.

### What proper cleaning and drying would have looked like

For comparison, the correct procedure for a cola-contaminated ZIF connector area is:

1. **Open the ZIF latch** and remove the FPC ribbon cable.
2. **Flush the FPC contact pads and ZIF socket** with ≥99% IPA, using a fine brush or lint-free swab. Multiple passes — dissolve, wipe, dissolve again — to ensure all residue is removed from the 0.5 mm-pitch gaps between pads.
3. **Inspect under magnification** (10× minimum) for any remaining amber/brown residue film between adjacent pads. Cola residue is typically visible as a thin discoloration.
4. **Force-dry the connector area** — compressed air directed into the ZIF slot, or a brief low-heat pass, to ensure no liquid remains trapped by capillary forces.
5. **Do NOT power on the device** until the connector area is confirmed dry and clean under magnification.
6. **Reassemble and test** only after steps 1–5 are complete.

This procedure takes 10–15 minutes and requires no specialised equipment beyond IPA, a lint-free swab, and a magnifying loupe. It directly addresses both failure modes identified above.
