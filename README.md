# IRL ROS rolling buildfarm (Pixi + Vinca + Rattler)

This repository is a fork of the upstream RoboStack ROS rolling packaging repo:

- https://github.com/RoboStack/ros-rolling

In this fork, we use the same toolchain (Pixi + Vinca + rattler-build) to build and publish **additional ROS 2 rolling packages** produced by the **Intelligent Robotics Lab (IRL)**:

- https://intelligentroboticslab.gsyc.urjc.es/

The resulting binaries are published to prefix.dev (notably the `irl-rolling` channel) and can also be consumed locally via a file-based channel.

## Documentation

- PlanSys2
    - Build & publish (package creators): [irl-docs/plansys2/buildfarm_plansys2.md](irl-docs/plansys2/buildfarm_plansys2.md)
    - User workspace template (Pixi): [irl-docs/plansys2/pixi.toml](irl-docs/plansys2/pixi.toml)

- EasyNav (EasyNavigation) + NavMap
    - Build & publish (package creators): [irl-docs/easynav/buildfarm_easynav.md](irl-docs/easynav/buildfarm_easynav.md)
    - User workspace template (Pixi): [irl-docs/easynav/pixi.toml](irl-docs/easynav/pixi.toml)

- Shared buildfarm runbook: [irl-docs/tutorial_buildfarm.md](irl-docs/tutorial_buildfarm.md)

## About Pixi

Pixi is a developer workflow tool that manages reproducible environments, tasks, and activation hooks on top of the Conda ecosystem.

- Pixi: https://pixi.sh/
- Commands & troubleshooting in this repo: [irl-docs/pixi.md](irl-docs/pixi.md)
- Prefix.dev channels:
    - [`irl-rolling`](https://prefix.dev/channels/irl-rolling)
    - [`robostack-rolling`](https://prefix.dev/channels/robostack-rolling)
