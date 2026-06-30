# ROS2 小白实战指南 — Yahboom 智能小车快速上手

> **适合人群**: 有 Python 基础、懂一点 YOLO 算法、但完全不懂 ROS2 的同学  
> **目标**: 把小车站起来跑一圈，顺便搞懂 ROS2 到底是个啥  
> **硬前提**: 你的电脑已经装好了 ROS2 Humble，小车能通电连接

---

## 第0章：一句话说清 ROS2 是个啥

**ROS2 不是操作系统，它是一个"软件通信架子"**。

想象一下你在公司上班：

| 现实中 | ROS2 里 |
|--------|---------|
| 你负责写代码 | 你是一个 **节点 (Node)** |
| 你在微信群里发消息 | 你在 **发布 (Publish)** 一个 **话题 (Topic)** |
| 别人在群里看到消息回复你 | 别人 **订阅 (Subscribe)** 了你的话题 |
| 你在群里问"谁有空帮我拿一下" | 你发起了一个 **服务 (Service)** |
| 你安排小李做事，他做完了告诉你 | 你发起了一个 **动作 (Action)** |

**所以 ROS2 = 微信群 + 私聊 + 任务分配系统**，让小车上的各个程序能互相说话。

`rclpy` 是 ROS2 的 Python 版 SDK，装上之后写 Python 就能用 ROS2。

---

## 第1章：小车的"器官"对应哪些 ROS2 概念

先搞懂你的小车上有什么硬件：

```
┌──────────────────────────────────────────────┐
│              你的电脑 (上位机)                  │
│  运行: 导航、AI大模型、自动驾驶等高阶功能       │
├──────────────────────────────────────────────┤
│              小车主板 (下位机)                  │
│  运行: 底盘驱动、传感器采集等底层功能           │
├──────────────────────────────────────────────┤
│   ┌───┐  ┌──────┐  ┌────┐  ┌───┐  ┌───┐     │
│   │电机│  │激光雷达│  │摄像头│  │舵机│  │IMU│     │
│   └───┘  └──────┘  └────┘  └───┘  └───┘     │
└──────────────────────────────────────────────┘
```

### 硬件 vs ROS2 模块对应表

| 硬件 | 对应的 ROS2 包 | 干的事 |
|------|---------------|--------|
| 电机+轮子 | `yahboomcar_bringup` | 收命令(`cmd_vel`)，让电机转 |
| 激光雷达 | `yahboomcar_laser` | 发 `LaserScan` 数据，告诉"前面有墙" |
| 摄像头 | `yahboomcar_astra` / `yahboomcar_vision` | 拍视频，发 `Image` 数据 |
| 舵机（云台） | 通过 `ServoControl` 消息控制 | 让摄像头上下左右转 |
| 喇叭+彩灯 | 通过 `Buzzer` / `RGBLight` 控制 | 滴滴叫、闪灯 |
| 全车大脑 | `largemodel` (大模型) | 听懂人话，指挥全车 |

---

## 第2章：ROS2 核心概念 — 用小车例子秒懂

### 2.1 节点 (Node) = 一个干活的程序

```python
# 这就是一个节点——小车的底盘驱动程序
ros2 run yahboomcar_bringup Ackman_driver_A1.py
# 这个节点只干两件事：
#   1. 听 cmd_vel 话题（别人叫它走它就开电机）
#   2. 发 IMU 数据（告诉别人自己歪了没）
```

**类比**: 节点就像公司里的每个员工——司机负责开车，导航员负责看地图，摄像头专员负责盯屏幕。

### 2.2 话题 (Topic) = 广播喊话

```
发布者  ----发消息到---->  [话题名]  ----被收到---->  订阅者

键盘节点 -----cmd_vel---->  🎯  -----被收到---->  底盘驱动
(你在键盘按"w")      (速度指令)              (电机开始转)
```

**关键命令**：
```bash
# 看所有话题（就像看公司群里有哪些群）
ros2 topic list

# 监听某个话题的内容（就像盯住某个群聊）
ros2 topic echo /cmd_vel

# 手动发一条消息测试（机器人版"AT全体成员"）
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.1}}" --once
```

### 2.3 服务 (Service) = 打电话问一下

```
请求方 --------打电话问-------->  服务端
         "帮我查下距离是多少？"
         <-------回答--------
         "距离是 2.5 米"
```

**类比**: 话题是群里广播，服务是私聊打电话。

### 2.4 动作 (Action) = 派任务并跟踪进度

```
派任务方 ----> 动作服务器
"去厨房拿杯水"
        然后：
        ← "正在去厨房的路上..." (反馈)
        ← "到了厨房，开始找杯子..." (反馈)
        ← "任务完成，拿到了水" (结果)
```

**在小车上的实际例子**：
```
大模型 ----动作指令----> 动作执行器
"navigation(厨房)"  
                  然后：
                  ← "正在导航到厨房"
                  ← "导航完成"
                  ← 舵机转到0度"seewhat()拍照"
                  ← 拍照完成，分析图片
```

### 2.5 Launch 文件 = 一键启动脚本

```bash
# 不用一个个 ros2 run 敲命令
# launch 文件帮你一次性启动所有需要的节点

ros2 launch yahboomcar_bringup yahboomcar_bringup_A1_launch.py
# 这一条命令 = 
#   启动了底盘驱动节点
#   启动了里程计节点
#   加载了参数配置
```

---

## 第3章：跟着我做 — 让小车动起来（5步走）

### 🚀 第一步：检查环境（5分钟）

```bash
# 1. 打开终端，检查 ROS2 环境
echo $ROS_DISTRO
# 应该显示: humble

# 2. 进入工作区
cd ~/yahboomcar_ros2_ws/yahboomcar_ws

# 3. 编译整个项目（第一次需要）
colcon build

# 4. 加载环境变量（每次新开终端都要执行）
source install/setup.bash
```

> 💡 **小贴士**: 把 `source install/setup.bash` 加到 `~/.bashrc` 里，以后就不用每次手动敲了
> ```bash
> echo "source ~/yahboomcar_ros2_ws/yahboomcar_ws/install/setup.bash" >> ~/.bashrc
> source ~/.bashrc
> ```

### 🚀 第二步：启动底盘（让小车"活过来"）

```bash
# A1 车型（阿克曼转向，像汽车一样）
ros2 launch yahboomcar_bringup yahboomcar_bringup_A1_launch.py

# M1 车型（麦克纳姆轮，可以横着走）
# ros2 launch yahboomcar_bringup yahboomcar_bringup_M1_launch.py
```

**能看到的效果**：
- 终端开始刷刷刷打印信息
- 小车（如果连上了）会初始化舵机

**这时发生了什么？**
```
底盘驱动节点启动了 → 它开始监听 /cmd_vel 话题
                  → 它开始发布 /imu/data_raw 传感器数据
                  → 它开始发布 /voltage 电池电压
```

### 🚀 第三步：键盘控制（让小车动起来）

**新开一个终端**（原来的不要关）：
```bash
ros2 run yahboomcar_ctrl yahboom_keyboard
```

你会看到：

```
Control Your SLAM-Bot!
---------------------------
Moving around:
   u    i    o
   j    k    l
   m    ,    .

q/z : 加速/减速
space : 急停
CTRL-C : 退出
```

**操作**：
- 按 `i` → 向前走
- 按 `,` → 向后走
- 按 `j` / `l` → 左转/右转
- 按 `space` → 急停

**背后原理**（这就是 ROS2 的精髓！）：
```
你按 "i" 键
    ↓
键盘节点发布: /cmd_vel 话题，内容是 "{linear: {x: 0.2}}"
    ↓
底盘驱动节点收到这条消息（因为它订阅了 /cmd_vel）
    ↓
底盘驱动调用 Rosmaster_Lib.set_car_motion(0.2, 0, 0)
    ↓
电机开始转 → 小车往前跑
```

**确认消息在传递**：
```bash
# 新开第三个终端
ros2 topic echo /cmd_vel
# 然后去键盘终端按 i/j/k/l
# 你会看到这里实时显示速度指令
```

### 🚀 第四步：启动激光雷达避障（不用自己开也能跑）

关掉键盘（按 CTRL-C），试试自动避障：

```bash
# A1 避障
ros2 run yahboomcar_laser laser_Avoidance_A1

# 小车会自己往前走，遇到墙自动转弯！
```

**原理**（超级简单）：
```
激光雷达扫描 → 发现前方有墙 → 计算左右有没有空间 → 
  左有空？左转 | 右有空？右转 | 都有？随机选
```

### 🚀 第五步：建图 + 导航（让小车认路）

#### 5.1 建地图（SLAM）

```bash
# 先启动激光雷达（如果没启动的话）
# 用 launch 一键启动 Cartographer 建图
ros2 launch yahboomcar_nav cartographer_bringup_launch.py
```

**建图操作**：
1. 新开终端，运行键盘控制
2. 慢慢推着小车走遍整个房间
3. 你会看到一个地图在 RViz 里逐渐展开

**建完存地图**：
```bash
ros2 launch yahboomcar_nav save_map_launch.py
# 地图会保存到 yahboomcar_nav/maps/ 目录下
```

#### 5.2 导航（让小车自己去指定的地方）

```bash
# 加载已建好的地图并启动导航
ros2 launch yahboomcar_nav navigation_dwa_launch.py
```

在 RViz 里：
1. 点击 "2D Pose Estimate" 告诉小车"你在这里"
2. 点击 "2D Nav Goal" 告诉小车"你去这里"
3. 小车会自动规划路径并开过去！

---

## 第4章：ROS2 常用命令速查表

### 日常调试命令

| 命令 | 作用 | 你的使用场景 |
|------|------|-------------|
| `ros2 topic list` | 查看所有话题 | 想知道小车现在在发什么数据 |
| `ros2 topic echo /cmd_vel` | 看某个话题的内容 | 检查键盘控制有没有发出速度指令 |
| `ros2 topic hz /scan` | 看话题发布频率 | 检查激光雷达是不是正常（一般10Hz） |
| `ros2 node list` | 查看所有运行中的节点 | 想知道有哪些程序在跑 |
| `ros2 node info /driver_node` | 看某个节点的详情 | 想知道这个节点在发什么、收什么 |
| `ros2 run 包名 节点名` | 运行一个节点 | 启动一个具体功能 |
| `ros2 launch 包名 launch文件` | 批量启动节点 | 一键启动一组功能 |
| `ros2 param list` | 查看节点参数 | 想知道能调哪些参数 |
| `ros2 param get /节点名 参数名` | 获取参数值 | 查当前速度限制是多少 |
| `ros2 param set /节点名 参数名 值` | 动态改参数 | 在线调小车的速度 |

### 实用调试案例

**场景1：小车不动了，排查问题**
```bash
# 1. 确认底盘驱动在跑
ros2 node list | grep driver

# 2. 确认有速度指令发出来
ros2 topic echo /cmd_vel
# 如果没数据 → 键盘节点没启动或没按

# 3. 手动发个速度指令测试（小车应该动）
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.1}}" --once

# 4. 急停
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{}" --once
```

**场景2：摄像头不出图**
```bash
# 列出所有话题，看看有没有 /image 相关
ros2 topic list | grep image

# 看话题信息
ros2 topic info /image_raw

# 用 rqt 看视频
rqt_image_view /image_raw
```

---

## 第5章：常用 Python 代码模板（拿来就用）

### 模板1：写一个 ROS2 节点（Hello World）

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import String  # 字符串消息类型

class MyFirstNode(Node):
    def __init__(self):
        super().__init__('my_first_node')  # 节点名称
        # 创建一个发布者：发 String 类型消息到 /my_topic
        self.pub = self.create_publisher(String, '/my_topic', 10)
        # 创建一个定时器：每秒执行一次 timer_callback
        self.timer = self.create_timer(1.0, self.timer_callback)
        # 创建一个订阅者：收 /cmd_vel 消息
        self.sub = self.create_subscription(
            String, '/cmd_vel', self.callback, 10)
        
    def timer_callback(self):
        msg = String()
        msg.data = 'hello world'
        self.pub.publish(msg)
        
    def callback(self, msg):
        self.get_logger().info(f'收到消息: {msg.data}')

def main():
    rclpy.init()
    node = MyFirstNode()
    rclpy.spin(node)  # 保持节点运行，直到 CTRL-C

if __name__ == '__main__':
    main()
```

### 模板2：订阅激光雷达数据

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
import numpy as np

class LaserListener(Node):
    def __init__(self):
        super().__init__('laser_listener')
        # 订阅激光雷达话题
        self.sub = self.create_subscription(
            LaserScan, '/scan', self.scan_callback, 1)
        
    def scan_callback(self, msg):
        # msg.ranges 是一个数组，存着360°每个角度的距离
        ranges = np.array(msg.ranges)
        
        # 检查正前方（0°附近）有没有障碍物
        front = ranges[0:10]  # 前10个点
        min_dist = np.min(front)
        
        if min_dist < 0.5:  # 小于0.5米
            self.get_logger().warn(f'⚠️ 前面有墙！距离: {min_dist:.2f}米')
        else:
            self.get_logger().info(f'✅ 前方畅通，最近: {min_dist:.2f}米')

def main():
    rclpy.init()
    node = LaserListener()
    rclpy.spin(node)

if __name__ == '__main__':
    main()
```

### 模板3：发布速度指令控制小车

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist
import time

class CarController(Node):
    def __init__(self):
        super().__init__('car_controller')
        # 发布到 /cmd_vel 就能控制小车
        self.pub = self.create_publisher(Twist, '/cmd_vel', 10)
        
    def go_forward(self, speed=0.2, duration=2.0):
        twist = Twist()
        twist.linear.x = speed   # 前进速度
        twist.angular.z = 0.0    # 不转弯
        self.pub.publish(twist)
        time.sleep(duration)
        self.stop()
        
    def turn_left(self, speed=0.5, duration=1.0):
        twist = Twist()
        twist.linear.x = 0.0     # 不前进
        twist.angular.z = speed  # 左转速度
        self.pub.publish(twist)
        time.sleep(duration)
        self.stop()
        
    def stop(self):
        self.pub.publish(Twist())  # 发空消息 = 停止

def main():
    rclpy.init()
    car = CarController()
    
    # 编一个简单的动作序列
    car.go_forward(0.2, 2.0)   # 前进2秒
    car.turn_left(0.5, 1.0)    # 左转1秒
    car.go_forward(0.2, 2.0)   # 再前进2秒
    
    car.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### 模板4：发布/订阅自定义消息（舵机控制）

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from yahboomcar_msgs.msg import ServoControl  # 自定义消息

class ServoController(Node):
    def __init__(self):
        super().__init__('servo_controller')
        self.pub = self.create_publisher(ServoControl, '/Servo', 10)
        
    def move_servo(self, s1_angle, s2_angle):
        msg = ServoControl()
        msg.s1 = s1_angle  # 水平舵机 0-180°
        msg.s2 = s2_angle  # 垂直舵机 0-110°
        self.pub.publish(msg)
        
def main():
    rclpy.init()
    servo = ServoController()
    servo.move_servo(90, 45)  # 舵机转到中间位置
    servo.destroy_node()
    rclpy.shutdown()
```

---

## 第6章：我想改 YOLO 检测 — 从哪下手？

你要是对 YOLO 熟悉，这个项目里 YOLO 用在了两个地方：

### 6.1 道路标志检测（自动驾驶）

**入口文件**: `yahboomcar_ws/src/auto_drive/auto_drive/yolo_detect.py`

```python
# 这个节点做三件事：
# 1. 订阅摄像头图像（Image 话题）
# 2. 用 YOLOv11 推理检测交通标志
# 3. 发布检测结果到 /sign_detect 话题
```

**可识别的 14 类交通标志**（在 `utils/yolov11_infer.py` 中定义）：
```python
coco_names = [
    "greenlight", "honking", "parkA", "parkB", "redlight", 
    "school", "sidewalk", "sidewalk_ground", "speedlimit", 
    "stop", "straight", "turnright", "turnright_ground", "yellowlight"
]
```

**如果你想自己训练一个新模型**：
1. 收集你的图片数据
2. 用 YOLOv11 训练，导出为 ONNX
3. 修改 `auto_drive/config/yolo_param.yaml` 里的模型路径
4. 改 `coco_names` 匹配你的类别

### 6.2 车道线检测（车道保持）

**入口文件**: `auto_drive/auto_drive/utils/yolov11_infer.py` 中的 `Lane_Detect` 类

这个模型只检测 1 个类别（车道线），然后用霍夫变换判断是左线还是右线。

### 6.3 如何在 ROS2 里用你自己的 YOLO

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import cv2
from ultralytics import YOLO  # 你的 YOLO 库

class MyYOLONode(Node):
    def __init__(self):
        super().__init__('my_yolo_node')
        self.bridge = CvBridge()
        # 订阅摄像头话题
        self.sub = self.create_subscription(
            Image, '/image_raw', self.image_callback, 1)
        # 加载你的 YOLO 模型
        self.model = YOLO('my_model.pt')
        
    def image_callback(self, msg):
        # ROS图像 → OpenCV图像
        cv_image = self.bridge.imgmsg_to_cv2(msg, 'bgr8')
        
        # 用你的 YOLO 推理
        results = self.model(cv_image)
        
        # 在这里处理结果...
        # 比如画框、检测目标、控制小车...
        
        # 显示结果
        annotated = results[0].plot()
        cv2.imshow('My YOLO', annotated)
        cv2.waitKey(1)

def main():
    rclpy.init()
    node = MyYOLONode()
    rclpy.spin(node)
```

---

## 第7章：常见问题（小白必看）

### Q1: "我改了代码，怎么让它生效？"
```bash
# 每次改完代码都要重新编译
cd ~/yahboomcar_ros2_ws/yahboomcar_ws
colcon build --packages-select 你改的那个包名
# 或者全部编译
colcon build
```

### Q2: "命令找不到 'command not found'"
```bash
# 八成是忘了 source 环境变量
source ~/yahboomcar_ros2_ws/yahboomcar_ws/install/setup.bash
```

### Q3: "端口拒绝 'port denied'"
```bash
# 小车通过 USB 串口连接电脑，需要权限
sudo chmod 777 /dev/ttyUSB0  # 看你的端口号
# 或者把用户加到 dialout 组
sudo usermod -a -G dialout $USER
```

### Q4: "小车不动，但键盘显示在发指令"
```bash
# 方法1：手动发指令测试
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.2}}" --once

# 方法2：检查话题连接
ros2 topic info /cmd_vel
# 看 Subscriber count 是不是 > 0

# 方法3：重启底盘驱动
# CTRL-C 关掉原来的，重新 ros2 launch
```

### Q5: "多个终端来回切换好麻烦"
```bash
# 推荐使用 tmux 终端复用器
sudo apt install tmux
tmux new -s robot  # 创建一个新会话

# Ctrl+B 然后 % → 左右分屏
# Ctrl+B 然后 " → 上下分屏
# Ctrl+B 然后方向键 → 切换面板
# 这样你就能在一个窗口里同时看多个终端了
```

### Q6: "我想看小车在 3D 界面里长什么样"
```bash
# 启动 rviz
rviz2

# 或者在 launch 时加上 rviz
ros2 launch yahboomcar_description display_A1.launch.py
```

### Q7: "怎样把 ROS2 数据录下来复盘？"
```bash
# 录制
ros2 bag record /cmd_vel /scan /imu/data_raw

# 回放
ros2 bag play <bag文件名>
```

---

## 附录：从小白到入门的学习路线

```
第1周：把上面的"5步走"全跑通一遍
       → 能用键盘控制小车
       → 能看话题数据
       
第2周：搞懂每个 launch 文件启动了哪些节点
       → 打开 launch 文件看看
       → 自己手动 ros2 run 启动每个节点
       
第3周：看懂一个 Python 节点的全部代码
       → 从 yahboom_keyboard.py 开始（最短最简单）
       → 然后看 laser_Avoidance_A1.py
       
第4周：自己改一个节点 + 写一个简单节点
       → 改激光避障的距离参数
       → 写一个"遇到墙就后退"的节点
       
第5周：把 YOLO 集成到 ROS2 里
       → 参考 6.3 节的模板
       → 让你的 YOLO 模型控制小车
       
第6周+：探索大模型和自动驾驶
       → 读懂 action_service_usb.py
       → 配置大模型 API Key
       → 理解 auto_drive.py 的流程
```

---

## 结语

ROS2 说穿了就几件事：
1. **节点** = 各个程序
2. **话题** = 程序之间广播数据
3. **服务** = 程序之间一问一答
4. **动作** = 程序之间派任务+反馈进度
5. **Launch** = 一键启动多个程序

你只要记住 `cmd_vel` 能让小车动、`/scan` 是激光雷达数据、写节点用 `rclpy`，就能开始玩了。遇到问题就 `ros2 topic echo` 看看数据流到哪断了。

**动手是学 ROS2 最好的方式，祝玩得开心！** 🚀