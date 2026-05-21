# Instances and Solutions for Team Formation and Routing for Airport Baggage Handling Tasks

This repository contains the test instances and best-found solutions for the computational study presented in:

> Andreas Hagn, Rainer Kolisch, Giacomo Dall'Olio, Stefan Weltge (2026): *A Branch-Price-Cut-And-Switch Approach for
> Optimizing Team Formation and Routing for Airport Baggage Handling Tasks with Stochastic Travel Times.*
> arXiv preprint, DOI: [10.48550/arXiv.2405.20912](https://doi.org/10.48550/arXiv.2405.20912)

The corresponding solver implementation (BPC&S algorithm) is available in [this repository](https://github.com/andreashagntum/BC_APR).

If you use this data, please cite:
```bibtex
@article{hagn2026bpcs,
  title   = {A Branch-Price-Cut-And-Switch Approach for Optimizing Team Formation and Routing for Airport Baggage
   Handling Tasks with Stochastic Travel Times},
  author  = {Hagn, Andreas; Kolisch, Rainer; Dall'Olio, Giacomo; Weltge, Stefan},
  journal = {arXiv preprint},
  year    = {2026},
  doi     = {10.48550/arXiv.2405.20912}
}
```

---

## Repository Structure

```
├── Deterministic/
│   └── ...                        # Best-found solutions under deterministic travel time assumptions
├── Stochastic/
│   └── worker-quantile<$\gamma$>/
│       └── <hor>min-<fph>fph/
│           └── branches-<bool>_dmp-<bool>_cuts-<int>/
│               └── <hor>min-<fph>fph-<modes>_<i>.json
├── (Parallelized stochastic) Results computational study.csv
└── (Parallelized deterministic) Results computational study.csv
```
File `Instances.rar` contains all instances used in the aforementioned paper. File `Solutions.rar` contains the corresponding 
solutions.

---

## Instance and File Naming

Filenames and directory paths encode key instance properties. For example:

```
Stochastic/worker-quantile0.9/60min-10fph/branches-False_dmp-False_cuts-0/60min-10fph-sif_155.json
```

encodes the following:

| Component | Meaning                                                                         |
|---|---------------------------------------------------------------------------------|
| `worker-quantile0.9` | Distributional quantile $\gamma$ used in workforce constraints (8) of the paper |
| `60min` | Planning horizon of 60 minutes                                                  |
| `10fph` | 10 flights per hour                                                             |
| `sif` | Available execution modes: [s]low, [i]ntermediate, [f]ast                       |
| `155` | Unique instance iterator                                                        |
| `branches-False` | Branching on task finish times disabled                                         |
| `dmp-False` | No switch to DMP when disaggregated-infeasible solutions are found              |
| `cuts-0` | No Gomory cuts added                                                            |

The mode string indicates which subset of team formations was available for each flight. Possible values are `s` (slow only), `sf` (slow and fast), `sif` (slow, intermediate, and fast), or `i` (intermediate only).

---

## Solution File Format

Each solution file is a JSON dictionary with one key per tour included in the best-found solution. For each tour, the following fields are stored:

| Field | Description                                                                                 |
|---|---------------------------------------------------------------------------------------------|
| `leave` | Depot leave time|
| `quantile_return` | Depot return time (in the $\gamma$-quantile scenario $\omega_{\gamma}$)                     |
| `formation_w_d` | Team formation used, including worker downgrading                                           |
| `formation` | Unique string identifier of `formation_w_d`                                                 |
| `skill_comp` | Skill composition used to assemble the team                                                 |
| `skill_comp_cnt` | Number of workers per skill level in the skill composition                                  |
| `tasks` | Ordered sequence of tasks performed by the team                                             |
| `tw_viol_prob` | Theoretical probability of violating each task's time window                                |
| `worst_case_start_time` | Latest start time of each task under worst-case travel times                                |
| `quantile_finish_time` | Finish time of each task in the $\gamma$-quantile scenario $\omega_{\gamma}$                |

---

## Results Files

Aggregate results, bounds, runtimes, and further KPIs are stored in two CSV files (`;` as column separator, `,` as decimal delimiter):

- **`(Parallelized stochastic) Results computational study.csv`** — stochastic instances
- **`(Parallelized deterministic) Results computational study.csv`** — deterministic instances

### Column Reference

| Column | Description                                                                                                                             |
|---|-----------------------------------------------------------------------------------------------------------------------------------------|
| `alpha` | Minimum service level required at each task                                                                                             |
| `worker_quantile` | Quantile level $\gamma$ used for workforce occupation durations                                                                         |
| `tw_viol` | Difference between latest extended finish times and latest finish times (in time steps)                                                 |
| `hor` | Planning horizon length (in minutes)                                                                                                    |
| `fph` | Flights per hour                                                                                                                        |
| `i` | Instance iterator                                                                                                                       |
| `rs` | Worker strength (fraction of the workforce needed to trivially solve the instance)                                                      |
| `modes` | Available execution modes: [s]low, [i]ntermediate, [f]ast                                                                               |
| `no_gomory_cuts` | Maximum number of Gomory cuts added                                                                                                     |
| `used_dmp` | If `True`, the solver switched to the DMP formulation upon finding disaggregated-infeasible solutions; otherwise trivial cuts were used |
| `is_feasible` | Whether at least one feasible solution was found                                                                                        |
| `is_optimal` | Whether the best solution found is proven optimal                                                                                       |
| `proven_infeasibility` | Whether infeasibility was proven (always `False` for feasible instances)                                                                |
| `runtime` | Total runtime of the BPC&S algorithm plus any fallback heuristic (in seconds)                                                           |
| `gap_percent` | Optimality gap in %                                                                                                                     |
| `net_ub` | Net upper bound                                                                                                                         |
| `net_lb` | Net lower bound                                                                                                                         |
| `net_lb_root` | Net lower bound at the root node                                                                                                        |
| `gap_root_percent` | Optimality gap at the root node in % (calculated ex-post once an integer feasible solution is known)                                    |
| `runtime_root` | Runtime of solving the root node (in seconds)                                                                                           |
| `nr_iterations_cg` | Number of column generation iterations                                                                                                  |
| `runtime_cg` | Total time spent in column generation, i.e., solving the pricing problem (in seconds)                                                   |
| `nodes_explored` | Number of nodes solved in the branching tree                                                                                            |
| `amp_nodes_explored` | Number of nodes solved using the AMP formulation                                                                                        |
| `dmp_nodes_explored` | Number of nodes solved using the DMP formulation                                                                                        |
| `cg_iteration_cnt` | Redundant (same as `nr_iterations_cg`, but retained for backwards compatibility)                                                        |
| `branches_task_finish_times` | Number of branches created by the task finish time branching rule                                                                       |
| `branches_vehicles` | Number of branches created by the vehicle count branching rule                                                                          |
| `branches_variables` | Number of branches created by the variable branching rule                                                                               |
| `gc_added_cnt` | Number of Gomory cuts added at the root node                                                                                            |
| `tw_violation_cnt` | Number of tasks whose time windows are violated with nonzero probability                                                                |
| `label_dominance_percent` | Percentage of labels dominated during the pricing step                                                                                  |
| `dmp_needed` | Whether the DMP was used at least once (always `False` if `used_dmp` is `False`)                                                        |
| `no_infeas_aggr_solutions` | Number of disaggregated-infeasible solutions found                                                                                      |
| `no_feas_aggr_solutions` | Number of disaggregated-feasible solutions found                                                                                        |
| `inst_had_aggr_infeas_sols` | Whether at least one disaggregated-infeasible solution was encountered                                                                  |

The deterministic results file additionally contains:

| Column | Description |
|---|---|
| `tt_type` | Type of deterministic travel time assumed: `best`, `mean`, `median`, or `worst` |

---

## Instance Generation

Instances were generated using the generator developed by Dall'Olio and Kolisch (2023) based on real flight schedules and aircraft data from Munich Airport. The following parameters were varied:

- **Planning horizon**: 60, 90, or 120 minutes
- **Flights per hour**: 10, 20, or 30
- **Available modes**: slow only (`i`), slow + fast (`sf`), or all three (`sif`)
- **Worker strength** (`rs`): 0.1 to 0.9 (as a fraction of the workforce required to execute each task at its earliest possible start time with the fastest available mode)
- **Worker quantile** $\gamma$: varied in `[0, 0.1, 0.3, 0.5, 0.7, 0.9, 1]`

Five random flight schedules were generated for each parameter combination, yielding a total of 1,215 instances per worker
quantile. Travel time distributions were derived from real-world ground vehicle data collected at Terminal 2 of Munich Airport throughout 2024, fitted using a quadratic regression model. All instances assume an intra-day on-peak time on a sunny weekday during holiday season. The minimum service level is set to α = 0.9, and the extended latest finish time is LF^e_i = LF_i + 5 time steps (i.e., 10 minutes).

**Note**: the worker quantile is not part of the instance .json files. Instead, it is computed ex-post by the solver.

For a detailed description of instance generation, see Section 6.1 and Appendices E–F of Dall'Olio and Kolisch (2023), and Section 6.1 of Hagn et al. (2026).

---

## Requirements

No special software is required to read the instance and solution files. All files are in standard JSON or CSV format.

To reproduce the results using the BPC&S solver, see the [code repository](https://github.com/andreashagntum/BC_APR) for installation instructions and usage examples.
