# 故障排查指南

本文档帮助你快速诊断和解决跑酷游戏开发中遇到的常见问题。

## 目录

1. [角色移动问题](#角色移动问题)
2. [输入系统问题](#输入系统问题)
3. [相机问题](#相机问题)
4. [跳跃问题](#跳跃问题)
5. [性能问题](#性能问题)
6. [编辑器问题](#编辑器问题)

---

## 角色移动问题

### 问题：角色完全不移动

**可能原因及解决方案：**

1. **ForwardSpeed 为 0**
   - 解决：在 BP_ParkourCharacter 的 Class Defaults 中检查 ForwardSpeed 是否为 700

2. **CharacterMovement 组件的 Max Walk Speed 为 0**
   - 解决：选中 CharacterMovement 组件，设置 Max Walk Speed = 700

3. **Event Tick 逻辑未正确连接**
   - 解决：检查 Event Tick -> Get Forward Vector -> Multiply -> Add Movement Input 的完整连接

4. **Add Movement Input 的 Scale Value 为 0**
   - 解决：确保 Scale Value 设置为 1.0

**调试步骤：**
```
1. 在 Event Tick 中添加 Print String 节点
2. 打印 ForwardSpeed 变量值
3. 打印 Forward Vector 的值
4. 运行游戏，查看输出日志
```

### 问题：角色移动速度不正确

**可能原因：**
- ForwardSpeed 值设置不当
- CharacterMovement 的 Max Walk Speed 限制了速度

**解决方案：**
1. 调整 ForwardSpeed 变量（建议范围 600-800）
2. 确保 Max Walk Speed >= ForwardSpeed
3. 检查 Add Movement Input 的 Scale Value 是否为 1.0

### 问题：角色向错误方向移动

**可能原因：**
- 使用了错误的向量（如 Right Vector 而非 Forward Vector）
- Actor 的 Rotation 设置错误

**解决方案：**
1. 确认使用 "Get Actor Forward Vector"
2. 检查 Player Start 的 Rotation 是否为 (0, 0, 0)
3. 确保角色面向 +X 方向

### 问题：左右移动不起作用

**可能原因：**
1. CurrentLane 变量未更新
2. Set Actor Location 逻辑有问题
3. LaneDistance 为 0

**解决方案：**
1. 在 IA_MoveRight 输入事件中添加 Print String，打印 CurrentLane 值
2. 检查 FInterp To 节点是否正确连接
3. 确认 LaneDistance = 300
4. 确保 Set Actor Location 的 Sweep 为 True

### 问题：左右移动太慢或太快

**解决方案：**
- 太慢：增加 LaneChangeSpeed 变量（如改为 15.0）
- 太快：降低 LaneChangeSpeed 变量（如改为 5.0）
- 或者调整 LaneDistance（跑道间距）

### 问题：角色左右移动时卡顿

**可能原因：**
- LaneChangeSpeed 太小导致插值太慢
- 每帧计算量过大

**解决方案：**
1. 增加 LaneChangeSpeed 到 10-15
2. 确保使用 FInterp To 而非直接 Set Location
3. 检查 Event Tick 中没有复杂计算

---

## 输入系统问题

### 问题：所有输入都无响应

**可能原因及解决方案：**

1. **Enhanced Input 插件未启用**
   - 解决：Edit -> Plugins -> 搜索 "Enhanced Input" -> 确保已勾选 Enabled

2. **Input Mapping Context 未添加**
   - 解决：检查 BP_ParkourPlayerController 的 Event BeginPlay
   - 确认有 "Add Mapping Context" 节点
   - 确认 Mapping Context 选择了 IMC_Default

3. **PlayerController 未正确设置**
   - 解决：
     - 打开 BP_ParkourGameMode
     - 确认 Player Controller Class = BP_ParkourPlayerController

4. **Input Actions 未创建**
   - 解决：确认 Content/Input 文件夹中有 IA_Jump 和 IA_MoveRight

**调试步骤：**
```
1. 在 PlayerController 的 BeginPlay 中添加 Print String "Controller Started"
2. 在 Add Mapping Context 后添加 Print String "Mapping Added"
3. 运行游戏，检查是否打印
4. 如果没打印，说明 PlayerController 未生效
```

### 问题：跳跃键无响应

**可能原因：**
1. IA_Jump 未在 IMC_Default 中映射
2. InputAction 事件未在 Character 中添加

**解决方案：**
1. 打开 IMC_Default，确认 IA_Jump 已映射到 Space/W/Up
2. 在 BP_ParkourCharacter 的 Event Graph 中：
   - 右键 -> "Enhanced Input Action"
   - 选择 IA_Jump
   - 选择 Triggered 事件
3. 添加 Print String "Jump Pressed" 测试

### 问题：左右移动键无响应

**解决方案：**
同跳跃键，检查：
1. IA_MoveRight 是否在 IMC_Default 中映射
2. A/D 键的 Scale 是否正确（A=-1.0, D=1.0）
3. BP_ParkourCharacter 中是否有 IA_MoveRight 的事件

### 问题：按键有延迟

**可能原因：**
- 使用了 "Started" 或 "Completed" 事件而非 "Triggered"

**解决方案：**
- 确保 Input Action 事件使用 "Triggered"
- Triggered 在按键按住期间持续触发
- Started 仅在首次按下触发

---

## 相机问题

### 问题：看不到角色

**可能原因：**
1. 相机位置错误
2. Spring Arm 长度太短
3. 相机朝向错误

**解决方案：**
1. 检查 Spring Arm（CameraBoom）设置：
   - Target Arm Length: 600.0
   - Socket Offset: (0, 0, 150)
   - Rotation: (-30, 0, 0)
2. 确认 Camera 附加到 Spring Arm 上
3. 在 Viewport 中按 F 聚焦到角色

### 问题：相机抖动或卡顿

**可能原因：**
1. Camera Lag 未启用
2. Camera Lag Speed 太高
3. Set Actor Location 未使用 Sweep

**解决方案：**
1. 在 Spring Arm 组件中：
   - Enable Camera Lag: True
   - Camera Lag Speed: 10.0
2. 在 Set Actor Location 节点中：
   - Sweep: True
   - Teleport: False

### 问题：相机太近/太远

**解决方案：**
- 太近：增加 Spring Arm 的 Target Arm Length（如 800）
- 太远：减少 Target Arm Length（如 400）

### 问题：相机角度不合适

**解决方案：**
1. 调整 Spring Arm 的 Rotation
   - X (Pitch): 控制俯仰角（-30 = 向下看 30 度）
   - Y (Yaw): 控制左右角度
   - Z (Roll): 控制翻滚（通常为 0）
2. 推荐俯视角：(-25, 0, 0) 到 (-40, 0, 0)

### 问题：相机跟随不流畅

**解决方案：**
1. 增加 Camera Lag Speed（如 15.0）
2. 减少 Camera Lag Max Distance（如 100.0）
3. 确保 Use Pawn Control Rotation = False

---

## 跳跃问题

### 问题：无法跳跃

**可能原因及解决方案：**

1. **Jump Z Velocity 为 0**
   - 解决：在 CharacterMovement 组件中设置 Jump Z Velocity = 600

2. **Jump 节点未调用**
   - 解决：确保 IA_Jump 的 Triggered 事件连接到 Jump 节点

3. **角色不在地面**
   - 解决：检查角色是否掉出地图
   - 确保地面有碰撞

4. **Is Falling 检测失败**
   - 解决：暂时移除 Is Falling 检测，直接调用 Jump 测试

**调试步骤：**
```
1. 简化跳跃逻辑：
   IA_Jump (Triggered) -> Jump
   （移除 Is Falling 检测）

2. 添加 Print String "Jump Called"
3. 运行游戏测试
4. 如果能跳，说明是 Is Falling 逻辑问题
```

### 问题：可以连续跳跃（二段跳）

**解决方案：**
1. 添加 Is Falling 检测：
```
IA_Jump (Triggered)
  -> Get Character Movement
  -> Is Falling
  -> Branch
     False -> Jump
     True -> Do Nothing
```

2. 或使用 bCanJump 变量：
```
IA_Jump (Triggered)
  -> Branch (bCanJump == True)
     True ->
       Jump
       Set bCanJump = False

Event Landed
  -> Set bCanJump = True
```

### 问题：跳跃太高

**解决方案：**
- 减少 Jump Z Velocity（如改为 400-500）
- 或增加 Gravity Scale（如改为 2.0）

### 问题：跳跃太低

**解决方案：**
- 增加 Jump Z Velocity（如改为 700-800）
- 或减少 Gravity Scale（如改为 1.0）

### 问题：跳跃后下落太慢

**解决方案：**
- 增加 CharacterMovement 的 Gravity Scale（推荐 1.5-2.0）

### 问题：跳跃时无法控制方向

**解决方案：**
- 增加 CharacterMovement 的 Air Control（如 0.2-0.5）

---

## 性能问题

### 问题：帧率低于 60 FPS

**诊断步骤：**
1. 按 `~` 打开控制台
2. 输入 `stat fps` 查看帧率
3. 输入 `stat unit` 查看各系统耗时

**常见原因及解决方案：**

1. **Event Tick 中计算过多**
   - 解决：将非必要计算移出 Tick
   - 使用 Timer 代替频繁的 Tick 计算

2. **场景过于复杂**
   - 解决：减少场景物体数量
   - 使用 LOD（细节层次）

3. **光照计算过重**
   - 解决：Build Lighting (Build -> Build Lighting)
   - 使用 Static Lighting

4. **调试信息过多**
   - 解决：移除或注释掉 Print String 节点

### 问题：编辑器卡顿

**解决方案：**
1. 降低编辑器预览质量：
   - Viewport 设置 -> View Mode -> Unlit
2. 关闭实时预览：
   - Viewport 右上角关闭 "Realtime"
3. 清理 DerivedDataCache：
   - 关闭编辑器
   - 删除项目文件夹中的 DerivedDataCache 文件夹

---

## 编辑器问题

### 问题：无法打开 .uproject 文件

**可能原因：**
1. Unreal Engine 版本不匹配
2. 项目文件损坏

**解决方案：**
1. 确保安装了 UE 5.3
2. 右键 .uproject -> Switch Unreal Engine Version
3. 如果失败，手动编辑 .uproject：
```json
{
  "EngineAssociation": "5.3",
  ...
}
```

### 问题：找不到 Blueprint 类

**解决方案：**
1. 确保 Blueprint 文件在正确的文件夹中
2. File -> Refresh All Nodes
3. 重启编辑器

### 问题：编译错误

**常见错误及解决方案：**

1. **"Cannot find InputAction"**
   - 确保 Input Action 已创建在 Content/Input 文件夹

2. **"Mapping Context not found"**
   - 确保 IMC_Default 已创建
   - 检查 PlayerController 中的引用路径

3. **"Class not found"**
   - 重启编辑器
   - File -> Refresh All Nodes

### 问题：Play 按钮无响应

**解决方案：**
1. 检查 Output Log（Window -> Developer Tools -> Output Log）
2. 确保没有编译错误
3. 尝试 PIE（Play In Editor）的不同模式：
   - Selected Viewport
   - New Editor Window
   - Standalone Game

### 问题：修改后不生效

**解决方案：**
1. 点击 Blueprint 的 Compile 按钮
2. File -> Save All
3. 重启 Play 会话

---

## 快速诊断清单

遇到问题时，按此顺序检查：

### ✅ 基础检查
- [ ] UE 5.3 已正确安装
- [ ] Enhanced Input 插件已启用
- [ ] 所有 Blueprint 已编译（绿色对勾）
- [ ] 所有文件已保存

### ✅ 输入系统检查
- [ ] IA_Jump 和 IA_MoveRight 已创建
- [ ] IMC_Default 已创建并配置
- [ ] PlayerController 的 BeginPlay 添加了 Mapping Context
- [ ] GameMode 设置了正确的 PlayerController 类

### ✅ 角色检查
- [ ] BP_ParkourCharacter 存在
- [ ] CharacterMovement 组件配置正确
- [ ] Spring Arm 和 Camera 组件已添加
- [ ] Event Tick 逻辑已实现
- [ ] Input Action 事件已添加

### ✅ 关卡检查
- [ ] TestLevel 已创建
- [ ] 地面平台存在且有碰撞
- [ ] Player Start 已放置
- [ ] 光照已添加
- [ ] World Settings 中 GameMode 已设置

---

## 获取帮助

如果以上方法都无法解决问题：

1. **查看 Output Log：**
   - Window -> Developer Tools -> Output Log
   - 查找红色错误信息

2. **检查 Blueprint 编译错误：**
   - 打开 Blueprint
   - 查看底部的 Compiler Results

3. **重启编辑器：**
   - 很多问题可以通过重启解决

4. **提交 Issue：**
   - 访问 GitHub 项目页
   - 提供详细的错误信息和截图

---

**提示：** 保持耐心，Blueprint 开发需要细心检查每个节点的连接和参数。建议使用 Print String 节点频繁打印变量值来调试！
