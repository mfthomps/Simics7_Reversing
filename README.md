# Simics7_Reversing
Reverse execution on Simics7

Simics version 7 deprecated reverse execution, which is arguably the coolest feature of Simics.
An explanation for why can be found on [this blog](\url{https://jakob.engbloms.se/archives/4452}).

This project uses the _ _snapshot_ _ feature of Simics 7 to recreate some of the reverse execution
functions that existed prior to Simics 7.  The Python module ReverseMgr implements two key functions:

1. reverse -- Reverses to the most recent breakpoint and stops, optionally invoking a callback to
simulate use of a Core\_Simulation\_Stopped HAP.

2. skipToCycle -- Skip to any cycle within the period over which reverse execution was enabled.

The ReverseMgr is part of the RESim reverse engineering platform, however it has no dependencies
other than Simics and thus can be integrated into other Simics projects.   The current link
to the module is [here](https://github.com/mfthomps/RESim/blob/reverse/simics/monitorCore/reverseMgr.py).

Usage
=====
The module is typically loaded and used by another Python module, but it can be loaded and used directly
for testing as follows:

- Use the link about to download the ReverseMgr and put it where Simcics can find it, e.g., your workspace.
- Load a Simics configuration, e.g., from a checkpoint.
- Use _ _run-script reverseMgr.py_ _ 
- Run to the point at which you wish to enable reverse execution.
- Use _ _@rev.enableReverse()_ _ to enable reverse execution.
- Run forward a while
- Set one or more breakpoints that you wish to reverse to.
- Use _ _@rev.reverse()_ _
- Use _ _@skipToCycle(cycle)_ _ to skip to a cycle.
- 
