# Godot 4.X 实验性项目（AI-first）

> 技术栈：Godot 4.x + GDScript（可选 C#）

面向 **Godot 4.X** 的实验性游戏/应用项目。项目定位为 **AI Agent 优先（AI-first）**：配置、开发、扩展、维护全流程由 AI Agent 完成，人类只提出需求、验收结果。

> 🤖 **AI 优先项目**：程序面向 AI Agent 使用与开发，人类不直接编码；加功能 = 人类向 Agent 提需求 → Agent 实现。开发规范见 [DEVELOPMENT.md](./DEVELOPMENT.md)，Agent 配置引导见 [AGENTS.md](./AGENTS.md)。

---

## 🎯 项目定位

- **实验性**：不预设最终形态，以需求驱动逐步演进——想到什么玩 / 用什么，就向 Agent 提需求，由 Agent 实现。
- **AI 优先**：开发由 AI Agent 完成，人类只提需求、验收结果。全部开发约束由本仓库文档预先固化，适用于所有为本项目开发功能的 AI Agent。
- **美术与光照人类掌控**：光照方案、材质、色调、美术资源、音频（音效 / 音乐 / 混音）等创作与视觉听觉表现的**决策权归人类**——Agent 不擅自创建或修改，可提供技术建议，创意决策由人类主导。
- **多平台**：目标平台为 **Windows 桌面（Windows 10 / 11）** 与 **Android 移动端**，开发时默认考虑两平台兼容。
- **高性能**：以 60 FPS 为基准，遵循 Godot 性能实践（静态类型、对象池、渲染优化、多线程），避免性能瓶颈。
- **模块化 + glue 架构**：功能模块化、可复用，模块之间**只通过 glue（胶水）代码链接**——模块自包含、只暴露公开接口，glue 层负责接线与消息路由，禁止模块间直接耦合；**实现功能前优先复用库内已有模块 / 共享类 / 资源，禁止重复造轮子**（详见 [DEVELOPMENT.md](./DEVELOPMENT.md) §5）。

---

## ✨ 功能（需求驱动，随开发补充）

> 实验性项目，功能列表随需求持续演进。当前暂无既定功能——第一个需求由使用者提出，Agent 实现后登记于此。

## 🚀 快速开始

> 本节步骤由 **AI Agent 执行**。作为使用者，你只需要：**① 提出需求**，**② 验收 Agent 的成果**。

> ⚠️ **人工操作边界**：**Godot 的安装**与**项目骨架（`project.godot` / `scenes/` / `scripts/` 等）的创建**由使用者（人类）**手动完成**——Agent 只提供操作指引与就绪检查，**不代为执行**。骨架就绪后的配置、开发、运行由 Agent 完成。

**前置条件**：Godot 4.5+（本项目锁定 Godot 4.5+，见 [AGENTS.md](./AGENTS.md) 的环境准备章节）。

```bash
# 克隆仓库
git clone <你的仓库地址>/godot-project.git
cd godot-project

# 用 Godot 打开项目（编辑器）
godot --path . --editor

# 或直接运行主场景
godot --path .
```

> 详细的环境准备、目录结构说明与常用操作见 **[AGENTS.md](./AGENTS.md)**。

## 📁 目录结构

```
.
├── project.godot            # Godot 项目配置（4.x 文本格式）
├── export_presets.cfg       # 多平台导出预设
├── icon.svg                 # 项目图标（唯一入库的 svg，项目必需）
├── modules/                 # 功能模块（模块化 + glue 架构：每模块自包含场景/脚本/资源/数据）
├── glue/                    # 胶水层：模块接线 / 信号总线 / 服务注册表（只连接，不含业务逻辑）
├── scripts/                 # 跨模块共享的纯逻辑类（class_name，glue/基础设施）
├── assets/                  # 美术资源目录（本地提供，不入库——见「资源管理」）
├── autoload/                # 全局单例（glue 层：消息总线 / 服务注册，不堆积业务逻辑）
├── addons/                  # Godot 插件（如测试框架 GUT / gdUnit4）
├── tests/                   # 测试脚本（GUT / gdUnit4，headless 可跑）
├── docs/                    # 项目文档
├── AGENTS.md                # Agent 配置引导
├── DEVELOPMENT.md           # 开发规范
└── README.md
```

> `.godot/` 目录（Godot 导入缓存）已被 .gitignore 排除，不提交。

## 🎨 资源管理（Asset 不入库）

**GitHub 仓库只提交功能实现代码与封装**（GDScript / 场景结构 `.tscn` / 配置 `.tres` / glue / 文档），**不包含任何贴图、音频、模型、字体等美术资源**：

- 所有美术资源（贴图 / 音频 / 模型 / 字体）由**使用者（人类）本地提供**，放入 `assets/`（全局）或 `modules/<模块名>/assets/`（模块私有）；
- 资源目录与常见资源格式（png/jpg/webp/ogg/wav/glb/fbx/ttf 等）已被 .gitignore 排除，**提交到 GitHub 的代码中不会夹带任何 Asset**；
- 场景通过 `@export` 引用资源，不写死路径；资源缺失时用占位（PlaceholderTexture2D / 代码生成占位纹理）保证项目可加载；
- 资源规格 / 用途清单记录在 `docs/RESOURCES.md`（路径、用途、建议规格），供使用者按清单放置资源；
- 唯一例外：`icon.svg`（Godot 项目必需图标）随仓库提交。

> 约定详见 [DEVELOPMENT.md](./DEVELOPMENT.md) §5「资源管理」与 §8 自检清单。

## 🖥 多平台支持

目标平台为 **Windows 桌面（Windows 10 / 11）** 与 **Android 移动端**，`export_presets.cfg` 中按需启用：

| 平台 | 说明 |
|------|------|
| Windows | 主目标，x86_64；最低 **Windows 10**（11 亦支持） |
| Android | 移动端，API 23+；纹理压缩 ASTC/ETC2，低端设备用 Mobile 渲染器 |

> **关于 Windows 7**：已放弃支持。Godot 官方自 4.5 起最低要求 Windows 10（[PR #106959](https://github.com/godotengine/godot/pull/106959)），且 Windows 7 早已停止安全更新。本项目锁定 Godot 4.5+。

> 开发约束见 [DEVELOPMENT.md](./DEVELOPMENT.md) §3「跨平台约束」：不假设运行环境、路径用 `ProjectSettings` / `OS` API、资源路径大小写差异、平台专属能力门控与降级等。

## ⚡ 高性能实践（本项目基线）

- **静态类型 GDScript**：所有变量 / 参数 / 返回值声明类型，提升运行性能与可维护性。
- **避免每帧分配**：热点路径使用对象池、预分配数组，减少 GC 压力。
- **信号驱动而非轮询**：用信号 / `_process` 按需工作，避免空转。
- **渲染优化**：合并 draw call、合理使用 LOD / 遮挡剔除、控制材质与光照数量。
- **多线程**：耗时任务用 `WorkerThreadPool` / `Thread`，主线程只做轻量工作。
- **性能剖析**：用 Godot Profiler / `Performance` 单例定位瓶颈后再优化。

> 详细性能规范见 [DEVELOPMENT.md](./DEVELOPMENT.md) §6「高性能约束」。

## 🧪 测试

- 使用 GUT 或 gdUnit4 插件（位于 `addons/`），测试脚本放 `tests/`。
- 无头模式运行测试（CI / 服务器可用）：

```bash
godot --headless --path . -s tests/run_tests.gd
```

## 🤝 贡献与 AI Agent 义务

- **fork 自由**：MIT 协议，任何人可 fork 自用或扩展。
- **发现缺陷必须上报**：Agent 在使用 / 开发中发现 bug，应当主动提交 issue + 修复 PR（详见 [DEVELOPMENT.md](./DEVELOPMENT.md) §1）。
- **PR 有要求**：单一职责、不破坏现有行为、文档同步、Conventional Commits 等，详见 [DEVELOPMENT.md](./DEVELOPMENT.md) §2。

## 📄 License

MIT — 见 [LICENSE](LICENSE)。
