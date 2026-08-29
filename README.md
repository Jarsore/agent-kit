# agent-kit

个人 Cursor Agent 工具箱：用同一个 Git 仓库同步 **skills**、**subagents（子智能体）** 和 **workflows（工作流说明）**。

本机建议克隆到 `~/Projects/agent-kit`，再按下方软链接到 Cursor 会扫描的目录。另一台电脑同样克隆并软链后，`git pull` 即可同步。

```bash
git clone https://github.com/Jarsore/agent-kit.git ~/Projects/agent-kit
ln -s ~/Projects/agent-kit/skills ~/.cursor/skills
ln -s ~/Projects/agent-kit/agents ~/.cursor/agents
```

不要把内容写进 `~/.cursor/skills-cursor/`，那是 Cursor 内置 skills 目录。

---

## 仓库目录结构

```text
agent-kit/
├── README.md                 # 本说明
├── skills/                   # Agent Skills（说明书）
│   └── daily-skills/
│       ├── SKILL.md
│       ├── examples.md
│       └── reference.md
├── agents/                   # 自定义子智能体
│   └── README.md
└── workflows/                # 工作流说明（自管文档，非 Cursor 强制目录）
    └── README.md
```

| 目录 | 对应概念 | 本机生效方式 |
|------|----------|--------------|
| `skills/` | Skill | 软链到 `~/.cursor/skills` |
| `agents/` | Subagent | 软链到 `~/.cursor/agents` |
| `workflows/` | Workflow 文档 | 给人与 Agent 阅读；需要自动化时再接到 Automation / Hook |

根目录只放 `README.md`、上述三个目录，以及 Git 必要文件。

---

## Skills

每个 skill 独占 `skills/` 下一层子目录，目录名与 `name` 一致。

```text
skills/
└── example-skill/
    ├── SKILL.md              # 必填
    ├── reference.md          # 可选
    ├── examples.md           # 可选
    └── scripts/              # 可选
```

约定：

| 规则 | 说明 |
|------|------|
| 一层目录 | 放在 `skills/` 下，不要 `skills/foo/bar/SKILL.md` |
| 目录名 = `name` | 小写字母、数字、连字符，最长 64 |
| `SKILL.md` 必填 | 没有它，Cursor 不会当成 skill |
| 附属文件按需 | 细节放 `reference.md` / `examples.md` / `scripts/`，`SKILL.md` 尽量 &lt; 500 行 |

### 如何新增一个 skill

1. 想清：做什么、何时触发、要不要自动调用、有无本机依赖。
2. 建目录：`mkdir -p ~/Projects/agent-kit/skills/your-skill-name`
3. 写 `SKILL.md`：

```markdown
---
name: your-skill-name
description: >
  用第三人称写：做什么，以及何时使用。写清触发词。
---

# Your Skill Name

## Instructions

步骤说明。

## Examples

输入 / 输出示例。
```

| 字段 | 必填 | 要求 |
|------|------|------|
| `name` | 是 | 与目录名一致 |
| `description` | 是 | 做什么 + 何时用；第三人称 |
| `disable-model-invocation` | 否 | `true` = 仅被点名时加载 |

4. 提交并同步：

```bash
cd ~/Projects/agent-kit
git add skills/your-skill-name
git commit -m "Add your-skill-name skill"
git push
```

### Skill 最小示例

```text
skills/commit-message/SKILL.md
```

```markdown
---
name: commit-message
description: 根据 git diff 按约定生成 commit message。当用户要求提交或撰写 commit message 时使用。
---

# Commit Message

## Instructions

1. 查看暂存区改动；没有暂存则看工作区。
2. 使用 `type(scope): summary`，summary 英文祈使句，不超过 72 字符。
3. type 仅限：feat、fix、docs、refactor、test、chore。
```

---

## Agents（子智能体）

子智能体是主 Agent 可委派的专用助手，配置为 `agents/` 下的单个 `.md` 文件。

```markdown
---
name: code-reviewer
description: 按团队规范审代码。在用户要求 code review 或刚改完关键逻辑后使用。
---

你是代码审查助手。审查时关注正确性、安全与可维护性，给出可执行的修改建议。
```

| 规则 | 说明 |
|------|------|
| 文件名 | 建议与 `name` 一致，如 `code-reviewer.md` |
| `description` | 写清何时应委派给它 |
| 正文 | 即该子智能体的系统提示 |

软链到 `~/.cursor/agents` 后，个人级子智能体在本机所有项目可用。项目专用子智能体仍可写在某个代码仓库的 `.cursor/agents/`（本仓库不替代那种用法）。

多数日常任务不必自建子智能体；只有需要固定人设、可复用委派时再加。

---

## Workflows（工作流）

`workflows/` 用来沉淀**可复用流程说明**（步骤、触发条件、要用到的 skill / subagent）。  
Cursor 不会自动扫描此目录；它是给你和 Agent 对照执行的文档，也可作为日后接 Automation / Hook 的设计稿。

建议每个流程一个 markdown，例如：

```text
workflows/
└── ship-hotfix.md
```

文档里写清：触发场景 → 步骤清单 → 调用哪些 skill/agent → 完成标准。

---

## 三类东西怎么选

| 你想要… | 放哪里 |
|---------|--------|
| 固定「怎么写 / 怎么输出」 | `skills/` |
| 固定「谁去干、什么人设」 | `agents/` |
| 固定「多步怎么串起来」 | `workflows/`（文档）；真正定时/事件触发再用 Automation |

---

## 另一台电脑

```bash
git clone https://github.com/Jarsore/agent-kit.git ~/Projects/agent-kit
ln -s ~/Projects/agent-kit/skills ~/.cursor/skills
ln -s ~/Projects/agent-kit/agents ~/.cursor/agents
```

之后只需：

```bash
cd ~/Projects/agent-kit
git pull
```

密钥、API Key 等仍放本机环境变量，不要提交进本仓库。
