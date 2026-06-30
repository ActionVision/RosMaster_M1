# ros2 run VS ros2 launch — 区别与使用场景详解

> **一句话**：`run` 是**启动一个程序**，`launch` 是**一键启动一堆程序**。

---

## 一、类比理解

想象你要开一家奶茶店：

| | `ros2 run` | `ros2 launch` |
|------|------------|---------------|
| **比喻** | 你去叫一个人上班 | 你按一下开关，全店自动开门 |
| **做的事** | 启动 **1 个节点** | 启动 **N 个节点 + 加载参数** |
| **操作** | `ros2 run 切水果 张三` | `ros2 launch 奶茶店 开店流程.py` |
| **还需要** | 每个员工你得自己去叫 | launch 文件里写好了要叫谁、给什么参数 |

---

## 二、技术区别

| 对比维度 | `ros2 run` | `ros2 launch` |
|---------|------------|---------------|
| **启动数量** | 1 个节点 | 可以启动 **N 个节点** |
| **参数设置** | 需要手动 `--ros-args -p` | 在 launch 文件里一次性配好 |
| **节点命名** | 默认用节点类名 | 可以自定义名字，防止重名 |
| **依赖管理** | 不管依赖，先启动再说 | 可以控制启动顺序、条件 |
| **你需要做的事** | 开多个终端，一个个敲命令 | **一条命令全部搞定** |

---

## 三、在小车项目里的实际例子

### 例子1：用 `ros2 run` 启动键盘控制

```bash
# 开一个终端，启动底盘驱动
ros2 run yahboomcar_bringup Mcnamu_driver_M1

# 再开一个终端，启动键盘控制
ros2 run yahboomcar_ctrl yahboom_keyboard

# 如果要看 RViz 可视化，还得再开一个
rviz2
```

**用 `run` 的问题**：你每次要把所有需要的节点**手动挨个敲一遍**，开 N 个终端，还容易忘。

### 例子2：用 `ros2 launch` 一键启动

```bash
# 一条命令搞定：手柄驱动 + 手柄控制 + 底盘驱动
ros2 launch yahboomcar_ctrl yahboomcar_joy_launch.py
```

这个 launch 文件里写了什么？打开看看：
```python
# yahboomcar_joy_launch.py 的内容
def generate_launch_description():
    node1 = Node(package='joy', executable='joy_node')           # 手柄驱动
    joy_node = Node(package='yahboomcar_ctrl', executable='yahboom_joy_M1')  # 控制逻辑
    bringup_node = Node(package='yahboomcar_bringup', executable='Mcnamu_driver_M1')  # 底盘驱动
    return LaunchDescription([node1, joy_node, bringup_node])
```

**3个 `ros2 run` 命令，变成了 1 个 `ros2 launch`！**

---

## 四、什么时候用 `run`？

### ✅ 用 `run` 的场景

**1. 测试/调试单个节点**
```bash
# 只想看激光避障行不行
ros2 run yahboomcar_laser laser_Avoidance_M1
```

**2. 已经启动了底盘，临时加一个功能**
```bash
# 底盘开着，想临时加个摄像头跟随
ros2 run yahboomcar_astra faceFollow
```

**3. 学习阶段，想一个一个看懂每个节点**
```bash
# 先启动底盘，看看它发什么话题
ros2 run yahboomcar_bringup Mcnamu_driver_M1
# 再启动键盘，看看 /cmd_vel 怎么传到驱动
ros2 run yahboomcar_ctrl yahboom_keyboard
```

**4. 查问题，排除故障**
```bash
# 小车不动，单独测试底盘驱动
ros2 run yahboomcar_bringup Mcnamu_driver_M1
# 再在另一个终端手动发速度指令
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.1}}" --once
```

### ❌ 什么时候别用 `run`
- 需要同时启动很多依赖节点时 → 用 launch
- 需要传一堆参数时 → 用 launch（手敲 `--ros-args -p` 容易出错）
- 每次都要重复做的事 → 写个 launch 一键搞定

---

## 五、什么时候用 `launch`？

### ✅ 用 `launch` 的场景

**1. 日常使用（最常用）**
```bash
# 早上来，一键启动小车
ros2 launch yahboomcar_bringup yahboomcar_bringup_M1_launch.py
# 这一条命令启动了：底盘驱动+里程计+IMU滤波+EKF+手柄+模型显示+RViz
```

**2. 需要配一堆参数**
```python
# launch 文件里可以一次性配好所有参数
Node(
    package='yahboomcar_bringup',
    executable='Mcnamu_driver_M1',
    parameters=[{
        'xlinear_limit': 1.0,
        'angular_limit': 5.0,
        'car_type': 'M1',
    }]
)
```

**3. 建图/导航（依赖一大堆东西）**
```bash
ros2 launch yahboomcar_nav cartographer_bringup_launch.py
# 自动启动：Cartographer + 激光雷达驱动 + TF变换 + 地图服务器...
```

### ❌ 什么时候别用 `launch`
- 只想快速测试一个功能 → 用 run 更快
- 想单独看某个节点的输出 → 用 run，输出不会被淹没

---

## 六、两者的对应关系

```
ros2 run <包名> <节点名>
        ↓
launch 文件里的:  Node(package='包名', executable='节点名')

ros2 run <包名> <节点名> --ros-args -p 参数:=值
        ↓
launch 文件里的:  Node(package='包名', executable='节点名', parameters=[{'参数': 值}])
```

### 你用过的命令对照

| `ros2 run`（单节点） | `ros2 launch`（一键启动） | 启动了哪些节点 |
|---|---|---|
| `ros2 run yahboomcar_ctrl yahboom_keyboard` | — | 只有键盘控制 |
| `ros2 run yahboomcar_laser laser_Avoidance_M1` | — | 只有激光避障 |
| `ros2 run yahboomcar_astra faceFollow` | — | 只有摄像头跟随 |
| — | `ros2 launch yahboomcar_ctrl yahboomcar_joy_launch.py` | joy_node + yahboom_joy_M1 + Mcnamu_driver_M1 |
| — | `ros2 launch yahboomcar_bringup yahboomcar_bringup_M1_launch.py` | 10+个节点（驱动+里程计+IMU+EKF+手柄+模型+RViz...） |
| — | `ros2 launch yahboomcar_nav cartographer_bringup_launch.py` | Cartographer + 激光 + TF + 地图... |

---

## 七、怎么知道 launch 文件启动了哪些东西？

### 方法1：直接看 launch 文件
```bash
cat yahboomcar_ws/src/yahboomcar_ctrl/launch/yahboomcar_joy_launch.py
```

看到 `Node(...)` 就说明启动了一个节点。

### 方法2：启动后再查
```bash
# 启动 launch
ros2 launch yahboomcar_ctrl yahboomcar_joy_launch.py &

# 查看现在有哪些节点在跑
ros2 node list
# 输出:
# /joy_node
# /joy_ctrl
# /driver_node
```

---

## 八、进阶：自己写一个 launch 文件

假设你每天早上要手动敲：
```bash
# 终端1
ros2 run yahboomcar_bringup Mcnamu_driver_M1
# 终端2
ros2 run yahboomcar_laser laser_Avoidance_M1
```

很烦对吧？写个 launch 文件 `my_start.launch.py`：

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    # 底盘驱动
    driver = Node(
        package='yahboomcar_bringup',
        executable='Mcnamu_driver_M1',
    )
    # 激光避障
    avoid = Node(
        package='yahboomcar_laser',
        executable='laser_Avoidance_M1',
        parameters=[{'linear': 0.2, 'ResponseDist': 0.5}]  # 顺便调参！
    )
    return LaunchDescription([driver, avoid])
```

然后你每天只需要：
```bash
ros2 launch my_package my_start.launch.py
# 一个命令 = 原来的两个终端 + 手动调参
```

---

## 九、总结

| 你想... | 用这个 | 命令示例 |
|--------|--------|---------|
| 测试/调试单个功能 | `ros2 run` | `ros2 run yahboomcar_laser laser_Avoidance_M1` |
| 学 ROS2，逐个理解节点 | `ros2 run` | `ros2 run yahboomcar_ctrl yahboom_keyboard` |
| 日常正常使用 | `ros2 launch` | `ros2 launch yahboomcar_bringup yahboomcar_bringup_M1_launch.py` |
| 建图/导航 | `ros2 launch` | `ros2 launc
h yahboomcar_nav cartographer_bringup_launch.py` |
| 一键启动大模型 | `ros2 launch` | `ros2 launch largemodel largemodel_control.launch.py` |

**记忆口诀**：调试学习用 `run`，日常使用用 `launch`。`launch` 内部其实就是在调一堆 `run`。