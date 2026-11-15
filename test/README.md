# GF180 SRAM simulation tests

The Cocotb testbench drives the actual Tiny Tapeout top-level interface and
independently exercises the official 64x8, 128x8, 256x8 and 512x8 SRAM models.

The Google/GlobalFoundries Verilog models include specify timing checks. The
Makefile enables Icarus specify support with `-gspecify`; the tests use a 100 ns
clock period (50 ns high and low) to satisfy the model's specified 55.6 ns
minimum cycle and 25 ns minimum pulse widths. Do not disable these checks to
make a test pass.

Run the RTL test:

```sh
make -B
```

For gate-level testing, first run the GF180 GDS workflow or copy the hardened
Tiny Tapeout netlist to `gate_level_netlist.v`, then run:

```sh
make -B GATES=yes
```

The ASIC source list contains only the project RTL. The official behavioral
SRAM models are simulation-only. The physical GDS, LEF and Liberty files are
provided by the pinned submodule and registered in `src/config.json`.
