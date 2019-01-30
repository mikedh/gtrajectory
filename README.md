gtrajectory
---------------

The aim of this project is to convert G- code to trajectories a standard commercial motion controller can accept, with a focus on the [Position-Velocity-Time and B-Spline forms](http://support.motioneng.com/Software-MPI_04_00/Topics/pt_pvt_motion.htm).

Traditional CNC controllers, used in milling, turning, laser cutting, and many other processes, take G-code, an "organic standard" with a lot of history and layers. At the simplest level it describes Cartesian trajectories (i.e. "move in a line between these two points at this velocity").

Other open source projects intend to entirely replace a traditional CNC controller (i.e. a [Fanuc 32i](https://www.fanucamerica.com/products/cnc/cnc-systems/series-3xi)) with a functionally equivalent unit. [GRBL](https://github.com/gnea/grbl) is a great project which interprets G-code on an Arduino and turns it into stepper motor pulses for small CNC rigs. [LinuxCNC](http://linuxcnc.org/) is similar but as a full PC- based operating system. The challenge with this approach is you have to re- solve the hard problems that motion controllers solve. 

The most similar thing to this project is probably [Mach3](https://www.machsupport.com/software/mach3/), a commercial solution which does the conversion to a number of endpoints, including DMC (for Galil motion controllers), although it includes the conversion as part of a larger application.

This project is intended as a pure Python library which takes G-code and then translates it to run on existing motion controllers. These controllers handle the difficult parts (i.e. control loop frequency, encoder readout, motor drive hookup, etc) with a generic, hardened, industrial solution. My totally unvalidated hypothesis is that motion control companies are better at implementing motion control than machine tool manufacturers.

The library is intended to do ONLY the trajectory conversion; actually running the code on a motion controller would be done at the application level, i.e. elsewhere. 

The motivation for this is pretty simple: in a lights out part- generic CNC cell with automatic loading and unloading (pallet changer + robot), we want our system controller to be able to run new tool paths automatically from a remote server. One could jerry rig the CNC controller to accept new motion automatically, but that would remove most of the point of the expensive box (operator interface) and be quite janky. It would be much nicer if the CNC controls were replaced by a regular motion controller which can take commands over the network.  

## Targeted Motion Controllers
- [Galil DMC-4xxx Series](http://www.galilmc.com/motion-controllers/multi-axis/dmc-42x0)
- [Elmo Maestro](https://www.elmomc.com/products/industrial-environment/multi-axis-controller/)
- [Aerotech Ensemble](https://www.aerotech.com/product-catalog/motion-controller/ensemble.aspx)
- [Clearpath-SC](https://www.teknic.com/products/clearpath-brushless-dc-servo-motors/clearpath-sc/)
- [Siemens SIMOTION C](https://new.siemens.com/global/en/products/automation/systems/motion-control/simotion-hardware/simotion-c.html)

## Workflow

1) Configuration file indicates motion controller type, hardware info (reducer ratios, paired axis, etc), IO maps (for things like tool changes, coolant, touch probes, lights, etc)
2) `gtrajectory` takes g-code and configuration file, and outputs motion- controller native file


## Problem Assumptions

- Position deviations smaller than 0.0001" effectively don't matter. The smallest tolerance one could reasonable expect from most mills and lathes is "two tenths," or 0.0002"
- Position is much more important than velocity. Position deviations larger than .0005" are a huge deal, but velocity within 10% of target is probably OK. 
- Most people doing this try to minimize jerk through lookahead.
