# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

此代码库包含两个独立的LiDAR SLAM项目，运行在不同的环境中：

1. **LeGO-LOAM** - 基于ROS的LiDAR里程计和建图系统，运行在Docker中，使用Ubuntu 18.04和ROS Melodic
2. **SGLC** - 基于语义图引导的回环闭合系统，本地运行，使用Ubuntu 20.04和ROS Noetic

## 代码库结构

- `docker_env/` - 包含Docker环境中的LeGO-LOAM项目
  - `docker_env/sc_lego_loam_ws/src/SC-LeGO-LOAM/LeGO-LOAM/` - LeGO-LOAM的主要实现
  - `docker_env/sc_lego_loam_ws/src/sglc_bridge/` - SGLC桥接节点，连接LeGO-LOAM和外部SGLC进程
- `loop_env/` - 包含用于语义回环检测的SGLC项目
  - `SGLC/` - 基于语义图的SGLC主要实现
- `docs/` - 项目文档，包含TODO、指南和测试计划

## 数据集位置 (Dataset Locations)

- **KITTI Sequence 05 ROSBag**: `data/rosbag/kitti_05.bag` (文件名待确认)
- **KITTI Sequence 05 Labels**: `data/label/kitti/05/`

## 开发环境

### LeGO-LOAM (Docker环境)
- 操作系统: Ubuntu 18.04
- ROS版本: Melodic
- 构建系统: catkin_make
- Docker化以避免与本地环境冲突
- - **C++标准**: C++14
- 依赖项：PCL 1.8

### SGLC (本地环境)
- 操作系统: Ubuntu 20.04
- ROS版本: Noetic
- 构建系统: CMake
- **C++标准**: C++17
- 依赖项: Eigen 3.3.7, PCL 1.10, Ceres-solver 2.1.0

## 常用开发命令

### 构建LeGO-LOAM
```bash
cd docker_env/sc_lego_loam_ws
catkin_make -j1  # 首次编译
catkin_make      # 后续编译
```

### 构建SGLC
```bash
cd loop_env/SGLC
mkdir build
cd build
cmake ..
make -j8
```

### 运行LeGO-LOAM完整系统
```bash
# 在Docker容器中运行
roslaunch lego_loam run.launch
# 这个启动文件会启动所有LeGO-LOAM节点以及sglc_bridge节点
```

### 运行SGLC回环检测
```bash
cd loop_env/SGLC/bin
./eval_lcd_seq  # 用于KITTI数据集
./eval_lcd_seq_kitti360  # 用于KITTI-360数据集
./eval_overlap  # 用于基于重叠的评估
```

### 运行SGLC回环姿态估计
```bash
cd loop_env/SGLC/bin
./eval_loop_poses  # 用于KITTI数据集
./eval_loop_poses_kitti360  # 用于KITTI-360数据集
```

## 架构概述

### LeGO-LOAM (已增强)
该版本的LeGO-LOAM在原有基础上进行了重要功能扩展，不再仅仅是一个激光雷达里程计，而是一个更完整的SLAM系统。
- **核心模块**:
  1. **ImageProjection**: 将点云投影到距离图像并进行地面分割。
  2. **FeatureAssociation**: 从分割后的点云中提取边缘点和平面点特征。
  3. **MapOptimization**: 进行了大幅修改，现在是系统的核心。它不仅使用GTSAM进行地图优化，
  4. **TransformFusion**: 融合IMU和里程计变换，发布最终位姿。


### SGLC
- 用于LiDAR SLAM的语义图引导粗-精-优完整回环闭合
- 为前景实例构建语义图
- 生成同时考虑拓扑属性和外观特征的LiDAR扫描描述符
- 实现粗-精-优配准方案以实现精确的6自由度姿态估计
- 主要组件：
  1. 语义图构建 - 为前景实例构建语义图
  2. 扫描描述符生成 - 考虑拓扑属性和外观特征
  3. 回环候选检索 - 从数据库中检索候选扫描
  4. 几何验证 - 使用实例节点描述符进行鲁棒稀疏节点匹配
  5. 精确姿态估计 - 采用粗-精-优配准方案

## 关键配置文件与节点

### LeGO-LOAM
- **核心代码**:
  - `docker_env/sc_lego_loam_ws/src/SC-LeGO-LOAM/LeGO-LOAM/src/mapOptmization.cpp`: **核心优化节点**。集成了闭环检测、地面约束和GTSAM图优化，发布稠密关键帧和回环候选。
  - `docker_env/sc_lego_loam_ws/src/SC-LeGO-LOAM/LeGO-LOAM/src/imageProjection.cpp`: 点云投影和地面分割节点，发布分割后的点云信息。
  - `docker_env/sc_lego_loam_ws/src/SC-LeGO-LOAM/LeGO-LOAM/src/featureAssociation.cpp`: 特征关联节点，处理分割后的点云。
  - `docker_env/sc_lego_loam_ws/src/SC-LeGO-LOAM/LeGO-LOAM/src/transformFusion.cpp`: 变换融合节点。
- **sglc_bridge节点**:
  - `docker_env/sc_lego_loam_ws/src/sglc_bridge/src/sglc_bridge_node_v2.cpp`: **桥接节点v2.0**。订阅LeGO-LOAM数据，调用外部SGLC进程，发布回环约束。
- **配置文件**:
  - `docker_env/sc_lego_loam_ws/src/SC-LeGO-LOAM/LeGO-LOAM/launch/run.launch`: 启动整个LeGO-LOAM系统的Launch文件。
  - `docker_env/sc_lego_loam_ws/src/sglc_bridge/launch/sglc_bridge.launch`: 启动sglc_bridge节点的Launch文件。
- **自定义消息**:
  - `docker_env/sc_lego_loam_ws/src/SC-LeGO-LOAM/cloud_msgs/msg/`: 包含DenseKeyframe、LoopCandidate、LoopConstraint等消息定义。

### SGLC
- **主要可执行文件**:
  - `loop_env/SGLC/bin/eval_lcd_seq`: KITTI数据集回环检测评估
  - `loop_env/SGLC/bin/eval_lcd_seq_kitti360`: KITTI-360数据集回环检测评估
  - `loop_env/SGLC/bin/eval_loop_poses`: KITTI数据集回环姿态估计评估
  - `loop_env/SGLC/bin/eval_loop_poses_kitti360`: KITTI-360数据集回环姿态估计评估
- **配置文件**:
  - `loop_env/SGLC/config/config_kitti_graph.yaml` - KITTI数据集配置
  - `loop_env/SGLC/config/config_kitti360_graph.yaml` - KITTI-360数据集配置
  - `loop_env/SGLC/config/config_ford_campus.yaml` - Ford Campus数据集配置
  - `loop_env/SGLC/config/config_apollo.yaml` - Apollo数据集配置

## 测试和评估

### Docker开发环境设置
```bash
# 进入Docker环境
cd docker_env
docker build -t lego-loam-env .
docker run -it --rm -v $(pwd):/workspace lego-loam-env

# 或者使用预配置的Docker容器（如果存在）
docker run -it --rm -v $(pwd):/workspace your-lego-loam-image
```

### 运行测试
```bash
# SGLC评估脚本
cd loop_env/SGLC/scripts
python pr_curve.py  # 用于精度-召回率曲线
python eval_overlap_dataset.py  # 用于基于重叠的评估
python eval_loop_poses.py  # 用于回环姿态估计评估
python registration_visual.py  # 用于可视化

# 调试和日志查看
# LeGO-LOAM调试
rostopic echo /lego_loam/keyframe/dense  # 查看稠密关键帧
rostopic echo /lego_loam/loop_candidate  # 查看回环候选
rostopic echo /lego_loam/loop_constraint  # 查看回环约束
```

## 重要说明

1. 两个项目运行在完全不同的环境中以避免依赖冲突
2. LeGO-LOAM需要Docker来正确设置ROS Melodic环境
3. SGLC需要手动安装特定版本的库(Eigen, PCL, Ceres)
4. 两个项目都使用KITTI数据集但有不同的处理方法
5. SGLC利用语义信息来改进回环检测和姿态估计
6. **当前状态**: 已成功实现SGLC与LeGO-LOAM的完整集成，采用异步架构v2.0
7. **优化方案**: 正在计划使用非地面点云直接优化SGLC输入（详见`docs/sglc_input_optimization_plan.md`）

## 黄金法则
- 所有的文档和注释都用中文
- 如果不确定实现细节，请务必咨询开发人员
- 每一次对命令行的操作都要用中文告诉我目的
- 所有的文档都写到'docs/'这个文件夹中
- 你的实现不要太复杂，不要用过于复杂的语法，只需要完成功能就可以了

## 关键架构决策

### 为什么选择 docker？
LeGO-LOAM 依赖于 ROS Melodic 和 Ubuntu 18.04，而 SGLC 依赖于 ROS Noetic 和 Ubuntu 20.04。
为了避免环境冲突，我们将 LeGO-LOAM 的开发放在 docker 中。

### 编译和构建系统
- **LeGO-LOAM**: 使用 `catkin_make` 进行ROS包构建，首次编译使用 `catkin_make -j1` 避免并行编译问题
- **SGLC**: 使用标准CMake构建系统，支持并行编译 `make -j8`
- **依赖管理**: LeGO-LOAM依赖PCL 1.8，SGLC依赖PCL 1.10，通过环境隔离解决版本冲突

## 系统架构与数据流

项目已实现**异步发布-订阅架构v2.0**，成功将SGLC语义回环检测集成到LeGO-LOAM系统中：

### 核心数据流
```
LeGO-LOAM (Docker Ubuntu 18.04)                    SGLC (Host Ubuntu 20.04)
┌─────────────────────────────────┐                ┌─────────────────────────┐
│  mapOptmization                 │                │                         │
│  ├─ 发布 /lego_loam/keyframe/dense           │                         │
│  └─ 发布 /lego_loam/loop_candidate  ────────▶│  sglc_bridge            │
│                                  │                │  ├─ 调用外部SGLC进程     │
│  └─ 订阅 /lego_loam/loop_constraint  │                │  └─ 发布回环约束       │
└─────────────────────────────────┘                │                         │
                                             ▶│  sglc_external_process  │
                                              │  (语义回环检测)         │
                                              │                         │
                                              │  ┌─────────────────────┐│
                                              │  │ SGLC算法             ││
                                              │  │ • 语义图构建         ││
                                              │  │ • 描述子生成         ││
                                              │  │ • 候选检索           ││
                                              │  │ • 几何验证           ││
                                              │  │ • 姿态估计           ││
                                              │  └─────────────────────┘│
                                              └─────────────────────────┘
```

### 关键ROS话题
- `/lego_loam/keyframe/dense`: 稠密关键帧点云 (DenseKeyframe消息)
- `/lego_loam/loop_candidate`: 回环候选帧信息 (LoopCandidate消息)
- `/lego_loam/loop_constraint`: 最终回环约束 (LoopConstraint消息)
- `/segmented_cloud_info`: 分割后的点云信息 (cloud_info消息，包含非地面点)

### 组件职责
1. **LeGO-LOAM**: 激光里程计和建图，发布关键帧和回环候选
2. **sglc_bridge**: 数据桥梁，准备点云+标签数据，调用外部SGLC进程
3. **SGLC**: 语义图引导的回环检测，利用语义信息提高检测精度

## 代码风格和模式

### 锚点注释

在代码库的适当位置添加特殊格式的注释，以便您自己轻松通过 `grep` 获取内联知识。

### 指南：

- 对于针对 AI 和开发人员的注释，请使用 `AIDEV-NOTE:`、`AIDEV-TODO:` 或 `AIDEV-QUESTION:`（全大写前缀）。
- **重要提示**：在扫描文件之前，请务必先尝试在相关子目录中 `grep` 查找现有锚点** `AIDEV-*`。
- 修改相关代码时，**更新相关锚点**。
- **未经人工明确指示，请勿删除 `AIDEV-NOTE`。
- 确保在文件或代码片段出现以下情况时添加相关的锚点注释：
* 过于复杂，或
* 非常重要，或
* 令人困惑，或
* 可能存在 bug


## AI 绝对不能做的事情

1. **切勿修改测试文件** - 测试包含人类意图
2. **切勿更改 API 契约** - 破坏实际应用程序
3. **切勿修改迁移文件** - 数据丢失风险
4. **切勿提交秘诀** - 使用环境变量
5. **切勿假设业务逻辑** - 务必咨询
6. **切勿删除 AIDEV 注释** - 它们的存在是有原因的

记住：我们优化可维护性，而非追求精妙之处。
如有疑问，请选择最常规的解决方案。

# AI助手工作流：step-by-step
- 在响应用户指令时，AI 助手（Claude、Cursor、GPT 等）应遵循此流程以确保清晰、准确和可维护性：
- 咨询相关指导：当用户给出指令时，查阅与请求相关的 AGENTS.md 文件中的相关指令（包括根目录和目录特定的文件）。
- 澄清模糊之处：根据你所能收集到的信息，判断是否需要进一步澄清。如有需要，请向用户提出针对性的问题再继续执行。
- 分解任务与规划：将手头的任务分解，并制定一个大致的执行计划，参考项目规范和最佳实践。
- 简单任务：如果计划/请求很简单，可以直接开始执行。
- 复杂任务：否则，将计划呈现给用户进行审核，并根据他们的反馈进行迭代。
- 跟踪进度：使用待办事项列表（内部或可选地在 TODOS.md 文件中）来跟踪多步骤或复杂任务的进度。
- 如果卡住，重新规划：如果你卡住了或被阻塞，返回到第三步重新评估并调整你的计划。
- 更新文档：在满足用户请求后，更新你修改过的文件和目录中相关的锚注释（ AIDEV-NOTE 等）和 AGENTS.md 文件。
- 用户审核：完成任务后，请用户审核你所做的工作，并根据需要重复此过程。
- 会话边界：如果用户请求与当前上下文无关，并且可以在新的会话中安全地开始，建议从头开始以避免混淆上下文。

# Knowledge Sharing and Persistence
知识共享与持久化
- 被要求记住某些信息时，务必以所有开发人员都能访问的方式持久化这些信息，而不仅仅是存放在对话记忆中
- 在适当的位置记录重要信息（如注释、文档、README 等），以便其他开发人员（无论是人类还是 AI）都能访问
- 信息应以符合项目规范的结构化方式存储
- 切勿仅在对话记忆中保留关键信息——这会创建知识孤岛
- 目标是在所有开发者（包括人类和 AI）之间实现完全的知识共享，没有任何例外
- 在建议存储信息的位置时，根据信息的类型（代码注释、文档文件、CLAUDE.md 等）推荐合适的存储位置

- 我是一个科研工作者，工程能力不是很强，所以代码不要写得太复杂，只需要验证功能就好，要不然我看不懂
- 你每次做一次大的文件的修改都要告诉我这个修改的目的和使用方法
- 我在CLAUDE.md里边明确说明了，你要给我写锚点注释，请按照我的指示来
- 请将你的所有任务的执行todo都在我指定的todo的md文档里边进行同步，这样我可以时刻清楚你做了什么调整，保证项目运行在一个正确的轨道上
- 我的docker_dev中的工程都是要运行在我的docker中的，docker中是ubuntu18的ROS Melodic，所以你直接进行测试肯定是不行的
- 我希望你写出来的代码尽可能精简，保证人类可读性，实现功能就可以了
- 每次修改都要告诉我你这么修改代码的目的是什么，然后一定要在文档中进行同步
- 完成一个阶段的开发工作后，一定要将开发文档和使用指南写到文档里，方便我后续的开发和测试、文档名字要写好，要包含功能名字、阶段等
- 我在docker中运行的代码需要的环境是ubuntu18、ROS Melodic、pcl 1.8。 -   │
- 我在本地环境中运行的代码需要的环境是ubuntu20、ROS Neotic、pcl 1.10环境是ubuntu20、ROS Neotic、pcl 1.10

## AI 行为约束 (AI Behavior Constraints)

根据2025年10月11日的讨论，AI在执行本项目时必须遵循以下核心准则：

1.  **精确修改 (Surgical Edits)**: 在修改代码时，必须采用逐个文件修改的精确、安全方式。禁止一次性对多个文件进行全局"搜索-替换"操作。
2.  **编译请求 (Compilation Requests)**: AI助手**禁止**自行尝试执行 `catkin_make` 或任何其他编译命令。在完成一组相关的代码修改后，AI必须暂停执行，并明确地请求用户在对应的环境（Docker或宿主机）中执行编译操作，然后等待用户反馈编译结果。

## 项目状态与TODO管理

### 当前项目状态
- **主要集成**: ✅ 已完成SGLC与LeGO-LOAM的完整集成
- **架构版本**: 异步发布-订阅模式 v2.0
- **最新进展**: 2025-10-14成功集成真实SGLC算法，从模拟升级为真实语义回环检测

### TODO文档系统
项目使用层次化的TODO文档系统进行任务管理：
- **主计划**: `docs/00_TODOS/01_project_implementation_roadmap.md` - 整体项目路线图
- **子任务**: `docs/00_TODOS/` 目录下的具体实施清单
- **日常工作**: `docs/00_TODOS/07_daily_work_summary_2025_10_14.md` - 每日工作总结

### 文档约定
- 所有新功能开发后必须在`docs/`目录中编写对应的使用指南
- 代码修改必须同步更新相关TODO文档
- 所有文档和注释必须使用中文
