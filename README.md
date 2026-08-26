# Duke Humanoid V2

A 36 kg bipedal humanoid with two 7-DOF arms, parallel grippers, and an actuated
head-camera gimbal whose two cameras aim independently of torso heading.

This repository is the entry point. The code lives in two submodules, split along
the line that matters in practice: what runs in simulation, and what runs on the
robot.

```bash
git clone --recurse-submodules git@github.com:generalroboticslab/duke_humanoid_v2.git
```

Already cloned without the flag:

```bash
git submodule update --init --recursive
```

## The two repositories

| | what it is | license |
| --- | --- | --- |
| [`simulation/`](https://github.com/generalroboticslab/duke_humanoid_v2_simulation) | Policy training and the paper's reproduction package: the visible-reachable workspace study, the two-target reach-and-grasp benchmark, and the checkpoints behind the reported numbers. | Apache-2.0 |
| [`deploy/`](https://github.com/generalroboticslab/duke_humanoid_v2_deploy) | The onboard control stack: the 50 Hz policy loop, perception bridge, cuRobo planning client, gripper service, and the autonomous operator. | MIT |

They are separate repositories, not directories, so each keeps its own history,
license, and issue tracker. Training exports a policy; `deploy/` runs the export.

## Where to start

Reproducing a number from the paper, or training a policy: `simulation/README.md`.
It regenerates the workspace figures from shipped caches in seconds, with no GPU
sweep and no training.

Running hardware: `deploy/control/docs/OPERATIONS.md` first. That stack moves a
36 kg machine with people next to it, and its safety gates have incidents behind
them.
