# Duke Humanoid V2

**A 31-DoF quasi-direct-drive humanoid whose two RGB-D cameras aim independently of each
other and of torso heading — so it can watch two separated work regions while both arms
reach into them.**

[Simulation & training](https://github.com/generalroboticslab/duke_humanoid_v2_simulation) ·
[Onboard control stack](https://github.com/generalroboticslab/duke_humanoid_v2_deploy) ·
[Hardware](#hardware) · [Quick start](#quick-start)

<!-- Add [Paper] / [Project page] / [Video] to the row above once the arXiv preprint is posted. -->

![Duke Humanoid V2](media/teaser.png)

## Why the cameras move

Reachability alone does not tell you where a robot can *work*. A target can be
kinematically reachable and still invisible to the cameras at the arm configurations
that reach it, which forces the robot to reorient its body just to look. We measure this
directly as the **visible-reachable workspace** (VRW) and design the sensing layout
against it.

On this robot, actuating the cameras moves visible-reachable coverage from **38% to 97%**
of the reachable volume — the same arms, the same body, only the cameras freed. A second
independent camera lifts pairwise coverage of two separated targets from **0.45 to 0.95**;
a third only reaches 0.97, for +0.58 kg, two more gimbal DoF, and about \$600. So two.

Across existing humanoids under the same geometric evaluation, visible-reachable coverage
runs from 16% (fixed-head Unitree G1) to 48–76% for actuated-neck platforms — Talos, T1,
GR-3, Apollo. None of them can aim two views independently.

![Visible-reachable workspace across platforms](media/workspace.png)

## Results

| | |
| --- | --- |
| Visible-reachable coverage, cameras fixed → actuated | 38% → **97%** |
| Pairwise coverage η₂, one → two actuated cameras | 0.45 → **0.95** (three: 0.97) |
| Two-target reach-and-grasp, vs. same robot cameras fixed | **−17%** completion time, **−19%** mechanical energy |
| Hardware | front/back and left/right pairs grasped with no torso reorientation |

The simulation benchmark is 6 scenarios × 3 repeats × 10 seeded layouts = 900 trials, and
it reproduces from this release.

## Hardware

![Duke Humanoid V2 hardware](media/hardware.png)

Orange numbers are actuated joints: (1–7) shoulder pitch/roll/yaw, elbow, wrist roll/pitch/yaw,
(8) waist, (9–14) hip pitch/roll/yaw, knee, ankle pitch/roll, (15–16) camera yaw/pitch. Green
labels are modules: (I) camera, (II) gripper, (III) onboard computer. Dimensions in mm.

| | |
| --- | --- |
| DoF | 31: 27-DoF body (waist ×1, legs 2×6, arms 2×7) + two 2-DoF camera gimbals, +1 per gripper |
| Mass / height | 36 kg / 1.2 m |
| Arm reach / leg length | 0.46 m / 0.39 m |
| Cameras | 2 × Intel RealSense D436, 90°×65° RGB FoV, 0.1–3.0 m, each on its own yaw-pitch gimbal |
| End effectors | parallel grippers, 324 g each, one mimic-coupled jaw slide |
| Actuation | quasi-direct-drive throughout |
| Control | 50 Hz learned whole-body policy onboard, 244 Hz CAN motor loop |

MJCF, meshes, camera modules, and gripper are in
[`simulation/asset/duke_v2/`](https://github.com/generalroboticslab/duke_humanoid_v2_simulation/tree/main/asset/duke_v2).

## Quick start

```bash
git clone --recurse-submodules git@github.com:generalroboticslab/duke_humanoid_v2.git
cd duke_humanoid_v2/simulation
pip install -r requirements.txt          # plus nvidia-curobo, see simulation/README.md
```

Regenerate the workspace figures — seconds, from shipped caches, no GPU sweep and no
training:

```bash
MUJOCO_GL=egl python mj_envs/asset_zoo/reachability_study/plot_workspace_curobo.py --reach-visible-compare
python mj_envs/asset_zoo/reachability_study/camera_count_ablation.py
```

Train the whole-body policy, or watch a shipped checkpoint:

```bash
python mj_envs/run.py train --task HumanoidRmaVelEstArmFlashSacv2ybsk_yaw_s4MixedArmsCam
python mj_envs/run.py play  --task HumanoidRmaVelEstArmFlashSacv2ybsk_yaw_s4MixedArmsCam
```

Everything needs an NVIDIA GPU. Full reproduction instructions, including the 900-trial
benchmark and the checkpoint provenance table, are in
[`simulation/README.md`](https://github.com/generalroboticslab/duke_humanoid_v2_simulation#readme).

## The two repositories

Split along the line that matters in practice: what runs in simulation, and what runs on
the robot. They are separate repositories, not directories, so each keeps its own history
and issue tracker. Training exports a policy; `deploy/` runs the export.

| | what it is |
| --- | --- |
| [`simulation/`](https://github.com/generalroboticslab/duke_humanoid_v2_simulation) | Policy training and the paper's reproduction package: the VRW study, the two-target reach-and-grasp benchmark, robot assets, and the checkpoints behind the reported numbers. |
| [`deploy/`](https://github.com/generalroboticslab/duke_humanoid_v2_deploy) | The onboard control stack: the 50 Hz policy loop, perception bridge, cuRobo planning client, gripper service, and the autonomous operator. |

Already cloned without the submodules:

```bash
git submodule update --init --recursive
```

## Running the hardware

Read [`deploy/control/docs/OPERATIONS.md`](https://github.com/generalroboticslab/duke_humanoid_v2_deploy/blob/main/control/docs/OPERATIONS.md)
first. That stack moves a 36 kg machine with people next to it, and its safety gates have
incidents behind them —
[`auto_operator_incidents.md`](https://github.com/generalroboticslab/duke_humanoid_v2_deploy/blob/main/control/docs/auto_operator_incidents.md)
lists each failure alongside the "simplification" that would bring it back.

## License

Apache-2.0, both repositories. Third-party robot models keep their upstream licenses
alongside their assets.
