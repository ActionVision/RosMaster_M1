# YahboomCar ROS2 工作区工程代码分析文档

> **适用平台**: Yahboom 品牌 ROS2 智能小车（A1 阿克曼转向、M1 麦克纳姆轮、R2 等车型）
> **ROS2 发行版**: Humble
> **编程语言**: Python (主要) + C++ 
> **工作区路径**: `~/yahboomcar_ros2_ws/src`
> **核心硬件**: RDK X5 / Jetson Orin / Xavier NX + M1 麦克纳姆轮小车
> **文档版本**: v2.0（新增与 yahboom_ws 的对比分析及 RDK X5 适配指南）

---

## 目录

1. [项目整体架构概览](#1-项目整体架构概览)
2. [包模块详解](#2-包模块详解)
   - [2.1 消息定义包 (interfaces, auto_interfaces, yahboomcar_msgs, yahboomcar_web_savmap_interfaces)](#21-消息定义包)
   - [2.2 底层驱动 (yahboomcar_bringup)](#22-底层驱动-yahboomcar_bringup)
   - [2.3 里程计节点 (yahboomcar_base_node)](#23-里程计节点-yahboomcar_base_node)
   - [2.4 控制输入 (yahboomcar_ctrl)](#24-控制输入-yahboomcar_ctrl)
   - [2.5 机器人描述 (yahboomcar_description)](#25-机器人描述-yahboomcar_description)
   - [2.6 激光雷达 (yahboomcar_laser)](#26-激光雷达-yahboomcar_laser)
   - [2.7 导航 (yahboomcar_nav)](#27-导航-yahboomcar_nav)
   - [2.8 多机器人 (yahboomcar_multi)](#28-多机器人-yahboomcar_multi)
   - [2.9 ASTRA摄像头视觉 (yahboomcar_astra)](#29-astra摄像头视觉-yahboomcar_astra)
   - [2.10 深度相机 (yahboomcar_depth)](#210-深度相机-yahboomcar_depth)
   - [2.11 视觉处理 (yahboomcar_vision)](#211-视觉处理-yahboomcar_vision)
   - [2.12 MediaPipe人体检测 (yahboomcar_mediapipe)](#212-mediapipe人体检测-yahboomcar_mediapipe)
   - [2.13 语音控制 (yahboomcar_voice_ctrl / yahboomcar_voice_ctrl_depth)](#213-语音控制)
   - [2.14 大模型AI服务 (largemodel)](#214-大模型ai服务-largemodel)
   - [2.15 自动驾驶 (auto_drive)](#215-自动驾驶-auto_drive)
   - [2.16 辅助工具包](#216-辅助工具包)
3. [ROS2 话题/服务/Action 汇总](#3-ros2-话题服务action-汇总)
4. [二次开发指南](#4-二次开发指南)

---

## 1. 项目整体架构概览

本项目是一个基于 ROS2 Humble 的完整智能小车控制系统，所有包位于 `yahboomcar_ws/src/` 下，共 **26 个 ROS2 包**。

### 架构分层

```
┌──────────────────────────────────────────────────────────────┐
│                      应用层 (Application)                     │
│  largemodel (AI大模型交互)  │  auto_drive (自动驾驶)          │
│  yahboomcar_vision (视觉)  │  yahboomcar_mediapipe (人体检测)│
│  yahboomcar_voice_ctrl* (语音控制)                           │
├──────────────────────────────────────────────────────────────┤
│                      功能层 (Function)                        │
│  yahboomcar_laser (激光避障/跟随)   │  yahboomcar_nav (导航)  │
│  yahboomcar_astra (摄像头跟随)      │  yahboomcar_depth (深度)│
│  yahboomcar_multi (多机器人)        │  text_chat (文字聊天)   │
├──────────────────────────────────────────────────────────────┤
│                      驱动层 (Driver)                          │
│  yahboomcar_bringup (底盘驱动)      │  yahboomcar_base_node   │
│  yahboomcar_ctrl (键盘/手柄控制)    │                        │
├──────────────────────────────────────────────────────────────┤
│                      描述/接口层 (Description/Interface)      │
│  yahboomcar_description (URDF/模型) │  yahboomcar_msgs (消息) │
│  interfaces / auto_interfaces       │  yahboomcar_web_savmap  │
│  laserscan_to_point_publisher       │  robot_pose_publisher   │
└──────────────────────────────────────────────────────────────┘
```

### 硬件支持
| 车型 | 驱动方式 | 适用包 |
|------|---------|-------|
| **A1** | 阿克曼转向（前轮转向） | `Ackman_driver_A1.py`, `base_node_A1` |
| **M1** | 麦克纳姆轮（全向移动） | `Mcnamu_driver_M1.py`, `base_node_M1` |
| **R2** | 两轮差速 | `base_node_R2` |

---

## 2. 包模块详解

---

### 2.1 消息定义包

#### 2.1.1 `interfaces` 包（大模型接口定义）
- **类型**: `ament_cmake`（ROS2 接口定义包）
- **文件结构**:
  - `action/Rot.action` — 大模型动作执行 Action
  - `srv/Audio.srv` — 语音播放服务
  - `srv/Audio2.srv` — 文字转语音（TTS）服务
  - `srv/LargeScaleModel.srv` — 大模型推理服务
  - `srv/Qwen25.srv` — 通义千问 2.5 模型服务
  - `srv/Vision.srv` — 视觉服务

#### 2.1.2 `auto_interfaces` 包（自动驾驶接口定义）
- **文件结构**:
  - `action/Rot.action` — 与 interfaces 相同的动作定义
  - `msg/DetectionObject.msg` — YOLO 检测到的目标（class_name, confidence, bbox）
  - `msg/LlmAutoDrive.msg` — LLM 自动驾驶指令（instruct + value）
  - `msg/LlmRequest.msg` — LLM 请求（robot_feedback + llm_request）
  - `msg/RoadRoute.msg` — 道路导航路径（start_pose + goal_pose + cancel_nav）
  - `srv/WebSaveMap.srv` — 网页保存地图服务

#### 2.1.3 `yahboomcar_msgs` 包（自定义消息）
- **定义**:
  - `msg/ServoControl.msg` — 舵机控制（s1, s2 两个舵机角度）
  - `msg/Position.msg` — 位置信息（anglex, angley, distance）
  - `msg/PointArray.msg` — 点数组（geometry_msgs/Point[]）

#### 2.1.4 `yahboomcar_web_savmap_interfaces` 包（网页保存地图接口）
- **定义**:
  - `src/srv/WebSaveMap.srv` — 保存地图（mapname → response）

---

### 2.2 底层驱动 `yahboomcar_bringup`

- **类型**: Python (`ament_python`)
- **核心功能**: 底盘运动控制、传感器数据采集

#### 核心文件说明

| 文件名 | 功能 |
|--------|------|
| `Ackman_driver_A1.py` | A1 阿克曼车型驱动：接收 `cmd_vel`，发布 IMU/磁力计/电池/关节状态 |
| `Mcnamu_driver_M1.py` | M1 麦克纳姆轮驱动：同上 + RGB 灯效控制 |
| `Ackman_driver_A1_descri.py` | A1 描述版驱动（关节状态发布略有不同） |
| `Mcnamu_driver_M1_descri.py` | M1 描述版驱动 |
| `calibrate_angular_A1/M1.py` | 角速度校准脚本 |
| `calibrate_linear_A1/M1.py` | 线速度校准脚本 |
| `patrol_A1/M1.py` | 巡逻模式脚本 |
| `transform_utils.py` | 坐标变换工具函数（四元数↔欧拉角） |

#### 关键依赖库
- `Rosmaster_Lib` — Yahboom 底盘控制库（控制电机、舵机、蜂鸣器、RGB 灯等）
- 订阅：`cmd_vel` (Twist), `Buzzer` (Bool), `Servo` (ServoControl)
- 发布：`imu/data_raw` (Imu), `imu/mag` (MagneticField), `voltage` (Float32), `vel_raw` (Twist), `joint_states` (JointState), `edition`

#### 主要数据流
```
cmd_vel → 底盘控制库 → 电机PWM
传感器 → 底盘控制库 → IMU/磁力计/电池/里程计 → ROS话题
```

---

### 2.3 里程计节点 `yahboomcar_base_node`

- **类型**: C++ (`ament_cmake`)
- **核心功能**: 基于编码器和 IMU 融合计算里程计

#### 可执行文件
| 文件名 | 车型 |
|--------|------|
| `base_node_R2.cpp` | R2 两轮差速车型 |
| `base_node_A1.cpp` | A1 阿克曼车型 |
| `base_node_M1.cpp` | M1 麦克纳姆轮车型 |

#### 依赖
- `tf2`, `tf2_ros` — 坐标变换
- `nav_msgs` — 里程计消息
- `geometry_msgs` — 几何消息
- `turtlesim` (仅 R2)

---

### 2.4 控制输入 `yahboomcar_ctrl`

- **类型**: Python (`ament_python`)
- **核心功能**: 键盘/手柄控制小车运动

#### 可执行文件

| 命令名 | 功能 |
|--------|------|
| `yahboom_keyboard` | 键盘控制：i/u/o/j/k/l 方向键，q/z 调速 |
| `yahboom_joy_A1` | A1 手柄控制：摇杆控制运动、按键控制舵机/蜂鸣器/RGB灯、取消导航 |
| `yahboom_joy_M1` | M1 手柄控制：类似 A1 + RGB 灯效 |

#### 关键话题
- 发布：`cmd_vel` (Twist), `JoyState` (Bool), `Buzzer`, `RGBLight`, `Servo`
- 订阅：`joy` (Joy)

---

### 2.5 机器人描述 `yahboomcar_description`

- **类型**: Python (`ament_python`)
- **核心功能**: URDF 模型、3D 网格文件、Rviz 可视化配置

#### 文件结构
| 目录 | 内容 |
|------|------|
| `urdf/` | URDF 模型文件（A1/M1 车型描述） |
| `meshes/A1Ackermann/` | A1 阿克曼车型 3D 网格文件（.stl） |
| `meshes/A1USBAckermann/` | A1 USB 版网格文件 |
| `meshes/M1Mecanum/` | M1 麦克纳姆轮网格文件 |
| `meshes/M1USBMecanum/` | M1 USB 版网格文件 |
| `rviz/` | RViz 配置文件 |
| `launch/` | 显示启动文件（支持单/多机器人） |

#### Launch 说明
- `display_A1.launch.py` — A1 模型显示
- `display_M1.launch.py` — M1 模型显示
- `description_*_multi_robot*.launch.py` — 多机器人模型显示

---

### 2.6 激光雷达 `yahboomcar_laser`

- **类型**: Python (`ament_python`)
- **核心功能**: 基于激光雷达的避障和跟随

#### 可执行文件

| 命令名 | 功能 |
|--------|------|
| `laser_Avoidance_A1` | A1 激光避障：检测前方/左/右障碍物并转向避让 |
| `laser_Avoidance_M1` | M1 激光避障 |
| `laser_Tracker_A1` | A1 激光跟踪：跟随前方最近物体 |
| `laser_Tracker_M1` | M1 激光跟踪 |
| `laser_Warning_M1` | M1 激光预警 |

#### 工具模块
- `common.py` — PID 控制器类

#### 避障原理
根据激光雷达扫描数据，将 360° 范围划分为前/左/右三个区域，根据不同区域的障碍物分布决定转向方向。

```
前+左+右都有障碍 → 右转
前+右有障碍 → 左转
前+左有障碍 → 右转
仅前有障碍 → 左转
无障碍 → 直行
```

---

### 2.7 导航 `yahboomcar_nav`

- **类型**: Python (`ament_python`)
- **核心功能**: SLAM 建图、自主导航、定位

#### Launch 文件说明（关键）

| Launch 文件 | 功能 |
|-------------|------|
| `cartographer_bringup_launch.py` | Cartographer 建图启动 |
| `cartographer_launch.py` | Cartographer 在线建图 |
| `cartographer_localization_launch.py` | Cartographer 纯定位模式 |
| `map_gmapping_launch.py` | Gmapping 建图 |
| `map_slam_toolbox_launch.py` | SLAM Toolbox 建图 |
| `map_rtabmap_launch.py` | RTAB-Map 建图 |
| `navigation_dwa_launch.py` | DWA 局部规划器导航 |
| `navigation_teb_launch.py` | TEB 局部规划器导航 |
| `navigation_cartodwb_launch.py` | Carto + DWA 组合导航 |
| `display_nav_launch.py` | 导航可视化 |
| `save_map_launch.py` | 保存地图 |
| `laser_bringup_launch.py` | 激光雷达启动 |
| `localization_imu_odom.launch.py` | IMU + 里程计定位 |

#### 参数配置

| 文件 | 用途 |
|------|------|
| `params/dwa_nav_params.yaml` | DWA 规划器参数 |
| `params/teb_nav_params.yaml` | TEB 规划器参数 |
| `params/cartoteb_nav_params.yaml` | Carto + TEB 参数 |
| `params/cartodwa_nav_params.yaml` | Carto + DWA 参数 |
| `params/rtabmap_nav_params.yaml` | RTAB-Map 参数 |
| `params/mapper_params_online_sync.yaml` | SLAM Toolbox 参数 |
| `params/lds_2d.lua` | Cartographer Lua 配置 |

#### 代码
- `scan_filter.py` — 激光扫描数据滤波

#### 支持的 SLAM 方案
| 方法 | 适用场景 |
|------|---------|
| Cartographer (谷歌) | 推荐，精度高，支持 2D/3D |
| SLAM Toolbox | 功能丰富，支持继续建图 |
| RTAB-Map | 视觉+激光融合 SLAM |
| Gmapping | 经典 2D SLAM |

#### 支持的局部规划器
| 规划器 | 特点 |
|--------|------|
| DWA (Dynamic Window Approach) | 动态窗口法，适合平滑运动 |
| TEB (Timed Elastic Band) | 时间弹性带，可处理复杂场景 |

---

### 2.8 多机器人 `yahboomcar_multi`

- **类型**: Python (`ament_python`)
- **核心功能**: 多机器人编队跟随

#### 可执行文件

| 命令名 | 功能 |
|--------|------|
| `yahboomcar_A1_ctrl` | A1 多机器人控制（joy） |
| `yahboomcar_M1_ctrl` | M1 多机器人控制（joy） |
| `get_follower_point` | 获取跟随目标点（通过位姿订阅） |
| `pub_follower_goal` | 发布跟随目标点 |
| `singlePID` | 单 PID 控制器 |

#### Launch 文件
- 支持 **2 台机器人**同时运行（robot1 / robot2）
- 每个机器人有独立的导航、AMCL定位、控制话题

#### 参数配置
- `param/` — 多机器人参数 YAML 文件

---

### 2.9 ASTRA摄像头视觉 `yahboomcar_astra`

- **类型**: Python (`ament_python`)
- **核心功能**: 基于 ASTRA（奥比中光）深度摄像头的视觉追踪

#### 可执行文件（24个节点）

| 类别 | 节点名 | 功能 |
|------|--------|------|
| **追踪类** | `colorTracker` | 颜色追踪（RGB） |
| | `qrTracker` | 二维码追踪 |
| | `faceTracker` | 人脸追踪 |
| | `gestureTracker` | 手势追踪 |
| | `poseTracker` | 姿态追踪 |
| | `apriltagTracker` | Apriltag 标签追踪 |
| | `monoTracker` | 单目标追踪（KCF） |
| **跟随类** | `colorFollow` | 颜色跟随（小车跟着颜色移动） |
| | `faceFollow` | 人脸跟随 |
| | `gestureFollow` | 手势跟随 |
| | `poseFollow` | 姿态跟随 |
| | `qrFollow` | 二维码跟随 |
| | `apriltagFollow` | Apriltag 跟随 |
| | `follow_line` | 巡线跟随 |
| **其他** | `HandCtrl` | 手势控制 |
| | `qrCtrl` | 二维码控制 |

#### 公共模块
- `common/astra_common.py` — ASTRA 相机通用函数
- `common/follow_common.py` — 跟随算法通用函数
- `common/track_common.py` — 追踪算法通用函数
- `common/media_common.py` — 媒体处理通用函数

#### 核心逻辑
```
摄像头采集 → OpenCV处理 → 检测目标位置 → PID控制 → cmd_vel → 底盘运动
```

---

### 2.10 深度相机 `yahboomcar_depth`

- **类型**: Python (`ament_python`)
- **核心功能**: 深度相机应用（距离测量、体积计算、高级视觉跟随）

#### 可执行文件

| 模块 | 节点名 | 功能 |
|------|--------|------|
| **Basic** | `depth_to_color` | 深度图对齐到彩色图 |
| | `get_center_dis` | 获取图像中心点距离 |
| | `calculate_volume` | 体积估算 |
| **Advanced** | `depth_colorTracker` | 深度辅助的颜色追踪 |
| | `depth_faceFollow` | 深度辅助的人脸跟随 |
| | `follow_line` | 深度+巡线 |
| | `apriltagFollow` | 深度+Apriltag跟随 |
| | `qrFollow` | 深度+二维码跟随 |
| | `Edge_Detection` | 边缘检测 |
| **MediaPipe** | `gestureFollow` | 深度+手势跟随 |
| | `poseFollow` | 深度+姿态跟随 |
| **KCF** | `KCF_Tracker` | KCF 目标跟踪算法 |

---

### 2.11 视觉处理 `yahboomcar_vision`

- **类型**: Python (`ament_python`)
- **核心功能**: 通用视觉处理应用

#### 可执行文件

| 节点名 | 功能 |
|--------|------|
| `create_qrcode` | 生成二维码 |
| `parse_qrcode` | 解析二维码 |
| `detect_pose` | 人体姿态检测 |
| `detect_object` | 目标检测 |
| `simple_ar` | 简单 AR 增强现实 |
| `astra_rgb_image` | ASTRA 相机 RGB 图像采集 |
| `astra_depth_image` | ASTRA 相机深度图像采集 |
| `pub_image` | 图像发布 |

#### 依赖
- `qrcode`, `Pillow` — 二维码生成/解析
- `cv_bridge` — ROS ↔ OpenCV 图像转换

---

### 2.12 MediaPipe人体检测 `yahboomcar_mediapipe`

- **类型**: Python (`ament_python`)
- **核心功能**: Google MediaPipe 人体检测演示

#### 可执行文件

| 节点名 | 功能 |
|--------|------|
| `01_HandDetector` | 手部关键点检测 |
| `02_PoseDetector` | 人体姿态检测（33个关键点） |
| `03_Holistic` | 全身检测（手+脸+姿态） |
| `04_FaceMesh` | 面部网格检测 |
| `05_FaceDetection` | 人脸检测 |
| `06_FaceLandmarks` | 面部关键点 |
| `07_Objectron` | 3D物体检测 |
| `08_VirtualPaint` | 虚拟画笔（手指跟踪绘画） |
| `09_HandCtrl` | 手势控制 |
| `10_GestureRecognition` | 手势识别 |
| `11_FindHand` | 找手演示 |
| `12_FingerTrajectory` | 手指轨迹跟踪 |

#### 工具模块
- `media_library.py` — MediaPipe 通用函数库

---

### 2.13 语音控制

#### 2.13.1 `yahboomcar_voice_ctrl`（基础语音控制）
- **功能**: 语音控制摄像头追踪/跟随（与 yahboomcar_astra 功能对应）
- **节点**: 与 astra 包相同的追踪/跟随节点，但可通过语音触发

#### 2.13.2 `yahboomcar_voice_ctrl_depth`（深度语音控制）
- **功能**: 语音控制 + 深度相机的高级应用
- **节点**: 
  - `voice_get_dist` — 语音获取距离
  - `colorSelect` — 颜色选择
  - `voice_face_follow` — 语音人脸跟随
  - `voice_colorTracker` — 语音颜色追踪
  - `voice_apriltagFollow` — 语音 AprilTag 跟随
  - `voice_qrFollow` — 语音二维码跟随
  - `voice_follow_line` — 语音巡线
  - `voice_gestureFollow` — 语音手势跟随
  - `voice_poseFollow` — 语音姿态跟随
  - `voice_KCF_Tracker` — 语音 KCF 跟踪

---

### 2.14 大模型AI服务 `largemodel`

- **类型**: Python (`ament_python`)
- **核心功能**: 集成大语言模型（LLM）实现智能语音交互、任务规划与执行

#### 架构设计 — 双模型推理模式

```
用户语音 → ASR（语音识别）
     ↓
决策层大模型（任务规划：百炼/Dify）
     ↓
执行层大模型（动作生成：通义千问多模态/Dify）
     ↓
ActionServer（解析动作列表并执行）
     ↓
底盘/舵机/导航/视觉追踪等执行单元
```

#### 核心文件

| 文件 | 功能 |
|------|------|
| `model_service.py` | 大模型服务主节点：处理 ASR/Seewhat 请求，双模型推理 |
| `asr.py` | 语音识别节点：关键词唤醒（KWS）+ VAD 检测 + ASR 转换 |
| `action_service_usb.py` | USB 摄像头版动作执行服务器（重要！1205行核心逻辑） |
| `action_service_nuwa.py` | 女娲摄像头版动作执行服务器 |
| `autonav.py` | 自主导航节点 |
| **utils/large_model_interface.py** | 大模型接口层（通义千问、Dify、讯飞、百度等） |
| **utils/dify_client2.py** | Dify 平台 API 客户端 |
| **utils/mic_serial.py** | 麦克风串口通信（KWS 唤醒词） |
| **utils/promot_nuwa.py** | 女娲版提示词模板 |
| **utils/promot_usb.py** | USB 版提示词模板 |

#### ActionServer 支持的动作函数（30+种）

| 动作函数 | 功能 |
|----------|------|
| `navigation(point_name)` | 导航到地图上的命名目标点 |
| `seewhat()` | 拍照并用多模态模型分析场景 |
| `set_cmdvel(lx, ly, az, duration)` | 设定速度并运动 |
| `move_left(angle, speed)` | 左转指定角度 |
| `move_right(angle, speed)` | 右转指定角度 |
| `drift()` | 漂移动作 |
| `dance()` | 跳舞动作 |
| `wait(duration)` | 等待指定时间 |
| `servo1_move(angle)` | 1号舵机转到指定角度（水平） |
| `servo2_move(angle)` | 2号舵机转到指定角度（垂直） |
| `servo_init()` | 舵机复位 |
| `servo_shake()` | 舵机摇头 |
| `servo_nod()` | 舵机点头 |
| `apriltagTracker/Follow` | AprilTag 追踪/跟随 |
| `faceTracker/Follow` | 人脸追踪/跟随 |
| `gestureTracker/Follow` | 手势追踪/跟随 |
| `poseTracker/Follow` | 姿态追踪/跟随 |
| `qrTracker/Follow` | 二维码追踪/跟随 |
| `colorTrack/Follow(color)` | 颜色追踪/跟随（红/绿/蓝/黄） |
| `monoTracker(x1,y1,x2,y2)` | 单目标追踪（指定ROI） |
| `follow_line(color)` | 巡线 |

#### 支持的大模型平台

| 平台 | 用途 | 地区 |
|------|------|------|
| 阿里云百炼（通义千问） | 决策层/执行层推理 | 🇨🇳 国内 |
| 阿里云通义千问多模态 | 执行层（带图像输入） | 🇨🇳 国内 |
| 阿里云语音识别（Paraformer/Gummy） | ASR 在线识别 | 🇨🇳 国内 |
| 阿里云语音合成（CosyVoice） | TTS 在线合成 | 🇨🇳 国内 |
| 百度智能云语音合成 | TTS 在线合成 | 🇨🇳 国内 |
| SenseVoiceSmall | ASR 本地离线识别 | 🇨🇳 国内 |
| Piper TTS | TTS 本地离线合成 | 通用 |
| Dify 开源平台 | 应用编排（自建API） | 🌍 国际 |
| 讯飞语音（ASR/TTS） | 语音识别+合成 | 🌍 国际 |

#### 工作流程

```
1. 用户说唤醒词（"你好小豆"等）
2. KWS 模块检测到唤醒 → 发布 wakeup 信号
3. 小车播放应答音效 → 开始录音
4. VAD 检测人声结束 → 保存 WAV 文件
5. ASR 识别（本地SenseVoice/在线阿里云/讯飞）
6. ASR结果发布到 asr 话题
7. model_service 收到后进入双模型推理：
   a. 决策层大模型进行任务规划（如"先去桌子再去找人"）
   b. 执行层大模型生成动作列表（JSON格式）
8. 通过 ActionClient 发送到 ActionServer
9. ActionServer 解析 JSON 动作列表并顺序执行
10. 执行完成后发布 "finish" 状态
```

---

### 2.15 自动驾驶 `auto_drive`

- **类型**: Python (`ament_python`)
- **核心功能**: YOLOv11 道路标志检测 + 车道线保持 + 自主导航

#### 可执行文件

| 节点名 | 功能 |
|--------|------|
| `auto_drive.py` | 自动驾驶主节点：车道保持 + 交通标志响应 + 路径规划 |
| `yolo_detect.py` | YOLO 道路标志检测节点：识别14种交通标志 |
| `cali_midline.py` | 车道中线标定 |
| `example_route.py` | 路径示例生成 |
| `pub_video.py` | 视频发布 |
| `record_video.py` | 视频录制 |

#### 工具模块
- `utils/yolov11_infer.py` — YOLOv11 推理引擎（支持 RDK 硬件加速）
- `utils/common_tools.py` — PID 控制器、RGB 话题获取等通用工具

#### 可识别的交通标志（14类）

| ID | 名称 | 含义 | 行为 |
|----|------|------|------|
| 0 | greenlight | 绿灯 | 继续前进 |
| 1 | honking | 鸣笛区域 | 播放鸣笛音效 |
| 2 | parkA | 停车场A | 入库动作 |
| 3 | parkB | 停车场B | 入库动作 |
| 4 | redlight | 红灯 | 停车等待 |
| 5 | school | 学校区域 | 减速鸣笛 |
| 6 | sidewalk | 人行道 | 减速 |
| 7 | sidewalk_ground | 地面人行道 | (已忽略) |
| 8 | speedlimit | 限速 | 减速 |
| 9 | stop | 停止 | 停车3秒 |
| 10 | straight | 直行 | 保持直行 |
| 11 | turnright | 右转 | 执行右转 |
| 12 | turnright_ground | 地面右转 | (已忽略) |
| 13 | yellowlight | 黄灯 | 停车 |

#### 车道线保持逻辑
```
YOLO检测左右车道线 → 计算中心点偏差 → PID控制 → cmd_vel
三种模式：
  模式0：左右线均存在 → 计算角度误差 → PID控制转向
  模式1：仅左线 → 左线偏航控制
  模式3：无线 → 盲区处理
```

#### 推理硬件支持
- **RDK (地平线)**: 使用 `hobot_dnn` 硬件加速
- **通用**: 支持 `torch` / `ultralytics` CPU 推理

---

### 2.16 辅助工具包

#### `text_chat` 包
- **功能**: 简单的文字聊天交互（与大模型配套使用）
- **节点**: `text_chat` — 文本聊天界面

#### `laserscan_to_point_publisher` 包
- **功能**: 将 LaserScan 数据转换为 PointCloud 点云数据
- **适用**: 需要点云数据的导航/建图方案（如 RTAB-Map）

#### `robot_pose_publisher_ros2` 包
- **类型**: C++
- **功能**: 发布机器人位姿（通过 TF 获取并发布为 PoseStamped）

#### `yahboomcar_app_save_map` 包
- **功能**: 提供 Web 保存地图的服务端/客户端节点
- **依赖**: `yahboomcar_web_savmap_interfaces`

---

## 3. ROS2 话题/服务/Action 汇总

### 核心话题 (Topics)

| 话题名 | 类型 | 发布者 | 订阅者 |
|--------|------|--------|--------|
| `/cmd_vel` | Twist | 控制/导航节点 | 底盘驱动 |
| `/imu/data_raw` | Imu | 底盘驱动 | 里程计/导航 |
| `/imu/mag` | MagneticField | 底盘驱动 | 导航 |
| `/voltage` | Float32 | 底盘驱动 | 监控 |
| `/vel_raw` | Twist | 底盘驱动 | 里程计 |
| `/joint_states` | JointState | 底盘驱动 | 可视化 |
| `/scan` | LaserScan | 激光雷达驱动 | 避障/导航 |
| `/joy` | Joy | 手柄驱动 | 手柄控制节点 |
| `/Buzzer` | Bool | 控制节点 | 底盘驱动 |
| `/RGBLight` | Int32 | 控制节点 | 底盘驱动 |
| `/Servo` | ServoControl | 控制/动作节点 | 底盘驱动 |
| `/asr` | String | ASR节点 | 大模型服务 |
| `/wakeup` | Bool | ASR节点 | 动作服务器 |
| `/actionstatus` | String | 动作服务器 | 大模型服务 |
| `/action_service` | Rot.action | 大模型服务→动作服务器 | - |
| `/sign_detect` | DetectionObject | YOLO检测 | 自动驾驶 |
| `/llm_event` | LlmAutoDrive | 大模型 | 自动驾驶 |
| `/video_frames` | Image | 视频发布 | 自动驾驶/YOLO |

### 关键 Services

| 服务名 | 类型 | 用途 |
|--------|------|------|
| `/audio_service` | Audio.srv | 播放音频 |
| `/audio2_service` | Audio2.srv | TTS 文字转语音 |
| `/LargeScaleModel` | LargeScaleModel.srv | 大模型推理 |
| `/Qwen25` | Qwen25.srv | 通义千问推理 |
| `/Vision` | Vision.srv | 视觉服务 |
| `/web_save_map` | WebSaveMap.srv | 保存地图 |

### 关键 Actions

| Action 名 | 类型 | 用途 |
|-----------|------|------|
| `action_service` | Rot.action | 大模型动作执行 |
| `navigate_to_pose` | NavigateToPose | 导航到目标点 |

---

## 4. 二次开发指南

### 4.1 如何快速上手

#### 第一步：了解项目结构
```bash
cd ~/yahboomcar_ros2_ws/src
ls -d */  # 查看所有包
```

#### 第二步：选择车型并启动底盘
```bash
# A1 阿克曼车型
ros2 launch yahboomcar_bringup yahboomcar_bringup_A1_launch.py

# M1 麦克纳姆轮车型
ros2 launch yahboomcar_bringup yahboomcar_bringup_M1_launch.py
```

#### 第三步：测试基本功能
```bash
# 键盘控制
ros2 run yahboomcar_ctrl yahboom_keyboard

# 导航建图
ros2 launch yahboomcar_nav cartographer_bringup_launch.py

# AI 大模型交互（需要配置API Key）
ros2 launch largemodel largemodel_control.launch.py
```

### 4.2 如何添加新功能

#### 添加新的底盘动作
1. 编辑 `action_service_usb.py`，在 `CustomActionServer` 类中添加新方法
2. 在 `feedback_largemoel_dict` 中添加对应的反馈文本（中/英文）
3. 在 `execute_callback` 的方法派发逻辑中注册新方法

#### 添加新的交通标志识别
1. 修改 `yolo_detect.py` 中的类别列表（coco_names）
2. 在 `auto_drive.py` 的 `handle_events()` 中添加新的处理分支
3. 重新训练 YOLOv11 模型，增加新类别

#### 添加新的大模型平台
1. 在 `model_interface.py` 中添加新平台的 API 调用方法
2. 在 `large_model_interface.yaml` 中配置新平台的 API Key
3. 在 `model_service.py` 的选择逻辑中增加新的平台分支

### 4.3 配置说明

#### 关键配置文件

| 文件路径 | 用途 |
|----------|------|
| `largemodel/config/large_model_interface.yaml` | 大模型 API Key 和模型选择 |
| `largemodel/config/map_mapping.yaml` | 导航点地图坐标映射 |
| `largemodel/config/yahboom.yaml` | 大模型服务参数（语言、地区等） |
| `auto_drive/config/yolo_param.yaml` | YOLO 模型路径和推理参数 |
| `auto_drive/config/midline_points.yaml` | 车道中线的标定点 |
| `yahboomcar_nav/params/*.yaml` | 导航规划器参数 |

#### 环境变量

| 变量名 | 可选值 | 说明 |
|--------|--------|------|
| `CARTYPE` | A1 / M1 | 选择车型 |
| `CAMERA_TYPE` | nuwa / usb | 选择摄像头类型 |
| `INIT_SERVO_S1` | 0-180 | 舵机1初始角度 |
| `INIT_SERVO_S2` | 0-110 | 舵机2初始角度 |
| `RPLIDAR_TYPE` | tmini / 其他 | 激光雷达类型 |
| `LIDAR_TYPE` | 激光雷达型号 | 雷达型号选择 |

### 4.4 常见开发问题

#### Q: 如何测试单个节点？
```bash
# 直接运行节点的 main 函数
ros2 run <包名> <节点名>
```

#### Q: 如何查看话题数据？
```bash
ros2 topic list                    # 列出所有话题
ros2 topic echo /topic_name        # 查看话题数据
ros2 topic hz /topic_name          # 查看话题频率
```

#### Q: 如何调试？
- 大多数 Python 节点支持 `--ros-args -p DEBUG_MODE:=True` 参数
- 自动驾驶模式支持录制调试视频
- 所有节点都有 `self.get_logger()` 日志输出

### 4.5 代码风格与规范

- **Python 包**: 使用 `setup.py` 和 `ament_python` 构建
- **C++ 包**: 使用 `CMakeLists.txt` 和 `ament_cmake` 构建
- **节点命名**: `create_publisher/subscription/service/action` 方式
- **话题命名**: 小写+下划线（如 `cmd_vel`, `joint_states`）
- **参数声明**: 使用 `declare_parameter()` + `get_parameter()` 方式

---

## 附录：包间依赖关系图

```
                      ┌──────────────────────┐
                      │    largemodel (AI)    │
                      └──┬──┬──┬──┬──┬──┬───┘
                         │  │  │  │  │  │
        ┌────────────────┘  │  │  │  │  └──────────────┐
        ▼                   ▼  ▼  ▼  ▼                 ▼
  ┌─────────┐  ┌────────┐  ┌──────────┐  ┌──────────────┐
  │auto_drive│  │nav/nav2│  │voice_ctrl│  │text_chat     │
  └────┬────┘  └───┬────┘  └────┬─────┘  └──────────────┘
       │           │             │
       ▼           ▼             ▼
  ┌───────────────────────────────────────────┐
  │           yahboomcar_laser                │
  │           yahboomcar_astra                │
  │           yahboomcar_depth                │
  │           yahboomcar_vision               │
  │           yahboomcar_mediapipe            │
  └───────────────────┬───────────────────────┘
                      │
                      ▼
  ┌───────────────────────────────────────────┐
  │           yahboomcar_ctrl                 │
  │           yahboomcar_bringup              │
  │           yahboomcar_base_node            │
  └───────────────────┬───────────────────────┘
                      │
                      ▼
              ┌───────────────┐
              │   物理底盘硬件  │
              │  (Rosmaster_Lib)│
              └───────────────┘
```

---

## 附录 B：与 `yahboom_ws` 工程的区别对比

> 对比目标工程: `~/yahboom_ws/`（另一个独立的 ROS2 工作区，位于 `/home/sunrise/yahboom_ws`，侧重于纯 AI 大模型交互）

### 整体定位对比

| 维度 | yahboomcar_ros2_ws (本工程) | yahboom_ws |
|------|---------------------------|-----------|
| **核心定位** | 智能小车底盘控制 + 传感器 + 导航 | AI 大模型交互 + 语音智能体 |
| **面向场景** | ROS2 智能小车（A1/M1/R2 车型） | Jetson 边缘 AI 机器人 |
| **包数量** | 26 个 ROS2 包 | 4 个 ROS2 包 |
| **编程语言** | Python + C++ | Python 全栈 |
| **主要硬件** | 小车底盘 + 激光雷达 + 深度摄像头 + 舵机 | Jetson 系列 + 麦克风 + 摄像头 |
| **AI 集成深度** | 浅（仅 largemodel 包，通过 ActionServer 方式） | 深（LLM Agent + 工具链 + MCP Server） |

### 模块详细对比

| 功能模块 | yahboomcar_ros2_ws (26包) | yahboom_ws (4包) |
|---------|---------------------------|-----------------|
| **消息接口** | `interfaces` + `auto_interfaces` + `yahboomcar_msgs` + `yahboomcar_web_savmap_interfaces` | 仅 `interfaces` (4 个 Service + 1 个 Action) |
| **摄像头** | `yahboomcar_astra` (ASTRA 深度相机, 24 个节点) + `yahboomcar_vision` | `camera` (USB + CSI + AR 增强现实) |
| **大模型 AI** | 基础 ASR + 大模型 + ActionService (30+种底盘动作) | 深度集成: AI Agent + ToolChain + MCP Server |
| **文本聊天** | `text_chat` (命令行交互) | `text_chat` (基本一致) |
| **语音控制** | `yahboomcar_voice_ctrl` + `yahboomcar_voice_ctrl_depth` (独立的语音控制包) | 通过 largemodel 集成 ASR+TTS |
| **底盘驱动** | ✅ `yahboomcar_bringup` (Ackman/Mcnamu 驱动) | ❌ 无 |
| **里程计** | ✅ `yahboomcar_base_node` (C++, 编码器+IMU 融合) | ❌ 无 |
| **控制输入** | ✅ `yahboomcar_ctrl` (键盘/手柄控制) | ❌ 无 |
| **机器人描述** | ✅ `yahboomcar_description` (URDF + 3D 模型) | ❌ 无 |
| **激光雷达** | ✅ `yahboomcar_laser` (避障 + 跟踪) | ❌ 无 |
| **SLAM/导航** | ✅ `yahboomcar_nav` (Cartographer/RTAB-Map/DWA/TEB) | ❌ 无 |
| **多机器人** | ✅ `yahboomcar_multi` (编队控制) | ❌ 无 |
| **深度相机** | ✅ `yahboomcar_depth` (深度 + 视觉跟随) | ❌ 无 |
| **MediaPipe** | ✅ `yahboomcar_mediapipe` (人体检测/手势识别) | ❌ 无 |
| **自动驾驶** | ✅ `auto_drive` (YOLOv11 道路标志+车道线) | ❌ 无 |

### 共有的包

两个工程共享以下 3 个包的功能：

| 包名 | 本工程版本 | yahboom_ws 版本差异 |
|------|-----------|-------------------|
| **interfaces** | 基础服务定义 | **完全相同**（消息定义完全一致） |
| **largemodel** | 旧版 (ActionServer + 双模型推理 + 30+种底盘动作) | **大幅重构**：引入 AI Agent、ToolChain、MCP Server、Tool Adapters 模式 |
| **text_chat** | 标准版 | 基本一致（终端交互体验略有优化） |

### largemodel 包的核心差异

| 特性 | 本工程 (旧版) | yahboom_ws (新版) |
|------|-------------|------------------|
| **动作执行机制** | ✅ `action_service_usb.py` / `action_service_nuwa.py` (1205 行核心逻辑) | ❌ 已移除，改为 Tool Adapters |
| **工具执行模式** | ❌ 直接在 Prompt 中定义动作函数 | ✅ ToolChainManager + Tool Adapters |
| **AI Agent** | ❌ 无 | ✅ AI Agent (规划→执行→反馈循环) |
| **MCP 协议** | ❌ 无 | ✅ MCP Server (JSON-RPC 标准) |
| **工具类型** | 30+ 种底盘/舵机/视觉动作 (navigation, seewhat, servo, tracker, follow 等) | 6 种 AI 工具 (seewhat, analyze_video, generate_image, scan_table, visual_positioning, write_document) |
| **硬件动作** | ✅ 支持舵机、漂移、跳舞、RGB 灯、蜂鸣器等 | ❌ 无硬件控制 |
| **对话持久化** | ❌ 无 | ✅ JSON 文件持久化多轮对话 |
| **System Prompt** | ❌ 固定 Prompt 模板 | ✅ 根据注册的工具列表动态生成 |

### 依赖关系对比

**本工程 (yahboomcar_ros2_ws) 依赖链**:
```
largemodel → ActionServer → 底盘驱动/舵机/激光雷达/视觉追踪
         ↓
    yahboomcar_ctrl → cmd_vel → yahboomcar_bringup → 物理底盘
         ↓
    yahboomcar_nav → 激光雷达/SLAM → 导航规划
```

**yahboom_ws 依赖链**:
```
text_chat_node ← asr 话题 → model_service ← image_raw ← camera
                                ↓
                     large_model_interface → Ollama/通义/星火/百度/OpenRouter
                                ↓
                     ToolsManager → ToolChainManager → Tool Adapters
                                ↓
                     AI Agent → 复杂任务规划
```

---

## 附录 C：RDK X5 + M1 麦克纳姆轮小车适配指南

### 针对您的硬件配置的工程选择建议

根据您的硬件配置（**RDK X5 开发板 + M1 麦克纳姆轮小车**），以下为具体建议：

### 本工程对 RDK X5 的适配程度

```
对本工程的 RDK X5 适配程度:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ M1 麦克纳姆轮驱动 ────── 原生支持 (Mcnamu_driver_M1.py)
✅ M1 里程计结算 ────────── 原生支持 (base_node_M1.cpp)
✅ M1 键盘/手柄控制 ─────── 原生支持 (yahboom_keyboard / yahboom_joy_M1)
✅ M1 URDF 模型 ────────── 原生支持 (meshes/M1Mecanum/)
✅ M1 激光雷达避障/跟踪 ──── 原生支持 (laser_Avoidance_M1 / laser_Tracker_M1)
✅ M1 SLAM 导航 ────────── 原生支持 (Cartographer / DWA / TEB)
✅ YOLO 推理 (RDK 硬件加速) ─ 原生支持 (hobot_dnn)
✅ 大模型 AI 交互 ───────── 支持 (Ollama / 通义 / 星火 / 百度等)
❌ ASTRA 深度相机 ──────── 需要适配 (RDK 需要不同驱动)
```

### RDK X5 适配修改清单

从 Jetson 平台迁移到 RDK X5 时，需关注以下修改点：

| 修改项 | 原代码 (Jetson) | 修改为 (RDK X5) | 涉及文件 |
|--------|----------------|----------------|---------|
| 摄像头初始化 | `jetcam.csi_camera` / ASTRA 驱动 | RDK 的 hobot 摄像头话题 `/hbmem_img` 或 USB 摄像头 | `model_service.py` 的 `init_camera()` |
| 摄像头驱动 | `camera` 包 / `ascamera` 驱动 | 移除或改用 `hobot_cam` 包 | `largemodel_control.launch.py` |
| YOLO 推理加速 | 通用 torch/ultralytics | 使用 `hobot_dnn` 硬件加速 | `auto_drive/utils/yolov11_infer.py` |
| 串口麦克风 | `/dev/myspeech` | 根据实际设备节点设置 | `yahboom.yaml` 的 `mic_serial_port` |
| 路径硬编码 | `/home/jetson/` | 改为 `$(env HOME)` 或 `/home/sunrise/` | 各配置文件中的硬编码路径 |
| TTS 离线模型 | Piper ONNX 指定路径 | 检查模型是否存在，不存在则下载 | `large_model_interface.yaml` |
| 编译架构 | aarch64 (Jetson) | aarch64 (RDK X5 同样适用) | **无需修改** |

### 获取新版 AI 能力（可选）

如果您希望获得 `yahboom_ws` 中新版 `largemodel` 的 **AI Agent + 工具链** 能力，可以将其替换到本工程：

```bash
# 1. 备份旧版 largemodel
cd ~/yahboomcar_ros2_ws/src
mv largemodel largemodel_old

# 2. 复制新版 largemodel (从 yahboom_ws)
cp -r ~/yahboom_ws/src/largemodel .

# 3. 适配摄像头
#    新版 model_service.py 使用了 /image_raw 话题订阅图像
#    需要修改 init_camera() 方法，将摄像头订阅话题改为 RDK 的 /hbmem_img 或 /usb_cam/image_raw

# 4. 重新编译（只需编译接口和 AI 相关包）
cd ~/yahboomcar_ros2_ws
colcon build --packages-select interfaces largemodel text_chat
```

### M1 小车快速启动流程

```bash
# 0. 先配置环境变量
export CARTYPE=M1
export CAMERA_TYPE=usb   # 根据实际摄像头选择: usb / nuwa

# 1. 启动底盘驱动
ros2 launch yahboomcar_bringup yahboomcar_bringup_M1_launch.py

# 2. 键盘控制（测试底盘是否正常）
ros2 run yahboomcar_ctrl yahboom_keyboard

# 3. 启动激光雷达 SLAM 建图
ros2 launch yahboomcar_nav cartographer_bringup_launch.py

# 4. 保存建好的地图
ros2 launch yahboomcar_nav save_map_launch.py

# 5. 启动自主导航
ros2 launch yahboomcar_nav navigation_dwa_launch.py

# 6. 启动大模型 AI 语音交互（需要先配置 API Key）
ros2 launch largemodel largemodel_control.launch.py