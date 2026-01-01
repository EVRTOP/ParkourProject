# 实现验收清单

在完成所有 Blueprint 创建后，使用此清单验证功能是否正确实现。

## 第一阶段：基础设置验证

### ✅ 项目配置
- [ ] 在 Edit -> Plugins 中确认 Enhanced Input 插件已启用
- [ ] 在 Edit -> Project Settings -> Maps & Modes 中确认：
  - [ ] Default GameMode 设置为 BP_ParkourGameMode
  - [ ] Editor Startup Map 设置为 TestLevel
  - [ ] Game Default Map 设置为 TestLevel

### ✅ 文件夹结构
- [ ] Content/Blueprints 文件夹已创建
- [ ] Content/Maps 文件夹已创建
- [ ] Content/Input 文件夹已创建

### ✅ 输入系统资产
- [ ] Content/Input/IA_Jump 已创建（Input Action, Digital）
- [ ] Content/Input/IA_MoveRight 已创建（Input Action, Axis1D）
- [ ] Content/Input/IMC_Default 已创建（Input Mapping Context）
- [ ] IMC_Default 中正确映射了所有按键

---

## 第二阶段：Blueprint 创建验证

### ✅ BP_ParkourGameMode
- [ ] 文件路径：Content/Blueprints/BP_ParkourGameMode
- [ ] 父类：Game Mode Base
- [ ] Default Pawn Class 设置为 BP_ParkourCharacter
- [ ] Player Controller Class 设置为 BP_ParkourPlayerController
- [ ] 已编译无错误（绿色对勾）

### ✅ BP_ParkourPlayerController
- [ ] 文件路径：Content/Blueprints/BP_ParkourPlayerController
- [ ] 父类：Player Controller
- [ ] Event BeginPlay 中有 "Get Subsystem" 节点
- [ ] "Add Mapping Context" 节点已连接
- [ ] Mapping Context 设置为 IMC_Default
- [ ] Priority 设置为 0
- [ ] 已编译无错误（绿色对勾）

### ✅ BP_ParkourCharacter - Components
- [ ] 文件路径：Content/Blueprints/BP_ParkourCharacter
- [ ] 父类：Character
- [ ] CapsuleComponent 存在且配置正确
  - [ ] Capsule Half Height = 88
  - [ ] Capsule Radius = 42
- [ ] Mesh 组件存在
  - [ ] Location = (0, 0, -88)
  - [ ] Rotation = (0, 0, -90)
  - [ ] （可选）添加了临时网格体
- [ ] SpringArm (CameraBoom) 组件已添加
  - [ ] Target Arm Length = 600
  - [ ] Socket Offset = (0, 0, 150)
  - [ ] Rotation = (-30, 0, 0)
  - [ ] Use Pawn Control Rotation = False
  - [ ] Enable Camera Lag = True
  - [ ] Camera Lag Speed = 10
- [ ] Camera (FollowCamera) 组件已添加
  - [ ] 附加到 SpringArm
- [ ] CharacterMovement 组件配置正确
  - [ ] Max Walk Speed = 700
  - [ ] Jump Z Velocity = 600
  - [ ] Air Control = 0.2
  - [ ] Gravity Scale = 1.5
  - [ ] Ground Friction = 8.0

### ✅ BP_ParkourCharacter - Variables
- [ ] ForwardSpeed (Float) = 700, Instance Editable
- [ ] LaneDistance (Float) = 300, Instance Editable
- [ ] CurrentLane (Integer) = 0
- [ ] LaneChangeSpeed (Float) = 10, Instance Editable
- [ ] bCanJump (Boolean) = True

### ✅ BP_ParkourCharacter - Event Graph
- [ ] Event BeginPlay 逻辑已实现
  - [ ] 初始化所有变量
- [ ] Event Tick 逻辑已实现
  - [ ] 自动前进功能
    - [ ] Get Actor Forward Vector 节点存在
    - [ ] Vector * Float 节点连接正确
    - [ ] Add Movement Input 节点连接正确
  - [ ] 平滑换道功能
    - [ ] CurrentLane * LaneDistance 计算目标位置
    - [ ] Get Actor Location 获取当前位置
    - [ ] Break Vector 分解坐标
    - [ ] FInterp To 插值计算
    - [ ] Make Vector 构建新位置
    - [ ] Set Actor Location 设置位置（Sweep = True）
- [ ] IA_Jump (Triggered) 输入事件已添加
  - [ ] Get Character Movement 节点存在
  - [ ] Is Falling 节点存在
  - [ ] Branch 节点正确连接
  - [ ] False 分支调用 Jump
  - [ ] False 分支设置 bCanJump = False
- [ ] Event Landed 事件已添加
  - [ ] 设置 bCanJump = True
- [ ] IA_MoveRight (Triggered) 输入事件已添加
  - [ ] Get Action Value 节点存在
  - [ ] 换道逻辑正确实现（Clamp + Round 或多分支）
  - [ ] Set CurrentLane 节点连接正确
- [ ] 已编译无错误（绿色对勾）

---

## 第三阶段：测试关卡验证

### ✅ TestLevel 基础设置
- [ ] 文件路径：Content/Maps/TestLevel
- [ ] 关卡类型：Empty Level（或已清理）
- [ ] Window -> World Settings 中 GameMode Override = BP_ParkourGameMode

### ✅ 环境光照
- [ ] Sky Light 已添加到场景
  - [ ] Intensity = 1.0
- [ ] Directional Light 已添加到场景
  - [ ] Rotation = (-30, 0, 0) 或类似
  - [ ] Intensity = 10.0
- [ ] Sky Atmosphere 已添加到场景
- [ ] Post Process Volume 已添加（可选）
  - [ ] Infinite Extent = True

### ✅ 跑道几何体
- [ ] 主地面平台已创建
  - [ ] Actor: Cube
  - [ ] Location ≈ (0, 0, -100)
  - [ ] Scale ≈ (100, 9, 1) 或类似
  - [ ] 有可见的材质
- [ ] 左右边界墙已创建（可选）
  - [ ] 左墙：Location ≈ (0, -450, 0)
  - [ ] 右墙：Location ≈ (0, 450, 0)
- [ ] 跑道标记线已创建（可选，帮助视觉引导）
  - [ ] 中间线：Y = 0
  - [ ] 左跑道线：Y = -300
  - [ ] 右跑道线：Y = 300

### ✅ 玩家起始点
- [ ] Player Start 已添加到场景
  - [ ] Location ≈ (-2000, 0, 0) 或在跑道起点
  - [ ] Rotation = (0, 0, 0) 朝向 +X 方向

### ✅ 碰撞设置
- [ ] 所有地面物体的碰撞已启用
  - [ ] 选中地面 Cube
  - [ ] Details -> Collision -> Collision Presets = BlockAll 或 WorldStatic

---

## 第四阶段：功能测试

### ✅ 启动测试
- [ ] 点击 Play 按钮（或 Alt+P）游戏能正常启动
- [ ] 没有出现红色错误提示
- [ ] Output Log 中没有严重错误
- [ ] 角色在 Player Start 位置正确生成

### ✅ 自动前进测试
- [ ] 角色启动后立即开始向前移动
- [ ] 移动方向正确（沿 +X 轴）
- [ ] 移动速度合理（看起来像在跑步）
- [ ] 移动流畅，无卡顿

### ✅ 相机测试
- [ ] 相机位于角色后上方
- [ ] 相机角度为俯视角
- [ ] 能清楚看到角色和前方跑道
- [ ] 相机跟随流畅，无明显抖动或卡顿
- [ ] 相机不会穿过地面或其他物体

### ✅ 左右移动测试
- [ ] 按 A 键，角色向左移动
- [ ] 按 D 键，角色向右移动
- [ ] 按 Left 方向键，角色向左移动
- [ ] 按 Right 方向键，角色向右移动
- [ ] 左右移动是平滑过渡，不是瞬移
- [ ] 角色能在三条跑道之间正确切换
- [ ] 角色最左只到左跑道（Y = -300）
- [ ] 角色最右只到右跑道（Y = 300）
- [ ] 在中间跑道时，左右都能移动

### ✅ 跳跃测试
- [ ] 按 Space 键能跳跃
- [ ] 按 W 键能跳跃
- [ ] 按 Up 方向键能跳跃
- [ ] 跳跃高度合理（不太高不太低）
- [ ] 在空中无法再次跳跃（防二段跳）
- [ ] 落地后可以再次跳跃
- [ ] 在空中可以继续控制左右移动
- [ ] 跳跃轨迹自然（抛物线）

### ✅ 碰撞测试
- [ ] 角色站在地面上，不会掉落
- [ ] 角色与地面有正确的碰撞
- [ ] 角色不会穿过墙壁（如果有）
- [ ] 跳跃后能正常着陆

### ✅ 性能测试
- [ ] 按 `~` 打开控制台
- [ ] 输入 `stat fps`
- [ ] 帧率显示为 60+ FPS
- [ ] 没有明显掉帧
- [ ] 输入 `stat unit`
- [ ] Game 线程耗时 < 10ms
- [ ] Render 线程耗时 < 16ms

---

## 第五阶段：高级验证

### ✅ 边界情况测试
- [ ] 连续快速按左右键，角色正常响应
- [ ] 在换道过程中按相反方向，能正确改变方向
- [ ] 在跳跃过程中换道，功能正常
- [ ] 长时间运行（2-3分钟），无崩溃或异常

### ✅ 输入组合测试
- [ ] 同时按 A 和 D，角色保持当前跑道
- [ ] 跳跃同时左右移动，两个动作都能执行
- [ ] 快速切换跑道（左-右-左），响应及时

### ✅ 视觉效果测试
- [ ] 角色移动动画流畅（如有动画）
- [ ] 相机视角舒适，不会眩晕
- [ ] 跑道标记清晰可见
- [ ] 光照效果良好，能清楚看到场景

### ✅ 手柄测试（可选）
- [ ] 连接游戏手柄
- [ ] 左摇杆左右能控制换道
- [ ] A 键（Xbox）或 X 键（PS）能跳跃

---

## 第六阶段：文档验证

### ✅ 文档完整性
- [ ] README.md 存在且内容准确
- [ ] IMPLEMENTATION_GUIDE.md 存在且步骤清晰
- [ ] QUICK_REFERENCE.md 存在且信息完整
- [ ] TROUBLESHOOTING.md 存在且问题覆盖全面
- [ ] ARCHITECTURE.md 存在且图表清晰

### ✅ 配置文件
- [ ] Config/DefaultInput.ini 配置正确
- [ ] Config/DefaultEngine.ini 配置正确
- [ ] .gitignore 正确忽略了构建文件

---

## 常见错误检查

### 🔴 如果角色不移动
1. [ ] 检查 ForwardSpeed 是否 > 0
2. [ ] 检查 CharacterMovement.MaxWalkSpeed 是否 > 0
3. [ ] 检查 Event Tick 中的自动前进逻辑
4. [ ] 检查 Add Movement Input 节点连接

### 🔴 如果输入无响应
1. [ ] 检查 Enhanced Input 插件是否启用
2. [ ] 检查 PlayerController 是否添加了 Mapping Context
3. [ ] 检查 GameMode 是否设置了正确的 PlayerController
4. [ ] 检查 Input Actions 是否在 Character 中有响应事件

### 🔴 如果无法跳跃
1. [ ] 检查 CharacterMovement.JumpZVelocity 是否 > 0
2. [ ] 检查 Jump 节点是否被调用
3. [ ] 检查角色是否在地面上
4. [ ] 检查 Is Falling 逻辑是否正确

### 🔴 如果相机有问题
1. [ ] 检查 Spring Arm 和 Camera 组件是否存在
2. [ ] 检查 Spring Arm 的 Target Arm Length
3. [ ] 检查 Enable Camera Lag 是否为 True
4. [ ] 检查 Camera 是否附加到 Spring Arm

---

## 最终验收标准

### ✅ 核心功能（必须全部通过）
- [ ] ✅ 角色能够自动持续向前奔跑
- [ ] ✅ 玩家可以通过键盘 A/D 键控制角色左右移动
- [ ] ✅ 跳跃功能正常工作，不能在空中连续跳跃
- [ ] ✅ 相机视角舒适，跟随流畅，无明显抖动或 bug
- [ ] ✅ 在 UE5 编辑器中点击 Play 可以直接进行游戏测试

### ✅ 性能要求（必须达到）
- [ ] ✅ 帧率稳定在 60 FPS 以上
- [ ] ✅ 没有内存泄漏或性能警告
- [ ] ✅ 游戏运行流畅，无明显卡顿

### ✅ 代码质量（推荐达到）
- [ ] ✅ Blueprint 结构清晰，逻辑易懂
- [ ] ✅ 变量命名规范（PascalCase）
- [ ] ✅ 复杂逻辑有注释说明
- [ ] ✅ 所有 Blueprint 编译无警告

---

## 通过标准

**如果以上所有必须项（标记为"必须"的）都通过，则第一阶段开发完成！**

恭喜！你已经完成了跑酷游戏的核心基础系统。现在可以开始第二阶段的开发：
- 跑道模块化生成系统
- 障碍物系统
- 收集道具与计分系统

---

**测试日期：** _____________

**测试人员：** _____________

**测试结果：** ☐ 通过  ☐ 需要修复

**备注：**

_______________________________________________________________

_______________________________________________________________

_______________________________________________________________
