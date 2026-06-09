# Markerless External Camera based Navigation System

## Overview

This repository contains a proof-of-concept perception and navigation pipeline for a mobile robot using Intel RealSense, YOLO object detection, tracking, localization, occupancy mapping, path planning, and control.

The project includes:
- Real-time object detection using YOLO models
- IoU-based tracking of detected objects
- Depth-based 3D localization from RealSense data
- Occupancy grid mapping with obstacle inflation
- A* path planning on a 2D grid
- Pure pursuit-style control commands
- ROS2-enabled simulation/robot command publishing

## Repository Structure

- `main.py` - Desktop demo pipeline using RealSense, YOLO detection, tracker, localization, grid mapping, A* planning, and local control. Goal selection is done with mouse clicks on detected objects.
- `simulation_go2.py` - ROS2 node implementation with camera perception, object/robot detection, tracking, occupancy mapping, A* planning, and publishing `Twist` commands to `/cmd_vel`.
- `simulation_diff_drive.py` - Another ROS2 pipeline variant targeting a differential-drive simulation topic namespace.
- `realsense_cam.py` - Intel RealSense wrapper for color/depth streaming and intrinsics.
- `yolodetect_botrob.py` - YOLO-based detector that uses a single `yolov8m.pt` model and maps one bottle detection to a `robot` label.
- `yolodetect_go2.py` - Detector combining a generic object model (`yolov8m.pt`) with a dedicated robot model (`best.pt`).
- `track.py` - Simple IoU matching tracker for persistent object IDs.
- `localization.py` - Converts pixel coordinates and depth into 3D world points and extracts robot pose from tracked objects.
- `map_builder.py` - Occupancy grid generation, obstacle insertion, inflation, and coordinate transforms.
- `astar_planner.py` - Grid-based A* path planner converting world coordinates to grid cells and back.
- `controller1.py` - Pure pursuit path tracking controller.
- `controller5.py` - Alternative goal-oriented controller for differential drive-style motion.
- `best.pt` - Custom-trained YOLO model expected to detect the robot.
- `yolov8m.pt` - General YOLO object detection model.

## Features

- Detects objects and robot candidates in camera frames
- Tracks objects across frames with persistent IDs
- Uses depth to localize objects in 3D space
- Builds a local occupancy grid from detected obstacles
- Plans a path to a clicked object location using A*
- Computes navigation commands for robot control
- Supports both standalone Python and ROS2 execution paths

## Requirements

- Python 3.x
- `opencv-python`
- `numpy`
- `pyrealsense2`
- `ultralytics`
- `rclpy` / ROS2 (for `simulation_go2.py` and `simulation_diff_drive.py`)

## Usage

### Run the local desktop demo

```bash
python main.py
```

- A window named `Full Perception Pipeline` will appear.
- Click on a detected object/robot bounding box to set a navigation goal.
- The pipeline will estimate robot pose and compute a path.
- Press `q` to quit.

### Run the ROS2 navigation node

```bash
python simulation_go2.py
```

- Requires a ROS2 environment and the appropriate topics available:
  - `/cmd_vel` publisher
  - `/odom` subscriber
- Uses both `yolov8m.pt` and `best.pt` for object and robot detection.

### Run the differential drive variant
Run the below command In terminal 1 :
source /opt/ros/humble/setup.bash
ros2 launch ros_gz_sim_demos diff_drive.launch.py

Run the below command in terminal 2:
```bash
python simulation_diff_drive.py
```

- Uses a different ROS2 topic namespace (`/model/vehicle_blue/cmd_vel`, `/model/vehicle_blue/odometry`).

## Notes

- The system assumes an Intel RealSense camera is connected and available.
- `main.py` and the ROS2 scripts use the 640x480 camera stream.
- Goal selection is performed by clicking on object bounding boxes in the displayed image.
- The occupancy grid is built from detected obstacle positions and inflated for safety.
- Path planning currently uses a top-down 2D grid and a simplified world-to-grid transform.

## Suggestions

- If needed, adjust the following values in the scripts:
  - YOLO confidence and IoU thresholds
  - occupancy grid `resolution`
  - planner and controller parameters
  - inflation radius for obstacles

## License

This repository does not include a license file. Add one if you plan to share or reuse the code publicly.
