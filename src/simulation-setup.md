# HoloOcean simulation setup

> **Warning: untested.** This HoloOcean setup has not been tested by building or running the simulation containers.

Simulation is separate from competition. Its containers are `holoocean-ros` and
`leviathan-sim`; the latter runs robot-side adapters and mounts HoloOcean interfaces.

## Prerequisites

Install Docker Engine and Docker Compose. Keep the external repositories beside
this repository:

```text
uw/
  2026-27WaterCode/
  holoocean-ros/
  HoloOcean/
```

The setup needs `holoocean-ros/docker/dev`, `holoocean-ros/holoocean_interfaces`,
`holoocean-ros/config`, and `HoloOcean/client`. Install the Ocean world into
`~/.local/share/holoocean`. Follow the external repositories' setup instructions.

On Ubuntu, install Docker using the [competition prerequisites](docker-setup.md).
The simulation installer also needs Git, curl, GnuPG, sudo, and Python's venv module:

```bash
sudo apt install git curl gnupg ca-certificates python3-venv
```

Then `bash docker/simulation/install.sh` clones the external repositories beside
this repo, installs the HoloOcean client into its own virtual environment, and
downloads Ocean. It also configures the NVIDIA container toolkit when NVIDIA
drivers are detected.

A graphical desktop is needed for the existing display mounts. The NVIDIA option
requires NVIDIA drivers and the NVIDIA container toolkit. `-N` omits the NVIDIA
runtime; whether rendering works depends on your machine and HoloOcean.

## Start simulation

From the repository root, choose one:

```bash
bash docker/simulation/setup.sh -H  # NVIDIA (default)
bash docker/simulation/setup.sh -N  # without NVIDIA runtime
```

These start containers; they do not automatically launch a simulation scenario.
Enter HoloOcean:

```bash
bash docker/simulation/enter.sh
```

Use the scenario and ROS launch instructions from `holoocean-ros` inside that
container. To enter the robot adapter container:

```bash
bash docker/simulation/enter.sh -R
```

Inside the adapter container, source ROS and build the packages you need:

```bash
source /opt/ros/humble/setup.bash
cd /home/ubuntu/2026-27WaterCode/fishROS_ws
colcon build --symlink-install --packages-select holoocean_interfaces controller_node
source install/setup.bash
```

The reconstruction converter additionally requires `sonar_oculus`, which this
setup does not install. The existing HoloOcean experiment files remain in
`controller_node`; this change only separates Docker setup.

Competition and simulation share the repository's build directories. Stop the
other workflow before switching and rebuilding. Both use host networking, so
avoid launching real robot controllers and simulation controllers together.

Stop the simulation containers from the host:

```bash
bash docker/simulation/stop.sh
```
