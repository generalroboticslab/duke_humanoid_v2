# Duke Humanoid V2

A 31-DoF quasi-direct-drive humanoid with two independently actuated yaw-pitch RGB-D
cameras. Each camera aims at its own work region, so the robot can watch two separated
places at once without reorienting its torso.

[Simulation & training](https://github.com/generalroboticslab/duke_humanoid_v2_simulation) ·
[Onboard control stack](https://github.com/generalroboticslab/duke_humanoid_v2_deploy) ·
[Hardware](#hardware) · [Quick start](#quick-start)

<!-- Add [Paper] / [Project page] / [Video] to the row above once the arXiv preprint is posted. -->

![Duke Humanoid V2 tracking two moving targets](media/hardware_tracking.webp)

On hardware. Two people stand on opposite sides, each holding a tagged cube and moving it.
One camera goes to each, and both arms track their own target.

![Duke Humanoid V2](media/teaser.png)

In simulation, the same task with four camera setups: two tables, one green cube on each.
Only dual actuated keeps both cubes in view and reaches both. The shaded cones show what
each camera sees.

![Two targets left and right, four camera setups](media/two_target_left_right.webp)

The same four setups, with the tables in front and behind instead of left and right:

![Two targets front and back, four camera setups](media/two_target_front_back.webp)

## Why the cameras move

Reachability tells you where the arm can put the end effector. It does not tell you
whether the robot can see the target once it gets there. A target can sit well inside the
arm workspace and still fall outside every camera view at the configurations that actually
realize the reach. Humanoids inherit this from the human form factor: broad arm
workspaces, vision concentrated in the head or the chest.

The cost does not stay in the perception system. To acquire a view, the robot redirects a
camera, rotates its torso, or walks to a new viewpoint. When the eyes cannot acquire the
workspace, the body moves instead, so a limitation that looks perceptual at design time
becomes whole-body motion at execution time.

The **visible-reachable workspace** (VRW) measures that coupling. Start from the reachable
workspace, then keep only the targets that can also be observed from a configuration that
reaches them. How the joints are partitioned is the part that matters: joints spent
realizing the reach do not count as gaze actuation. A wrist camera on the reaching arm is
therefore not an independent view, while a gimbal that leaves the end-effector pose alone
is.

## What the measure decided

Where the cameras went, and how many, came out of the measure.

**Top of the body, not the front or back faces.** A front- or back-mounted camera covers
one side of the robot. On top, it reaches front, back, left and right.

**No wrist cameras.** A wrist camera on the reaching arm adds no independent gaze
actuation under the partition above. Wrist cameras complement body cameras rather than
replace them.

**Articulation for coverage.** At every camera count, articulation drives single-target
coverage: 38% to 97% here. Once actuated, one module is nearly saturated, and a second or
third adds under 3%.

**Count for two regions at once.** With one camera, both regions have to fall inside the
same unoccluded view, so pairwise coverage falls off as the targets separate. Two
independently actuated cameras each hold their own view, and coverage stays nearly flat:
0.45 to 0.95. A third reaches 0.97, for 0.58 kg, two more gimbal DoF and about \$600.
Hence two.

![Visible-reachable workspace across platforms](media/workspace.png)

Evaluated the same way, other humanoids run from 16% visible-reachable coverage for the
fixed-head Unitree G1 to 48-76% for the actuated-neck platforms: PAL Talos, Booster T1,
Fourier GR-3, Apptronik Apollo. All of them fall below this design on pairwise coverage,
because their cameras share the same neck joints. However wide or steerable that view is,
it is still one viewing direction. GR-3 has the widest field of view in the group and
scores lower for exactly this reason.

## What it buys

Success rates barely move across the five configurations, all between 0.967 and 0.994. The
cost does. Two actuated cameras finish 17% faster than the same robot with them welded,
using 19% less mechanical energy.

Most of that is search time. A welded camera has to rotate the torso, or walk toward the
region until the cube appears. Actuated cameras look. That halves search time at two
cameras and cuts it by 82% at one. Approach time drops too, since a far cube can be found
before the robot starts walking, so it heads for the cube rather than the table.

The second camera is a separate question. One actuated camera covers almost the same
single-target space as two. Two matter when both cubes have to be watched at once, so the
planner can reach for both together.

We ran the dual-actuated build on hardware for all four tabletop scenarios. Each clip below
is four separate trials playing together.

![Left and right tables, close, four hardware trials](media/hardware_close_left_right.webp)

Tables to the left and right, both within reach. One camera goes to each cube and both arms
go out together, with the feet staying where they are.

![Front and back tables, close, four hardware trials](media/hardware_close_front_back.webp)

The same, with one table in front and the other behind. The rear cube is the case a
forward-facing camera cannot cover at all.

![Far tables, walking, four hardware trials](media/hardware_far_walk.webp)

Far apart, so neither cube is reachable from a standstill. The robot locates both first,
then walks to each in turn. Finding them before setting off is what lets it walk at a cube
rather than at a table.

Table IV has the full numbers, from 900 trials that reproduce from this release.

## Hardware

![Duke Humanoid V2 hardware](media/hardware.png)

Orange numbers are actuated joints: (1-7) shoulder pitch/roll/yaw, elbow, wrist roll/pitch/yaw,
(8) waist, (9-14) hip pitch/roll/yaw, knee, ankle pitch/roll, (15-16) camera yaw/pitch. Green
labels are modules: (I) camera, (II) gripper, (III) onboard computer. Dimensions in mm.

| | |
| --- | --- |
| DoF | 31: 27-DoF body (waist ×1, legs 2×6, arms 2×7) + two 2-DoF camera gimbals, +1 per gripper |
| Mass / height | 36 kg / 1.2 m |
| Arm reach / leg length | 0.46 m / 0.39 m |
| Cameras | 2 × Intel RealSense D436, 90°×65° RGB FoV, 0.1-3.0 m, each on its own yaw-pitch gimbal |
| End effectors | parallel grippers, 350 g each, one mimic-coupled jaw slide |
| Actuation | quasi-direct-drive throughout |
| Control | 50 Hz learned whole-body policy onboard, 200 Hz CAN motor loop |

MJCF, meshes, camera modules, and gripper are in
[`simulation/asset/duke_v2/`](https://github.com/generalroboticslab/duke_humanoid_v2_simulation/tree/main/asset/duke_v2).

## Quick start

```bash
git clone --recurse-submodules git@github.com:generalroboticslab/duke_humanoid_v2.git
cd duke_humanoid_v2/simulation
pip install -r requirements.txt          # plus nvidia-curobo, see simulation/README.md
```

Regenerate the workspace figures. These read the shipped caches and finish in seconds,
without a GPU sweep or any training:

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

The split is between what runs in simulation and what runs on the robot. They are separate
repositories, not directories, so each keeps its own history and issue tracker. Training exports a policy; `deploy/` runs the export.

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
incidents behind them.
[`auto_operator_incidents.md`](https://github.com/generalroboticslab/duke_humanoid_v2_deploy/blob/main/control/docs/auto_operator_incidents.md)
lists each failure alongside the "simplification" that would bring it back.

## License

Apache-2.0, both repositories. Third-party robot models keep their upstream licenses
alongside their assets.
