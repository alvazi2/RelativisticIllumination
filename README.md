# RelativisticIllumination

Calculations around relativistic illumination effects.

These calculations are inspired by the following book:
- [Faster than Light: How Your Shadow Can Do It but You Can't](https://www.amazon.com/Faster-than-Light-Your-Shadow/dp/1662933843)
- Author: [Robert J. Nemiroff](https://www.mtu.edu/physics/department/faculty/nemiroff/)

The calculations are done in Jupyter notebooks with Python programming.

## Jupyter notebook *MoonSweep*

This notebook does analytical calculations for the following scenario:
- An observer on Earth sweeps a laser pointer with constant angular speed across the surface of the
  Moon, along a straight path through the center of the observed Moon disk.

Notebook details:
- Draw the geometry &mdash; sweep angle, path length, latitude, angle of incidence &mdash; as a
  labelled vector diagram.
- Derive the Earth&ndash;Moon-surface distance as a function of the sweep angle.
- Calculate the time when the laser ray arrives at each point of the path that is traced by the
  beam on the Moon's surface.
- Calculate the speed of the illuminated spot as it travels across the Moon's surface.
- Plot the data.
- Draw the photon front &mdash; the string of photons the sweep has in flight &mdash; meeting the
  Moon at six moments, so the doubling can be seen happening in space.

This visualizes the relativistic image doubling that happens for arrival times that are realized by
two scenarios: early departure but longer path vs later departure but shorter path. Nemiroff's book
explains this in detail.

### The two spots, and why departure time is the parameter

During the doubling window two spots exist at once, so there are two speeds. The notebook resolves
this by parameterizing everything by the photon's **departure time** `t`. Each departure time labels
exactly one illumination event, so latitude, arrival time and spot speed are all single-valued in
`t`. The doubling appears because the arrival time is *not monotonic* in `t` &mdash; two departure
times share one arrival time. A single speed formula therefore covers both spots; it just cannot be
drawn as a function of arrival time, so the plots are parametric curves that fold back on themselves
at the pair-creation event.

Writing the spot speed as

```
v = u / (1 + (k/c) dL/dt)
```

with `u` the geometric aim-point speed and `k` the number of light-travel legs makes the mechanism
explicit. The denominator is the light-delay compression factor &mdash; the same `1/(1 - B cos T)`
that makes astrophysical jets look superluminal.

### The photon front

The sweep does not emit one photon but a string of them, and at any moment that string is a curve in
space: the photon that left at `t` has covered `c(tau - t)` along the aim direction `alpha(t)`. The
illuminated points are where that curve meets the sphere, so the number of spots is the number of
solutions of

```
c (tau - t) = L(alpha(t))
```

A double root of that needs `1 + (1/c) dL/dt = 0`, which is exactly the fold `dtau/dt = 0`. A double
root against a sphere is a tangency &mdash; so **pair creation is the instant the photon front stops
missing the Moon and grazes it**, and the picture is an independent route to the same `t*`. The
notebook asserts the tangency to 5e-11 km.

At the true `D/R = 220` the front is almost straight: it bends 7.4 km away from the chord through its
ends, 0.16 % of that chord's 4575 km. A flat front tilted so that it advances at `c` while the aim
point runs `|omega| D` across it touches the sphere at `arctan(c/(|omega| D)) = 40.80` deg, against
the exact `40.73` deg. The notebook therefore draws the construction twice: once at true proportions,
and once at `D/R = 8` where the bend is visible, with the sweep rate chosen to keep the real
sub-Earth spot speed.

### Results

For a 0.01 s sweep across the Moon (`D` = 382500 km, `R` = 1737.5 km, `c` = 300000 km/s):

| | light arriving at the Moon (`t + L/c`) | light seen back on Earth (`t + 2L/c`) |
|---|---|---|
| pair-creation event | `t* = 0.00172618 s` | `t* = 0.00301148 s` |
| ... at latitude | `40.73 deg` | `23.33 deg` |
| doubling window | `2.6438 ms` | `7.5942 ms` |
| spot speed at both limbs | exactly `c` | exactly `c/2` |
| slowest spot speed | `0.755980 c` | `0.458928 c` |
| spot speed at the sub-Earth point | `-1.153076 c` | `-1.153076 c` |

Three results worth highlighting:

- **The first light to land is not at the limb the beam was aimed at first.** It appears at 40.7 deg
  latitude, because the limb is ~1300 km farther away and its light is still in flight.
- **The spot speed at both limbs is exactly `c/k`**, independent of the distance, the Moon's radius
  and the sweep rate. There the ray is tangent to the surface, so surface displacement is parallel
  to the ray and the path length changes at exactly the rate the spot moves.
- **The sub-Earth speed is the same for both observers.** At that point the path length is at its
  minimum, so `dL/dt = 0` and the leg count `k` drops out of the formula exactly. Everywhere else,
  apparent speed depends on where you stand &mdash; the limb speed halves from `c` to `c/2`.

Nothing here is information travelling faster than light. The spot is a projection, like the
intersection point of a pair of scissors; no photon overtakes another.

## Jupyter notebook *WallSweep*

The flat-target companion to *MoonSweep*. Same sweep, same observers, but the target is an infinite
wall instead of a sphere:

- A laser pointer at the origin sweeps a wall at `y = D`, counterclockwise at uniform angular speed,
  from `alpha = 0` (along `+x`) through to `alpha = pi`, so `omega = pi/T`.

At both ends of the sweep the beam runs parallel to the wall and never meets it; in between, the
illuminated spot enters from `x = +infinity`, crosses `x = 0`, and leaves towards `x = -infinity`.

### Why do it again on a plane

Because the algebra closes. The Moon gave the physics but its ray&ndash;sphere intersection carries a
square root, so the fold, the slowest spot and the limb speed all had to be found numerically. A
plane has no square root, and the whole problem collapses onto a **single dimensionless number**

```
mu = k D omega / c
```

with `k` the number of light-travel legs. Everything is then a closed form in `mu`, obtained from
SymPy rather than written down:

| quantity | closed form |
|---|---|
| path length, spot position | `L = D/sin(a)`,  `x = D cot(a)` |
| pair creation (the fold) | `cos(a*) = (sqrt(mu^2 + 4) - mu)/2` |
| slowest spot | `cos(a_slow) = -mu/2`,  `\|v\|min = 4 c mu / (k (4 + mu^2))` |
| where `\|v\| = c` | roots of `1 - u^2 - mu u = +-mu/k`,  `u = cos(a)` |
| grazing limits | `v/c -> +1/k` as `a -> 0`,  `-1/k` as `a -> pi` |
| closest approach | `v = -D omega`, with no `k` and no `c` in it |

The trick is that `sin^2(a) - mu cos(a)`, the shared denominator of `dtau/dt` and of the spot speed,
is a **quadratic in `cos(a)`**. Two of these results were out of MoonSweep's reach: `limit()` returns
the grazing speeds as exactly `1/k` and `-1/k` with `k` still symbolic, in well under a second, where
MoonSweep had to abandon `limit()` after seven minutes and demonstrate the limb speed numerically.
And `1 - |v|min/c` at `k = 1` factors to `(mu-2)^2/(mu^2+4)` &mdash; a square over a positive &mdash;
proving in one line that the spot at the wall always slows to at most `c`.

### The threshold, and the scaling symmetry

`mu = 2` separates two regimes: below it the spot decelerates to a genuine minimum and speeds up
again; above it there is no minimum at all and `|v|` slides towards `c/k` without ever reaching it.
At `k = 1` the minimum touches exactly `c` at `mu = 2`, so a sweep at `D omega = 2c` is the one that
grazes the light barrier without crossing.

And since only the product `mu = k nu` enters, **changing observer is the same as changing the sweep
rate**: an observer at the laser watching a wall swept at `nu` sees exactly the geometry that light
landing on a wall swept at `2 nu` would have, with every speed halved.

### The photon front is an Archimedean spiral &mdash; exactly

The photon that left at `t` has covered `c(tau - t)` along the direction `alpha = omega t`, so at any
moment the string of photons in flight is

```
r(alpha) = c (tau - alpha/omega)
```

which is `r` linear in `alpha`: an Archimedean spiral, of radial pitch `2 pi c / omega`. MoonSweep
had to argue that its front was "a straight chord to within 0.16 %"; across a 180-degree sweep no
such approximation is available, and none is needed.

The spots are where that spiral meets the wall, so pair creation is a **double root** &mdash; the
instant the spiral stops missing the wall and grazes it. Eliminating `tau` between the intersection
condition and its derivative gives back the fold condition `sin^2(a) = nu cos(a)` *identically*: the
difference simplifies to zero, where for the Moon the same statement was a 5e-11 numerical assertion.

### Results

Two systems run through the same machinery, since the scale is a parameter rather than a constant.
A laser pointer on a wall three metres away, swept through 180 degrees in 60 nanoseconds
(`nu = D omega / c = pi/6`):

| | light arriving at the wall (`t + L/c`) | light seen back at the laser (`t + 2L/c`) |
|---|---|---|
| pair creation | `alpha* = 39.4750 deg` | `alpha* = 52.7578 deg` |
| ... at | `t*/T = 0.2193`, `x* = 1.2142 D` | `t*/T = 0.2931`, `x* = 0.7602 D` |
| spot speed at both grazing ends | exactly `c` | exactly `c/2` |
| slowest spot speed | `0.490014 c` | `0.410938 c` |
| spot speed at closest approach | `-0.523599 c` | `-0.523599 c` |
| crossings of `\|v\| = c` | 1 | 2 |

The second system puts the wall on the **Moon's tangent plane at the sub-Earth point** and sweeps it
at *MoonSweep*'s own rate. Its speed at closest approach comes out at `-1.153076 c` for both `k`
&mdash; *MoonSweep*'s sub-Earth number to all printed digits, because near the middle of the sweep the
Moon simply is its own tangent plane. The notebook asserts it.

Four results worth highlighting:

- **The doubling never ends.** On the sphere the backward-running spot died at the limb, closing the
  window after 2.64 ms. A plane has no limb: the spot runs out to `x = +infinity` for ever, so past
  `tau*` there are **exactly two spots for every arrival time**, without end.
- **Doubling is unconditional.** `cos(a*)` lies in `(0, 1)` for every `mu > 0`, so a fold always
  exists however slowly the pointer is turned. The far wings of the wall are far enough away that
  their light is always overtaken.
- **The spot speed at both grazing ends is exactly `c/k`**, independent of distance and sweep rate.
  This is the same result as the Moon's limb speed and the same mechanism &mdash; grazing incidence,
  where the spot's displacement is parallel to the ray. The Moon reaches it at a tangency on its
  limb; the wall reaches it asymptotically at infinity.
- **Before `tau*` the wall is completely dark**, although the laser has been on since `t = 0`. The
  first light to land is not where the beam was first aimed, because that part of the wall is
  infinitely far away and its light is still in flight.

As on the Moon, none of this is information travelling faster than light. The spot is a projection,
like the intersection point of a pair of scissors; no photon overtakes another.

## Running them

The notebooks need only SymPy, NumPy and Matplotlib (no SciPy). They were last run under Python
3.13.1 with sympy 1.14.0, numpy 2.5.2 and matplotlib 3.10.0.

```
jupyter notebook MoonSweep.ipynb
jupyter notebook WallSweep.ipynb
```

The constants are deliberately rounded (`c` = 300000 km/s rather than 299792.458, `D` = 382500 km;
`c` = 0.3 m/ns in *WallSweep*, the same rounding). None of the qualitative results depend on that.
To use exact figures, adjust the values in *MoonSweep*'s "Values for the Earth&ndash;Moon system"
cell, or the `WallSystem(...)` constructions in *WallSweep* &mdash; there the scale is an argument, so
any wall, sweep rate and unit system can be run through the same machinery.

`MoonSweep_original.ipynb` is the pre-review version of that notebook, kept for comparison.
