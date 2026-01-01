# Blueprint 节点连接速查表

本文档提供了创建跑酷游戏 Blueprint 时需要的节点连接速查表，便于快速实现。

## BP_ParkourPlayerController

### Event Graph

```
Event BeginPlay
├─> Get Player Controller
    └─> Get Local Player
        └─> Get Subsystem (EnhancedInputLocalPlayerSubsystem)
            └─> Add Mapping Context
                ├─ Mapping Context: IMC_Default
                └─ Priority: 0
```

**搜索关键词：**
- "BeginPlay"
- "Get Subsystem" -> 选择 "EnhancedInputLocalPlayerSubsystem"
- "Add Mapping Context"

---

## BP_ParkourCharacter

### 变量列表

| 变量名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| ForwardSpeed | Float | 700.0 | 前进速度 |
| LaneDistance | Float | 300.0 | 跑道间距 |
| CurrentLane | Integer | 0 | 当前跑道（-1左，0中，1右）|
| LaneChangeSpeed | Float | 10.0 | 换道插值速度 |
| bCanJump | Boolean | True | 是否允许跳跃 |

### Event Graph - 自动前进

```
Event Tick (Delta Seconds)
├─> Get Actor Forward Vector
    └─> Vector * Float (Forward Vector * ForwardSpeed)
        └─> Add Movement Input
            ├─ World Direction: [结果向量]
            └─ Scale Value: 1.0
```

**节点搜索：**
- "Event Tick"
- "Get Actor Forward Vector"
- "Multiply" (Vector * Float)
- "Add Movement Input"

### Event Graph - 平滑换道

```
Event Tick (Delta Seconds)
├─> Integer * Float (CurrentLane * LaneDistance)
│   └─> [存储为局部变量：TargetY]
│
├─> Get Actor Location
│   └─> Break Vector (分解为 X, Y, Z)
│       └─> Y -> [当前Y位置]
│
├─> FInterp To
│   ├─ Current: [当前Y位置]
│   ├─ Target: [TargetY]
│   ├─ Delta Time: [Delta Seconds from Event Tick]
│   └─ Interp Speed: LaneChangeSpeed
│       └─> [存储为局部变量：NewY]
│
└─> Make Vector
    ├─ X: [原始X]
    ├─ Y: [NewY]
    └─ Z: [原始Z]
        └─> Set Actor Location
            ├─ New Location: [新向量]
            ├─ Sweep: True
            └─ Teleport: False
```

**节点搜索：**
- "Multiply" (Int * Float)
- "Get Actor Location"
- "Break Vector"
- "FInterp To" 或 "Float Interpolate To"
- "Make Vector"
- "Set Actor Location"

### Input - 跳跃

```
InputAction IA_Jump (Triggered)
├─> Get Character Movement
    └─> Is Falling
        └─> Branch
            ├─ False (在地面):
            │   ├─> Jump (from Character)
            │   └─> Set bCanJump (False)
            └─ True (在空中):
                └─> [什么都不做]

Event Landed
└─> Set bCanJump (True)
```

**节点搜索：**
- 右键 -> "Enhanced Input Action" -> 选择 IA_Jump -> Triggered
- "Get Character Movement"
- "Is Falling"
- "Branch"
- "Jump"
- "Set bCanJump"
- "Event Landed"

### Input - 左右移动（方法一：多分支）

```
InputAction IA_MoveRight (Triggered)
├─> Get Action Value (Axis1D)
│   └─> [存储为局部变量：InputValue]
│
├─> Branch (InputValue > 0.5)
│   └─ True: Set CurrentLane (1)
│
├─> Branch (InputValue < -0.5)
│   └─ True: Set CurrentLane (-1)
│
└─> Branch (Abs(InputValue) < 0.1)
    └─ True: Set CurrentLane (0)
```

### Input - 左右移动（方法二：简化版，推荐）

```
InputAction IA_MoveRight (Triggered)
├─> Get Action Value (Axis1D)
    └─> Clamp (Min: -1.0, Max: 1.0)
        └─> Round to Int
            └─> Set CurrentLane
```

**节点搜索：**
- 右键 -> "Enhanced Input Action" -> 选择 IA_MoveRight -> Triggered
- "Get Action Value" -> 设置类型为 Float
- "Clamp (Float)"
- "Round" 或 "Float to Int"
- "Set CurrentLane"

---

## Components 设置快速参考

### BP_ParkourCharacter Components

#### CapsuleComponent
```
Capsule Half Height: 88.0
Capsule Radius: 42.0
```

#### Mesh (SkeletalMeshComponent)
```
Location: (0, 0, -88)
Rotation: (0, 0, -90)
Static Mesh: Shape_Capsule (临时)
```

#### SpringArm (CameraBoom)
```
Target Arm Length: 600.0
Socket Offset: (0, 0, 150)
Rotation: (-30, 0, 0)
Use Pawn Control Rotation: False
Do Collision Test: True
Enable Camera Lag: True
Camera Lag Speed: 10.0
Camera Lag Max Distance: 0.0
```

#### Camera (FollowCamera)
```
[附加到 SpringArm]
Field of View: 90.0
Aspect Ratio: 1.777778
```

#### CharacterMovement
```
Max Walk Speed: 700.0
Jump Z Velocity: 600.0
Air Control: 0.2
Gravity Scale: 1.5
Ground Friction: 8.0
Braking Deceleration Walking: 2048.0
```

---

## Input System 配置

### IA_Jump (Input Action)
```
Value Type: Digital (bool)
```

### IA_MoveRight (Input Action)
```
Value Type: Axis1D (float)
```

### IMC_Default (Input Mapping Context)

#### Mappings

**IA_Jump:**
- Key: Space Bar
- Key: W
- Key: Up Arrow

**IA_MoveRight:**
- Key: D, Modifiers: None, Scale: 1.0
- Key: A, Modifiers: None, Scale: -1.0
- Key: Right, Modifiers: None, Scale: 1.0
- Key: Left, Modifiers: None, Scale: -1.0
- Key: Gamepad Left Thumbstick X

---

## 常用节点参数速查

### Add Movement Input
```
World Direction: Vector (方向)
Scale Value: Float (强度，通常 1.0)
Force: Boolean (是否强制，通常 False)
```

### Set Actor Location
```
New Location: Vector
Sweep: Boolean (True = 碰撞检测)
Teleport: Boolean (False = 平滑移动)
```

### FInterp To (Float Interpolate)
```
Current: Float (当前值)
Target: Float (目标值)
Delta Time: Float (Event Tick 的 Delta Seconds)
Interp Speed: Float (插值速度，越大越快)
```

### Branch
```
Condition: Boolean
True: 执行真分支
False: 执行假分支
```

### Get Action Value
```
Action Value: 选择类型
- Boolean (Digital)
- Float (Axis1D)
- Vector2D (Axis2D)
- Vector (Axis3D)
```

---

## 测试关卡快速搭建

### 地面平台
```
Actor: Cube
Location: (0, 0, -100)
Scale: (100, 9, 1)  # 10000 x 900 x 100
Material: M_Concrete_Tiles
```

### 左边界
```
Actor: Cube
Location: (0, -450, 0)
Scale: (100, 1, 3)
```

### 右边界
```
Actor: Cube
Location: (0, 450, 0)
Scale: (100, 1, 3)
```

### 跑道标记线（中间）
```
Actor: Cube
Location: (0, 0, -99)
Scale: (100, 0.5, 0.1)
Material: 亮色（黄色/白色）
```

### 跑道标记线（左）
```
Actor: Cube
Location: (0, -300, -99)
Scale: (100, 0.5, 0.1)
```

### 跑道标记线（右）
```
Actor: Cube
Location: (0, 300, -99)
Scale: (100, 0.5, 0.1)
```

### Player Start
```
Actor: Player Start
Location: (-2000, 0, 0)
Rotation: (0, 0, 0)
```

### Lighting

**Sky Light:**
```
Actor: Sky Light
Intensity: 1.0
Source Type: SLS Captured Scene
```

**Directional Light:**
```
Actor: Directional Light
Rotation: (-30, 0, 0)
Intensity: 10.0
```

**Sky Atmosphere:**
```
Actor: Sky Atmosphere
[使用默认设置]
```

---

## 调试命令速查

在游戏运行时按 `~` 打开控制台，输入以下命令：

| 命令 | 说明 |
|------|------|
| `stat fps` | 显示帧率 |
| `stat game` | 显示游戏统计 |
| `stat unit` | 显示各系统耗时 |
| `show collision` | 显示碰撞体 |
| `show bounds` | 显示边界框 |
| `slomo 0.5` | 慢动作（0.5倍速）|
| `slomo 2.0` | 快进（2倍速）|
| `slomo 1.0` | 恢复正常速度 |
| `god` | 上帝模式（无敌）|
| `fly` | 飞行模式 |
| `walk` | 恢复行走模式 |

---

## 快捷键速查

| 快捷键 | 功能 |
|--------|------|
| `Alt + P` | 在编辑器中开始游戏 |
| `Esc` | 停止游戏 |
| `F8` | 弹出角色（自由相机）|
| `F9` | 开启/关闭碰撞显示 |
| `~` | 打开控制台 |
| `Tab` | 在 Viewport 中切换聚焦 |

---

**提示：** 将此文件放在第二个显示器或打印出来，在创建 Blueprint 时作为参考，可以大大提高效率！
