
# Sereact Teleoperation Challenge – Dual Arm Gesture Teleoperation

## Project Overview

This project implements a ROS 2 Humble based dual-arm teleoperation system using:

- Dual 6DOF robot arms
- Two-finger grippers
- MoveIt 2 motion planning
- RViz visualization
- MediaPipe hand tracking
- OpenCV
- Dockerized ROS 2 environment

The operator controls the dual robot arms using hand gestures captured from a normal RGB camera.

The system supports:

- 2D end-effector motion control
- 1D wrist rotation
- Real-time gripper actuation
- Delta-based teleoperation
- Collision-aware motion planning
- Joint angle publishing
- Dual-arm operation

The implementation was designed according to the Sereact Teleoperation Challenge requirements.

---

# Task Requirements Coverage

## 1. Set Up the Environment

- Developed as a ROS 2 Humble workspace
- Uses Docker + Docker Compose
- Uses standard RGB webcam
- Runs on Ubuntu Linux

---

## 2. Robot Arm Visualization

Implemented in RViz (not simulation).

Features:

- Dual 6DOF robot arms
- Two-finger grippers
- Motion planning with MoveIt 2
- Self-collision avoidance
- Real-time end-effector pose visualization
- Interactive planning support

---

## 3. Message Publishing

The system publishes:

- End-effector target poses
- Gripper commands
- Joint angle values

Topics include:

```bash
/left_target_pose
/right_target_pose
/left_gripper_cmd
/right_gripper_cmd
/joint_states
/current_joint_angles
```

---

## 4. Teleoperation Node

The teleoperation node:

- Captures RGB camera frames
- Performs hand tracking using MediaPipe
- Maps hand motion to robot motion
- Controls:
  - End-effector planar movement
  - Wrist rotation
  - Gripper open/close action

The implementation uses:

- Delta mapping
- Kalman filtering
- Deadband filtering
- Motion limiting

to reduce noisy tracking and improve stability.

---

## 5. Version Control

The project uses Git for version control.

Clone locally:

```bash
git clone https://github.com/AnanthaPrajith/sereact_teleoperation_task.git
```

---

## 6. Video Demonstration

A demonstration video should include:

- RViz visualization
- Hand tracking
- Dual-arm control
- Gripper actuation
- Motion planning
- Smooth teleoperation

---

## 7. Open-Source References

This project uses the following open-source tools/libraries:

- ROS 2 Humble
- MoveIt 2
- RViz 2
- MediaPipe Hands
- OpenCV
- Docker
- Docker Compose
### Credits

1. Universal Robots — ROS2 Description Repository  
   https://github.com/UniversalRobots/Universal_Robots_ROS2_Description.git

2. PickNik Robotics — Robotiq Gripper Repository  
   https://github.com/PickNikRobotics/ros2_robotiq_gripper.git

3. William Woodall — Serial Communication Repository  
   https://github.com/wjwwood/serial.git

All referenced repositories were used strictly for research, integration, and learning purposes. The teleoperation logic, gesture-control pipeline, dual-arm coordination, delta-mapping control strategy, and ROS2 integration were implemented and modified specifically for this assignment.

---

# System Architecture

```text
Webcam
   ↓
eye_bridge.py
   ↓ TCP Socket
gesture_control_node.py
   ↓ ROS Topics
MoveIt IK Commander
   ↓
RViz Visualization
```

---

# Repository Setup

## 1. Clone Repository

Either clone with the git repo or extract the zip file attached in submission link and proceed with the remaining process in terminal.

```bash
git clone https://github.com/AnanthaPrajith/sereact_teleoperation_task.git
cd gesture_bot_ws
```

---

# Configure Gesture Teleoperation IP

Open:

```bash
nano gesture_bot_ws/src/gesture_teleop/gesture_teleop/gesture_control_node.py
```

Find:

```python
self.client_socket.connect(('127.0.0.1', 9999))
```

## Same Machine

Use:

```python
self.client_socket.connect(('127.0.0.1', 9999))
```

## Different Device / Network

Replace with local IP:

```python
self.client_socket.connect(('192.168.x.x', 9999))
```

Example:

```python
self.client_socket.connect(('192.168.0.177', 9999))
```

Find local IP:

```bash
hostname -I
```

---

# Start Camera Bridge

Run on host machine:

```bash
python3 ~/gesture_bot_ws/eye_bridge.py
```

Expected:

```text
Windows Eye Bridge is LIVE. Waiting for WSL...
```

This streams webcam frames to ROS nodes.

---

# Build Docker Workspace

```bash
docker compose build
```

---

# Launch Full System

```bash
docker compose run --rm gesture_bot .
```

This launches:

- ROS 2 Humble
- MoveIt 2
- RViz
- IK commander
- Teleoperation nodes

---

# RViz Setup

When RViz opens:

## Change Fixed Frame

Left panel:

```text
Global Options → Fixed Frame
```

Change:

```text
map
```

to:

```text
world
```

---

## Add MotionPlanning Plugin

In RViz:

```text
Add → By display type → MotionPlanning
```

or

```text
Panels → Add New Panel → MotionPlanning
```

---

## Configure Motion Planning

Inside MotionPlanning panel:

```text
Context → Planning Library
```

Select:

```text
OMPL
```

Planner:

```text
RRTConnectkConfigDefault
```

---

# Run Gesture Teleoperation Node

If not auto-started:

```bash
source /opt/ros/humble/setup.bash
source install/setup.bash

ros2 run gesture_teleop gesture_control_node
```

Expected:

```text
GESTURE NODE: STABLE DUAL-ARM DELTA MAPPING ACTIVE
```

---

# Check Published Topics

## End-Effector Pose Topics

```bash
ros2 topic echo /left_target_pose
ros2 topic echo /right_target_pose
```

---

## Gripper Topics

```bash
ros2 topic echo /left_gripper_cmd
ros2 topic echo /right_gripper_cmd
```

Pinch gesture:

```yaml
data: true
```

Open hand:

```yaml
data: false
```

---

# Joint Angle Publishing

## Standard Joint States

```bash
ros2 topic echo /joint_states
```

## Custom Joint Angle Topic

If included:

```bash
ros2 run gesture_teleop joint_angle_publisher
```

Check:

```bash
ros2 topic echo /current_joint_angles
```

Message type:

```text
sensor_msgs/msg/JointState
```

---

# Useful Commands

## List ROS Topics

```bash
ros2 topic list
```

## Topic Info

```bash
ros2 topic info /left_target_pose
ros2 topic info /right_target_pose
```

## Rebuild Workspace

```bash
colcon build
source install/setup.bash
```

---

# Noise Handling and Stability

The system includes:

- Delta pose mapping
- Kalman filtering
- Deadband suppression
- Step limiting
- Workspace constraints
- Safe pose clipping

These reduce jitter and improve stability for real-world teleoperation.

---

# Real-World Considerations

The implementation was designed considering:

- Single operator control
- Human reachability limitations
- No depth camera availability
- Smooth and stable motion
- Fast ROS topic publishing
- Safety constraints
- Real-time teleoperation behavior

---

# Notes

- RViz is used only for visualization (not simulation)
- MoveIt performs motion planning and collision checking
- The robot receives target poses through ROS topics
- The system supports simultaneous dual-arm operation

