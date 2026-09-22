# Linorobot2 Simulation Quick Start

This workspace contains a ROS 2 Humble Linorobot2 simulation. The examples below
use a 2-wheel-drive robot and the included `playground.world` Gazebo world.

## 1. Build the workspace

Run this after cloning the workspace or changing source/configuration files:

```bash
cd /home/tuanmac/linorobot_ws
source /opt/ros/humble/setup.bash
rosdep install --from-paths src --ignore-src -r -y \
  --skip-keys "microxrcedds_agent micro_ros_agent"
colcon build --symlink-install
```

## 2. Prepare each terminal

Open a new terminal for each process below and run:

```bash
cd /home/tuanmac/linorobot_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
export LINOROBOT2_BASE=2wd
```

Valid base values are `2wd`, `4wd`, and `mecanum`. Use the same value in every
terminal.

## 3. Run Gazebo

Terminal 1:

```bash
ros2 launch linorobot2_gazebo gazebo.launch.py
```

Wait until Gazebo opens and the robot has spawned before starting the other
nodes. The default world is `playground.world`.

To use another included world or change the spawn pose:

```bash
ros2 launch linorobot2_gazebo gazebo.launch.py \
  world:=$(ros2 pkg prefix linorobot2_gazebo)/share/linorobot2_gazebo/worlds/speedbump_test.world \
  spawn_x:=0.5 spawn_y:=0.0 spawn_z:=0.15 spawn_yaw:=0.0
```

Other included worlds are `gas_station.world`, `playground.world`, and
`speedbump_test.world`.

## 4. Drive with keyboard teleop

Terminal 2:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

Important keys:

- `i`: forward
- `,`: reverse
- `j` / `l`: rotate left / right
- `k`: stop
- `q` / `z`: increase / decrease all speeds
- `Shift+J` / `Shift+L`: strafe left / right (Mecanum only)

Keep the teleop terminal focused while driving.

## 5. Open RViz

For a basic robot view, Terminal 3:

```bash
ros2 run rviz2 rviz2 -d \
  $(ros2 pkg prefix linorobot2_gazebo)/share/linorobot2_gazebo/rviz/view_urdf.rviz \
  --ros-args -p use_sim_time:=true
```

For SLAM or navigation, use the dedicated RViz configurations in the next
sections instead.

## 6. SLAM and map creation

Keep Gazebo running. In Terminal 3, start SLAM Toolbox, Nav2, and RViz:

```bash
ros2 launch linorobot2_navigation slam.launch.py sim:=true rviz:=true
```

Drive around with keyboard teleop until the required area is mapped. Then save
the map from another prepared terminal:

```bash
mkdir -p /home/tuanmac/linorobot_ws/maps
ros2 run nav2_map_server map_saver_cli \
  -f /home/tuanmac/linorobot_ws/maps/my_map \
  --ros-args -p save_map_timeout:=10000
```

This creates `my_map.yaml` and `my_map.pgm`. Stop SLAM with `Ctrl+C` before
starting localization/navigation.

## 7. Autonomous navigation with Nav2

Keep Gazebo running. Start Nav2 with an existing map:

```bash
ros2 launch linorobot2_navigation navigation.launch.py \
  sim:=true rviz:=true \
  map:=/home/tuanmac/linorobot_ws/maps/my_map.yaml
```

The repository also includes a ready-made playground map:

```bash
ros2 launch linorobot2_navigation navigation.launch.py \
  sim:=true rviz:=true \
  map:=$(ros2 pkg prefix linorobot2_navigation)/share/linorobot2_navigation/maps/playground.yaml
```

In RViz:

1. Select **2D Pose Estimate** and click/drag at the robot's approximate pose.
2. Select **Nav2 Goal** (or **2D Goal Pose**) and click/drag to set a goal.

The initial `map` to `base_link` transform warning is expected until the initial
pose is set.

## 8. Run the real robot instead of Gazebo

Do not run Gazebo for this mode. On the robot computer:

```bash
ros2 launch linorobot2_bringup bringup.launch.py \
  base_serial_port:=/dev/ttyACM0
```

To enable a configured joystick:

```bash
ros2 launch linorobot2_bringup bringup.launch.py joy:=true
```

For a real robot, omit `sim:=true` from the SLAM and navigation commands.

## Useful checks

List active nodes and topics:

```bash
ros2 node list
ros2 topic list
```

Confirm that velocity commands and laser scans are being published:

```bash
ros2 topic echo /cmd_vel
ros2 topic hz /scan
```

View the TF tree:

```bash
ros2 run tf2_tools view_frames
```

If a package or launch file cannot be found, rebuild and source the workspace
again. Stop any running command with `Ctrl+C` before closing its terminal.

