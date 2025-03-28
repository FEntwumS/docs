# Waveform Interactor

To integrate the Netlist-Viewer seamless into the development workflow of OneWare, the functionality of its Waveform-Viewer has to be extended:

Right-clicking on the signalname in OneWare's waveformviewer should offer context menus to:

1. Navigate to the signal in the netlist-viewer.

2. Jump to the HDLs source origin 

This component can be seen as an optional add-on to the Netlist-Viewer, it is useless without the Netlist-Viewer. 

It yet has to be merged into the production package.

## Steps to ensure navigation to netlist-viewer via waveform-viewer is possible

The following section will describe the implementation approach of the Waveform-Interactor component. It utilizes several steps to ensure proper communication with the Netlist-Viewer plugin is possible:

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

To map Bit Indeces -> VCD Identifier, at first the signalname of the processed signal is used. After the yosys passes described in 2., these signalnames contain their scope, and are therefore unique.

Bit Indices can be retrieved via the from the backend via its `/get-net-information` endpoint (see [API information](../netlist_viewer/api_information.md)).

To make bitmapping-per-vcd persistent, this mapping is saved in a .json after successfully obtaining it via the backend.
The mapping of each .vcd in a OneWare project is saved in `<OneWare_project/build/simulation/bitmapping.json>`

Each .vcds body is hashed, as it uniquely represents the waveform, even when the scope hierarchy has been altered and recreated _(See next chapter)_.
This `vcdBodyHash` holds the mapping of all VCD Identifier and their respective BitIndices of the .vcd.

```json
{
    "<vcdBodyHash1>":{
        "<vcdIdentifier>":{
            "BitIndexId": 2,
            "BitIndices": [
                2,
                3,
                4
            ]
        },
        ...
    },
    "<vcdBodyHash2>":{
        "<vcdIdentifier>":{
            "BitIndexId": 1,
            "BitIndices": [
                1
            ]
        },
        ...
    }
}

```

#### _Why not use the signalname as UID, if it is unique after the respective Yosys passes?_

The signalname will change after recreating the original scope hierarchy. This will lead to incorrect mapping of signalname -> vcdIdentifier.
The mapping of the original .vcd can be reused, since the original and recreated vcd share the same body hash.  

### 5. Recreate the original design hierarchy from the flattened netlist

Since flattening the design is currently required, the signallist in the .vcd contains no scopes but the top scope. All signals lie in the same scope hierarchy.

The idea is to recreate the VCD scope hierarchy from the net information contained in the netlist. This information is contained in the signalname generated by yosys after flattening. The backend also offers the `/get-net-information` endpoint to retrieve the scope information. 

To recreate this, the OneWare VCD datastructure has to be altered. This is currently not possible, since the corresponding signal and scope objects in the vcdView component, from which we can access OneWares parsed .vcd, is read-only.

#### Parse the .vcd and write a new one with recreated hierarchy

An alternative approach is to parse the .vcd definitions section into a dedicated data structure and reconstruct the original hierarchy based on the signal names. 
Each signal name encodes its parent scopes, with nested scopes separated by a specific character. 
Currently, this separator is ';', meaning a signal with two levels of hierarchy follows this format (for 2 scopes deep hierarchy):

```
<scope_parent>;<scope>;<signal>
```

From this simple structure the original hierarchy easily be recreated and written into the definitions section of the new .vcd.

_More on the .vcd format can be found [here](https://zipcpu.com/blog/2017/07/31/vcd.html)_

Since the signal values remain unchanged, the body of the original and the hierarchy-recreated .vcd stays identical. After writing the reconstructed hierarchy into the definitions section, the body is streamed directly to the new .vcd file.

_In OneWare_ right-clicking on the .vcd and selecting "Recreate Hierarchy from Signal Names" will generate the updated hierarchy. 
The new file will be saved as <original_vcd>_recreated.vcd.

