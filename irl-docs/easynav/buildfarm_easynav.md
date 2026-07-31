# EasyNav (EasyNavigation) on ROS 2 rolling

This documentation covers two related workflows:

1. **For users**: how to set up a Pixi-based ROS workspace that consumes EasyNav/NavMap binaries and builds your own code with `colcon`.
2. **For package creators**: how to build EasyNav/NavMap packages in this repository (`ros-rolling/`) and publish them to prefix.dev.

Project websites:

- EasyNavigation (EasyNav): https://easynavigation.github.io/
- NavMap: https://easynavigation.github.io/ (NavMap is part of the EasyNavigation ecosystem)

Pixi background and common troubleshooting lives in:

- `irl-docs/pixi.md`

---

## 1) What is this?

EasyNav is a navigation stack developed by IRL (and collaborators) for ROS 2.

In this repo we build EasyNav-related ROS packages as Conda artifacts (`.conda`) and publish them to prefix.dev (channel `irl-rolling`).

---

## 2) For users: workspace with Pixi + colcon

This folder contains a ready-to-use workspace template:

- `irl-docs/easynav/pixi.toml`

### 2.1 Create a new workspace

Create a new folder for your workspace and copy the template files:

```bash
mkdir -p ~/easynav_plugins_ws
cd ~/easynav_plugins_ws

cp /path/to/pixi-buildfarm/ros-rolling/irl-docs/easynav/pixi.toml ./pixi.toml
```

Then update the local file-based channel path if needed:

- `file:///.../ros-rolling/output`

Important: use the **channel root**, not `.../output/linux-64`. See `irl-docs/pixi.md`.

What the template does by default:

- Prefers Zenoh (`RMW_IMPLEMENTATION=rmw_zenoh_cpp`).
- Pins `eigen < 5` to avoid CMake falling back to system Eigen via PCL.
- Enables bash completion for `ros2` (via `argcomplete`) in interactive shells.

### 2.2 Install dependencies

```bash
pixi install
```

### 2.3 Build your ROS workspace

Add your ROS packages under `src/` (as in a normal colcon workspace) and build:

```bash
pixi run build
```

### 2.4 Running EasyNav

- Use `ros2 launch ...` for launching systems.
- Use `ros2 run ...` for single executables.

If you need a curated set of binaries from `irl-rolling`, keep the channel order in `pixi.toml` so local artifacts override remote ones.

---

## 3) For package creators: buildfarm in this repo

All buildfarm work happens under `ros-rolling/`.

### 3.1 Quick path (Pixi tasks)

```bash
cd ros-rolling
pixi install
pixi run easynav-all
```

This runs, in order: clean → generate recipes → build → index.

### 3.2 How the subset is defined

The EasyNav subset is defined in:

- `ros-rolling/easynav_subset/vinca.yaml`

Notes:

- Vinca reads `vinca.yaml` from the directory passed via `-d`.
- Generated recipes always go into `ros-rolling/recipes/`.

### 3.3 Manual flow (if you need it)

Generate recipes:

```bash
cd ros-rolling
pixi run remove-recipes
pixi run -v vinca -d ./easynav_subset --platform linux-64 -m
```

Build `.conda` artifacts:

```bash
pixi run rattler-build build \
  --recipe-dir ./recipes \
  --target-platform linux-64 \
  -m ./conda_build_config.yaml \
  -c https://prefix.dev/robostack-rolling \
  -c https://repo.prefix.dev/conda-forge \
  --skip-existing
```

Index the local channel:

```bash
pixi run rattler-index fs ./output --force
```

### 3.4 Upload to prefix.dev

Typical upload to `irl-rolling`:

```bash
cd ros-rolling
export PREFIX_API_KEY="..."

pixi run rattler-build upload prefix \
  --channel irl-rolling \
  --force \
  output/linux-64/*.conda
```

If consumers can't see newly-published packages, clear repodata cache on the consumer side:

```bash
pixi clean cache --repodata -y
```

For rate-limited uploads, upload one file at a time with backoff:

```bash
cd ros-rolling
export PREFIX_API_KEY="..."

for pkg in output/linux-64/*.conda; do
  for attempt in 1 2 3 4 5; do
    if pixi run rattler-build upload prefix --channel irl-rolling --force "$pkg"; then
      break
    fi
    sleep $((attempt * 2))
  done
done
```
