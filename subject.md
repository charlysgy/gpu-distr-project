---
marp: true
theme: academic
math: mathjax
paginate: true
---

<!-- _class: lead -->

# AXIS 6: Power & Cooling

Charly Saugey  
Paul Haardt  
Corentin Colmel  

Image Major / Epita  
Distributed GPU Systems 2026

---

<!-- header: "AXIS 6: Power & Cooling" -->

## Mission
- Explain the power and cooling challenges of modern AI infrastructures  
- Compare cooling technologies and analyze datacenter design trade-offs

---

## Part A: Evolution of AI System Power

---

<!-- header: "Part A: Evolution of AI System Power" -->

### GPU Power Evolution

**Key questions**
- How has GPU power consumption evolved?
- What is driving the increase in power?
- Is there a maximum ceiling to GPU power?
- How does power scale with performance?

---

<style scoped>
  section {
    font-size: 2.0rem;
    padding: 0;
    padding-top: 100px;
  }
</style>
| GPU | Year | TDP (W) | FP16 TFLOPs | W/TFLOP | Process Node |
|---|---:|---:|---:|---:|---|
| V100 | 2017 | 300 [🔗](https://images.nvidia.com/content/technologies/volta/pdf/volta-v100-datasheet-update-us-1165301-r5.pdf#page=1) | 125 [🔗](https://images.nvidia.com/content/technologies/volta/pdf/volta-v100-datasheet-update-us-1165301-r5.pdf#page=1) | 2.40 [🔗](https://images.nvidia.com/content/technologies/volta/pdf/volta-v100-datasheet-update-us-1165301-r5.pdf#page=1) | TSMC 12nm FFN [🔗](https://images.nvidia.com/content/volta-architecture/pdf/volta-architecture-whitepaper.pdf) |
| A100 | 2020 | 400 [🔗](https://www.nvidia.com/en-us/data-center/a100/) | 624 [🔗](https://www.nvidia.com/en-us/data-center/a100/) | 0.64 [🔗](https://www.nvidia.com/en-us/data-center/a100/) | TSMC 7nm [🔗](https://images.nvidia.com/aem-dam/en-zz/Solutions/data-center/nvidia-ampere-architecture-whitepaper.pdf) |
| H100 SXM | 2022 | 700 [🔗](https://www.nvidia.com/en-us/data-center/h100/) | 1979 [🔗](https://www.nvidia.com/en-us/data-center/h100/) | 0.35 [🔗](https://www.nvidia.com/en-us/data-center/h100/) | TSMC 4N [🔗](https://developer.nvidia.com/blog/nvidia-hopper-architecture-in-depth/) |
| H200 SXM | 2024 | 700 [🔗](https://www.nvidia.com/en-us/data-center/h200/) | 1979 [🔗](https://www.nvidia.com/en-us/data-center/h200/) | 0.35 [🔗](https://www.nvidia.com/en-us/data-center/h200/) | TSMC 4N [🔗](https://developer.nvidia.com/blog/nvidia-hopper-architecture-in-depth/) |
| B200 SXM | 2024 | 1000 [🔗](https://lenovopress.lenovo.com/lp2226.pdf) | 4500 [🔗](https://lenovopress.lenovo.com/lp2226.pdf) | 0.22 [🔗](https://lenovopress.lenovo.com/lp2226.pdf) | TSMC 4NP [🔗](https://www.nvidia.com/en-us/data-center/technologies/blackwell-architecture/) |
| B300 | 2025 | 1100 [🔗](https://lenovopress.lenovo.com/lp2264.pdf#page=16) | 4500 [🔗](https://lenovopress.lenovo.com/lp2264.pdf#page=16) | 0.24 [🔗](https://lenovopress.lenovo.com/lp2264.pdf#page=16) | TSMC 4NP [🔗](https://www.nvidia.com/en-us/data-center/technologies/blackwell-architecture/) |
| R100 (Rubin) | 2026 | _~2300_ <sup><a href="https://rcrtech.com/semiconductor-news/nvidia-bumps-vera-rubin-specs/">🔗</a></sup> | _~4000_ <sup><a href="https://www.glennklockwood.com/garden/processors/R200">🔗</a></sup> | _~0.58_ <sup><a href="https://www.glennklockwood.com/garden/processors/R200">🔗</a></sup> | _TSMC N3_ <sup><a href="https://www.nextplatform.com/2026/01/05/nvidias-vera-rubin-platform-obsoletes-current-ai-iron-six-months-ahead-of-launch/">🔗</a></sup> |


<!-- Aucune info publique n'est dispo pour la R100 Rubin -->

---
<style scoped>
  section {
    font-size: 2.1rem;
    padding: 0;
    padding-top: 100px;
  }
</style>
**System-level power (full node)**

| System | GPUs | GPU Power | Total System Power | Year |
|---|---|---:|---:|---:|
| DGX-1 | 8× V100 | 2.4 kW [🔗](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/dgx-1/dgx-1-rhel-datasheet-nvidia-us-808336-r3-web.pdf) | ~3.5 kW [🔗](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/dgx-1/dgx-1-rhel-datasheet-nvidia-us-808336-r3-web.pdf) | 2017 [🔗](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/dgx-1/dgx-1-rhel-datasheet-nvidia-us-808336-r3-web.pdf) |
| DGX A100 | 8× A100 | 3.2 kW [🔗](https://docs.nvidia.com/dgx/dgxa100-user-guide/introduction-to-dgxa100.html#power-specifications) | ~6.5 kW [🔗](https://docs.nvidia.com/dgx/dgxa100-user-guide/introduction-to-dgxa100.html#power-specifications) | 2020 [🔗](https://docs.nvidia.com/dgx/dgxa100-user-guide/introduction-to-dgxa100.html#power-specifications) |
| DGX H100 | 8× H100 SXM | 5.6 kW [🔗](https://www.amax.com/content/files/2024/02/nvidia-dgx-h100-datasheet-partner-us-web3-1.pdf) | ~10.2 kW [🔗](https://www.amax.com/content/files/2024/02/nvidia-dgx-h100-datasheet-partner-us-web3-1.pdf) | 2022 [🔗](https://www.amax.com/content/files/2024/02/nvidia-dgx-h100-datasheet-partner-us-web3-1.pdf) |
| DGX B200 | 8× B200 SXM | 8.0 kW [🔗](https://docs.nvidia.com/dgx/dgxb200-user-guide/dgxb200-user-guide.pdf#page=11) | ~14.3 kW [🔗](https://docs.nvidia.com/dgx/dgxb200-user-guide/dgxb200-user-guide.pdf#page=11) | 2024 [🔗](https://docs.nvidia.com/dgx/dgxb200-user-guide/dgxb200-user-guide.pdf#page=11) |
| GB200 NVL72 | 72× B200 | 72.0 kW [🔗](https://flopper.io/system/nvidia-gb200-nvl72) | ~120 kW [🔗](https://flopper.io/system/nvidia-gb200-nvl72) | 2024 [🔗](https://flopper.io/system/nvidia-gb200-nvl72) |

---
<style scoped>
  section {
    font-size: 2.2rem;
    padding: 0;
    padding-top: 100px;
  }
</style>
**Evolution of rack power density**

| Era | Typical Rack Power | kW/rack | Cooling Method |
|---|---:|---:|---|
| Traditional IT (2010) | 5–10 kW/rack [🔗](https://www.nlyte.com/blog/data-center-rack-density/) | ~8 [🔗](https://www.nlyte.com/blog/data-center-rack-density/) | Air |
| GPU compute (2018) | 15–25 kW/rack [🔗](https://www.nlyte.com/blog/data-center-rack-density/) | ~20 [🔗](https://www.nlyte.com/blog/data-center-rack-density/) | Air |
| AI training (2022) | 30–50 kW/rack [🔗](https://www.nlyte.com/blog/data-center-rack-density/) | ~40 [🔗](https://www.nlyte.com/blog/data-center-rack-density/) | Air / DLC |
| AI training (2024) | 60–120 kW/rack [🔗](https://uptimeinstitute.com/blog/electrical-considerations-for-large-ai-compute) | ~80 [🔗](https://uptimeinstitute.com/blog/electrical-considerations-for-large-ai-compute) | DLC required |
| AI training (2026) | 150–300 kW/rack [🔗](https://uptimeinstitute.com/blog/electrical-considerations-for-large-ai-compute) | ~200 [🔗](https://blogs.nvidia.com/blog/800v-hvdc-power-datacenters/) | DLC / Immersion |

---

![w:1050 center](./gpu-consumtion-evolution.webp)

---

<!-- header: "AXIS 6: Power & Cooling" -->
## Part B: GPU Power Delivery

---

<!-- header: "Part B: GPU Power Delivery" -->
### How does GPU power delivery work?

**Key questions**
- From grid to chip: what are the conversion stages?
- Where do efficiency losses occur?
- How is power distributed inside a GPU?

![center](power-delivery.png)

---

<style scoped>
  section {
    font-size: 2.0rem;
    padding: 0;
    padding-top: 100px;
  }
</style>

| Stage | Input | Output | Typical Efficiency | Loss |
|---|---|---|---:|---:|
| Utility transformer | HV AC (10–30 kV) | LV AC (400–480 V) | 98–99% <sup><a href="https://www.maddox.com/resources/articles/transformer-doe-efficiency-standards#transformer-efficiency-pre-2010-regulation">🔗</a></sup> | 1–2% <sup><a href="https://www.maddox.com/resources/articles/transformer-doe-efficiency-standards#transformer-efficiency-pre-2010-regulation">🔗</a></sup> |
| UPS | AC | AC (conditioned) | 94–97% <sup><a href="https://www.eaton.com/content/dam/eaton/markets/healthcare/knowledge-center/white-paper/is-an-energy-wasting-data-center-draining-your-bottom-line.pdf#page=2">🔗</a></sup> | 3–6% <sup><a href="https://www.eaton.com/content/dam/eaton/markets/healthcare/knowledge-center/white-paper/is-an-energy-wasting-data-center-draining-your-bottom-line.pdf#page=2">🔗</a></sup> |
| PDU | AC | AC | 98–99% <sup><a href="https://www.eaton.com/content/dam/eaton/markets/healthcare/knowledge-center/white-paper/is-an-energy-wasting-data-center-draining-your-bottom-line.pdf#page=2">🔗</a></sup> | 1–2% <sup><a href="https://www.eaton.com/content/dam/eaton/markets/healthcare/knowledge-center/white-paper/is-an-energy-wasting-data-center-draining-your-bottom-line.pdf#page=2">🔗</a></sup> |
| PSU (AC-DC) | AC (200–240 V) | DC (12 V / 48 V) | 92–96% <sup><a href="https://www.arrow.com/en/research-and-events/articles/transformerless-power-supplies-and-data-centers">🔗</a></sup> | 4–8% <sup><a href="https://www.arrow.com/en/research-and-events/articles/transformerless-power-supplies-and-data-centers">🔗</a></sup> |
| VRM (DC-DC) | DC (12 V / 48 V) | DC (~0.7–1.1 V) | 85–95% <sup><a href="https://www.analog.com/en/resources/technical-articles/voltage-conversion-at-low-energy-levels.html">🔗</a></sup> | 5–15% <sup><a href="https://www.analog.com/en/resources/technical-articles/voltage-conversion-at-low-energy-levels.html">🔗</a></sup> |

---

### Voltage Regulator Modules (VRMs)

| VRM Aspect | Description | Typical Value |
|---|---|---|
| Input voltage | Voltage supplied to VRM | 12 V or 48 V <sup><a href="https://www.velocitymicro.com/blog/what-are-motherboard-vrms/#motherboard-vrms-a-quick-explainer">🔗</a></sup> <sup><a href="https://www.dell.com/support/kbdoc/en-us/000146132/keeping-pace-with-power-efficiency#other-advantages">🔗</a></sup> |
| Output voltage | Core supply voltage (Vcore) | 0.7–1.1 V <sup><a href="https://forums.developer.nvidia.com/t/limited-clock-for-the-new-rtx3090ti-ubuntu-20-04/235807">🔗</a></sup> |
| Phase count | Parallel VRM phases | 10–20 phases <sup><a href="https://www.velocitymicro.com/blog/what-are-motherboard-vrms/#motherboard-vrms-vs-power-phases">🔗</a></sup> |
| Efficiency | DC-DC conversion efficiency | 85–95% <sup><a href="https://www.analog.com/en/resources/technical-articles/voltage-conversion-at-low-energy-levels.html">🔗</a></sup> |
| Power loss (1kW GPU) | Heat dissipated by VRM | 50–150 W <sup><a href="https://www.analog.com/en/resources/technical-articles/voltage-conversion-at-low-energy-levels.html">🔗</a></sup> |

---

### 12V vs 48V Power Distribution

| Aspect | 12V Distribution | 48V Distribution |
|---|---|---|
| Current for 1kW | ~83 A <sup><a href="https://www.dell.com/support/kbdoc/en-us/000146132/keeping-pace-with-power-efficiency#other-advantages">🔗</a></sup> | ~21 A <sup><a href="https://www.dell.com/support/kbdoc/en-us/000146132/keeping-pace-with-power-efficiency#other-advantages">🔗</a></sup> |
| Cable thickness | Thick (high current) <sup><a href="https://www.dell.com/support/kbdoc/en-us/000146132/keeping-pace-with-power-efficiency#other-advantages">🔗</a></sup> | Thinner <sup><a href="https://www.dell.com/support/kbdoc/en-us/000146132/keeping-pace-with-power-efficiency#other-advantages">🔗</a></sup> |
| I²R losses | High (×16 vs 48V) <sup><a href="https://www.dell.com/support/kbdoc/en-us/000146132/keeping-pace-with-power-efficiency#other-advantages">🔗</a></sup> | Much lower <sup><a href="https://www.dell.com/support/kbdoc/en-us/000146132/keeping-pace-with-power-efficiency#other-advantages">🔗</a></sup> |
| Connector size | Large / many pins <sup><a href="https://www.dell.com/support/kbdoc/en-us/000146132/keeping-pace-with-power-efficiency#other-advantages">🔗</a></sup> | Smaller / fewer pins <sup><a href="https://www.dell.com/support/kbdoc/en-us/000146132/keeping-pace-with-power-efficiency#other-advantages">🔗</a></sup> |
| Industry adoption | Legacy servers <sup><a href="https://www.dell.com/support/kbdoc/en-us/000146132/keeping-pace-with-power-efficiency#other-advantages">🔗</a></sup> | AI & hyperscale <sup><a href="https://www.dell.com/support/kbdoc/en-us/000146132/keeping-pace-with-power-efficiency#other-advantages">🔗</a></sup> |

---

### Connector considerations

| Connector | Max Power | Pins | Issues |
|---|---:|---:|---|
| 8-pin PCIe | 150 W <sup><a href="https://en.wikipedia.org/wiki/PCI_Express#Power">🔗</a></sup> | 8 <sup><a href="https://en.wikipedia.org/wiki/PCI_Express#Power">🔗</a></sup> | Limited power, multiple cables needed <sup><a href="https://en.wikipedia.org/wiki/PCI_Express#Power">🔗</a></sup> |
| 12VHPWR | 600 W <sup><a href="https://www.anandtech.com/show/17530/nvidia-12vhpwr-issues-and-analysis">🔗</a></sup> | 16 (12+4 sense) <sup><a href="https://www.anandtech.com/show/17530/nvidia-12vhpwr-issues-and-analysis">🔗</a></sup> | Overheating, insertion sensitivity <sup><a href="https://www.anandtech.com/show/17530/nvidia-12vhpwr-issues-and-analysis">🔗</a></sup> |
| 12V-2x6 | 600 W <sup><a href="https://en.wikipedia.org/wiki/12VHPWR_connector#12V-2x6">🔗</a></sup> | 16 (12+4 sense) <sup><a href="https://en.wikipedia.org/wiki/12VHPWR_connector#12V-2x6">🔗</a></sup> | Improved safety, still high current <sup><a href="https://en.wikipedia.org/wiki/12VHPWR_connector#12V-2x6">🔗</a></sup> |

---

### 800V DC for AI factories

**Questions**
- Why is NVIDIA pushing 800V DC?
- What efficiency gains are possible?
- How does this change datacenter design?
- What are the safety considerations?
<!-- To reduce current inside ultra-high-power racks and therefore reduce resistive losses.
Fewer conversion stages, fewer loss points, and better power delivery along racks.

High-voltage DC = persistent arcs, higher insulation, stricter disconnection systems, procedures, and trained personnel -->

---

<!-- header: "AXIS 6: Power & Cooling" -->
## Part C: Cooling Technologies

---

<!-- header: "Part C: Cooling Technologies" -->
### Air Cooling

**Questions**
- How does air cooling work?
- What are the design principles of a heat sink?
- What are the limits?
- When does air cooling fail?
- At what TDP does air become impractical?
- What are the acoustic limits?
- How does altitude affect air cooling?
<!-- By convection: the hot component heats surrounding air; with a fan or airflow, hot air is expelled, restoring temperature gradient.
The second law of thermodynamics drives heat flow from hot component to cooler air repeatedly. -->
<!-- Goal: maximize thermal exchange surface between hot component and air -->
<!-- Convective cooling is less efficient than conductive cooling -->
<!-- If ambient air is hot, it may no longer absorb enough heat -->
<!-- ~750 W -->
<!-- ~75 dB -->
<!-- Higher altitude = lower air density = less effective cooling for same airflow volume = higher airflow required -->

---

| Air Cooling Aspect | Typical Value | Limit |
|---|---:|---:|
| Max heat dissipation per GPU | 300–600 W <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=4">🔗</a></sup> | ~800 W <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=4">🔗</a></sup> |
| Max rack density | 20–40 kW <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=3">🔗</a></sup> | ~50 kW <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=3">🔗</a></sup> |
| Airflow per rack | ~160 CFM/kW <sup><a href="https://www.datacenterknowledge.com/archives/2014/10/01/how-much-airflow-does-it-take-to-cool-a-data-center">🔗</a></sup> | >160 CFM/kW <sup><a href="https://www.datacenterknowledge.com/archives/2014/10/01/how-much-airflow-does-it-take-to-cool-a-data-center">🔗</a></sup> |
| Fan power overhead | 2–10% <sup><a href="https://tc090.ashraetcs.org/documents/ASHRAE_TC090_WhitePaper_LiquidCooling.pdf#page=13">🔗</a></sup> | 10–20% <sup><a href="https://tc090.ashraetcs.org/documents/ASHRAE_TC090_WhitePaper_LiquidCooling.pdf#page=13">🔗</a></sup> |

---

### Direct Liquid Cooling (DLC)

**How does DLC work?**
- Cold plate design and attachment
- Manifolds and distribution systems
- Coolant types and flow rates
- Heat rejection (CDU, dry coolers, cooling towers)
<!-- Metal plate in direct contact with GPU, heat transferred to fluid -->
<!-- Piping networks distributing fluid to each cold plate -->
<!-- Water/glycol or dielectric fluids, flow optimized for target ΔT -->
<!-- CDU (Coolant Distribution Units) + external heat exchangers (dry coolers or cooling towers) -->

---

<style scoped>
  section {
    font-size: 1.8rem;
    padding: 0;
    padding-top: 100px;
  }
</style>
| DLC Component | Function | Key Specifications |
|---|---|---|
| Cold plate | Heat transfer from GPU to liquid | Metal base plate (typically copper), microchannels <sup><a href="https://www.opencompute.org/documents/2p-refrigerant-based-dlc-wp-v1-pdf-1#page=17">🔗</a></sup> |
| Manifold | Distribute fluid to cold plates | Row/rack manifolds for supply/return distribution <sup><a href="https://www.opencompute.org/documents/2p-refrigerant-based-dlc-wp-v1-pdf-1#page=11">🔗</a></sup> |
| Quick disconnects | Serviceable maintenance connections | Serviceable quick disconnect fittings between loop and cold plates <sup><a href="https://www.opencompute.org/documents/2p-refrigerant-based-dlc-wp-v1-pdf-1#page=11">🔗</a></sup> |
| CDU (Coolant Distribution Unit) | Pump + heat exchanger + filtration | 60–100 L/min (example CDU spec) <sup><a href="https://www.vertiv.com/4acae4/globalassets/shared/vertiv-coolchip-cdu-70-brochure-sl-71227.pdf#page=4">🔗</a></sup> |
| Facility water loop | Reject heat to building | Heat exchanger to facility water / external heat rejection loop <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=10">🔗</a></sup> |

---

<style scoped>
  section {
    font-size: 1.9rem;
    padding: 0;
    padding-top: 100px;
  }
</style>
| Aspect | Air Cooling | Direct Liquid Cooling |
|---|---|---|
| Max GPU TDP supported | ~800 W <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=4">🔗</a></sup> | 1,000 W+ <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=4">🔗</a></sup> |
| Max rack density | ~40–50 kW <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=3">🔗</a></sup> | ~100–200+ kW <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=3">🔗</a></sup> |
| PUE impact | Higher overhead <sup><a href="https://www.vertiv.com/en-emea/solutions/learn-about/liquid-cooling-options-for-data-centers/">🔗</a></sup> | Lower overhead <sup><a href="https://www.vertiv.com/en-emea/solutions/learn-about/liquid-cooling-options-for-data-centers/">🔗</a></sup> |
| Maintenance complexity | Low <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=9">🔗</a></sup> | Medium/High <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=9">🔗</a></sup> |
| Capital cost | Lower <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=8">🔗</a></sup> | Higher <sup><a href="https://www.vertiv.com/4926c8/globalassets/documents/white-papers/liquid-cooling/deploying-liquid-cooling-in-the-data-center-a-guide-to-high-density-cooling-white-paper.pdf#page=8">🔗</a></sup> |
| Operating cost | Higher fan energy <sup><a href="https://tc090.ashraetcs.org/documents/ASHRAE_TC090_WhitePaper_LiquidCooling.pdf#page=13">🔗</a></sup> | Lower pumping energy (typically) <sup><a href="https://tc090.ashraetcs.org/documents/ASHRAE_TC090_WhitePaper_LiquidCooling.pdf#page=13">🔗</a></sup> |

---

<style scoped>
  section {
    font-size: 1.9rem;
    padding: 0;
    padding-top: 100px;
  }
</style>
| Coolant | Thermal Properties | Cost | Safety | Use Case |
|---|---|---|---|---|
| Water/glycol | Common liquid-cooling working fluid (e.g., PG-25) <sup><a href="https://www.vertiv.com/4acae4/globalassets/shared/vertiv-coolchip-cdu-70-brochure-sl-71227.pdf#page=4">🔗</a></sup> | Low <sup><a href="https://www.vertiv.com/4acae4/globalassets/shared/vertiv-coolchip-cdu-70-brochure-sl-71227.pdf#page=4">🔗</a></sup> | Conductive, leak risk <sup><a href="https://submer.com/immersion-cooling/">🔗</a></sup> | Cold plates, DLC loops <sup><a href="https://www.opencompute.org/documents/2p-refrigerant-based-dlc-wp-v1-pdf-1#page=11">🔗</a></sup> |
| Propylene glycol | Used as glycol mix / antifreeze variant <sup><a href="https://www.vertiv.com/4acae4/globalassets/shared/vertiv-coolchip-cdu-70-brochure-sl-71227.pdf#page=4">🔗</a></sup> | Low–Medium <sup><a href="https://www.vertiv.com/4acae4/globalassets/shared/vertiv-coolchip-cdu-70-brochure-sl-71227.pdf#page=4">🔗</a></sup> | Less toxic than ethylene glycol <sup><a href="https://www.opencompute.org/documents/2p-refrigerant-based-dlc-wp-v1-pdf-1#page=18">🔗</a></sup> | Cold climates, DLC <sup><a href="https://www.vertiv.com/4acae4/globalassets/shared/vertiv-coolchip-cdu-70-brochure-sl-71227.pdf#page=4">🔗</a></sup> |
| Dielectric fluids | Electrically insulating immersion fluids <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> | High <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> | Non-conductive (safer for electronics) <sup><a href="https://submer.com/immersion-cooling/">🔗</a></sup> | Immersion cooling <sup><a href="https://submer.com/immersion-cooling/">🔗</a></sup> |

---

### Immersion Cooling

**Topics**
- Single-phase vs two-phase  
- Tank design and fluid management  
- Heat rejection methods  
- Maintenance considerations

---

<style scoped>
  section {
    font-size: 1.9rem;
    padding: 0;
    padding-top: 100px;
  }
</style>
| Aspect | Single-Phase Immersion | Two-Phase Immersion |
|---|---|---|
| Fluid type | Dielectric fluid (no phase change) <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> | Low-boiling dielectric fluid <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> |
| Operating principle | Liquid absorbs heat, remains liquid <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> | Liquid boils, vapor condenses <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> |
| Max heat flux | High <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> | Very high <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> |
| Fluid cost | Medium <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> | High <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> |
| Complexity | Medium <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> | High <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> |
| Maturity | Commercially mature <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> | Emerging / niche <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> |

---

| Advantage | Challenge |
|---|---|
| Highest heat density | Specialized fluids |
| No fans required | Hardware compatibility |
| Reduced PUE | Higher CAPEX |
| Component longevity | Maintenance procedures |

---

### Rear-Door Heat Exchangers (RDHx)

**Questions**
- What is RDHx?  
- How does it supplement air cooling?  
- What power densities can it support?  
- When is it the right choice?  
<!-- Air-to-liquid heat exchanger mounted at rear of rack that captures exhaust air and removes heat using a water loop -->
<!-- It cools air locally at rack level, reducing room thermal load and improving efficiency of existing air cooling -->
<!-- Roughly 20,000–80,000 W per rack depending on passive or active design -->
<!-- When an air-cooled datacenter must support denser racks without immediate transition to DLC or immersion -->

---

| RDHx Type | Cooling Capacity | Best For |
|---|---:|---|
| Passive RDHx | ~20–35 kW per rack <sup><a href="https://www.vertiv.com/48ea7e/globalassets/shared/vertiv-coolloop-rdhx-data-sheet-mka4l0ukrdhx.pdf#page=2">🔗</a></sup> | Medium-density racks, retrofit <sup><a href="https://www.vertiv.com/48ea7e/globalassets/shared/vertiv-coolloop-rdhx-data-sheet-mka4l0ukrdhx.pdf#page=1">🔗</a></sup> |
| Active RDHx | ~30–80 kW per rack <sup><a href="https://www.vertiv.com/48ea7e/globalassets/shared/vertiv-coolloop-rdhx-data-sheet-mka4l0ukrdhx.pdf#page=2">🔗</a></sup> | High-density GPU racks <sup><a href="https://www.vertiv.com/48ea7e/globalassets/shared/vertiv-coolloop-rdhx-data-sheet-mka4l0ukrdhx.pdf#page=1">🔗</a></sup> |
---

<!-- header: "AXIS 6: Power & Cooling" -->
## Part D: Power Usage Effectiveness (PUE)

---

<!-- header: "Part D: Power Usage Effectiveness (PUE)" -->
### Understanding PUE

**Definition**
- PUE = Power Usage Effectiveness  
- PUE = Total Facility Power / IT Equipment Power  
- 1 PUE = (IT Load + Cooling + Power Distribution + Lighting + Other) / IT Load

---

**Questions**
- What does PUE measure and not measure?
- How does cooling choice affect PUE?
- What are the components of overhead?
<!-- PUE measures datacenter infrastructure efficiency but not application, server, or workload efficiency -->
<!-- Cooling, power distribution losses, UPS, lighting, security, auxiliary systems -->
<!-- Liquid cooling solutions significantly reduce cooling energy and therefore PUE -->

---

| PUE Component | Typical % of Overhead | Reduction Strategies |
|---|---:|---|
| Cooling | ~40% <sup><a href="https://www.iea-4e.org/wp-content/uploads/2025/05/Data-Centre-Energy-Use-Critical-Review-of-Models-and-Results.pdf#page=31">🔗</a></sup> | DLC, free cooling, containment |
| Power distribution losses | ~10–12% <sup><a href="https://www.energystar.gov/products/data_center_equipment/16-more-ways-cut-energy-waste-data-center/reduce-energy-losses-power-distribution-units-pdus">🔗</a></sup> | High-efficiency UPS/PSU |
| Lighting and other | ~3–5% <sup><a href="https://www.datacenterknowledge.com/sustainability/the-true-value-of-data-center-lighting-efficiency">🔗</a></sup> | LED, automation |

---

**PUE benchmarks**

| Datacenter Type | Typical PUE | Best-in-Class PUE |
|---|---:|---:|
| Legacy enterprise | 1.8–2.5 <sup><a href="https://sustainability.atmeta.com/wp-content/uploads/2023/04/Meta-2023-ESG-Report-Data-Index.pdf#page=14">🔗</a></sup><sup><a href="https://www.energystar.gov/sites/default/files/tools/DataCenterFAQs.pdf#page=1">🔗</a></sup> | ~1.6 <sup><a href="https://www.google.com/about/datacenters/efficiency/">🔗</a></sup> |
| Modern enterprise | 1.4–1.6 <sup><a href="https://datacenter.uptimeinstitute.com/rs/711-RIA-145/images/2024.GlobalDataCenterSurvey.Report.pdf#page=7">🔗</a></sup> | ~1.3 <sup><a href="https://datacenter.uptimeinstitute.com/rs/711-RIA-145/images/2024.GlobalDataCenterSurvey.Report.pdf#page=7">🔗</a></sup> |
| Hyperscale (air) | 1.2–1.4 <sup><a href="https://www.nrel.gov/data/data-center-efficiency.html">🔗</a></sup> | ~1.09 <sup><a href="https://www.google.com/about/datacenters/efficiency/">🔗</a></sup> |
| Hyperscale (liquid) | 1.1–1.2 <sup><a href="https://tc090.ashraetcs.org/documents/ASHRAE_TC090_WhitePaper_LiquidCooling.pdf#page=21">🔗</a></sup> | ~1.02–1.03 <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> |
| AI-optimized | 1.1–1.3 <sup><a href="https://news.microsoft.com/en-nz/2022/09/21/hyperscale-cloud-to-support-new-zealands-growing-digital-economy/">🔗</a></sup> | ~1.05–1.1 <sup><a href="https://tc090.ashraetcs.org/documents/ASHRAE_TC090_WhitePaper_LiquidCooling.pdf#page=21">🔗</a></sup> |

---

| Cooling Method | Typical PUE | Why |
|---|---:|---|
| Traditional air (CRAC) | ~1.6 <sup><a href="https://www.google.com/about/datacenters/efficiency/">🔗</a></sup> | Energy-intensive compressors |
| Hot/cold aisle containment | ~1.4 <sup><a href="https://datacenter.uptimeinstitute.com/rs/711-RIA-145/images/2024.GlobalDataCenterSurvey.Report.pdf#page=7">🔗</a></sup> | Optimized airflow |
| Free air cooling | ~1.2 <sup><a href="https://www.nrel.gov/data/data-center-efficiency.html">🔗</a></sup> | Minimal active cooling |
| Direct liquid cooling | ~1.1 <sup><a href="https://tc090.ashraetcs.org/documents/ASHRAE_TC090_WhitePaper_LiquidCooling.pdf#page=21">🔗</a></sup> | Direct heat transfer |
| Immersion cooling | ~1.02–1.03 <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> | Near elimination of HVAC |

---

**Best-in-class examples**

| Company | Facility | PUE | How Achieved |
|---|---|---:|---|
| Google | Hyperscale DC | 1.09 (2024) <sup><a href="https://www.google.com/about/datacenters/efficiency/">🔗</a></sup> | Free cooling + control/optimization |
| Meta | Hyperscale DC | 1.09 (reported) <sup><a href="https://sustainability.atmeta.com/wp-content/uploads/2023/04/Meta-2023-ESG-Report-Data-Index.pdf#page=14">🔗</a></sup> | Efficiency improvements (site-dependent) |
| Microsoft | Hyperscale DC (example) | 1.12 (expected) <sup><a href="https://news.microsoft.com/en-nz/2022/09/21/hyperscale-cloud-to-support-new-zealands-growing-digital-economy/">🔗</a></sup> | Efficiency-focused design (example site) |

---

**Beyond PUE: other efficiency metrics**

| Metric | Definition | Typical Values | Best-in-Class |
|---|---|---|---|
| PUE | Total / IT ratio <sup><a href="https://www.energystar.gov/sites/default/files/tools/DataCenterFAQs.pdf#page=1">🔗</a></sup> | ~1.1–1.6 <sup><a href="https://datacenter.uptimeinstitute.com/rs/711-RIA-145/images/2024.GlobalDataCenterSurvey.Report.pdf#page=7">🔗</a></sup> | ~1.02–1.09 <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup><sup><a href="https://www.google.com/about/datacenters/efficiency/">🔗</a></sup> |
| WUE | Liters water / kWh IT <sup><a href="https://airatwork.com/wp-content/uploads/The-Green-Grid-White-Paper-35-WUE-Usage-Guidelines.pdf">🔗</a></sup> | ~0.2–1.8 <sup><a href="https://www.vertiv.com/en-latam/about/news-and-insights/articles/blog-posts/Keys-to-achieving-water-efficiency-in-data-centers/">🔗</a></sup> | ≤0.2 <sup><a href="https://www.vertiv.com/en-latam/about/news-and-insights/articles/blog-posts/Keys-to-achieving-water-efficiency-in-data-centers/">🔗</a></sup> |
| CUE | kgCO₂ / kWh IT <sup><a href="https://www.netalis.fr/wp-content/uploads/2016/04/Carbon-Usage-Effectiveness-White-Paper_v3.pdf">🔗</a></sup> | Variable <sup><a href="https://www.opencompute.org/documents/dcf-sustainability-metrics-final-r3-docx-pdf">🔗</a></sup> | Near zero (low-carbon energy) <sup><a href="https://www.netalis.fr/wp-content/uploads/2016/04/Carbon-Usage-Effectiveness-White-Paper_v3.pdf">🔗</a></sup> |

<!-- WUE = Water Usage Effectiveness -->
<!-- CUE = Carbon Usage Effectiveness -->

---

<!-- header: "AXIS 6: Power & Cooling" -->
## Part E: Infrastructure Requirements

---

<!-- header: "Part E: Infrastructure Requirements" -->
### Power Density Planning

**Questions**
- How do you plan for increasing density?
- What infrastructure upgrades are needed?
- How do you handle mixed densities?
<!-- Modular power and cooling, oversizing pathways, liquid-ready racks -->
<!-- Higher-capacity busways, transformers, UPS, liquid cooling loops -->
<!-- Zoning: air-cooled rows + liquid-cooled rows -->

---

| Density Tier | kW/Rack | Infrastructure Requirements |
|---|---:|---|
| Low density | 5–10 kW <sup><a href="https://datacenter.uptimeinstitute.com/rs/711-RIA-145/images/2024.GlobalDataCenterSurvey.Report.pdf#page=2">🔗</a></sup> | Air cooling, standard power |
| Medium density | 10–30 kW <sup><a href="https://datacenter.uptimeinstitute.com/rs/711-RIA-145/images/2024.GlobalDataCenterSurvey.Report.pdf#page=2">🔗</a></sup> | Hot/cold aisle, high airflow |
| High density | 30–80 kW <sup><a href="https://www.opencompute.org/documents/acs-door-hx-whitepaper-final-230419-pdf">🔗</a></sup> | RDHx or DLC |
| Ultra-high density | >80 kW <sup><a href="https://www.se.com/ww/en/about-us/newsroom/news/press-releases/schneider-electric-announces-new-solutions-to-address-the-energy-and-sustainability-challenges-spurred-by-ai-674f09f25ad2a1e7eb07b6f3">🔗</a></sup> | DLC or immersion |

---

### Power distribution architecture

| Component | Function | Sizing Consideration |
|---|---|---|
| Utility feed | Power from grid | Peak MW + redundancy |
| Main switchgear | Distribution & protection | Fault current rating |
| UPS systems | Backup & conditioning | MW load, minutes runtime |
| PDUs | Rack-level distribution | kW per rack |
| RPPs | Branch circuits | Density zones |

---

### Cooling capacity planning

| Cooling capacity Unit | Conversion | Context |
|---|---|---|
| 1 ton of cooling | 12,000 BTU/h <sup><a href="https://www.engineeringtoolbox.com/cooling-capacity-d_1850.html">🔗</a></sup> | 3.52 kW <sup><a href="https://www.engineeringtoolbox.com/cooling-capacity-d_1850.html">🔗</a></sup> |
| 1 kW IT load (heat) | ~0.284 ton <sup><a href="https://www.engineeringtoolbox.com/cooling-capacity-d_1850.html">🔗</a></sup> | Rule of thumb (1 kW power ≈ 1 kW heat) |
| 1 MW IT load (heat) | ~284 ton <sup><a href="https://www.engineeringtoolbox.com/cooling-capacity-d_1850.html">🔗</a></sup> | Useful for plant-level sizing |

---

**Water requirements for liquid cooling**

| Cooling Method | Water Usage | m³/h per MW |
|---|---|---:|
| Evaporative (cooling tower) | High <sup><a href="https://www.eesi.org/papers/view/fact-sheet-data-centers-and-servers">🔗</a></sup> | 1.5–2 <sup><a href="https://www.eesi.org/papers/view/fact-sheet-data-centers-and-servers">🔗</a></sup> |
| Dry cooler | Low <sup><a href="https://www.airsysnorthamerica.com/blog/data-center-water-usage-effectiveness-wue/">🔗</a></sup> | 0.1–0.3 <sup><a href="https://www.airsysnorthamerica.com/blog/data-center-water-usage-effectiveness-wue/">🔗</a></sup> |
| DLC (closed loop) | Very low <sup><a href="https://www.thegreengrid.org/en/resources/library-and-tools/155-Water-Usage-Effectiveness-WUE-A-Green-Grid-Data-Center-Sustainability-Metric">🔗</a></sup> | ~0.05 <sup><a href="https://www.thegreengrid.org/en/resources/library-and-tools/155-Water-Usage-Effectiveness-WUE-A-Green-Grid-Data-Center-Sustainability-Metric">🔗</a></sup> |

---

### Backup power systems

| UPS Type | Efficiency | Runtime | Best For |
|---|---:|---:|---|
| Double conversion | 94–97% <sup><a href="https://blog.se.com/datacenter/2015/07/07/a-new-take-on-ups-eco-mode-delivers-efficiency-without-sacrificing-reliability-and-availability/">🔗</a></sup> | 5–15 min | Critical loads |
| Line interactive | 96–98% <sup><a href="https://media.zones.com/images/pdf/White_paper_APC_Technical_Comparison.pdf#page=1">🔗</a></sup> | 5–10 min | Edge DC |
| Rotary UPS | 96–98% <sup><a href="https://www.vertiv.com/globalassets/documents/white-papers/vertiv-the_future-proofed_choice_for_a_shifting_energy_landscape-wp-en-emea_255008_0.pdf">🔗</a></sup> | Seconds | Large DC |
| Battery + flywheel | 95–98% <sup><a href="https://www.activepower.com/wp-content/uploads/2023/11/high-efficiency-ups-systems-for-a-power-hungry-world_compressed.pdf">🔗</a></sup> | Seconds–minutes | High power |

---

**Generator requirements**

| Facility Size | Generator Capacity | Fuel Storage | Startup Time |
|---:|---|---|---|
| 10 MW | 10–12 MW <sup><a href="https://download.schneider-electric.com/files?p_enDocType=White+Paper&p_File_Name=SPD_WP286_EN.pdf">🔗</a></sup> | Hours | <10 s <sup><a href="https://www.cummins.com/sites/default/files/2019-04/FAQ_NFPA%20110%20Time%20to%20Readiness.pdf">🔗</a></sup> |
| 100 MW | 120 MW <sup><a href="https://download.schneider-electric.com/files?p_enDocType=White+Paper&p_File_Name=SPD_WP286_EN.pdf">🔗</a></sup> | Hours | <10 s <sup><a href="https://www.cummins.com/sites/default/files/2019-04/FAQ_NFPA%20110%20Time%20to%20Readiness.pdf">🔗</a></sup> |
| 500 MW | 600 MW <sup><a href="https://download.schneider-electric.com/files?p_enDocType=White+Paper&p_File_Name=SPD_WP286_EN.pdf">🔗</a></sup> | Hours | <15 s <sup><a href="https://www.facilitiesnet.com/powercommunication/article/What-Is-the-Best-Start-time-Delay-For-A-Standby-Generator-In-A-Data-Center--14154">🔗</a></sup> |
| 1 GW | 1.2 GW <sup><a href="https://download.schneider-electric.com/files?p_enDocType=White+Paper&p_File_Name=SPD_WP286_EN.pdf">🔗</a></sup> | Hours | <15 s <sup><a href="https://www.facilitiesnet.com/powercommunication/article/What-Is-the-Best-Start-time-Delay-For-A-Standby-Generator-In-A-Data-Center--14154">🔗</a></sup> |

---

### Grid connection for gigawatt-scale sites

| Scale | Grid Requirements | Typical Lead Time |
|---:|---|---|
| 50 MW | Substation tie-in <sup><a href="https://gridlab.org/wp-content/uploads/2025/03/GridLab-Report-Large-Loads-Interim-Report.pdf">🔗</a></sup> | 1–2 years <sup><a href="https://gridlab.org/wp-content/uploads/2025/03/GridLab-Report-Large-Loads-Interim-Report.pdf">🔗</a></sup> |
| 200 MW | Dedicated substation <sup><a href="https://www.iea.org/reports/energy-and-ai/executive-summary">🔗</a></sup> | 2–3 years <sup><a href="https://www.iea.org/reports/energy-and-ai/executive-summary">🔗</a></sup> |
| 500 MW | Transmission upgrade <sup><a href="https://www.iea.org/reports/energy-and-ai/executive-summary">🔗</a></sup> | 3–5 years <sup><a href="https://www.iea.org/reports/energy-and-ai/executive-summary">🔗</a></sup> |
| 1 GW+ | New transmission lines <sup><a href="https://www.iea.org/reports/energy-and-ai/executive-summary">🔗</a></sup> | 5+ years <sup><a href="https://www.iea.org/reports/energy-and-ai/executive-summary">🔗</a></sup> |

---

<style scoped>
  section {
    font-size: 1.8rem;
    padding: 0;
    padding-top: 100px;
  }
</style>
### Power sourcing strategies

| Strategy | Description | Pros | Cons |
|---|---|---|---|
| Grid connection | Utility power | Simple | Carbon intensity |
| On-site generation | Gas/diesel | Fast backup | Emissions |
| PPA | Long-term renewable contract | Green energy | Price lock |
| Behind-the-meter solar/wind | Local generation | Low carbon | Intermittent |
| Nuclear (SMR) | Small modular reactors | Stable low-carbon | Regulatory |

---

<!-- header: "AXIS 6: Power & Cooling" -->
## Part F: Deep Dive Topics

---

<!-- header: "Part F: Deep Dive Topics" -->
### Chip-Level Power Management

**Dynamic Voltage and Frequency Scaling (DVFS)**
- How does DVFS work?
- What is the power/frequency relationship?
<!-- The processor dynamically adjusts voltage and frequency based on workload and temperature.
Dynamic power approximately follows: P ∝ V² × f -->

---


| Power State | Voltage | Frequency | Power | Use Case |
|---|---:|---:|---:|---|
| Max boost | High <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | High <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | Very high <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | Peak compute |
| Base clock | Nominal <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | Nominal <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | High <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | Sustained |
| Idle | Low <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | Low <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | Low <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | Low utilization |
| Sleep | Very low <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | ~0 <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | Very low <sup><a href="https://en.wikipedia.org/wiki/Dynamic_voltage_scaling#Power">🔗</a></sup> | Inactive |

---

### Thermal throttling behavior

| Threshold | Temperature | Action |
|---|---:|---|
| Target | 83 °C <sup><a href="https://forums.developer.nvidia.com/t/nvidia-smi-gpu-target-temperature-maximum-operating-temperature/229325">🔗</a></sup> | Nominal cooling control <sup><a href="https://forums.developer.nvidia.com/t/nvidia-smi-gpu-target-temperature-maximum-operating-temperature/229325">🔗</a></sup> |
| Max operating | 88 °C <sup><a href="https://forums.developer.nvidia.com/t/nvidia-smi-gpu-target-temperature-maximum-operating-temperature/229325">🔗</a></sup> | Increased cooling / management <sup><a href="https://forums.developer.nvidia.com/t/nvidia-smi-gpu-target-temperature-maximum-operating-temperature/229325">🔗</a></sup> |
| Slowdown | 92 °C <sup><a href="https://forums.developer.nvidia.com/t/nvidia-smi-gpu-target-temperature-maximum-operating-temperature/229325">🔗</a></sup> | Clock throttling begins <sup><a href="https://forums.developer.nvidia.com/t/nvidia-smi-gpu-target-temperature-maximum-operating-temperature/229325">🔗</a></sup> |
| Shutdown | 95 °C <sup><a href="https://forums.developer.nvidia.com/t/nvidia-smi-gpu-target-temperature-maximum-operating-temperature/229325">🔗</a></sup> | Hardware shutdown <sup><a href="https://forums.developer.nvidia.com/t/nvidia-smi-gpu-target-temperature-maximum-operating-temperature/229325">🔗</a></sup> |

---

### Heat sink and cold plate design

| Parameter | Impact | Tradeoff |
|---|---|---|
| Fin density | Heat transfer surface <sup><a href="https://www.electronics-cooling.com/wp-content/uploads/2023/09/Heat-Sink-Design-for-High-Heat-Flux-Applications.pdf#page=2">🔗</a></sup> | Airflow restriction <sup><a href="https://www.electronics-cooling.com/wp-content/uploads/2023/09/Heat-Sink-Design-for-High-Heat-Flux-Applications.pdf#page=2">🔗</a></sup> |
| Base thickness | Heat spreading <sup><a href="https://www.electronics-cooling.com/wp-content/uploads/2023/09/Heat-Sink-Design-for-High-Heat-Flux-Applications.pdf#page=2">🔗</a></sup> | Mass <sup><a href="https://www.electronics-cooling.com/wp-content/uploads/2023/09/Heat-Sink-Design-for-High-Heat-Flux-Applications.pdf#page=2">🔗</a></sup> |
| Heat pipe count | Heat transport <sup><a href="https://www.mersen.com/sites/default/files/files_imported_ep/WP-Characterization-of-a-Heat-Sink-with-Embedded-Heat-Pipe.pdf#page=1">🔗</a></sup> | Cost <sup><a href="https://www.mersen.com/sites/default/files/files_imported_ep/WP-Characterization-of-a-Heat-Sink-with-Embedded-Heat-Pipe.pdf#page=1">🔗</a></sup> |
| Material (Cu vs Al) | Thermal conductivity <sup><a href="https://www.engineeringtoolbox.com/thermal-conductivity-metals-d_858.html">🔗</a></sup> | Weight / price <sup><a href="https://www.engineeringtoolbox.com/thermal-conductivity-metals-d_858.html">🔗</a></sup> |

---

| Design Aspect | Consideration | Best Practice |
|---|---|---|
| Contact area | Thermal resistance <sup><a href="https://www.electronics-cooling.com/2015/03/design-optimization-of-a-multi-device-single-phase-branching-microchannel-cold-plate/">🔗</a></sup> | Maximum coverage <sup><a href="https://www.electronics-cooling.com/2015/03/design-optimization-of-a-multi-device-single-phase-branching-microchannel-cold-plate/">🔗</a></sup> |
| Channel design | Turbulence / heat transfer <sup><a href="https://www.electronics-cooling.com/2022/10/heat-transfer-and-pressure-drop-correlations-for-manifold-microchannel-heat-sinks/">🔗</a></sup> | Microchannels <sup><a href="https://www.electronics-cooling.com/2022/10/heat-transfer-and-pressure-drop-correlations-for-manifold-microchannel-heat-sinks/">🔗</a></sup> |
| Flow rate | Liquid ΔT / thermal performance <sup><a href="https://par.nsf.gov/servlets/purl/10058150">🔗</a></sup> | Sufficient, not excessive <sup><a href="https://par.nsf.gov/servlets/purl/10058150">🔗</a></sup> |
| Pressure drop | Pump load <sup><a href="https://www.electronics-cooling.com/2022/10/heat-transfer-and-pressure-drop-correlations-for-manifold-microchannel-heat-sinks/">🔗</a></sup> | Minimize <sup><a href="https://www.electronics-cooling.com/2022/10/heat-transfer-and-pressure-drop-correlations-for-manifold-microchannel-heat-sinks/">🔗</a></sup> |
---

### Stranded Power in Datacenters

**Questions**
- What is stranded power?
- Why does it occur?
- How much power is typically stranded?
- How do you minimize stranded capacity?
<!-- Installed electrical capacity that cannot be used.
Cooling limits or rack-level imbalance.
10–30% of capacity.
Density zoning, liquid cooling, workload orchestration. -->

---

<!-- header: "AXIS 6: Power & Cooling" -->
## Part G: Companies & Industry Landscape

---

<!-- header: "Part G: Companies & Industry Landscape" -->
### System Vendors

<style scoped>
  section {
    font-size: 1.8rem;
    padding: 0;
    padding-top: 100px;
  }
</style>
| Company | Products | Cooling Approach | Market Position |
|---|---|---|---|
| NVIDIA <sup><a href="https://www.nvidia.com/en-us/data-center/products/">🔗</a></sup> | DGX <sup><a href="https://www.nvidia.com/en-us/data-center/dgx-platform/">🔗</a></sup>, MGX <sup><a href="https://www.nvidia.com/en-us/data-center/products/mgx/">🔗</a></sup>, HGX <sup><a href="https://www.nvidia.com/en-us/data-center/hgx/">🔗</a></sup> | Air + DLC ready <sup><a href="https://www.nvidia.com/en-us/data-center/dgx-platform/">🔗</a></sup> | AI platform leader <sup><a href="https://www.nvidia.com/en-us/data-center/products/">🔗</a></sup> |
| Dell <sup><a href="https://www.dell.com/en-us/shop/scc/sc/servers">🔗</a></sup> | PowerEdge XE <sup><a href="https://www.delltechnologies.com/asset/en-us/products/servers/technical-support/poweredge-xe-ai-spec-sheet.pdf">🔗</a></sup> | Air + DLC <sup><a href="https://www.delltechnologies.com/asset/en-us/products/servers/technical-support/poweredge-xe-ai-spec-sheet.pdf">🔗</a></sup> | Enterprise AI servers <sup><a href="https://www.delltechnologies.com/asset/en-us/products/servers/technical-support/poweredge-xe-ai-spec-sheet.pdf">🔗</a></sup> |
| HPE <sup><a href="https://www.hpe.com/psnow/doc/a00094635enw">🔗</a></sup> | Cray EX <sup><a href="https://www.hpe.com/psnow/doc/a00094635enw">🔗</a></sup> | DLC <sup><a href="https://www.hpe.com/psnow/doc/a00094635enw">🔗</a></sup> | HPC & AI leader <sup><a href="https://www.hpe.com/psnow/doc/a00094635enw">🔗</a></sup> |
| Supermicro <sup><a href="https://www.supermicro.com/en/products/gpu">🔗</a></sup> | GPU servers <sup><a href="https://www.supermicro.com/en/products/gpu">🔗</a></sup> | Air + DLC <sup><a href="https://www.supermicro.com/en/products/gpu">🔗</a></sup> | Broad OEM supplier <sup><a href="https://www.supermicro.com/en/products/gpu">🔗</a></sup> |
| Lenovo <sup><a href="https://www.lenovo.com/us/en/c/servers-storage/servers/racks/">🔗</a></sup> | ThinkSystem <sup><a href="https://www.lenovo.com/us/en/c/servers-storage/servers/racks/">🔗</a></sup> | Air + DLC <sup><a href="https://www.lenovo.com/fr/fr/servers-storage/servers/?srsltid=AfmBOooc9oE6RdoP4-pwKjlgYJ3sDAaUQ4lDjPwLn_mLz-_ox2S9yYY9">🔗</a></sup> | Enterprise / HPC <sup><a href="https://www.lenovo.com/us/en/c/servers-storage/servers/racks/">🔗</a></sup> |

---

### Cooling Infrastructure Vendors
<style scoped>
  section {
    font-size: 1.7rem;
    padding: 0;
    padding-top: 100px;
  }
</style>
| Company | Products | Technology Focus |
|---|---|---|
| Vertiv <sup><a href="https://www.vertiv.com/en-emea/products/trusted-names/liebert/">🔗</a></sup> | Liebert (cooling) <sup><a href="https://www.vertiv.com/en-emea/products/trusted-names/liebert/">🔗</a></sup>, thermal chain <sup><a href="https://www.vertiv.com/en-emea/products/thermal-management/cooling/">🔗</a></sup> | Full-stack cooling <sup><a href="https://www.vertiv.com/en-emea/products/thermal-management/cooling/">🔗</a></sup> |
| Schneider Electric <sup><a href="https://www.se.com/ww/en/work/solutions/for-business/data-centers-and-networks/">🔗</a></sup> | APC + cooling <sup><a href="https://www.se.com/ww/en/work/solutions/for-business/data-centers-and-networks/">🔗</a></sup> | Power + thermal <sup><a href="https://www.se.com/ww/en/work/solutions/for-business/data-centers-and-networks/">🔗</a></sup> |
| Asetek <sup><a href="https://www.asetek.com/liquid-cooling/data-center/">🔗</a></sup> | Cold plates / CDU <sup><a href="https://www.asetek.com/liquid-cooling/data-center/">🔗</a></sup> | DLC pioneer <sup><a href="https://www.asetek.com/liquid-cooling/data-center/">🔗</a></sup> |
| CoolIT <sup><a href="https://www.coolitsystems.com/">🔗</a></sup> | DLC systems <sup><a href="https://www.coolitsystems.com/">🔗</a></sup> | Rack-level DLC <sup><a href="https://www.coolitsystems.com/">🔗</a></sup> |
| GRC <sup><a href="https://www.grcooling.com/iceraq/">🔗</a></sup> | ICEraQ <sup><a href="https://www.grcooling.com/iceraq/">🔗</a></sup> | Single-phase immersion <sup><a href="https://www.grcooling.com/iceraq/">🔗</a></sup> |
| LiquidCool <sup><a href="https://liquidcoolsolutions.com/innovative-immersion-new/">🔗</a></sup> | Immersion tanks <sup><a href="https://liquidcoolsolutions.com/innovative-immersion-new/">🔗</a></sup> | Two-phase immersion <sup><a href="https://liquidcoolsolutions.com/innovative-immersion-new/">🔗</a></sup> |
| Submer <sup><a href="https://submer.com/blog/conscious-data-centre-solutions/">🔗</a></sup> | SmartPod <sup><a href="https://submer.com/blog/conscious-data-centre-solutions/">🔗</a></sup> | Immersion systems <sup><a href="https://submer.com/blog/conscious-data-centre-solutions/">🔗</a></sup> |

---

### Power Infrastructure Vendors

<style scoped>
  section {
    font-size: 2.0rem;
    padding: 0;
    padding-top: 100px;
  }
</style>
| Company | Products | Specialty |
|---|---|---|
| Schneider Electric <sup><a href="https://www.se.com/ww/en/work/solutions/for-business/data-centers-and-networks/">🔗</a></sup> | UPS, PDUs, switchgear <sup><a href="https://www.se.com/ww/en/work/solutions/for-business/data-centers-and-networks/">🔗</a></sup> | End-to-end power <sup><a href="https://www.se.com/ww/en/work/solutions/for-business/data-centers-and-networks/">🔗</a></sup> |
| Vertiv <sup><a href="https://www.vertiv.com/en-emea/products/thermal-management/cooling/">🔗</a></sup> | UPS, PDUs <sup><a href="https://www.vertiv.com/en-emea/products/trusted-names/liebert/">🔗</a></sup> | Critical power <sup><a href="https://www.vertiv.com/en-emea/products/trusted-names/liebert/">🔗</a></sup> |
| Eaton <sup><a href="https://www.eaton.com/us/en-us/products/backup-power-ups-surge-it-power-distribution/backup-power-ups.html">🔗</a></sup> | UPS <sup><a href="https://www.eaton.com/us/en-us/products/backup-power-ups-surge-it-power-distribution/backup-power-ups.html">🔗</a></sup>, PDUs <sup><a href="https://www.eaton.com/us/en-us/products/backup-power-ups-surge-it-power-distribution/power-distribution-for-it-equipment.html">🔗</a></sup> | Power distribution <sup><a href="https://www.eaton.com/us/en-us/products/backup-power-ups-surge-it-power-distribution/power-distribution-for-it-equipment.html">🔗</a></sup> |
| ABB <sup><a href="https://new.abb.com/docs/default-source/ewea-doc/built-for-reliability-and-efficiencyfe8e4be2c1f463c09537ff0000433538.pdf">🔗</a></sup> | Transformers <sup><a href="https://new.abb.com/docs/default-source/ewea-doc/built-for-reliability-and-efficiencyfe8e4be2c1f463c09537ff0000433538.pdf">🔗</a></sup>, switchgear <sup><a href="https://new.abb.com/medium-voltage/switchgear">🔗</a></sup> | Utility-scale <sup><a href="https://new.abb.com/medium-voltage/switchgear">🔗</a></sup> |
| Caterpillar <sup><a href="https://www.cat.com/en_GB/by-industry/electric-power/electric-power-industries/cat-standby-generators.html">🔗</a></sup> | Generators <sup><a href="https://www.cat.com/en_GB/by-industry/electric-power/electric-power-industries/cat-standby-generators.html">🔗</a></sup> | Backup power <sup><a href="https://www.cat.com/en_GB/by-industry/electric-power/electric-power-industries/cat-standby-generators.html">🔗</a></sup> |
---

<!-- header: "AXIS 6: Power & Cooling" -->
## Part H: Final Comparison

---

<style scoped>
  section {
    font-size: 1.5rem;
    padding: 0;
    padding-top: 100px
  }
</style>
<!-- header: "Part H: Final Comparison" -->

### Cooling Technology Comparison

| Aspect | Air Cooling | Rear-Door HX | Direct Liquid | Single-Phase Immersion | Two-Phase Immersion |
|---|---|---|---|---|---|
| Max kW/rack | ~20–50 kW <sup><a href="https://www.stulz.com/newsroom/detail/efficient-cooling-for-high-performance-computing-1/">🔗</a></sup> | ~20–80 kW <sup><a href="https://www.vertiv.com/en-us/about/news-and-insights/corporate-news/vertiv-announces-global-launch-of-chilled-water-rear-door-heat-exchanger-for-ai-and-hpc-applications/">🔗</a></sup> | ~50–200+ kW <sup><a href="https://www.1-act.com/resources/blog/ai-rack-cooling-data-centers/">🔗</a></sup> | ~80–250+ kW <sup><a href="https://www.grcooling.com/learning-center/tech-comparison-two-phase-vs-single-phase-immersion-cooling/">🔗</a></sup><sup><a href="https://www.sunbirddcim.com/glossary/single-phase-immersion-cooling">🔗</a></sup> | ~100–300+ kW <sup><a href="https://www.datacenterdynamics.com/en/analysis/getting-hotter-getting-denser/">🔗</a></sup> |
| PUE achievable | ~1.2–1.6 <sup><a href="https://www.energystar.gov/sites/default/files/tools/DataCenterFAQs.pdf">🔗</a></sup> | ~1.15–1.4 <sup><a href="https://datacenter.uptimeinstitute.com/rs/711-RIA-145/images/2024.GlobalDataCenterSurvey.Report.pdf">🔗</a></sup> | ~1.05–1.2 <sup><a href="https://tc090.ashraetcs.org/documents/ASHRAE_TC090_WhitePaper_LiquidCooling.pdf">🔗</a></sup> | ~1.03 <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> | ~1.02 <sup><a href="https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/">🔗</a></sup> |
| Capital cost | Low | Medium | High | High | Very high |
| Operating cost | Medium/High | Medium | Low/Medium | Low | Low |
| Maintenance | Simple | Moderate | Moderate/Complex | Complex | Very complex |
| Maturity | Very mature | Mature | Mature | Mature (commercial) | More niche |
| GPU compatibility | Universal | Universal | Cold plate required | Immersion-compatible hardware | Immersion-compatible hardware |

---

<style scoped>
  section {
    font-size: 1.7rem;
    padding: 0;
    padding-top: 100px
  }
</style>
### AI System Power Summary

<style scoped>
  section {
    font-size: 1.6rem;
    padding: 0;
    padding-top: 100px
  }
</style>
| System | GPUs | Total Power | Cooling Method | Rack Density |
|---|---|---:|---|---:|
| DGX A100 | 8× A100 | ~6.5 kW <sup><a href="https://images.nvidia.com/aem-dam/Solutions/Data-Center/nvidia-dgx-a100-datasheet.pdf">🔗</a></sup> | Air | Variable (depends on servers per rack) |
| DGX H100 | 8× H100 | ~10.2 kW <sup><a href="https://docs.nvidia.com/dgx/dgxh100-user-guide/introduction-to-dgxh100.html">🔗</a></sup> | Air / DLC | Variable |
| DGX B200 | 8× B200 | ~14.3 kW <sup><a href="https://www.nvidia.com/en-us/data-center/dgx-b200/">🔗</a></sup> | DLC | Variable |
| GB200 NVL72 | 72× B200 | ~120 kW <sup><a href="https://flopper.io/system/nvidia-gb200-nvl72">🔗</a></sup> | DLC | ~120 kW/rack <sup><a href="https://flopper.io/system/nvidia-gb200-nvl72">🔗</a></sup> |
| AMD MI300X (8-way) | 8× MI300X | ~6.0 kW (GPUs only) <sup><a href="https://www.phoronix.com/review/amd-instinct-mi300x-rocm6">🔗</a></sup> | Air | Variable |
| Google TPU v5p pod | 8,960 TPU v5p chips <sup><a href="https://cloud.google.com/blog/products/ai-machine-learning/introducing-cloud-tpu-v5p-and-ai-hypercomputer">🔗</a></sup> | Not disclosed | DLC | Not disclosed |

---

<style scoped>
  section {
    font-size: 1.4rem;
    padding: 0;
    padding-top: 100px
  }
</style>
### Datacenter Efficiency Comparison

| Operator | Facility Type | PUE | WUE | Cooling Method |
|---|---|---:|---:|---|
| Google | Hyperscale | 1.09 (TTM) <sup><a href="https://datacenters.google/efficiency">🔗</a></sup> | Not publicly disclosed (global) | Mix (free cooling + liquid depending on site) |
| Meta | Hyperscale | 1.08 (2023 avg) <sup><a href="https://sustainability.atmeta.com/wp-content/uploads/2024/08/Meta-2024-Sustainability-Report.pdf">🔗</a></sup> | WUE reported (see sustainability report) <sup><a href="https://sustainability.atmeta.com/wp-content/uploads/2024/08/Meta-2024-Sustainability-Report.pdf">🔗</a></sup> | Mix (air optimizations + water systems) |
| Microsoft | Azure | Not publicly disclosed (global) | WUE improvement reported <sup><a href="https://www.microsoft.com/en-us/microsoft-cloud/blog/2024/12/09/sustainable-by-design-next-generation-datacenters-consume-zero-water-for-cooling/">🔗</a></sup> | Mix (air + “zero-water” innovations on new sites) |
| AWS | Cloud | 1.15 (global) <sup><a href="https://aws.amazon.com/sustainability/data-centers/">🔗</a></sup> | Not publicly disclosed (global) | Mix (air + optimizations) |
| CoreWeave | AI-focused | 1.15 (Barcelona site announced) <sup><a href="https://www.edged.es/news/coreweave-partners-with-edged">🔗</a></sup> | “Zero water” announced (Barcelona site) <sup><a href="https://www.edged.es/news/coreweave-partners-with-edged">🔗</a></sup> | Free air + optimized design <sup><a href="https://www.edged.es/news/coreweave-partners-with-edged">🔗</a></sup> |
| Lambda Labs | AI-focused | Not disclosed | Not disclosed | Not disclosed |

---

<style scoped>
  section {
    font-size: 1.8rem;
    padding: 0;
    padding-top: 100px
  }
</style>
### TCO Impact of Cooling Choice

| Cost Component | Air Cooling | DLC | Immersion |
|---|---:|---:|---:|
| Capital ($/kW IT) | Low | High | High / Very high |
| Power cost ($/kW-yr) | High (fans/HVAC) | Lower | Lower |
| Maintenance ($/kW-yr) | Low | Medium | High |
| Floor space ($/kW-yr) | High (limited density) | Lower | Lower |
| 5-year TCO ($/kW) | Variable (often higher at high density) | Often lower at high density | Often lower if extreme density |