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

### Running it

The notebook needs only SymPy, NumPy and Matplotlib (no SciPy). It was last run under Python 3.13.1
with sympy 1.14.0, numpy 2.5.2 and matplotlib 3.10.0.

```
jupyter notebook MoonSweep.ipynb
```

The constants are deliberately rounded (`c` = 300000 km/s rather than 299792.458, `D` = 382500 km).
None of the qualitative results depend on that; adjust the values in the "Values for the
Earth&ndash;Moon system" cell to use exact figures.

`MoonSweep_original.ipynb` is the pre-review version of the notebook, kept for comparison.
