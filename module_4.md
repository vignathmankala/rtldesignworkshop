<h1 align="center">SKY130 – Pre-Layout Timing Analysis</h1>
<p align="center"><b>Timing Libraries · Synthesis · OpenSTA · Clock Tree Synthesis</b></p>

<p align="center">
  <img alt="PDK" src="https://img.shields.io/badge/PDK-SkyWater%20SKY130-blue?style=flat-square">
  <img alt="Synthesis" src="https://img.shields.io/badge/Synthesis-Yosys-orange?style=flat-square">
  <img alt="Timing" src="https://img.shields.io/badge/STA-OpenSTA-brightgreen?style=flat-square">
  <img alt="Flow" src="https://img.shields.io/badge/Flow-OpenLane-informational?style=flat-square">
</p>

<p align="center"><i>A study of how timing information is prepared and analyzed before and after clock-tree implementation, using the SkyWater SKY130 PDK, Yosys, OpenSTA and OpenLane.</i></p>

---

## 📌 Aim
To understand and perform timing analysis of a digital design in **SKY130**: timing libraries and PVT corners, synthesis, ideal-clock STA with **OpenSTA**, clock tree synthesis (CTS), and real-clock setup/hold analysis.

## 📖 Project Overview
Timing is modelled with Liberty (`.lib`) libraries and delay tables, then the design is synthesized and analyzed with static timing analysis (STA). The project covers:

- Ideal-clock timing analysis
- Clock jitter and uncertainty
- Clock Tree Synthesis (CTS) and clock buffering
- Crosstalk considerations
- Real-clock (propagated) setup and hold analysis, with a pre-CTS vs post-CTS slack comparison

## 🎯 Objectives
- [x] Convert grid information into routing tracks
- [x] Study SKY130 timing libraries across PVT corners (slow / typical / fast)
- [x] Understand LEF physical information for standard cells
- [x] Configure and run RTL-to-gate-level synthesis using Yosys
- [x] Perform ideal-clock setup timing analysis using OpenSTA
- [x] Understand clock jitter and clock uncertainty
- [x] Run placement and CTS
- [x] Perform real-clock (post-CTS) setup and hold analysis
- [x] Compare pre-CTS and post-CTS slack values

## 🧰 Prerequisites & Setup

| Tool | Purpose | Check |
|---|---|---|
| [SkyWater SKY130 PDK](https://github.com/google/skywater-pdk) | Liberty (`.lib`) and LEF files | `echo $PDK_ROOT` |
| [Yosys](https://yosyshq.net/yosys/) | RTL synthesis | `yosys -V` |
| [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA) | Static timing analysis | `sta -version` |
| [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane) | Placement and CTS flow | `./flow.tcl -help` (inside the OpenLane directory/container) |

> Make sure the Liberty and LEF files for all three corners are present and referenced correctly before running synthesis or STA.

---

## Table of Contents
- [Grid-to-Track Conversion](#-grid-to-track-conversion)
- [Timing Libraries](#-timing-libraries)
- [Library Information (LEF)](#-library-information-lef)
- [Synthesis Configuration](#-synthesis-configuration)
- [Timing Analysis with Ideal Clocks](#-timing-analysis-with-ideal-clocks)
- [Clock Jitter and Uncertainty](#-clock-jitter-and-uncertainty)
- [OpenSTA Configuration](#-opensta-configuration)
- [Clock Tree Synthesis](#-clock-tree-synthesis)
- [Timing Analysis with Real Clocks](#-timing-analysis-with-real-clocks)
- [Results](#-results)
- [Learning Outcomes](#-learning-outcomes)
- [Conclusion](#-conclusion)
- [Repository Structure](#-repository-structure)
- [Tools Used](#-tools-used)
- [References](#-references)
- [License](#-license)

---

## 🔲 Grid-to-Track Conversion
Routing tracks must align with the standard-cell grid. In OpenLane, track pitch and offset per metal layer are defined in `tracks.info` (format: `<layer> <X|Y> <offset> <pitch>`), and Magic's grid is set to match so that pins land on routing tracks.

```tcl
# In Magic, set the grid to the li1 track pitch (check your tracks.info)
grid 0.46um 0.34um 0.23um 0.17um
```

<p align="center">
  <img width="800" alt="Technology grid information" src="https://github.com/user-attachments/assets/555de6e5-c7ee-46ff-8cb8-5d95053ae5ac" />
</p>
<p align="center"><sub><b>Figure 1.</b> SKY130 technology grid information used to derive routing tracks.</sub></p>

<p align="center">
  <img width="800" alt="Track information" src="https://github.com/user-attachments/assets/1c367663-b5ec-42ad-a0ef-a60a8b5940ea" />
</p>
<p align="center"><sub><b>Figure 2.</b> Routing track information from the SKY130 files.</sub></p>

---

## 📚 Timing Libraries
STA relies on **Liberty (`.lib`)** libraries characterized across process, voltage and temperature (PVT). Each cell has delay and transition tables indexed by input slew and output load.

```
Clock Source → Clock Tree → Launch Flip-Flop → Combinational Logic → Capture Flip-Flop
```

| Corner | Typical SKY130 library file | Used for |
|---|---|---|
| Slow | `sky130_fd_sc_hd__ss_100C_1v60.lib` | Setup checks (worst-case delay) |
| Typical | `sky130_fd_sc_hd__tt_025C_1v80.lib` | Nominal behavior |
| Fast | `sky130_fd_sc_hd__ff_n40C_1v95.lib` | Hold checks (best-case delay) |

> Confirm exact file names in your PDK install.

<p align="center">
  <img width="700" alt="Fast corner library" src="https://github.com/user-attachments/assets/a7c436da-908c-4824-988d-243257384763" />
</p>
<p align="center"><sub><b>Figure 3.</b> Fast-corner timing library.</sub></p>

<p align="center">
  <img width="700" alt="Typical corner library" src="https://github.com/user-attachments/assets/e8243854-03e9-4a4e-b6bd-c73dbb590657" />
</p>
<p align="center"><sub><b>Figure 4.</b> Typical-corner timing library.</sub></p>

<p align="center">
  <img width="700" alt="Slow corner library" src="https://github.com/user-attachments/assets/1e8de6d6-94f7-4855-b5c8-20368255248c" />
</p>
<p align="center"><sub><b>Figure 5.</b> Slow-corner timing library.</sub></p>

---

## 📄 Library Information (LEF)
LEF files hold **physical** cell data (dimensions, pin shapes and locations, obstructions) used in place-and-route. Liberty gives timing; LEF gives geometry.

<p align="center">
  <img width="800" alt="Standard-cell LEF" src="https://github.com/user-attachments/assets/e94b9bf4-5385-4f52-adac-ed11592c8dc4" />
</p>
<p align="center"><sub><b>Figure 6.</b> SKY130 standard-cell LEF information.</sub></p>

---

## ⚙️ Synthesis Configuration
Synthesis maps RTL onto the SKY130 standard cells. A script that maps both flip-flops and combinational logic to the library:

```tcl
# synth.ys, run with: yosys synth.ys
read_verilog rtl/<design>.v
read_liberty -lib libs/sky130_fd_sc_hd__tt_025C_1v80.lib
synth -top <top_module>
dfflibmap -liberty libs/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty libs/sky130_fd_sc_hd__tt_025C_1v80.lib
opt_clean -purge
stat -liberty libs/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr synthesis/<design>_netlist.v
```

<p align="center">
  <img width="800" alt="Synthesis run and statistics" src="https://github.com/user-attachments/assets/5a09832a-5407-48de-9e20-c7ed61b3ba6e" />
</p>
<p align="center"><sub><b>Figure 7.</b> Synthesis run and cell statistics.</sub></p>

<p align="center">
  <img width="800" alt="Post-synthesis inspection" src="https://github.com/user-attachments/assets/fa180794-3afd-4118-815e-843f44c15a31" />
</p>
<p align="center"><sub><b>Figure 8.</b> Post-synthesis design inspection.</sub></p>

---

## ⏱️ Timing Analysis with Ideal Clocks
With an **ideal clock**, the clock arrives at every flip-flop at the same instant (no tree delay, no skew).

**Setup:** data must arrive before the capture edge.

```
slack_setup = T_clk − (t_clk→Q + t_comb + t_setup) − t_uncertainty
```

**Hold:** data must not change too soon after the capture edge.

```
slack_hold = (t_clk→Q + t_comb) − t_hold − t_uncertainty
```

A negative slack is a violation.

### Example constraints (`sta/design.sdc`)
```tcl
create_clock -name clk -period 12 [get_ports clk]
set_clock_uncertainty -setup 0.25 [get_clocks clk]
set_clock_uncertainty -hold  0.10 [get_clocks clk]
set_clock_transition 0.15 [get_clocks clk]

set data_in [remove_from_collection [all_inputs] [get_ports clk]]
set_input_delay  2.0 -clock clk $data_in
set_output_delay 2.0 -clock clk [all_outputs]
set_load 0.05 [all_outputs]
```

These are example values (12 ns period, 0.25 ns setup and 0.10 ns hold uncertainty, 2 ns I/O delays). Adjust them to your design. **Re-run the flow after changing them, since all slack numbers depend on these constraints.**

---

## 🕰️ Clock Jitter and Uncertainty
- **Jitter** is cycle-to-cycle variation in clock arrival.
- **Uncertainty** is the margin STA subtracts to cover jitter and other non-idealities (skew estimates before CTS, OCV, crosstalk margin).
- Setup uncertainty tightens the setup check; hold uncertainty tightens the hold check.

## 🧮 OpenSTA Configuration
Setup (slow corner) and hold (fast corner) are checked with separate library runs.

```tcl
# sta/sta_setup.tcl, run with: sta -no_init sta/sta_setup.tcl
read_liberty libs/sky130_fd_sc_hd__ss_100C_1v60.lib
read_verilog synthesis/<design>_netlist.v
link_design <top_module>
read_sdc sta/design.sdc
report_checks -path_delay max -format full_clock_expanded -digits 3
report_worst_slack -max
```

```tcl
# sta/sta_hold.tcl
read_liberty libs/sky130_fd_sc_hd__ff_n40C_1v95.lib
read_verilog synthesis/<design>_netlist.v
link_design <top_module>
read_sdc sta/design.sdc
report_checks -path_delay min -digits 3
report_worst_slack -min
```

---

## 🌳 Clock Tree Synthesis
CTS builds a buffered tree so the clock reaches all flip-flops with low **skew** and bounded insertion delay.

### Placement
Standard cells are placed in the floorplan before CTS.

<p align="center">
  <img width="800" alt="Placement overview" src="https://github.com/user-attachments/assets/46f604af-3b1a-4f02-b5e7-7fef8421299f" />
</p>
<p align="center"><sub><b>Figure 9.</b> Standard-cell placement, overview.</sub></p>

<p align="center">
  <img width="800" alt="Placement detail" src="https://github.com/user-attachments/assets/224aaece-c890-4de4-91c7-bf5bea00f3b4" />
</p>
<p align="center"><sub><b>Figure 10.</b> Placement in greater detail.</sub></p>

<p align="center">
  <img width="800" alt="Placement enlarged" src="https://github.com/user-attachments/assets/1356f4df-794e-4cd8-9b5f-3dcf64538b64" />
</p>
<p align="center"><sub><b>Figure 11.</b> Enlarged placement view.</sub></p>

### Running CTS (OpenLane interactive mode)
```tcl
run_placement
run_cts
```

<p align="center">
  <img width="800" alt="CTS run" src="https://github.com/user-attachments/assets/da85fbaa-2032-41ff-96dd-06f3c201c679" />
</p>
<p align="center"><sub><b>Figure 12.</b> Clock Tree Synthesis run.</sub></p>

### Clock buffering
CTS inserts clock buffers/inverters from a configured buffer list. Sizing and branching control skew, transition time and clock power.

### Crosstalk considerations
Neighbouring wires couple capacitively. An aggressor switching alongside a victim can slow it (setup risk) or speed it up (hold risk). Clock nets are especially sensitive, so they are commonly given extra spacing or shielding, and crosstalk-induced delay is included in signoff STA.

---

## 🔁 Timing Analysis with Real Clocks
After CTS, the clock is **propagated** (actual tree delays and skew) instead of ideal:

```tcl
# in openroad, after CTS
read_liberty libs/sky130_fd_sc_hd__tt_025C_1v80.lib
read_db <cts_output>.odb          ;# or read_lef / read_def for older flows
read_sdc sta/design.sdc
set_propagated_clock [all_clocks]
estimate_parasitics -placement
report_checks -path_delay max
report_checks -path_delay min
```

### Pre-STA (post-CTS) checks
<p align="center">
  <img width="800" alt="Pre-STA overview" src="https://github.com/user-attachments/assets/99c9ac3f-6066-4af0-ba6b-a56db93c2f24" />
</p>
<p align="center"><sub><b>Figure 13.</b> Pre-STA report, overview.</sub></p>

<p align="center">
  <img width="800" alt="Pre-STA timing detail" src="https://github.com/user-attachments/assets/45986cfa-f1e2-49a5-a1a4-8c46883c224e" />
</p>
<p align="center"><sub><b>Figure 14.</b> Pre-STA timing report detail.</sub></p>

### Slack comparison (before vs. after CTS)
<p align="center">
  <img width="700" alt="Slack before CTS (1)" src="https://github.com/user-attachments/assets/82e86202-118e-4daf-8c31-5670b565b1b6" />
  <img width="700" alt="Slack after CTS (1)" src="https://github.com/user-attachments/assets/8dafd94a-d91a-4e56-8fba-afa0692a035a" /><br/>
  <img width="700" alt="Slack before CTS (2)" src="https://github.com/user-attachments/assets/aedb6295-0013-433d-af56-100af706b888" />
  <img width="700" alt="Slack after CTS (2)" src="https://github.com/user-attachments/assets/bcae774a-9d4f-4a34-806b-e4aad7d1c41d" />
</p>
<p align="center"><sub><b>Figure 15.</b> Setup/hold slack before (old) and after (new) CTS.</sub></p>

---

## 📊 Results
Fill from your own reports (the screenshots above are the evidence). Clock period used: `__ ns`.

| Check | Ideal clock (pre-CTS) | Propagated clock (post-CTS) |
|---|---|---|
| Worst setup slack (WNS) | *fill in* | *fill in* |
| Total negative slack (TNS) | *fill in* | *fill in* |
| Worst hold slack | *fill in* | *fill in* |
| Clock skew | n/a | *fill in* |

---

## 🎓 Learning Outcomes
- Reading SKY130 Liberty and LEF files and relating them to PVT corners.
- Running Yosys synthesis with library mapping and reading cell statistics.
- Writing SDC constraints and OpenSTA scripts for setup and hold checks.
- Distinguishing ideal-clock from propagated-clock STA.
- Understanding jitter, uncertainty, skew and crosstalk effects on slack.
- Running placement and CTS and comparing pre- and post-CTS slack.

## ✅ Conclusion
The SKY130 timing flow was carried out from libraries and synthesis through ideal-clock STA, then CTS and propagated-clock STA. Comparing slack before and after CTS shows how clock-tree skew and insertion delay shift setup and hold margins, and why uncertainty and crosstalk must be accounted for. Catching these effects early lets timing problems be fixed before final layout.

## 📁 Repository Structure
```
.
├── libs/        # SKY130 slow/typical/fast Liberty libraries + LEF
├── rtl/         # Verilog source
├── synthesis/   # Yosys scripts, logs, stat reports, netlist
├── sta/         # OpenSTA scripts, .sdc constraints, timing reports
├── cts/         # OpenLane placement + CTS outputs
├── images/      # Screenshots referenced in this README
└── README.md
```

## 🛠️ Tools Used
- **SkyWater SKY130**: PDK (Liberty + LEF)
- **Yosys**: RTL synthesis
- **OpenSTA / OpenROAD**: static timing analysis
- **OpenLane**: placement and CTS

## 📚 References
- SkyWater SKY130 PDK documentation
- OpenSTA user guide (The OpenROAD Project)
- OpenLane documentation (The OpenROAD Project)
- Yosys manual

## 📝 License
Add a license of your choice (e.g. MIT) as `LICENSE` in the repository root.

---

<p align="center"><sub>⭐ If you found this walkthrough useful, consider starring the repository.</sub></p>
