# MoonSweep

A laser swept across the Moon from Earth: derives the arrival time of the light and the speed of the
illuminated spot, and shows the relativistic image doubling.

Notebook: [`MoonSweep.ipynb`](../MoonSweep.ipynb). Repo-wide conventions, traps and verification
discipline are in [`CLAUDE.md`](../CLAUDE.md); the flat-target companion is
[`WallSweep.md`](WallSweep.md).

## The scenario

An observer on Earth sweeps a laser pointer at constant angular speed across the surface of the
Moon, along a straight path through the centre of the observed Moon disk.

The notebook:

- draws the geometry — sweep angle, path length, latitude, angle of incidence — as a labelled vector
  diagram;
- derives the Earth–Moon-surface distance as a function of the sweep angle;
- calculates the time at which the ray arrives at each point of the path the beam traces on the
  surface;
- calculates the speed of the illuminated spot as it travels across the surface;
- plots the data;
- draws the photon front — the string of photons the sweep has in flight — meeting the Moon at six
  moments, so the doubling can be seen happening in space.

## The two spots, and why departure time is the parameter

During the doubling window two spots exist at once, so there are two speeds. The notebook resolves
this by parameterizing everything by the photon's **departure time** $t$. Each departure time labels
exactly one illumination event, so latitude, arrival time and spot speed are all single-valued in
$t$. The doubling appears because the arrival time is *not monotonic* in $t$ — two departure times
share one arrival time. A single speed formula therefore covers both spots; it just cannot be drawn
as a function of arrival time, so the plots are parametric curves that fold back on themselves at
the pair-creation event.

Writing the spot speed as

$$v = \frac{u}{1 + (k/c)\,dL/dt}$$

with $u$ the geometric aim-point speed and $k$ the number of light-travel legs makes the mechanism
explicit. The denominator is the light-delay compression factor — the same $1/(1 - \beta\cos\theta)$
that makes astrophysical jets look superluminal. §6.2 asserts the chain-rule speed against this
compact form to about $10^{-13}$.

## The photon front

The sweep does not emit one photon but a string of them, and at any moment that string is a curve in
space: the photon that left at $t$ has covered $c(\tau - t)$ along the aim direction $\alpha(t)$.
The illuminated points are where that curve meets the sphere, so the number of spots is the number
of solutions of

$$c(\tau - t) = L(\alpha(t)).$$

A double root of that needs $1 + (1/c)\,dL/dt = 0$, which is exactly the fold $d\tau/dt = 0$. A
double root against a sphere is a tangency — so **pair creation is the instant the photon front
stops missing the Moon and grazes it**, and the picture is an independent, purely geometric route to
the same $t^*$. This is not a coincidence to be admired but the same equation; the notebook asserts
the tangency to $5 \times 10^{-11}$ km in radius, and to $4 \times 10^{-8}$ in the cosine
between the radius and the front.

At the true $D/R = 220$ the front is almost straight: it bends $7.43$ km away from the chord through
its ends, $0.162\,\%$ of that chord's $4575.02$ km. A flat front tilted so that it advances at $c$
while the aim point runs $\lvert\omega\rvert D$ across it touches the sphere at
$\arctan\bigl(c/(\lvert\omega\rvert D)\bigr) = 40.8043^\circ$, against the exact $40.7315^\circ$.
The notebook therefore draws the construction twice: once at true proportions, and once at
$D/R = 8$ where the bend is visible, with the sweep rate chosen to keep the real sub-Earth spot
speed.

## Geometry and sign conventions

Geometry is **Moon-centric**: $a$ transverse to the Earth–Moon axis, $b$ along it measured from the
Moon's centre *towards Earth*. The near-side (illuminated) ray–sphere root is selected by the
physical condition $L(\alpha = 0) = D - R$, never by list index.

Domain is departure time $t \in [0, T]$, $T = 0.01$ s — *not* arrival time $\tau \approx 1.27$ s.
$\omega < 0$ (the sweep runs $+\alpha_{max} \to -\alpha_{max}$); $v < 0$ means the spot moves with
the sweep, $v > 0$ is the backward-running spot.

## Structure

`SweepCase(k, label)` lambdifies the expressions and locates the events (`tStar`, `tClose`,
`cCross`, `tSlow`); `plotCase` draws the branch-split parametric figure. `drawGeometry(ratio)`
(§3.1) renders the labelled geometry as SVG — it draws the Moon at $D/R = 2.4$ instead of 220 so the
angles are visible, but locates each point with `pathLength`/`surfaceAngle` at the drawing's own $D$
and $R$, and re-asserts $L(0) = D - R$, the point being on the sphere, and $\beta$ being the centre
angle. `limbValue(f, values)` evaluates a closed form exactly at the limb; both `tau0` and `tauEnd`
in `SweepCase` come from it, since $\tau(0)$ and $\tau(T)$ land on the tangency.

§6 draws the same physics in space. `FrontCase(values, sweepRate, label)` is the `SweepCase` of that
section — same shape, but parameterized over the *system* rather than over $k$, so the true
Earth–Moon values and an exaggerated schematic cost one call each. Its `front(tau)` clips each
photon's radius to $L$, so the landed ones freeze on the surface as the swept trail; `spots(tau,
extra)` brackets the roots of $\tau(t) = \tau$ *at the fold*, so it returns 0, 1 or 2 — that count is
the doubling. `drawFrontGrid` renders the six-panel SVG for either case.
`sweepRateFor(values, subEarthSpeed)` picks the schematic's $\omega$ so its sub-Earth spot keeps the
true $1.153076\,c$; only $D/R$ is faked.

## Traps specific to this notebook

- **The limb is a tangency, and float64 cannot sit on it.** At $\alpha = \pm\alpha_{max}$ the
  ray–sphere discriminant $R^2 - D^2\sin^2\alpha$ is analytically zero, so a *rounded*
  $\alpha_{max}$ puts it one ulp negative: `sqrt` then returns a complex number (SymPy) or `nan`
  (lambdified to NumPy), and every quantity at the limb is lost — `float()` raises
  `TypeError: Cannot convert complex to float`, and a `bisect` bracket built on it dies with
  `no sign change ... f = nan, nan`. This is version-sensitive: it surfaced on the upgrade to
  sympy 1.14 / numpy 2.5, having been latent before. Evaluate limb quantities via
  `limbValue(f, values)`, which keeps $\alpha_{max}$ symbolic so $\sin(\arcsin(R/D))$ cancels to
  $R/D$ and the discriminant is exactly zero before any number goes in. Never feed a float
  $\alpha_{max}$ (or $t = 0$ / $t = T$) to the lambdified expressions.
- Two more limb traps live in §6: $L(t)$ is `nan` at $t = 0$ and $t = T$, so `front` overwrites those
  two entries with `limbPath` under `np.errstate(all='ignore')` (the warning is otherwise captured
  into the committed output), and `FrontCase.latitude` returns $\pm(90^\circ - \alpha_{max})$ at the
  limbs rather than calling the lambdified $\beta$.
- **`limit()` does not terminate here** — 7+ minutes on the speed expression. The limb speed is
  demonstrated numerically instead, and the notebook says so. `WallSweep.ipynb` gets the same result
  symbolically, for all $k$ at once.

## Reference values

For a $0.01$ s sweep, confirmed independently with mpmath at 30–60 digits:

| | $k=1$ (at the surface) | $k=2$ (observed on Earth) |
|---|---|---|
| pair-creation event $t^*$ | $0.00172618$ s | $0.00301148$ s |
| $\tau^*$ / latitude $\beta^*$ | $1.272343$ s / $40.7315^\circ$ | $2.542379$ s / $23.3313^\circ$ |
| doubling window | $2.6438$ ms | $7.5942$ ms |
| spot speed at both limbs | exactly $c$ | exactly $c/2$ |
| slowest spot speed | $0.755980\,c$ | $0.458928\,c$ |
| sub-Earth spot speed | $-1.153076\,c$ | $-1.153076\,c$ |

Geometric invariants, asserted after substitution: $L(0) = D - R = 380762.5$ km,
$L(\alpha_{max}) = \sqrt{D^2 - R^2} = 382496.0537$ km, and $\beta$ at the limb
$= 90^\circ - \alpha_{max} = 89.739734^\circ$.

§6 values: bend $7.43$ km in $4575.02$ km ($0.162\,\%$); flat-front estimate $40.8043^\circ$ against
the exact $40.7315^\circ$. The $D/R = 8$ schematic gives $\beta^* = 35.48^\circ$, $t^*/T = 0.1784$,
window/sweep $0.2308$, bend $4.551\,\%$ (true: $40.73^\circ$, $0.1726$, $0.2091$, $0.162\,\%$) —
exaggerating the ratio keeps the shape, not the numbers, so the title says so.

Constants $c = 300000$ km/s (not 299792.458) and $D = 382500$ km are deliberately rounded;
$R = 1737.5$ km. No qualitative result depends on this, but every number above moves if changed.

## What the results mean

- **The first light to land is not at the limb the beam was aimed at first.** It appears at
  $40.7^\circ$ latitude, because the limb is ~1300 km farther away and its light is still in flight.
- **The spot speed at both limbs is exactly $c/k$**, independent of the distance, the Moon's radius
  and the sweep rate. There the ray is tangent to the surface, so surface displacement is parallel
  to the ray and the path length changes at exactly the rate the spot moves.
- **The sub-Earth speed is the same for both observers.** At that point the path length is at its
  minimum, so $dL/dt = 0$ and the leg count $k$ drops out of the formula exactly — a free check that
  $k$ is threaded correctly. Everywhere else, apparent speed depends on where you stand: the limb
  speed halves from $c$ to $c/2$.

Nothing here is information travelling faster than light. The spot is a projection, like the
intersection point of a pair of scissors; no photon overtakes another.
