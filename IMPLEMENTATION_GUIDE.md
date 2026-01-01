# 跑酷游戏 - 核心系统实现指南

本文档提供了在Unreal Engine 5.3中实现跑酷游戏核心系统的详细步骤。

## 目录结构

创建以下文件夹结构（在Content Browser中右键 -> New Folder）：

```
Content/
├── Blueprints/          # 蓝图文件夹
│   ├── BP_ParkourCharacter
│   ├── BP_ParkourGameMode
│   └── BP_ParkourPlayerController
├── Maps/                # 地图文件夹
│   └── TestLevel
└── Input/               # 输入配置文件夹
    ├── IA_Jump
    ├── IA_MoveRight
    └── IMC_Default
```

## 第一步：创建输入系统（Enhanced Input）

### 1.1 创建输入动作 (Input Actions)

#### IA_Jump（跳跃动作）
1. 在 `Content/Input` 文件夹中右键 -> Input -> Input Action
2. 命名为 `IA_Jump`
3. 打开后设置：
   - Value Type: Digital (bool)

#### IA_MoveRight（左右移动动作）
1. 在 `Content/Input` 文件夹中右键 -> Input -> Input Action
2. 命名为 `IA_MoveRight`
3. 打开后设置：
   - Value Type: Axis1D (float)

### 1.2 创建输入映射上下文 (Input Mapping Context)

#### IMC_Default
1. 在 `Content/Input` 文件夹中右键 -> Input -> Input Mapping Context
2. 命名为 `IMC_Default`
3. 打开后添加映射：

**跳跃映射：**
- 点击 `+` 添加映射
- 选择 Action: `IA_Jump`
- 添加键位：
  - Space Bar（空格键）
  - W（W键）
  - Up Arrow（上方向键）

**左右移动映射：**
- 点击 `+` 添加映射
- 选择 Action: `IA_MoveRight`
- 添加键位和修改器：
  - D键：Scale = 1.0
  - A键：Scale = -1.0
  - Right Arrow：Scale = 1.0
  - Left Arrow：Scale = -1.0
  - Gamepad Left Thumbstick X

## 第二步：创建游戏模式蓝图

### BP_ParkourGameMode
1. 在 `Content/Blueprints` 文件夹中右键 -> Blueprint Class -> Game Mode Base
2. 命名为 `BP_ParkourGameMode`
3. 打开蓝图，在 Class Defaults 中设置：
   - Default Pawn Class: BP_ParkourCharacter（稍后创建）
   - Player Controller Class: BP_ParkourPlayerController（稍后创建）

## 第三步：创建玩家控制器蓝图

### BP_ParkourPlayerController
1. 在 `Content/Blueprints` 文件夹中右键 -> Blueprint Class -> Player Controller
2. 命名为 `BP_ParkourPlayerController`
3. 打开蓝图，添加以下逻辑：

#### Event Graph（事件图表）：

**Event BeginPlay：**
```
Event BeginPlay
  -> Enhanced Input Local Player Subsystem (Get Subsystem)
  -> Add Mapping Context
     - Mapping Context: IMC_Default
     - Priority: 0
```

具体步骤：
1. 添加 Event BeginPlay 节点
2. 从 Event BeginPlay 拖出，搜索 "Get Subsystem"
3. 选择 "Enhanced Input Local Player Subsystem"
4. 从该节点拖出，搜索 "Add Mapping Context"
5. 设置 Mapping Context 为 IMC_Default
6. Priority 设置为 0

## 第四步：创建角色蓝图（最重要）

### BP_ParkourCharacter
1. 在 `Content/Blueprints` 文件夹中右键 -> Blueprint Class -> Character
2. 命名为 `BP_ParkourCharacter`

### 4.1 组件设置（Components面板）

**CapsuleComponent（已有）：**
- Capsule Half Height: 88.0
- Capsule Radius: 42.0

**Mesh（已有）：**
- 可选：添加临时网格体（Content/StarterContent/Shapes/Shape_Capsule）
- Location: (0, 0, -88)
- Rotation: (0, 0, -90)

**SpringArm（新增组件）：**
1. 添加组件 -> Spring Arm
2. 命名：CameraBoom
3. 设置：
   - Target Arm Length: 600.0
   - Socket Offset: (0, 0, 150)
   - Rotation: (-30, 0, 0)（俯视角度）
   - Use Pawn Control Rotation: False
   - Enable Camera Lag: True
   - Camera Lag Speed: 10.0

**Camera（新增组件）：**
1. 附加到 SpringArm 组件
2. 命名：FollowCamera
3. 设置保持默认

**CharacterMovement（已有组件，在组件列表中找到）：**
- Max Walk Speed: 700.0（前进速度）
- Jump Z Velocity: 600.0（跳跃高度）
- Air Control: 0.2
- Gravity Scale: 1.5
- Ground Friction: 8.0

### 4.2 变量定义（Variables面板）

创建以下变量：

1. **ForwardSpeed**（前进速度）
   - Type: Float
   - Default Value: 700.0
   - Instance Editable: True
   - Category: "Movement"

2. **LaneDistance**（跑道间距）
   - Type: Float
   - Default Value: 300.0
   - Instance Editable: True
   - Category: "Movement"

3. **CurrentLane**（当前跑道）
   - Type: Integer
   - Default Value: 0
   - Instance Editable: False
   - Category: "Movement"

4. **LaneChangeSpeed**（换道速度）
   - Type: Float
   - Default Value: 10.0
   - Instance Editable: True
   - Category: "Movement"

5. **bCanJump**（是否可以跳跃）
   - Type: Boolean
   - Default Value: True
   - Instance Editable: False
   - Category: "Movement"

### 4.3 事件图表逻辑（Event Graph）

#### 1. Event BeginPlay（初始化）
```
Event BeginPlay
  -> Set ForwardSpeed to 700.0
  -> Set CurrentLane to 0
  -> Set bCanJump to True
```

#### 2. Event Tick（持续更新）
```
Event Tick
  -> Branch (Is Valid: Character Movement Component)
       True ->
         // 自动向前移动
         Get Forward Vector (from Actor)
         -> Multiply (Vector * Float)
            Vector: Forward Vector
            Float: ForwardSpeed
         -> Add Movement Input
            World Direction: Result
            Scale Value: 1.0

         // 平滑换道
         Calculate Target Y Position:
           CurrentLane * LaneDistance
         
         Get Actor Location
         -> Break Vector (get current Y)
         
         FInterp To (Float)
           Current: Current Y
           Target: Target Y Position
           Delta Time: Delta Seconds
           Interp Speed: LaneChangeSpeed
         
         -> Make Vector (X=current X, Y=interpolated Y, Z=current Z)
         -> Set Actor Location (New Location, Sweep=true)
```

详细实现步骤：

**自动向前移动部分：**
1. 在 Event Tick 后，添加 "Get Actor Forward Vector" 节点
2. 添加 "Vector * Float" 节点，将 Forward Vector 连接到 Vector 引脚
3. 将 ForwardSpeed 变量连接到 Float 引脚
4. 添加 "Add Movement Input" 节点
5. 将乘法结果连接到 World Direction
6. Scale Value 设置为 1.0

**平滑换道部分：**
1. 添加 "Integer * Float" 节点
   - A: CurrentLane 变量
   - B: LaneDistance 变量
   - 结果是目标 Y 位置（TargetY）
2. 添加 "Get Actor Location" 节点
3. 添加 "Break Vector" 节点，将 Location 分解为 X, Y, Z
4. 添加 "FInterp To" 节点（Float Interpolate）
   - Current: 当前 Y 位置
   - Target: TargetY（步骤1的结果）
   - Delta Time: Event Tick 的 Delta Seconds
   - Interp Speed: LaneChangeSpeed 变量
5. 添加 "Make Vector" 节点
   - X: 保持当前 X
   - Y: 插值后的 Y（步骤4的结果）
   - Z: 保持当前 Z
6. 添加 "Set Actor Location" 节点
   - New Location: 步骤5的结果
   - Sweep: True

#### 3. 输入绑定 - 跳跃

```
Enhanced Input Action (IA_Jump)
  -> Triggered ->
       Get Character Movement
       -> Is Falling? (Branch)
            False (在地面) ->
              Jump (from Character)
              Set bCanJump to False
              
            True (在空中) ->
              Do Nothing (防止二段跳)
              
Landed Event ->
  Set bCanJump to True
```

详细实现步骤：
1. 右键 -> Enhanced Input Action -> IA_Jump
2. 选择 Triggered 事件
3. 添加 "Get Character Movement" 节点
4. 添加 "Is Falling" 节点
5. 添加 "Branch" 节点
6. False 分支：
   - 添加 "Jump" 节点（从 Character 类继承）
   - 添加 "Set bCanJump" 设置为 False
7. 在另一处添加 "Event Landed" 节点
8. 从 Event Landed 拖出 "Set bCanJump" 设置为 True

#### 4. 输入绑定 - 左右移动

```
Enhanced Input Action (IA_MoveRight)
  -> Triggered ->
       Get Action Value (float)
       -> Branch (Value > 0.5)
            True -> Set CurrentLane to 1 (右边跑道)
       -> Branch (Value < -0.5)
            True -> Set CurrentLane to -1 (左边跑道)
       -> Branch (Abs(Value) < 0.1)
            True -> Set CurrentLane to 0 (中间跑道)
```

详细实现步骤：
1. 右键 -> Enhanced Input Action -> IA_MoveRight
2. 选择 Triggered 事件
3. 添加 "Get Action Value" 节点，设置类型为 Float
4. 添加三个分支判断：
   - Branch 1: Value > 0.5 -> Set CurrentLane = 1
   - Branch 2: Value < -0.5 -> Set CurrentLane = -1
   - Branch 3: Abs(Value) < 0.1 -> Set CurrentLane = 0

**简化版本（推荐）：**
```
Enhanced Input Action (IA_MoveRight)
  -> Triggered ->
       Get Action Value (float)
       -> Clamp (min=-1, max=1)
       -> Round (取整)
       -> Set CurrentLane
```

## 第五步：创建测试关卡

### TestLevel
1. 在 `Content/Maps` 文件夹中 File -> New Level -> Empty Level
2. 保存为 `TestLevel`

### 5.1 添加基础环境

**1. 添加天空光照（Skylight）：**
- Place Actors 面板 -> Lights -> Sky Light
- 拖入场景
- Intensity: 1.0

**2. 添加定向光（Directional Light）：**
- Place Actors 面板 -> Lights -> Directional Light
- 拖入场景
- Rotation: (-30, 0, 0)
- Intensity: 10.0

**3. 添加天空盒（Sky Atmosphere）：**
- Place Actors 面板 -> Visual Effects -> Sky Atmosphere
- 拖入场景

**4. 添加后处理体积（Post Process Volume）：**
- Place Actors 面板 -> Visual Effects -> Post Process Volume
- 拖入场景
- Settings -> Infinite Extent (Unbound): True

### 5.2 创建测试跑道

**1. 地面平台：**
- Place Actors 面板 -> Basic -> Cube
- Transform:
  - Location: (0, 0, -100)
  - Rotation: (0, 0, 0)
  - Scale: (100, 9, 1)（长10000单位，宽900单位）
- Material: 使用 StarterContent 中的 M_Concrete_Tiles 或任意材质

**2. 添加左右边界（可选，用于视觉引导）：**

左边界墙：
- Cube
- Transform:
  - Location: (0, -450, 0)
  - Scale: (100, 1, 3)

右边界墙：
- Cube
- Transform:
  - Location: (0, 450, 0)
  - Scale: (100, 1, 3)

**3. 跑道标记（帮助识别三条跑道）：**

中间线（Y=0）：
- Cube
- Transform:
  - Location: (0, 0, -99)
  - Scale: (100, 0.5, 0.1)
- Material: 设置为亮色（如黄色）

左跑道线（Y=-300）：
- Cube
- Transform:
  - Location: (0, -300, -99)
  - Scale: (100, 0.5, 0.1)

右跑道线（Y=300）：
- Cube
- Transform:
  - Location: (0, 300, -99)
  - Scale: (100, 0.5, 0.1)

### 5.3 设置玩家起始位置

1. Place Actors 面板 -> Basic -> Player Start
2. 拖入场景
3. Transform:
   - Location: (-2000, 0, 0)（在跑道起点）
   - Rotation: (0, 0, 0)（朝向 +X 方向）

## 第六步：配置和测试

### 6.1 设置世界设置

1. 打开 TestLevel
2. Window -> World Settings
3. Game Mode -> GameMode Override: BP_ParkourGameMode

### 6.2 配置项目设置

1. Edit -> Project Settings
2. Maps & Modes:
   - Default GameMode: BP_ParkourGameMode
   - Editor Startup Map: TestLevel
   - Game Default Map: TestLevel

### 6.3 测试游戏

1. 点击工具栏的 Play 按钮（或按 Alt+P）
2. 测试以下功能：
   - ✅ 角色自动向前奔跑
   - ✅ 按 A/D 或 左右方向键切换跑道
   - ✅ 按 Space/W/上方向键跳跃
   - ✅ 在空中不能二段跳
   - ✅ 相机平滑跟随
   - ✅ 换道平滑过渡

## 常见问题排查

### 问题1：角色不移动
- 检查 CharacterMovement 组件的 Max Walk Speed 是否设置正确
- 确认 Event Tick 中的自动前进逻辑已正确实现
- 检查 ForwardSpeed 变量值

### 问题2：输入无响应
- 确认 IMC_Default 已在 PlayerController 的 BeginPlay 中添加
- 检查 Input Actions 是否正确映射到 IMC_Default
- 验证 Enhanced Input 插件已启用

### 问题3：相机抖动
- 增加 Camera Lag Speed（降低灵敏度）
- 确认 Enable Camera Lag 已勾选
- 检查 Set Actor Location 是否使用了 Sweep

### 问题4：跳跃太高或太低
- 调整 CharacterMovement 的 Jump Z Velocity
- 修改 Gravity Scale（增加重力感）

### 问题5：换道太快或太慢
- 调整 LaneChangeSpeed 变量
- 修改 LaneDistance 变量

## 性能优化建议

1. **Tick 优化：**
   - 如果性能有问题，可以将 Event Tick 中的换道逻辑改为定时器
   - 使用 Set Actor Location 时，确保 Sweep 为 True 避免穿墙

2. **输入优化：**
   - Enhanced Input 已经很高效，无需额外优化

3. **相机优化：**
   - Camera Lag 已经提供了平滑效果
   - 避免在 Tick 中频繁计算相机位置

## 后续扩展建议

完成基础系统后，可以继续实现：

1. **动画系统：**
   - 创建 Animation Blueprint
   - 实现跑步、跳跃、空中动画

2. **障碍物系统：**
   - 创建障碍物 Blueprint
   - 实现碰撞检测和游戏结束逻辑

3. **道具系统：**
   - 实现金币收集
   - 添加道具效果（无敌、磁铁等）

4. **UI系统：**
   - 创建 Widget Blueprint
   - 实现分数、生命值显示

5. **关卡生成：**
   - 实现程序化跑道生成
   - 添加跑道循环和回收机制

## 验收清单

实现完成后，确保以下功能正常：

- [ ] 角色能够自动持续向前奔跑（速度约 700 单位/秒）
- [ ] 玩家可以通过 A/D 键或方向键控制角色左右移动（切换三条跑道）
- [ ] 左右切换有平滑过渡效果
- [ ] 跳跃功能正常（Space/W/上方向键）
- [ ] 不能在空中二段跳
- [ ] 落地后可以再次跳跃
- [ ] 相机位于角色后上方，俯视角度
- [ ] 相机跟随流畅，无明显抖动
- [ ] 点击 Play 后游戏立即开始，无需额外操作
- [ ] 帧率稳定在 60fps 以上
- [ ] 角色不会掉出跑道（或有掉落后的处理）

---

**祝开发顺利！这是跑酷游戏的核心基础，后续所有功能都将基于此系统构建。**
