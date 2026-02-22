# PX4 Gazebo 自定义仿真资源仓库（课程/实验版）

> 这个仓库基于官方 `PX4-gazebo-models` 克隆后进行二次开发，目标是为**PX4 + Gazebo 仿真教学/实验**提供一个“开箱即用、同学易复现”的资源包。  
> 你可以把它理解为：官方资源库 + 我们课程场景需要的模型、世界坐标、障碍物布局与传感器调整。

---

## 1. 仓库定位与用途

本仓库不是 PX4 主仓库（`PX4-Autopilot`），而是仿真资源子仓库，主要提供：

- Gazebo 世界文件（`worlds/*.sdf`）
- 飞行器与传感器模型（`models/*`）
- 一键启动脚本（`simulation-gazebo`）

典型使用方式是：

1. 用这个仓库提供模型与场景；
2. 在另一个终端运行 PX4 SITL；
3. 让 PX4 在这个世界里完成定位、避障、任务飞行等实验。

---

## 2. 相对官方仓库的关键修改（给同学先看）

为了让同学快速理解你改了什么，先给结论：

### 2.1 新增了三种圆柱障碍物模型

新增模型目录：

- `models/cylinder_small`
- `models/cylinder_middle`
- `models/cylinder_large`

每个模型都采用 STL 网格，且在 `model.sdf` 内统一按 `0.001` 缩放导入（通常表示 mm→m 量纲转换）。

### 2.2 修改了默认世界 `worlds/default.sdf`

主要变化：

- 将地理坐标调整到国内某操场附近（便于课程演示中的“本地化语义”）；
- 在世界中新增两组目标区域（`target_area` / `target_area2`）；
- 每组区域内布置了多个圆柱障碍；
- 每组区域附带黄色矩形边界线（可视化任务区域）。

并且你在注释中明确告诉使用者：只需要改 `target_area` / `target_area2` 的父 `pose`，就能整体平移一组目标物体。

### 2.3 调整了 OakD-Lite 相机传感器参数

在 `models/OakD-Lite/model.sdf` 中：

- 相机与深度相机 pose 改为朝向侧向（含 `pitch=1.570796`）并向前平移；
- RGB 相机分辨率从 `1920x1080` 调整为 `640x480`（降低仿真负载、提升实时性）。

### 2.4 修改了 x500 基础机体的风场响应

在 `models/x500_base/model.sdf` 中给 `base_link` 和各 `rotor_*` 增加了 `<enable_wind>true</enable_wind>`，即该机体链路可受风场系统影响（若世界里启用了风场插件）。

---

## 3. 目录结构说明（按“同学能看懂”组织）

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
├── simulation-gazebo        # 启动脚本（下载模型+启动仿真）
└── README.md
```

---

## 4. 环境要求

建议 Ubuntu 22.04 / 24.04（Linux）。

最少需要：

- Gazebo（`gz sim` 命令可用，建议 Harmonic / Garden 系列）
- Python 3（用于执行 `simulation-gazebo`）
- PX4-Autopilot（用于 SITL 飞控）

快速自检：

```bash
python3 --version
gz sim --versions
```

> 如果 `gz sim` 不可用，先安装 Gazebo 再继续。

---

## 5. 快速开始（推荐给同学的标准流程）

下面给两个常用流程：

## 5.1 方式 A：直接用本仓库本地资源启动 Gazebo（推荐）

在本仓库目录执行：

```bash
cd /path/to/gz-custom
GZ_SIM_RESOURCE_PATH=$PWD/models gz sim -r $PWD/worlds/default.sdf
```

这样 Gazebo 会直接加载你当前仓库内的模型与世界，不依赖远程下载。

## 5.2 方式 B：使用自带脚本 `simulation-gazebo`

```bash
cd /path/to/gz-custom
python3 simulation-gazebo --world default --overwrite
```

脚本行为说明：

- 默认从 GitHub 下载 zip 资源到 `~/.simulation-gazebo`；
- 解压后使用 `gz sim -r <world>.sdf` 启动；
- 可用 `--headless` 启动无 GUI 模式；
- 可用 `--dryrun` 只走准备流程不启动 Gazebo。

---

## 6. 与 PX4 SITL 联动（两终端）

> 注意：仅打开 Gazebo 不会自动起飞控；必须同时启动 PX4 SITL。

### 终端 1：启动 Gazebo 世界

```bash
cd /path/to/gz-custom
GZ_SIM_RESOURCE_PATH=$PWD/models gz sim -r $PWD/worlds/default.sdf
```

### 终端 2：启动 PX4

在 PX4-Autopilot 仓库里执行（示例）：

```bash
make px4_sitl gz_x500
```

如果你的课程工程使用别的模型名，请按实际目标替换（例如 `gz_x500_depth`、`gz_x500_vision` 等）。

---

## 7. default.sdf 场景设计说明（重点）

本节专门解释你改造后的默认世界，让同学知道“改哪里、会发生什么”。

### 7.1 世界基础配置

- 物理引擎：ODE
- 仿真步长：`0.004`
- 更新率：`250Hz`
- 包含常用系统插件：Physics / Imu / AirPressure / NavSat / Sensors 等

### 7.2 地理坐标

`<spherical_coordinates>` 里的经纬度被设置为国内某学校操场附近坐标（你已在文件中标注中文注释）。

### 7.3 目标区域 1：`target_area`

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

### 7.4 目标区域 2：`target_area2`

包含：

- 5 个中圆柱（不同相对位姿）
- 1 个黄色边框

同样只需改 `target_area2` 的父 `pose` 就能整体平移。

---

## 8. 新增圆柱模型说明

三种圆柱模型结构一致：

- `model.config`：模型元信息
- `model.sdf`：碰撞体 + 可视化网格
- `meshes/*.stl`：实际几何文件

在 `model.sdf` 中：

- 碰撞几何与可视化几何都使用同一 STL；
- 统一缩放 `0.001`；
- 设为 `static=true`（障碍物不参与动力学运动）。

---

## 9. 传感器与机体参数修改说明

## 9.1 OakD-Lite

你将相机姿态调整后，常见效果是：

- 画面主方向变化（不再是原始朝向）；
- 更容易满足课程中“侧视/前视对准障碍区域”的感知需求；
- 分辨率下降后渲染和通信压力减小。

## 9.2 x500_base 与风场

`enable_wind=true` 的意义：

- 机体和旋翼链路可以被 Gazebo 风场系统施加扰动；
- 如果世界启用风场插件，将更接近真实环境中的抗风控制任务。

> 你当前 `default.sdf` 已去掉风场插件，因此默认场景下不会出现明显风扰。若课程需要“有风实验”，可在 world 中重新加入 WindEffects 插件。

---

## 10. 常见实验任务建议（给同学参考）

你可以围绕两个区域快速组织实验：

1. **定位 + 视觉观察**：
   在 `target_area` 低速悬停，观察 OakD-Lite 图像与深度输出。
2. **路径规划/避障**：
   在 `target_area2` 进行点到点穿越，统计碰撞率与轨迹长度。
3. **鲁棒性测试**：
   在相同障碍布局下，改变障碍整体位姿（父 `pose`）验证算法泛化。

---

## 11. 排错指南

### 问题 1：Gazebo 启动后提示找不到模型

- 检查是否设置：`GZ_SIM_RESOURCE_PATH=$PWD/models`
- 检查 world 中 `model://xxx` 名称是否与目录一致

### 问题 2：运行脚本后 world 不是你当前仓库版本

`simulation-gazebo` 默认会下载官方主分支 zip，而不是自动使用你本地改过的仓库。  
若要强制用当前仓库，建议使用“方式 A”直接启动。

### 问题 3：Gazebo 打开但飞机不动

只启动 Gazebo 不够；你还需要在 PX4 仓库里启动 `px4_sitl`。

---

## 12. 推荐给同学的最小复现实验清单

把以下步骤发给同学，基本都能复现：

```bash
# 1) 克隆你这个仓库
# 2) 打开终端 A
cd /path/to/gz-custom
GZ_SIM_RESOURCE_PATH=$PWD/models gz sim -r $PWD/worlds/default.sdf

# 3) 打开终端 B（PX4-Autopilot 仓库）
make px4_sitl gz_x500
```

可选检查：

- Gazebo 中是否看到两组黄色边框区域；
- 是否看到不同尺寸圆柱障碍；
- 飞机起飞后是否在场景中正常显示传感器与姿态。

---

## 13. 后续维护建议（可选）

- 给 `worlds/default.sdf` 中新增区域参数化（例如脚本生成障碍布局）；
- 给圆柱模型补充纹理和材质标签，提升可视辨识度；
- 如需“有风任务”，将 WindEffects 插件做成开关版 world（`default_wind.sdf`）；
- 为课程发布一个 `reproduce.sh`，一条命令拉起 Gazebo + PX4（适合新同学）。

---

## 14. 致谢

- 原始资源库：PX4 官方 `PX4-gazebo-models`
- 本仓库：在其基础上为课程实验进行定制化改造

如果你是第一次接触 PX4/Gazebo，建议先按“第 5 章 + 第 6 章”跑通，再去改世界参数。
