# Tiny Tapeout GF180 SRAM Verilog Template

[![GDS](https://github.com/opilitethesmartdev/ttgf-verilog-sram/actions/workflows/gds.yaml/badge.svg)](https://github.com/opilitethesmartdev/ttgf-verilog-sram/actions/workflows/gds.yaml)
[![RTL tests](https://github.com/opilitethesmartdev/ttgf-verilog-sram/actions/workflows/test.yaml/badge.svg)](https://github.com/opilitethesmartdev/ttgf-verilog-sram/actions/workflows/test.yaml)
[![Docs](https://github.com/opilitethesmartdev/ttgf-verilog-sram/actions/workflows/docs.yaml/badge.svg)](https://github.com/opilitethesmartdev/ttgf-verilog-sram/actions/workflows/docs.yaml)
[![FPGA](https://github.com/opilitethesmartdev/ttgf-verilog-sram/actions/workflows/fpga.yaml/badge.svg)](https://github.com/opilitethesmartdev/ttgf-verilog-sram/actions/workflows/fpga.yaml)

A Tiny Tapeout GF180MCU Verilog starter project that integrates the official
Google/GlobalFoundries SRAM macros. The default project instantiates a physical
64x8 SRAM and uses the official vendor model for RTL simulation.

## What is included

- Official Google GF180MCU SRAM IP as a pinned Git submodule at
  `src/gf180mcu_fd_ip_sram`.
- Physical GDS and LEF views for the 64x8, 128x8, 256x8 and 512x8 macros.
- 3.3 V typical, slow and fast Liberty timing views for each macro.
- Separate black-box Verilog declarations for ASIC synthesis and the official
  functional/timing models for simulation.
- A Cocotb testbench that exercises the example interface and all four SRAM
  model sizes.
- A separate inferred-memory implementation for the iCE40 FPGA workflow.

## Getting started

Clone the repository with its submodule:

```sh
git clone --recurse-submodules https://github.com/opilitethesmartdev/ttgf-verilog-sram.git
cd ttgf-verilog-sram
```

For an existing clone, initialize the macro submodule with:

```sh
git submodule update --init --recursive
```

Add project Verilog to `src`, update `info.yaml`, and keep the `source_files`
list synchronized with the test Makefile. The SRAM physical views and timing
libraries are registered in `src/config.json`.

## Default pin mapping

| Tiny Tapeout pin | Function |
|---|---|
| `ui_in[5:0]` | SRAM address |
| `ui_in[6]` | Write enable, active high |
| `uio_in[7:0]` | Write data |
| `uo_out[7:0]` | Read data |
| `ui_in[7]` | Unused |

The example uses the macro's full-byte write mask (`WEN=8'h00`). Reset does
not clear SRAM contents: the macro has no reset pin, and the contents after
power-up must be treated as undefined.

## SRAM sizes and floorplan

The supplied macros are 8 bits wide. The minimum supported Tiny Tapeout tile
dimensions that contain each macro are:

| Macro | LEF size (µm) | Minimum tile |
|---|---:|---|
| 64x8 | 431.86 x 232.88 | 2x2 |
| 128x8 | 431.86 x 268.88 | 2x2 |
| 256x8 | 431.86 x 340.88 | 3x4 |
| 512x8 | 431.86 x 484.88 | 3x4 |

Only the 64x8 macro is instantiated by default and has a placement in
`src/config.json`. The other macros are registered as optional physical IP
and are covered by standalone simulation tests. When changing the instance,
update the Verilog module name/address width, `MACROS.instances`, and
`info.yaml` tile count together.

## Workflows

GitHub Actions runs Cocotb RTL tests, the GF180 GDS build, precheck, gate-level
tests, documentation build and iCE40 FPGA bitstream build. All repository
workflows are enabled on pushes to `main`. The GDS/GL and FPGA workflows use
recursive submodule checkout.

See [docs/sram.md](docs/sram.md) for the SRAM interface, timing and integration
details. The official source is
[Google's GF180MCU SRAM IP repository](https://github.com/google/globalfoundries-pdk-ip-gf180mcu_fd_ip_sram).
