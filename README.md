# VHDL Matrix Processing Core


**Archive status:** the preserved implementation computes **D = C × (A × B)**. The assignment requires **D = (A × B) × C**. These operations are generally different. This repository documents the submitted implementation and its known limitation; it does not claim full specification compliance.

## Design

| Item | Archived implementation |
|---|---|
| HDL | VHDL, `numeric_std` signed arithmetic |
| Matrices | Three 16 × 16 inputs, signed 8-bit elements |
| Datapath | Two sequential multiply–accumulate stages |
| Memory | Three 256 × 8 input buffers, 256 × 32 intermediate buffer, 256 × 56 output buffer |
| Control | Finite-state machine and row/column address counters |
| Target | Xilinx Virtex-7 `xc7vx485tffg1157-1` |
| Original tool | Vivado 2018.3 |
| Simulation clock | 200 ns period (5 MHz); not a measured maximum clock rate |

The top-level entity is `IntMatMulCore`; the testbench is `tb_IntMatMulCore`. Source filenames and header comments still say “4x4” because they originated from the teaching reference. The source in `rtl/` uses 16 × 16 matrices.

`BufferSel` selects A (`00`), B (`01`), or C (`10`). Addresses are 8 bits and input data is 8 bits. `ReadData` is 56 bits. `DataReady` pulses at final completion. Follow the supplied testbench's loading sequence: the controller assumes a fixed transfer schedule and does not implement a general backpressure interface.

## Repository contents

- `rtl/`: preserved coursework RTL.
- `tb/`: preserved file-based testbench.
- `matlab/`: original numerical reference script.
- `ip/`: three Xilinx Block Memory Generator configurations; regenerate vendor output products in Vivado.
- `vectors/`: the archived input matrices and reference output.
- `results/`: saved simulation comparisons and synthesis utilization report.
- `scripts/check_archived_results.py`: newly added independent arithmetic check.
- `docs/REVIEW.md`: findings, evidence limits, and improvements.
- `images/`: space for screenshots.

## Results and verification

The three non-empty archived comparison logs each contain 256 matching MATLAB/simulation values. An independent Python integer calculation confirms all 256 archived reference values equal `C × (A × B)`; none equal `(A × B) × C` for this dataset.

Run the archive check from the repository root:

```sh
python scripts/check_archived_results.py
```

This checks saved data, not a fresh HDL simulation. Vivado simulation and synthesis have not been rerun during repository preparation. No physical FPGA test is claimed.

The saved top-level synthesis report lists 240 LUTs, 170 registers, and 2 DSPs. It also lists five memory black boxes, so its zero-BRAM count is **not** a complete device utilization result.

## Recreate in Vivado

1. Create an RTL project targeting `xc7vx485tffg1157-1`, preferably with the archived tool version, Vivado 2018.3.
2. Add the file in `rtl/` as a design source and select `IntMatMulCore` as top.
3. Add the three `.xci` files under `ip/`; generate their output products and simulation models. Newer Vivado versions may require an IP upgrade.
4. Add the file in `tb/` as a simulation source and select `tb_IntMatMulCore` as simulation top.
5. Add all four files in `vectors/` as simulation data files. Ensure they are copied into the simulator working directory using their exact names. Input lines contain the five-character prefix consumed by the testbench; preserve their formatting.
6. Run behavioral simulation for approximately 4 ms, then inspect the generated `OutputC.txt`. Compare against the archived results.

These are reconstruction instructions, not a verified automated build. The MATLAB script displays hexadecimal arrays but does not automatically write the testbench files. The original `.xpr` and generated caches were omitted to avoid stale absolute paths. No board pin constraints or timing-closure claim is supplied.

## Screenshots


<!-- ![RTL waveform](images/rtl-waveform.png) -->
<!-- ![Synthesized schematic](images/schematic.png) -->
<!-- ![Resource utilization](images/utilization.png) -->

## Credits and licensing

AMD/Xilinx owns its tools and vendor IP; their applicable terms remain in force. No blanket open-source license has been assigned to the mixed-provenance coursework. Attribution alone does not grant redistribution permission for the teaching reference. Confirm the course material's redistribution terms before publishing those derived files publicly.
