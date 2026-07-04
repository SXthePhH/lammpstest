# `mid` Integrator Unittest Report

Date: 2026-06-26

## Executive Summary

The `mid` integrator related timestepping unittests passed in the local `build_prcheck` build.

Current verification result:

| Item | Result |
| --- | --- |
| Test set | `FixTimestep:.*mid` |
| Build directory | `build_prcheck` |
| Number of YAML test cases | 14 |
| Latest live rerun result | `100% tests passed, 0 tests failed out of 14` |
| Historical full log result | `Test Passed` for all 14 cases in `build_prcheck/Testing/Temporary/LastTest.log` |

## Evidence

### 1. Live rerun used for this report

```bash
/bin/bash -lc 'source /opt/intel/oneapi/setvars.sh >/tmp/oneapi_setvars_ctest_mid.log 2>&1 && export I_MPI_FABRICS=shm && export FI_PROVIDER=tcp && ctest --output-on-failure -R "FixTimestep:.*mid"'
```

Observed result:

```text
100% tests passed, 0 tests failed out of 14
Total Test time (real) =   4.15 sec
```

### 2. Historical CTest log in the same build

File: `build_prcheck/Testing/Temporary/LastTest.log`

The log shows that on `Jun 20 20:42 CST` all 14 `FixTimestep:*mid*` cases ended with `Test Passed.`

## What This Test Framework Checks

According to the LAMMPS timestepping-fix unittest workflow, each YAML test checks the following behavior:

| Checked behavior | Meaning |
| --- | --- |
| Short MD trajectory reproducibility | The fix produces the expected coordinates and velocities for the reference system. |
| Restart continuity | The system is run, restarted, and continued; the continued trajectory must still match the reference. |
| Fix global outputs | The fix global scalar and global vector are compared with stored reference values. |
| Thermostat target extraction | If the fix exposes `t_target`, that value is also checked against the reference. |
| Per-atom mass path | The framework repeats the second stage with per-atom masses, so `mass` and `rmass` paths are both exercised. |

In practical terms, these tests show that the implemented `mid` fix behavior is correct for the covered input combinations, and that restart state handling is working as expected.

## Coverage Matrix

| Test case | Main coverage point | Pass status |
| --- | --- | --- |
| `FixTimestep:nvt_mid` | Base `nvt/mid` path with default NH thermostat and `middle` ordering | Passed |
| `FixTimestep:nvt_mid_langevin` | Particle Langevin thermostat path for `nvt/mid` | Passed |
| `FixTimestep:nph_mid` | Base `nph/mid` pressure-control path | Passed |
| `FixTimestep:nph_mid_tri_langevin_side` | Triclinic `nph/mid` with Langevin barostat and `side` ordering | Passed |
| `FixTimestep:npt_mid` | Base `npt/mid` iso NPT path | Passed |
| `FixTimestep:npt_mid_tri` | Triclinic NPT path | Passed |
| `FixTimestep:npt_mid_iso_mtk_no` | MTK-off branch | Passed |
| `FixTimestep:npt_mid_aniso_couple_none` | Anisotropic pressure control with `couple none` | Passed |
| `FixTimestep:npt_mid_aniso_couple_xyz` | Coupled anisotropic pressure control with `couple xyz` | Passed |
| `FixTimestep:npt_mid_aniso_couple_yz_langevin` | `couple yz` plus Langevin barostat branch | Passed |
| `FixTimestep:npt_mid_aniso_couple_xz_langevin_side` | `couple xz` plus Langevin barostat and `side` ordering | Passed |
| `FixTimestep:npt_mid_x_nh_langevin_side` | Mixed NH thermostat plus Langevin barostat, `x` pressure control, `side` ordering | Passed |
| `FixTimestep:npt_mid_y_langevin_nh` | Mixed Langevin thermostat plus NH barostat, `y` pressure control | Passed |
| `FixTimestep:npt_mid_z_langevin_langevin_side` | Particle Langevin plus barostat Langevin, `z` pressure control, `side` ordering | Passed |

## What These Passing Tests Demonstrate

| Conclusion | Supported by unittest? | Notes |
| --- | --- | --- |
| `nvt/mid`, `nph/mid`, `npt/mid` are registered and runnable | Yes | Covered by the 14 passing `FixTimestep:*mid*` cases. |
| New keywords such as `thermostat`, `barostat`, `integrator`, `seed`, `zero` reach the intended code paths | Yes, for covered combinations | Proven only for the combinations present in the YAML suite. |
| `middle` and `side` operator ordering both work in tested cases | Yes | Both orderings are explicitly covered. |
| NH and Langevin branches both work in tested cases | Yes | Particle thermostat and barostat Langevin paths are both exercised. |
| Pressure styles `iso`, `aniso`, and `tri` work in tested cases | Yes | All three are covered. |
| Restart write/read state is consistent | Yes | This is one of the main checks performed by `test_fix_timestep`. |
| All possible user input combinations are correct | No | The suite is strong, but not exhaustive. |
| Long-time ensemble sampling correctness is fully proven | No | These are short regression-style unittests, not long statistical validation. |
| `respa` support is covered | No | The `mid` YAML inputs use `skip_tests: respa`. |
| OMP/Kokkos accelerated variants are covered in this build | No | In the recorded test log, `FixTimestep.omp` and `FixTimestep.kokkos_omp` were skipped. |

## Recommended Reporting Wording

Suggested one-sentence version:

> I completed the `mid` integrator unittest validation in the LAMMPS timestepping-fix framework; all 14 `mid`-related YAML tests passed, covering `nvt/mid`, `nph/mid`, `npt/mid`, NH/Langevin branches, `middle/side` ordering, multiple pressure-coupling modes, and restart consistency.

Suggested more careful version:

> The current unittest suite gives strong regression evidence that the implemented `mid` fixes behave correctly for the covered input combinations and preserve restart continuity, while `respa`, accelerated variants, and long-time statistical correctness still require separate validation if needed.
