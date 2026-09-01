# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A set of analytical physics notebooks working through the relativistic observer effects in Robert J.
Nemiroff's *Faster than Light: How Your Shadow Can Do It but You Can't*. The book describes these
effects **qualitatively**; the point of this repo is to derive them **analytically and numerically** —
closed forms, substituted numbers, and independently verified results.

Currently one notebook, `MoonSweep.ipynb`; more on related topics from the same book are planned.
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
`beta` at the limb `= 90° - alpha_max = 89.739734°`.

The sub-Earth speed is identical for both `k` because `dL/dt = 0` there, so the leg count cancels —
a free check that `k` is threaded correctly.

Constants `c = 300000 km/s` (not 299792.458) and `D = 382500 km` are deliberately rounded;
`R = 1737.5 km`. No qualitative result depends on this, but every number above moves if changed.
