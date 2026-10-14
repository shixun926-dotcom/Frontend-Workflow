# Frontend-Workflow · 前端开发总控 Skill

> 让 AI 按「视觉设计 → 工程实现 → 视觉验收」三个阶段做前端，而不是打开编辑器就开始写代码。

一个面向 AI 编程代理（DSH / Claude Code / Codex 等）的**总控编排 Skill**。它本身不实现任何组件，只负责在动手前读取项目规范、识别真实技术栈，然后把三个专职 Skill 按固定顺序串起来，并在每个阶段结束后要求汇报结论。解决的核心问题是：AI 直接写前端时常见的**跳过设计、套用模板、只看代码不开页面**。

<p>
<img alt="type" src="https://img.shields.io/badge/type-agent%20skill-6E56CF">
<img alt="stages" src="https://img.shields.io/badge/stages-3-FF6B35">
<img alt="deps" src="https://img.shields.io/badge/dependencies-3%20skills-2E6BE6">
<img alt="stack" src="https://img.shields.io/badge/stack-React%20%7C%20Vue%203%20%7C%20Any-73B84B">
<img alt="license" src="https://img.shields.io/badge/license-%E8%A7%81%E4%B8%8B%E6%96%87-lightgrey">
</p>

---

## 目录

- [这是什么](#这是什么)
- [三个阶段](#三个阶段)
- [依赖的 Skill](#依赖的-skill)
- [安装](#安装)
- [技术栈适配](#技术栈适配)
- [执行规则](#执行规则)
- [最终交付](#最终交付)
- [已知坑](#已知坑)
- [目录结构](#目录结构)
- [文档索引](#文档索引)
- [使用前检查清单](#使用前检查清单)
- [已知限制](#已知限制)
- [License](#license)

---

## 这是什么

`frontend-workflow` 是一个**编排层** Skill，不是实现层。它规定的是「顺序」和「纪律」：

- **动手前先读项目**：`AGENTS.md` / `CLAUDE.md` / `package.json` / 锁文件 / 构建与路由配置都要实际打开看，**不得根据用户描述或目录名猜技术栈**。
- **三个阶段不可跳**：视觉设计 → 工程实现 → 视觉验收，每阶段结束先汇报再进入下一阶段。
- **验收必须看真实页面**：优先启动项目、用浏览器或截图工具检查；**没有截图能力时必须明说无法完成真实视觉验收，不得声称已经检查过**。
- **不为了视觉效果牺牲**可访问性、可维护性、性能或业务正确性。

> 与一般「前端最佳实践」提示词的区别：它把工作**拆给三个上游专用 Skill**，自己只负责调度、技术栈判定和阶段间的把关。

---

## 三个阶段

| 阶段 | 调用 | 产出 |
| --- | --- | --- |
| **一、视觉设计** | `frontend-design` | 页面目标与核心操作、视觉方向（字体 / 色彩 / 间距 / 布局 / 信息层级）、可执行的设计规范 |
| **二、工程实现** | `frontend-production-shadcn`（或仅取其设计原则） | 完整页面、导航、表单、响应式布局与交互状态；复用现有组件优先 |
| **三、视觉验收** | `premium-ui` | 真实页面的排版 / 对齐 / 留白 / 色彩 / 层级 / 组件一致性检查，问题修复后复查 |

**阶段一（视觉设计）**要求：

- 明确页面目标、主要用户和核心操作
- 提出**有实质差异**的设计方向，并说明最终选择理由
- 形成可执行的设计规范后，再开始实现

**阶段二（工程实现）**要求：

- 先复用现有组件和依赖
- 处理加载、空数据、错误、禁用、成功反馈等状态
- 保留真实业务逻辑，**不使用假交互冒充功能完成**
- 未经授权不得大规模重构或更换框架

**阶段三（视觉验收）**要求：

- 检查真实实现，而不只是阅读代码
- 检查按钮、表单、导航和主要交互是否真的可用
- 记录设计目标与实际效果之间的差距，并推动修复

---

## 依赖的 Skill

本仓库**只包含总控逻辑**，三个专职 Skill 需另行安装：

| Skill | 上游仓库 | 许可 | 文件数 | 作用 |
| --- | --- | --- | --- | --- |
| `frontend-design` | [vercel-labs/open-agents](https://github.com/vercel-labs/open-agents)（`.agents/skills/frontend-design/`） | MIT | 1 | 生成有辨识度的、避免「AI 通用审美」的界面设计 |
| `frontend-production-shadcn` | [wzx2002/codex-frontend-skill](https://github.com/wzx2002/codex-frontend-skill) | 未声明 | 4 | React + TS + Tailwind + shadcn/ui 的生产级实现规范 |
| `premium-ui` | [rpraharaj/front-end-agent-skill](https://github.com/rpraharaj/front-end-agent-skill) | MIT | 24 | Brief → 方案探索 → 设计系统 → Token → 构建 → 评审 的高端 UI 流程 |

安装后应位于（以 DSH 为例）：

```
~/.dsh/skills/frontend-design/SKILL.md
~/.dsh/skills/frontend-production-shadcn/SKILL.md
~/.dsh/skills/premium-ui/SKILL.md
~/.dsh/skills/frontend-workflow/SKILL.md      ← 本仓库内容
```

> ⚠️ **Skill 名 ≠ 仓库名**。`frontend-design` 来自 `open-agents`，`frontend-production-shadcn` 来自 `codex-frontend-skill`，`premium-ui` 来自 `front-end-agent-skill`。目录名请按 **Skill 名**取，不要按仓库名取。

---

## 安装

三个依赖 Skill 用 `codeload` 直下（Windows Git Bash / macOS / Linux 均可）：

```bash
# 1) frontend-design —— open-agents 是个 11MB 的单仓多包仓库，
#    这里只取 .agents/skills/frontend-design 一个目录（--strip-components=4）
mkdir -p ~/.dsh/skills/frontend-design
curl -sL https://codeload.github.com/vercel-labs/open-agents/tar.gz/refs/heads/main \
  | tar xz -C ~/.dsh/skills/frontend-design --strip-components=4 \
        open-agents-main/.agents/skills/frontend-design

# 2) frontend-production-shadcn
mkdir -p ~/.dsh/skills/frontend-production-shadcn
curl -sL https://codeload.github.com/wzx2002/codex-frontend-skill/tar.gz/refs/heads/main \
  | tar xz -C ~/.dsh/skills/frontend-production-shadcn --strip-components=1

# 3) premium-ui（保留 scripts/ 的可执行位）
mkdir -p ~/.dsh/skills/premium-ui
curl -sL https://codeload.github.com/rpraharaj/front-end-agent-skill/tar.gz/refs/heads/main \
  | tar xz -C ~/.dsh/skills/premium-ui --strip-components=1

# 4) 本技能
mkdir -p ~/.dsh/skills/frontend-workflow
curl -sL https://codeload.github.com/shixun926-dotcom/Frontend-Workflow/tar.gz/refs/heads/main \
  | tar xz -C ~/.dsh/skills/frontend-workflow --strip-components=1
```

其他工具的 Skill 根目录：Claude Code 用 `~/.claude/skills/`，Codex 用 `~/.codex/skills/`，OpenCode 用 `~/.config/opencode/skills/`——把上面的 `~/.dsh/skills` 换掉即可。

**验证安装**：随便开一个前端任务问 AI「现在要改这个页面」，它应当先读 `AGENTS.md` 和 `package.json`，而不是直接改代码。

---

## 技术栈适配

本 Skill 会在动手前判定技术栈，再决定怎么用另外三个 Skill。

### 情况 A：React 项目

三者全量依次执行：`frontend-design` → `frontend-production-shadcn` → `premium-ui`。

只有在项目**已采用或明确允许** React、Tailwind CSS、shadcn/ui 时，才直接使用第二个 Skill 的具体组件实现方案。

### 情况 B：Vue 3 项目

1. `frontend-design`：确定视觉方向与页面布局
2. `frontend-production-shadcn`：**仅作为设计规范和工程质量参考**，不执行其中专属于 React / JSX / React Hooks / shadcn/ui 的实现指令
3. 用项目现有的 Vue 3、TypeScript、路由、UI 组件库和 CSS 方案完成实现
4. `premium-ui`：检查实际页面并推动修复

> **不得为了适配第二个 Skill 而把 Vue 3 项目迁移到 React。**

### 情况 C：其他技术栈

先检查项目已有技术和约束；`frontend-design` 用于视觉规划，`frontend-production-shadcn` 中**可迁移的设计原则**作为参考，用当前项目适用的技术实现，最后用 `premium-ui` 的评审方法验收。

---

## 执行规则

1. 每个阶段完成后，简要报告结论，再进入下一阶段
2. 如果某个 Skill 不存在、未被加载或无法执行，**明确说明**并采用可行的替代方案
3. 不得假设其他 Skill 会自动运行；必要时明确请求调用或读取相应 Skill
4. 遵循用户的明确要求、项目规范和现有技术栈
5. 不为追求视觉效果而牺牲可访问性、可维护性、性能或业务正确性
6. 完成后运行项目支持的构建、检查和测试命令，并汇报结果

---

## 最终交付

完成后必须报告：

- 识别出的技术栈及版本
- 采用的设计方向和关键设计决策
- 修改的主要文件及功能
- 三个阶段各自的执行情况
- 构建、测试和实际视觉检查的结果
- 尚未解决的问题和需要人工确认的事项

---

## 已知坑

### 1. 文件名必须是大写 `SKILL.md`

DSH 的发现逻辑是**严格字符串比较**，源码中写死：

```js
segments[1] === "SKILL.md"
path: join(entry.path, "SKILL.md")
```

- **Linux / macOS**（大小写敏感）：小写 `skill.md` **根本不会被发现**，Skill 静默不存在，且不报任何错
- **Windows**：文件系统不敏感，初次扫描可能仍能读到；但目录监听走上面的严格比较，新增 / 改名事件不触发，行为不可靠

结论：一律用大写 `SKILL.md`。本仓库已是大写，克隆 / 解压后请确认未被改小写。

### 2. 目录名建议与 frontmatter `name` 一致

本仓库叫 `Frontend-Workflow`，而 frontmatter 是 `name: frontend-workflow`。DSH 按目录名解析 entry（`<目录名>/SKILL.md`），两者不一致容易产生歧义——**建议安装到 `frontend-workflow/`**（安装命令已这样写）。

### 3. `SKILL.md` 超过 8192 码点会被工具结果裁剪

DSH 的 tool-result pruner 默认 `thresholdChars = 8192 / headChars = 4096 / tailChars = 1024`，**超过 8192 码点**的技能正文会被裁成「头 4096 + 尾 1024」。实测：

| SKILL.md | 码点 | 是否被裁剪 | 丢失内容 |
| --- | --- | --- | --- |
| `frontend-design` | 4,401 | 否 | — |
| `frontend-production-shadcn` | 14,459 | **是** | `§5 Tailwind 设计规则` ~ `§20` 主体，约 10,450 字符 |
| `premium-ui` | 27,149 | **是** | `## Workflow` 全流程，约 21,790 字符 |
| `frontend-workflow`（本技能） | 2,084 | 否 | — |

**缓解方式**：需要完整正文时**用 read 工具按 `offset` / `limit` 分段读取**，每次结果都落在阈值内；或把大技能的关键小节拆成平级 Skill。

### 4. 上游三个仓库非本仓库维护

上游改动（改名、移动目录、改许可）会让上面的安装命令失效。安装命令若报 404，先确认上游默认分支是否仍为 `main`、目录是否仍在原位。

---

## 目录结构

```
Frontend-Workflow/
├── SKILL.md      # 总控 Skill 本体（frontmatter + 三阶段编排规则）
├── NOTES.md      # 依赖路径、安装命令、失效坑与排查
└── README.md     # 本文件
```

---

## 文档索引

| 文档 | 内容 |
| --- | --- |
| `SKILL.md` | Skill 本体：技术栈判定、三个阶段的要求、执行规则、交付清单 |
| `NOTES.md` | 依赖 Skill 的仓库 / 安装路径对照、名称陷阱、其他工具的 Skill 目录、5 类静默失效坑、安装命令与排查 |
| `README.md` | 本文件：项目介绍、阶段说明、安装与技术栈适配 |

---

## 使用前检查清单

- [ ] 三个依赖 Skill 都已装进 `~/.dsh/skills/`（或对应工具的 Skill 根目录）
- [ ] 每个 `SKILL.md` 的文件名都是**大写**
- [ ] 目录名与 frontmatter 的 `name` 一致（建议统一小写 kebab-case）
- [ ] 项目根目录存在 `AGENTS.md` / `CLAUDE.md` / README，且内容与当前代码一致
- [ ] 确认技术栈是 React 还是 Vue 3 还是其他——**不要靠猜**
- [ ] 环境有浏览器或截图能力；**没有就在阶段三明确声明无法完成真实视觉验收**
- [ ] `premium-ui` 若要用 `scripts/shoot.sh` 截图，需先装好 Playwright

---

## 已知限制

- 本 Skill **只做编排**，不含任何具体组件实现；实现质量取决于三个依赖 Skill 是否安装且可用
- 阶段三依赖浏览器 / 截图工具。纯命令行环境无法完成真实视觉验收——Skill 要求此时**必须声明**，不得假装检查过
- `frontend-production-shadcn` 面向 React + Tailwind + shadcn/ui；Vue 3 项目只能取其设计原则，不能执行其实现在指令
- `premium-ui/scripts/shoot.sh` 依赖 Playwright，未安装时会以 exit code 3 失败
- 三个依赖 Skill 均为上游第三方仓库，本仓库不跟进其更新
- 阶段划分会增加 token 消耗与往返轮次，简单的单组件微调不必强行走完整流程

---

## License

本仓库的**编排规则与文档**由 [shixun926-dotcom](https://github.com/shixun926-dotcom) 编写，仓库暂未附 `LICENSE` 文件。

三个依赖 Skill 各自保留其上游许可：

| Skill | 许可 |
| --- | --- |
| `frontend-design` | MIT（vercel-labs/open-agents） |
| `frontend-production-shadcn` | 上游未声明许可，使用前请自行确认 |
| `premium-ui` | MIT（rpraharaj/front-end-agent-skill） |

使用 / 分发前请一并遵守上游许可条款。
