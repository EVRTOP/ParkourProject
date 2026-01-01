# Blueprint 可视化连接指南

本文档使用 ASCII 图表展示 Blueprint 节点的具体连接方式，便于在 UE5 编辑器中对照实现。

## 图例说明

```
┌──────────────┐
│  节点名称    │  ← Blueprint 节点
└──┬───────┬───┘
   │       │
   │       └─► 输出引脚
   └─► 执行引脚（白色箭头）
```

---

## BP_ParkourPlayerController - Event Graph

### 完整连接图

```
┌─────────────────┐
│ Event BeginPlay │
└────────┬────────┘
         │ (执行)
         ▼
┌─────────────────────────────┐
│ Get Player Controller       │
└────────┬────────────────────┘
         │ Return Value
         ▼
┌─────────────────────────────┐
│ Get Local Player            │
└────────┬────────────────────┘
         │ Return Value
         ▼
┌─────────────────────────────────────────┐
│ Get Subsystem                           │
│ (EnhancedInputLocalPlayerSubsystem)     │
└────────┬────────────────────────────────┘
         │ Return Value
         ▼
┌─────────────────────────────────────────┐
│ Add Mapping Context                     │
│                                         │
│ Mapping Context: [IMC_Default]         │
│ Priority: 0                             │
└─────────────────────────────────────────┘
```

### 详细步骤

1. **Event BeginPlay 节点**
   - 自动存在，无需添加
   - 白色执行引脚向右

2. **Get Player Controller**
   - 搜索："Get Player Controller"
   - 连接：BeginPlay 执行引脚 → Get Player Controller

3. **Get Local Player**
   - 从 Player Controller 引脚拖出
   - 搜索："Get Local Player"

4. **Get Subsystem**
   - 从 Local Player 引脚拖出
   - 搜索："Get Subsystem"
   - 在下拉菜单选择："EnhancedInputLocalPlayerSubsystem"

5. **Add Mapping Context**
   - 从 Subsystem 引脚拖出
   - 搜索："Add Mapping Context"
   - 设置参数：
     - Mapping Context: 选择 IMC_Default
     - Priority: 0

---

## BP_ParkourCharacter - Event Graph

### 1. 自动前进系统

```
┌─────────────────────────────┐
│ Event Tick                  │
│ Delta Seconds ───┐          │
└────────┬──────────┼─────────┘
         │          │
         │          └──────────────────────────┐
         │ (执行)                              │ (备用)
         ▼                                     │
┌─────────────────────────────┐               │
│ Get Actor Forward Vector    │               │
└────────┬────────────────────┘               │
         │ Return Value                        │
         ▼                                     │
┌─────────────────────────────┐               │
│ Vector * Float              │               │
│                             │               │
│ A (Vector) ◄────┘           │               │
│ B (Float)  ◄─[ForwardSpeed] │               │
└────────┬────────────────────┘               │
         │ Return Value                        │
         ▼                                     │
┌─────────────────────────────┐               │
│ Add Movement Input          │               │
│                             │               │
│ World Direction ◄─────┘     │               │
│ Scale Value: 1.0            │               │
└─────────────────────────────┘               │
```

#### 连接步骤：

1. **Event Tick**
   - 自动存在
   - Delta Seconds 引脚备用（后面用）

2. **Get Actor Forward Vector**
   - 从 Event Tick 执行引脚拖出
   - 搜索："Get Actor Forward Vector"

3. **Vector * Float (乘法)**
   - 右键空白处 → 搜索 "Multiply" 或 "*"
   - 选择 "Float * Vector" 或 "Vector * Float"
   - 连接：Forward Vector 输出 → Vector 输入
   - 拖拽 ForwardSpeed 变量到 Float 输入

4. **Add Movement Input**
   - 从乘法结果引脚拖出
   - 搜索："Add Movement Input"
   - Scale Value 保持 1.0

### 2. 平滑换道系统

```
                    ┌─────────────────────────────┐
                    │ Event Tick                  │
                    └────────┬────────────────────┘
                             │ (执行，继续上面的流程)
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        │ 计算目标Y           │ 获取当前位置        │
        ▼                    ▼                    │
┌───────────────┐    ┌──────────────────┐        │
│ CurrentLane   │    │ Get Actor        │        │
│   (Int)       │    │ Location         │        │
└───────┬───────┘    └────────┬─────────┘        │
        │                     │ Return Value      │
        ▼                     ▼                   │
┌───────────────┐    ┌──────────────────┐        │
│ Int * Float   │    │ Break Vector     │        │
│               │    │                  │        │
│ A ◄─────┘     │    │ X ─┐             │        │
│ B ◄─[Lane     │    │ Y ─┼─► CurrentY  │        │
│    Distance]  │    │ Z ─┘             │        │
└───────┬───────┘    └──────┬───────────┘        │
        │ TargetY           │                    │
        │                   │                    │
        └─────────┬─────────┘                    │
                  │                              │
                  ▼                              │
        ┌──────────────────┐                     │
        │ FInterp To       │                     │
        │ (Float)          │                     │
        │                  │                     │
        │ Current ◄─ CurrentY                    │
        │ Target  ◄─ TargetY                     │
        │ Delta   ◄─ [Delta Seconds from Tick]  │
        │ Speed   ◄─ [LaneChangeSpeed]           │
        └──────────┬───────┘                     │
                   │ Return Value (NewY)         │
                   │                             │
                   ▼                             │
        ┌──────────────────┐                     │
        │ Make Vector      │                     │
        │                  │                     │
        │ X ◄─ [原始 X]     │                     │
        │ Y ◄─ [NewY]      │                     │
        │ Z ◄─ [原始 Z]     │                     │
        └──────────┬───────┘                     │
                   │ Return Value                │
                   ▼                             │
        ┌──────────────────┐                     │
        │ Set Actor        │                     │
        │ Location         │                     │
        │                  │                     │
        │ New Location ◄───┘                     │
        │ Sweep: True                            │
        │ Teleport: False                        │
        └────────────────────────────────────────┘
```

#### 连接步骤：

1. **计算目标Y位置**
   - 拖拽 CurrentLane 变量到图表（Get）
   - 拖拽 LaneDistance 变量到图表（Get）
   - 右键 → "Multiply" (Int * Float)
   - 连接两个变量到乘法节点
   - 输出即为 TargetY

2. **获取当前位置**
   - 搜索："Get Actor Location"
   - 从返回值拖出 → "Break Vector"
   - Y 输出即为 CurrentY

3. **插值计算**
   - 搜索："FInterp To" 或 "Float Interpolate To"
   - 连接：
     - Current ← CurrentY
     - Target ← TargetY
     - Delta Time ← Event Tick 的 Delta Seconds
     - Interp Speed ← LaneChangeSpeed 变量
   - 输出即为 NewY

4. **构建新位置向量**
   - 搜索："Make Vector"
   - 连接：
     - X ← Break Vector 的 X
     - Y ← NewY（插值结果）
     - Z ← Break Vector 的 Z

5. **设置位置**
   - 搜索："Set Actor Location"
   - New Location ← Make Vector 的结果
   - Sweep: 勾选 True
   - Teleport: 不勾选（False）

### 3. 跳跃系统

```
┌──────────────────────────────┐
│ InputAction IA_Jump          │
│ (Triggered)                  │
└────────┬─────────────────────┘
         │ (执行)
         ▼
┌──────────────────────────────┐
│ Get Character Movement       │
└────────┬─────────────────────┘
         │ Return Value
         ▼
┌──────────────────────────────┐
│ Is Falling                   │
└────────┬─────────────────────┘
         │ Return Value (Bool)
         ▼
┌──────────────────────────────┐
│ Branch                       │
│ Condition ◄─────┘            │
└────┬─────────────────────┬───┘
     │ False (在地面)      │ True (在空中)
     ▼                     ▼
┌─────────────┐    ┌──────────────┐
│ Jump        │    │ (什么也不做)  │
│ (Character) │    └──────────────┘
└─────┬───────┘
      │ (执行)
      ▼
┌─────────────┐
│ Set bCanJump│
│ = False     │
└─────────────┘


(另一个独立的事件)

┌──────────────────────────────┐
│ Event Landed                 │
└────────┬─────────────────────┘
         │ (执行)
         ▼
┌──────────────────────────────┐
│ Set bCanJump                 │
│ = True                       │
└──────────────────────────────┘
```

#### 连接步骤：

1. **添加输入事件**
   - 右键 → "Enhanced Input Action"
   - 选择：IA_Jump
   - 选择事件类型：Triggered

2. **获取角色移动组件**
   - 从执行引脚拖出
   - 搜索："Get Character Movement"

3. **检查是否在空中**
   - 从 Character Movement 引脚拖出
   - 搜索："Is Falling"

4. **分支判断**
   - 搜索："Branch"
   - 连接 Is Falling 的返回值到 Condition

5. **False 分支（在地面）**
   - 搜索："Jump"（从 Character 类继承）
   - 连接到 False 执行引脚
   - 从 Jump 执行引脚拖出
   - 拖拽 bCanJump 变量（Set）
   - 取消勾选（设为 False）

6. **True 分支（在空中）**
   - 不连接任何节点（或添加注释说明）

7. **着陆事件**
   - 搜索："Event Landed"（独立事件）
   - 从执行引脚拖出
   - 拖拽 bCanJump 变量（Set）
   - 勾选（设为 True）

### 4. 左右移动输入（简化版）

```
┌──────────────────────────────┐
│ InputAction IA_MoveRight     │
│ (Triggered)                  │
└────────┬─────────────────────┘
         │ (执行)
         ▼
┌──────────────────────────────┐
│ Get Action Value             │
│ (选择: Axis1D - Float)       │
└────────┬─────────────────────┘
         │ Return Value (Float)
         ▼
┌──────────────────────────────┐
│ Clamp (Float)                │
│                              │
│ Value ◄─────┘                │
│ Min: -1.0                    │
│ Max: 1.0                     │
└────────┬─────────────────────┘
         │ Return Value
         ▼
┌──────────────────────────────┐
│ Round (Float to Int)         │
└────────┬─────────────────────┘
         │ Return Value (Int)
         ▼
┌──────────────────────────────┐
│ Set CurrentLane              │
│ New Value ◄─────┘            │
└──────────────────────────────┘
```

#### 连接步骤：

1. **添加输入事件**
   - 右键 → "Enhanced Input Action"
   - 选择：IA_MoveRight
   - 选择事件类型：Triggered

2. **获取输入值**
   - 从执行引脚拖出
   - 搜索："Get Action Value"
   - 在节点上点击下拉菜单
   - 选择类型：Float (Axis1D)

3. **限制范围**
   - 从 Action Value 输出拖出
   - 搜索："Clamp"
   - 设置 Min: -1.0, Max: 1.0

4. **转为整数**
   - 从 Clamp 输出拖出
   - 搜索："Round" 或 "Float to Int"

5. **设置当前跑道**
   - 拖拽 CurrentLane 变量（Set）
   - 连接 Round 的输出到 CurrentLane 的输入

---

## 组件设置可视化

### BP_ParkourCharacter 组件层次

```
Components (组件面板左侧)
│
├─ ● CapsuleComponent (默认根组件)
│  │
│  └─ ◆ Mesh (SkeletalMeshComponent)
│
└─ ◎ SpringArm (CameraBoom) [新添加]
   │
   └─ ◉ Camera (FollowCamera) [新添加]

● = 碰撞组件
◆ = 网格组件
◎ = 弹簧臂组件
◉ = 相机组件
```

### 添加组件步骤

1. **添加 Spring Arm**
   - 点击 "+ Add" 按钮
   - 搜索："Spring Arm"
   - 重命名为：CameraBoom
   - 自动附加到根组件（CapsuleComponent）

2. **添加 Camera**
   - 选中 SpringArm (CameraBoom)
   - 点击 "+ Add" 按钮
   - 搜索："Camera"
   - 重命名为：FollowCamera
   - 自动附加到 SpringArm

---

## 输入系统配置可视化

### IMC_Default 映射表

```
Input Mapping Context: IMC_Default
│
├─ Mappings:
│
│  [跳跃动作]
│  ├─ IA_Jump
│  │  ├─ Key: Space Bar
│  │  ├─ Key: W
│  │  └─ Key: Up Arrow
│
│  [左右移动]
│  └─ IA_MoveRight
│     ├─ Key: D          [Modifiers: Scale 1.0]
│     ├─ Key: A          [Modifiers: Scale -1.0]
│     ├─ Key: Right      [Modifiers: Scale 1.0]
│     ├─ Key: Left       [Modifiers: Scale -1.0]
│     └─ Key: Gamepad Left Thumbstick X
```

### 在 IMC_Default 中添加映射

```
1. 打开 IMC_Default
2. 在 Mappings 数组中点击 "+"
3. 选择 Input Action (IA_Jump 或 IA_MoveRight)
4. 在展开的映射下点击 "+"
5. 选择按键（Key）
6. 如果需要修改器：
   - 展开 Modifiers 数组
   - 点击 "+"
   - 选择 Scalar (for scale)
   - 设置 Scalar 值
```

---

## 调试技巧

### 添加 Print String 调试

```
┌──────────────────────┐
│ 任意节点              │
└────────┬─────────────┘
         │ (执行引脚)
         ▼
┌──────────────────────┐
│ Print String         │
│                      │
│ In String: "测试"    │
│ Duration: 2.0        │
│ Text Color: (R=1.0)  │
└──────────────────────┘
```

### 打印变量值

```
┌──────────────────────┐
│ 获取变量 (Get)       │
└────────┬─────────────┘
         │ (数据引脚)
         ▼
┌──────────────────────┐
│ To String            │
│ (或 Float to String) │
└────────┬─────────────┘
         │
         ▼
┌──────────────────────┐
│ Print String         │
│ In String ◄─────┘    │
└──────────────────────┘
```

---

## 常用快捷键

| 操作 | 快捷键 |
|------|--------|
| 编译 Blueprint | F7 |
| 保存 | Ctrl + S |
| 全部保存 | Ctrl + Shift + S |
| 查找 | Ctrl + F |
| 添加注释框 | C |
| 对齐节点 | Q |
| 复制节点 | Ctrl + C |
| 粘贴节点 | Ctrl + V |
| 删除节点 | Delete |
| 撤销 | Ctrl + Z |
| 重做 | Ctrl + Y |

---

## 节点颜色说明

- **白色箭头**：执行流程线
- **绿色线**：Boolean 数据
- **蓝色线**：Object Reference 数据
- **黄色线**：Float 数据
- **红色线**：Integer 数据
- **粉色线**：Vector 数据
- **紫色线**：Transform 数据

---

**提示：** 在 UE5 编辑器中，可以按住 Ctrl 键并单击节点快速跳转到定义！
