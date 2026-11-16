# Evaluation results (quick reference)

Basalt VIO on EuRoC, evaluated with evo (SE(3) alignment, no scale correction). Headline numbers are mirrored from the root [README](../../README.md); raw values are in `all_results.json`.

| Sequence | ATE RMSE | RPE RMSE | Duration |
| --- | --- | --- | --- |
| MH_01_easy | 0.069 m | 0.023 m | 181.9 s |
| V1_03_difficult | 0.054 m | 0.035 m | 104.7 s |

Layout: `all_results.json` (combined), and per sequence `evaluation_results.json`, `trajectory_3d.png`, `trajectory_2d.png`, `ate_over_time.png`, `rpe_over_time.png`, `xyz_errors.png`.

Re-run: `python scripts/evaluate_trajectories.py --dataset all`.

Comparison numbers for VINS-Mono and ORB-SLAM2 appear in the root README only as values quoted from their papers; they were not run here.
