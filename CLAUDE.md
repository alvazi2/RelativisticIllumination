# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A set of analytical physics notebooks working through the relativistic observer effects in Robert J.
Nemiroff's *Faster than Light: How Your Shadow Can Do It but You Can't*. The book describes these
effects **qualitatively**; the point of this repo is to derive them **analytically and numerically** —
closed forms, substituted numbers, and independently verified results.

More notebooks on related topics from the same book are planned. Structure new work so it reads as
another chapter of the same project, not a standalone script.

No package, no build, no test suite, no `requirements.txt` — SymPy, NumPy and Matplotlib only.
**SciPy is deliberately absent**; use bracketed root-finders (see below) rather than adding it.

`MoonSweep_original.ipynb` is a frozen pre-review copy of that notebook, kept for comparison. Do not
edit it. It is byte-identical to commit `a3adef0`.

## The notebooks

Each notebook has a detailed spec under `docs/` — geometry, sign conventions, structure, reference
values and its own traps. **Read the relevant one before editing or reasoning about a notebook**;
this file carries only what generalizes across all of them.

| notebook | target | spec |
|---|---|---|
| `MoonSweep.ipynb` | a sphere (the Moon at `D/R = 220`), swept from Earth | [`docs/MoonSweep.md`](docs/MoonSweep.md) |
| `WallSweep.ipynb` | an infinite plane; same physics, but the algebra closes | [`docs/WallSweep.md`](docs/WallSweep.md) |

Reference values live in those specs and nowhere else — do not copy tables back into this file or
into `README.md`. `README.md` is the short reader-facing overview and also points at `docs/`.

What holds across both, and should be expected to hold in a new one:

- **The light-leg count `k`** parameterizes the observer: `k=1` light reaching the target, `k=2` the
  round trip back to the observer.
- **At grazing incidence the spot moves at exactly `c/k`** — at both ends of the sweep, independent
  of distance, target size and sweep rate. The Moon reaches it at a tangency on its limb, the wall
  asymptotically at infinity. Any new target geometry should reproduce this.
- **`v < 0` means the spot runs with the sweep**, `v > 0` is the backward-running spot. This holds
  even though the notebooks disagree on the sign of `ω` (MoonSweep `< 0`, WallSweep `> 0`).
- **Cross-notebook anchor**: a wall on the Moon's tangent plane at the sub-Earth point, swept at
  MoonSweep's rate, gives a closest-approach speed of `-1.153076 c` for both `k` — MoonSweep's
  sub-Earth spot speed to all printed digits. Asserted in `WallSweep.ipynb` §8.1.
- **The constants are deliberately rounded** (`c = 300000 km/s`, not 299792.458). No qualitative
  result depends on it, but every published number moves if it is changed.

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

- **Layer the calculation**: symbolic closed forms → a single flat numeric substitution dict →
  `lambdify` to NumPy → plots. Keep SymPy as the source of truth; never re-type a derived expression
  by hand for plotting.
- **Parameterize variants instead of duplicating cells.** `MoonSweep.ipynb` threads `k` through every
  function, so both observers cost one extra call; `WallSweep.ipynb` does the same with the whole
  system scale (`WallSystem`). Prefer that shape for any "same physics, different
  observer/geometry/scale" variant.
- **Everything is a function of the photon's *departure* time**, never of its arrival time. This is
  what makes doubled images tractable: one departure time labels exactly one illumination event, so
  all quantities stay single-valued, while the doubling shows up as a non-monotonic arrival time.
  Observables that are double-valued in arrival time must be plotted **parametrically** and split
  into branches at the fold (`d(arrival)/d(departure) = 0`, which is the pair-creation event).
- **Root-finding uses brackets, not initial guesses.** `bisect`, `signChanges` and `minimise` are
  defined in `MoonSweep.ipynb` and copied into `WallSweep.ipynb`. A bare `nsolve` guess converges by
  luck here.
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
- **Both ends of the sweep are singular, in every geometry.** Grazing incidence is where the
  interesting result lives *and* where float64 fails, so no grid may include its endpoints and no
  endpoint quantity may be evaluated by substituting a rounded number. The failure mode differs by
  target — a tangency the discriminant cannot sit on for the sphere (`nan`/complex), a pole for the
  plane (`inf`) — so see the notebook's own spec, but assume the ends are hostile before checking.
- **Evaluate a singular point from a form that keeps the symbol symbolic.** `limbValue(f, values)`
  in `MoonSweep.ipynb` is the pattern: hold `α_max` symbolic so `sin(asin(R/D))` cancels to
  `R/D` and the discriminant is exactly zero *before* any float enters. Substituting a rounded
  singular point instead puts the expression one ulp on the wrong side.
- **Float64 constants destroy high-precision limits.** Rounding a constant displaces a singular point
  by ~1e-16, so evaluating closer than that measures the rounding error, not the limit — and raising
  `evalf` precision does not help. Build such checks from exact `Rational`/`Integer` values.
- **Decide boundary tests on exact Rationals.** When a root must be classified against a boundary it
  may sit exactly on (`Abs(u) < 1` at a grazing asymptote), substitute `Rational(value)`, not a
  float: a float lands one ulp inside and reports a root that is not there.
- **`limit()` may not terminate** on large derived expressions (7+ minutes on the MoonSweep speed
  expression, abandoned). Demonstrate limits numerically instead and say so in the notebook — but
  try it first, since the same limit returns in under a second on the WallSweep forms.
- Select roots of `solve()` by a physical condition, never by list index; the ordering is not
  guaranteed.

## Verification discipline

A result is finished when it has been checked against something that did not produce it:

- **Assert the derivation against a second route.** Both notebooks check the chain-rule spot speed
  against the independent compact form `v = u / (1 + (k/c)·dL/dt)`. Keep such assertions live — they
  are what catch sign errors.
- **Cross-check numerically with mpmath**, which ships with SymPy and so costs no dependency. Running
  the same physics at 30–60 digits without touching SymPy confirms the symbolic pipeline instead of
  assuming it.
- **State geometric invariants and assert them** after substitution.
- **A second, independent geometric route beats a tighter tolerance.** The photon front reaching the
  target is the same equation as the fold, so its tangency re-derives `t*` from the picture alone —
  numerically for the sphere (`5e-11`), identically for the plane (`simplify(...) == 0`).
- **Match the tolerance to the method, and say why in the assertion.** A golden-section minimum is
  only good to ~`sqrt(eps)`, so assert the *position* loosely and the *value* — second-order
  insensitive there — tightly. Two forms that both diverge at the fold cancel digits in their
  difference, so `1e-10` is the honest bound, not `1e-13`. Do not tighten a tolerance past what the
  method can deliver, and do not loosen one without a stated reason.
