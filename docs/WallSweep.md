# WallSweep

The flat-target companion to [`MoonSweep.md`](MoonSweep.md): the same sweep, the same two observers,
but the target is an infinite wall instead of a sphere — and the algebra closes.

Notebook: [`WallSweep.ipynb`](../WallSweep.ipynb). Repo-wide conventions, traps and verification
discipline are in [`CLAUDE.md`](../CLAUDE.md).

## The scenario

A laser pointer at the origin sweeps a wall at $y = D$, counterclockwise at uniform angular speed,
from $\alpha = 0$ (along $+x$) through to $\alpha = \pi$, so

$$\alpha(t) = \omega t, \qquad \omega = \frac{\pi}{T} > 0 .$$

At both ends of the sweep the beam runs parallel to the wall and never meets it; in between, the
illuminated spot enters from $x = +\infty$, crosses $x = 0$, and leaves towards $x = -\infty$.

## Why do it again on a plane

Because the algebra closes. The Moon gave the physics, but its ray–sphere intersection carries a
square root, so the fold, the slowest spot and the limb speed all had to be found numerically. A
plane has no square root, and the whole problem collapses onto a **single dimensionless number**

$$\mu = \frac{kD\omega}{c}$$

with $k$ the number of light-travel legs ($\nu = D\omega/c$ is $\mu$ at $k = 1$).

## The reduction: one parameter, and everything in closed form

Each of these is produced by a SymPy call in the notebook and asserted, never typed in:

| quantity | closed form | how |
|---|---|---|
| path length, spot position | $L = D/\sin\alpha$, $x = D\cot\alpha$ | `solve` on $\{y = D,\ (x,y) = L(\cos\alpha, \sin\alpha)\}$ |
| $d\tau/dt$ | $1 - \mu\cos\alpha/\sin^2\alpha$ | `diff`, then asserted against the reduced form |
| $v$ | $-\dfrac{c\mu}{k(\sin^2\alpha - \mu\cos\alpha)}$ | ditto |
| fold $\alpha^*$ | $\cos\alpha^* = \dfrac{\sqrt{\mu^2+4} - \mu}{2}$ | `solve` of `denPoly(u) = 0`, with $u = \cos\alpha$ |
| slowest spot | $\cos\alpha_{slow} = -\mu/2$, $\lvert v\rvert_{min} = \dfrac{4c\mu}{k(4+\mu^2)}$ | `solve(diff(denPoly(cos alpha), alpha))` |
| $\lvert v\rvert = c$ | roots of `denPoly(u)` $= \pm\mu/k$ | `solve`, one quadratic per branch |
| grazing limits | $v/c \to +1/k$ at $\alpha \to 0^+$, $-1/k$ at $\alpha \to \pi^-$ | `limit()`, **with $k$ symbolic** |
| closest approach | $v = -D\omega$, no $k$, no $c$ | `subs(alpha, pi/2)` |

The position system is linear, so there is exactly one root and **no root selection and no square
root anywhere**. `denPoly = Lambda(u, 1 - u**2 - mu*u)` is the rest of the trick:
$\sin^2\alpha - \mu\cos\alpha$ is a quadratic in $u = \cos\alpha$, and it is the shared denominator
of both $d\tau/dt$ and $v$.

Two results MoonSweep could not reach. `limit()` returns $1/k$ and $-1/k$ in well under a second
(MoonSweep abandoned `limit()` after 7+ minutes and demonstrated the limb speed numerically), so the
$c/k$ grazing law is proved for all $k$ at once. And $1 - \lvert v\rvert_{min}/c$ at $k = 1$ factors
to $\dfrac{(\mu-2)^2}{\mu^2+4}$ — a square over a positive — which proves in one line that the spot
at the wall always slows to at most $c$, touching $c$ exactly at $\mu = 2$.

## The threshold at $\mu = 2$, and the scaling symmetry

- $\mu \le 2$: a genuine interior minimum $4c\mu/\bigl(k(4+\mu^2)\bigr)$, attained.
- $\mu > 2$: $\cos\alpha_{slow} = -\mu/2$ is not a cosine, there is no interior minimum, and
  $\lvert v\rvert$ slides monotonically towards $c/k$ without reaching it. `WallCase` sets
  `vSlow = alphaSlow = tSlow = None` in this branch — do not assume `minimise` has something to
  find.

At $k = 1$ the minimum touches exactly $c$ at $\mu = 2$, so a sweep at $D\omega = 2c$ is the one that
grazes the light barrier without crossing.

Because only the product $\mu = k\nu$ enters, **changing observer is the same as changing the sweep
rate**: a $k = 2$ observer sees exactly the $k = 1$ geometry of a sweep at twice the rate, with every
speed halved. §8.1 asserts this between two `WallCase`s.

## The photon front is an Archimedean spiral — exactly

The photon that left at $t$ has covered $c(\tau - t)$ along the direction $\alpha = \omega t$, so at
any moment the string of photons in flight is

$$r(\alpha) = c\left(\tau - \frac{\alpha}{\omega}\right)$$

which is $r$ linear in $\alpha$: an Archimedean spiral, of radial pitch $2\pi c/\omega$ (asserted via
`diff(r, alpha, 2) == 0`). MoonSweep had to argue that its front was "a straight chord to within
$0.16\,\%$"; across a $180^\circ$ sweep no such approximation is available, and none is needed.

The spots are where that spiral meets the wall, so pair creation is a **double root** — the instant
the spiral stops missing the wall and grazes it. Eliminating $\tau$ between $r\sin\alpha = D$ and its
derivative gives back the $k = 1$ fold condition **identically**: `simplify(...) == 0`, where for the
Moon the same statement was a $5 \times 10^{-11}$ numerical assertion.

## Geometry and sign conventions

Geometry is **laser-centric**: $\alpha$ measured from the $+x$ axis.

$\omega > 0$ here (MoonSweep had $\omega < 0$), but the meaning of $v$ is unchanged — $v < 0$ means
the spot runs with the sweep, $v > 0$ is the backward-running spot.

## Structure

`WallSystem(label, distance, speedOfLight, lengthUnit, timeUnit, sweepDuration= | sweepRate=)` is the
scale, parameterized rather than hard-wired: it resolves $\omega$ to a number once, builds the one
flat `values` dict, and carries unit strings for the axis labels. Give exactly one of duration or
rate. `WallCase(system, k, label)` is the `SweepCase` analogue — it takes every event from the closed
forms of §3 and uses `bisect`/`signChanges`/`minimise` (copied from `MoonSweep.ipynb`) only to
*check* them. `SpiralCase(system, label)` is the `FrontCase` analogue; `drawGeometry()` and
`drawSpiralGrid()` render the two SVG diagrams, `plotCase` the branch-split parametric figure.

Two systems: `desk` ($D = 3$ m, $T = 60$ ns, $c = 0.3$ m/ns — the same rounding as MoonSweep's
$300000$ km/s) and `moonWall`, a wall on the **Moon's tangent plane at the sub-Earth point**
($D - R = 380762.5$ km) swept at MoonSweep's own $\omega = 2\arcsin(R/D)/0.01$, rebuilt from
$R = 1737.5$, $D = 382500$, $T = 0.01$ rather than copied.

## Traps specific to this notebook

- **The hazard is a pole, not a tangency.** There is no `sqrt`, so none of MoonSweep's float64
  limb/complex-result traps apply. Instead $L$ and $v$ blow up at $\alpha = 0, \pi$: every grid must
  be **open at both ends** (`np.linspace(...)[1:-1]`, or `branchGrid`), and `SpiralCase.front` wraps
  its $L$ evaluation in `np.errstate(all='ignore')` because $L$ is `+inf` at the poles by design —
  that `inf` is what makes `landed = flown >= reach` correctly false there.
- **Filter roots as exact Rationals.** In `lightSpeedCrossings`, $\mu$ is substituted as
  `Rational(muValue)` so `Abs(root) < 1` is decided exactly. At $k = 1$ the boundary roots are
  *exactly* $u = \pm1$ — the two grazing asymptotes, where $\lvert v\rvert \to c$ without crossing —
  and a float substitution lands one ulp inside the interval and reports a spurious crossing.
- **Golden-section cannot locate a quadratic minimum to $10^{-13}$.** `minimise` gets $t_{slow}$ to
  about $\sqrt{\varepsilon}$ relative; the notebook therefore checks the *position* at $10^{-6}\,T$
  and the *speed*, which is second-order insensitive there, at $10^{-12}$.
- The §6.2 cross-check of $v = u/\bigl(1 + (k/c)\,dL/dt\bigr)$ is asserted at $10^{-10}$, not
  $10^{-13}$: both forms diverge at the fold, so digits cancel in the difference.
- `SpiralCase.front` returns `(x, y, landed)`. Draw the in-flight photons and the landed trail
  **separately** (blank the landed ones with `NaN`), or the green spiral overdraws the orange trail
  and the picture stops meaning anything.

## Reference values

| | $k=1$ (at the wall) | $k=2$ (seen at the laser) |
|---|---|---|
| **desk**, $\nu = \pi/6 = 0.523599$ | $\mu = 0.523599$ | $\mu = 1.047198$ |
| $\alpha^*$ | $39.4750^\circ$ | $52.7578^\circ$ |
| $t^*$ / $t^*/T$ | $13.15833$ ns / $0.2193$ | $17.58592$ ns / $0.2931$ |
| $\tau^*$ | $28.887998$ ns | $42.708898$ ns |
| $x^*$ | $1.2142\,D$ | $0.7602\,D$ |
| slowest speed | $0.490014\,c$ at $105.1769^\circ$ | $0.410938\,c$ at $121.5740^\circ$ |
| $\lvert v\rvert = c$ crossings | 1 (forward, $61.5494^\circ$) | 2 ($35.1940^\circ$, $69.9556^\circ$) |
| **moonWall**, $\nu = 1.153076$ | $\mu = 1.153076$ | $\mu = 2.306151$ |
| $\alpha^*$ | $54.7071^\circ$ | $68.0856^\circ$ |
| $t^*/T$ | $0.3039$ | $0.3783$ |
| $\tau^*$ | $2.605988$ s | $4.044122$ s |
| $x^*$ | $0.7079\,D$ | $0.4023\,D$ |
| slowest speed | $0.865415\,c$ at $125.2074^\circ$ | none — $\mu > 2$ |
| speed at both grazing ends | exactly $c$ | exactly $c/2$ |
| speed at closest approach | $-\nu c$, identical for both $k$ | ditto |

**Cross-notebook anchor**: `moonWall`'s closest-approach speed is $-1.153076\,c$ for both $k$, which
is MoonSweep's sub-Earth spot speed to all printed digits — near the middle of the sweep the Moon is
its own tangent plane. Asserted in §8.1 against the literal `MOONSWEEP_SUB_EARTH`.

Constants are deliberately rounded: $c = 0.3$ m/ns and $c = 300000$ km/s (not 299792.458), matching
MoonSweep. No qualitative result depends on it, but every number above moves if changed.

## What differs qualitatively from the sphere

- **The doubling never ends.** The domain is open: $\tau \to +\infty$ at both ends and $\tau(t)$ has
  a single interior minimum, so there are exactly two spots for every $\tau > \tau^*$, without end.
  On the sphere the backward-running spot died at the limb, closing the window after $2.64$ ms; a
  plane has no limb, and the spot runs out to $x = +\infty$ for ever.
- **Doubling is unconditional.** $\cos\alpha^*$ lies in $(0,1)$ for every $\mu > 0$, so a fold always
  exists however slowly the pointer is turned. It is not a high-speed effect: the far wings of the
  wall are far enough away that their light is always overtaken.
- **Grazing incidence is the same condition as the Moon's limb.** The wall's $\alpha \to 0$ asymptote
  and the Moon's limb both give exactly $c/k$, by the same mechanism — the spot's displacement is
  parallel to the ray. The Moon reaches it at a tangency, the wall at infinity.
- **Before $\tau^*$ the wall is completely dark**, although the laser has been on since $t = 0$. The
  first light to land is not where the beam was first aimed, because that part of the wall is
  infinitely far away and its light is still in flight.

As on the Moon, none of this is information travelling faster than light. The spot is a projection,
like the intersection point of a pair of scissors; no photon overtakes another.
