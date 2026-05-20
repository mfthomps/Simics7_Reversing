# Simics7_Reversing
Reverse execution on Simics 7

Simics version 7 deprecated reverse execution, which is arguably the coolest feature of Simics.
An explanation for why can be found on [this blog](https://jakob.engbloms.se/archives/4452).

This project uses the _snapshot_ feature of Simics 7 to recreate some of the reverse execution
functions that existed in Simics 6.  The Python module ReverseMgr implements two key functions:

1. reverse -- Reverses to the most recent breakpoint and stops, optionally invoking a callback to
simulate use of a Core\_Simulation\_Stopped HAP.

2. skipToCycle -- Skip to any cycle within the period over which reverse execution was enabled.

The module also reimplements CLI reversing commands from Simics 6 including:
* enable-reverse-execution, will result in recording snapshots cycles for use in reverse and skip to functions
* disable-reverse-execution
* rev [n] -- where n is cycles to reverse, default is to just reverse.  Reversing will stop either at a breakpoint, or the point at which reversing was enabled.
* skip-to-cycle \<cycle\>

## Demonstration video
A demonstration of the reversing functions is at: https://youtu.be/bbBkQ39JBKo

## Installation
The ReverseMgr is part of the [RESim](https://github.com/mfthomps/RESim) reverse engineering platform, however it has no dependencies
other than Simics and thus can be integrated into other Simics projects.   The current link
to the module is [here](https://github.com/mfthomps/RESim/blob/reverse/simics/monitorCore/reverseMgr.py).

## Usage
Either run the module directly via the Simics _run-script_ command, or import it into a Python script and instantiate the ReverseMgr.  If run via run-script, set a _REVERSE_SPAN_ env value to a suitable value before starting Simics.  If using the Python API, pass that value as _span=value_. (The arg, int_t, and output_modes parameters are Simics variables available in the first level Python script run from _run-script_.)   Experiment with the span value for your simulation.  It controls how frequently a snapshot is made based on cycles within the reference cpu.

Once the module is loaded, the CLI commands can be used, or the module APIs, e.g., enableReverse.

If Python API is used, your Python scripts must first be refactored to call the reverseMgr SIM_breakpoint function instead of the Simics SIM_breakpoint function.  Use the _setCallback_ function to mimic a Simics Core_Simulation_Stopped HAP.  The callback will be passed a struture having fields from the initiating memory transaction, e.g., logical_address, size, etc.

## Notes
These functions behave in a manner similar to reversing functions available
in Simics 6.  As with Simics 6, reliable reversing requires adherence to some
constraints.  For the reverseMgr, these include:
* No Haps should be set while reversing.  See setCallback to simulate a Core_Simulation_Stopped hap
* Real networks and other external events should not be present.
* Breakpoints set prior to reverse execution via the Simics SIM_breakpoint API must be altered to use the reverseMgr's SIM_breakpoint API.

The stratgey is simple.  When reverse is enabled, we take in-memory snapshots
periodically (every cycle_span cycles, ensuring each snapshot falls on multiple of the span).
To reverse or skip, we restore snapshots and run forward to hit either
breakpoints or the requested number of cycles.  The choice of the cycle_span
value can have dramatic effects on performance, and should depend on your simulation.

The reverseMgr module could be instantiated for each target CPU (cell), however, only
one should be enabled at any time via enableReverse.   Each instance has a reference
cpu, whose cycles are associated with snapshots.  Breakpoints on any cell or cpu will be caught during reverse,
and the simulation will stop at the appropriate breakpoint.  Do not enable reversing on multiple instances.

## Limitations
* Assumes each instruction takes one cycle.
* The reverseMgr has been tested on a multicore target.  However RESim uses single-core targets and thus there may be untested conditions.
* Reverses to memory breakpoints (physical or linear).  Does not yet catch other forms of breakpoints while reversing.

