TO DO:
Count dot-dot and dot-wall collisions separately.
Store several successive snapshots of x, y, v, and theta, for both rendering and tabulating, if user wishes
    to step back.
include analog clock, to provide reassurance of continuous flow of time?
Tabulate the mass (but don't make it input-able).
input coefficient of restitution for each particle, separately
determine if real-time sorting of n^2 interactions would be more performant than only looking for one w/min dt.
Reintroduce max/min limits on x & y?

Make each object a separate component (ie, make this approach more object-oriented)

Enable user to throw an object.

Decide between two directions below:
1) general n, controls (density, diameter, position, speed, direction, coefficient of restitution, angular speed?), wall reflectivity, and meters (speeds and angles)
buttons, checkboxes, etc:
walls: reflect, none, temperature-controlled, periodic?

use arrows to indicate velocity?
tabulate individual and/or total momenta and/or KE?

1- and 2-d versions of this would be similar to PhET's ["Collision Lab"](https://phet.colorado.edu/sims/html/collision-lab/latest/collision-lab_all.html)

2) higher n, stat-mech demo, Maxwellian comparison, controlling T of walls
(Similar to PhET's ["Gas properties"](https://phet.colorado.edu/en/simulation/gas-properties)
