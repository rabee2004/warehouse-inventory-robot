# warehouse-inventory-robot
# 🏭 Warehouse Inventory Robot

An autonomous ROS2-based robot that navigates warehouse aisles, scans inventory using sensors, and updates stock information in real-time — eliminating the need for manual inventory counts.

![ROS2](https://img.shields.io/badge/ROS2-Humble-blue?logo=ros)
![Python](https://img.shields.io/badge/Python-3.10+-green?logo=python)
![License](https://img.shields.io/badge/License-MIT-yellow)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

---

## 📋 Table of Contents
- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Results](#results)
- [Future Work](#future-work)
- [Author](#author)

---

## Overview

Manual inventory tracking in warehouses is slow, error-prone, and labor-intensive. This project implements a fully autonomous mobile robot that:

- **Navigates** warehouse aisles using SLAM-based localization (ROS2 + Nav2)
- **Scans** inventory at each shelf using barcode/RFID sensors
- **Updates** stock counts in real-time to a central database
- **Alerts** warehouse staff when stock drops below threshold levels

---

## ✨ Features

| Feature | Details |
|---|---|
| **Autonomous Navigation** | Nav2 stack with SLAM-based localization (slam_toolbox) |
| **Multi-Sensor Integration** | LiDAR (obstacle detection) + Barcode/RFID (scanning) + IMU + Encoders |
| **Real-Time Stock Updates** | Publishes inventory data via ROS2 topics as each shelf is scanned |
| **Low Stock Alerts** | Automatic alerts when item count falls below configurable threshold |
| **CSV Report Export** | Auto-generates timestamped inventory reports |
| **Configurable Waypoints** | Define warehouse layout via JSON config — no code changes needed |
| **Localization Monitoring** | AMCL confidence tracking with automatic low-confidence warnings |

---

## System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   WAREHOUSE ROBOT SYSTEM                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌───────────────┐    ┌───────────────┐                 │
│  │  SLAM Manager │    │  Inventory    │                 │
│  │  (slam_node)  │    │  Robot Node   │                 │
│  │               │    │               │                 │
│  │ • Map building│    │ • Waypoint    │                 │
│  │ • Localization│    │   navigation  │                 │
│  │ • AMCL pose  │    │ • Scan trigger│                 │
│  └──────┬────────┘    └──────┬────────┘                 │
│         │ /map, /amcl_pose   │ /inventory/updates       │
│         ▼                    ▼                          │
│  ┌──────────────────────────────────────────┐           │
│  │         Inventory Database Node           │           │
│  │  • Stock management  • Low-stock alerts  │           │
│  │  • CSV export        • JSON persistence  │           │
│  └──────────────────────────────────────────┘           │
│                                                         │
│  Sensors: LiDAR → /scan | Scanner → /scanner/result     │
│  Navigation: Nav2 ActionServer ← navigate_to_pose       │
└─────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- **Ubuntu 22.04** (recommended)
- **ROS2 Humble** — [Installation guide](https://docs.ros.org/en/humble/Installation.html)
- **Nav2** — `sudo apt install ros-humble-navigation2`
- **slam_toolbox** — `sudo apt install ros-humble-slam-toolbox`
- **Python 3.10+**

---

## Installation

```bash
# 1. Clone the repository
git clone https://github.com/rabee-siddiqui/warehouse-inventory-robot.git
cd warehouse-inventory-robot

# 2. Install Python dependencies
pip install -r requirements.txt

# 3. Source ROS2
source /opt/ros/humble/setup.bash

# 4. Build the package (if using as ROS2 package)
colcon build
source install/setup.bash
```

---

## Usage

### Run all nodes

```bash
# Terminal 1: Start SLAM manager
ros2 run warehouse_robot slam_manager

# Terminal 2: Start inventory database node
ros2 run warehouse_robot inventory_database

# Terminal 3: Start main robot node (begins mission automatically)
ros2 run warehouse_robot inventory_robot
```

### Monitor inventory updates live

```bash
ros2 topic echo /inventory/updates
```

### Monitor low-stock alerts

```bash
ros2 topic echo /inventory/alerts
```

### Launch with Gazebo simulation

```bash
ros2 launch warehouse_robot warehouse_sim.launch.py
```

---

## Configuration

Edit `config/waypoints.json` to define your warehouse layout:

```json
{
  "waypoints": [
    { "id": "A1", "x": 1.0, "y": 0.0, "shelf": "A", "row": 1 },
    { "id": "A2", "x": 1.0, "y": 2.5, "shelf": "A", "row": 2 }
  ],
  "navigation": {
    "max_speed_mps": 0.5,
    "obstacle_clearance_m": 0.3
  }
}
```

---

## Project Structure

```
warehouse-inventory-robot/
├── src/
│   ├── inventory_robot.py      # Main navigation & scanning node
│   ├── slam_manager.py         # SLAM & localization manager
│   └── inventory_database.py   # Stock database & alert node
├── config/
│   └── waypoints.json          # Warehouse layout & waypoints
├── launch/
│   └── warehouse_sim.launch.py # Gazebo simulation launch file
├── maps/                       # Pre-built warehouse maps (.pgm/.yaml)
├── tests/                      # Unit tests
├── docs/                       # Additional documentation
├── requirements.txt
└── README.md
```

---

## How It Works

### 1. SLAM & Localization
The robot uses **slam_toolbox** to build an occupancy grid map of the warehouse. **AMCL** (Adaptive Monte Carlo Localization) tracks the robot's pose within the map using LiDAR scan matching + odometry fusion.

### 2. Autonomous Navigation
**Nav2** handles path planning and execution. Given a goal waypoint, Nav2 computes a collision-free path using the global planner (NavFn/Smac) and executes it with the local planner (DWA/TEB), avoiding dynamic obstacles in real-time.

### 3. Inventory Scanning
Upon reaching each waypoint, the robot triggers the barcode/RFID scanner. Scan results are published as JSON on `/scanner/result`, picked up by the database node, and stored with timestamps.

### 4. Real-Time Updates
The `InventoryDatabase` node maintains a live stock database, publishes low-stock alerts, and auto-exports CSV reports every 60 seconds and at mission end.

---

## Results

- ✅ Successfully navigated a simulated 3-aisle, 12-shelf warehouse
- ✅ Real-time stock updates with < 1 second latency
- ✅ Automatic low-stock detection and alerting
- ✅ SLAM localization maintained with > 85% confidence throughout mission

---

## Future Work

- [ ] Integration with real barcode scanner hardware (USB/RS-232)
- [ ] Multi-robot coordination for parallel aisle scanning
- [ ] 3D shelf scanning with depth camera (Intel RealSense)
- [ ] REST API for web dashboard integration
- [ ] Time-of-flight sensors for precise shelf-distance measurement

---

## Author

**Rabee Siddiqui**
B.Tech Robotics & Automation — Bharati Vidyapeeth University, Pune
📧 Siddiquirabee@gmail.com
🔗 [LinkedIn](https://www.linkedin.com/in/rabee-siddiqui)

---

## License

MIT License — see [LICENSE](LICENSE) for details.
