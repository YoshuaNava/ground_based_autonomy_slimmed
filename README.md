The repository contains code for collision avoidance to use with ground robots. In an autonomous navigation system, this repository is often at the mid-level, above the state estimation module and below the high-level planning module. To help users start easily, the repository includes a vehicle simulator and does not need to run onboard a real robot. The code implementation is targeted at executing on a robot computer with limited processing power. Except for the vehicle simulator, all code in the repository is optimized and lightweight. The code operates in two modes: *autonomy mode* takes waypoints as the input and navigates towards the waypoints while avoiding obstacles along the way, *smart joystick mode* takes commands from an operator through a joystick controller and avoids obstacles that the robot encounters. The *smart joystick mode* is often useful in debugging the navigation system and is highly recommended.

A video showing functionalities of the code is available at: https://youtu.be/JgJG3Y8JCt0

# Prerequisite

To use *smart joystick mode*, the system requires a PS3 controller (*autonomy mode* can run without a controller). Most PS3 controllers with a USB interface would work out of the box. The one that has been tested is an EasySMX 2.4G Wireless Controller.

<img src="img/ps3_controller.jpg" alt="PS3 Controller" width="55%"/>

Some controllers have different modes. Make sure the controller is in the right mode (usually the factory default mode) and is powered on. If using the controller in the image, the two LEDs on top of the center button should be lit. Holding the center button for a few seconds changes the mode.

The code runs on a Ubuntu 18.04 computer installed with ROS Melodic. Install ROS joystick driver,

```sudo apt update```

```sudo apt install ros-melodic-joystick-drivers```

Make sure to add the username to dialout group (change 'username' in the command line) and reboot the computer,

```sudo adduser username dialout```

# Quick Start

Clone the repository. In a terminal, go to the folder and compile,

```catkin_make```

Source the ROS workspace,

```source devel/setup.sh```

To launch the code in *smart joystick mode*, plugin the controller to the computer,

```roslaunch vehicle_simulator vehicle_simulator.launch```

Wait for Gazebo to initialize in a few sections, the following image will show up in RVIZ. Now, use the right joystick on the controller to navigate the robot. Pushing the right joystick to the front and back drives the robot around and pushing the right joystick to the left and right makes rotations. Holding the obstacle-check button cancels obstacles checking.

<img src="img/smart_joystick.jpg" alt="Smart Joystick" width="75%"/>

Alternatively, one can launch the code in *autonomy mode*. If a controller is available, uncomment

```<include file='$(find waypoint_example)/launch/waypoint_example.launch' />```

in 'src/vehicle_simulator/launch/vehicle_simulator.launch'. This launches the 'waypoint_example' - an example of sending waypoints, speed, and navigation boundaries. Then,

```roslaunch vehicle_simulator vehicle_simulator.launch```

Hold the mode-switch button on the controller and at the same time push the right joystick. The robot will follow the waypoints for one circle. Here, the right joystick gives the speed. If only the mode-switch button is held, the system will start taking speed from the 'waypoint_example' in a few seconds and the robot will start moving.

<img src="img/waypoint_following.jpg" alt="Waypoint Following" width="75%"/>

If a controller is unavailable, set 'autonomyMode = true' in 'src/local_planner/launch/local_planner.launch'. The system will start in *autonomy mode*. Note that you can set 'autonomyMode = true' even if a controller is plugged-in. Pressing any button on the controller will bring the system to *smart joystick mode* and holding the mode-switch button will bring the system back to *autonomy mode*.

# Advanced

**Running on real robot**: The system is setup to use a vehicle simulator for a quick start. The vehicle simulator generates 'nav_msgs::Odometry' typed state estimation messages on ROS topic '/state_estimation' and 'sensor_msgs::PointCloud2' typed registered scan messages on ROS topic '/velodyne_points'. The scans are simulated based on a Velodyne VLP-16 Lidar and are registered in the '/map' frame. To use the code with a real robot, replace the vehicle simulator by the state estimation module on the robot and forward the 'geometry_msgs::TwistStamped' typed command velocity messages on ROS topic '/cmd_vel' to the motion controller.

**Integrating with high-level planner**: To use the code with a high-level planner, e.g. a route planner, follow the example code in 'src/waypoint_example/src/waypointExample.cpp' to send waypoints, speed, and navigation boundaries. The navigation boundaries in a message are considered connected if they are at the same height and disconnected if at different height.

**Adding additional sensors**: The system can take data from additional sensors for collision avoidance. The data can be sent in as 'sensor_msgs::PointCloud2' typed messages on ROS topic '/added_obstacles'. The points in the messages are in the '/map' frame.

**Obstacle checking on/off**: Obstacle checking can be turned on and off with a 'std_msgs::Bool' typed message on ROS topic '/check_obstacle'. The number in the message indicates obstacle checking is on or off. Alternatively, one can hold the obstacle-check button on the controller to turn off obstacle checking.

**Turning on terrain analysis**: If driving over 3D terrains, terrain analysis is necessary. To launch terrain analysis, uncomment

```<include file='$(find terrain_analysis)/launch/terrain_analysis.launch' />```

set 'useTerrainAnalysis = true' and adjust 'obstacleHeightThre' in 'src/local_planner/launch/local_planner.launch'. With terrain analysis running, clicking the clear-terrain-map button on the controller reinitializes the terrain analysis. Alternatively, one can send a 'std_msgs::Float32' typed message on ROS topic '/map_clearing'. The number in the message indicates the radius of the area to be cleared. Note that terrain analysis does require scans to be registered well. If the state estimation on the robot is imprecise and scans are misregistered, the terrain analysis will likely sacrifice.

**Handling negative obstacles**: While the best way to handle negative obstacles is mounting a sensor high up on the robot looking downward into the negative obstacles, a quick solution is turning on terrain analysis and setting 'noDataObstacle = true' in 'src/terrain_analysis/launch/terrain_analysis.launch'. Negative obstacles usually cause some areas to have no data. The system will consider these areas to be non-traversable.

**Adjusting path following control**: To change driving speed, set 'maxSpeed', 'autonomySpeed', and 'maxAccel' in 'src/local_planner/launch/local_planner.launch'. Other path following control parameters such as 'maxYawRate', 'yawRateGain', and 'lookAheadDis' are in the same file. If setting 'twoWayDrive = false', the robot will only drive forward.

**Changing robot size**: The code uses motion primitives generated by a MatLab script. To change the robot size, set 'searchRadius' in 'src/local_planner/paths/path_generator.m' and run the MatLab script. This will generate a set of path files in the same folder.

**Robot motion model**: The system uses a differential motion model. Adapting to other motion models, e.g. a car-like motion model, needs more work.

# Reference

J. Zhang, C. Hu, R. Gupta Chadha, and S. Singh. Falco: Fast Likelihood-based Collision Avoidance with Extension to Human-guided Navigation. Journal of Field Robotics. 2020.

# Author Website

https://frc.ri.cmu.edu/~zhangji
