# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A set of analytical physics notebooks working through the relativistic observer effects in Robert J.
Nemiroff's *Faster than Light: How Your Shadow Can Do It but You Can't*. The book describes these
effects **qualitatively**; the point of this repo is to derive them **analytically and numerically** —
closed forms, substituted numbers, and independently verified results.

Two notebooks so far — `MoonSweep.ipynb` (sweep against a sphere) and `WallSweep.ipynb` (the same
sweep against an infinite plane); more on related topics from the same book are planned.
Structure new work so it reads as another chapter of the same project, not a standalone script.

No package, no build, no test suite, no `requirements.txt` — SymPy, NumPy and Matplotlib only.
**SciPy is deliberately absent**; use bracketed root-finders (see below) rather than adding it.

`MoonSweep_original.ipynb` is a frozen pre-review copy of that notebook, kept for comparison. Do not
edit it. It is byte-identical to commit `a3adef0`.

## Commands

```bash
jupyter notebook <Notebook>.ipynb                   # interactive

# Headless full re-run — the primary check that nothing is broken
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=1200 <Notebook>.ipynb

# Did anything raise, and did every cell run?
python3 -c "
import json,sys; nb=json.load(open(sys.argv[1]))
print([(i,o['ename']) for i,c in enumerate(nb['cells']) for o in c.get('outputs',[]) if o.get('output_type')=='error'] or 'no errors')
print([c.get('execution_count') for c in nb['cells'] if c['cell_type']=='code'])" <Notebook>.ipynb
```

Prefer editing a notebook by generating it from a script and executing with `nbconvert`, rather than
hand-patching JSON — round-tripping `.ipynb` by hand is error-prone.

Notebooks are committed **with** their outputs (embedded PNGs), which is intentional; expect large,
noisy diffs on every re-run.

## Repo-wide conventions

These generalize beyond any one notebook.

- **Layer the calculation**: symbolic closed forms → a single flat numeric substitution dict →
  `lambdify` to NumPy → plots. Keep SymPy as the source of truth; never re-type a derived expression
  by hand for plotting.
- **Parameterize variants instead of duplicating cells.** `MoonSweep.ipynb` threads a light-leg count
  `k` (`k=1` light reaching the target, `k=2` the round trip back to the observer) through every
  function, so both observers cost one extra call. Prefer that shape for any "same physics, different
  observer/geometry" variant.
- **Everything is a function of the photon's *departure* time**, never of its arrival time. This is
  what makes doubled images tractable: one departure time labels exactly one illumination event, so
  all quantities stay single-valued, while the doubling shows up as a non-monotonic arrival time.
  Observables that are double-valued in arrival time must be plotted **parametrically** and split
  into branches at the fold (`d(arrival)/d(departure) = 0`, which is the pair-creation event).
- **Root-finding uses brackets, not initial guesses.** `bisect`, `signChanges` and `minimise` are
  defined in `MoonSweep.ipynb` and are worth copying. A bare `nsolve` guess converges by luck here.
- **Explanatory diagrams are Matplotlib line art shown as inline SVG** (`fig.savefig(buf,
  format='svg', bbox_inches='tight')` → `IPython.display.SVG`), so they stay sharp at any zoom while
  the data plots stay PNG. Place every point in such a diagram with the derived symbolic expressions
  rather than by hand, and assert the same geometric invariants on it that the calculation asserts.
  A schematic may exaggerate a ratio for legibility; say so in the title, and show the true
  proportions alongside.
- Use plain-language identifiers matching the physics (`pathLength`, `surfaceAngle`, `sweepAngle`,
  `pointSpeed`), and `#` comments that state the physical meaning rather than the code's mechanics.

## SymPy traps that have actually caused bugs here

- **Silent complex results.** Evaluating a function outside its real domain sends a `sqrt` negative;
  the result is complex, and SymPy's plotting then silently *discards* those points and draws a
  plausible-looking but meaningless curve. Always check outputs for imaginary parts after a change.
- **Never put two substitutions in one dict when one introduces the other's symbol.** SymPy
  substitutes simultaneously, so in `{T: value, omega: -2*asin(R/D)/T}` the `T` inside `omega`
  survives the pass. Resolve the dependent symbol to a number first. Chaining repeated
  `.evalf(subs=...)` calls to force it through is the old broken idiom — do not reintroduce it.
- **`limit()` may not terminate** on large derived expressions (7+ minutes on the MoonSweep speed
  expression). Demonstrate limits numerically instead, and say so in the notebook.
- **The limb is a tangency, and float64 cannot sit on it.** At `alpha = +-alpha_max` the ray-sphere
  discriminant `R^2 - D^2 sin^2(alpha)` is analytically zero, so a *rounded* `alpha_max` puts it one
  ulp negative: `sqrt` then returns a complex number (SymPy) or `nan` (lambdified to NumPy), and
  every quantity at the limb is lost — `float()` raises `TypeError: Cannot convert complex to
  float`, and a `bisect` bracket built on it dies with `no sign change ... f = nan, nan`. This is
  version-sensitive: it surfaced on the upgrade to sympy 1.14 / numpy 2.5, having been latent
  before. Evaluate limb quantities via `limbValue(f, values)`, which keeps `alpha_max` symbolic so
  `sin(asin(R/D))` cancels to `R/D` and the discriminant is exactly zero before any number goes in.
  Never feed a float `alpha_max` (or `t = 0` / `t = T`) to the lambdified expressions.
- **Float64 constants destroy high-precision limits.** Rounding a constant displaces a singular point
  by ~1e-16, so evaluating closer than that measures the rounding error, not the limit — and raising
  `evalf` precision does not help. Build such checks from exact `Rational`/`Integer` values.
- Select roots of `solve()` by a physical condition, never by list index; the ordering is not
  guaranteed.

## Verification discipline

A result is finished when it has been checked against something that did not produce it:

- **Assert the derivation against a second route.** `MoonSweep.ipynb` checks its chain-rule spot speed
  against the independent compact form `v = u / (1 + (k/c)·dL/dt)` to ~1e-13. Keep such assertions
  live — they are what catch sign errors.
- **Cross-check numerically with mpmath**, which ships with SymPy and so costs no dependency. Running
  the same physics at 30–60 digits without touching SymPy confirms the symbolic pipeline instead of
  assuming it.
- **State geometric invariants and assert them** after substitution.

## Notebook: MoonSweep.ipynb

A laser swept across the Moon from Earth; derives light arrival time and the speed of the
illuminated spot, showing relativistic image doubling.

Geometry is Moon-centric: `a` transverse to the Earth–Moon axis, `b` along it measured from the
Moon's centre *towards Earth*. The near-side (illuminated) ray–sphere root is selected by
`L(alpha=0) == D - R`. `SweepCase(k, label)` lambdifies the expressions and locates the events
(`tStar`, `tClose`, `cCross`, `tSlow`); `plotCase` draws the branch-split parametric figure.
`drawGeometry(ratio)` (§3.1) renders the labelled geometry as SVG — it draws the Moon at `D/R = 2.4`
instead of 220 so the angles are visible, but locates each point with `pathLength`/`surfaceAngle` at
the drawing's `D` and `R`, and re-asserts `L(0) = D - R`, the point being on the sphere, and `beta`
being the centre angle. `limbValue(f, values)` evaluates a closed form exactly at the limb; both
`tau0` and `tauEnd` in `SweepCase` come from it, since `tau(0)` and `tau(T)` land on the tangency.

§6 draws the same physics in space: the **photon front**, the string of photons the sweep has in
flight, whose intersections with the sphere are the live spots. `FrontCase(values, sweepRate, label)`
is the `SweepCase` of that section — same shape, but parameterized over the *system* rather than over
`k`, so the true Earth–Moon values and an exaggerated schematic cost one call each. Its `front(tau)`
clips each photon's radius to `L`, so the landed ones freeze on the surface as the swept trail;
`spots(tau, extra)` brackets the roots of `tau(t) = tau` *at the fold*, so it returns 0, 1 or 2 —
that count is the doubling. `drawFrontGrid` renders the six-panel SVG for either case.
`sweepRateFor(values, subEarthSpeed)` picks the schematic's `omega` so its sub-Earth spot keeps the
true `1.153076 c`; only `D/R` is faked. Two more limb traps live here: `L(t)` is `nan` at `t = 0` and
`t = T`, so `front` overwrites those two entries with `limbPath` under `np.errstate(all='ignore')`
(the warning is otherwise captured into the committed output), and `FrontCase.latitude` returns
`±(90° - alpha_max)` at the limbs rather than calling the lambdified `beta`.

Domain is departure time `t ∈ [0, T]`, `T = 0.01 s` — *not* arrival time `tau ≈ 1.27 s`. Sign
conventions: `omega < 0` (sweep runs `+alpha_max → -alpha_max`); `v < 0` means the spot moves with
the sweep, `v > 0` is the backward-running spot.

Reference values, confirmed independently with mpmath at 30–60 digits:

| | `k=1` (at the surface) | `k=2` (observed on Earth) |
|---|---|---|
| pair-creation event `t*` | `0.00172618 s` | `0.00301148 s` |
| `tau*` / `beta*` | `1.272343 s` / `40.7315°` | `2.542379 s` / `23.3313°` |
| doubling window | `2.6438 ms` | `7.5942 ms` |
| spot speed at both limbs | exactly `c` | exactly `c/2` |
| slowest spot speed | `0.755980 c` | `0.458928 c` |
| sub-Earth spot speed | `-1.153076 c` | `-1.153076 c` |

Geometric invariants: `L(0) = D - R = 380762.5 km`, `L(alpha_max) = sqrt(D²-R²) = 382496.0537 km`,
`beta` at the limb `= 90° - alpha_max = 89.739734°`. And, from §6: **at `t*` the photon front is
tangent to the sphere** — asserted to `5e-11 km` in radius and `4e-8` in `cos(radius, front)`. That
is not a coincidence to be admired but the same equation: the front meets the sphere where
`c(tau - t) = L(alpha(t))`, and a double root of that is `1 + (1/c) dL/dt = 0`, i.e. `dtau/dt = 0`.
It is the notebook's second, purely geometric route to `t*`.

§6 reference values: the front departs from the straight chord through its ends by `7.43 km` in
`4575.02 km` (0.162 %), so at the true `D/R = 220` it is a straight tilted chord translating at `c`;
the flat-front estimate `arctan(c/(|omega| D)) = 40.8043°` against the exact `40.7315°`. The
schematic at `D/R = 8` gives `beta* = 35.48°`, `t*/T = 0.1784`, window/sweep `0.2308`, bend `4.551 %`
(true: `40.73°`, `0.1726`, `0.2091`, `0.162 %`) — exaggerating the ratio keeps the shape, not the
numbers, so say so in the title.

The sub-Earth speed is identical for both `k` because `dL/dt = 0` there, so the leg count cancels —
a free check that `k` is threaded correctly.

Constants `c = 300000 km/s` (not 299792.458) and `D = 382500 km` are deliberately rounded;
`R = 1737.5 km`. No qualitative result depends on this, but every number above moves if changed.

## Notebook: WallSweep.ipynb

A laser pointer at the origin sweeps an infinite wall at `y = D`, counterclockwise at uniform rate
from `alpha = 0` (along `+x`) to `alpha = pi`, so `omega = pi/T > 0`. The flat-target companion to
`MoonSweep.ipynb`: same physics, but the algebra closes.

Geometry is laser-centric: `alpha` measured from the `+x` axis, `L(alpha) = D/sin(alpha)`,
`x(alpha) = D*cot(alpha)`. Both come from `solve` on `{y = D, (x,y) = L(cos a, sin a)}` — the system
is linear, so there is exactly one root and **no root selection and no square root anywhere**.

Sign conventions: `omega > 0` here (MoonSweep had `omega < 0`), but the meaning of `v` is unchanged —
`v < 0` means the spot runs with the sweep, `v > 0` is the backward-running spot.

### The reduction: one parameter, and everything in closed form

Everything depends on the system only through `mu = k*D*omega/c` (`nu = D*omega/c` is `mu` at
`k = 1`). Each of these is produced by a SymPy call in the notebook and asserted, never typed in:

| quantity | closed form | how |
|---|---|---|
| `dtau/dt` | `1 - mu*cos(a)/sin(a)^2` | `diff`, then asserted against the reduced form |
| `v` | `-c*mu/(k*(sin(a)^2 - mu*cos(a)))` | ditto |
| fold `alpha*` | `cos(a*) = (sqrt(mu^2+4) - mu)/2` | `solve` of `denPoly(u) = 0`, `u = cos(alpha)` |
| slowest spot | `cos(a_slow) = -mu/2`, `\|v\|min = 4*c*mu/(k*(4+mu^2))` | `solve(diff(denPoly(cos a), a))` |
| `\|v\| = c` | roots of `denPoly(u) = ±mu/k` | `solve`, one quadratic per branch |
| grazing limits | `v/c -> +1/k` at `a->0+`, `-1/k` at `a->pi-` | `limit()`, **with `k` symbolic** |
| closest approach | `v = -D*omega`, no `k`, no `c` | `subs(alpha, pi/2)` |

`denPoly = Lambda(u, 1 - u**2 - mu*u)` is the whole trick: `sin^2(a) - mu*cos(a)` is a quadratic in
`u = cos(alpha)`, and it is the shared denominator of both `dtau/dt` and `v`.

Two results MoonSweep could not reach. `limit()` returns `1/k` and `-1/k` in well under a second
(MoonSweep abandoned `limit()` after 7+ minutes and demonstrated the limb speed numerically), so the
`c/k` law is proved for all `k` at once. And `1 - |v|min/c` at `k = 1` factors to `(mu-2)^2/(mu^2+4)`
— a square over a positive — which proves in one line that the spot at the wall always slows to at
most `c`, touching `c` exactly at `mu = 2`.

### The threshold at `mu = 2`

- `mu <= 2`: a genuine interior minimum `4*c*mu/(k*(4+mu^2))`, attained.
- `mu > 2`: `cos(a_slow) = -mu/2` is not a cosine, there is no interior minimum, and `|v|` slides
  monotonically towards `c/k` without reaching it. `WallCase` sets `vSlow = alphaSlow = tSlow = None`
  in this branch — do not assume `minimise` has something to find.

Because only `mu = k*nu` enters, a `k = 2` observer sees exactly the `k = 1` geometry of a sweep at
twice the rate, with every speed halved. §8.1 asserts this between two `WallCase`s.

### Structure

`WallSystem(label, distance, speedOfLight, lengthUnit, timeUnit, sweepDuration= | sweepRate=)` is the
scale, parameterized rather than hard-wired: it resolves `omega` to a number once, builds the one
flat `values` dict, and carries unit strings for the axis labels. Give exactly one of duration or
rate. `WallCase(system, k, label)` is the `SweepCase` analogue — it takes every event from the closed
forms of §3 and uses `bisect`/`signChanges`/`minimise` (copied from `MoonSweep.ipynb`) only to
*check* them. `SpiralCase(system, label)` is the `FrontCase` analogue; `drawGeometry()` and
`drawSpiralGrid()` render the two SVG diagrams, `plotCase` the branch-split parametric figure.

Two systems: `desk` (`D = 3 m`, `T = 60 ns`, `c = 0.3 m/ns` — the same rounding as MoonSweep's
`300000 km/s`) and `moonWall`, a wall on the **Moon's tangent plane at the sub-Earth point**
(`D - R = 380762.5 km`) swept at MoonSweep's own `omega = 2*asin(R/D)/0.01`, rebuilt from
`R = 1737.5`, `D = 382500`, `T = 0.01` rather than copied.

### Reference values

| | `k=1` (at the wall) | `k=2` (seen at the laser) |
|---|---|---|
| **desk**, `nu = pi/6 = 0.523599` | `mu = 0.523599` | `mu = 1.047198` |
| `alpha*` | `39.4750°` | `52.7578°` |
| `t*` / `t*/T` | `13.15833 ns` / `0.2193` | `17.58592 ns` / `0.2931` |
| `tau*` | `28.887998 ns` | `42.708898 ns` |
| `x*` | `1.2142 D` | `0.7602 D` |
| slowest speed | `0.490014 c` at `105.1769°` | `0.410938 c` at `121.5740°` |
| `\|v\| = c` crossings | 1 (forward, `61.5494°`) | 2 (`35.1940°`, `69.9556°`) |
| **moonWall**, `nu = 1.153076` | `mu = 1.153076` | `mu = 2.306151` |
| `alpha*` | `54.7071°` | `68.0856°` |
| `t*/T` | `0.3039` | `0.3783` |
| `tau*` | `2.605988 s` | `4.044122 s` |
| `x*` | `0.7079 D` | `0.4023 D` |
| slowest speed | `0.865415 c` at `125.2074°` | none — `mu > 2` |
| speed at both grazing ends | exactly `c` | exactly `c/2` |
| speed at closest approach | `-nu*c`, identical for both `k` | ditto |

**Cross-notebook anchor**: `moonWall`'s closest-approach speed is `-1.153076 c` for both `k`, which
is `MoonSweep.ipynb`'s sub-Earth spot speed to all printed digits — near the middle of the sweep the
Moon is its own tangent plane. Asserted in §8.1 against the literal `MOONSWEEP_SUB_EARTH`.

### What differs qualitatively from the sphere

- The domain is **open**: `tau -> +infinity` at both ends, `tau(t)` has a single interior minimum, so
  there are exactly two spots for every `tau > tau*` and **the doubling window never closes**. The
  Moon's backward spot died at the limb; here it runs to `x = +infinity` for ever.
- The Moon's limb and the wall's `alpha -> 0` asymptote are the *same* condition — grazing incidence
  — which is why both give exactly `c/k`. The Moon reaches it at a tangency, the wall at infinity.
- A fold exists for **every** `mu > 0`, since `cos(a*)` is in `(0,1)` always. Doubling here is not a
  high-speed effect, it is unconditional.
- The photon front is an **exact** Archimedean spiral `r = c*(tau - alpha/omega)` (asserted via
  `diff(r, alpha, 2) == 0`), of radial pitch `2*pi*c/omega`. MoonSweep needed "a straight chord to
  0.16 %"; across a 180° sweep no such approximation exists and none is needed. Eliminating `tau`
  between `r*sin(a) = D` and its derivative gives the `k = 1` fold condition **identically** —
  `simplify(...) == 0`, not a 5e-11 numerical assertion as it was for the Moon.

### Traps specific to this notebook

- **The hazard is a pole, not a tangency.** There is no `sqrt`, so none of MoonSweep's float64
  limb/complex-result traps apply. Instead `L` and `v` blow up at `alpha = 0, pi`: every grid must be
  **open at both ends** (`np.linspace(...)[1:-1]`, or `branchGrid`), and `SpiralCase.front` wraps its
  `L` evaluation in `np.errstate(all='ignore')` because `L` is `+inf` at the poles by design — that
  `inf` is what makes `landed = flown >= reach` correctly false there.
- **Filter roots as exact Rationals.** In `lightSpeedCrossings`, `mu` is substituted as
  `Rational(muValue)` so `Abs(root) < 1` is decided exactly. At `k = 1` the boundary roots are
  *exactly* `u = ±1` — the two grazing asymptotes, where `|v| -> c` without crossing — and a float
  substitution lands one ulp inside the interval and reports a spurious crossing.
- **Golden-section cannot locate a quadratic minimum to 1e-13.** `minimise` gets `t_slow` to about
  `sqrt(eps)` relative; the notebook therefore checks the *position* at `1e-6*T` and the *speed*,
  which is second-order insensitive there, at `1e-12`.
- The §6.2 cross-check of `v = u/(1 + (k/c) dL/dt)` is asserted at `1e-10`, not `1e-13`: both forms
  diverge at the fold, so digits cancel in the difference.
- `SpiralCase.front` returns `(x, y, landed)`. Draw the in-flight photons and the landed trail
  **separately** (blank the landed ones with `NaN`), or the green spiral overdraws the orange trail
  and the picture stops meaning anything.

Constants are deliberately rounded: `c = 0.3 m/ns` and `c = 300000 km/s` (not 299792.458), matching
MoonSweep. No qualitative result depends on it, but every number above moves if changed.
