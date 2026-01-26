---
marp: true
theme: academic
math: mathjax
paginate: true
---

<!-- _class: lead -->

# AXE 6: Puissance & Refroidissement

Charly Saugey
Paul Haardt
Corentin Colmel

Majeure Image / Epita
GPU distribué 2026

---

<!-- header: "AXE 6: Puissance & Refroidissement" -->

## Mission
- Expliquer les enjeux de puissance et de refroidissmenet des infrastructure IA moderne
- Comparer les technologie de refroidissement et analyser les astuces de design des datacenters

---

## Partie A : Evolution de la puissance des système IA

---

<!-- header: "Partie A : Evolution de la puissance des système IA" -->

### Evolution de la puissance des GPU

**Questions clés**
- Comment la consommation électrique des GPU a-t-elle évolué ?
- Qu'est-ce qui provoque l’augmentation de la puissance ?
- Existe-t-il une limite maximale à la puissance des GPU ?
- Comment la puissance évolue-t-elle avec les performances ?

---

<style scoped>
  section {
    font-size: 2.6em;
    padding: 0;
    padding-top: 100px;
  }
</style>
| GPU | Year | TDP (W) | FP16 TFLOPs | W/TFLOP | Process Node |
|---|---:|---:|---:|---:|---|
| V100 | 2017 | 300 | 125 | 2.40 | TSMC 12nm FFN |
| A100 | 2020 | 400 | 624 | 0.64 | TSMC 7nm |
| H100 SXM | 2022 | 700 | 1979 | 0.35 | TSMC 4N |
| H200 SXM | 2024 | 700 | 1979 | 0.35 | TSMC 4N |
| B200 SXM | 2024 | 1000 | 4500 | 0.22 | TSMC 4NP |
| B300 | 2025 | 1100 | 5000 | 0.22 | TSMC 4NP |
| R100 (Rubin) | 2026 | _~2300_ | _~4000_ | _~0.58_ | _TSMC N3_ |

---
<style scoped>
  section {
    font-size: 2.7em;
    padding: 0;
    padding-top: 100px;
  }
</style>
**Puissance au niveau système (nœud complet)**

| System | GPUs | GPU Power | Total System Power | Year |
|---|---|---:|---:|---:|
| DGX-1 | 8× V100 | 2.4 kW | ~3.5 kW | 2017 |
| DGX A100 | 8× A100 | 3.2 kW | ~6.5 kW | 2020 |
| DGX H100 | 8× H100 SXM | 5.6 kW | ~10.2 kW | 2022 |
| DGX B200 | 8× B200 SXM | 8.0 kW | ~14.3 kW | 2024 |
| GB200 NVL72 | 72× B200 | 72.0 kW | ~120 kW | 2024 |

---
<style scoped>
  section {
    font-size: 2.7em;
    padding: 0;
    padding-top: 100px;
  }
</style>
**Évolution de la densité de puissance des racks**

| Era | Typical Rack Power | kW/rack | Cooling Method |
|---|---:|---:|---|
| Traditional IT (2010) | 5–10 kW/rack | ~8 | Air |
| GPU compute (2018) | 15–25 kW/rack | ~20 | Air |
| AI training (2022) | 30–50 kW/rack | ~40 | Air / DLC |
| AI training (2024) | 60–120 kW/rack | ~80 | DLC required |
| AI training (2026) | 150–300 kW/rack | ~200 | DLC / Immersion |

---

![w:1050 center](./gpu-consumtion-evolution.webp)

---

<!-- header: "AXE 6: Puissance & Refroidissement" -->
## Partie B: Distribution de puissance aux GPU

---

<!-- header: "Partie B: Distribution de puissance aux GPU" -->
### Comment fonctionne l’alimentation électrique d’un GPU ?

**Questions clés**
- Du réseau électrique à la puce : quelles sont les étapes de conversion ?
- Où se situent les pertes d’efficacité ?
- Comment la puissance est-elle distribuée à l’intérieur d’un GPU ?

![center](power-delivery.png)

---

<style scoped>
  section {
    font-size: 2.5em;
    padding: 0;
    padding-top: 100px;
  }
</style>
| Stage | Input | Output | Typical Efficiency | Loss |
|---|---|---|---:|---:|
| Utility transformer | HV AC (10–30 kV) | LV AC (400–480 V) | 98–99% | 1–2% |
| UPS | AC | AC (conditioned) | 94–97% | 3–6% |
| PDU | AC | AC | 98–99% | 1–2% |
| PSU (AC-DC) | AC (200–240 V) | DC (12 V / 48 V) | 92–96% | 4–8% |
| VRM (DC-DC) | DC (12 V / 48 V) | DC (~0.7–1.1 V) | 85–95% | 5–15% |

---

### Modules régulateurs de tension (VRM)

**Questions**
- Qu'est-ce que VRM et pourquoi est-ce un élément critique ?
<!-- Le VRM est chargé de convertir le 12/48 V en tension < 1V pour les cors. Il est critique dans le sens où il exige une tension d'entrée stable et sans bruit. -->
- Qu'est-ce que les phases et comment fonctionnent-elles ?
<!-- Une phase = controleur + MOSFETs + inductance
Les phases fonctionnent en parallèle pour partager le courant et réduire l'ondulation. -->
- Pourquoi le rendement d'un VRm est important ?
<!-- Si le GPU consomme 1 kW d'éléctricité et possède un rendement de 90%, il dissipe donc 10% soit 100 W en chaleur qu'il faut évacuer. -->
- Quels sont les défis thermiques ?
<!-- Plusieurs défit :
 - Réduire la perte de puissance d'entrée sous forme de chaleur
 - Limiter l'impact de la chaleur sur les performance
 - Evacuer la chaleur le plus efficacement possible -->

---

| VRM Aspect | Description | Typical Value |
|---|---|---|
| Input voltage | Voltage supplied to VRM | 12 V or 48 V |
| Output voltage | Core supply voltage (Vcore) | 0.7–1.1 V |
| Phase count | Parallel VRM phases | 10–20 phases |
| Efficiency | DC-DC conversion efficiency | 85–95% |
| Power loss (1kW GPU) | Heat dissipated by VRM | 50–150 W |

---

### Distribution de puissance 12V vs 48V

- En quoi le 48V est meilleur pour les systèmes de haute puissance?
<!-- Parce qu'avec une tension 4 x plus élevée on diminue l'intensité par 4 pour avoir la même puissance tout en réduisant les pertes sous formes de chaleurs. -->
- Qui promeut le 48V?
<!-- Un peu tout le monde qui gravite autour des GPU -->
- Quels sont les défis?
<!-- Les VRM sont plus complexes, le risques de pont éléctrique entre deux composant unltra-proche est plus élevé, il faut assurer la compatibilité sur les appareils supports... -->

---

| Aspect | 12V Distribution | 48V Distribution |
|---|---|---|
| Intensité pour 1kW | ~83 A | ~21 A |
| Epaisseur du câble | Thick (high current) | Thinner |
| Pertes par effet Joule | High (×16 vs 48V) | Much lower |
| Taille du connecteur | Large / multiple pins | Smaller / fewer pins |
| Adoption industrielle | Legacy servers | AI & hyperscale |

---

### Considérations sur les connecteurs

| Connector | Max Power | Pins | Issues |
|---|---:|---:|---|
| 8-pin PCIe | 150 W | 8 | Limited power, multiple cables needed |
| 12VHPWR | 600 W | 16 (12+4 sense) | Overheating, insertion sensitivity |
| 12V-2x6 | 600 W | 16 (12+4 sense) | Improved safety, still high current |

---

### 800V DC pour les usines IA

**Questions**
- Pourquoi NVIDIA promeut-t-il 800V DC?
- Quels sont les avantages en termes d'efficacité?
- Comment cela change-t-il la conception des datacenter?
- Quels sont les considérations de sécurité?
<!-- Pour réduire l'intensité qui circule dans les racks de très haute puissance et réduire par conséquent les pertes par effet Joule.
Il y aurait moins d'étage de conversion donc moins de source de perte et meilleur acheminement du courant le long des racks.

DC haute tension = arcs persistants, exigences accrues d’isolation, de coupure, de procédures et de conformité (personnel formé). -->

---

<!-- header: "AXE 6: Puissance & Refroidissement" -->
## Partie C: Technologies de refroidissement

---

<!-- header: "Partie C: Technologies de refroidissement" -->
### Refroidissement par air

**Questions**
- Comment fonctionne le refroidissement par air?
- Quels sont les principes de conception d'un heat sink?
- Quels sont les limites?
- Quand le refroidissement par air échoue-t-il?
- A quel TDP l'air devient inutil?
- Quels sont les limites acoustiques?
- Comment l'altitude affecte-t-elle le refroidissement par air?
<!-- Par convection, l'élément chaud rechauffe l'air autour de lui, avec un ventilateur ou autre, l'air chaud est chassé, réablissant le déséquilibre de température. Le second principe de thermodynamique prend le relai en refroidissant l'élément chaud et en rechauffant l'air et ainsi de suite. -->
<!-- Le but est d'optimisé la surface d'échange thermique entre l'élément chaud et l'air -->
<!-- Le refroissiement par convection a rendement plus faible que par conduction. -->
<!-- Si l'air ambiant est chaud, celui-ci peut ne plus absorber assez de chaleur pour refroidir efficacement l'élément chaud -->
<!-- ~750 W -->
<!-- ~75dB -->
<!-- altitude = air moins dense = refroidissmeent moins efficace pour un même volume d'air = débit plus élévé nécessaire -->

---

| Air Cooling Aspect | Typical Value | Limit |
|---|---:|---:|
| Max heat dissipation per GPU | 400–600 W | ~700–800 W |
| Max rack density | 20–40 kW | ~50 kW |
| Airflow per rack | 85–170 $m^3m^{-1}$ | ~225 $m^3m^{-1}$  |
| Fan power overhead | 5–10% | ~15% |

---

### Direct Liquid Cooling (DLC)

**Comment fonctionne le DLC ?**
- Conception et fixation des cold plates
- Manifolds et systèmes de distribution
- Types de fluides et débits
- Rejet de chaleur (CDU, dry coolers, tours de refroidissement)
<!-- Plaque métallique en contact direct avec GPU, chaleur transférée au fluide -->
<!-- réseaux de tuyauterie qui distribuent le fluide vers chaque cold plate -->
<!-- eau/glycol ou fluides diélectriques, débit optimisé pour ΔT cible -->
<!-- CDU (unités de distribution de fluide) + échangeurs externes (dry coolers ou tours) -->

---

<style scoped>
  section {
    font-size: 2.5em;
    padding: 0;
    padding-top: 100px;
  }
</style>
| DLC Component | Function | Key Specifications |
|---|---|---|
| Cold plate | Transfert de chaleur du GPU vers le liquide | Surface alu/cuivre, low ΔT |
| Manifold | Distribution du fluide aux cold plates | Multi-port, équilibrage de débit |
| Quick disconnects | Connexions sans fuite pour maintenance | ≤ 0.5% fuite, haute pression |
| CDU (Coolant Distribution Unit) | Pompe + échangeur + filtration | Débit 10–100 L/min, filtres |
| Facility water loop | Rejet de chaleur vers le bâtiment | Dry coolers ou cooling towers |

---

| Aspect | Air Cooling | Direct Liquid Cooling |
|---|---|---|
| Max GPU TDP supported | ~700–800 W | ~1500+ W |
| Max rack density | ~40–50 kW | ~100–200+ kW |
| PUE impact | Higher overhead | Lower overhead |
| Maintenance complexity | Low | Medium/High |
| Capital cost | Lower | Higher (install + CDU) |
| Operating cost | Higher fan energy | Lower pumping energy |

---
<style scoped>
  section {
    font-size: 2.4em;
    padding: 0;
    padding-top: 100px;
  }
</style>

| Coolant | Thermal Properties | Cost | Safety | Use Case |
|---|---|---|---|---|
| Water/glycol | Very high heat capacity, high thermal conductivity | Low | Conductive, leak risk | Cold plates, DLC loops |
| Propylene glycol | Slightly lower than water, freeze protection | Low–Medium | Less toxic than ethylene glycol | Cold climates, DLC |
| Dielectric fluids | Lower than water, electrically insulating | High | Non-conductive, safer for electronics | Immersion cooling |

---

### Immersion Cooling

**Sujets**
- Monophasé vs diphasé  
- Conception des bacs et gestion des fluides  
- Méthodes de rejet thermique  
- Considérations de maintenance

---

| Aspect | Single-Phase Immersion | Two-Phase Immersion |
|---|---|---|
| Fluid type | Dielectric oil | Low-boiling dielectric fluid |
| Operating principle | Liquid absorbs heat, stays liquid | Liquid boils, vapor condenses |
| Max heat flux | High | Very high |
| Fluid cost | Medium | High |
| Complexity | Medium | High |
| Maturity | Commercially mature | Emerging / niche |

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
- Qu'est-ce que RDHx ?  
- Comment est-ce que cela aide le refroidissement à air ?  
- Quelles puissances peut-il supporter ?  
- Quand est-ce le bon choix ?  
<!-- Un échangeur thermique air-liquide monté à l’arrière d’un rack qui capte l’air chaud sortant et en extrait la chaleur via un circuit d’eau. -->
<!-- Il refroidit l’air directement au niveau du rack, réduisant la charge thermique de la salle et améliorant l’efficacité du refroidissement à air existant. -->
<!-- Environ 20 000 à 80 000 W par rack selon le type (passif ou actif). -->
<!-- Lorsqu’un datacenter refroidi à l’air doit supporter des racks plus denses sans passer immédiatement au liquid cooling direct ou à l’immersion. -->

---

| RDHx Type | Cooling Capacity | Best For |
|---|---:|---|
| Passive RDHx | ~20 000–35 000 W par rack | Racks densité moyenne, retrofit |
| Active RDHx | ~30 000–80 000 W par rack | Racks haute densité (GPU) |

---

<!-- header: "AXE 6 : Puissance & Refroidissement" -->
## Partie D: Rendement énergétique (PUE)

---

<!-- header: "Partie D: Rendement énergétique (PUE)" -->
### Comprendre le PUE

**Définition**
- PUE = Power Usage Effectiveness
- PUE = Total Facility Power / IT Equipment Power  
- 1 PUE = (IT Load + Cooling + Power Distribution + Lighting + Other) / IT Load

---

**Questions**
- Que mesure le PUE et que ne mesure-t-il pas ?
- Comment le choix du refroidissement affecte-t-il le PUE ?
- Quels sont les composants du surcoût énergétique ?
<!-- PUE mesure l’efficacité énergétique de l’infrastructure du datacenter mais ne mesure pas l’efficacité des applications, des serveurs ou de l’usage des ressources IT. -->
<!-- Refroidissement, pertes de distribution électrique, UPS, éclairage, sécurité, auxiliaires. -->
<!-- Les solutions liquid cooling réduisent fortement l’énergie dédiée au refroidissement, ce qui diminue le PUE. -->

---

| PUE Component | Typical % of Overhead | Reduction Strategies |
|---|---:|---|
| Cooling | 40–60 % | DLC, free cooling, confinement |
| Power distribution losses | 10–15 % | Haut rendement UPS/PSU |
| Lighting and other | 5 % | LED, automatisation |

---

**PUE benchmarks**

| Datacenter Type | Typical PUE | Best-in-Class PUE |
|---|---:|---:|
| Legacy enterprise | 1.8–2.5 | 1.6 |
| Modern enterprise | 1.4–1.6 | 1.3 |
| Hyperscale (air) | 1.2–1.4 | 1.1 |
| Hyperscale (DLC) | 1.1–1.2 | 1.05 |
| AI-optimized | 1.1–1.3 | 1.05 |

---

| Cooling Method | Typical PUE | Why |
|---|---:|---|
| Traditional air (CRAC) | ~1.6 | Compresseurs énergivores |
| Hot/cold aisle containment | ~1.4 | Airflow optimisé |
| Free air cooling | ~1.2 | Peu de refroidissement actif |
| Direct liquid cooling | ~1.1 | Transfert thermique direct |
| Immersion cooling | ~1.05 | Quasi suppression HVAC |

---

**Exemples de référence (best-in-class)**
<!-- Best-in-class = Les exemples optimaux pour les autres entreprises -->

| Company | Facility | PUE | How Achieved |
|---|---|---:|---|
| Google | Hyperscale DC | 1.10 | Free cooling + AI control |
| Meta | Hyperscale DC | 1.09 | Free cooling |
| Microsoft | Azure DC | 1.12 | Free air + DLC |
| NVIDIA DGX Cloud | AI DC | ~1.1 | Liquid cooling |

---

**Au-delà du PUE : autres métriques d’efficacité**

| Metric | Definition | Typical Values | Best-in-Class |
|---|---|---|---|
| PUE | Ratio total/IT | 1.1–1.6 | ~1.05 |
| WUE | L d’eau / kWh IT | 0.2–1.8 | <0.2 |
| CUE | kgCO₂ / kWh IT | Variable | Near zero |

<!-- WUE = Water Usage Efficiency -->
<!-- CUE = Carbon Usage Efficiency -->

---

<!-- header: "AXE 6 : Puissances & Refroidissement" -->
## Partie E: Spécifications de l'infrastructure

---

<!-- header: "Partie E: Spécifications de l'infrastructure" -->
### Planification de la densité de puissance

**Questions**
- Comment planifier l’augmentation de densité ?
- Quelles mises à niveau d’infrastructure sont nécessaires ?
- Comment gérer des densités mixtes ?
<!-- Modular power and cooling, oversizing pathways, liquid-ready racks.
Higher-capacity busways, transformers, UPS, liquid cooling loops.
Zoning: air-cooled rows + liquid-cooled rows. -->

---

| Density Tier | kW/Rack | Infrastructure Requirements |
|---|---:|---|
| Low density | 5 000–10 000 W | Air cooling, standard power |
| Medium density | 10 000–30 000 W | Hot/cold aisle, high airflow |
| High density | 30 000–80 000 W | RDHx or DLC |
| Ultra-high density | >80 000 W | DLC or immersion |

---

### Architecture de distribution électrique

| Component | Function | Sizing Consideration |
|---|---|---|
| Utility feed | Power from grid | Peak MW + redundancy |
| Main switchgear | Distribution & protection | Fault current rating |
| UPS systems | Backup & conditioning | MW load, minutes runtime |
| PDUs | Rack-level distribution | kW per rack |
| RPPs | Branch circuits | Density zones |

---

### Planification de capacité de refroidissement

| Cooling capacity Unit | Conversion | Context |
|---|---|---|
| 1 ton of cooling | 12 000 BTU/h | 3.52 kW |
| 1 kW IT load (air) | 0.0003 ton | Includes overhead |
| 1 kW IT load (DLC) | 0.00028 ton | Direct rejection |

---

**Besoins en eau pour le refroidissement liquide**

| Cooling Method | Water Usage | m³/h per MW |
|---|---|---:|
| Evaporative (cooling tower) | High | 1.5–2 |
| Dry cooler | Low | 0.1–0.3 |
| DLC (closed loop) | Very low | ~0.05 |

---

### Systèmes d’alimentation de secours

| UPS Type | Efficiency | Runtime | Best For |
|---|---:|---:|---|
| Double conversion | 94–97% | 5–15 min | Critical loads |
| Line interactive | 96–98% | 5–10 min | Edge DC |
| Rotary UPS | 96–98% | Seconds | Large DC |
| Battery + flywheel | 95–98% | Seconds–minutes | High power |

---

**Exigences des générateurs**

| Facility Size | Generator Capacity | Fuel Storage | Startup Time |
|---:|---|---|---|
| 10 MW | 10–12 MW | Hours | <10 s |
| 100 MW | 120 MW | Hours | <10 s |
| 500 MW | 600 MW | Hours | <15 s |
| 1 GW | 1.2 GW | Hours | <15 s |

---

### Connexion réseau pour sites à l’échelle du GigaWatt

| Scale | Grid Requirements | Typical Lead Time |
|---:|---|---|
| 50 MW | Substation tie-in | 1–2 years |
| 200 MW | Dedicated substation | 2–3 years |
| 500 MW | Transmission upgrade | 3–5 years |
| 1 GW+ | New transmission lines | 5+ years |

---
<style scoped>
  section {
    font-size: 2.3em;
    padding: 0;
    padding-top: 100px;
  }
</style>
### Stratégies d’approvisionnement énergétique

| Strategy | Description | Pros | Cons |
|---|---|---|---|
| Grid connection | Utility power | Simple | Carbon intensity |
| On-site generation | Gas/diesel | Fast backup | Emissions |
| PPA | Long-term renewable contract | Green energy | Price lock |
| Behind-the-meter solar/wind | Local generation | Low carbon | Intermittent |
| Nuclear (SMR) | Small modular reactors | Stable low-carbon | Regulatory |

---

<!-- header: "AXE 6 : Puissance & Refroidissement" -->
## Partie F: Sujets approfondis

---

<!-- header: "Partie F: Sujets approfondis" -->
### Gestion de l’énergie au niveau puce

**Mise à l’échelle dynamique tension/fréquence (DVFS)**
- Comment fonctionne le DVFS ?
- Quelle est la relation puissance/fréquence ?
<!-- Le processeur ajuste dynamiquement la tension et la fréquence selon la charge et la température.
La puissance dynamique suit approximativement : P ∝ V² × f. -->

---

| Power State | Voltage | Frequency | Power | Use Case |
|---|---:|---:|---:|---|
| Max boost | Élevée | Élevée | Très élevée | Peak compute |
| Base clock | Nominale | Nominale | Élevée | Sustained |
| Idle | Basse | Basse | Faible | Low utilization |
| Sleep | Très basse | ~0 | Très faible | Inactive |

---

### Comportement du throttling thermique

| Threshold | Temperature | Action |
|---|---|---|
| Target | ~83 °C | Fréquence nominale |
| Throttle start | ~85 °C | Réduction fréquence |
| Max operating | ~90 °C | Limitation puissance |
| Shutdown | ~95 °C | Arrêt matériel |

---

### Conception des dissipateurs et cold plates

| Parameter | Impact | Tradeoff |
|---|---|---|
| Fin density | Surface d’échange | Restriction airflow |
| Base thickness | Diffusion chaleur | Masse |
| Heat pipe count | Transport chaleur | Coût |
| Material (Cu vs Al) | Conductivité | Poids / prix |

---

| Design Aspect | Consideration | Best Practice |
|---|---|---|
| Contact area | Résistance thermique | Couverture maximale |
| Channel design | Turbulence | Micro-canaux |
| Flow rate | ΔT liquide | Suffisant sans excès |
| Pressure drop | Charge pompe | Minimiser |

---

### Puissance échouée (« Stranded Power ») dans les datacenters

**Questions**
- Qu’est-ce que la puissance échouée ?
- Pourquoi apparaît-elle ?
- Quelle part de puissance est typiquement échouée ?
- Comment minimiser cette capacité perdue ? 
<!-- Puissance électrique installée mais inutilisable.
Limites de refroidissement ou déséquilibre rack-level.
10–30 % de capacité.
Zoning densité, liquid cooling, orchestration workload. -->

---

<!-- header: "AXE 6 : Puissance & Refroidissement" -->
## Partie G: Entreprises & paysage industriel

---

<!-- header: "Partie G: Entreprises & paysage industriel" -->
### Constructeurs de systèmes

| Company | Products | Cooling Approach | Market Position |
|---|---|---|---|
| NVIDIA | DGX, MGX, HGX | Air + DLC ready | Leader AI platforms |
| Dell | PowerEdge XE | Air + DLC | Enterprise AI servers |
| HPE | Cray EX | DLC | HPC & AI leader |

---

| Company | Products | Cooling Approach | Market Position |
|---|---|---|---|
| Supermicro | GPU servers | Air + DLC | Broad OEM supplier |
| Lenovo | ThinkSystem | Air + DLC | Enterprise/HPC |

---

### Fournisseurs d’infrastructure de refroidissement

| Company | Products | Technology Focus |
|---|---|---|
| Vertiv | Liebert, CDUs | Full stack cooling |
| Schneider Electric | APC, cooling | Power + thermal |
| Asetek | Cold plates, CDUs | DLC pioneer |
| CoolIT | DLC systems | Rack-level DLC |
| GRC | ICEraQ | Single-phase immersion |
| LiquidCool | Immersion tanks | Two-phase immersion |
| Submer | SmartPod | Immersion systems |

---

### Fournisseurs d’infrastructure électrique

| Company | Products | Specialty |
|---|---|---|
| Schneider Electric | UPS, PDUs, switchgear | End-to-end power |
| Vertiv | UPS, PDUs | Critical power |
| Eaton | UPS, PDUs | Power distribution |
| ABB | Transformers, switchgear | Utility-scale |
| Caterpillar | Generators | Backup power |

---

<!-- header: "AXE 6 : Puissance & Refroidissement" -->
## Partie H: Comparaison finale

---

<style scoped>
  section {
    font-size: 1.5em;
    padding: 0;
    padding-top: 100px
  }
</style>
<!-- header: "Partie H: Comparaison finale" -->

### Comparaison des technologies de refroidissement


| Aspect | Air Cooling | Rear-Door HX | Direct Liquid | Single-Phase Immersion | Two-Phase Immersion |
|---|---|---|---|---|---|
| Max kW/rack | ~20–50 kW | ~20–80 kW | ~50–200+ kW | ~80–250+ kW | ~100–300+ kW |
| PUE achievable | ~1.2–1.6 | ~1.15–1.4 | ~1.05–1.2 | ~1.03 | ~1.02 |
| Capital cost | Faible | Moyen | Élevé | Élevé | Très élevé |
| Operating cost | Moyen/Élevé | Moyen | Faible/Moyen | Faible | Faible |
| Maintenance | Simple | Moyenne | Moyenne/Complexe | Complexe | Très complexe |
| Maturity | Très mature | Mature | Mature | Mature (commercial) | Plus niche |
| GPU compatibility | Universelle | Universelle | Cold-plate requis | Matériel compatible immersion | Matériel compatible immersion |

---
<style scoped>
  section {
    font-size: 2.1em;
    padding: 0;
    padding-top: 100px
  }
</style>
### Résumé de puissance des systèmes IA

| System | GPUs | Total Power | Cooling Method | Rack Density |
|---|---|---:|---|---:|
| DGX A100 | 8× A100 | ~6.5 kW | Air | Variable (dépend du nombre de serveurs/rack) |
| DGX H100 | 8× H100 | ~10.2 kW | Air/DLC | Variable |
| DGX B200 | 8× B200 | ~14.3 kW | DLC | Variable |
| GB200 NVL72 | 72× B200 | ~120 kW | DLC | ~120 kW/rack |
| AMD MI300X (8-way) | 8× MI300X | ~12.5 kW | Air | Variable |
| Google TPU v5p pod | 8 960 TPU v5p chips | Non communiqué | DLC | Non communiqué |

---
<style scoped>
  section {
    font-size: 1.7em;
    padding: 0;
    padding-top: 100px
  }
</style>
### Comparaison d’efficacité des datacenters

| Operator | Facility Type | PUE | WUE | Cooling Method |
|---|---|---:|---:|---|
| Google | Hyperscale | 1.09 (2024) | Non communiqué (global public) | Mix (free cooling + liquid selon sites) |
| Meta | Hyperscale | 1.08 (2023 avg) | WUE communiqué (voir rapport) | Mix (optimisations air + water systems) |
| Microsoft | Azure | Non communiqué (global public) | Amélioration WUE (ordre de grandeur communiqué) | Mix (air + innovations “zéro eau” sur nouveaux sites) |
| AWS | Cloud | 1.15 (global) | Non communiqué (global public) | Mix (air + optimisations) |
| CoreWeave | AI-focused | 1.15 (site Barcelone annoncé) | “Zéro eau” annoncé (site Barcelone) | Air (free cooling) + design optimisé |
| Lambda Labs | AI-focused | Non communiqué | Non communiqué | Non communiqué |

---
<style scoped>
  section {
    font-size: 2.2em;
    padding: 0;
    padding-top: 100px
  }
</style>
### Impact du choix de refroidissement sur le TCO

| Cost Component | Air Cooling | DLC | Immersion |
|---|---:|---:|---:|
| Capital ($/kW IT) | Faible | Élevé | Élevé/Très élevé |
| Power cost ($/kW-yr) | Élevé (ventilos/HVAC) | Plus faible | Plus faible |
| Maintenance ($/kW-yr) | Faible | Moyen | Élevé |
| Floor space ($/kW-yr) | Élevé (densité limitée) | Plus faible | Plus faible |
| 5-year TCO ($/kW) | Variable (souvent ↑ à haute densité) | Souvent ↓ à haute densité | Souvent ↓ si densité extrême |

---

<!-- header: "AXE 6 : Puissance & Refroidissement" -->
## Sources

---

<!-- header: "Sources" -->
### Part A
 - https://www.nvidia.com
 - https://www.tsmc.com/english/dedicatedFoundry/technology
 - https://www.anandtech.com/  
 - https://www.servethehome.com/
 - https://www.nvidia.com/en-us/data-center/
 - https://www.servethehome.com/nvidia-gb200-nvl72-power-cooling-and-rack-scale-design/
 - https://www.anandtech.com/
---
 - https://www.se.com/ww/en/work/solutions/for-business/data-centers-and-networks/
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/
 - https://uptimeinstitute.com/resources/research-and-reports
 - https://www.nvidia.com/en-us/data-center/
 - https://www.servethehome.com/nvidia-gb200-nvl72-power-cooling-and-rack-scale-design/
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments
 - https://engineering.fb.com/

---

### Part B

 - https://www.pcisig.com/specifications/pciexpress
 - https://www.nvidia.com/en-us/geforce/news/12vhpwr-connector/
 - https://www.pcisig.com/news/pcisig-introduces-12v-2x6-connector
 - https://www.anandtech.com/show/17530/nvidia-12vhpwr-issues-and-analysis
 - https://www.opencompute.org/documents/ocp-48v-dc-power-distribution
 - https://www.nvidia.com/en-us/data-center/energy-efficiency/
 - https://www.ti.com/power-management/48v.html
---
 - https://www.analog.com/en/analog-dialogue/articles/why-48v.html
 - https://www.anandtech.com/show/17626/nvidia-hopper-h100-architecture-deep-dive
 - https://www.schneider-electric.com/en/work/solutions/for-business/data-centers-and-networks/
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/data-center-power-chain/
 - https://www.eaton.com/us/en-us/products/backup-power-ups-surge-it-power-distribution/data-center-ups-efficiency.html
 - https://www.plugloadsolutions.com/80pluspowersupplies.aspx
---
 - https://www.ti.com/power-management/vrm.html
 - https://www.intel.com/content/www/us/en/products/docs/processors/core/vrm-design-guide.html
 - https://www.ti.com/power-management/vrm.html
 - https://www.intel.com/content/www/us/en/products/docs/processors/core/vrm-design-guide.html
 - https://www.analog.com/en/analog-dialogue/articles/multiphase-buck-converters.html
 - https://www.anandtech.com/show/17626/nvidia-hopper-h100-architecture-deep-dive

---

### Part C

 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/high-density-data-center-cooling/
 - https://www.schneider-electric.com/en/work/solutions/for-business/data-centers-and-networks/
 - https://www.servethehome.com/nvidia-gb200-nvl72-power-cooling-and-rack-scale-design/
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments
---
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/data-center-liquid-cooling/
 - https://www.nvidia.com/en-us/data-center/energy-efficiency/
 - https://www.servethehome.com/nvidia-gb200-nvl72-power-cooling-and-rack-scale-design/
 - https://www.coolingbestpractices.com/knowledge_center/whitepapers/direct_liquid_cooling
 - https://www.schneider-electric.com/en/work/solutions/for-business/data-centers-and-networks/
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments  
---
 - https://www.submer.com/immersion-cooling/  
 - https://www.grcooling.com/immersion-cooling/  
 - https://www.3m.com/3M/en_US/data-center-us/solutions/liquid-cooling/  
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/data-center-liquid-cooling/
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/rear-door-heat-exchangers/  
 - https://www.schneider-electric.com/en/work/solutions/for-business/data-centers-and-networks/rear-door-heat-exchanger/ 
---
 - https://www.asetek.com/liquid-cooling/technologies/rear-door-heat-exchanger/  
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments

---

### Part D

 - https://uptimeinstitute.com/resources/what-is-pue  
 - https://www.google.com/about/datacenters/efficiency/  
 - https://engineering.fb.com/2020/03/12/data-center-engineering/data-center-efficiency/  
 - https://learn.microsoft.com/en-us/azure/sustainability/  
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments  
 - https://www.iea.org/reports/data-centres-and-data-transmission-networks

---

### Part E

 - https://uptimeinstitute.com/resources/what-is-pue  
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments  
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/data-center-power-and-cooling/  
 - https://www.schneider-electric.com/en/work/solutions/for-business/data-centers-and-networks/  
 - https://www.iea.org/reports/data-centres-and-data-transmission-networks  
 - https://www.energy.gov/ne/articles/small-modular-reactors

---

### Part F

 - https://developer.nvidia.com/blog/nvidia-hopper-architecture-in-depth/  
 - https://www.anandtech.com/show/17626/nvidia-hopper-h100-architecture-deep-dive  
 - https://www.intel.com/content/www/us/en/products/docs/processors/core/vrm-design-guide.html  
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments  
---
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/data-center-power-and-cooling/  
 - https://uptimeinstitute.com/resources

---

### Part G

 - https://www.nvidia.com/en-us/data-center/  
 - https://www.dell.com/en-us/dt/servers/poweredge-xe.htm  
 - https://www.hpe.com/us/en/compute/hpc/cray-ex.html  
 - https://www.supermicro.com/en/products/gpu  
 - https://www.lenovo.com/us/en/servers-storage/servers/thinksystem/  
 - https://www.vertiv.com/  
 - https://www.se.com/ww/en/work/solutions/for-business/data-centers-and-networks/  
---
 - https://www.asetek.com/data-center-liquid-cooling/  
 - https://www.coolitsystems.com/  
 - https://www.grcooling.com/  
 - https://www.submer.com/  
 - https://www.eaton.com/  
 - https://new.abb.com/  
 - https://www.caterpillar.com/

---

### Part H

 - https://images.nvidia.com/aem-dam/Solutions/Data-Center/nvidia-dgx-a100-datasheet.pdf
 - https://docs.nvidia.com/dgx/dgxh100-user-guide/introduction-to-dgxh100.html
 - https://www.nvidia.com/en-us/data-center/dgx-b200/
 - https://www.nvidia.com/en-us/data-center/gb200-nvl72/
 - https://www.sunbirddcim.com/blog/your-data-center-ready-nvidia-gb200-nvl72
 - https://www.dell.com/support/manuals/en-us/poweredge-xe9680/xe9680_ism_pub/safety-instructions?guid=guid-b85aa68d-c5c9-4ea1-8358-7388139bcfda&lang=en-us
---
 - https://docs.cloud.google.com/tpu/docs/v5p
 - https://cloud.google.com/blog/products/ai-machine-learning/introducing-cloud-tpu-v5p-and-ai-hypercomputer
 - https://datacenters.google/efficiency
 - https://sustainability.atmeta.com/wp-content/uploads/2024/08/Meta-2024-Sustainability-Report.pdf
 - https://aws.amazon.com/sustainability/data-centers/
 - https://www.microsoft.com/en-us/microsoft-cloud/blog/2024/12/09/sustainable-by-design-next-generation-datacenters-consume-zero-water-for-cooling/
 - https://www.edged.es/news/coreweave-partners-with-edged
---
 - https://www.vertiv.com/en-us/about/news-and-insights/articles/white-papers/rear-door-heat-exchangers/
 - https://submer.com/blog/single-phase-vs-two-phase-immersion-cooling/
 - https://www.ashrae.org/technical-resources/bookstore/thermal-guidelines-for-data-processing-environments
