# 🚀 快速开始指南

欢迎来到跑酷游戏项目！这是一个 5 分钟快速开始指南。

## 📖 你现在的位置

你已经克隆了这个项目，接下来只需要 3 个步骤就能开始开发：

---

## 第 1 步：了解项目结构（1 分钟）

```
ParkourProject/
├── 📄 Config/                 ✅ 已配置好的引擎和输入设置
├── 📁 Content/                ⏳ 你需要在这里创建 Blueprint
├── 📚 Documentation/           ✅ 8 个详细文档帮助你
└── 🎮 ParkourProject.uproject ✅ 双击这个文件打开项目
```

---

## 第 2 步：打开项目（1 分钟）

1. **双击** `ParkourProject.uproject`
2. 如果提示选择 UE 版本，选择 **5.3** 或更高
3. 等待编辑器加载（首次可能需要几分钟）
4. ✅ 编辑器打开成功！

---

## 第 3 步：开始创建（3 分钟阅读）

### 🎯 你的任务

按照这个顺序创建 Blueprint 资产：

```
1️⃣ Content/Input/          ← 输入系统（3 个文件）
   ├── IA_Jump.uasset
   ├── IA_MoveRight.uasset
   └── IMC_Default.uasset

2️⃣ Content/Blueprints/     ← 游戏逻辑（3 个文件）
   ├── BP_ParkourGameMode.uasset
   ├── BP_ParkourPlayerController.uasset
   └── BP_ParkourCharacter.uasset

3️⃣ Content/Maps/           ← 测试关卡（1 个文件）
   └── TestLevel.umap
```

### 📚 使用哪个文档？

根据你的需求选择：

| 需求 | 使用文档 |
|------|----------|
| **我要按步骤实现** | 👉 [IMPLEMENTATION_GUIDE.md](IMPLEMENTATION_GUIDE.md) |
| **我需要快速查参数** | 👉 [QUICK_REFERENCE.md](QUICK_REFERENCE.md) |
| **我要看连接图** | 👉 [VISUAL_GUIDE.md](VISUAL_GUIDE.md) |
| **我遇到问题了** | 👉 [TROUBLESHOOTING.md](TROUBLESHOOTING.md) |
| **我要验收测试** | 👉 [CHECKLIST.md](CHECKLIST.md) |
| **我想了解架构** | 👉 [ARCHITECTURE.md](ARCHITECTURE.md) |

### ⚡ 推荐工作流

```
┌─────────────────────────────────────┐
│ 第一次使用？                         │
│ 建议按这个顺序：                     │
│                                     │
│ 1. 快速阅读 README.md (10分钟)      │
│ 2. 打开 IMPLEMENTATION_GUIDE.md     │
│ 3. 准备好 QUICK_REFERENCE.md 查阅   │
│ 4. 开始在 UE5 中创建                │
│ 5. 遇到问题查 TROUBLESHOOTING.md    │
│ 6. 完成后用 CHECKLIST.md 验收       │
└─────────────────────────────────────┘
```

---

## 🎮 预期结果

完成所有 Blueprint 创建后，你将得到：

- ✅ 角色自动向前奔跑
- ✅ 按 A/D 键左右切换跑道
- ✅ 按 Space 键跳跃
- ✅ 相机平滑跟随角色
- ✅ 帧率稳定在 60+ FPS

**测试方法：** 在 UE5 编辑器中按 `Alt+P` 或点击 Play 按钮

---

## 📊 时间估算

| 任务 | 时间 | 难度 |
|------|------|------|
| 创建输入系统 | 30 分钟 | ⭐⭐ |
| 创建 GameMode & Controller | 15 分钟 | ⭐ |
| 创建 Character Blueprint | 90-120 分钟 | ⭐⭐⭐⭐ |
| 创建测试关卡 | 30-45 分钟 | ⭐⭐ |
| 测试和调试 | 30-60 分钟 | ⭐⭐⭐ |

**总计：约 3-4 小时**（适合周末完成）

---

## 🆘 需要帮助？

### 常见问题

**Q: 我是 UE5 新手，能完成吗？**
A: 能！文档提供了逐步指导，包括截图位置的文字说明。

**Q: 我不会 Blueprint？**
A: 没关系！[VISUAL_GUIDE.md](VISUAL_GUIDE.md) 有详细的节点连接图。

**Q: 卡住了怎么办？**
A: 查看 [TROUBLESHOOTING.md](TROUBLESHOOTING.md)，里面有常见问题的解决方案。

**Q: 想了解设计原理？**
A: 阅读 [ARCHITECTURE.md](ARCHITECTURE.md)，有完整的系统架构说明。

---

## 💡 专业提示

1. **准备双显示器或打印文档** - 一边看文档，一边操作 UE5
2. **先看再做** - 先通读一遍步骤，再开始创建
3. **频繁保存** - 每完成一个部分就 `Ctrl+S`
4. **使用 Print String 调试** - 不确定时打印变量值
5. **编译后再测试** - 每次修改后点 Compile（F7）

---

## 🎯 第一个里程碑

完成这个阶段后，你将拥有：

✅ **完整的角色控制系统**
✅ **流畅的游戏体验**
✅ **可扩展的代码架构**
✅ **为后续开发打好基础**

然后你可以继续实现：
- 🏃 无限跑道生成
- 🚧 障碍物系统
- 💰 道具收集
- 🎨 UI 界面
- 🎵 音效特效

---

## 📝 最后检查

开始之前，确保：

- [ ] 已安装 Unreal Engine 5.3+
- [ ] 系统至少有 8GB RAM
- [ ] 显卡支持 DirectX 12
- [ ] 硬盘有 20GB 可用空间
- [ ] 已阅读 README.md
- [ ] 准备好文档（打印或第二显示器）
- [ ] 有 3-4 小时不被打扰的时间 ☕

---

## 🚀 现在开始！

1. **双击** `ParkourProject.uproject` 打开项目
2. **打开** [IMPLEMENTATION_GUIDE.md](IMPLEMENTATION_GUIDE.md)
3. **开始创建** 第一个 Input Action！

---

## 📞 联系和支持

- 🐛 发现 Bug？→ [GitHub Issues](https://github.com/EVRTOP/ParkourProject/issues)
- 💬 有疑问？→ [GitHub Discussions](https://github.com/EVRTOP/ParkourProject/discussions)
- ⭐ 觉得有用？→ 给项目一个 Star！

---

**准备好了吗？Let's build something amazing! 🎮✨**

---

<details>
<summary>📚 完整文档列表（点击展开）</summary>

1. **README.md** - 项目主页，概览和 FAQ
2. **GETTING_STARTED.md** - 本文件，快速开始
3. **IMPLEMENTATION_GUIDE.md** - 详细实现步骤（最重要）
4. **QUICK_REFERENCE.md** - 参数和节点速查表
5. **VISUAL_GUIDE.md** - 可视化连接指南
6. **ARCHITECTURE.md** - 系统架构设计
7. **TROUBLESHOOTING.md** - 问题排查指南
8. **CHECKLIST.md** - 验收测试清单
9. **PROJECT_SUMMARY.md** - 项目完成总结

</details>

---

_最后更新：2026-01-01 | 版本：1.0.0_
