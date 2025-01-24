### Design Tools

The design tools offer an easy-to-use design flow to implement and debug digital circuits in OneWare. 

#### NetlistViewer

The main feature is the [NetlistViewer](https://github.com/FEntwumS/Oneware.NetlistReaderFrontend). 

It is built on top of the open-source tool [yosys](https://github.com/YosysHQ/yosys). Yosys parses the HDL design and outputs the netlist, which is then converted by the [FEntwumS Backend](https://github.com/FEntwumS/NetlistReaderBackend) into a layout. This layout is used by the Frontend to render a graph view.

To make development with OneWare seamless, the FEntwumS Plugins add interactions to OneWare's Waveform Viewer. This way, the Waveform Viewer, Netlist Viewer, and Source Code are connected via mouse clicks, making it easy to jump from one view to another while debugging.
