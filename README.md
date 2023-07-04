# Agx Fleet Manager

## Installation Instructions

### Prerequisites

* [Ubuntu 20.04 LTS](https://releases.ubuntu.com/20.04/)
* [ROS2 - Galactic](https://docs.ros.org/en/galactic/index.html)

Install all non-ROS prerequisite packages,

```bash
sudo apt update && sudo apt install \
  git wget qt5-default \
  python3-rosdep \
  python3-vcstool \
  python3-colcon-common-extensions \
  # maven default-jdk   # Uncomment to install dependencies for message generation
```

### Client and Server in ROS 2

Start a new ROS 2 workspace, and pull in the necessary repositories,

```bash
mkdir -p ~/agx_fleet_ros2_ws/src
cd ~/agx_fleet_ros2_ws/src
git clone https://github.com/lagrangeluo/agx_fleet_client.git
```

Install all the dependencies through `rosdep`,

```bash
cd ~/agx_fleet_ros2_ws
rosdep install --from-paths src --ignore-src --rosdistro galactic -yr
```

Source ROS 2 and build, 

```bash
cd ~/agx_fleet_ros2_ws
source /opt/ros/galactic/setup.bash
colcon build

# Optionally use the command below to only build the relevant packages,
# colcon build --packages-up-to \
#     free_fleet ff_examples_ros2 free_fleet_server_ros2 free_fleet_client_ros2
```



## Examples

### Client

This repository also has client code which is used in ros2, this example emulates a running ROS 2 robot,

```bash
source ~/agx_fleet_ros2_ws/install/setup.bash
ros2 launch agx_fleet_manager fake_client.launch.xml
```

The clients will then start subscribing to all the necessary topics, and start publishing robot states over DDS to the server. Start the server using

```bash
source ~/agx_fleet_ros2_ws/install/setup.bash
ros2 launch ff_examples_ros2 fake_server.launch.xml

# Verify that the server registers the fake clients
# [INFO] [1636706176.184055177] [fake_server_node]: registered a new robot: [fake_ros1_robot]
# [INFO] [1636706176.184055177] [fake_server_node]: registered a new robot: [fake_ros2_robot]
```

ROS 2 messages over the `/fleet_states` topic can also be used to verify that the clients are registered,

```bash
source ~/agx_fleet_ros2_ws/install/setup.bash
ros2 topic echo /fleet_states
```

### Fleet Server

This launches a fake server for a fleet of robots, use this to test DDS communication.

```bash
source ~/ff_ros2_ws/install/setup.bash
ros2 launch agx_fleet_ros2_ws fake_server.launch.xml
```

At this point, the server should register any clients running, if any of the simulations below are running.

use command to launch a servcer for agx fleet.

``` bash
ros2 launch agx_fleet_ros2_ws agx_fleet_server.launch
```

### Commands and Requests

Now the fun begins! There are 3 types of commands/requests that can be sent to the simulated robots through `free_fleet`,

#### Destination requests

which allows single destination commands for the robots,

```bash
ros2 run agx_fleet_manager send_destination_request.py -f FLEET_NAME -r ROBOT_NAME -x 1.725 -y -0.39 --yaw 0.0 -i UNIQUE_TASK_ID
```

for example:

```bash
ros2 run agx_fleet_manager send_destination_request.py -f agx_fleet -r limo_1 -x 1.71 -y -1.28 --yaw -0.83 -i UNIQUE_TASK_ID
```

#### Path requests

which requests that the robot perform a string of destination commands,

```bash
ros2 run ff_examples_ros2 send_path_request.py -f FLEET_NAME -r ROBOT_NAME -i UNIQUE_TASK_ID -p '[{"x": 1.725, "y": -0.39, "yaw": 0.0, "level_name": "B1"}, {"x": 1.737, "y": 0.951, "yaw": 1.57, "level_name": "B1"}, {"x": -0.616, "y": 1.852, "yaw": 3.14, "level_name": "B1"}, {"x": -0.626, "y": -1.972, "yaw": 4.71, "level_name": "B1"}]'
```

**Note** that the task IDs need to be unique, if a request is sent using a previously used task ID, the request will be ignored by the free fleet clients.

#### RMF Pannel

If you want to send request to robots,you can use the web tools developed by rmf developers:[RMF_Pannel](https://open-rmf.github.io/rmf-panel-js/)

### ROS 2 Turtlebot3 Simulation

TODO...
