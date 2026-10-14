# SKY130 CMOS Inverter: SPICE Simulation, Layout & Parasitic Extraction

> Module 3 VLSI Physical Design Laboratory using the SkyWater **SKY130** open-source PDK.

## Table of Contents
- [Aim](#aim)
- [Project Overview](#project-overview)
- [Objectives](#objectives)
- [Prerequisites](#prerequisites)
- [Part 1: SKY130_03_SK1, CMOS Inverter SPICE Labs](#part-1--sky130_03_sk1-cmos-inverter-spice-labs)
- [Part 2: SKY130_03_SK2, Inception of Layout](#part-2--sky130_03_sk2-inception-of-layout)
- [Part 3: SKY130_03_SK3, Technology File Labs](#part-3--sky130_03_sk3-technology-file-labs)
- [Design Flow Summary](#design-flow-summary)
- [Results](#results)
- [Conclusion](#conclusion)
- [Repository Structure](#repository-structure)
- [Tools Used](#tools-used)
- [References](#references)

---

## Aim
To design and analyze a CMOS inverter in the SkyWater **SKY130** process: run SPICE simulations and switching analysis, build and verify the physical layout in **Magic**, and extract a netlist (with parasitics) from the layout to cross-check it against the original schematic-level SPICE deck.

## Project Overview
This repository documents the **Module 3 SKY130 VLSI laboratory flow**, split into three linked sub-projects:

| Sub-project | Focus |
|---|---|
| `SKY130_03_SK1` | CMOS inverter theory and SPICE simulation labs |
| `SKY130_03_SK2` | Inception and construction of the physical layout |
| `SKY130_03_SK3` | SKY130 technology-file labs, Magic verification and extraction |

## Objectives
- Understand the operation of a CMOS inverter.
- Write a transistor-level SPICE netlist using SKY130 device models.
- Simulate with **ngspice** and obtain the voltage transfer characteristic (VTC).
- Determine the switching threshold, **Vm**.
- Measure propagation delay, rise time and fall time.
- Understand SKY130 layout layers and build an inverter layout in **Magic**.
- Run DRC and fix violations (including `poly.9`).
- Extract a SPICE netlist with parasitics from the layout and compare it with the original deck.

## Prerequisites
- SKY130 PDK installed (e.g. via `open_pdks`), with `PDK_ROOT` set
- `ngspice`, `magic` (with Tcl/Tk), `vim` or any text editor

---

## Part 1: SKY130_03_SK1, CMOS Inverter SPICE Labs

### 1.1 Pre-Lab Concepts
- A CMOS inverter is one PMOS (pull-up) and one NMOS (pull-down) transistor.
- Both gates are tied to the input; both drains are tied to the output.
- PMOS source goes to `VDD`, NMOS source goes to `GND`.
- Low input gives high output; high input gives low output.
- The **VTC** plots `Vout` against `Vin`.

### 1.2 SPICE Deck Creation
A SPICE deck is a text file describing devices, supplies, stimulus and analyses.

```bash
vim sky130inv.spice
```

Deck used in this lab (adjust the `.lib` path to your PDK install):

```spice
* SKY130 CMOS inverter
.lib "sky130_fd_pr/models/sky130.lib.spice" tt
.param temp=27

XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1.0  l=0.15
XM2 out in 0   0   sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50f

Vdd vdd 0 1.8
Vin in 0 PULSE(0 1.8 0 0.1n 0.1n 2n 4n)

.control
  * Static: VTC and switching threshold
  dc Vin 0 1.8 0.01
  meas dc vm WHEN v(out)=v(in)
  plot v(out) vs v(in)

  * Dynamic: delay, rise and fall time
  tran 10p 8n
  meas tran tpHL TRIG v(in)  VAL=0.9  RISE=1 TARG v(out) VAL=0.9  FALL=1
  meas tran tpLH TRIG v(in)  VAL=0.9  FALL=1 TARG v(out) VAL=0.9  RISE=1
  meas tran trise TRIG v(out) VAL=0.18 RISE=1 TARG v(out) VAL=1.62 RISE=1
  meas tran tfall TRIG v(out) VAL=1.62 FALL=1 TARG v(out) VAL=0.18 FALL=1
  plot v(in) v(out)
.endc
.end
```

<p align="center">
  <img width="800" alt="SPICE deck in editor (1)" src="https://github.com/user-attachments/assets/e2c0dd78-4dbb-4aa3-ab12-0e8afeabe7f5" /><br/>
  <img width="800" alt="SPICE deck in editor (2)" src="https://github.com/user-attachments/assets/2aacfcea-eebe-4405-bff0-8b9efdc45534" />
</p>

**Figure 1.** Creation of the SKY130 CMOS inverter SPICE deck.

### 1.3 SPICE Simulation
```bash
ngspice sky130inv.spice
```

<p align="center">
  <img width="800" alt="Inverter simulation result" src="https://github.com/user-attachments/assets/c2de5c00-f210-4bf1-bc79-87458e55ca43" />
</p>

**Figure 2.** SKY130 CMOS inverter simulation result.

### 1.4 Switching Threshold (Vm)
**Vm** is the input voltage at which `Vout = Vin` on the VTC. It depends on the PMOS/NMOS width ratio, supply voltage and process corner. The `meas dc vm` line in the deck reports it directly.

| Parameter | Value |
|---|---|
| `VDD` | *fill from your run* |
| `Vm`  | *fill from your run* |

### 1.5 Static and Dynamic Simulation
- **Static (`dc`)**: VTC, switching threshold, logic-high and logic-low regions.
- **Dynamic (`tran`)**: propagation delay, rise time, fall time, output waveform shape.

---

## Part 2: SKY130_03_SK2, Inception of Layout

### 2.1 Active Regions
The active (diffusion) region is where source and drain form. NMOS and PMOS each have their own active region.

<p align="center">
  <img width="800" alt="Active regions (1)" src="https://github.com/user-attachments/assets/e1138343-f301-4ee7-8cfd-73e4611f257f" /><br/>
  <img width="800" alt="Active regions (2)" src="https://github.com/user-attachments/assets/0552de96-47d1-4ef2-be90-4323c7c7dc7f" />
</p>

**Figure 3.** Active regions in the inverter layout.

### 2.2 N-Well and P-Well Formation
- **N-well** hosts the PMOS devices.
- **P-well / p-substrate** hosts the NMOS devices (SKY130 uses a p-type substrate).

### 2.3 Gate Terminal Formation
```
Polysilicon crossing an active region → MOS transistor gate
```

<p align="center">
  <img width="800" alt="Layout node and connection selection" src="https://github.com/user-attachments/assets/7e509893-e0ea-4169-b56d-e1df54117d63" />
</p>

**Figure 4.** Layout node/connection selection during inverter layout formation.

### 2.4 Lightly Doped Drain (LDD)
A lightly doped region near the drain edge that reduces the peak electric field and improves long-term reliability (hot-carrier robustness).

### 2.5 Source-Drain Formation

| Terminal | Connection |
|---|---|
| PMOS source | `VDD` |
| NMOS source | `GND` |
| PMOS and NMOS drains | Output node |

<p align="center">
  <img width="800" alt="NMOS GND connection" src="https://github.com/user-attachments/assets/836e7c24-6352-4b4f-8163-d6c4bcc5cf7a" />
</p>

**Figure 5.** NMOS GND connection in the inverter layout.

### 2.6 Local Interconnect
Local interconnect (`li1` in SKY130) links nearby device terminals, reducing the need for longer metal routes.

### 2.7 Higher-Level Metal Routing
```
Lower layers  (li1, met1)  → local and device-level connections, cell power rails
Higher layers (met2, ...)  → longer signal and power connections
```
Contacts and vias (`licon`, `mcon`, `via`) join the layers.

### 2.8 SKY130 PDK Basics
The PDK provides device models, layer definitions, design rules, technology files and standard-cell data, so the design targets a real manufacturing process. The corner libraries below were inspected during the lab.

<p align="center">
  <img width="700" alt="Slow corner library" src="https://github.com/user-attachments/assets/f6a4482e-e761-48e9-a2e4-36c3ce7f6885" />
  <img width="700" alt="Typical corner library" src="https://github.com/user-attachments/assets/89967293-0a9a-4055-9027-c747b2f83835" /><br/>
  <img width="700" alt="Fast corner library" src="https://github.com/user-attachments/assets/36a81246-f896-47a1-9880-6a163bae2e42" />
  <img width="700" alt="Pshort library" src="https://github.com/user-attachments/assets/cde833e3-4a11-4293-a7d0-53460b8cc7e6" />
</p>

**Figure 6.** SKY130 library corner files.

### 2.9 Standard-Cell Layout
A standard cell has fixed height, defined power rails and fixed pin positions. An inverter cell contains PMOS and NMOS devices, `VDD` and `GND` rails, input and output pins, contacts, and the required metal layers.

---

## Part 3: SKY130_03_SK3, Technology File Labs

### 3.1 Final SPICE Netlist
The finalized deck, run again with ngspice:

```bash
ngspice sky130inv.spice
```

<p align="center">
  <img width="800" alt="Final inverter SPICE deck" src="https://github.com/user-attachments/assets/fe4751c5-4f22-4125-81c1-df5010285a52" />
</p>

**Figure 7.** Modified/final SKY130 CMOS inverter SPICE deck.

### 3.2 Inverter Characterization
Measured parameters: switching threshold, propagation delay (`tpHL`, `tpLH`), rise time, fall time and power-related behavior. All values come from the `.meas` results in your own runs (see [Results](#results)).

### 3.3 Introduction to Magic
Magic is used to view and edit layout, run DRC, inspect nodes and extract netlists.

```bash
magic -T sky130A.tech inverter.mag
# or, with the PDK startup script:
magic -d XR -rcfile $PDK_ROOT/sky130A/libs.tech/magic/sky130A.magicrc inverter.mag
```

### 3.4 SKY130 PDK Introduction
The tools rely on the PDK for layer information, device models, design rules and physical-design data.

### 3.5 Magic to SPICE Extraction Flow

```
Layout → DRC / inspection → Extraction → SPICE netlist → SPICE simulation
```

In the Magic console (`tkcon`):

```tcl
extract all
ext2spice cthresh 0 rthresh 0
ext2spice
```

`cthresh 0 rthresh 0` keeps all parasitic capacitances and resistances in the extracted `inverter.spice`. Simulate it with ngspice and compare the Vm and delay figures with the original deck.

### 3.6 Fixing the `poly.9` DRC Error
`poly.9` is a poly spacing rule. Read the exact message Magic prints for your layout, since it names the rule and the offending geometry.

1. Open the layout in Magic.
2. Run DRC (`drc check`, then `drc why` with the error selected).
3. Locate the highlighted error region.
4. Read the rule-violation message.
5. Edit the geometry (or, where the lab calls for it, the tech-file rule) to satisfy the rule.
6. Re-run DRC and repeat until clean.

| Item | Your entry |
|---|---|
| Rule message from `drc why` | *fill in* |
| Geometry/rule changed | *fill in* |
| DRC count after fix | *fill in* |

> Fix the violation. Do not hide it by deleting the offending layer.

### 3.7 Synthesis, Floorplan and GRIDBOX Statistics
These screenshots come from the OpenLane-side flow (synthesis, floorplan, statistics) run alongside the layout labs.

<p align="center">
  <img width="800" alt="Gridboxes" src="https://github.com/user-attachments/assets/17bfe2ce-a03b-4102-b249-d32aa7df8f58" />
</p>

**Figure 8.** GRIDBOX view.

**Synthesis**
<p align="center">
  <img width="800" alt="Run synthesis" src="https://github.com/user-attachments/assets/7676540d-8b7a-4d7d-a6a1-b8d8c1a948f3" />
</p>

**Floorplan**
<p align="center">
  <img width="800" alt="Floorplan" src="https://github.com/user-attachments/assets/683a604f-64e7-45a2-be2c-cdb05518f60e" />
</p>

**Statistics**
<p align="center">
  <img width="800" alt="Statistics" src="https://github.com/user-attachments/assets/95cd6e28-52cc-46c8-9954-f3d4e05b55cd" />
</p>

---

## Design Flow Summary

```
CMOS Inverter → SPICE Netlist → ngspice (VTC, Vm, timing)
      → SKY130 Layout → Magic DRC → SPICE Extraction → Layout vs. schematic-level comparison
```

## Results

Populate from your own simulation and extraction runs only.

| Metric | Pre-layout deck | Post-extraction netlist |
|---|---|---|
| Supply voltage (`VDD`) | *fill in* | *fill in* |
| Switching threshold (`Vm`) | *fill in* | *fill in* |
| Propagation delay (`tpHL` / `tpLH`) | *fill in* | *fill in* |
| Rise time | *fill in* | *fill in* |
| Fall time | *fill in* | *fill in* |
| DRC status (after fix) | n/a | *fill in* |

## Conclusion
This project covers the full small-scale design flow for a SKY130 CMOS inverter: SPICE simulation, switching-threshold analysis, static and dynamic characterization, layout construction in Magic, DRC correction (including `poly.9`), and parasitic extraction from the finished layout. The screenshots in this repository document each step.

## Repository Structure
```
.
├── SKY130_03_SK1/   # SPICE deck, ngspice scripts, VTC results
├── SKY130_03_SK2/   # Layout inception files (Magic .mag, screenshots)
├── SKY130_03_SK3/   # Tech-file labs, DRC fixes, extracted netlists
├── images/          # Simulation/layout screenshots
└── README.md
```

## Tools Used
- **SkyWater SKY130** open-source PDK
- **ngspice**: SPICE simulation
- **Magic VLSI**: layout, DRC and extraction
- **OpenLane**: synthesis and floorplan screenshots (Section 3.7)
- **Vim**: deck editing

## References
- SkyWater SKY130 PDK documentation: https://skywater-pdk.readthedocs.io
- ngspice user manual: https://ngspice.sourceforge.io/docs.html
- Magic VLSI: http://opencircuitdesign.com/magic/
