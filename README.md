 Embodied AI Tracked Robot 

A high-performance tracked mobile robot built from scratch. Powered by Nvidia Orin Nano for edge AI computing and STM32 for low-level motor control, running on the ROS 2 ecosystem.

 Hardware Architecture
* High-Level Compute: Nvidia Orin Nano (Ubuntu 22.04 LTS)
* Low-Level Control: STM32F407 running FreeRTOS
* Actuation: Tracked chassis with DC gear motors & incremental encoders
* Sensors: Single-line 2D LiDAR, ICM20948 IMU

 Tech Stack
* Robotics & Control: ROS 2 (Humble) | Nav2 | PID Control | Kinematics
* AI & Vision (Planned): YOLOv8 | PyTorch | Nvidia TensorRT | OpenCV

 Project Status

-  Hardware assembly and wiring validation
-  Network configuration (SSH & static IP binding)
-  Initial ROS 2 workspace compilation
-  Bench testing (Suspension test): Teleop node and chassis driver communication verified
-  Ground testing and motor PID tuning
-  2D SLAM mapping using Cartographer
-  Autonomous navigation (Nav2) and YOLO deployment

> Dev Note (Day 0): > The system has successfully passed the bench suspension test. SSH connection between the host PC and Orin board is stable. Waiting for full battery charge before initiating physical ground testing.

 Current Project Status
Phase 1: Hardware integration and initial system deployment.

-  Hardware assembly and wiring validation
-  Network configuration (SSH & static IP binding)
-  Initial ROS 2 workspace compilation
-  Bench testing: Teleop node and chassis driver communication verified
-  Phase 1.5 (Vision Setup): Astra RGB-D Camera initialized; RGB and Depth streams strictly verified via RViz2.
-  Phase 2: Ground testing and motor PID tuning
-  Phase 3: 2D SLAM mapping using Cartographer
-  Phase 4: Autonomous navigation (Nav2) and YOLO deployment


 Phase 3: AI Vision & Target Detection
Status: Successfully Deployed
Hardware: Nvidia Orin Core & Astra RGB-D Camera
Algorithm: YOLOv8n (Ultralytics) running on ROS2 Humble

Milestones Achieved:
-  Offline Model Injection: Successfully bypassed local network constraints by injecting pre-trained `yolov8n.pt` weights via SFTP.
-  Pipeline Bridge: Established forced relay bridge from Astra RGB stream (`/camera/color/image_raw`) to YOLO detection node.
-  QoS Penetration: Resolved ROS2 Quality of Service (QoS) mismatch, shifting Reliability Policy to `Best Effort` for real-time inference rendering.
-  Visualization: Achieved real-time object detection with bounding boxes and confidence scores in RViz2 (`/detected_image`).

Deployment Result:
![YOLOv8 Detection Result](yolo_vision_success.png)

 Autonomous Tea Disease Inspection Robot (Visual Perception Module)

 Project Overview
This repository contains the visual perception module for an autonomous agricultural inspection robot. It integrates a custom-trained YOLOv8 object detection model into a ROS2 framework, deployed natively on an NVIDIA Jetson Orin edge computing platform.

 Core Achievements & Technical Highlights
Local Native Training: Bypassed cloud-based API limitations by natively training the dataset on a local RTX GPU, extracting the pure `.pt` weights for seamless physical deployment.
ROS2 Topic Remapping: Successfully bridged the Astra RGBD camera stream (`/camera/color/image_raw`) with the YOLO detection node using precise ROS2 command-line remapping arguments (`--remap`).
QoS (Quality of Service) Optimization: Overcame strict ROS2 communication barriers by resolving reliability and durability mismatches. Configured the detection node and RViz2 visualization tool to utilize `Best Effort` and `Volatile` policies, ensuring zero-latency real-time video processing.

 Deployment & Launch Instructions
Here we will list the 3 terminal commands we used today

 Deployment & Launch Instructions

To replicate the visual perception pipeline, open three separate terminal sessions on the edge device and source the ROS2 workspace in each:
`source ~/wheeltec_ros2/install/setup.bash`

 Terminal 1: Initialize the Camera Stream
Launch the Astra RGBD camera node to publish the raw image topics.
```bash
ros2 launch turn_on_wheeltec_robot wheeltec_camera.launch.py

```
Terminal 2: Deploy the YOLOv8 Detection Node

Run the core detection node. This command loads the locally trained `best.pt` custom weights and uses `--remap` to correctly bridge the hardware camera topic with the neural network's input interface.

```bash
ros2 run ultralytics_ros2 detection_node --ros-args -p model:=/home/wheeltec/best.pt --remap /camera/image_raw:=/camera/color/image_raw

```
Terminal 3: Visualization & QoS Configuration

Launch RViz2 to verify the bounding boxes and real-time inference results.

```bash
rviz2

```

Critical QoS Configurations in RViz2:
To prevent image starvation caused by ROS2 DDS (Data Distribution Service) policy mismatches between the edge node and the visualization tool, you MUST manually configure the `/detected_image` topic settings in RViz2:

Reliability Policy: Set to `Best Effort`
Durability Policy: Set to `Volatile`


markdown
Autonomous Tea Disease Inspection Robot (Visual Perception Module)

 Project Overview
This repository contains the visual perception module for an autonomous agricultural inspection robot. It integrates a custom-trained YOLOv8 object detection model into a ROS2 framework, deployed natively on an edge computing platform.

 Core Achievements & Technical Highlights
Local Native Training: Bypassed cloud API limitations by natively training the dataset on a local GPU, extracting pure `.pt` weights for seamless physical deployment.
ROS2 Topic Remapping: Successfully bridged the Astra RGBD camera stream with the YOLO detection node using precise ROS2 command-line remapping.
QoS Optimization: Overcame strict ROS2 communication barriers by resolving reliability and durability mismatches, ensuring real-time video processing.

 Quick Start & Deployment Instructions

To replicate the visual perception pipeline, open three separate terminal sessions and source the ROS2 workspace in each: `source ~/wheeltec_ros2/install/setup.bash`

Terminal 1: Initialize the Camera Stream
```bash
ros2 launch turn_on_wheeltec_robot wheeltec_camera.launch.py

```

Terminal 2: Deploy the YOLOv8 Detection Nod

```bash
ros2 run ultralytics_ros2 detection_node --ros-args -p model:=/home/wheeltec/best.pt --remap /camera/image_raw:=/camera/color/image_raw

```

Terminal 3: Visualization & QoS Configuration

```bash
rviz2

```
Build & Deployment Instructions

1. Create the Package:

```bash
cd ~/wheeltec_ros2/src
ros2 pkg create --build-type ament_cmake disease_locator --dependencies rclcpp sensor_msgs vision_msgs cv_bridge image_transport

```

2. Configure `CMakeLists.txt`:
(Add the following immediately above `if(BUILD_TESTING)`)

```cmake
find_package(cv_bridge REQUIRED)

add_executable(locator_node src/locator_node.cpp)
ament_target_dependencies(locator_node rclcpp sensor_msgs vision_msgs cv_bridge image_transport)

install(TARGETS
  locator_node
  DESTINATION lib/${PROJECT_NAME}
)

```

3. Compile and Run:

```bash
cd ~/wheeltec_ros2
colcon build --packages-select disease_locator
source install/setup.bash
ros2 run disease_locator locator_node

```

Performance Diagnostic Report: Message Accumulation (Buffer Bloat)

 Symptom Observed

During real-time physical testing, a significant latency ("ghosting effect") was observed. When the target (tea disease image) was removed from the camera's physical field of view, the node continued to output `TARGET LOCKED` coordinates for several seconds before halting.

 Root Cause Analysis

1. Frame Rate Asymmetry: The raw Astra camera publishes high-frequency RGBD streams (~30 FPS), while the edge-deployed YOLOv8 inference engine processes frames at a lower frequency (e.g., 5-15 FPS) due to compute constraints.
2. DDS Queue Congestion: The ROS2 subscriber queue (History QoS) defaults to allowing un-processed frames to accumulate. Instead of dropping stale frames, the system forces the YOLO node to sequentially process "historical" images sitting in the buffer, causing severe pipeline lag.

 Next Optimization Steps (Action Plan)

* QoS Queue Reduction: Force a strict `Keep Last` history policy with a depth of `1` or `Best Effort` reliability to automatically drop outdated frames.
* Exact Time Synchronization: Implement `message_filters::sync_policies` to securely bind 2D bounding box timestamps with the exact corresponding depth image matrices.

* 3D Spatial Unification (TF2 Integration): Successfully integrated the ROS2 TF2 coordinate transform engine to perform real-time deprojection of 2D pixel coordinates. It automatically converts these into absolute 3D physical coordinates relative to the chassis center (`base_footprint`), providing precise, millimeter-level navigation data for the autonomous targeting and inspection operations.
