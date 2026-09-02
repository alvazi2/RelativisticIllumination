# RelativisticIllumination

Calculations around relativistic illumination effects — what an observer actually *sees* when a spot
of light is swept across a distant target faster than light.

These calculations are inspired by the following book:
- [Faster than Light: How Your Shadow Can Do It but You Can't](https://www.amazon.com/Faster-than-Light-Your-Shadow/dp/1662933843)
- Author: [Robert J. Nemiroff](https://www.mtu.edu/physics/department/faculty/nemiroff/)

The book describes these effects qualitatively. Here they are worked out analytically and
numerically in Jupyter notebooks: closed forms from SymPy, substituted numbers, and results
cross-checked against an independent route.

## The notebooks

| notebook | scenario | write-up |
|---|---|---|
| [`MoonSweep.ipynb`](MoonSweep.ipynb) | An observer on Earth sweeps a laser pointer across the Moon, along a straight path through the centre of the observed disk. | [docs/MoonSweep.md](docs/MoonSweep.md) |
| [`WallSweep.ipynb`](WallSweep.ipynb) | The same sweep against an infinite wall — the flat-target companion, where the algebra closes into one dimensionless parameter. | [docs/WallSweep.md](docs/WallSweep.md) |

Each write-up carries the scenario, the derivation, the reference values and what the results mean.

## What comes out of them

Both notebooks derive the arrival time of the light and the speed of the illuminated spot, and both
find the same three things.

**The spot is doubled.** For a range of arrival times the target carries *two* spots at once, one
running forward and one backward. They are born together at a pair-creation event, when the string
of photons the sweep has in flight first grazes the target. This is why every quantity is
parameterized by the photon's *departure* time: arrival time is not monotonic, so it cannot be the
independent variable, and the plots fold back on themselves at the event.

**The first light to land is not where the beam was aimed first.** That part of the target is
farther away, and its light is still in flight. On the Moon the first spot appears at $40.7^\circ$ latitude;
on the wall, the whole wall stays dark until the pair-creation event, although the laser has been on
from the start.

**At grazing incidence the spot moves at exactly $c/k$**, where $k$ counts the light-travel legs to
the observer — independent of distance, target size and sweep rate. The Moon reaches that condition
at a tangency on its limb, the wall asymptotically at infinity. Everywhere else the apparent speed
depends on where you stand: the same limb runs at $c$ for an observer at the surface and $c/2$ for
the one back on Earth.

None of this is information travelling faster than light. The spot is a projection, like the
intersection point of a pair of scissors; no photon overtakes another.

## Running them

The notebooks need only SymPy, NumPy and Matplotlib (no SciPy). They were last run under Python
3.13.1 with sympy 1.14.0, numpy 2.5.2 and matplotlib 3.10.0.

```
jupyter notebook MoonSweep.ipynb
jupyter notebook WallSweep.ipynb
```

The constants are deliberately rounded ($c = 300000$ km/s rather than 299792.458, $D = 382500$ km;
$c = 0.3$ m/ns in *WallSweep*, the same rounding). None of the qualitative results depend on that.
To use exact figures, adjust the values in *MoonSweep*'s "Values for the Earth&ndash;Moon system"
cell, or the `WallSystem(...)` constructions in *WallSweep* &mdash; there the scale is an argument, so
any wall, sweep rate and unit system can be run through the same machinery.

`MoonSweep_original.ipynb` is the pre-review version of that notebook, kept for comparison.
