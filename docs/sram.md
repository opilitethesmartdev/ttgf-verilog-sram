# GF180MCU SRAM integration

This template uses the official Google/GlobalFoundries GF180MCU SRAM IP from
`google/globalfoundries-pdk-ip-gf180mcu_fd_ip_sram`. It is pinned as a Git
submodule so the physical and simulation views remain a matched set.

## Included macros

All four official single-port, 8-bit-wide macro variants are registered in
`src/config.json`:

| Cell | Capacity | LEF dimensions (µm) | Minimum tile containing the macro |
|---|---:|---:|---:|
| `gf180mcu_fd_ip_sram__sram64x8m8wm1` | 64 x 8 | 431.86 x 232.88 | 2x2 |
| `gf180mcu_fd_ip_sram__sram128x8m8wm1` | 128 x 8 | 431.86 x 268.88 | 2x2 |
| `gf180mcu_fd_ip_sram__sram256x8m8wm1` | 256 x 8 | 431.86 x 340.88 | 3x4 |
| `gf180mcu_fd_ip_sram__sram512x8m8wm1` | 512 x 8 | 431.86 x 484.88 | 3x4 |

The current design instantiates only the 64x8 cell. It is explicitly placed at
(140, 45) µm in the 2x2 tile. The larger cells are available in the macro
configuration and have black-box declarations and simulation coverage, but
are not placed until instantiated by a design.

Each macro entry includes GDS, LEF, a Verilog black-box declaration and three
3.3 V Liberty corners: typical 25°C, slow 125°C and fast -40°C. The Liberty
views are included in STA; physical GDS and LEF views are used for stream-out
and place-and-route. The official Verilog functional/timing models are used
only by simulation.

## Default example interface

| Signal | Function |
|---|---|
| `ui_in[5:0]` | 64-word SRAM address |
| `ui_in[6]` | Write enable, active high |
| `uio_in[7:0]` | Write data |
| `uo_out[7:0]` | Registered SRAM read data |
| `ena` | Enables SRAM access when high |
| `rst_n` | Holds SRAM chip enable inactive while reset is asserted |
| `clk` | Rising-edge SRAM clock |

The top-level ties `WEN=8'h00`, enabling all eight data bits. It drives
`GWEN=0` for writes and `GWEN=1` for reads. Both `CEN` and `GWEN` are
active low. `CEN` is kept high while reset is asserted or `ena` is low.

The macro has no reset input. Reset only disables the macro; it does not clear
the memory array. Do not rely on power-up SRAM contents in a manufactured
device. The vendor simulation model initializes its internal array for
simulation, but that initialization is not a silicon reset mechanism.

## Timing and simulation

The official GF180 model specifies a 55.6 ns minimum clock cycle, 25 ns
minimum high and low pulse widths, 5 ns input setup and 10 ns input hold.
Icarus Verilog cannot elaborate the model's conditional bit-specific output
path declarations, so the Cocotb functional tests do not enable specify path
checking. They use a 100 ns clock period, but this does not constitute timing
signoff and does not test setup/hold violations.

The physical implementation uses the vendor 3.3 V Liberty files and the
project's 20 ns clock constraint (50 MHz). LibreLane STA is responsible for
checking the actual setup/hold and macro timing paths. The functional RTL test
must not be used as a substitute for that analysis.

## Adding a different SRAM size

To switch the default instance:

1. Replace the cell module in `src/project.v` with the required macro name and
   match its address width (6, 7, 8 or 9 bits).
2. Update `src/config.json` so the `instances` placement names the actual
   instance and its coordinates fit the selected tile.
3. Update `info.yaml` `tiles` to a size that contains the LEF dimensions.
4. Update the RTL and Cocotb top-level tests for the changed address range.

The 256x8 and 512x8 macros are taller than a 2x2 tile, so use at least 3x4
for those variants. Do not treat these hard macros as inferred memories:
keep the black-box declaration for ASIC synthesis and use the official
behavioral model only for simulation.

## Source and licensing

The macro views are maintained by the Google GF180MCU SRAM IP repository and
carry their upstream GlobalFoundries PDK Authors copyright and Apache-2.0
license. This project pins the upstream submodule commit; it does not modify
or regenerate the GDS, LEF, Liberty or functional Verilog views.
