# symcelmech

An open-source Maxima package for analytical celestial mechanics based on Lie transformations.

`symcelmech` automates the construction of high-order perturbation theories for Hamiltonian systems using the Hori-Deprit method (Lie series). It provides a complete symbolic pipeline — from potential derivation through orbital averaging to generating function computation — supporting both closed-form (exact in eccentricity) and series expansion modes.

## Features

- Automated Hori-Deprit/Mersman recursive Lie triangle up to arbitrary order
- Dual-track integration: closed-form (true anomaly domain) and Poisson series (mean anomaly domain)
- Perturbing potential derivation: zonal harmonics (J_n), tesseral/sectorial harmonics (C_nm, S_nm), third-body perturbations, and solar radiation pressure (SRP)
- Hansen coefficient engine for exact orbital averaging
- Memoized Poisson bracket computation with antisymmetry exploitation
- Mean-to-osculating element transformation via Lie series corrections
- Deprit's elimination of the parallax
- Lagrange, Gauss, and Hamilton planetary equations (classical and non-singular equinoctial)
- Delaunay-Poincare canonical transformations
- Elliptic expansion series (equation of the center, r/a, cos f, sin f, cos E)
- Physical constants database (Earth, Moon, Mercury, Apophis, Didymos)

## Requirements

- [Maxima](https://maxima.sourceforge.io/) (tested with 5.47+)
- Maxima packages: `orthopoly`, `vect` (bundled with standard Maxima distributions)

## Installation

### Option 1: Add to Maxima search path (recommended)

Clone the repository:

```bash
git clone https://github.com/<your-username>/symcelmech.git
```

Add the package directory to your `maxima-init.mac` (created automatically in `~/.maxima/` on Linux/macOS or `%USERPROFILE%/maxima/` on Windows):

```maxima
/* Add this line to your maxima-init.mac */
file_search_maxima: cons("/path/to/symcelmech/", file_search_maxima)$
```

Then, from any Maxima session:

```maxima
load("symcelmech")$
```

### Option 2: Install via maxima-packages

`symcelmech` follows the [maxima-packages](https://github.com/maxima-project-on-github/maxima-packages) convention and can be installed from the community repository (pending acceptance):

```maxima
/* After maxima-packages is set up */
load("symcelmech/symcelmech.mac")$
```

### Option 3: Direct load

If you prefer not to modify `maxima-init.mac`, load the entry point with the full path:

```maxima
load("/path/to/symcelmech/symcelmech.mac")$
```

### Option 4: Load individual modules

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
| `derivepotential.mac` | Perturbing potential derivation: zonal, tesseral, third-body, and SRP |
| `average.mac` | Secular/periodic separation, Hansen coefficient computation, orbital averaging |
| `celmechseries.mac` | Elliptic expansion series and eccentricity truncation tools |
| `algebraicutils.mac` | Memoization, expression sanitization, coordinate substitution rules |
| `planetaryequations.mac` | Lagrange, Gauss, and Hamilton equations; Delaunay-Poincare transformations |
| `astrodata.mac` | Physical and orbital constants for supported celestial bodies |

### Dependency Graph

```
lietransformations.mac
├── analyticalfunctions.mac
├── average.mac
│   └── analyticalfunctions.mac
├── celmechseries.mac
└── algebraicutils.mac
    └── analyticalfunctions.mac

derivepotential.mac
├── analyticalfunctions.mac
└── celmechseries.mac

planetaryequations.mac  (standalone)
astrodata.mac           (standalone)
```

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
result: solve_hori_deprit_recurrence(
    2,                          /* max perturbation order */
    [H0, U2, 0],               /* Hamiltonian list [F0, F1, F2] */
    vars,                       /* canonical variables */
    f,                          /* averaging variable */
    n_mean,                     /* fundamental frequency */
    2,                          /* max order for generating functions */
    true                        /* closed-form mode */
)$

/* Extract transformed Hamiltonians and generating functions */
f_star_list: result[1]$
s_gen_list:  result[2]$
```

### Example: Lagrange Planetary Equations

```maxima
load("planetaryequations.mac")$

/* Compute rates of change under a disturbing function R */
eqs: lagrange_planetary_equations(R_pot, a, e, i, h, g, M, n)$
```

## Companion Package

`symcelmech` is designed to work alongside [CelestialMechanics.jl](https://github.com/guilhermedeolivpaes/CelestialMechanics.jl), a high-performance numerical toolkit in Julia for orbital propagation and validation of analytical theories. Together they form a hybrid symbolic-numerical ecosystem for mission design and perturbation analysis.

## Citation

If you use `symcelmech` in your research, please cite the accompanying paper (under review):

```bibtex
@article{paes2026symcelmech,
  author  = {Paes, Guilherme de Oliveira and Berton, Lilian and Vilhena de Moraes, Rodolpho},
  title   = {A Hybrid Symbolic-Numerical Framework for Artificial Satellite 
             Theory and Dynamics using Maxima and Julia},
  year    = {2026},
  note    = {Submitted to Springer Nature}
}
```

## Acknowledgements

Parts of the documentation (docstrings), code comments, test scripts, and some code optimizations were drafted or suggested with assistance from AI language models (Claude, Anthropic and Gemini, Google) and subsequently reviewed and validated by the author. All scientific content, mathematical formulations, numerical methods, and architectural decisions are the author's own work.

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
├── analyticalfunctions.mac     # Poisson brackets, quadrature, trigonometric tools
├── algebraicutils.mac          # memoization, sanitization, substitution rules
├── astrodata.mac               # physical constants database
├── average.mac                 # Hansen coefficients, orbital averaging
├── celmechseries.mac           # elliptic expansion series
├── derivepotential.mac         # perturbing potential derivation
├── lietransformations.mac      # Hori-Deprit solver, Lie corrections
├── planetaryequations.mac      # Lagrange, Gauss, Hamilton equations
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