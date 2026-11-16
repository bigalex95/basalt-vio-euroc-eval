# Evaluating Basalt VIO on EuRoC

A reproducible Docker + [evo](https://github.com/MichaelGrupp/evo) harness that runs the upstream [Basalt](https://github.com/VladyslavUsenko/basalt) visual-inertial odometry on two EuRoC MAV sequences (MH_01_easy, V1_03_difficult) and reports ATE/RPE.

Technical report: [docs/technical_report.pdf](docs/technical_report.pdf) ([Markdown](docs/technical_report.md)).

![Basalt VIO running on MH_01_easy](results/visual_proof.png)

_Basalt running on MH_01_easy. Left: camera input with tracking status. Right: estimated trajectory._

## What is mine and what is upstream

| Part | Source |
| --- | --- |
| VIO algorithm (optical flow, square-root keyframe BA, marginalization) | Upstream Basalt (BSD-3-Clause), used unmodified as a git submodule in `external/basalt` |
| Docker environments (`docker/`), dev container, compose services | This repo |
| Camera/IMU calibration for EuRoC (`configs/my_euroc_calib.json`) | This repo |
| Run and dataset scripts (`scripts/run_vio_tests.sh`, `scripts/download_euroc.sh`) | This repo |
| Evaluation with evo, plots, statistics (`scripts/evaluate_trajectories.py`, `evaluate_results.sh`) | This repo |
| Literature review, setup notes, evaluation write-up, loop-closure proposal (`docs/`) | This repo |

No VIO code was written or modified here. The accuracy numbers below are Basalt's performance under this setup.

## Results

Measured with evo after SE(3) alignment without scale correction (stereo gives metric scale). Source: [results/evaluation/all_results.json](results/evaluation/all_results.json).

| Sequence | ATE RMSE | ATE mean | RPE RMSE | Duration | Poses |
| --- | --- | --- | --- | --- | --- |
| MH_01_easy | 0.069 m | 0.064 m | 0.023 m | 181.9 s | 3,638 |
| V1_03_difficult | 0.054 m | 0.050 m | 0.035 m | 104.7 s | 2,094 |

Each sequence was run once; no repeated runs or variance are reported.

| MH_01_easy: 3D trajectory | MH_01_easy: ATE over time |
| :---: | :---: |
| ![](results/evaluation/mh_01_easy/trajectory_3d.png) | ![](results/evaluation/mh_01_easy/ate_over_time.png) |

| V1_03_difficult: 3D trajectory | V1_03_difficult: ATE over time |
| :---: | :---: |
| ![](results/evaluation/v1_03_difficult/trajectory_3d.png) | ![](results/evaluation/v1_03_difficult/ate_over_time.png) |

More plots: [results/evaluation/](results/evaluation/), [results/plots/](results/plots/).

### Published numbers for context

The values below are **quoted from the literature, not run here**, and may use different alignment, settings and sequence handling, so they are not a like-for-like comparison.

| Method | MH_01 ATE | V1_03 ATE | Note |
| --- | --- | --- | --- |
| VINS-Mono | 0.16 m | 0.30 m | as reported in papers |
| ORB-SLAM2 | 0.06 m | 0.14 m | as reported in papers, with loop closure |

## Getting started

Prerequisites: Docker with Compose, about 2 GB for datasets and 4 GB for images. Clone with the submodule:

```bash
git clone --recurse-submodules https://github.com/bigalex95/VIO-SLAM-Assignment
cd VIO-SLAM-Assignment
./scripts/download_euroc.sh      # datasets into data/ (or download manually, see script)
```

Expected layout: `data/MH_01_easy/mav0/{cam0,cam1,imu0,state_groundtruth_estimate0}` and the same for `data/V1_03_difficult/`.

Automated pipeline (build Basalt, run VIO on both sequences, evaluate, plot):

```bash
cd docker
docker-compose up run
```

Interactive development container (X11 GUI possible):

```bash
cd docker
docker-compose up -d dev
docker-compose exec dev bash
# inside: cd /workspace/external/basalt && cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release && cmake --build build
# then:   /workspace/scripts/run_vio_tests.sh [--gui]
```

Re-run only the evaluation (needs `pip install -r requirements.txt`):

```bash
python scripts/evaluate_trajectories.py --dataset all     # or MH_01_easy / V1_03_difficult
```

The pinned Basalt commit is the submodule revision (`git -C external/basalt log --oneline -1`). More Docker details: [docker/README.md](docker/README.md).

## IMU parameters

`configs/my_euroc_calib.json` holds the camera/IMU calibration; IMU noise values are taken from the ADIS16448 datasheet and the EuRoC specification.

## Project structure

```
configs/       camera/IMU calibration (my_euroc_calib.json)
docker/        Dockerfile (pipeline), Dockerfile.dev, docker-compose.yml
docs/          technical report (md, pdf) and phase A-D notes
external/      basalt (git submodule, upstream)
results/       evaluation metrics, plots, stats, EuRoC ground truth as CSV
scripts/       dataset download, VIO run, evo evaluation, report/figure generation
```

## Limitations

- Two EuRoC sequences only; no other datasets, no repeated runs.
- VIO only: no loop closure or relocalization (a Bag-of-Words loop-closure proposal is in `docs/phase_d_improvement_proposal.md`, not implemented).
- Not validated on embedded hardware, outdoors, or in low-light scenes.
- `results/groundtruth/*.csv` are EuRoC ground truth converted for evo; the dataset itself is not included.
- No automated tests or CI.
- Some files in `docs/` and `results/evaluation/README.md` were written in an earlier, more promotional tone; the tables in this README are the reference.

## Credits

- [Basalt](https://github.com/VladyslavUsenko/basalt) by Usenko, Demmel, Schubert, Stueckler, Cremers (CVPR/RA-L 2020), BSD-3-Clause.
- [EuRoC MAV dataset](https://doi.org/10.1177/0278364915620033) by Burri et al., IJRR 2016.
- [evo](https://github.com/MichaelGrupp/evo) by Michael Grupp.

```bibtex
@article{usenko2020basalt,
  title   = {Visual-Inertial Mapping with Non-Linear Factor Recovery},
  author  = {Usenko, Vladyslav and Demmel, Nikolaus and Schubert, David and St{\"u}ckler, J{\"o}rg and Cremers, Daniel},
  journal = {IEEE Robotics and Automation Letters},
  year    = {2020}
}
```

## License

See [LICENSE](LICENSE). Basalt keeps its own license in `external/basalt`.
