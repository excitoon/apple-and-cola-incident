# Chemical Analysis

← [Back to overview](../README.md)

## Chemical Analysis of Coca-Cola Zero Residue

The spill chemistry matters here because **Coca-Cola Zero is still chemically damaging to electronic circuits even without sugar**. A typical Coca-Cola Zero / Coke Zero Sugar formulation contains:

| Ingredient | Chemical Formula | Percentage (Estimated/Approx.) |
|---|---|---|
| Carbonated Water | H₂O + CO₂ | ~90% |
| Carbon Dioxide | CO₂ | ~0.5–1% (volume dependent) |
| Caramel Color | Complex polymers (E150d) | ~0.1–0.2% |
| Phosphoric Acid | H₃PO₄ | ~0.05–0.07% |
| Aspartame | C₁₄H₁₈N₂O₅ | ~0.024% (87 mg per 12 oz) |
| Acesulfame Potassium | C₄H₄KNO₄S | ~0.013% (47 mg per 12 oz) |
| Caffeine | C₈H₁₀N₄O₂ | ~0.009% (34 mg per 12 oz) |
| Sodium Citrate | Na₃C₆H₅O₇ | < 0.05% |
| Natural Flavors | Proprietary mixture | < 0.05% |

*Percentages are approximate, calculated by mass assuming ~355 mL (12 oz) serving at ~1 g/mL density.*

The key damaging components from an electronics perspective are:

- **Phosphoric acid (H₃PO₄)** — lowers pH to ~2.5–3.2, drives copper/tin corrosion.
- **Potassium/sodium salts** (acesulfame K, sodium citrate, potassium benzoate/citrate depending on market) — create an ionic electrolyte that increases conductivity.
- **Artificial sweeteners** (aspartame, acesulfame K) — not the main conductor, but part of the organic residue left behind after drying.
- **Caramel color (E150d) / flavor residues** — sticky organics that help the dried film adhere to pads, connector fingers, and membrane surfaces.
- **Caffeine** — mildly hygroscopic, contributes to residue film.

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

### Detailed Chemical Process: Coca-Cola Zero Exposure on Film, PCBs, and Traces

This section describes the step-by-step chemical processes that occur when Coca-Cola Zero contacts the materials found in a MacBook keyboard assembly — polyimide flex film, copper traces, solder joints, nickel/gold plating, and FR-4 or flex-PCB substrates. The reactions are grouped by material and by time phase.

See the accompanying diagrams for visual illustrations:
- [Chemical attack cross-section](../diagrams/chemical-attack-cross-section.svg) — FPC layer stack with attack zones
- [Chemical timeline — three phases](../diagrams/chemical-timeline-phases.svg) — reaction summary and corrosion rate graph
- [Galvanic corrosion cell](../diagrams/galvanic-corrosion-cell.svg) — electrochemical cell at bimetallic junctions
- [Dendrite / electrochemical migration](../diagrams/dendrite-electrochemical-migration.svg) — how permanent trace bridges form
- [Connector chemical vulnerability](../diagrams/connector-chemical-vulnerability.svg) — why the ZIF area is worst-case

#### 1. Immediate contact (seconds to minutes) — the wet phase

**On polyimide (Kapton-type) film (FPC base and coverlay):**

Polyimide (chemical formula approximation: repeating unit of PMDA-ODA, poly(4,4′-oxydiphenylene–pyromellitimide)) is one of the most chemically resistant polymer films used in electronics. Under normal Coca-Cola Zero exposure:

- The **phosphoric acid (H₃PO₄, ~0.05–0.07% w/v, pH ≈ 2.5–3.2)** is far too dilute and weak to hydrolyse the imide rings of the polyimide backbone at room temperature. Polyimide hydrolysis requires concentrated strong bases (e.g. KOH at elevated temperature) or concentrated sulfuric acid — conditions nowhere near what cola provides.
- The polyimide film therefore acts as an **inert, impermeable barrier** during the wet phase. Cola sitting on top of intact coverlay does not penetrate or degrade the polymer in any meaningful way.
- **However**, any mechanical defect in the coverlay — a scratch, a laser-cut edge, a via opening, or the intentionally exposed pads at connector fingers and dome-switch contacts — allows cola direct access to the copper beneath.

**On exposed copper traces and pads:**

Where copper is not protected by coverlay (ZIF connector fingers, dome-switch contact lands, vias, FPC edge pads), the following reactions begin immediately:

1. **Dissolution of the native oxide layer.** Copper in air always carries a thin Cu₂O film (~2–5 nm). Phosphoric acid dissolves this oxide directly:

   ```
   Cu₂O + 2 H₃PO₄ → 2 CuH₂PO₄ + H₂O     (Cu⁺ dihydrogen phosphate)
   ```

   In the presence of dissolved oxygen, the Cu⁺ product is quickly oxidised to Cu²⁺, so the net practical result is loss of the passivation layer within seconds, exposing bare metallic copper to further attack.

2. **Acidic corrosion of copper.** With dissolved oxygen present in the cola (especially immediately after pouring, when CO₂ is still outgassing and entraining air), copper oxidises and dissolves:

   ```
   2 Cu + O₂ + 4 H₃PO₄ → 2 Cu(H₂PO₄)₂ + 2 H₂O
   ```

   The copper(II) phosphate product is soluble and is carried away by the liquid, thinning the trace. The reaction rate depends on temperature, oxygen concentration, and acid strength — at room temperature with dilute phosphoric acid, the rate is slow but measurable on the timescale of minutes to hours.

3. **Galvanic corrosion at bimetallic junctions.** Where copper meets a different metal — e.g. tin (solder), nickel (barrier layer), gold (contact plating), or the spring-contact alloy inside the ZIF socket — a **galvanic cell** forms with the cola acting as electrolyte:

   ```
   Anode (less noble, e.g. tin):  Sn → Sn²⁺ + 2e⁻
   Cathode (more noble, e.g. gold or copper):  O₂ + 2H₂O + 4e⁻ → 4OH⁻
   ```

   The galvanic potential difference accelerates dissolution of the less noble metal. In a typical FPC, tin or lead-free solder (SAC305: Sn-3.0Ag-0.5Cu) at a solder joint is anodic relative to both copper traces and gold-plated pads, so **solder joints corrode preferentially** when bridged by cola electrolyte.

4. **Ionic conduction path established.** The dissolved CO₂ (forming carbonic acid, H₂CO₃), phosphoric acid, potassium citrate/benzoate salts, and the freshly dissolved metal ions together create a **conductive electrolyte film** across the surface. Even a thin liquid bridge (~10–100 μm) between two adjacent traces can carry enough leakage current (microamps to milliamps) to register as a false keypress on the keyboard controller's sense lines.

**On the FR-4 or flex-substrate (between copper layers):**

The FPC substrate in a MacBook keyboard is typically polyimide-based flex, not rigid FR-4. But the same principles apply to both:

- The substrate material itself is not significantly attacked by dilute phosphoric acid at room temperature.
- However, if the cola penetrates to the **adhesive layer** between the copper foil and the polyimide base (e.g., through an edge, crack, or via), it can undermine adhesion over time via osmotic blistering: water molecules diffuse through the adhesive, and dissolved ions create an osmotic gradient that draws more water in, eventually delaminating the copper from the substrate.

**On nickel/gold (ENIG) plating:**

ZIF connector pads and some switch contacts use **ENIG (Electroless Nickel Immersion Gold)**: a ~3–5 μm nickel layer topped by ~0.05–0.1 μm gold.

- Gold is essentially inert to phosphoric acid at these concentrations. The gold layer itself is not attacked.
- However, the gold layer on ENIG is extremely thin and porous at the microscopic level. Cola can seep through **pinholes** in the gold to reach the nickel underneath.
- Nickel dissolves in the acidic electrolyte via oxygen-depolarized corrosion (nickel is not reactive enough to displace hydrogen from dilute phosphoric acid at room temperature):

  ```
  2 Ni + O₂ + 4 H₃PO₄ → 2 Ni(H₂PO₄)₂ + 2 H₂O
  ```

  This undermines the gold from below, a process known as **"black pad" corrosion** in electronics manufacturing (though the classic black-pad failure involves hypophosphite-rich nickel, the same undercut mechanism applies here at a slower rate).

**On solder (SAC305 / lead-free):**

Modern MacBook boards use lead-free solder, primarily SAC305 (96.5% Sn, 3.0% Ag, 0.5% Cu). Cola exposure:

- Tin, being the majority component and less noble than copper or gold, is the primary corrosion target. Like nickel, tin corrosion in dilute cola proceeds via dissolved oxygen rather than direct hydrogen displacement:

  ```
  2 Sn + O₂ + 4 H₃PO₄ → 2 Sn(H₂PO₄)₂ + 2 H₂O
  ```

- The silver and copper in the alloy are more resistant but can dissolve slowly at grain boundaries, weakening the solder joint mechanically.
- In the presence of chloride ions (trace amounts from caramel coloring additives), tin can also form tin chloride complexes that are more soluble, accelerating attack.

#### 2. Drying phase (minutes to hours) — concentration and film formation

As the bulk liquid evaporates:

1. **The acid concentrates.** A droplet that was 0.06% H₃PO₄ by weight becomes orders of magnitude more concentrated as water leaves. The shrinking liquid puddle becomes a progressively more aggressive etchant — the corrosion rate actually **increases** during the drying phase, not decreases.

2. **Residue film forms.** The non-volatile components — phosphoric acid, potassium salts, artificial sweeteners (aspartame, acesulfame K), caramel color compounds (a complex mixture of high-molecular-weight melanoidins), and citric acid (where present) — deposit as a **thin, sticky, hygroscopic film** on the surface. This film is typically 1–10 μm thick and covers the entire area that was wet.

3. **Capillary retention in tight spaces.** Inside the scissor-switch mechanism (clearances < 0.3 mm) and under the FPC at dome-switch contacts, liquid is retained much longer by capillary forces. These regions dry last and experience the most concentrated acid attack.

4. **Dendrite nucleation begins.** At the boundary of the drying front, where two adjacent traces are bridged by a thinning electrolyte film under the influence of the keyboard controller's scanning voltage (~3.3 V), **electrochemical migration** can nucleate metallic dendrites:

   ```
   At the anode trace:    Cu → Cu²⁺ + 2e⁻  (copper dissolves)
   At the cathode trace:  Cu²⁺ + 2e⁻ → Cu  (copper plates out)
   ```

   Over time, the plated copper grows from cathode toward anode as a branching dendrite that can permanently short adjacent traces even after the electrolyte dries completely.

#### 3. Dried residue phase (hours to days and beyond)

Once visibly dry, the surface appears clean but is not:

1. **Hygroscopic film absorbs ambient moisture.** Phosphoric acid is strongly hygroscopic (it is used industrially as a desiccant). The dried residue film can absorb enough water vapor from ambient air (especially at >40% RH) to become **conductive again** without any new liquid being added. This explains symptoms that appear or disappear with changes in room humidity or device temperature (which affects local RH at the surface).

2. **Continued slow corrosion under the film.** The concentrated acid film — now essentially a paste — maintains an electrochemical environment on the metal surface. Copper continues to corrode at a slow but nonzero rate:

   ```
   2 Cu + O₂ + 4 H₃PO₄ (concentrated) → 2 Cu(H₂PO₄)₂ + 2 H₂O
   ```

   Over days, this can thin a trace enough to increase its resistance or open it entirely. It can also widen the corroded zone laterally, extending the bridge between adjacent traces.

3. **Organic residue darkens and hardens.** The caramel color compounds and Maillard-reaction products (melanoidins) in the residue undergo slow oxidation and cross-linking. The film becomes progressively **darker, harder, and more adherent** — making it increasingly difficult to remove with mild solvents or wipes. This is why ultrasonic cleaning (with an appropriate solvent) is required: the mechanical agitation of cavitation bubbles is needed to undercut and lift the film from the metal surface.

4. **Copper patina formation.** In the long term, the corroded copper forms a green-blue patina of mixed copper phosphate and copper carbonate hydroxide:

   ```
   Cu(H₂PO₄)₂ + Cu(OH)₂ → Cu₃(PO₄)₂ · Cu(OH)₂  (approximate)
   ```

   This patina is visible under magnification as a green discoloration around exposed pads. It is **non-conductive** in itself, but it is porous and traps moisture, so it can paradoxically maintain a leakage path even though the corrosion product is nominally insulating.

#### 4. Summary of material-specific vulnerability

| Material | Location on keyboard FPC | Chemical vulnerability | Rate of attack | Reversible by cleaning? |
|---|---|---|---|---|
| **Polyimide film** | FPC base, coverlay | Essentially immune to dilute H₃PO₄ at RT | Negligible | N/A — not attacked |
| **Copper traces (buried)** | Under coverlay | Protected unless coverlay is breached | Negligible if intact | N/A — not reached |
| **Copper traces (exposed)** | Dome-switch contacts, via walls | Dissolves in acid; galvanic acceleration at junctions | Moderate (μm/day range) | Yes, if corrosion hasn't severed trace |
| **Copper pads (ZIF fingers)** | Connector edge | Same as exposed copper, plus mechanical wear | Moderate | Yes |
| **ENIG plating (Ni/Au)** | Connector pads, switch contacts | Gold intact; nickel undermined through pinholes | Slow | Yes, but gold may be compromised |
| **SAC305 solder** | FPC-to-connector joints | Tin dissolves preferentially (anodic vs Cu/Au) | Moderate to fast | Partially — weakened joints may need reflow |
| **Adhesive layers** | Copper-to-polyimide bond | Osmotic blistering if electrolyte reaches interface | Slow (days to weeks) | No — delamination is permanent |
| **Rubber dome (silicone)** | Under each key switch | Silicone is chemically inert to cola | Negligible | N/A |
| **Scissor arms (POM/nylon)** | Key switch mechanism | Resistant to dilute acid | Negligible | N/A |

#### 5. Timing estimates for key chemical processes

The following table summarizes approximate timescales for each chemical process after a Coca-Cola Zero spill on an FPC keyboard assembly at room temperature (~22 °C, ~50% RH). All times assume a typical spill volume (a few mL reaching the keyboard internals) and no cleaning intervention.

| Process | Onset | Duration / Peak | Notes |
|---|---|---|---|
| **Cu₂O passivation stripping** | 0–5 s | Complete within 30–60 s | Native oxide is only ~2–5 nm; dissolves on contact with acid |
| **Ionic conduction path formation** | Immediate | Persists while liquid is present | Cola is conductive as-poured (~1–5 mS/cm); leakage current begins instantly |
| **Oxygen-depolarized Cu dissolution** | ~30 s | Continuous; rate ~0.1–1 μm/h in dilute H₃PO₄ | Accelerates as acid concentrates during drying |
| **Galvanic corrosion at bimetallic junctions** | ~30 s | Continuous while electrolyte bridges junction | Rate depends on potential difference (ΔV ~0.3–0.8 V for Sn/Au or Cu/Au pairs) |
| **ENIG pinhole undermining (Ni attack)** | ~1–5 min | Hours to days under residue film | Slow because acid must seep through gold pinholes first |
| **SAC305 solder dissolution (Sn attack)** | ~1–2 min | Continuous; rate ~0.5–2 μm/h in dilute acid | Sn is anodic to Cu/Au — dissolves preferentially |
| **Bulk liquid evaporation** | Immediate | Most surface liquid gone in 10–30 min | Depends on volume, airflow, temperature |
| **Acid concentration peak** | ~10–30 min | Lasts until residue reaches hygroscopic equilibrium | H₃PO₄ concentration increases 100–1000× as water evaporates |
| **Residue film deposition** | ~15–45 min (initial film visible) | Fully deposited when visibly dry (~1–2 h) | Film thickness ~1–10 μm; contains all non-volatile components |
| **Capillary retention in tight gaps** | Immediate | Liquid persists 1–4 h in gaps < 0.3 mm | Scissor-switch clearances and ZIF slot dry last |
| **Dendrite nucleation (electrochemical migration)** | ~30 min – 2 h | Can grow to bridging length in 2–24 h under bias | Requires bias voltage (~3.3 V) and thinning electrolyte |
| **Hygroscopic moisture reabsorption** | After visual drying (~1–2 h) | Cyclical; varies with RH | H₃PO₄ residue absorbs moisture at >30–40% RH |
| **Residue hardening (melanoidin cross-linking)** | ~6–12 h | Progressive over days to weeks | Film becomes increasingly adherent; mild solvents fail after ~24 h |
| **Copper patina formation (green discoloration)** | ~24–72 h | Continues indefinitely | Cu₃(PO₄)₂ · Cu(OH)₂ buildup visible under magnification |
| **Adhesive osmotic blistering** | ~24–48 h | Weeks (slow diffusion process) | Permanent delamination if electrolyte reaches adhesive interface |

**Key timing insight:** The most dangerous period is not the initial wet phase but the **drying phase (10 min – 2 h)**, when acid concentration rises sharply and the corrosion rate peaks. A spill that is wiped up within the first 5 minutes causes far less damage than one left to dry naturally. Once the residue film has set (~2+ hours), only professional cleaning (ultrasonic + appropriate solvent) can reliably remove it.

#### 6. Why the FPC connector area is the most chemically critical site

Combining the above, the ZIF connector area is the worst-case scenario for cola damage because it concentrates **all** the vulnerabilities in one place:

- **Exposed copper pads** (no coverlay — must make electrical contact).
- **ENIG plating with pinholes** (allows acid to reach nickel sublayer).
- **Tight pitch** (~0.5 mm pad-to-pad) — easy for a thin electrolyte film or dendrite to bridge.
- **Bimetallic junction** (gold pad vs. phosphor-bronze spring contact in the ZIF socket) — galvanic corrosion is maximized.
- **Capillary trap** — the narrow slot of the ZIF socket retains liquid and dries last, receiving the most concentrated acid dose.
- **Bias voltage present during operation** (3.3 V scanning voltage) — drives electrochemical migration and dendrite growth.

This is consistent with the observed fault pattern: a single contamination site at or near the connector bridging three physically adjacent column traces (C7, Ca, Cb), producing the characteristic "correct key plus two extra keys" output on every key in the affected column.
