# Spot ROS 2 FAST-LIO Simulation

This repository provides a complete ROS 2 Gazebo simulation environment for the Boston Dynamics Spot robot, tightly integrated with the [FAST-LIO2](https://github.com/hku-mars/FAST_LIO) LiDAR-inertial odometry package for robust 3D mapping and state estimation.

## Features
- **Spot Robot Simulation**: Accurately modeled Spot URDF/SDF running within Ignition/Gazebo.
- **Sensor Suite**: Equipped with a simulated 16-channel Velodyne-style LiDAR and high-frequency IMU.
- **Optimized FAST-LIO2**: Incorporates a custom fork of `FAST_LIO_ROS2` that cleanly parameterizes mathematical TF frames (`camera_init` & `body`), allowing it to natively publish perfectly compliant ROS transforms (`odom` -> `base_link`) without conflicting with typical robot TF trees.

---

## 🛠️ Prerequisites
- **Ubuntu 22.04**
- **ROS 2 Humble**
- **Ignition / Gazebo** (Garden or Fortress depending on your ROS 2 setup)
- **PCL & Eigen** (Required for FAST-LIO2)

Install standard dependencies:
```bash
sudo apt update
sudo apt install -y python3-colcon-common-extensions ros-humble-teleop-twist-keyboard libpcl-dev
```

---

## 🚀 Installation & Build

Since this package relies on specific customized submodules (such as our FAST-LIO fork and Livox drivers), **you must clone recursively**:

```bash
# 1. Clone the repository recursively
git clone --recursive https://github.com/haasithp/spot_ros2_fastlio.git
cd spot_ros2_fastlio

# NOTE: If you forgot to use --recursive when cloning, you can fetch submodules via:
# git submodule update --init --recursive

# 2. Build the workspace
colcon build --symlink-install

# 3. Source the environment
source install/setup.bash
```

---

## 🎯 Running the Simulation

To test the system, you will need to open **three separate terminals**. Make sure to run `source install/setup.bash` in each new terminal before executing the following commands.

### Terminal 1: Launch the Spot Gazebo Environment
This launches the Spot URDF inside a predefined Gazebo world, spawning the simulated LiDAR, IMU, cameras, and joint controllers.
```bash
ros2 launch spot_bringup spot.gazebo.launch.py
```

### Terminal 2: Launch FAST-LIO2 Mapping
This boots up the FAST-LIO2 pipeline. It uses a bespoke `spot.yaml` configuration which utilizes the standard unstructured PointCloud2 handler and configures the LiDAR-to-IMU extrinsics internally.
```bash
ros2 launch fast_lio mapping.launch.py config_file:=spot.yaml use_sim_time:=true
```

### Terminal 3: Drive Spot Around!
Spot is ready to move! Use the standard ROS 2 `teleop_twist_keyboard` node to send velocity commands. Make sure you remap the topic to strictly publish to `/cmd_vel`:
```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -r cmd_vel:=/cmd_vel
```
*Note: Use `i`, `j`, `k`, `l`, `,` to move. Press `k` to stop.*

---

## 🔧 Notable Configuration Details

### The FAST-LIO TF Tree Fix
Standard FAST-LIO repositories hardcode academic visual-inertial dataset frames (`camera_init` and `body`) deep within their C++ implementations. This repository uses [a custom submodule fork of FAST_LIO_ROS2](https://github.com/haasithp/FAST_LIO_ROS2) which successfully extracts these internal frames into ROS 2 parameters (`odom_frame` and `body_frame`). 

Thanks to this, running FAST-LIO inside this repo correctly generates the standard ROS graph: `odom` -> `base_link` without breaking the robot model.

### spot.yaml
Located in `src/FAST_LIO_ROS2/config/spot.yaml`, this file contains tuning parameters for simulating the Boston Dynamics Spot:
- `lidar_type: 5` utilizes the default generic `pcl::PointXYZI` handler rather than Velodyne, compensating for Gazebo's omission of embedded point-wise `time` and `ring` fields.
- `extrinsic_T` manages the direct transformation constraint between Spot's base IMU and the back-mounted LiDAR.
