# Godot 4.X 实验性项目 — Agent 配置引导

> 本文档面向 AI Agent：如何准备环境、运行项目、开发与验证。建议 agent 在本仓库目录内运行（Godot 可执行文件路径可自动探测或显式设置）。

> 项目定位：实验性、AI 优先、多平台、高性能。开发规范见 [DEVELOPMENT.md](./DEVELOPMENT.md)，项目概览见 [README.md](./README.md)。

## 1. 环境准备（Agent 必读）

> ⚠️ **人工操作边界**：**Godot 的安装**与**项目骨架（`project.godot` / `scenes/` / `scripts/` 等）的创建**由使用者（人类）**手动完成**。Agent **不代为安装 Godot、不代为创建骨架**，只提供操作指引与就绪检查（如 `godot --version` 验证安装结果）。
>
> 🤝 **工程独立性保证**：Agent 的工具链与 Godot 工程文件物理隔离、互不依赖，工程始终可由人类直接在 Godot 中人工接管（见 §4.5）。Agent 的自动化 / 工具 / 专属文件只放 `agent/` 目录，不混入工程；工程不引用 `agent/` 内任何内容。

### 1.1 安装 Godot 4.5+（人工操作）

- 由使用者（人类）从 [godotengine.org/download](https://godotengine.org/download) 下载并安装 **Godot 4.5+ 标准版**（本项目锁定 4.5+；含 GDScript，如项目启用 C# 则需 .NET 版）。
- 安装后将 `godot` 命令加入 PATH（Windows 可将 `Godot_v4.5+-stable_win64.exe` 加入 PATH，或直接用完整路径）。
- Agent 负责的就绪检查：

```bash
godot --version
```

> 若 `godot` 不在 PATH，可设置环境变量 `GODOT_BIN` 指向 Godot 可执行文件路径（由使用者确认路径后 Agent 使用），后续所有命令用它代替 `godot`。

### 1.2 获取代码（Agent 执行）

```bash
git clone <你的仓库地址>/godot-project.git
cd godot-project
```

> 建议 fork 到自己的 GitHub 账号再克隆，便于保留 AI Agent 实现的变更（详见 README）。

### 1.3 创建项目骨架（人工操作）

- 由使用者（人类）在 Godot 编辑器中创建项目骨架（`project.godot`、`icon.svg`、主场景、`scenes/` / `scripts/` / `assets/` 等目录），或用 `godot --path . --editor` 打开已有目录初始化。
- Agent **不代为创建骨架**；骨架就绪后，Agent 才在其上进行开发。
- Agent 负责的就绪检查：确认 `project.godot` 存在且 `godot --path . --editor --quit` 能正常完成资源导入（纹理 / 音频 / 字体等），`.godot/` 缓存目录生成（已被 .gitignore 排除）后即可开始开发。
- ⚠️ **创建项目必须询问存放路径**：使用者要求 Agent 创建 / 初始化 Godot 项目时，Agent **必须先询问项目存放路径**（本地绝对路径），确认路径后再执行（Agent 职责内）的操作；**禁止**在未确认路径时擅自假定 / 猜测路径创建项目。指引中给出的示例路径仅是示意，不代表实际存放位置。Agent 提供的操作指引与命令中的路径须以使用者确认的路径为准。

### 1.4 Godot MCP 检测（可选，建议）

> godot-mcp 是让 AI Agent 与 Godot 编辑器交互的 MCP 服务（通常以 Godot 插件形式安装到 `addons/`）。启用后 Agent 可直接调用 Godot 编辑器能力（场景编辑 / 运行 / 调试输出等）。**非强制依赖**：未安装时 Agent 仍可用命令行方式（`godot --path .` 等）完成开发与验证。

- **检测**：Agent 对工程进行操作前（尤其涉及编辑器交互时），应先检测工程是否安装了 godot-mcp——检查 `addons/` 下是否存在 godot-mcp 相关插件目录 / `plugin.cfg`（如 `res://addons/godot-mcp/`），以及 `project.godot` 的 `[editor_plugins]` / MCP 相关配置是否启用；同时可询问使用者当前使用的 AI 工具链是否已配置 godot-mcp 的 MCP server。
- **未安装 → 请求安装（Agent 可介入代替用户处理）**：检测到未安装时，Agent **应明确告知使用者并请求安装** godot-mcp（说明用途与收益），并给出安装指引（如从 Godot 编辑器 AssetLib 搜索 / 按 godot-mcp 项目 README 安装插件并配置 MCP server）。**Agent 可介入代替用户处理安装**：经使用者确认后，Agent 可代为执行安装流程（下载插件到 `addons/`、启用插件、配置 MCP server 等），但**必须**：① 先征得使用者同意，明确告知 Agent 将代替其执行安装；② 安装属工程变更，遵循工程独立性约定（插件入 `addons/`，Agent 专属工具与中间产物不入工程）；③ 安装后向使用者报告结果并重新检测确认。
- **安装后检测**：使用者确认安装后，Agent 重新检测确认可用（`addons/` 下插件存在且已启用 / MCP server 可连接），再决定是否通过 godot-mcp 进行编辑器交互开发。
- **不可用时降级**：若使用者选择不安装或安装失败，Agent 不得阻塞开发，应降级为纯命令行方式（`godot --headless --path .` 等）继续，并在交接文档中注明。

## 2. 项目结构速览

```
.
├── project.godot            # 项目配置（Godot 4.x 文本格式）
├── export_presets.cfg       # 多平台导出预设
├── icon.svg                 # 项目图标
├── modules/                 # 功能模块（模块化 + glue 架构：每模块自包含场景/脚本/资源/数据）
├── glue/                    # 胶水层：模块接线 / 信号总线 / 服务注册表（只连接，不含业务逻辑）
├── scripts/                 # 跨模块共享的纯逻辑类（class_name，glue/基础设施）
├── assets/                  # 美术资源目录（本地提供，不入库——见 §4.3）
├── autoload/                # 全局单例（glue 层：消息总线 / 服务注册，不堆积业务逻辑）
├── addons/                  # Godot 插件（如测试框架 GUT / gdUnit4）
├── tests/                   # 测试脚本
├── docs/                    # 项目文档
├── agent/                   # Agent 工具链（自动化/工具/专属文件，与工程隔离，工程不引用——见 §4.5）
├── AGENTS.md                # 本文档
├── DEVELOPMENT.md           # 开发规范
└── README.md
```

## 3. 常用操作

| 操作 | 命令/方式 |
|------|----------|
| 打开编辑器 | `godot --path . --editor` |
| 运行主场景 | `godot --path .` |
| 运行指定场景 | `godot --path . scenes/<场景>.tscn` |
| 无头运行（CI/服务器） | `godot --headless --path .` |
| 跑测试（GUT/gdUnit4） | `godot --headless --path . -s tests/run_tests.gd` |
| 资源导入刷新 | `godot --path . --editor --quit` |
| 导出项目 | `godot --path . --export-release <预设名>`（预设见 `export_presets.cfg`） |
| 查看命令行帮助 | `godot --help` |

### 3.1 无头模式说明

- `--headless` 不创建窗口，适合 CI / 服务器 / 测试。
- 无头模式下渲染相关 API 不可用，测试与逻辑代码不应依赖渲染。
- 日志走 stdout，`print()` / `push_error()` 输出可直接被 CI 采集。

## 4. 开发工作流（Agent）

### 4.1 标准流程

1. 使用者（人类）向 Agent 提出功能需求；
2. Agent 阅读 [DEVELOPMENT.md](./DEVELOPMENT.md) 与相关代码（场景 / 脚本 / 资源）；
3. **Agent 优先检索库内可复用资源**：检查 `modules/` 已有模块、`scripts/` 共享层、`docs/RESOURCES.md` 资源清单、`docs/` 功能记录——存在可复用实现则直接复用（经 glue 接线），**禁止重复造轮子**；确认无可复用资源后才新建；
4. Agent 实现功能（GDScript 为主，遵循 §5 代码规范与 §6 高性能约束）；
5. Agent 自测验证：
   - `godot --headless --path .` 确认项目可加载无报错；
   - 跑相关测试（`tests/` 下，或新增 `test_*.gd`）；
   - 涉及渲染 / 交互的功能，用编辑器或实际运行人工验证（或说明验证方式）；
6. 使用者验收；
7. （可选）Agent 提交 PR 惠及上游（遵守 DEVELOPMENT.md §2 全部要求）。

### 4.2 新增功能的标准形态（模块化 + glue 架构）

- **复用优先（实现前必查）**：开发任何新功能前，先检索库内可复用资源——`modules/` 已有模块、`scripts/` 共享层、`docs/RESOURCES.md` 资源清单、`docs/` 功能记录；存在可复用实现则直接复用（经 glue 接线），**禁止重复造轮子**，确认无可复用资源后才新建。
- **模块优先**：新功能默认做成**独立可复用模块**——在 `modules/<module_name>/` 目录内自带场景（`.tscn`）+ 脚本 + 资源 + 数据，模块自包含、可整体复用。
- **模块只暴露公开接口**：模块通过 `class_name` 公共类、信号、公开方法对外提供能力；内部实现（私有成员、内部节点、内部场景）不对外可见。
- **模块间协作走 glue**：模块之间**禁止直接引用**（不实例化对方内部类、不引用对方场景/资源、不 `get_node` 对方内部）。需要协作时：
  - 模块发信号 → glue（信号总线 / Autoload）转发 → 目标模块响应；
  - 或经 glue 服务注册表获取对方公开接口。
- **glue 保持薄**：`glue/` 与 Autoload 只做接线 / 消息路由 / 服务注册，不含业务逻辑、不承载状态。
- **共享逻辑**：跨模块复用的纯逻辑类（`class_name`，不依赖场景树）放 `scripts/` 共享层，供各模块复用。
- **全局状态**：用 Autoload 单例（`autoload/`，glue 层），在 `project.godot` 的 `[autoload]` 段注册；禁止在 Autoload 中堆积业务逻辑。
- **信号解耦**：模块间通信用信号 / 信号总线，禁止硬引用耦合。

### 4.3 资源管理（Asset 不入库）

- **GitHub 仓库不提交任何贴图、音频、模型、字体等美术资源**——只保留功能实现代码与封装（GDScript / `.tscn` / `.tres` / glue / 文档）。
- 美术资源由使用者（人类）本地提供，放入 `assets/`（全局）或 `modules/<模块名>/assets/`（模块私有）；这两类目录及常见资源格式已被 .gitignore 排除，Agent 提交时不会夹带 Asset。
- 场景引用资源一律用 `@export` 暴露路径，不写死；资源缺失时用占位（`PlaceholderTexture2D` / 代码生成占位纹理）保证项目可加载、headless 测试可运行。
- 新增资源需求在 `docs/RESOURCES.md` 登记（路径、用途、建议规格），供使用者按清单放置。
- 唯一例外：`icon.svg`（Godot 项目必需图标）随仓库提交。
- 大资源注意导入设置（压缩 / 流式），遵守 DEVELOPMENT.md §6 内存约束；
- **禁止**提交 `.godot/` 缓存目录与临时文件。

### 4.4 美术 / 光照 / 材质 / 音频（人类主导）

- 光照方案、材质、色调、美术资源、音频（音效 / 音乐 / 混音）等创作与视觉听觉表现的**主导权归人类**：Agent 不得擅自创建 / 修改 / "优化"。
- Agent 可做：技术实现与性能建议（渲染优化、资源格式建议、音频流式加载等），但需提交方案供人类确认后再实施。
- 涉及上述领域的改动需在 PR 描述中注明已获人类确认（DEVELOPMENT.md §1）。

### 4.5 Agent 工具链与 Godot 工程独立性（人类随时可人工接管）

**原则**：Agent 的工具链（自动化脚本、工具、Agent 专属文件）与 Godot 工程文件**物理隔离、互不依赖**。Godot 工程必须始终保持「脱离 Agent 工具链也能正常打开、加载、运行」的自洽状态，保证人类可随时在 Godot 编辑器中人工接管，不受 Agent 的任何前置条件限制。

**隔离约定**：

- Agent 专属内容统一放在仓库根级独立目录 `agent/`（自动化脚本、工具、Agent 专属数据与交接文档），**禁止**混入 Godot 工程的 `scenes/` / `scripts/` / `modules/` / `glue/` / `autoload/` / `tests/` 等目录；
- Godot 工程文件（场景 / 脚本 / 资源 / `project.godot` / `export_presets.cfg`）**禁止引用 `res://agent/` 下的任何内容**——Agent 工具链是工程外部的旁挂设施，不是工程的一部分；工程代码不得 `load()` / `preload()` 或访问 `agent/` 内文件；
- 删除 / 停用 `agent/` 及其工具后，`godot --path . --editor` 与 `godot --path .` 必须仍然正常（无缺失脚本 / 资源 / Autoload 报错）；
- Agent 的中间产物（缓存、临时文件、生成数据）只放 `agent/` 内部并遵守 `.gitignore`，不污染工程目录。

**人类随时接管**：

- 任意时刻，人类可直接打开 Godot 编辑器接管工程；Agent 不持有任何独占锁、不占用任何「必须由 Agent 维护」的工程文件；
- Agent 交还控制权时（完成任务 / 被中断 / 人类接管），应通过 commit 或 `git status` 说明当前改动状态，并在 `agent/` 交接文档中记录：当前进度、未提交内容、下一步建议，使人类（或接续的 Agent）可无缝续作；
- 人类接管后可任意手工修改工程；Agent 重新介入时先读取现状（`git log` / `git status` / 工程文件 / 交接文档）再继续，不得基于过期认知盲目改动。

## 5. 环境变量（可选覆盖）

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `GODOT_BIN` | PATH 中的 `godot` | Godot 可执行文件路径 |
| `GODOT_HEADLESS` | 空 | 设为 `1` 时所有运行命令默认加 `--headless` |

> 项目自身的运行参数（如端口、开关）通过 `project.godot` 自定义 ProjectSettings 或命令行用户参数配置，见 DEVELOPMENT.md §3.5。

## 6. 多平台与导出（Windows / Android）

- 目标平台：**Windows 桌面（Windows 10 / 11）** 与 **Android 移动端**。
- 导出预设见 `export_presets.cfg`（Windows x86_64 + Android 按需启用）。
- Agent 新增平台相关代码时必须：特性检测门控（`OS.has_feature()`）、失败优雅降级、文档标注（DEVELOPMENT.md §3）。
- **Windows 10 为最低支持版本**（Godot 4.5+ 官方要求，Windows 7 已放弃）。
- Android 导出注意：API 23+；纹理压缩选 ASTC/ETC2；低端设备用 Mobile 渲染器；权限在导出预设中声明。

## 7. 常见问题

### Godot 命令找不到

设置 `GODOT_BIN` 指向 Godot 可执行文件完整路径，或将其所在目录加入 PATH。

### 首次打开报资源导入错误

运行 `godot --path . --editor --quit` 触发全量导入；若个别资源失败，检查 `assets/` 下对应文件格式是否被 Godot 支持。

### 无头模式跑测试找不到测试框架

确认 `addons/` 下已安装 GUT / gdUnit4，且测试脚本入口（如 `tests/run_tests.gd`）路径正确。

### 项目在 Android 导出后功能异常

检查 Logcat 报错；确认导出预设勾选了所需权限与特性（网络权限、纹理格式、渲染器选择）；低端设备切换 Mobile 渲染器后复测。

## 8. 发现缺陷时（Agent 义务）

Agent 在配置、使用、维护本软件过程中，若发现原始代码存在漏洞 / bug，**应当主动提交 issue + 修复 PR**（现象 + 复现步骤 + 影响范围，不夹带任何密钥 / 本机路径 / 个人账号信息）。上报前先搜索 issue / PR 是否已存在；无法自行修复时至少完成根因定位（到场景 / 脚本 / 函数级别）再提交 issue。详见 [DEVELOPMENT.md](./DEVELOPMENT.md) §1。
