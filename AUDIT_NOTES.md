# Scientific code audit notes — review before public release

**Status:** Initial code-sharing draft, preserving the six supplied scripts.
No scientific correction has been applied to these originals.

## A. Critical: collective movement bug in Supplementary Figs. 24–27

In `sfig24_phi_0p015.py` through `sfig27_phi_0p125.py`, the cluster proposal
checks trial coordinates for each particle. After this check, however, it uses:

```python
if valid_move:
    for i in cluster:
        new_positions[i] = new_pos
```

`new_pos` is only the last particle's proposed coordinate. Thus, all members
of a cluster containing more than one particle are assigned the same location.
This creates overlaps and typically leads to rejected proposed cluster moves
due to the large repulsive-energy penalty.

A candidate repair is to compute and validate **every** translated position:

```python
trial = (positions[cluster] + delta_pos) % L
if all(is_in_active_zone(p) for p in trial):
    new_positions[cluster] = trial
    new_energy = compute_energy(new_positions, dipoles, H)
    if new_energy <= energy or np.random.rand() < np.exp(-(new_energy - energy)):
        positions = new_positions
        energy = new_energy
```

This repair must be **re-run and compared to the manuscript figures**.
Do not silently claim that the original and repaired results are identical.

## B. High-density implementation differs from the others

The `sfig28_phi_0p25.py` script switches to a neighbor-list energy method.
Two aspects require investigation:

1. `compute_total_energy()` adds the field energy once for each particle
   and then divides the whole sum by two, unintentionally halving the
   total field-energy contribution in the stored total-energy value.
   In this script, Metropolis acceptance for individual/cluster moves uses
   local old/new energies, so this bookkeeping problem does not *by itself*
   prove that accepted trajectories are different.
2. The neighbor list is constructed with `r < r_coff` and refreshed only every
   100 steps. It has no extra Verlet skin and is not rebuilt for each
   proposed configuration. Newly close pairs can therefore be missing from
   trial-energy calculations, potentially affecting acceptance decisions.

Before relying on these results, validate local-energy differences against a
direct full-pair energy evaluation for representative proposed moves.

## C. Active population is not constant across concentrations

The 50x50 box contains four 10x10 active squares (16% of box area).
With the original seed and non-overlap initialization:

| Figure | N | Initial active count |
|---|---:|---:|
| S24 | 50 | 7 |
| S25 | 100 | 12 |
| S26 | 200 | 29 |
| S27 | 400 | 55 |
| S28 | 800 | 125 |

The Supplementary Information should not describe 120–140 active particles
as a property of all five concentration cases. That magnitude applies
only to the highest-loading case.

## D. Output and manuscript consistency

- The scripts write snapshots and an Excel workbook, **not** chain-length
  distribution or PDI source data. Provide the actual downstream data
  analysis if Fig. 6 statistical results depend on it.
- `supplementary_movie_1.py` writes PNG frames but not a ready-to-submit
  video file. Preserve and describe the actual video-encoding procedure.
- Step 0 is saved after the first update attempt, not directly from the
  pristine initialized configuration.
- The confined algorithms use square active zones; experiments involve
  patterned cavities of different geometry. This approximation should
  be acknowledged when comparing simulation and experiment.
- `Phi` is total-box 2D **area fraction**, not 3D volume fraction and not
  area fraction within the active squares.
- Random seed 42 generates one realization per script. Report whether
  additional independent seeds/runs and uncertainty estimates were used
  for the manuscript's statistical plots.
- A Monte Carlo iteration is not an experimentally measured time step.

## E. Release checklist

- [ ] Correct cluster move implementation in S24–S27 and rerun.
- [ ] Validate and, if necessary, fix S28 neighbor-list energy handling.
- [ ] Supply the statistics/PDI analysis code and source data.
- [ ] Reconcile SI parameters and terminology with actual scripts.
- [ ] Reproduce panels using the intended version of the code.
- [ ] Pin tested Python/dependency versions.
- [ ] Choose an open-source license.
- [ ] Create a fixed GitHub release and optionally archive on Zenodo.
