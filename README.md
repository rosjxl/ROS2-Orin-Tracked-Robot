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

*  Visual Servoing & Chassis Control Successfully closed the loop between machine vision and physical motor control. The system dynamically calculates angular and linear velocities based on 3D coordinates derived from YOLO and the depth camera. It features an integrated ABS-level environmental monitoring logic that triggers a zero-latency emergency stop upon target loss or reaching the 30cm operational limit, ensuring hardware safety.


Final line-up: 1.Chassis Master Control Wake‑up
```
source ~/wheeltec_ros2/install/setup.bash
ros2 launch turn_on_wheeltec_robot turn_on_wheeltec_robot.launch.py
 ```

2.Awaken the Depth Eye
```
source ~/wheeltec_ros2/install/setup.bash
ros2 launch turn_on_wheeltec_robot wheeltec_camera.launch.py
```

3.Deploy YOLO Vision Neural Network
```
source ~/anaconda3/bin/activate wheeltec
source ~/wheeltec_ros2/install/setup.bash
python3 ~/wheeltec_ros2/install/ultralytics_ros2/lib/ultralytics_ros2/detection_node --ros-args -p model:=/home/wheeltec/best.pt -p conf:=0.6 --remap /camera/image_raw:=/camera/color/image_raw
```

4.Set Up the TF Spatial Skybridge
```
source ~/wheeltec_ros2/install/setup.bash
ros2 run tf2_ros static_transform_publisher 0.2 0.0 0.3 0 0 0 base_footprint camera_link
```

5.Inject the C++ Control and Data Logging Brainstem
```
source ~/wheeltec_ros2/install/setup.bash
ros2 run disease_locator locator_node
```

Key Features:

Multi‑modal Data Fusion (3D Matrix Projection):  
Using the pinhole camera model, the 2D pixel coordinates \((u, v)\) extracted by YOLO are fused with the corresponding depth value (Depth) and back‑projected to generate a 3D point in the camera coordinate system. The point is then projected in real time to the chassis physical coordinate system through TF2 tree matrix transformation.  

IFF Defense Line (IFF Class Filtering):  
The node integrates a strict filtering mechanism. The trigger is activated only when the class ID of the detection topic exactly matches the custom disease (`class_id == "0"`), completely shielding false‑positive interference from complex backgrounds such as faces, phones, and foreign objects.  

Safety Killer Threshold and Battlefield Black Box (Inspection Logger):  
- Chasing phase: When the distance to the target is > 0.35 m, the system publishes speed commands to guide the robot to approach steadily.  
- Killer record: When the distance to the target is ≤ 0.35 m, the chassis performs emergency braking (brake). Simultaneously, it triggers a file stream lock and, under a 5‑second cooling protection (to prevent data bursts), automatically appends an inspection report (`Inspection_Log.csv`) containing [absolute timestamp, X coordinate, Y coordinate, Z coordinate] to the local hard disk.


In real bench‑scale vehicle tests and physical image‑guided trials, the entire pipeline demonstrated extremely high perception‑control convergence speed. However, under complex laboratory/field backgrounds, the system encountered the following technical challenges, which have now been consolidated into core outcomes of the system’s robustness design:

Topic Mismatch:  
In the early integration phase, all nodes were alive but the system remained silent. Using the hacker‑grade command `ros2 topic echo` to capture the underlying inter‑com channels, it was discovered that the vision publisher and the control subscriber were misaligned on the topics `/detections` and `/yolo_detections`. This communication mismatch has now been corrected by a source‑level surgical adjustment that unifies both sides onto the standard frequency band.

False Positive Suppression:  
Because the negative samples in the custom dataset were limited during early training, the AI tended to over‑generalise and misclassify objects with similar shapes or textures (e.g., facial contours) as true targets. This project resolved the issue with a two‑pronged defence tactic:
- The confidence threshold of the vision node was forcibly raised to `conf:=0.6` to filter out low‑probability noise.
- A category conditional branch was deeply nested in the C++ motion kernel to completely discard empty IDs and non‑target bounding boxes.
