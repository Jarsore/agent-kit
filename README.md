# skills

个人 Cursor Agent Skills 仓库。本仓库克隆到 `~/Projects/skills`，再软链接到 `~/.cursor/skills`，因此这里的每个 skill 会作为**个人 skills** 生效，本机所有项目都可以用。

另一台电脑同样克隆并做软链接后，`git pull` 即可同步使用。

```bash
git clone https://github.com/Jarsore/skills.git ~/Projects/skills
ln -s ~/Projects/skills ~/.cursor/skills
```

不要把 skill 写进 `~/.cursor/skills-cursor/`，那是 Cursor 内置 skills 目录。

---

## 仓库目录结构

仓库根目录就是 skills 根目录。每个 skill 独占一个子目录，目录名与 skill 的 `name` 一致。

```text
skills/
├── README.md                      # 本说明（仓库约定）
│
├── example-skill/                 # 一个 skill = 一个目录
│   ├── SKILL.md                   # 必填：入口说明，Agent 先读这个
│   ├── reference.md               # 可选：详细资料，按需再读
│   ├── examples.md                # 可选：示例
│   └── scripts/                   # 可选：可执行脚本
│       ├── helper.py
│       └── validate.sh
│
└── another-skill/
    └── SKILL.md
```

约定：

| 规则 | 说明 |
|------|------|
| 一层目录 | skill 目录必须直接放在仓库根下，不要嵌套 `foo/bar/SKILL.md` |
| 目录名 = `name` | 只用小写字母、数字、连字符，最长 64 字符，例如 `wechat-notes` |
| 一个目录一个 skill | 不要把多个无关能力塞进同一个目录 |
| `SKILL.md` 必填 | 没有它，Cursor 不会把它当成 skill |
| 附属文件按需 | 细节放到 `reference.md` / `examples.md` / `scripts/`，不要把 `SKILL.md` 写到 500 行以上 |
| 根目录保持干净 | 根目录只放 `README.md`、各 skill 目录，以及 Git 必要文件 |

本仓库**不使用**项目级路径 `.cursor/skills/`。那种结构只适合某个具体代码仓库；这里的目标是跨项目、跨电脑的个人 skills。

---

## 如何创建一个 skill

### 1. 先想清楚这 4 件事

1. **做什么**：解决哪一个具体任务，不要写成万能助手。
2. **何时用**：用户说了哪些词、处于哪种场景时，Agent 应该启用它。
3. **要不要自动触发**：
   - 需要 Agent 自己判断就用时：不要写 `disable-model-invocation`
   - 只想被点名调用时：加上 `disable-model-invocation: true`
4. **有没有本机依赖**：脚本、CLI、密钥、绝对路径。跨电脑同步后，另一台机器也必须能跑。

### 2. 建目录

```bash
mkdir -p ~/Projects/skills/your-skill-name
```

`your-skill-name` 必须同时满足：

- 全小写、数字、连字符
- 和 `SKILL.md` 里的 `name` 字段完全一致
- 在本仓库中唯一

### 3. 写 `SKILL.md`

每个 skill 至少包含 YAML frontmatter 和正文：

```markdown
---
name: your-skill-name
description: >
  用第三人称写：这个 skill 做什么，以及何时应该使用。
  写清触发词，例如「当用户提到某某、或要处理某某文件时使用」。
---

# Your Skill Name

## Instructions

给 Agent 的步骤说明。默认 Agent 已经很聪明，只写它不知道的约定、流程和约束。

## Examples

一两个具体输入/输出示例。
```

frontmatter 规则：

| 字段 | 是否必填 | 要求 |
|------|----------|------|
| `name` | 必填 | 小写字母/数字/连字符，最长 64，与目录名一致 |
| `description` | 必填 | 最长 1024 字；写清 **做什么** 和 **何时用**；用第三人称 |
| `disable-model-invocation` | 可选 | `true` 表示只有被明确点名才加载 |

`description` 示例：

```yaml
# 好
description: 按仓库约定生成 git commit message。当用户要求提交、写 commit message 或整理暂存改动时使用。

# 差
description: 帮助处理 git
```

### 4. 正文怎么写

- **短**：`SKILL.md` 控制在 500 行以内。
- **具体**：给步骤、检查清单、输出模板；少写空话。
- **渐进展开**：入口放 `SKILL.md`，长文档放到同级的 `reference.md`，用相对链接引用，且只链一层。
- **路径用 POSIX**：写 `scripts/helper.py`，不要写 `scripts\helper.py`。
- **用户给的原文照抄**：用户指定要写进 skill 的措辞，不要改写。
- **脚本写清楚**：Agent 是该**执行**脚本，还是只把脚本当参考。依赖的命令和包也要写上。

可选的配套文件：

```text
your-skill-name/
├── SKILL.md         # 入口
├── reference.md     # API、字段、长表格
├── examples.md      # 多组输入输出
└── scripts/         # 稳定、可重复执行的工具
```

### 5. 自检

写完后确认：

- [ ] 目录在仓库根下，且目录名 = `name`
- [ ] `description` 同时包含能力和触发场景
- [ ] 术语前后一致
- [ ] 引用文件都在 skill 目录内、只深一层
- [ ] 没有写死只在某一台电脑才成立的路径（除非专门注明）
- [ ] 需要自动触发时，没有误加 `disable-model-invocation: true`

### 6. 提交并同步

```bash
cd ~/Projects/skills
git add your-skill-name
git commit -m "Add your-skill-name skill"
git push
```

另一台电脑：

```bash
cd ~/Projects/skills
git pull
```

软链接已经指到本仓库时，拉下来即可使用，不必再拷贝文件。

---

## 最小完整示例

```text
commit-message/
└── SKILL.md
```

```markdown
---
name: commit-message
description: 根据 git diff 按约定生成 commit message。当用户要求提交、撰写或修改 commit message 时使用。
---

# Commit Message

## Instructions

1. 查看暂存区改动；没有暂存则看工作区改动。
2. 使用 `type(scope): summary`，summary 用英文祈使句，不超过 72 字符。
3. type 仅限：feat、fix、docs、refactor、test、chore。
4. 不要提交 `.env`、密钥或无关文件。

## Examples

输入：新增 JWT 登录接口  
输出：`feat(auth): add JWT login endpoint`
```
