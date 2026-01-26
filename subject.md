# AXIS 6: Power & Cooling (Comparative Analysis)

**Presentation Format:** 30–45 min presentation (through Google Meet)

## Mission
- Explain the power and thermal challenges of modern AI infrastructure
- Compare cooling technologies and analyze datacenter design tradeoffs
- YOU CAN USE CHATGPT but always add source to every statements you say
- If unsure of statements (because no sources found), put in *italic in a different colors*
- Some data in tables below were not checked. If you decide to use them, feel free but it needs sources OR a formula that explains how to get the value

---

## Part A: The Power Trajectory of AI Systems

### GPU Power Evolution

**Key questions**
- How has GPU power consumption evolved?
- What is driving the power increases?
- Is there a ceiling to GPU power?
- How does power scale with performance?

| GPU | Year | TDP (W) | FP16 TFLOPs | W/TFLOP | Process Node |
|---|---:|---:|---:|---:|---|
| V100 | 2017 | 300 | 125 | 2.40 | TSMC 12nm FFN |
| A100 | 2020 | 400 | 624 | 0.64 | TSMC 7nm |
| H100 SXM | 2022 | 700 | 1979 | 0.35 | TSMC 4N |
| H200 SXM | 2024 | 700 | 1979 | 0.35 | TSMC 4N |
| B200 SXM | 2024 | 1000 | 4500 | 0.22 | TSMC 4NP |
| B300 | 2025 | 1100 | 5000 | 0.22 | TSMC 4NP |
| R100 (Rubin) | 2026 | _~2300_ | _~4000_ | _~0.58_ | _TSMC N3_ |

Sources:
 - https://www.nvidia.com
 - https://www.tsmc.com/english/dedicatedFoundry/technology
 - https://www.anandtech.com/  
 - https://www.servethehome.com/

**System-level power (full node)**

| System | GPUs | GPU Power | Total System Power | Year |
|---|---|---:|---:|---:|
| DGX-1 | 8× V100 | 2.4 kW | ~3.5 kW | 2017 |
| DGX A100 | 8× A100 | 3.2 kW | ~6.5 kW | 2020 |
| DGX H100 | 8× H100 SXM | 5.6 kW | ~10.2 kW | 2022 |
| DGX B200 | 8× B200 SXM | 8.0 kW | ~14.3 kW | 2024 |
| GB200 NVL72 | 72× B200 | 72.0 kW | ~120 kW | 2024 |

Sources:
- https://www.nvidia.com/en-us/data-center/
- https://www.servethehome.com/nvidia-gb200-nvl72-power-cooling-and-rack-scale-design/
- https://www.anandtech.com/

**Rack power density evolution**

| Era | Typical Rack Power | kW/rack | Cooling Method |
|---|---:|---:|---|
| Traditional IT (2010) | 5–10 kW/rack | ~8 | Air |
| GPU compute (2018) | 15–25 kW/rack | ~20 | Air |
| AI training (2022) | 30–50 kW/rack | ~40 | Air / DLC |
| AI training (2024) | 60–120 kW/rack | ~80 | DLC required |
| AI training (2026) | 150–300 kW/rack | ~200 | DLC / Immersion |

Sources:
- https://www.se.com/ww/en/work/solutions/for-business/data-centers-and-networks/
- https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/
- https://uptimeinstitute.com/resources/research-and-reports
- https://www.nvidia.com/en-us/data-center/
- https://www.servethehome.com/nvidia-gb200-nvl72-power-cooling-and-rack-scale-design/
- https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments
- https://engineering.fb.com/

---

## Part B: GPU Power Delivery

### How does GPU power delivery work?

**Questions**
- From grid to chip: what are the conversion stages?
- Where are the efficiency losses?
- How is power distributed within a GPU?

**Power delivery hierarchy**
- Grid (AC) → Transformer → UPS → PDU → PSU → VRM → GPU Die

| Stage | Input | Output | Typical Efficiency | Loss |
|---|---|---|---:|---:|
| Utility transformer | HV AC (10–30 kV) | LV AC (400–480 V) | 98–99% | 1–2% |
| UPS | AC | AC (conditioned) | 94–97% | 3–6% |
| PDU | AC | AC | 98–99% | 1–2% |
| PSU (AC-DC) | AC (200–240 V) | DC (12 V / 48 V) | 92–96% | 4–8% |
| VRM (DC-DC) | DC (12 V / 48 V) | DC (~0.7–1.1 V) | 85–95% | 5–15% |

Sources:
 - https://www.schneider-electric.com/en/work/solutions/for-business/data-centers-and-networks/
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/data-center-power-chain/
 - https://www.eaton.com/us/en-us/products/backup-power-ups-surge-it-power-distribution/data-center-ups-efficiency.html
 - https://www.plugloadsolutions.com/80pluspowersupplies.aspx
 - https://www.ti.com/power-management/vrm.html
 - https://www.intel.com/content/www/us/en/products/docs/processors/core/vrm-design-guide.html


### Voltage Regulator Modules (VRMs)

**Questions**
- What is a VRM and why is it critical?
Le VRM est chargé de convertir le 12/48 V en tension <1V pour les cors.
Il est critique dans le sens où il exige une tension d'entrée stable et sans bruit.
- What are the phases and how do they work?
Une phase = controleur + MOSFETs + inductance
Les phases fonctionnent en parallèle pour partager le courant et réduire l'ondulation.
- Why is VRM efficiency important?
Si le GPU consomme 1 kW d'éléctricité et possède un rendement de 90%, il dissipe donc 10% soit 100 W en chaleur qu'il faut évacuer.
- What are the thermal challenges?
Plusieurs défit :
 - Réduire la perte de puissance d'entrée en chaleur
 - Limiter l'impact de la chaleur sur les performance
 - Evacuer la chaleur le plus efficacement possible

| VRM Aspect | Description | Typical Value |
|---|---|---|
| Input voltage | Voltage supplied to VRM | 12 V or 48 V |
| Output voltage | Core supply voltage (Vcore) | 0.7–1.1 V |
| Phase count | Parallel VRM phases | 10–20 phases |
| Efficiency | DC-DC conversion efficiency | 85–95% |
| Power loss (1kW GPU) | Heat dissipated by VRM | 50–150 W |

Sources:
 - https://www.ti.com/power-management/vrm.html
 - https://www.intel.com/content/www/us/en/products/docs/processors/core/vrm-design-guide.html
 - https://www.analog.com/en/analog-dialogue/articles/multiphase-buck-converters.html
 - https://www.anandtech.com/show/17626/nvidia-hopper-h100-architecture-deep-dive

### 12V vs 48V power distribution

**Questions**
- Why is 48V better for high-power systems?
Parce qu'avec une tension 4 x plus élevée on diminue l'intensité par 4 pour avoir la même puissance tout en réduisant les pertes sous formes de chaleurs.
- What is the current reduction benefit?
- Who is pushing 48V adoption?
Un peu tout le monde qui gravite autour des GPU
- What are the challenges?
Les VRM sont plus complexes, le risques de pont éléctrique entre deux composant unltra-proche est plus élevé, il faut assurer la compatibilité sur les appareils supports...

| Aspect | 12V Distribution | 48V Distribution |
|---|---|---|
| Current for 1kW | ~83 A | ~21 A |
| Cable thickness | Thick (high current) | Thinner |
| I²R losses | High (×16 vs 48V) | Much lower |
| Connector size | Large / multiple pins | Smaller / fewer pins |
| Industry adoption | Legacy servers | AI & hyperscale |

Sources:
 - https://www.opencompute.org/documents/ocp-48v-dc-power-distribution
 - https://www.nvidia.com/en-us/data-center/energy-efficiency/
 - https://www.ti.com/power-management/48v.html
 - https://www.analog.com/en/analog-dialogue/articles/why-48v.html
 - https://www.anandtech.com/show/17626/nvidia-hopper-h100-architecture-deep-dive

### Connector considerations

| Connector | Max Power | Pins | Issues |
|---|---:|---:|---|
| 8-pin PCIe | 150 W | 8 | Limited power, multiple cables needed |
| 12VHPWR | 600 W | 16 (12+4 sense) | Overheating, insertion sensitivity |
| 12V-2x6 | 600 W | 16 (12+4 sense) | Improved safety, still high current |

Sources:
 - https://www.pcisig.com/specifications/pciexpress
 - https://www.nvidia.com/en-us/geforce/news/12vhpwr-connector/
 - https://www.pcisig.com/news/pcisig-introduces-12v-2x6-connector
 - https://www.anandtech.com/show/17530/nvidia-12vhpwr-issues-and-analysis

### 800V DC for AI factories

**Questions**
- Why is NVIDIA pushing 800V DC?
Pour réduire l'intensité qui circule dans les racks de très haute puissance et réduire par conséquent les pertes par effet Joule.
- What efficiency gains are possible?
Il y aurait moins d'étage de conversion donc moins de source de perte et meilleur acheminement du courant le long des racks.
- How does this change datacenter design?
- What are the safety considerations?
DC haute tension = arcs persistants, exigences accrues d’isolation, de coupure, de procédures et de conformité (personnel formé).

---

## Part C: Cooling Technologies

### Air Cooling

**Questions**
- How does air cooling work?
Par convection, l'élément chaud rechauffe l'air autour de lui, avec un ventilateur ou autre, l'air chaud est chassé, réablissant le déséquilibre de température. Le second principe de thermodynamique prend le relai en refroidissant l'élément chaud et en rechauffant l'air et ainsi de suite.
- Heat sink design principles
Le but est d'optimisé la surface d'échange thermique entre l'élément chaud et l'air
- What are the limits?
Le refroissiement par convection a rendement plus faible que par conduction.
- When does air cooling fail?
Si l'air ambiant est chaud, celui-ci peut ne plus absorber assez de chaleur pour refroidir efficacement l'élément chaud
- At what TDP does air become impractical?
~750 W
- What are the acoustic limits?
~75dB
- How does altitude affect air cooling?
+altitude = air moins dense = refroidissmeent moins efficace pour un même volume d'air = débit plus élévé nécessaire

| Air Cooling Aspect | Typical Value | Limit |
|---|---:|---:|
| Max heat dissipation per GPU | 400–600 W | ~700–800 W |
| Max rack density | 20–40 kW | ~50 kW |
| Airflow per rack | 85–170 $m^3m^{-1}$ | ~225 $m^3m^{-1}$  |
| Fan power overhead | 5–10% | ~15% |

Sources:
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/high-density-data-center-cooling/
 - https://www.schneider-electric.com/en/work/solutions/for-business/data-centers-and-networks/
 - https://www.servethehome.com/nvidia-gb200-nvl72-power-cooling-and-rack-scale-design/

### Direct Liquid Cooling (DLC)

**How does DLC work?**
- Cold plate design and attachment
plaque métallique en contact direct avec GPU, chaleur transférée au fluide
- Manifold and distribution systems
réseaux de tuyauterie qui distribuent le fluide vers chaque cold plate.
- Coolant types and flow rates
eau/glycol ou fluides diélectriques, débit optimisé pour ΔT cible.
- Heat rejection (CDU, dry coolers, cooling towers)
CDU (unités de distribution de fluide) + échangeurs externes (dry coolers ou tours).

| DLC Component | Function | Key Specifications |
|---|---|---|
| Cold plate | Transfert de chaleur du GPU vers le liquide | Surface alu/cuivre, low ΔT |
| Manifold | Distribution du fluide aux cold plates | Multi-port, équilibrage de débit |
| Quick disconnects | Connexions sans fuite pour maintenance | ≤ 0.5% fuite, haute pression |
| CDU (Coolant Distribution Unit) | Pompe + échangeur + filtration | Débit 10–100 L/min, filtres |
| Facility water loop | Rejet de chaleur vers le bâtiment | Dry coolers ou cooling towers |


| Aspect | Air Cooling | Direct Liquid Cooling |
|---|---|---|
| Max GPU TDP supported | ~700–800 W | ~1500+ W |
| Max rack density | ~40–50 kW | ~100–200+ kW |
| PUE impact | Higher overhead | Lower overhead |
| Maintenance complexity | Low | Medium/High |
| Capital cost | Lower | Higher (install + CDU) |
| Operating cost | Higher fan energy | Lower pumping energy |

Sources:
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/data-center-liquid-cooling/
 - https://www.nvidia.com/en-us/data-center/energy-efficiency/
 - https://www.servethehome.com/nvidia-gb200-nvl72-power-cooling-and-rack-scale-design/
 - https://www.coolingbestpractices.com/knowledge_center/whitepapers/direct_liquid_cooling
 - https://www.schneider-electric.com/en/work/solutions/for-business/data-centers-and-networks/


### Coolant types

| Coolant | Thermal Properties | Cost | Safety | Use Case |
|---|---|---|---|---|
| Water/glycol | Very high heat capacity, high thermal conductivity | Low | Conductive, leak risk | Cold plates, DLC loops |
| Propylene glycol | Slightly lower than water, freeze protection | Low–Medium | Less toxic than ethylene glycol | Cold climates, DLC |
| Dielectric fluids | Lower than water, electrically insulating | High | Non-conductive, safer for electronics | Immersion cooling |

### Immersion Cooling

**Topics**
- Single-phase vs two-phase  
- Tank design and fluid management  
- Heat rejection methods  
- Maintenance considerations  

| Aspect | Single-Phase Immersion | Two-Phase Immersion |
|---|---|---|
| Fluid type | Dielectric oil | Low-boiling dielectric fluid |
| Operating principle | Liquid absorbs heat, stays liquid | Liquid boils, vapor condenses |
| Max heat flux | High | Very high |
| Fluid cost | Medium | High |
| Complexity | Medium | High |
| Maturity | Commercially mature | Emerging / niche |

| Advantage | Challenge |
|---|---|
| Highest heat density | Specialized fluids |
| No fans required | Hardware compatibility |
| Reduced PUE | Higher CAPEX |
| Component longevity | Maintenance procedures |

Sources:
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments  
 - https://www.submer.com/immersion-cooling/  
 - https://www.grcooling.com/immersion-cooling/  
 - https://www.3m.com/3M/en_US/data-center-us/solutions/liquid-cooling/  
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/data-center-liquid-cooling/


### Rear-Door Heat Exchangers (RDHx)

**Questions**
- What is RDHx?  
  Un échangeur thermique air-liquide monté à l’arrière d’un rack qui capte l’air chaud sortant et en extrait la chaleur via un circuit d’eau.

- How does it supplement air cooling?  
  Il refroidit l’air directement au niveau du rack, réduisant la charge thermique de la salle et améliorant l’efficacité du refroidissement à air existant.

- What densities can it support?  
  Environ 20 000 à 80 000 W par rack selon le type (passif ou actif).

- When is it the right choice?  
  Lorsqu’un datacenter refroidi à l’air doit supporter des racks plus denses sans passer immédiatement au liquid cooling direct ou à l’immersion.

| RDHx Type | Cooling Capacity | Best For |
|---|---:|---|
| Passive RDHx | ~20 000–35 000 W par rack | Racks densité moyenne, retrofit |
| Active RDHx | ~30 000–80 000 W par rack | Racks haute densité (GPU) |

Sources:
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/rear-door-heat-exchangers/  
 - https://www.schneider-electric.com/en/work/solutions/for-business/data-centers-and-networks/rear-door-heat-exchanger/  
 - https://www.asetek.com/liquid-cooling/technologies/rear-door-heat-exchanger/  
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments


---

## Part D: Power Usage Effectiveness (PUE)

### Understanding PUE

**Definition**
- PUE = Total Facility Power / IT Equipment Power  
- 1 PUE = (IT Load + Cooling + Power Distribution + Lighting + Other) / IT Load

**Questions**
- What does PUE measure and not measure?  
  PUE mesure l’efficacité énergétique de l’infrastructure du datacenter mais ne mesure pas l’efficacité des applications, des serveurs ou de l’usage des ressources IT.

- What are the components of overhead?  
  Refroidissement, pertes de distribution électrique, UPS, éclairage, sécurité, auxiliaires.

- How does cooling choice affect PUE?  
  Les solutions liquid cooling réduisent fortement l’énergie dédiée au refroidissement, ce qui diminue le PUE.

| PUE Component | Typical % of Overhead | Reduction Strategies |
|---|---:|---|
| Cooling | 40–60 % | DLC, free cooling, confinement |
| Power distribution losses | 10–15 % | Haut rendement UPS/PSU |
| Lighting and other | 5 % | LED, automatisation |

**PUE benchmarks**

| Datacenter Type | Typical PUE | Best-in-Class PUE |
|---|---:|---:|
| Legacy enterprise | 1.8–2.5 | 1.6 |
| Modern enterprise | 1.4–1.6 | 1.3 |
| Hyperscale (air) | 1.2–1.4 | 1.1 |
| Hyperscale (DLC) | 1.1–1.2 | 1.05 |
| AI-optimized | 1.1–1.3 | 1.05 |

| Cooling Method | Typical PUE | Why |
|---|---:|---|
| Traditional air (CRAC) | ~1.6 | Compresseurs énergivores |
| Hot/cold aisle containment | ~1.4 | Airflow optimisé |
| Free air cooling | ~1.2 | Peu de refroidissement actif |
| Direct liquid cooling | ~1.1 | Transfert thermique direct |
| Immersion cooling | ~1.05 | Quasi suppression HVAC |

**Best-in-class examples**

| Company | Facility | PUE | How Achieved |
|---|---|---:|---|
| Google | Hyperscale DC | 1.10 | Free cooling + AI control |
| Meta | Hyperscale DC | 1.09 | Free cooling |
| Microsoft | Azure DC | 1.12 | Free air + DLC |
| NVIDIA DGX Cloud | AI DC | ~1.1 | Liquid cooling |

**Beyond PUE: other efficiency metrics**

| Metric | Definition | Typical Values | Best-in-Class |
|---|---|---|---|
| PUE | Ratio total/IT | 1.1–1.6 | ~1.05 |
| WUE | L d’eau / kWh IT | 0.2–1.8 | <0.2 |
| CUE | kgCO₂ / kWh IT | Variable | Near zero |

Sources:
 - https://uptimeinstitute.com/resources/what-is-pue  
 - https://www.google.com/about/datacenters/efficiency/  
 - https://engineering.fb.com/2020/03/12/data-center-engineering/data-center-efficiency/  
 - https://learn.microsoft.com/en-us/azure/sustainability/  
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments  
 - https://www.iea.org/reports/data-centres-and-data-transmission-networks


---

## Part E: Infrastructure Requirements

### Power Density Planning

**Questions**
- How do you plan for increasing density?  
  Modular power and cooling, oversizing pathways, liquid-ready racks.
- What infrastructure upgrades are needed?  
  Higher-capacity busways, transformers, UPS, liquid cooling loops.
- How do you handle mixed densities?  
  Zoning: air-cooled rows + liquid-cooled rows.

| Density Tier | kW/Rack | Infrastructure Requirements |
|---|---:|---|
| Low density | 5 000–10 000 W | Air cooling, standard power |
| Medium density | 10 000–30 000 W | Hot/cold aisle, high airflow |
| High density | 30 000–80 000 W | RDHx or DLC |
| Ultra-high density | >80 000 W | DLC or immersion |

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

### Cooling Capacity Planning

| Cooling capacity Unit | Conversion | Context |
|---|---|---|
| 1 ton of cooling | 12 000 BTU/h | 3.52 kW |
| 1 kW IT load (air) | 0.0003 ton | Includes overhead |
| 1 kW IT load (DLC) | 0.00028 ton | Direct rejection |

**Water requirements for liquid cooling**

| Cooling Method | Water Usage | m³/h per MW |
|---|---|---:|
| Evaporative (cooling tower) | High | 1.5–2 |
| Dry cooler | Low | 0.1–0.3 |
| DLC (closed loop) | Very low | ~0.05 |

---

### Backup Power Systems

| UPS Type | Efficiency | Runtime | Best For |
|---|---:|---:|---|
| Double conversion | 94–97% | 5–15 min | Critical loads |
| Line interactive | 96–98% | 5–10 min | Edge DC |
| Rotary UPS | 96–98% | Seconds | Large DC |
| Battery + flywheel | 95–98% | Seconds–minutes | High power |

**Generator requirements**

| Facility Size | Generator Capacity | Fuel Storage | Startup Time |
|---:|---|---|---|
| 10 MW | 10–12 MW | Hours | <10 s |
| 100 MW | 120 MW | Hours | <10 s |
| 500 MW | 600 MW | Hours | <15 s |
| 1 GW | 1.2 GW | Hours | <15 s |

---

### Grid Connection for GW-Scale Facilities

| Scale | Grid Requirements | Typical Lead Time |
|---:|---|---|
| 50 MW | Substation tie-in | 1–2 years |
| 200 MW | Dedicated substation | 2–3 years |
| 500 MW | Transmission upgrade | 3–5 years |
| 1 GW+ | New transmission lines | 5+ years |

---

### Power sourcing strategies

| Strategy | Description | Pros | Cons |
|---|---|---|---|
| Grid connection | Utility power | Simple | Carbon intensity |
| On-site generation | Gas/diesel | Fast backup | Emissions |
| PPA | Long-term renewable contract | Green energy | Price lock |
| Behind-the-meter solar/wind | Local generation | Low carbon | Intermittent |
| Nuclear (SMR) | Small modular reactors | Stable low-carbon | Regulatory |

Sources:
 - https://uptimeinstitute.com/resources/what-is-pue  
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments  
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/data-center-power-and-cooling/  
 - https://www.schneider-electric.com/en/work/solutions/for-business/data-centers-and-networks/  
 - https://www.iea.org/reports/data-centres-and-data-transmission-networks  
 - https://www.energy.gov/ne/articles/small-modular-reactors


---

## Part F: Deep Dive Topics

### Chip-Level Power Management

**Dynamic Voltage and Frequency Scaling (DVFS)**
- How does DVFS work?  
  Le processeur ajuste dynamiquement la tension et la fréquence selon la charge et la température.
- What is the power/frequency relationship?  
  La puissance dynamique suit approximativement : P ∝ V² × f.

| Power State | Voltage | Frequency | Power | Use Case |
|---|---:|---:|---:|---|
| Max boost | Élevée | Élevée | Très élevée | Peak compute |
| Base clock | Nominale | Nominale | Élevée | Sustained |
| Idle | Basse | Basse | Faible | Low utilization |
| Sleep | Très basse | ~0 | Très faible | Inactive |

---

### Thermal Throttling Behavior

| Threshold | Temperature | Action |
|---|---|---|
| Target | ~83 °C | Fréquence nominale |
| Throttle start | ~85 °C | Réduction fréquence |
| Max operating | ~90 °C | Limitation puissance |
| Shutdown | ~95 °C | Arrêt matériel |

---

### Heat Sink and Cold Plate Design

| Parameter | Impact | Tradeoff |
|---|---|---|
| Fin density | Surface d’échange | Restriction airflow |
| Base thickness | Diffusion chaleur | Masse |
| Heat pipe count | Transport chaleur | Coût |
| Material (Cu vs Al) | Conductivité | Poids / prix |

| Design Aspect | Consideration | Best Practice |
|---|---|---|
| Contact area | Résistance thermique | Couverture maximale |
| Channel design | Turbulence | Micro-canaux |
| Flow rate | ΔT liquide | Suffisant sans excès |
| Pressure drop | Charge pompe | Minimiser |

---

### Stranded Power in Datacenters

**Questions**
- What is stranded power?  
  Puissance électrique installée mais inutilisable.

- Why does it occur?  
  Limites de refroidissement ou déséquilibre rack-level.

- How much power is typically stranded?  
  10–30 % de capacité.

- How do you minimize stranded capacity?  
  Zoning densité, liquid cooling, orchestration workload.

Sources:
 - https://developer.nvidia.com/blog/nvidia-hopper-architecture-in-depth/  
 - https://www.anandtech.com/show/17626/nvidia-hopper-h100-architecture-deep-dive  
 - https://www.intel.com/content/www/us/en/products/docs/processors/core/vrm-design-guide.html  
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments  
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/data-center-power-and-cooling/  
 - https://uptimeinstitute.com/resources


---

## Part G: Companies & Industry Landscape

### System Vendors

| Company | Products | Cooling Approach | Market Position |
|---|---|---|---|
| NVIDIA | DGX, MGX, HGX | Air + DLC ready | Leader AI platforms |
| Dell | PowerEdge XE | Air + DLC | Enterprise AI servers |
| HPE | Cray EX | DLC | HPC & AI leader |
| Supermicro | GPU servers | Air + DLC | Broad OEM supplier |
| Lenovo | ThinkSystem | Air + DLC | Enterprise/HPC |

### Cooling Infrastructure Vendors

| Company | Products | Technology Focus |
|---|---|---|
| Vertiv | Liebert, CDUs | Full stack cooling |
| Schneider Electric | APC, cooling | Power + thermal |
| Asetek | Cold plates, CDUs | DLC pioneer |
| CoolIT | DLC systems | Rack-level DLC |
| GRC | ICEraQ | Single-phase immersion |
| LiquidCool | Immersion tanks | Two-phase immersion |
| Submer | SmartPod | Immersion systems |

### Power Infrastructure Vendors

| Company | Products | Specialty |
|---|---|---|
| Schneider Electric | UPS, PDUs, switchgear | End-to-end power |
| Vertiv | UPS, PDUs | Critical power |
| Eaton | UPS, PDUs | Power distribution |
| ABB | Transformers, switchgear | Utility-scale |
| Caterpillar | Generators | Backup power |

Sources:
 - https://www.nvidia.com/en-us/data-center/  
 - https://www.dell.com/en-us/dt/servers/poweredge-xe.htm  
 - https://www.hpe.com/us/en/compute/hpc/cray-ex.html  
 - https://www.supermicro.com/en/products/gpu  
 - https://www.lenovo.com/us/en/servers-storage/servers/thinksystem/  
 - https://www.vertiv.com/  
 - https://www.se.com/ww/en/work/solutions/for-business/data-centers-and-networks/  
 - https://www.asetek.com/data-center-liquid-cooling/  
 - https://www.coolitsystems.com/  
 - https://www.grcooling.com/  
 - https://www.submer.com/  
 - https://www.eaton.com/  
 - https://new.abb.com/  
 - https://www.caterpillar.com/

---

## Part H: Summary Comparison

### Cooling Technology Comparison

| Aspect | Air Cooling | Rear-Door HX | Direct Liquid | Single-Phase Immersion | Two-Phase Immersion |
|---|---|---|---|---|---|
| Max kW/rack | ? | ? | ? | ? | ? |
| PUE achievable | ? | ? | ? | ? | ? |
| Capital cost | ? | ? | ? | ? | ? |
| Operating cost | ? | ? | ? | ? | ? |
| Maintenance | ? | ? | ? | ? | ? |
| Maturity | ? | ? | ? | ? | ? |
| GPU compatibility | ? | ? | ? | ? | ? |

### AI System Power Summary

| System | GPUs | Total Power | Cooling Method | Rack Density |
|---|---|---:|---|---:|
| DGX A100 | 8× A100 | ~6.5 kW | Air | ? |
| DGX H100 | 8× H100 | ~10.2 kW | Air/DLC | ? |
| DGX B200 | 8× B200 | ~14.3 kW | DLC | ? |
| GB200 NVL72 | 72× B200 | ~120 kW | DLC | ? |
| AMD MI300X (8-way) | 8× MI300X | ? | ? | ? |
| Google TPU v5p pod | ? | ? | ? | ? |

### Datacenter Efficiency Comparison

| Operator | Facility Type | PUE | WUE | Cooling Method |
|---|---|---:|---:|---|
| Google | Hyperscale | ? | ? | ? |
| Meta | Hyperscale | ? | ? | ? |
| Microsoft | Azure | ? | ? | ? |
| AWS | Cloud | ? | ? | ? |
| CoreWeave | AI-focused | ? | ? | ? |
| Lambda Labs | AI-focused | ? | ? | ? |

### TCO Impact of Cooling Choice

| Cost Component | Air Cooling | DLC | Immersion |
|---|---:|---:|---:|
| Capital ($/kW IT) | ? | ? | ? |
| Power cost ($/kW-yr) | ? | ? | ? |
| Maintenance ($/kW-yr) | ? | ? | ? |
| Floor space ($/kW-yr) | ? | ? | ? |
| 5-year TCO ($/kW) | ? | ? | ? |
