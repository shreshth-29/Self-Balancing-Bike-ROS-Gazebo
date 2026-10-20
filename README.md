# Inter IIT Self-Balancing Bike

Controller for balancing and autonomous navigation of a flywheel-based bike, built for the "Programobot" problem statement at Inter IIT Tech Symphony 2021. The bike is simulated in Gazebo with ROS Noetic, balances using a reaction flywheel and navigates to a goal while avoiding obstacles with LiDAR.

## Team BotterHeads (IIT Roorkee)
- Aryan Rajput
- Ritvik Mahajan
- Sameer Talwar
- Shreshth Mehrotra

The original submission repository is at [ritvikmahajan01/InterIIT_SelfBalancingBike_Submission](https://github.com/ritvikmahajan01/InterIIT_SelfBalancingBike_Submission). This copy is the same code.

## What it does
- Self-balancing using a flywheel driven by a PD controller on the roll angle from the IMU
- Autonomous navigation towards a goal position using GPS and IMU heading
- Obstacle detection and avoidance using LiDAR
- Stand mechanism that lifts at the start and lowers again once the goal is reached, after which the flywheel is spun down

Demo videos and the supporting design document are in `BotterHeads/Supporting document and videos/`.

## Prerequisites

### System
- Ubuntu 20.04 or later
- ROS Noetic
- Gazebo 11
- Python 3.8 or later

### ROS packages
- rospy, sensor_msgs, geometry_msgs, std_msgs, tf, gazebo_ros
- hector_gazebo_plugins and gazebo_ros_motors (both included under `BotterHeads/sbb_dependencies/`)

## Installation

1. Install ROS Noetic
   ```bash
   sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
   sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
   sudo apt update
   sudo apt install ros-noetic-desktop-full
   ```

2. Install dependencies
   ```bash
   sudo apt install python3-pip python3-catkin-tools
   pip3 install numpy
   ```

3. Set up the workspace
   ```bash
   mkdir -p ~/catkin_ws/src
   cd ~/catkin_ws/src

   git clone https://github.com/shreshth-29/InterIIT_SelfBalancingBike

   # Copy the bike package and the custom Gazebo plugins into the workspace
   cp -r InterIIT_SelfBalancingBike/BotterHeads/inter_iit_sbb_description .
   cp -r InterIIT_SelfBalancingBike/BotterHeads/sbb_dependencies/hector_gazebo_plugins .
   cp -r InterIIT_SelfBalancingBike/BotterHeads/sbb_dependencies/gazebo_ros_motors-master .

   cd ~/catkin_ws
   catkin_make
   source devel/setup.bash
   ```

## Running the simulation

1. Launch the world
   ```bash
   source ~/catkin_ws/devel/setup.bash
   roslaunch inter_iit_sbb_description world.launch
   ```

2. Start the controller in a second terminal
   ```bash
   source ~/catkin_ws/devel/setup.bash
   rosrun inter_iit_sbb_description controller.py
   ```

The bike starts balancing and heads to the goal at (15.7, -40.7) in the world frame. LiDAR readings are used to steer around obstacles on the way, and the flywheel keeps the bike upright throughout.

## Project structure

```
BotterHeads/
├── inter_iit_sbb_description/
│   ├── launch/      # world.launch, empty_world.launch
│   ├── meshes/      # STL meshes for the bike, flywheel, sensors and stand
│   ├── params/      # joint and motor parameters
│   ├── scripts/     # controller.py (balancing + navigation), printpose.py
│   ├── urdf/        # generated URDF, transmissions and Gazebo tags
│   ├── world/       # competition world files
│   └── xacro/       # XACRO robot description and materials
├── sbb_dependencies/
│   ├── hector_gazebo_plugins/     # IMU, GPS and magnetometer plugins
│   └── gazebo_ros_motors-master/  # DC motor and joint motor plugins
└── Supporting document and videos/
    ├── Supporting Doc - Programobot.pdf
    ├── Explanation_Video.mp4
    └── Gazebo_Simulation.mp4
```

## Key components

1. `controller.py`
   - Runs at 100 Hz. Subscribes to the IMU, GPS, front LiDAR and the stand and handle joint states.
   - Balancing: PD control on roll angle and roll rate gives the required torque, which is integrated into a flywheel angular velocity command. The damping gain is chosen for critical damping so the bike comes upright without oscillating.
   - Navigation: heading to the goal is computed from GPS, the 180 LiDAR samples are split into sectors, and the free sector closest to the goal direction is chosen. The handle is turned with P control on heading error and the rear wheel speed is proportional to the remaining distance.
   - Stands are lifted at the start and lowered on arrival, then the flywheel is slowed to a stop.

2. Robot description
   - XACRO files define the frame, flywheel, front and back wheels, handle, stand and sensors, with inertial properties and joint limits.
   - Gazebo plugins provide the IMU, GPS, LiDAR and motor models.

3. Launch files
   - `world.launch` loads the competition world and spawns the bike.
   - `empty_world.launch` spawns the bike in an empty world for testing.
