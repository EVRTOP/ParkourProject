# 跑酷游戏项目 (Parkour Project)

## 项目简介

这是一个使用 Unreal Engine 5.3 开发的第三人称跑酷游戏。玩家控制角色在三条跑道上自动前进，通过左右移动躲避障碍物，收集道具，挑战高分。

## 当前开发阶段

**✅ 第一阶段：核心角色系统（当前）**
- 角色自动向前奔跑
- 左右换道机制（三条跑道）
- 基础跳跃系统
- 第三人称相机跟随
- 输入系统配置

**🔜 后续开发计划：**
- 第二阶段：跑道生成系统（无限循环）
- 第三阶段：障碍物系统
- 第四阶段：道具与计分系统
- 第五阶段：UI 和菜单系统
- 第六阶段：音效与视觉优化

## 技术规格

- **引擎版本：** Unreal Engine 5.3
- **开发语言：** Blueprint（蓝图）
- **目标平台：** PC (Windows)
- **目标帧率：** 60+ FPS
- **输入系统：** Enhanced Input System

## 快速开始

### 环境要求

1. **Unreal Engine 5.3** 或更高版本
   - 从 Epic Games Launcher 安装
   - 确保安装了 Blueprint 相关组件

2. **系统要求：**
   - Windows 10/11 64位
   - 至少 8GB RAM（推荐 16GB+）
   - 支持 DirectX 12 的显卡
   - 至少 20GB 可用硬盘空间

### 安装步骤

1. **克隆项目：**
   ```bash
   git clone https://github.com/EVRTOP/ParkourProject.git
   cd ParkourProject
   ```

2. **打开项目：**
   - 双击 `ParkourProject.uproject` 文件
   - 首次打开可能需要编译着色器，请耐心等待

3. **实现核心系统：**
   - 查看 `IMPLEMENTATION_GUIDE.md` 获取详细实现步骤
   - 按照指南在 UE5 编辑器中创建 Blueprint 资产

### 项目结构

```
ParkourProject/
├── Config/                      # 配置文件
│   ├── DefaultEngine.ini       # 引擎配置（已配置游戏模式和地图）
│   ├── DefaultInput.ini        # 输入配置（已配置键盘映射）
│   └── DefaultGame.ini         # 游戏配置
├── Content/                     # 内容文件夹
│   ├── Blueprints/             # 蓝图文件（需创建）
│   │   ├── BP_ParkourCharacter.uasset
│   │   ├── BP_ParkourGameMode.uasset
│   │   └── BP_ParkourPlayerController.uasset
│   ├── Maps/                   # 地图文件（需创建）
│   │   └── TestLevel.umap
│   ├── Input/                  # 输入资产（需创建）
│   │   ├── IA_Jump.uasset
│   │   ├── IA_MoveRight.uasset
│   │   └── IMC_Default.uasset
│   └── StarterContent/         # UE5 起始内容包
├── IMPLEMENTATION_GUIDE.md      # 详细实现指南
├── README.md                    # 本文件
└── ParkourProject.uproject      # UE5 项目文件
```

### 测试游戏

1. 按照 `IMPLEMENTATION_GUIDE.md` 完成所有 Blueprint 创建
2. 在 UE5 编辑器中打开 `Content/Maps/TestLevel`
3. 点击工具栏的 **Play** 按钮（或按 `Alt+P`）
4. 使用键盘控制：
   - **A / 左方向键**：向左切换跑道
   - **D / 右方向键**：向右切换跑道
   - **Space / W / 上方向键**：跳跃

## 核心功能详解

### 1. 角色移动系统

- **自动前进：** 角色以 700 单位/秒的速度持续向前奔跑
- **跑道系统：** 三条平行跑道（左、中、右），间距 300 单位
- **换道机制：** 使用插值实现平滑的左右移动过渡

### 2. 跳跃机制

- **跳跃高度：** Jump Z Velocity = 600.0
- **防二段跳：** 使用 `Is Falling` 检测，确保只能在地面跳跃
- **重力设置：** Gravity Scale = 1.5，提供更快的下落速度

### 3. 相机系统

- **跟随距离：** 600 单位
- **视角角度：** 俯视 30 度
- **平滑跟随：** 使用 Spring Arm 的 Camera Lag 功能
- **稳定性：** 固定相机角度，不受玩家旋转影响

### 4. 输入系统

采用 UE5 的 Enhanced Input System：

**已配置的输入映射：**
- **Jump Action:** Space, W, Up Arrow
- **MoveRight Axis:** A(-1), D(+1), Left(-1), Right(+1)
- **手柄支持：** Left Thumbstick

## 开发指南

### 添加新功能

1. 参考 `IMPLEMENTATION_GUIDE.md` 了解现有系统结构
2. 在对应的 Blueprint 中添加新逻辑
3. 遵循现有的命名规范和代码结构
4. 确保性能影响最小化

### 调试技巧

1. **使用 Print String：**
   - 在 Blueprint 中添加 "Print String" 节点查看变量值

2. **启用调试工具：**
   - 按 `~` 打开控制台
   - 输入 `stat fps` 查看帧率
   - 输入 `stat game` 查看游戏统计

3. **检查碰撞：**
   - 控制台输入 `show collision` 显示碰撞体

### 性能优化

当前系统已经过优化，维持 60+ FPS：

- ✅ 使用插值代替每帧计算
- ✅ Enhanced Input 高效处理输入
- ✅ Camera Lag 减少相机计算
- ✅ Character Movement 组件自带优化

## 游戏设计文档

### 核心玩法循环

1. 角色自动前进
2. 玩家左右移动躲避障碍（待实现）
3. 收集道具增加分数（待实现）
4. 难度逐渐提升（待实现）
5. 失败后重新开始（待实现）

### 跑道设计

```
  左跑道 (Y=-300)    中间跑道 (Y=0)    右跑道 (Y=300)
       |                  |                  |
       |    障碍物        |                  |
       |                  |    障碍物        |
       |                  |                  |
       |    道具          |    障碍物        |
       |                  |                  |
```

### 难度曲线（计划）

- **0-500米：** 教学阶段，障碍物少，速度慢
- **500-1000米：** 普通难度，障碍物增多
- **1000米+：** 困难难度，速度提升，障碍物密集

## 常见问题 (FAQ)

### Q: 如何修改角色速度？
A: 在 `BP_ParkourCharacter` 中修改 `ForwardSpeed` 变量（默认 700）

### Q: 如何调整跑道间距？
A: 在 `BP_ParkourCharacter` 中修改 `LaneDistance` 变量（默认 300）

### Q: 如何让相机更远/更近？
A: 在 `BP_ParkourCharacter` 的 Spring Arm 组件中修改 `Target Arm Length`

### Q: 跳跃太高/太低怎么办？
A: 在 Character Movement 组件中修改 `Jump Z Velocity`

### Q: 换道速度太快/太慢？
A: 在 `BP_ParkourCharacter` 中修改 `LaneChangeSpeed` 变量

### Q: 无法跳跃？
A: 检查：
1. IMC_Default 是否在 PlayerController 中添加
2. IA_Jump 是否正确映射到按键
3. Character Movement 的 Jump Z Velocity 是否 > 0

### Q: 输入无响应？
A: 确认：
1. Enhanced Input 插件已启用（默认启用）
2. Input Mapping Context 已添加到 PlayerController
3. Input Actions 已正确配置

## 贡献指南

欢迎贡献！请遵循以下步骤：

1. Fork 本项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

### 代码规范

- **Blueprint 命名：** 使用 `BP_` 前缀
- **变量命名：** 使用 PascalCase（如 `ForwardSpeed`）
- **函数命名：** 使用动词开头（如 `CalculateTargetPosition`）
- **注释：** 为复杂逻辑添加 Comment 节点

## 许可证

本项目采用 MIT 许可证 - 详见 LICENSE 文件

## 联系方式

- **项目主页：** https://github.com/EVRTOP/ParkourProject
- **问题反馈：** 请使用 GitHub Issues

## 致谢

- Unreal Engine 5 by Epic Games
- StarterContent 资产包

---

**当前版本：** v0.1.0 - 核心角色系统  
**最后更新：** 2026-01-01

祝你开发愉快！🎮
