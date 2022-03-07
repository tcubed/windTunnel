# windTunnel

A simple 2D wind tunnel simulator in python.

# TLDR (too long, didn't read)

This simulator takes a 2D design of a car profile or wing profile, such as the following:

<img src="https://github.com/tcubed/windTunnel/blob/master/content/car.png" style="height:50px">

The simulator will create velocity and pressure maps showing the air stream trajectories flowing around the object.

<img src="https://github.com/tcubed/windTunnel/blob/master/content/car_out.png" style="height:400px">

The "fx" shows the horizontal drag force on the object ("fy" indicates lift), with lower numbers having less drag.  Re-run different designs to improve your numbers.

# Introduction

Making vehicles or components more aerodynamic leads to increased fuel efficiency.
<a href="https://en.wikipedia.org/wiki/Wind_tunnel">Wind Tunnels</a> are one approach to study the flow of air around vehicles.
This project is a simple 2D wind tunnel simulator, allowing users to evaluate and tweak designs virtually before physical
modifications are made.

This simulator is based on the <a href="https://github.com/tcubed/lbm">tcubed/lbm</a> python implementation of a 
<a href="https://en.wikipedia.org/wiki/Lattice_Boltzmann_methods">lattice Boltzmann method (LBM)</a> fluid simulator.  LBM is a popular technique for simulating air and liquid flows, which achieves realistic, complicated flows with a surprisingly simple computational approach.

## LBM: practical insights

All quantities in LBM are in "lattice" units, such as lattice length, lattice time, and lattice mass.  LBM simulations can be related to "real-world" air flows through keeping various <a href="https://en.wikipedia.org/wiki/Dimensionless_numbers_in_fluid_mechanics">dimensionless ratios</a>.  For example the <a href="https://en.wikipedia.org/wiki/Reynolds_number">Reynold's number</a> represents, basically how turbulent an air flow is.  The practical outcomes of this are:

 - Scaling the car size to a smaller width (e.g. 100 pixels) will simulate slower, less-turbulent airflows.  Being smaller in computational size, the simulation will also run faster for each iteration.
 - Scaling the car size to a larger width (e.g. 500 pixels) will simulate faster, more-turbulent airflows.  The simulation will take longer to run.
 - Lowering the air viscosity (in LBM), we can model more turbulent air flows with smaller car sizes.  There are limits to this, as the simulation will become more unstable.

Without any explanation here, 0.1 lattice units (lu)/lattice time (lt) is traditionally the maximum velocity used in LBM.  This isn't a limitation in most situations (except as you reach the speed of sound), but rather represents the <i>a priori</i> assignment of max velocity in the simulation to make the simulation proceed as fast as possible.

# An example

<a href="https://en.wikipedia.org/wiki/Pinewood_derby">Pinewood derbies</a> are a toy car competition where wooden cars are rolled down a ramp to a finish line.  Starting with a block of wood, competitors can modify the cars for design or performance within established rules.  A common modification is adjusting the profile of the car, cutting off pieces of the block.  Hence, this is a perfect "toy" application of this wind tunnel simulator.

Years ago, competitors stated a rectangular block of wood.  These days, there are many starting profiles that can be used.  We took a picture of one such  profile, and made a black-and-white map of it and saved a cropped version of the image.

<img src="https://github.com/tcubed/windTunnel/blob/master/content/car.png" style="height:50px">

To create the input image used for the simulator, the image is rescaled to be 300 pixels in width.   Using the 7" length of the car, the image was padded 75mm in front and behind, 50mm above, and 5mm beneath the car to represent the total tunnel area.

The air flow entering from the left and the "road" speed on the bottom are the same, and the top and right boundaries are open to the atmosphere.  Remember, for our simple analysis, we don't care about actual values, but we are looking for designs which will improve performance.  In this case, reducing drag.

## Diagnostic and output results

For several hundred iterations (or more, depending on input size), the simulation will permeate the domain and evolve.  We don't want to stop the simulation before it has "settled down" to a consistent flow profile.  Here, convergence is measured by how much the velocity field changes from iteration to iteration, and the simulation is ended when the difference drops below a certain level.  We've created a diagnostic image below to show the simulation convergence on the left, the horizontal (x) forces in the middle, and the vertical (y) forces at the right.  Our aim is to create designs with the lowest fx-total value, which is the drag.

<img src="https://github.com/tcubed/windTunnel/blob/master/content/car_diag.png" style="height:300px">

In the evaluation script, the diagnostic and output images are updated every 100 iterations.  The output image, below, shows the velocity results on the top and the pressure results on the bottom.  The velocity field is has warmer colors for higher velocity, where it increases going over the car.  The pressure field has warmer colors for higher pressure, where it is higher at the front of the car and lower behind the car.  The air streamlines are shown which show how tracer particles in the air would flow over the car.

<img src="https://github.com/tcubed/windTunnel/blob/master/content/car_out.png" style="height:300px">

## Interpretation

The front of the car has a negative incline, leading to a high pressure zone.  The back of the car, shown darker in the figure, is a low pressure zone.  The net effect is the front of the car is pushed backward and the back of the car is pulled backward compared to being stationary stationary.  The fx value of 1.53 represents the "value to beat".  That is, we want to make changes to the car, like reshaping the front of the car, to reduce this value and thus reduce the drag.

# More theory and method details

The air flow from the left was set to 0.1 lu/lt and the road at the bottom of the input moves to the right at 0.1 lu/lt.  The top and right boundaries are open.  The fluid (i.e. air) density was set to 1 lm/lu^3.  Remember, for our simple analysis, we don't care about actual values (therefore there is no need to change them), but we are looking for designs which will improve performance.  In this case, reducing drag.

The LBM simulator solves for the local velocity and pressure fields iteratively.  The drag consists of the normal net force between the front-facing and back-facing lattice points and the tangential force on the top-facing and the bottom-facing lattice points.

