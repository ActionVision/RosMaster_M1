# yahboomcar_ws 文件结构与分类速查手册

> **工作区根目录**: `~/yahboomcar_ros2_ws/yahboomcar_ws`
> **内容**: 本手册列出了 `yahboomcar_ws` 下**所有非编译生成的文件**，标注每个文件的类型、作用、运行方式

---

## 一、文件类型速查（3秒看懂）

| 文件后缀 | 是什么 | 你能干嘛 |
|---------|--------|---------|
| `.py` | **Python 源文件** — 小车的"大脑" | ✅ 这是你要改的代码！ |
| `.cpp` | **C++ 源文件** — 性能要求高的部分 | ⚠️ 有C++基础再改 |
| `.launch.py` | **启动文件** — 一键启动一组节点 | ✅ 直接运行，不用改 |
| `.msg` / `.srv` / `.action` | **消息定义** — 自定义通信格式 | ⚠️ 加新功能时才需要动 |
| `.yaml` / `.cfg` | **配置文件** — 参数、模型路径 | ✅ 经常要改（调参） |
| `.urdf.xacro` | **机器人模型** — 描述小车长啥样 | ⚠️ 换硬件才需要动 |
| `.STL` | **3D 网格文件** — 可视化用的模型 | ❌ 不用改 |
| `.bin` / `.pb` / `.pt` | **模型文件** — AI 训练好的权重 | ✅ 换成你自己的模型 |
| `.mp3` / `.wav` | **音效文件** — 喇叭发声用 | ✅ 可以替换 |
| `.rviz` | **RViz 配置** — 3D可视化布局 | ⚠️ 想调显示效果再改 |
| `.xml` | **参数/配置** — 导航规划参数 | ⚠️ 调导航时改 |
| `.lua` | **Cartographer 配置** — 建图参数 | ⚠️ 调建图时改 |
| `.ttf` | **字体文件** — 图像上写字用 | ❌ 不用改 |
| `package.xml` | **包描述文件** — 每个包的身份ID | ⚠️ 添加依赖时才改 |
| `setup.py` / `CMakeLists.txt` | **构建文件** — 编译和安装配置 | ⚠️ 加新节点时才需要改 |
| `resource/` | **资源标记文件** — ROS2 识别包用 | ❌ 不用管 |

### 🔴 你要改的（重点标记）
- `**/*.py` — Python 代码
- `**/*.yaml` — 配置文件
- `**/*.bin` / `**/*.pb` — AI 模型文件（换你自己的）

### 🟢 你直接运行的（启动命令）
- `**/*.launch.py` — 一键启动
- `ros2 run <包名> <节点名>` — 单独启动某个节点

### ⚫ 完全不用管的
- `build/`, `install/`, `log/` — 编译自动生成
- `__pycache__/`, `*.pyc`, `*.so` — Python/C++ 编译缓存
- `.STL` — 3D模型，只在 RViz 显示时用到
- `resource/` — ROS2 标准标记文件

---

## 二、每个包的完整文件清单

### 2.1 `auto_drive` — 自动驾驶（YOLO 车道线+交通标志）

#### 源文件（你要改的）

| 文件路径 | 节点名 | 功能 | 启动命令 |
|---------|--------|------|---------|
| `auto_drive/auto_drive.py` | `AutoDrive` | **自动驾驶主程序**：车道保持+交通标志响应+车辆控制 | `ros2 run auto_drive auto_drive` |
| `auto_drive/yolo_detect.py` | `YOLODetection` | YOLO 检测交通标志，发布到 `/sign_detect` | `ros2 run auto_drive yolo_detect` |
| `auto_drive/pub_video.py` | - | 发布视频帧 | `ros2 run auto_drive pub_video` |
| `auto_drive/record_video.py` | - | 录制视频 | `ros2 run auto_drive record_video` |
| `auto_drive/cali_midline.py` | - | **车道中线标定**（调车时用） | `ros2 run auto_drive cali_midline` |
| `auto_drive/example_route.py` | - | 示例路径生成 | `ros2 run auto_drive example_route` |
| `auto_drive/utils/yolov11_infer.py` | - | **YOLOv11 推理引擎**（Sign_Detect+Lane_Detect两个类） | 被其他文件 import，不单独运行 |
| `auto_drive/utils/common_tools.py` | - | PID 控制器、工具函数 | 被 import |
| `tools/conver_model.py` | - | 模型转换工具 | `python3 conver_model.py` |

#### 配置文件

| 文件 | 用途 | 你要改吗？ |
|------|------|-----------|
| `config/yolo_param.yaml` | YOLO 模型路径、推理参数（conf/iou阈值） | ✅ **经常改** |
| `config/midline_points.yaml` | 车道中线标定点坐标 | ✅ **标定时要改** |
| `config/auto_drive_node.yaml` | 自动驾驶节点参数 | ⚠️ 调参时改 |
| `config/left_points.yaml` | 左车道线标定点 | ⚠️ 标定 |
| `config/right_points.yaml` | 右车道线标定点 | ⚠️ 标定 |
| `config/requirements.txt` | Python 依赖列表 | ✅ **装依赖用** |
| `config/*.bin` | **YOLO 模型文件**（RDK硬件加速格式） | ✅ **换你的模型** |

#### 音效文件

| 文件 | 用途 |
|------|------|
| `system_vioce/honk.mp3` | 鸣笛声 |
| `system_vioce/unlock.mp3` | 解锁声 |
| `system_vioce/fire_alarm.mp3` | 警报声 |

#### 构建/标记/测试文件（不用管）

| 文件 | 说明 |
|------|------|
| `package.xml` | 包描述（依赖声明） |
| `setup.py`, `setup.cfg` | Python 包安装配置 |
| `resource/auto_drive` | ROS2 资源标记 |
| `test/test_copyright.py` | 版权检查测试 |
| `test/test_flake8.py` | Python 代码风格检查 |
| `test/test_pep257.py` | Python 文档字符串检查 |

#### Launch 文件

| 文件 | 用途 | 启动命令 |
|------|------|---------|
| `launch/auto_drive.launch.py` | 启动自动驾驶主程序 | `ros2 launch auto_drive auto_drive.launch.py` |
| `launch/auto_drive_bringup.launch.py` | 启动自动驾驶全套（YOLO+主程序） | `ros2 launch auto_drive auto_drive_bringup.launch.py` |

---

### 2.2 `auto_interfaces` — 自动驾驶消息定义

| 文件 | 类型 | 内容 |
|------|------|------|
| `action/Rot.action` | Action | LLM 动作执行（actions[] + llm_response → success） |
| `msg/DetectionObject.msg` | 消息 | YOLO 检测结果（class_name, confidence, bbox） |
| `msg/LlmAutoDrive.msg` | 消息 | LLM 自动驾驶指令（instruct, value） |
| `msg/LlmRequest.msg` | 消息 | LLM 请求（robot_feedback, llm_request） |
| `msg/RoadRoute.msg` | 消息 | 道路导航路径（start_pose, goal_pose, cancel_nav） |
| `srv/WebSaveMap.srv` | 服务 | 网页保存地图（mapname → response） |
| `CMakeLists.txt`, `package.xml` | 构建 | 接口定义的编译配置 |

---

### 2.3 `interfaces` — 大模型接口定义

| 文件 | 类型 | 内容 |
|------|------|------|
| `action/Rot.action` | Action | 动作执行（与 auto_interfaces 相同） |
| `srv/Audio.srv` | 服务 | 播放音频 |
| `srv/Audio2.srv` | 服务 | TTS 文字转语音 |
| `srv/LargeScaleModel.srv` | 服务 | 大模型推理（promtrequest → response + action_list） |
| `srv/Qwen25.srv` | 服务 | 通义千问2.5推理 |
| `srv/Vision.srv` | 服务 | 视觉识别 |

---

### 2.4 `largemodel` — AI 大模型（语音交互+任务规划）

#### 源文件

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `largemodel/model_service.py` | **大模型服务主节点**：接收ASR/Seewhat，双模型推理 | `ros2 run largemodel model_service` |
| `largemodel/asr.py` | **语音识别节点**：关键词唤醒+VAD录音+ASR转文字 | `ros2 run largemodel asr` |
| `largemodel/action_service_usb.py` | **USB摄像头版动作执行器**（1205行，核心！） | `ros2 run largemodel action_service_usb` |
| `largemodel/action_service_nuwa.py` | 女娲摄像头版动作执行器 | `ros2 run largemodel action_service_nuwa` |
| `largemodel/autonav.py` | 自主导航节点 | `ros2 run largemodel autonav` |
| `utils/large_model_interface.py` | **大模型接口层**（通义千问/Dify/讯飞/百度API） | 被 import |
| `utils/dify_client2.py` | Dify 平台 API 客户端 | 被 import |
| `utils/mic_serial.py` | 麦克风串口通信（KWS唤醒词监听） | 被 import |
| `utils/promot_nuwa.py` | 女娲版提示词模板 | 被 import |
| `utils/promot_usb.py` | USB版提示词模板 | 被 import |

#### 配置文件（这些你经常要改！）

| 文件 | 用途 | 你要改吗？ |
|------|------|-----------|
| `config/large_model_interface.yaml` | **大模型API Key、模型选择** | ✅ **必须配！** |
| `config/map_mapping.yaml` | **导航点地图坐标**（"厨房"→坐标） | ✅ **必须配！** |
| `config/yahboom.yaml` | 大模型服务参数（语言、地区等） | ✅ **必须配！** |

#### 音效文件

| 文件 | 用途 |
|------|------|
| `resources_file/system_vioce/zh/longwan-women-1.mp3` | 中文唤醒应答音 |
| `resources_file/system_vioce/zh/longwan-women-2.mp3` | 中文错误提示音 |
| `resources_file/system_vioce/en/longxiaochun-women-1.mp3` | 英文唤醒应答音 |
| `resources_file/system_vioce/en/longxiaochun-women-2.mp3` | 英文错误提示音 |

#### Launch 文件

| 文件 | 用途 | 启动命令 |
|------|------|---------|
| `launch/largemodel_control.launch.py` | 启动大模型全套服务 | `ros2 launch largemodel largemodel_control.launch.py` |

---

### 2.5 `yahboomcar_bringup` — 底盘驱动（最重要的包！）

#### 源文件

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `Ackman_driver_A1.py` | **A1车型底盘驱动**：收cmd_vel，发IMU/电池/关节状态 | `ros2 run yahboomcar_bringup Ackman_driver_A1` |
| `Mcnamu_driver_M1.py` | **M1车型底盘驱动**：同上+RGB灯 | `ros2 run yahboomcar_bringup Mcnamu_driver_M1` |
| `Ackman_driver_A1_descri.py` | A1描述版驱动（关节状态发布不同） | `ros2 run yahboomcar_bringup Ackman_driver_A1_descri` |
| `Mcnamu_driver_M1_descri.py` | M1描述版驱动 | `ros2 run yahboomcar_bringup Mcnamu_driver_M1_descri` |
| `calibrate_angular_A1.py` | A1角速度校准 | `ros2 run yahboomcar_bringup calibrate_angular_A1` |
| `calibrate_angular_M1.py` | M1角速度校准 | `ros2 run yahboomcar_bringup calibrate_angular_M1` |
| `calibrate_linear_A1.py` | A1线速度校准 | `ros2 run yahboomcar_bringup calibrate_linear_A1` |
| `calibrate_linear_M1.py` | M1线速度校准 | `ros2 run yahboomcar_bringup calibrate_linear_M1` |
| `patrol_A1.py` | A1巡逻模式 | `ros2 run yahboomcar_bringup patrol_A1` |
| `patrol_M1.py` | M1巡逻模式 | `ros2 run yahboomcar_bringup patrol_M1` |
| `transform_utils.py` | 坐标变换工具（四元数↔欧拉角） | 被 import |

#### 参数

| 文件 | 用途 |
|------|------|
| `param/imu_filter_param.yaml` | IMU 滤波参数 |

#### Launch 文件（你经常用的）

| 文件 | 用途 | 启动命令 |
|------|------|---------|
| `launch/yahboomcar_bringup_A1_launch.py` | **A1全套启动**（驱动+里程计+IMU+手柄+模型显示+导航） | `ros2 launch yahboomcar_bringup yahboomcar_bringup_A1_launch.py` |
| `launch/yahboomcar_bringup_M1_launch.py` | **M1全套启动** | `ros2 launch yahboomcar_bringup yahboomcar_bringup_M1_launch.py` |
| `launch/bringup_A1_no_odom_launch.py` | A1驱动（无里程计） | `ros2 launch yahboomcar_bringup bringup_A1_no_odom_launch.py` |
| `launch/bringup_M1_no_odom_launch.py` | M1驱动（无里程计） | `ros2 launch yahboomcar_bringup bringup_M1_no_odom_launch.py` |
| `launch/laser_bringup_launch.py` | 激光雷达启动 | `ros2 launch yahboomcar_bringup laser_bringup_launch.py` |

---

### 2.6 `yahboomcar_base_node` — 里程计（C++）

#### 源文件

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `src/base_node_A1.cpp` | A1里程计计算 | 被 launch 启动，不需要单独运行 |
| `src/base_node_M1.cpp` | M1里程计计算 | 同上 |
| `src/base_node_R2.cpp` | R2里程计计算 | 同上 |
| `src/talker.cpp` | 测试用 | 不用管 |

---

### 2.7 `yahboomcar_ctrl` — 控制输入

#### 源文件

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `yahboom_keyboard.py` | **键盘控制**：i/j/k/l 控制小车 | `ros2 run yahboomcar_ctrl yahboom_keyboard` |
| `yahboom_joy_A1.py` | **A1手柄控制** | `ros2 run yahboomcar_ctrl yahboom_joy_A1` |
| `yahboom_joy_M1.py` | **M1手柄控制** | `ros2 run yahboomcar_ctrl yahboom_joy_M1` |

#### Launch

| 文件 | 启动命令 |
|------|---------|
| `launch/yahboomcar_joy_launch.py` | `ros2 launch yahboomcar_ctrl yahboomcar_joy_launch.py`（启动手柄+驱动） |

---

### 2.8 `yahboomcar_laser` — 激光雷达

#### 源文件

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `laser_Avoidance_A1.py` | **A1激光避障** | `ros2 run yahboomcar_laser laser_Avoidance_A1` |
| `laser_Avoidance_M1.py` | **M1激光避障** | `ros2 run yahboomcar_laser laser_Avoidance_M1` |
| `laser_Tracker_A1.py` | A1激光跟随（跟物体走） | `ros2 run yahboomcar_laser laser_Tracker_A1` |
| `laser_Tracker_M1.py` | M1激光跟随 | `ros2 run yahboomcar_laser laser_Tracker_M1` |
| `laser_Warning_M1.py` | M1激光预警 | `ros2 run yahboomcar_laser laser_Warning_M1` |
| `common.py` | PID控制器 | 被 import |

---

### 2.9 `yahboomcar_nav` — 导航（SLAM建图+自主导航）

#### Launch 文件（最重要的部分）

| Launch 文件 | 用途 | 启动命令 |
|------------|------|---------|
| `launch/cartographer_bringup_launch.py` | **Cartographer建图（推荐）** | `ros2 launch yahboomcar_nav cartographer_bringup_launch.py` |
| `launch/cartographer_launch.py` | Cartographer 在线建图 | `ros2 launch yahboomcar_nav cartographer_launch.py` |
| `launch/cartographer_localization_launch.py` | Cartographer 纯定位（已有地图） | `ros2 launch yahboomcar_nav cartographer_localization_launch.py` |
| `launch/map_gmapping_launch.py` | Gmapping 建图 | `ros2 launch yahboomcar_nav map_gmapping_launch.py` |
| `launch/map_slam_toolbox_launch.py` | SLAM Toolbox 建图 | `ros2 launch yahboomcar_nav map_slam_toolbox_launch.py` |
| `launch/map_rtabmap_launch.py` | RTAB-Map 建图 | `ros2 launch yahboomcar_nav map_rtabmap_launch.py` |
| `launch/navigation_dwa_launch.py` | **DWA规划器导航（推荐）** | `ros2 launch yahboomcar_nav navigation_dwa_launch.py` |
| `launch/navigation_teb_launch.py` | TEB规划器导航 | `ros2 launch yahboomcar_nav navigation_teb_launch.py` |
| `launch/navigation_cartodwb_launch.py` | Carto+DWA组合导航 | `ros2 launch yahboomcar_nav navigation_cartodwb_launch.py` |
| `launch/navigation_rtabmap_launch.py` | RTAB-Map导航 | `ros2 launch yahboomcar_nav navigation_rtabmap_launch.py` |
| `launch/save_map_launch.py` | **保存地图** | `ros2 launch yahboomcar_nav save_map_launch.py` |
| `launch/laser_bringup_launch.py` | 激光雷达启动 | `ros2 launch yahboomcar_nav laser_bringup_launch.py` |
| `launch/display_map_launch.py` | 查看地图 | `ros2 launch yahboomcar_nav display_map_launch.py` |
| `launch/display_nav_launch.py` | 导航可视化 | `ros2 launch yahboomcar_nav display_nav_launch.py` |
| `launch/localization_imu_odom.launch.py` | IMU+里程计定位 | `ros2 launch yahboomcar_nav localization_imu_odom.launch.py` |

#### 参数文件（调导航时改这些）

| 文件 | 用途 |
|------|------|
| `params/dwa_nav_params.yaml` | DWA 规划器参数 |
| `params/teb_nav_params.yaml` | TEB 规划器参数 |
| `params/cartoteb_nav_params.yaml` | Carto+TEB 参数 |
| `params/cartodwa_nav_params.yaml` | Carto+DWA 参数 |
| `params/rtabmap_nav_params.yaml` | RTAB-Map 参数 |
| `params/mapper_params_online_sync.yaml` | SLAM Toolbox 参数 |
| `params/lds_2d.lua` | Cartographer Lua 配置 |

#### RViz 配置

| 文件 | 用途 |
|------|------|
| `rviz/nav.rviz` | 导航可视化布局 |
| `rviz/nav_multi.rviz` | 多机器人导航布局 |
| `rviz/map.rviz` | 地图查看布局 |
| `rviz/rtabmap_map.rviz` | RTAB-Map 地图布局 |
| `rviz/rtabmap_nav.rviz` | RTAB-Map 导航布局 |

#### 源代码

| 文件 | 功能 |
|------|------|
| `yahboomcar_nav/scan_filter.py` | 激光数据滤波 |

---

### 2.10 `yahboomcar_astra` — ASTRA 摄像头视觉

#### 追踪类节点（小车不动，摄像头跟着目标转）

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `colorTracker.py` | **颜色追踪** | `ros2 run yahboomcar_astra colorTracker` |
| `qrTracker.py` | 二维码追踪 | `ros2 run yahboomcar_astra qrTracker` |
| `faceTracker.py` | 人脸追踪 | `ros2 run yahboomcar_astra faceTracker` |
| `gestureTracker.py` | 手势追踪 | `ros2 run yahboomcar_astra gestureTracker` |
| `poseTracker.py` | 姿态追踪 | `ros2 run yahboomcar_astra poseTracker` |
| `apriltagTracker.py` | Apriltag标签追踪 | `ros2 run yahboomcar_astra apriltagTracker` |
| `monoTracker.py` | 单目标追踪（KCF） | `ros2 run yahboomcar_astra monoTracker` |

#### 跟随类节点（小车跟着目标走）

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `colorFollow.py` | 颜色跟随 | `ros2 run yahboomcar_astra colorFollow` |
| `faceFollow.py` | 人脸跟随 | `ros2 run yahboomcar_astra faceFollow` |
| `gestureFollow.py` | 手势跟随 | `ros2 run yahboomcar_astra gestureFollow` |
| `poseFollow.py` | 姿态跟随 | `ros2 run yahboomcar_astra poseFollow` |
| `qrFollow.py` | 二维码跟随 | `ros2 run yahboomcar_astra qrFollow` |
| `apriltagFollow.py` | Apriltag跟随 | `ros2 run yahboomcar_astra apriltagFollow` |
| `follow_line.py` | **巡线** | `ros2 run yahboomcar_astra follow_line` |

#### 其他

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `HandCtrl.py` | 手势控制 | `ros2 run yahboomcar_astra HandCtrl` |
| `qrCtrl.py` | 二维码控制 | `ros2 run yahboomcar_astra qrCtrl` |

#### 公共模块

| 文件 | 功能 |
|------|------|
| `common/astra_common.py` | ASTRA 相机通用函数 |
| `common/follow_common.py` | 跟随算法通用函数 |
| `common/track_common.py` | 追踪算法通用函数 |
| `common/media_common.py` | 媒体处理通用函数 |

#### 配置文件

| 文件 | 用途 |
|------|------|
| `config/Block_Simplified.TTF` | 字体文件 |
| `config/haarcascade_frontalface_default.xml` | OpenCV 人脸检测模型 |
| `cfg/monoTrackerPID.cfg` | 单目标追踪 PID 参数 |

#### OpenCV 学习脚本（独立 Python 脚本，不用 ROS2）

| 文件 | 内容 | 运行方式 |
|------|------|---------|
| `scripts/opencv/1_1.py` ~ `4_3.py` | 共18个OpenCV入门示例 | `python3 scripts/opencv/1_1.py` |
| `scripts/opencv/yahboom.jpg` | 测试图片 | 被 OpenCV 脚本读取 |

---

### 2.11 `yahboomcar_depth` — 深度相机

#### 基本功能

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `Basic/get_center_dis.py` | 获取图像中心点距离 | `ros2 run yahboomcar_depth get_center_dis` |
| `Basic/depth_to_color.py` | 深度图对齐彩色图 | `ros2 run yahboomcar_depth depth_to_color` |
| `Basic/calculate_volume.py` | 体积估算 | `ros2 run yahboomcar_depth calculate_volume` |

#### 高级功能

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `Advanced/colorTracker.py` | 深度辅助颜色追踪 | `ros2 run yahboomcar_depth depth_colorTracker` |
| `Advanced/faceFollow.py` | 深度辅助人脸跟随 | `ros2 run yahboomcar_depth depth_faceFollow` |
| `Advanced/apriltagFollow.py` | 深度+Apriltag跟随 | `ros2 run yahboomcar_depth depth_apriltagFollow` |
| `Advanced/qrFollow.py` | 深度+二维码跟随 | `ros2 run yahboomcar_depth depth_qrFollow` |
| `Advanced/follow_line.py` | 深度+巡线 | `ros2 run yahboomcar_depth depth_follow_line` |
| `Advanced/Edge_Detection.py` | 边缘检测 | `ros2 run yahboomcar_depth depth_Edge_Detection` |

#### MediaPipe + 深度

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `MediaPipe/gestureFollow.py` | 深度+手势跟随 | `ros2 run yahboomcar_depth depth_gestureFollow` |
| `MediaPipe/poseFollow.py` | 深度+姿态跟随 | `ros2 run yahboomcar_depth depth_poseFollow` |

#### KCF 追踪

| 文件 | 功能 |
|------|------|
| `kcf/KCF_Tracker.py` | KCF 目标追踪算法 |

#### 相机应用

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `camera_app.py` | 相机应用主程序 | `ros2 run yahboomcar_depth camera_app` |
| `launch/camera_app.launch.py` | 相机应用启动 | `ros2 launch yahboomcar_depth camera_app.launch.py` |

---

### 2.12 `yahboomcar_vision` — 通用视觉

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `create_qrcode.py` | 生成二维码 | `ros2 run yahboomcar_vision create_qrcode` |
| `parse_qrcode.py` | 解析二维码 | `ros2 run yahboomcar_vision parse_qrcode` |
| `detect_pose.py` | 人体姿态检测 | `ros2 run yahboomcar_vision detect_pose` |
| `detect_object.py` | 目标检测（SSD MobileNet） | `ros2 run yahboomcar_vision detect_object` |
| `simple_ar.py` | 简单AR增强现实 | `ros2 run yahboomcar_vision simple_ar` |
| `astra_rgb_image.py` | ASTRA 相机RGB采集 | `ros2 run yahboomcar_vision astra_rgb_image` |
| `astra_depth_image.py` | ASTRA 相机深度采集 | `ros2 run yahboomcar_vision astra_depth_image` |
| `pub_image.py` | 发布图像到话题 | `ros2 run yahboomcar_vision pub_image` |

#### 配置文件

| 文件 | 用途 |
|------|------|
| `config/frozen_inference_graph.pb` | SSD MobileNet 模型（TensorFlow） |
| `config/graph_opt.pb` | 优化后的模型 |
| `config/object_detection_coco.txt` | COCO 类别列表 |
| `config/ssd_mobilenet_v2_coco.txt` | 模型标签 |
| `config/camera.yaml` | 相机参数 |
| `config/Block_Simplified.TTF` | 字体 |
| `config/yahboom_logo.png` | 品牌Logo |
| `config/yahboom_logo_qr.jpg` | 品牌二维码 |

---

### 2.13 `yahboomcar_mediapipe` — Google MediaPipe 人体检测

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `01_HandDetector.py` | 手部关键点检测 | `ros2 run yahboomcar_mediapipe 01_HandDetector` |
| `02_PoseDetector.py` | 人体33个关键点 | `ros2 run yahboomcar_mediapipe 02_PoseDetector` |
| `03_Holistic.py` | 全身检测 | `ros2 run yahboomcar_mediapipe 03_Holistic` |
| `04_FaceMesh.py` | 面部网格 | `ros2 run yahboomcar_mediapipe 04_FaceMesh` |
| `05_FaceDetection.py` | 人脸检测 | `ros2 run yahboomcar_mediapipe 05_FaceDetection` |
| `06_FaceLandmarks.py` | 面部关键点 | `ros2 run yahboomcar_mediapipe 06_FaceLandmarks` |
| `07_Objectron.py` | 3D物体检测 | `ros2 run yahboomcar_mediapipe 07_Objectron` |
| `08_VirtualPaint.py` | 虚拟画笔 | `ros2 run yahboomcar_mediapipe 08_VirtualPaint` |
| `09_HandCtrl.py` | 手势控制 | `ros2 run yahboomcar_mediapipe 09_HandCtrl` |
| `10_GestureRecognition.py` | 手势识别 | `ros2 run yahboomcar_mediapipe 10_GestureRecognition` |
| `11_FindHand.py` | 找手 | `ros2 run yahboomcar_mediapipe 11_FindHand` |
| `12_FingerTrajectory.py` | 手指轨迹 | `ros2 run yahboomcar_mediapipe 12_FingerTrajectory` |

---

### 2.14 `yahboomcar_msgs` — 自定义消息

| 文件 | 内容 |
|------|------|
| `msg/ServoControl.msg` | 舵机控制（s1角度, s2角度） |
| `msg/Position.msg` | 位置（anglex, angley, distance） |
| `msg/PointArray.msg` | 点数组 |

> `lib/` 下的 `.so` 文件是编译生成的动态库，不用管。

---

### 2.15 `yahboomcar_multi` — 多机器人

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `yahboom_A1_joy.py` | A1多机器人手柄控制 | `ros2 run yahboomcar_multi yahboomcar_A1_ctrl` |
| `yahboom_M1_joy.py` | M1多机器人手柄控制 | `ros2 run yahboomcar_multi yahboomcar_M1_ctrl` |
| `get_follower_point.py` | 获取跟随点 | `ros2 run yahboomcar_multi get_follower_point` |
| `pub_follower_goal.py` | 发布跟随目标 | `ros2 run yahboomcar_multi pub_follower_goal` |
| `singlePID.py` | PID控制器 | 被 import |

#### Launch 文件

| 文件 | 用途 |
|------|------|
| `launch/A1_nav_robot1_launch.py` | A1机器人1号导航 |
| `launch/A1_nav_robot2_launch.py` | A1机器人2号导航 |
| `launch/M1_nav_robot1_launch.py` | M1机器人1号导航 |
| `launch/M1_nav_robot2_launch.py` | M1机器人2号导航 |
| `launch/bringup_launch.py` | 多机器人底盘启动 |
| `launch/navigation_launch.py` | 多机器人导航 |
| `launch/display_multi_nav_launch.py` | 多机器人导航可视化 |

---

### 2.16 `yahboomcar_voice_ctrl` — 语音控制（基础版）

与 `yahboomcar_astra` 功能完全相同（颜色追踪/跟随、人脸、手势等），多了一个 `colorSelect.py`（颜色选择）。
启动命令前面加 `ros2 run yahboomcar_voice_ctrl`。

### 2.17 `yahboomcar_voice_ctrl_depth` — 语音控制（深度版）

| 文件 | 功能 | 启动命令 |
|------|------|---------|
| `voice_get_dist.py` | 语音获取距离 | `ros2 run yahboomcar_voice_ctrl_depth voice_get_dist` |
| `colorSelect.py` | 颜色选择 | `ros2 run yahboomcar_voice_ctrl_depth colorSelect` |
| `follow/voice_face_follow.py` | 语音人脸跟随 | `ros2 run yahboomcar_voice_ctrl_depth voice_face_follow` |
| `follow/voice_colorTracker.py` | 语音颜色追踪 | `ros2 run yahboomcar_voice_ctrl_depth voice_colorTracker` |
| `follow/voice_qrFollow.py` | 语音二维码跟随 | `ros2 run yahboomcar_voice_ctrl_depth voice_qrFollow` |
| `follow/voice_apriltagFollow.py` | 语音Apriltag跟随 | `ros2 run yahboomcar_voice_ctrl_depth voice_apriltagFollow` |
| `follow/voice_follow_line.py` | 语音巡线 | `ros2 run yahboomcar_voice_ctrl_depth voice_follow_line` |
| `mediapipe/voice_gestureFollow.py` | 语音手势跟随 | `ros2 run yahboomcar_voice_ctrl_depth voice_gestureFollow` |
| `mediapipe/voice_poseFollow.py` | 语音姿态跟随 | `ros2 run yahboomcar_voice_ctrl_depth voice_poseFollow` |
| `kcf/voice_KCF_Tracker.py` | 语音KCF追踪 | `ros2 run yahboomcar_voice_ctrl_depth voice_KCF_Tracker` |

---

### 2.18 辅助包

| 包名 | 文件 | 功能 | 启动命令 |
|------|------|------|---------|
| `text_chat` | `text_chat.py` | 文字聊天界面 | `ros2 run text_chat text_chat` |
| `laserscan_to_point_publisher` | `laserscan_to_point_publish.py` | 激光→点云转换 | `ros2 run laserscan_to_point_publisher laserscan_to_point_publish` |
| `robot_pose_publisher_ros2` | `src/robot_pose_publisher.cpp` (C++) | 发布机器人位姿 | 被launch启动 |
| `yahboomcar_app_save_map` | `yahboom_app_save_map.py` | Web保存地图服务端 | `ros2 run yahboomcar_app_save_map server` |
| | `yahboom_app_save_map_client.py` | Web保存地图客户端 | `ros2 run yahboomcar_app_save_map client` |

---

## 三、快速定位：想做什么 → 找什么文件

### 想让小车动起来

| 想做的事 | 找这个文件 | 命令 |
|---------|-----------|------|
| 启动底盘 | `yahboomcar_bringup/launch/yahboomcar_bringup_M1_launch.py` | `ros2 launch yahboomcar_bringup yahboomcar_bringup_M1_launch.py` |
| 键盘控制 | `yahboomcar_ctrl/yahboom_keyboard.py` | `ros2 run yahboomcar_ctrl yahboom_keyboard` |
| 手柄控制 | `yahboomcar_ctrl/yahboom_joy_M1.py` | `ros2 launch yahboomcar_ctrl yahboomcar_joy_launch.py` |

### 想让小车自己避障

| 想做的事 | 找这个文件 | 命令 |
|---------|-----------|------|
| 激光避障 | `yahboomcar_laser/laser_Avoidance_M1.py` | `ros2 run yahboomcar_laser laser_Avoidance_M1` |
| 激光跟随 | `yahboomcar_laser/laser_Tracker_M1.py` | `ros2 run yahboomcar_laser laser_Tracker_M1` |

### 想让小车认路（建图+导航）

| 想做的事 | 找这个文件 | 命令 |
|---------|-----------|------|
| 建地图 | `yahboomcar_nav/launch/cartographer_bringup_launch.py` | `ros2 launch yahboomcar_nav cartographer_bringup_launch.py` |
| 导航 | `yahboomcar_nav/launch/navigation_dwa_launch.py` | `ros2 launch yahboomcar_nav navigation_dwa_launch.py` |
| 保存地图 | `yahboomcar_nav/launch/save_map_launch.py` | `ros2 launch yahboomcar_nav save_map_launch.py` |

### 想让小车识别人/物

| 想做的事 | 找这个文件 | 命令 |
|---------|-----------|------|
| 人脸跟随 | `yahboomcar_astra/faceFollow.py` | `ros2 run yahboomcar_astra faceFollow` |
| 颜色跟随 | `yahboomcar_astra/colorFollow.py` | `ros2 run yahboomcar_astra colorFollow` |
| 手势控制 | `yahboomcar_astra/gestureTracker.py` | `ros2 run yahboomcar_astra gestureTracker` |

### 想让小车听懂人话（大模型）

| 想做的事 | 找这个文件 | 命令 |
|---------|-----------|------|
| 全套AI启动 | `largemodel/launch/largemodel_control.launch.py` | `ros2 launch largemodel largemodel_control.launch.py` |
| 改API Key | `largemodel/config/large_model_interface.yaml` | 编辑它 |
| 改导航目标点 | `largemodel/config/map_mapping.yaml` | 编辑它 |

### 想用 YOLO

| 想做的事 | 找这个文件 |
|---------|-----------|
| YOLO检测交通标志 | `auto_drive/yolo_detect.py`（启动） |
| YOLO推理引擎 | `auto_drive/utils/yolov11_infer.py`（代码逻辑） |
| 换你自己的模型 | `auto_drive/config/yolo_param.yaml`（配置文件） |
| 换成你自己的YOLO | 参考 `auto_drive/config/*.bin` 模型文件路径 |

### 想调参数

| 调什么 | 找这个文件 |
|--------|-----------|
| 底盘速度限制 | `launch 文件里或 yahboom_joy_M1.py 中的 declare_parameter` |
| 避障距离 | `yahboomcar_laser/laser_Avoidance_M1.py` 里的 `ResponseDist` |
| 导航规划参数 | `yahboomcar_nav/params/dwa_nav_params.yaml` |
| PID参数 | `auto_drive/config/auto_drive_node.yaml` |
| 建图参数 | `yahboomcar_nav/params/mapper_params_online_sync.yaml` |

---

## 四、哪些是自动生成的（不用管）

| 目录/文件 | 怎么生成的 | 能不能删 |
|-----------|-----------|---------|
| `build/` | `colcon build` 编译产生 | ✅ 可删，重新编译就有 |
| `install/` | `colcon build` 安装产生 | ✅ 可删，重新编译就有 |
| `log/` | 运行时的日志 | ✅ 可删 |
| `**/__pycache__/` | Python 运行时生成 | ✅ 可删 |
| `**/*.pyc` | Python 编译缓存 | ✅ 可删 |
| `**/*.so` | C++ 编译的动态库 | ✅ 可删，重新编译就有 |
| `yahboomcar_msgs/lib/*.so` | 消息包编译生成 | ❌ 删了要重新 `colcon build` |
| `xgo.db` | `yahboomcar_app_save_map` 的数据库 | ❌ 存着地图数据 |

---

## 五、二次开发标准流程

### 场景1：改代码 → 测试

```bash
# 1. 编辑代码（比如改 laser_Avoidance_M1.py）
vim yahboomcar_ws/src/yahboomcar_laser/yahboomcar_laser/laser_Avoidance_M1.py

# 2. 重新编译（只编译改的那个包，速度更快）
cd ~/yahboomcar_ros2_ws/yahboomcar_ws
colcon build --packages-select yahboomcar_laser

# 3. 运行测试
ros2 run yahboomcar_laser laser_Avoidance_M1
```

### 场景2：加一个新节点

```bash
# 1. 在已有的包目录下建新文件
# 比如在 yahboomcar_laser 里加一个 "遇到墙就后退" 的节点
touch yahboomcar_ws/src/yahboomcar_laser/yahboomcar_laser/laser_backoff.py

# 2. 编辑 setup.py，在 entry_points 里注册新节点
# 加一行: 'laser_backoff = yahboomcar_laser.laser_backoff:main'

# 3. 编译
colcon build --packages-select yahboomcar_laser

# 4. 运行
ros2 run yahboomcar_laser laser_backoff
```

### 场景3：加一个新包

```bash
# 1. 在 src 下创建包
cd yahboomcar_ws/src
ros2 pkg create my_package --build-type ament_python --dependencies rclpy

# 2. 在里面写代码
# 3. 编译
cd ..
colcon build --packages-select my_package
```

---

## 六、文件统计

| 类别 | 文件数 |
|------|--------|
| Python 源文件 (`.py`) | 约 230 个 |
| C++ 源文件 (`.cpp`) | 4 个 |
| Launch 文件 (`.launch.py`) | 约 60 个 |
| 配置文件 (`.yaml`/`.cfg`) | 约 30 个 |
| 消息定义 (`.msg`/`.srv`/`.action`) | 约 15 个 |
| 3D 模型文件 (`.STL`) | 约 36 个 |
| URDF 模型 (`.urdf.xacro`) | 12 个 |
| AI 模型文件 (`.bin`/`.pb`) | 约 5 个 |
| 音效文件 (`.mp3`) | 约 7 个 |
| 测试文件 (`test/*.py`) | 约 40 个 |
| 构建文件 (`setup.py`/`CMakeLists.txt`/`package.xml`) | 约 80 个 |

> **二次开发重点关注的部分**：约 230 个 Python 文件 + 30 个 YAML 配置 + 5 个 AI 模型文件