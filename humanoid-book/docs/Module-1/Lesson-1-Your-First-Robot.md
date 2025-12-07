---
sidebar_position: 1
---

# Lesson 1: Your First Robot "Hello World"

In this lesson, we'll create our first robot in ROS 2. This is the "Hello World" of robotics, where we'll set up a minimal robot that can be launched and visualized in a simulation environment.

## Prerequisites
- ROS 2 Humble Hawksbill installed (or your preferred ROS 2 distribution)
- Basic understanding of Linux command line
- Text editor or IDE for development

## What You'll Build
By the end of this lesson, you'll have:
1. Created a minimal ROS 2 package for your robot
2. Defined a simple robot model in URDF (Unified Robot Description Format)
3. Launched a minimal robot simulation
4. Visualized your robot in RViz

## Setting Up Your Workspace

First, let's create a workspace for our robot project:

```bash
mkdir -p ~/physical_ai_ws/src
cd ~/physical_ai_ws
```

## Creating Your Robot Package

Now we'll create a package for our robot:

```bash
cd src
ros2 pkg create --build-type ament_python my_first_robot --dependencies rclpy std_msgs geometry_msgs
```

## Understanding the Package Structure

After creating the package, you'll see this structure:

```
my_first_robot/
├── CMakeLists.txt
├── package.xml
├── setup.cfg
├── setup.py
└── my_first_robot/
    └── __init__.py
```

## Creating the Robot Description (URDF)

Let's create a simple robot model. Create a new directory for robot descriptions:

```bash
mkdir -p my_first_robot/urdf
```

Then create `my_first_robot/urdf/simple_robot.urdf`:

```xml
<?xml version="1.0"?>
<robot name="simple_robot">
  <!-- Base link -->
  <link name="base_link">
    <visual>
      <geometry>
        <box size="0.5 0.3 0.2"/>
      </geometry>
      <material name="blue">
        <color rgba="0 0 1 1"/>
      </material>
    </visual>
  </link>

  <!-- Left wheel -->
  <joint name="left_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="left_wheel"/>
    <origin xyz="-0.15 -0.2 0" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
  </joint>

  <link name="left_wheel">
    <visual>
      <geometry>
        <cylinder radius="0.1" length="0.05"/>
      </geometry>
      <material name="black">
        <color rgba="0 0 0 1"/>
      </material>
    </visual>
  </link>

  <!-- Right wheel -->
  <joint name="right_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="right_wheel"/>
    <origin xyz="-0.15 0.2 0" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
  </joint>

  <link name="right_wheel">
    <visual>
      <geometry>
        <cylinder radius="0.1" length="0.05"/>
      </geometry>
      <material name="black">
        <color rgba="0 0 0 1"/>
      </material>
    </visual>
  </link>
</robot>
```

## Launching Your Robot

*Note: This section would include a GIF demonstrating the robot in RViz. The actual GIF would show the robot model loaded in RViz with its different links visible.*

Create a launch directory:

```bash
mkdir -p my_first_robot/launch
```

Create `my_first_robot/launch/simple_robot.launch.py`:

```python
from launch import LaunchDescription
from launch.substitutions import Command
from launch_ros.actions import Node
from ament_index_python.packages import get_package_share_directory
import os


def generate_launch_description():
    pkg_share = get_package_share_directory('my_first_robot')
    
    # Robot State Publisher node
    robot_state_publisher = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        name='robot_state_publisher',
        parameters=[{
            'robot_description': Command([
                'xacro ', os.path.join(pkg_share, 'urdf', 'simple_robot.urdf')
            ])
        }]
    )
    
    # Joint State Publisher node
    joint_state_publisher = Node(
        package='joint_state_publisher',
        executable='joint_state_publisher',
        name='joint_state_publisher'
    )
    
    # RViz node
    rviz = Node(
        package='rviz2',
        executable='rviz2',
        name='rviz2',
        arguments=['-d', os.path.join(pkg_share, 'rviz', 'config.rviz')]
    )
    
    return LaunchDescription([
        robot_state_publisher,
        joint_state_publisher,
        rviz
    ])
```

## Creating the RViz Configuration

Create an rviz directory:

```bash
mkdir -p my_first_robot/rviz
```

Create `my_first_robot/rviz/config.rviz`:

```yaml
Panels:
  - Class: rviz_common/Displays
    Name: Displays
  - Class: rviz_common/Views
    Name: Views
Visualization Manager:
  Displays:
    - Class: rviz_default_plugins/RobotModel
      Name: RobotModel
      Topic: /robot_description
  Global Options:
    Fixed Frame: base_link
  Views:
    Current:
      Class: rviz_default_plugins/Orbit
```

## Building and Running

Build your workspace:

```bash
cd ~/physical_ai_ws
colcon build
source install/setup.bash
```

Run your first robot:

```bash
ros2 launch my_first_robot simple_robot.launch.py
`