# Topside Docker setup

Run these commands on the above-water laptop. This is the competition setup;
it does not install or require HoloOcean.

Docker and Docker Compose must already be installed. Check:

```bash
docker --version
docker compose version
```

On Ubuntu, install [Docker Engine](https://docs.docker.com/engine/install/ubuntu/)
and the [Docker Compose plugin](https://docs.docker.com/compose/install/linux/).
Your user needs permission to run Docker; follow the
[Linux post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/).
If you add your user to the Docker group, log out and back in before continuing.

## Start the container

From the `2026-27WaterCode` repository root:

```bash
cd docker
bash setup.sh
bash enter.sh
```

This starts the `leviathan` container and opens a terminal inside it.

## Build and launch topside

Inside the container:

```bash
source /opt/ros/humble/setup.bash
cd /home/ubuntu/2026-27WaterCode/fishROS_ws
colcon build --symlink-install --packages-select controller_node
source install/setup.bash
ros2 launch controller_node topside_combined_launch.py
```

Plug the gamepad into the laptop. Connect the laptop to the Jetson through the tether network. Start the Jetson hardware launch separately.

## Stop the container

Press `Ctrl+C` to stop topside, then run `exit`. From the host repository root:

```bash
bash docker/stop.sh
```

For HoloOcean, use the separate [simulation setup](simulation-setup.md).
