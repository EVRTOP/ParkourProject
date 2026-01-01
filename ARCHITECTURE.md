# 系统架构图

本文档提供了跑酷游戏核心系统的可视化架构说明。

## 整体系统架构

```
┌─────────────────────────────────────────────────────────────┐
│                      游戏启动流程                            │
└─────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│  BP_ParkourGameMode (游戏模式)                               │
│  ├─ Default Pawn: BP_ParkourCharacter                       │
│  └─ Player Controller: BP_ParkourPlayerController           │
└─────────────────────────────────────────────────────────────┘
                             │
                             ▼
        ┌────────────────────┴────────────────────┐
        │                                          │
        ▼                                          ▼
┌─────────────────────┐                ┌─────────────────────┐
│ PlayerController    │                │  Character          │
│ (输入处理)          │───────────────▶│  (角色控制)         │
│                     │    输入事件     │                     │
│ - IMC_Default       │                │ - 自动前进           │
│ - IA_Jump           │                │ - 左右移动           │
│ - IA_MoveRight      │                │ - 跳跃               │
└─────────────────────┘                │ - 相机跟随           │
                                       └─────────────────────┘
```

## 角色组件层次结构

```
BP_ParkourCharacter (Character)
│
├─ CapsuleComponent (碰撞胶囊)
│   └─ Mesh (SkeletalMesh，临时使用 Shape_Capsule)
│
├─ CharacterMovement (移动组件)
│   ├─ Max Walk Speed: 700
│   ├─ Jump Z Velocity: 600
│   ├─ Air Control: 0.2
│   ├─ Gravity Scale: 1.5
│   └─ Ground Friction: 8.0
│
└─ SpringArm (CameraBoom)
    ├─ Target Arm Length: 600
    ├─ Socket Offset: (0, 0, 150)
    ├─ Rotation: (-30, 0, 0)
    ├─ Enable Camera Lag: True
    └─ Camera Lag Speed: 10.0
        │
        └─ Camera (FollowCamera)
            ├─ FOV: 90
            └─ Aspect Ratio: 16:9
```

## 输入系统数据流

```
┌──────────────┐
│ 物理输入设备  │ (键盘/手柄)
└──────┬───────┘
       │ Space/W/Up
       │ A/D/Left/Right
       ▼
┌──────────────────┐
│ Enhanced Input   │
│ System           │
└──────┬───────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│  IMC_Default (Input Mapping Context)    │
│  映射物理按键到逻辑动作                  │
│                                         │
│  Space → IA_Jump (Digital)              │
│  W → IA_Jump (Digital)                  │
│  A → IA_MoveRight (Axis1D, -1.0)        │
│  D → IA_MoveRight (Axis1D, +1.0)        │
└─────────┬───────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────┐
│  BP_ParkourPlayerController             │
│  (BeginPlay 时添加 Mapping Context)      │
└─────────┬───────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────┐
│  BP_ParkourCharacter                    │
│  响应 Input Action 事件                  │
│                                         │
│  IA_Jump → 跳跃逻辑                      │
│  IA_MoveRight → 换道逻辑                 │
└─────────────────────────────────────────┘
```

## 角色移动系统流程

### 自动前进系统

```
Event Tick (每帧执行)
│
├─ Get Actor Forward Vector → (1, 0, 0)
│
├─ Multiply by ForwardSpeed (700) → (700, 0, 0)
│
└─ Add Movement Input → CharacterMovement
                        │
                        ▼
                   角色持续向前移动
```

### 左右换道系统

```
玩家输入: 按 D 键
│
▼
IA_MoveRight (Triggered)
│
├─ Get Action Value → 1.0
│
├─ Round → 1
│
└─ Set CurrentLane = 1
         │
         ▼
    Event Tick (每帧)
         │
         ├─ CurrentLane (1) * LaneDistance (300) = 300
         │                                           │
         │  目标位置                                  │
         ▼                                           ▼
    Get Actor Location → (X, CurrentY, Z)
         │
         ├─ FInterp To (CurrentY → 300, Speed: 10)
         │        │
         │        ▼ NewY
         │
         └─ Set Actor Location (X, NewY, Z)
                  │
                  ▼
            平滑移动到右跑道
```

### 跳跃系统

```
玩家输入: 按 Space
│
▼
IA_Jump (Triggered)
│
├─ Get Character Movement
│   │
│   └─ Is Falling?
│        │
│        ├─ True (在空中) → 忽略输入 (防止二段跳)
│        │
│        └─ False (在地面)
│             │
│             ▼
│        Jump (Launch Character)
│             │
│             ├─ Velocity.Z = Jump Z Velocity (600)
│             │
│             └─ Set bCanJump = False
│
▼
角色跳起
│
│ (空中飞行...)
│
▼
Event Landed (着陆)
│
└─ Set bCanJump = True (允许再次跳跃)
```

## 相机系统

```
角色移动
│
▼
SpringArm (弹簧臂)
├─ 固定在角色位置
├─ 向后延伸 600 单位
├─ 向上偏移 150 单位
├─ 俯视角 -30 度
│
├─ Camera Lag (延迟跟随)
│   ├─ 当前位置 → 目标位置
│   └─ 插值速度: 10.0
│
▼
Camera (相机)
├─ 附加在 SpringArm 末端
├─ 自动面向角色
└─ 渲染视图
    │
    ▼
玩家看到的画面：
俯视角度，角色在画面下方，前方跑道清晰可见
```

## 跑道布局

```
俯视图:
                    ┌────────── 前进方向 (+X) ──────────►

左边界墙             │                                       右边界墙
    │               │                                           │
    │    左跑道     │     中间跑道      │      右跑道           │
    │   (Lane=-1)   │     (Lane=0)      │     (Lane=1)          │
    │   Y=-300      │      Y=0          │      Y=300            │
    │               │                   │                       │
────┼───────────────┼───────────────────┼───────────────────────┼────
    │       ●       │                   │                       │
    │   (玩家)      │                   │                       │
    │               │                   │                       │
────┼───────────────┼───────────────────┼───────────────────────┼────
    │               │                   │                       │
    │               │         ○         │                       │
    │               │     (障碍物)      │                       │
────┼───────────────┼───────────────────┼───────────────────────┼────
    │               │                   │                       │
    │               │                   │          ◆            │
    │               │                   │       (道具)          │
────┼───────────────┼───────────────────┼───────────────────────┼────
    
跑道宽度: 900 单位 (每条 300)
跑道长度: 10000 单位 (可扩展到无限)
```

## 世界坐标系

```
        +Z (上)
         │
         │
         │
         └────────── +X (前进方向)
        ╱
       ╱
     +Y (右)

角色初始位置: (-2000, 0, 0)
地面 Z 坐标: -100
相机 Z 偏移: +150 (相对角色)
```

## 变量关系图

```
BP_ParkourCharacter 变量:

ForwardSpeed (700)
    │
    └─► Add Movement Input → 前进速度
    
LaneDistance (300)
    │
    ├─► CurrentLane * LaneDistance → 目标 Y 位置
    │
    └─► 决定跑道间距

CurrentLane (-1, 0, 1)
    │
    ├─► -1: 左跑道 (Y = -300)
    ├─►  0: 中间跑道 (Y = 0)
    └─►  1: 右跑道 (Y = 300)

LaneChangeSpeed (10.0)
    │
    └─► FInterp To → 换道插值速度

bCanJump (True/False)
    │
    ├─► True: 允许跳跃
    └─► False: 禁止跳跃 (空中时)
```

## 性能优化策略

```
Event Tick 优化:
├─ 自动前进 (轻量级向量运算)
├─ 平滑换道 (单次插值计算)
└─ 总耗时: < 0.1ms

输入系统优化:
├─ Enhanced Input (引擎级优化)
└─ 事件驱动 (非轮询)

相机系统优化:
├─ Spring Arm Camera Lag (内置插值)
└─ 固定角度 (无额外旋转计算)

总体性能目标:
└─ 60+ FPS (16.67ms per frame)
```

## 游戏状态流程

```
┌─────────────┐
│  游戏启动    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  加载地图    │
│  TestLevel   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 生成角色     │
│ (Player     │
│  Start)     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 初始化输入   │
│ (Add        │
│  Mapping)   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 游戏循环     │◄────┐
│ - Event     │      │
│   Tick      │      │
│ - 输入处理   │      │
│ - 物理更新   │      │
│ - 渲染      │      │
└──────┬──────┘      │
       │             │
       └─────────────┘
       
(后续扩展)
       │
       ▼
┌─────────────┐
│ 碰撞检测     │
│ (障碍物)     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 游戏结束     │
└─────────────┘
```

## 文件依赖关系

```
Config/DefaultEngine.ini
├─► 定义 GameDefaultMap → Content/Maps/TestLevel
└─► 定义 GlobalDefaultGameMode → BP_ParkourGameMode

Config/DefaultInput.ini
├─► 定义 Enhanced Input 配置
├─► 映射 Jump Action → Space/W/Up
└─► 映射 MoveRight Axis → A/D/Left/Right

BP_ParkourGameMode
├─► 引用 BP_ParkourCharacter (Default Pawn)
└─► 引用 BP_ParkourPlayerController

BP_ParkourPlayerController
└─► 引用 IMC_Default (Input Mapping Context)

BP_ParkourCharacter
├─► 引用 IA_Jump (Input Action)
└─► 引用 IA_MoveRight (Input Action)

IMC_Default
├─► 引用 IA_Jump
└─► 引用 IA_MoveRight
```

## 测试关卡布局

```
TestLevel (俯视图):

                    Player Start
                        │
                        ▼
    ┌───────────────────●───────────────────┐
    │                 (-2000, 0, 0)         │
    │                                       │
    │  ═══════════════════════════════════  │ ← 地面平台
    │  ═══════════════════════════════════  │   (10000 x 900)
    │  ═══════════════════════════════════  │
    │                                       │
    └───────────────────────────────────────┘

    ☼ ← Directional Light (上方照射)
    ☁ ← Sky Atmosphere
    ✦ ← Sky Light

跑道标记:
    ═══ 黄色线 (Y=-300, 中间, +300)
```

---

## 调试视图

使用以下控制台命令查看系统运行状态：

```
stat fps          → 帧率监控
stat game         → 游戏线程性能
stat unit         → CPU/GPU/Render 耗时
show collision    → 显示碰撞体
show bounds       → 显示边界框
```

---

**提示：** 建议将此架构图打印出来或在第二显示器显示，作为开发时的参考！
