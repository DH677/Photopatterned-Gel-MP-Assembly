# Monte Carlo Simulations of Magnetic-Particle Assembly in Photopatterned Gels

This repository collects the original Python Monte Carlo simulation scripts associated
with the manuscript **"Photopatterned Gel Confinement Enables Programmable Assembly
of Magnetic Particles"**.

The files were supplied as `.txt` and are preserved **without changes to the
simulation logic**, with the extension changed to `.py`. The repository is an
**initial code-sharing draft**: see [AUDIT_NOTES.md](AUDIT_NOTES.md) for issues
that require investigation before describing the simulations as fully reproducible.

## 1. Code-to-figure correspondence

| Case | Original script | Particles (N) | Calculated area fraction (Phi) | Active particles at initialization |
|---|---|---:|---:|---:|
| Supplementary Movie 1 | `original_scripts/supplementary_movie_1.py` | 200 | 0.06283 | 200 (unconfined) |
| Supplementary Fig. 24 | `original_scripts/sfig24_phi_0p015.py` | 50 | 0.01571 | 7 |
| Supplementary Fig. 25 | `original_scripts/sfig25_phi_0p031.py` | 100 | 0.03142 | 12 |
| Supplementary Fig. 26 | `original_scripts/sfig26_phi_0p062.py` | 200 | 0.06283 | 29 |
| Supplementary Fig. 27 | `original_scripts/sfig27_phi_0p125.py` | 400 | 0.12566 | 55 |
| Supplementary Fig. 28 | `original_scripts/sfig28_phi_0p25.py` | 800 | 0.25133 | 125 |

The area fraction in the original scripts is computed relative to the **entire**
simulation box, not the combined area of the active regions:

`Phi = N * pi * (d/2)^2 / L^2`.

For the confined simulations, the values shown in the last column were
recalculated from the original random initialization using seed 42; these
are initial counts, **not** the number of clusters or chains.

## 2. Model as implemented in the supplied scripts

- Two-dimensional, periodic square simulation box with `L = 50` and particle
  diameter `d = 1`.
- Initial non-overlapping particle positions drawn using NumPy seed `42`.
- Initial dipole directions aligned with the applied field, `H = (1, 0)`.
- Dimensionless particle-particle dipolar coupling `lambda_m = 4.5` and
  field coupling `xi = 80`.
- Pair interaction cutoff `r_coff = 10d` and geometric cluster cutoff `r_c = 1.2d`.
- Single-particle trial displacements drawn independently in each coordinate
  from `[-0.5d, +0.5d]`, with an angular perturbation within `+/-10 degrees`.
- A collective cluster-move proposal is attempted every 70 simulation steps.
- The programmed simulation contains 30,000 iterations. **These are Monte Carlo
  iterations, not experimentally calibrated elapsed time or necessarily full
  N-particle Monte Carlo sweeps.**
- In the five confined scripts, the active zones are four 10 by 10 squares:
  `[10,20] x [10,20]`, `[10,20] x [30,40]`,
  `[30,40] x [10,20]`, and `[30,40] x [30,40]`.
  The four squares cover 400/2500 = 16% of the total box area.
- A particle's active/inactive status is determined at initialization.
  Particles initialized outside the active squares are immobilized.
- The highest-density script uses a neighbor-list implementation that differs
  from the full pair-energy calculation in the other scripts.

The simulation code is the authoritative record of the present implementation.
See the manuscript Supplementary Information for the intended scientific model.
**Please resolve the discrepancies described in AUDIT_NOTES.md before public release.**

## 3. Repository contents

```text
Photopatterned-Gel-MP-Assembly/
├── README.md
├── AUDIT_NOTES.md
├── requirements.txt
├── .gitignore
├── run.py
└── original_scripts/
    ├── supplementary_movie_1.py
    ├── sfig24_phi_0p015.py
    ├── sfig25_phi_0p031.py
    ├── sfig26_phi_0p062.py
    ├── sfig27_phi_0p125.py
    └── sfig28_phi_0p25.py
```

## 4. Installation

Python 3 is required; the exact Python and dependency versions used to generate
the manuscript figures **have not yet been confirmed**.

```bash
python -m venv .venv
```

Activate on Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Alternatively, on macOS or Linux:

```bash
source .venv/bin/activate
```

Then install the declared dependencies:

```bash
python -m pip install -r requirements.txt
```

`requirements.txt` lists imports required by the supplied files but does
not pin historically verified versions. Add a tested version lock before
formal archiving.

## 5. Running the original scripts

Print the available cases without running any simulation:

```bash
python run.py --list
```

Preview the command and output directory:

```bash
python run.py sfig24 --dry-run
```

Execute a case:

```bash
python run.py movie1
python run.py sfig24
python run.py sfig25
python run.py sfig26
python run.py sfig27
python run.py sfig28
```

**Each command is independent; run only the required case.** The `run.py` wrapper
changes only the working/output folder. It does **not** modify the Monte Carlo
algorithm.

Outputs are stored separately under `outputs/<case>/` because the original
scripts all write `snapshot_step_<step>.png` and `snapshot_coordinates.xlsx`.
The scripts also show intermediate plots; simulations with many particles
may take substantial time and create large numbers of files.

The Movie 1 script saves PNG frames every 50 steps; **it does not encode an MP4
video itself**. The confined scripts save snapshots every 1,000 steps.

## 6. What these scripts generate

The original scripts generate:

1. Particle-position PNG snapshots for selected simulation steps.
2. Particle coordinates in an Excel workbook.
3. In confined cases, a distinction between active and inactive particles.

**They do not directly export chain-length distribution or PDI tables/plots.**
Any separately used statistical post-processing scripts and/or source data
for Fig. 6 should be deposited alongside these scripts before submission.

## 7. Reproducibility status

- Source-script Python syntax: checked.
- Input particle counts and nominal area fractions: checked.
- Initial active-particle counts: independently recalculated.
- Full 30,000-step simulations: **not re-executed for this archive**.
- Numerical equivalence to reported figure panels: **not verified**.
- Scientific issues requiring review: listed in `AUDIT_NOTES.md`.

Please do not cite this initial draft as a fully validated reproduction
package until these checks and relevant code fixes have been completed.

## 8. Publication and citation

Associated manuscript: *Photopatterned Gel Confinement Enables Programmable
Assembly of Magnetic Particles* (under preparation/submission).

Add the final publication reference, GitHub release tag, and an archival DOI
(e.g., Zenodo) after validation and deposit.

## 9. License and contact

A license has **not** been selected. The authors should confirm the appropriate
open-source license and add a `LICENSE` file prior to public release.

For scientific inquiries, use the contact information given in the manuscript.
