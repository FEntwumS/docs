# Waveform Interactor

To integrate the Netlist-Viewer seamless into the development workflow of OneWare, the functionality of its Waveform-Viewer has to be extended:

Right-clicking on the signalname in OneWare's waveformviewer should offer context menus to:

1. Navigate to the signal in the netlist-viewer.

2. Jump to the HDLs source origin 

## Steps to ensure navigation to netlist-viewer via waveform-viewer is possible

### 1. Creating Verilog corresponding to Design _(with Yosys)_

First a flattened verilog file has to be generated with yosys from the whole design (from the same yosys processing passes, which have been applied to the HDL to generate the netlist for the netlist-viewer).

```
yosys -p "read_verilog -nooverwrite <top_verilog.v> <other_verilog.v>; scratchpad -set flatten.separator ";"; hierarchy -check -top <toplevel>; proc; memory -nomap; flatten -scopename; write_verilog <flattened_verilog.v>"
```

### 2. Creating the simulation binary _(with Verilator)_

This verilog file is verilated with Verilator to create compilable C++ sources.

```
verilator --timescale-override 10ns/10ns -top-module <toplevel> -Wall --trace --exe --build -cc <testbench.cpp> <flattened_verilog.v>
```

A simulation binary is compiled from a manual-written C++ Verilator testbench and the verilated files.
More on Verilator and its simulation [here](https://verilator.org/guide/latest/verilating.html).

### 3. Running the Simulation

The simulation binary has to be executed, to run the simulation and dump simulation data in .vcd Waveform file.

This has to be configured in the testbench.

### 4. Map yosys/netlist bit-indices to .vcd identifiers

To ensure uniqueness, the Netlist-Viewer and Waveform Interactor identify signals via Bit-Index.
This index is assigned by yosys during the processing passes.

Each signal in the .vcd wavform file also has its own UID, consisting of 1 or 2 ASCII characters.

To map Bit Indeces -> VCD UID, at first the signalname of the processed signal is used. After the yosys passes described in 2., these signalnames contain their scope, and are therefore unique.

Bit Indices can be retrieved via the from the backend via its `/get-net-information` endpoint (see [API information](../netlist_viewer/api_information.md)).

#### _Why not use the signalname as UID, if it is unique after the respective Yosys passes?_

### 5. Recreate the original design hierarchy from the flattened netlist

Since flattening the design is currently required, the signallist in the .vcd contains no scopes at all. All signals lie in the same scope hierarchy.

The idea is to recreate the .vcd scope hierarchy from the net information contained in the netlist. This information is also contained in the .json returned by the backends `/get-net-information` endpoint. 

To recreate this, the OneWare VCD datastructure has to be altered. This is currently not possible, since the corresponding attributes in the vcdView component, from which we can access OneWares parsed .vcd, is read-only.

Another way would be to read the .vcd, alter it before it is parsed by OneWare.

Maybe this is all not necessary, since Flojo is working on a pass without flattening.