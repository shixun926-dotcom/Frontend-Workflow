# Frontend-Workflow 注意事项

> 本文档随 `frontend-workflow` 总控 Skill 一起分发，说明它**依赖的外部 Skill、安装路径**，以及几个会让它**静默失效**的坑。
> 所有结论均在 DSH（DeepSeek Harness）上实测验证，非推测。

---

## 一、这个 Skill 做什么

`frontend-workflow` 本身不写代码，它是一个**总控 / 编排（orchestrator）Skill**：在任何前端任务开始前，先读项目规范、识别技术栈，再按三个阶段调度另外三个 Skill 完成工作。

```
阶段一 · 视觉设计  →  frontend-design
阶段二 · 工程实现  →  frontend-production-shadcn
阶段三 · 视觉验收  →  premium-ui
```

它的价值在于**顺序和边界**：先定方向再写代码，最后用真实页面而非代码来验收，并且明令禁止为了迁就某个 Skill 而更换项目技术栈。

---

## 二、依赖的 Skill 及其路径

⚠️ 本 Skill **不包含**下面三个 Skill 的任何内容，必须**分别单独安装**。缺任何一个，对应阶段就无法执行。

| 阶段 | Skill 名（frontmatter `name`） | 上游仓库 | DSH 安装路径 |
|---|---|---|---|
| 一 · 视觉设计 | `frontend-design` | [vercel-labs/open-agents](https://github.com/vercel-labs/open-agents) 的子目录 `.agents/skills/frontend-design/` | `~/.dsh/skills/frontend-design/` |
| 二 · 工程实现 | `frontend-production-shadcn` | [wzx2002/codex-frontend-skill](https://github.com/wzx2002/codex-frontend-skill) | `~/.dsh/skills/frontend-production-shadcn/` |
| 三 · 视觉验收 | `premium-ui` | [rpraharaj/front-end-agent-skill](https://github.com/rpraharaj/front-end-agent-skill) | `~/.dsh/skills/premium-ui/` |

Windows 上的实际路径：

```
C:\Users\Administrator\.dsh\skills\frontend-design\SKILL.md
C:\Users\Administrator\.dsh\skills\frontend-production-shadcn\SKILL.md
C:\Users\Administrator\.dsh\skills\premium-ui\SKILL.md
```

### 坑：Skill 名 ≠ 仓库名

装的时候**目录必须用 frontmatter 里的 `name`**，不能用仓库名：

| 仓库名 | 实际 Skill 名 |
|---|---|
| `codex-frontend-skill` | `frontend-production-shadcn` |
| `front-end-agent-skill` | `premium-ui` |
| `open-agents`（monorepo） | `frontend-design` |

### 其他 Agent 工具的默认 Skill 目录（供参考）

Claude Code `~/.claude/skills/` · Codex `~/.codex/skills/` · OpenCode `~/.config/opencode/skills/` · 通用 `~/.agents/skills/`

---

## 三、⚠️ 会导致静默失效的坑

### 1. 文件名必须是大写 `SKILL.md`

本 Skill 最初的文件名是小写 `skill.md`，**本仓库已按下面的要求改为大写 `SKILL.md`**，请勿再改回小写。

DSH 的发现逻辑是**严格字符串比较**，源码中写死：

```js
segments[1] === "SKILL.md"
path: join(entry.path, "SKILL.md")
```

- 在 **Linux / macOS**（大小写敏感文件系统）上：`skill.md` **根本不会被发现**，Skill 静默不存在，且不会报任何错。
- 在 **Windows** 上：由于文件系统不敏感，初次扫描时 `join(dir, "SKILL.md")` 打开会被解析到 `skill.md`，**可能**仍能读到；但目录监听走的是上面的严格字符串比较，新增 / 改名事件不触发，行为不可靠、不可依赖。

结论：这是**跨平台差异**，不是 Windows 能不能用的问题 —— 一律用大写 `SKILL.md` 才能保证任何平台都正常。

**若你的副本仍是小写，务必改名：**

```bash
git mv skill.md SKILL.md
```

### 2. 目录名建议与 frontmatter `name` 一致

当前目录名是 `Frontend-Workflow`，而 frontmatter 是 `name: frontend-workflow`。

DSH 按目录名解析 entry（`<目录名>/SKILL.md`），两者不一致容易产生歧义。建议把目录统一为 `frontend-workflow`。

另外 DSH 要求 **`name` 必须是 kebab-case**，`frontend-workflow` 符合要求。

### 3. 8192 字符裁剪 —— 对本 Skill 影响最大

DSH 的 tool-result pruner 默认参数（已从构建产物中确认）：

```
thresholdChars: 8192    headChars: 4096    tailChars: 1024
```

`skill` 工具加载 `SKILL.md` 属于一次 tool 结果，**只要超过 8192 个 Unicode 码点，就会被裁成「头 4096 + 尾 1024」，中间整段丢弃**。

| Skill | SKILL.md 码点数 | 加载结果 |
|---|---|---|
| `frontend-workflow`（本 Skill） | 2,084 | ✅ 完整 |
| `frontend-design` | 4,401 | ✅ 完整 |
| `frontend-production-shadcn` | 14,459 | ❌ 被裁，丢失约 64% |
| `premium-ui` | 27,149 | ❌ 被裁，丢失约 81% |

**这正是本总控 Skill 最脆弱的地方**：它要求「阶段三调用 `premium-ui` 做视觉验收」，但 `premium-ui` 被加载进来时，核心流程已经被裁掉了。

具体丢哪些段落（按码点偏移实测）：

- **`premium-ui`** —— `## Workflow (Route A — follow in order)` 起始于码点 **4671**，直到约 **26460** 的**整个流程正文全部落在被丢弃的中间段**。
  侥幸的是头部 4096 保住了 `Route first`（码点 1907）和第一条 `references/` 引用（码点 3340），所以「该读哪个文件」它还知道；尾部 1024 保住了 Support Files 的最后几条（`scripts/shoot.sh`、`scripts/audit-ui.sh`、`assets/design-showcase.html`）。
- **`frontend-production-shadcn`** —— 保住 §0–§4（到 "Visual Direction"），**丢掉 §5 Tailwind Design Rules 一直到 §20 的绝大部分**，也就是最核心的约束部分。

**缓解办法（任选其一）：**

1. 让 Agent 用 `read` 工具**分段读取** `SKILL.md`（指定 `offset` / `limit`），绕过单次结果上限；
2. 调高 pruner 的 `thresholdChars`（该配置可改，允许的键只有 `thresholdChars`、`headChars`、`tailChars`）；
3. 把 `SKILL.md` 压到 8192 码点以内，正文搬进 `references/` —— `premium-ui` 本来就是 references 结构，搬运很自然。

### 4. 视觉验收依赖 Playwright

`premium-ui` 的 `scripts/shoot.sh` 依赖 Playwright 做截图，**缺失时退出码为 3** 并给出提示。

若验收环境没有浏览器或截图能力，必须按本 Skill 第四节的要求**明确声明「无法完成真实视觉验收」，不得声称已经检查过页面效果**。

### 5. 不要为了适配 Skill 而迁移技术栈

`skill.md` 第二节已写明：**Vue 3 项目不得为了适配 `frontend-production-shadcn` 而迁移到 React**；该 Skill 在 Vue 项目中**只能作为设计规范与工程质量参考**，不得执行其中专属于 React / JSX / React Hooks / shadcn/ui 的实现指令。

这是本 Skill 的核心约束，请勿删改。

---

## 四、安装

```bash
SK=~/.dsh/skills

# 阶段一：frontend-design（注意它在 monorepo 的子目录里，需要 --strip-components=4）
mkdir -p "$SK/frontend-design"
curl -sL https://codeload.github.com/vercel-labs/open-agents/tar.gz/refs/heads/main \
  | tar xz -C "$SK/frontend-design" --strip-components=4 \
        open-agents-main/.agents/skills/frontend-design

# 阶段二：frontend-production-shadcn
mkdir -p "$SK/frontend-production-shadcn"
curl -sL https://codeload.github.com/wzx2002/codex-frontend-skill/tar.gz/refs/heads/main \
  | tar xz -C "$SK/frontend-production-shadcn" --strip-components=1

# 阶段三：premium-ui
mkdir -p "$SK/premium-ui"
curl -sL https://codeload.github.com/rpraharaj/front-end-agent-skill/tar.gz/refs/heads/main \
  | tar xz -C "$SK/premium-ui" --strip-components=1
```

> 如果没有 `curl`，把 `curl -sL <url> |` 换成 `git clone --depth 1 <仓库> /tmp/x &&`（但部分网络环境下 git 协议会被重置，codeload 通常更稳）。
>
> **目录名必须用 frontmatter 的 `name`**，见第二节的坑。

---

## 五、上线前检查清单

- [x] 文件名是 **`SKILL.md`**（大写）—— 克隆 / 解压后请确认未被改成小写
- [ ] 目录名与 frontmatter `name` 一致
- [ ] 三个依赖 Skill 都已安装到 `~/.dsh/skills/`
- [ ] 已确认各 `SKILL.md` 码点数（`premium-ui`、`frontend-production-shadcn` 需特别注意）
- [ ] 若视觉验收不可用，报告中已明确说明

---

## 附：三阶段边界速查

| | 输入 | 输出 | 禁止 |
|---|---|---|---|
| **一 · 视觉设计** `frontend-design` | 页面目标、用户、核心操作 | 视觉方向、字体、色彩、间距、布局、信息层级 | 未定方向就开始写代码 |
| **二 · 工程实现** `frontend-production-shadcn` | 阶段一的设计规范 | 完整页面、导航、表单、响应式、全部交互状态 | 大规模重构、更换框架、用假交互冒充完成 |
| **三 · 视觉验收** `premium-ui` | 真实运行的页面 | 差距清单 + 修复后复检 | 只读代码就声称验收通过 |
