# Power Distribution Network (PDN) & Detailed Routing (TritonRoute)

> Module 5 of the RTL-to-GDSII flow: picorv32a RISC-V core, SkyWater SKY130 PDK, OpenLane interactive flow.

## Table of Contents
- [Aim](#aim)
- [Objectives](#objectives)
- [Background](#background)
- [Procedure](#procedure)
- [Results](#results)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)
- [Conclusion](#conclusion)

---

## Aim
To build the Power Distribution Network (PDN), perform detailed routing with TritonRoute, stream out GDSII, and run Design Rule Checking (DRC) in Magic for the `picorv32a` core, using the SKY130 PDK in the OpenLane flow.

## Objectives
- Generate power and ground rings, straps and standard-cell rails (`gen_pdn`) after Clock Tree Synthesis (CTS).
- Run global and detailed routing (FastRoute + TritonRoute) and check the routing and post-route timing reports.
- Stream out GDSII with Magic and run DRC on it.
- Handle the state and disk-space problems that come up in interactive runs.

## Background

**PDN.** Every standard cell needs `VDD` and `VSS`. The PDN delivers them through a hierarchy: core ring(s) around the design, vertical and horizontal **straps** on upper metals, and **rails** on `met1` that run along each cell row. Pitch and width are set in the flow configuration (the `FP_PDN_*` variables). A weak grid causes IR drop and electromigration problems, so PDN is built before routing and the router treats it as an obstruction.

**Routing.** OpenLane runs it in two steps:
1. **Global routing** assigns each net to coarse routing regions (guides).
2. **Detailed routing (TritonRoute)** places exact wire segments and vias inside those guides while obeying the SKY130 design rules. It runs in iterations and reports remaining violations after each one; the goal is zero.

---

## Procedure

### 1. Environment Setup
Launch OpenLane interactively and load the design:

```tcl
package require openlane 0.9
prep -design picorv32a -tag RUN_MOD5_CLEAN -overwrite
```

Then run the prerequisite stages in order:

```tcl
run_synthesis
run_floorplan
run_placement
run_cts
```

### 2. PDN Generation
Set the power and ground net names if your configuration doesn't already define them, then generate the grid:

```tcl
set ::env(VDD_NET) "VDD"
set ::env(GND_NET) "VSS"
gen_pdn
```

<img width="1280" height="767" alt="PDN structure generated across the floorplan" src="https://github.com/user-attachments/assets/12e3af38-b4b7-4203-bea3-7dcc21400078" />

*Fig. 1. Power distribution network generated across the floorplan.*

<img width="1280" height="767" alt="Power and ground rails connected to standard cells" src="https://github.com/user-attachments/assets/86393601-2599-45da-8b37-827c87b1f5c7" />

*Fig. 2. Power/ground rails connected to standard-cell rows.*

### 3. Detailed Routing
```tcl
run_routing
```

Check the routing logs and reports in the run folder (`logs/routing/`, `reports/routing/`) for the final TritonRoute violation count, any unrouted nets, and the post-route timing summary.

To inspect the routed design in Magic (paths depend on your run):

```bash
magic -T $PDK_ROOT/sky130A/libs.tech/magic/sky130A.tech \
  lef read tmp/merged.lef def read results/routing/picorv32a.def &
```

### 4. GDSII Streaming & DRC
```tcl
run_magic
run_magic_drc
```

`run_magic` streams the final layout to GDSII, and `run_magic_drc` runs Magic's DRC on it.

<img width="1280" height="767" alt="Streamed-out GDSII layout viewed in Magic" src="https://github.com/user-attachments/assets/8536be3d-8b58-4756-8613-1206b286a171" />

*Fig. 3. Streamed-out GDSII layout viewed in Magic.*

<img width="1280" height="767" alt="Magic DRC verification in progress" src="https://github.com/user-attachments/assets/a90eb03a-6826-4667-a7b9-ca0d53ee0554" />

*Fig. 4. DRC executing in Magic.*

<img width="1280" height="767" alt="Magic DRC results showing zero violations" src="https://github.com/user-attachments/assets/7e66377a-7178-4fbf-b815-4af99c37ad9a" />

*Fig. 5. DRC report showing zero rule violations.*

### 5. Report Generation
```tcl
calc_total_runtime
generate_final_summary_report
```

<img width="1280" height="767" alt="Final summary report output" src="https://github.com/user-attachments/assets/0276d402-3260-433c-ba7f-d62308821f01" />

*Fig. 6. Final summary report for the run.*

---

## Results

Take the numbers from your own reports; the screenshots above are the evidence.

| Metric | Outcome |
|---|---|
| PDN | Grid generated and connected to standard-cell supply pins (Figs. 1-2) |
| Detailed routing | Final TritonRoute violation count: *fill in*; unrouted nets: *fill in* |
| Post-route setup / hold slack | *fill in from the routing/STA report* |
| Magic DRC | *fill in*, e.g. 0 violations (Fig. 5) |
| Total runtime | *fill in* |
| Summary report | `/openLANE_flow/designs/picorv32a/runs/RUN_MOD5_CLEAN/reports/final_summary_report.csv` |

## Troubleshooting

**Disk full during the run.** Interactive runs write a lot of data, and the VM ran out of space here. Check usage first, then remove only old runs, not the current one:

```bash
df -h
du -sh /openLANE_flow/designs/picorv32a/runs/*
rm -rf /openLANE_flow/designs/picorv32a/runs/<old_run_tag>
```

Double-check the path before `rm -rf`, and never delete `RUN_MOD5_CLEAN` while it is in use.

**Tcl state errors.** If a stage complains about missing variables, re-run `prep` with `-overwrite`, redo the earlier stages in order, and set `VDD_NET` / `GND_NET` before `gen_pdn`.

## Limitations
A clean Magic DRC is one signoff check, not the whole list. This module does not cover LVS, antenna checks, or final signoff STA with extracted parasitics across all corners, so the GDSII should not be called tape-out-ready on this evidence alone.

## Conclusion
The `picorv32a` design went through PDN generation, global and detailed routing, GDSII streaming and Magic DRC in the OpenLane flow. The results table records the DRC and routing outcomes from this run. LVS, antenna checks and multi-corner signoff STA remain as the next steps toward a tape-out-quality layout.
