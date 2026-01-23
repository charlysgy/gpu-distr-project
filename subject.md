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
- Heat sink design principles
- What are the limits?
- When does air cooling fail?
- At what TDP does air become impractical?
- What are the acoustic limits?
- How does altitude affect air cooling?

| Air Cooling Aspect | Typical Value | Limit |
|---|---:|---:|
| Max heat dissipation per GPU | ? | ? |
| Max rack density | ? | ? |
| Airflow per rack | ? | ? |
| Fan power overhead | ? | ? |

### Direct Liquid Cooling (DLC)

**How does DLC work?**
- Cold plate design and attachment
- Manifold and distribution systems
- Coolant types and flow rates
- Heat rejection (CDU, dry coolers, cooling towers)

| DLC Component | Function | Key Specifications |
|---|---|---|
| Cold plate | ? | ? |
| Manifold | ? | ? |
| Quick disconnects | ? | ? |
| CDU (Coolant Distribution Unit) | ? | ? |
| Facility water loop | ? | ? |

| Aspect | Air Cooling | Direct Liquid Cooling |
|---|---|---|
| Max GPU TDP supported | ? | ? |
| Max rack density | ? | ? |
| PUE impact | ? | ? |
| Maintenance complexity | ? | ? |
| Capital cost | ? | ? |
| Operating cost | ? | ? |

### Coolant types

| Coolant | Thermal Properties | Cost | Safety | Use Case |
|---|---|---|---|---|
| Water/glycol | ? | ? | ? | ? |
| Propylene glycol | ? | ? | ? | ? |
| Dielectric fluids | ? | ? | ? | ? |

### Immersion Cooling

**Topics**
- Single-phase vs two-phase
- Tank design and fluid management
- Heat rejection methods
- Maintenance considerations

| Aspect | Single-Phase Immersion | Two-Phase Immersion |
|---|---|---|
| Fluid type | ? | ? |
| Operating principle | ? | ? |
| Max heat flux | ? | ? |
| Fluid cost | ? | ? |
| Complexity | ? | ? |
| Maturity | ? | ? |

| Advantage | Challenge |
|---|---|
| Highest heat density | ? |
| No fans required | ? |
| Reduced PUE | ? |
| Component longevity | ? |

### Rear-Door Heat Exchangers (RDHx)

**Questions**
- What is RDHx?
- How does it supplement air cooling?
- What densities can it support?
- When is it the right choice?

| RDHx Type | Cooling Capacity | Best For |
|---|---:|---|
| Passive RDHx | ? | ? |
| Active RDHx | ? | ? |

---

## Part D: Power Usage Effectiveness (PUE)

### Understanding PUE

**Definition**
- PUE = Total Facility Power / IT Equipment Power  
- 1 PUE = (IT Load + Cooling + Power Distribution + Lighting + Other) / IT Load

**Questions**
- What does PUE measure and not measure?
- What are the components of overhead?
- How does cooling choice affect PUE?

| PUE Component | Typical % of Overhead | Reduction Strategies |
|---|---:|---|
| Cooling | ? | ? |
| Power distribution losses | ? | ? |
| Lighting and other | ? | ? |

**PUE benchmarks**

| Datacenter Type | Typical PUE | Best-in-Class PUE |
|---|---:|---:|
| Legacy enterprise | ? | ? |
| Modern enterprise | ? | ? |
| Hyperscale (air) | ? | ? |
| Hyperscale (DLC) | ? | ? |
| AI-optimized | ? | ? |

| Cooling Method | Typical PUE | Why |
|---|---:|---|
| Traditional air (CRAC) | ? | ? |
| Hot/cold aisle containment | ? | ? |
| Free air cooling | ? | ? |
| Direct liquid cooling | ? | ? |
| Immersion cooling | ? | ? |

**Best-in-class examples**

| Company | Facility | PUE | How Achieved |
|---|---|---:|---|
| Google | ? | ? | ? |
| Meta | ? | ? | ? |
| Microsoft | ? | ? | ? |
| NVIDIA DGX Cloud | ? | ? | ? |

**Beyond PUE: other efficiency metrics**

| Metric | Definition | Typical Values | Best-in-Class |
|---|---|---|---|
| PUE | ? | ? | ? |
| WUE | ? | ? | ? |
| CUE | ? | ? | ? |

---

## Part E: Infrastructure Requirements

### Power Density Planning

**Questions**
- How do you plan for increasing density?
- What infrastructure upgrades are needed?
- How do you handle mixed densities?

| Density Tier | kW/Rack | Infrastructure Requirements |
|---|---:|---|
| Low density | ? | ? |
| Medium density | ? | ? |
| High density | ? | ? |
| Ultra-high density | ? | ? |

### Power distribution architecture

| Component | Function | Sizing Consideration |
|---|---|---|
| Utility feed | ? | ? |
| Main switchgear | ? | ? |
| UPS systems | ? | ? |
| PDUs | ? | ? |
| RPPs (Remote Power Panels) | ? | ? |

### Cooling Capacity Planning

| Cooling capacity Unit | Conversion | Context |
|---|---|---|
| 1 ton of cooling | ? BTU/hr | ? kW |
| 1 kW IT load (air) | ? tons | Includes overhead |
| 1 kW IT load (DLC) | ? tons | Direct rejection |

**Water requirements for liquid cooling**

| Cooling Method | Water Usage | GPM per MW |
|---|---|---:|
| Evaporative (cooling tower) | ? | ? |
| Dry cooler | ? | ? |
| DLC (closed loop) | ? | ? |

### Backup Power Systems

| UPS Type | Efficiency | Runtime | Best For |
|---|---:|---:|---|
| Double conversion | ? | ? | ? |
| Line interactive | ? | ? | ? |
| Rotary UPS | ? | ? | ? |
| Battery + flywheel | ? | ? | ? |

**Generator requirements**

| Facility Size | Generator Capacity | Fuel Storage | Startup Time |
|---:|---|---|---|
| 10 MW | ? | ? | ? |
| 100 MW | ? | ? | ? |
| 500 MW | ? | ? | ? |
| 1 GW | ? | ? | ? |

### Grid Connection for GW-Scale Facilities

| Scale | Grid Requirements | Typical Lead Time |
|---:|---|---|
| 50 MW | ? | ? |
| 200 MW | ? | ? |
| 500 MW | ? | ? |
| 1 GW+ | ? | ? |

### Power sourcing strategies

| Strategy | Description | Pros | Cons |
|---|---|---|---|
| Grid connection | ? | ? | ? |
| On-site generation | ? | ? | ? |
| PPA (Power Purchase Agreement) | ? | ? | ? |
| Behind-the-meter solar/wind | ? | ? | ? |
| Nuclear (SMR) | ? | ? | ? |

---

## Part F: Deep Dive Topics

### Chip-Level Power Management

**Dynamic Voltage and Frequency Scaling (DVFS)**
- How does DVFS work?
- What is the power/frequency relationship?

| Power State | Voltage | Frequency | Power | Use Case |
|---|---:|---:|---:|---|
| Max boost | ? | ? | ? | Peak compute |
| Base clock | ? | ? | ? | Sustained |
| Idle | ? | ? | ? | Low utilization |
| Sleep | ? | ? | ? | Inactive |

### Thermal Throttling Behavior

| Threshold | Temperature | Action |
|---|---|---|
| Target | ~83°C | ? |
| Throttle start | ~85°C | ? |
| Max operating | ~90°C | ? |
| Shutdown | ~95°C | ? |

### Heat Sink and Cold Plate Design

| Parameter | Impact | Tradeoff |
|---|---|---|
| Fin density | ? | ? |
| Base thickness | ? | ? |
| Heat pipe count | ? | ? |
| Material (Cu vs Al) | ? | ? |

| Design Aspect | Consideration | Best Practice |
|---|---|---|
| Contact area | ? | ? |
| Channel design | ? | ? |
| Flow rate | ? | ? |
| Pressure drop | ? | ? |

### Stranded Power in Datacenters

**Questions**
- What is stranded power?
- Why does it occur?
- How much power is typically stranded?
- How do you minimize stranded capacity?

---

## Part G: Companies & Industry Landscape

### System Vendors

| Company | Products | Cooling Approach | Market Position |
|---|---|---|---|
| NVIDIA | DGX, MGX, HGX | Air + DLC ready | ? |
| Dell | PowerEdge XE | ? | ? |
| HPE | Cray EX | ? | ? |
| Supermicro | GPU servers | ? | ? |
| Lenovo | ThinkSystem | ? | ? |

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
