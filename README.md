# symcelmech

An open-source Maxima package for analytical celestial mechanics based on Lie transformations.

`symcelmech` automates the construction of high-order perturbation theories for Hamiltonian systems using the Hori-Deprit method (Lie series). It provides a complete symbolic pipeline — from potential derivation through orbital averaging to generating function computation — supporting both closed-form (exact in eccentricity) and series expansion modes.

## Features

- Automated Hori-Deprit/Mersman recursive Lie triangle up to arbitrary order
- Dual-track integration: closed-form (true anomaly domain) and Poisson series (mean anomaly domain)
- Perturbing potential derivation: zonal harmonics (J_n), tesseral/sectorial harmonics (C_nm, S_nm), third-body perturbations, and solar radiation pressure (SRP)
- Triaxial ellipsoid gravity model: closed-form spherical harmonic coefficients (C_2l,2m) for homogeneous asteroids via Balmino's formula
- Hansen coefficient engine for exact orbital averaging
- Dual averaging routes for positive-power radial terms (third-body, SRP): Hansen coefficients and eccentric-anomaly quadrature (see Averaging Routes)
- Memoized Poisson bracket computation with antisymmetry exploitation
- Mean-to-osculating element transformation via Lie series corrections
- Deprit's elimination of the parallax
- Lagrange, Gauss, and Hamilton planetary equations (classical and non-singular equinoctial)
- Delaunay-Poincare canonical transformations
- Elliptic expansion series and exact anomaly relations (equation of the center, r/a, cos f, sin f, and the inverse relations cos E, sin E of the true anomaly)
- Physical constants database (Earth, Moon, Mercury, Apophis, Didymos)

## Requirements

- [Maxima](https://maxima.sourceforge.io/) (tested with 5.47+)
- Maxima packages: `orthopoly`, `vect` (bundled with standard Maxima distributions)

## Installation

### Option 1: Add to Maxima search path (recommended)

#### Step 1: Clone the repository

```bash
git clone https://github.com/guilhermedeolivpaes/symcelmech.git
```

#### Step 2: Add to Maxima search path

Edit (or create) the file `~/.maxima/maxima-init.mac` (Linux/macOS) or `%USERPROFILE%/maxima/maxima-init.mac` (Windows):

```maxima
file_search_maxima: cons("/full/path/to/symcelmech/###.mac", file_search_maxima)$
```

Replace `/full/path/to/symcelmech/` with the actual path to the cloned repository.

#### Step 3: Load the package

From any Maxima or wxMaxima session:

```maxima
load("symcelmech")$
```

> **Note (wxMaxima users):** Make sure your current notebook/script file is saved before calling `load`. wxMaxima may fail to resolve paths if the working file has unsaved changes.

### Option 2: Direct load

If you prefer not to modify `maxima-init.mac`, load the entry point with the full path:

```maxima
load("/full/path/to/symcelmech/symcelmech.mac")$
```

### Option 3: Load individual modules

```maxima
load("/path/to/symcelmech/derivepotential.mac")$
load("/path/to/symcelmech/planetaryequations.mac")$
```

Internal dependencies are resolved automatically.

## Running Tests

The package includes a regression test suite compatible with Maxima's `run_testsuite`:

```maxima
batch("rtest_symcelmech.mac", test)$
```

## Package Structure

| Module | Description |
|---|---|
| `lietransformations.mac` | Core Hori-Deprit solver (`solve_hori_deprit_recurrence`) and Lie series corrections (`apply_lie_corrections`) |
| `analyticalfunctions.mac` | Poisson bracket engines (series and closed-form), generating function quadrature, trigonometric expansion tools, elimination of the parallax |
| `derivepotential.mac` | Perturbing potential derivation: zonal, tesseral, third-body, SRP, and triaxial ellipsoid harmonics |
| `average.mac` | Secular/periodic separation, Hansen coefficient computation, orbital averaging |
| `celmechseries.mac` | Elliptic expansion series and eccentricity truncation tools |
| `algebraicutils.mac` | Memoization, expression sanitization, coordinate substitution rules |
| `planetaryequations.mac` | Lagrange, Gauss, and Hamilton equations; Delaunay-Poincare transformations |
| `astrodata.mac` | Physical and orbital constants for supported celestial bodies |

### Dependency Graph

```
lietransformations.mac
├── poissonbracket.mac
│   └── canonicalsetup.mac
├── perturbationtools.mac
│   └── postprocessing.mac
├── average.mac
├── keplerkinematics.mac
├── postprocessing.mac
└── canonicalsetup.mac

potentialmodels.mac
├── keplerkinematics.mac
└── celmechseries.mac

hamiltoniansystems.mac
└── canonicalsetup.mac

planetaryequations.mac   (standalone)
canonicalsetup.mac       (leaf)
keplerkinematics.mac     (leaf)
celmechseries.mac        (leaf)
postprocessing.mac       (leaf)
astrodata.mac            (leaf)
average.mac              (leaf)
```

## Averaging Routes for Positive Radial Powers

Perturbations whose net radial power satisfies k > -1 (third-body, SRP) cannot
be averaged over the satellite true anomaly by the plain closed-form route,
which is designed for the gravitational regime k <= -2. `symcelmech` provides
two independent routes that supply this same need:

- **Hansen coefficients** (`hansen_third_body_average`, `hansen_srp_average`):
  return the exact secular mean via the X0 coefficients. These functions return
  the secular part only. No jacobian is applied, since the Hansen coefficients
  already embed the averaging measure over the mean anomaly.

- **Eccentric-anomaly quadrature** (`prepare_potential_closedform` with the
  optional eccentric-anomaly argument, followed by `map_average`): rewrites the
  explicit true anomaly through the exact E relations and integrates with the
  mean-to-eccentric jacobian. This route returns both the secular mean and the
  periodic part, the latter carrying the jacobian and convertible back to the
  true anomaly for the generating-function quadrature.

The two routes agree on the secular mean (cross-validated to machine-exact
symbolic equality). The practical difference is the periodic part: the
eccentric-anomaly route produces it, the Hansen route does not.

## Quick Start

### Example: J2 Frozen Orbit Theory (Closed-Form)

```maxima
/* Load the pipeline */
load("lietransformations.mac")$
load("derivepotential.mac")$
load("astrodata.mac")$

/* Set up canonical dependencies for closed-form mode */
setup_canonical_dependencies()$

/* Derive the J2 zonal potential */
U2: derive_zonal_potential(2, mu, r, f, a, e, i, g, R, true)$

/* Solve the Hori-Deprit recurrence to second order */
vars: [l, g, h, L, G, H]$
solve_hori_deprit_recurrence(
    2,                          /* max perturbation order */
    [H0, U2, 0],               /* Hamiltonian list [F0, F1, F2] */
    vars,                       /* canonical variables */
    f,                          /* averaging variable */
    n_mean,                     /* fundamental frequency */
    2,                          /* max order for generating functions */
    true                        /* closed-form mode */
)$

/* Transformed Hamiltonians and generating functions are stored in the
   global arrays f_star[n] and s_gen[n] */
K0: f_star[0]$
K1: f_star[1]$
S1: s_gen[1]$
K2: f_star[2]$
S2: s_gen[2]$
```

### Example: Lagrange Planetary Equations

```maxima
load("planetaryequations.mac")$

/* Compute rates of change under a disturbing function R */
eqs: lagrange_planetary_equations(R_pot, a, e, i, h, g, M, n)$
```

## Limitations

- **Short-period generating function for third-body terms.** Requesting the
  short-period generating function of a third-body perturbation during the
  elimination of the satellite true anomaly f leads to symbolic expressions
  whose conic denominators (1 + e cos f) raised to high powers cause the
  computer-algebra memory to exhaust during the order-two Poisson brackets and
  the generating-function quadrature. In practice the short-period modulation
  of the third-body potential in f has negligible amplitude and is rarely
  propagated, so the recommended usage is to pass the periodic part as zero and
  keep only the secular mean (obtained exactly through either averaging route).
  The secular dynamics, which dominate the long-term evolution, are unaffected.
  Note that with a zero periodic part the mean-to-osculating reconstruction will
  not recover the third-body short-period correction.

- **Averaging routes cover order two.** The eccentric-anomaly and Hansen routes
  are applied as a pre-processing step feeding the Hori-Deprit solver as a
  [secular, periodic] pair. Automatic term-by-term routing of positive radial
  powers inside the solver recursion (needed when cross Poisson brackets raise
  the radial power at order three and above) is planned as future work.

  ## Roadmap

- **Polar-nodal canonical variables.** The elimination of the parallax is
  currently implemented as a symbolic simplification that removes the explicit
  true anomaly while operating in Delaunay variables, with the radial distance
  and true anomaly carried as symbols through the Keplerian chain rule. A
  planned extension is to promote the polar-nodal set (r, theta, nu, R, Theta, N)
  to an explicit canonical system, following the Deprit (1981) formulation used
  by the Zaragoza school (Abad, Elipe, Deprit). This would express the parallax
  elimination as a genuine canonical transformation in its natural variables and
  enable the standard radial intermediary and its higher-order theories.

- **Internal routing of positive radial powers.** As noted in Limitations, the
  eccentric-anomaly and Hansen averaging routes are applied as pre-processing.
  Automatic term-by-term routing inside the Hori-Deprit recursion, required when
  cross Poisson brackets raise the radial power at third order and above, is
  planned as future work.

## Companion Package

`symcelmech` is designed to work alongside [CelestialMechanics.jl](https://github.com/guilhermedeolivpaes/CelestialMechanics.jl), a high-performance numerical toolkit in Julia for orbital propagation and validation of analytical theories. Together they form a hybrid symbolic-numerical ecosystem for mission design and perturbation analysis.

## Citation

If you use `symcelmech` in your research, please cite the accompanying paper (under review):

```bibtex
@article{paes2026symcelmech,
  author  = {de Oliveira Paes, Guilherme and Berton, Lilian and Vilhena de Moraes, Rodolpho},
  title   = {A Hybrid Symbolic-Numerical Framework for Artificial Satellite 
             Theory and Dynamics using Maxima and Julia},
  year    = {2026},
  note    = {Submitted to Springer Nature}
}
```

## Acknowledgements

Parts of the documentation, code comments, test scripts, and some code optimizations were developed with assistance from AI language models (Claude/Anthropic and Gemini/Google) and reviewed by the author. All scientific content, mathematical formulations, and architectural decisions are the author's own work.

## Author

**Guilherme de Oliveira Paes**

- Mathematician
- MSc in Orbital Dynamics — National Institute for Space Research (INPE)
- PhD candidate in Computer Science — Federal University of São Paulo
- Contact: oliveira.guilherme1643@gmail.com / guilherme.paes@unifesp.br

## Repository Structure

```
symcelmech/
├── symcelmech.mac              # entry point (load this)
├── astrodata.mac               # physical constants database
├── canonicalsetup.mac          # canonical dependency setup (gradef/depends)
├── keplerkinematics.mac        # anomaly relations, conic, jacobians, celmec_trigexpand
├── celmechseries.mac           # elliptic expansion series
├── postprocessing.mac          # sanitization, deatomization, Julia export
├── poissonbracket.mac          # Poisson brackets (series and closed-form)
├── perturbationtools.mac       # quadrature, parallax elimination, potential preparation
├── potentialmodels.mac         # perturbing potential derivation, triaxial harmonics
├── average.mac                 # Hansen coefficients, orbital averaging
├── planetaryequations.mac      # Lagrange planetary equations (classical and nonsingular)
├── hamiltoniansystems.mac      # Hamilton equations, canonical transformations
├── lietransformations.mac      # Hori-Deprit solver, Lie corrections
├── rtest_symcelmech.mac        # regression test suite
├── README.md
├── LICENSE
└── .gitignore
```

## Submitting to maxima-packages

This repository is structured for submission to the official [maxima-packages](https://github.com/maxima-project-on-github/maxima-packages) community repository. To submit:

1. Fork `maxima-project-on-github/maxima-packages`
2. Create a `symcelmech/` subdirectory in your fork
3. Copy all `.mac` files and the `rtest_symcelmech.mac` test script
4. Open a pull request

## License

This project is licensed under the [GNU General Public License v2.0](LICENSE).