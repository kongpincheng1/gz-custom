

## 1. 仓库定位与用途

本仓库不是 PX4 主仓库（`PX4-Autopilot`），而是仿真资源子仓库，主要提供：

- Gazebo 世界文件（`worlds/*.sdf`）
- 飞行器与传感器模型（`models/*`）

典型使用方式是：

1. 用这个仓库提供模型与场景；
2. 在另一个终端运行 PX4 SITL；
3. 让 PX4 在这个世界里完成定位、任务飞行等实验。

---

## 2. 相对官方仓库的关键修改

### 2.1 新增了三种圆柱障碍物模型

新增模型目录：

- `models/cylinder_small`
- `models/cylinder_middle`
- `models/cylinder_large`

每个模型都采用 STL 网格，且在 `model.sdf` 内统一按 `0.001` 缩放导入（通常表示 mm→m 量纲转换）。

### 2.2 修改了默认世界 `worlds/default.sdf`

主要变化：

- 将地理坐标调整到国内某操场附近；
- 在世界中新增两组目标区域（`target_area` / `target_area2`）；
- 每组区域内布置了多个圆柱障碍；
- 每组区域附带黄色矩形边界线（可视化任务区域）。
- 只需要改 `target_area` / `target_area2` 的父 `pose`，就能整体平移一组目标物体。

### 2.3 调整了 OakD-Lite 相机传感器参数

在 `models/OakD-Lite/model.sdf` 中：

- 相机与深度相机 pose 改为朝向侧向（含 `pitch=1.570796`）并向前平移；
- RGB 相机分辨率从 `1920x1080` 调整为 `640x480`（降低仿真负载、提升实时性）。

### 2.4 修改了 x500 基础机体的风场响应

在 `models/x500_base/model.sdf` 中给 `base_link` 和各 `rotor_*` 增加了 `<enable_wind>true</enable_wind>`，即该机体链路可受风场系统影响（若世界里启用了风场插件）。

---

## 3. 目录结构说明

```text
.
├── models/                  # 所有仿真模型（飞行器、传感器、障碍物）
│   ├── x500_base/           # x500 的基础机体定义
│   ├── OakD-Lite/           # OakD-Lite 传感器模型
│   ├── cylinder_small/      # 新增：小号圆柱障碍物
│   ├── cylinder_middle/     # 新增：中号圆柱障碍物
│   └── cylinder_large/      # 新增：大号圆柱障碍物
├── worlds/
│   └── default.sdf          # 课程实验默认世界（已修改）
└── README.md
```

---



## 4. 与 PX4 SITL 联动


### 终端 ：启动 PX4

在 PX4-Autopilot 仓库里执行（示例）：

```bash
make px4_sitl gz_x500
```

如果你的课程工程使用别的模型名，请按实际目标替换（例如 `gz_x500_depth`、`gz_x500_vision` 等）。

---

## 5. default.sdf 场景设计说明


### 5.1 世界基础配置

- 物理引擎：ODE
- 仿真步长：`0.004`
- 更新率：`250Hz`
- 包含常用系统插件：Physics / Imu / AirPressure / NavSat / Sensors 等

### 5.2 地理坐标

`<spherical_coordinates>` 里的经纬度被设置为国内某学校操场附近坐标（已在文件中标注中文注释）。

### 5.3 目标区域 1：`target_area`

包含：

- 1 个大圆柱
- 1 个中圆柱
- 1 个小圆柱
- 1 个黄色边框（由 4 条 box 线段组成）

如果你想**整体移动这组障碍**，只改：

```xml
<model name='target_area'>
  <pose>...</pose>
</model>
```

### 5.4 目标区域 2：`target_area2`

包含：

- 5 个中圆柱（不同相对位姿）
- 1 个黄色边框

同样只需改 `target_area2` 的父 `pose` 就能整体平移。

---

## 6. 新增圆柱模型说明

三种圆柱模型结构一致：

- `model.config`：模型元信息
- `model.sdf`：碰撞体 + 可视化网格
- `meshes/*.stl`：实际几何文件

在 `model.sdf` 中：

- 碰撞几何与可视化几何都使用同一 STL；
- 统一缩放 `0.001`；
- 设为 `static=true`（障碍物不参与动力学运动）。

---

