# VS Code 全局使用手顺

本仓库作为唯一维护源，通过一个目录 junction（或直接把仓库放在用户主目录）让 VS Code 直接读取 Skills 和 Agents，无需复制文件。日常使用默认 Agent 直接提问即可；Coding Agent / Review Agent 是可选项，不会被自动选中。

- 仓库位置（示例）：`D:\code\copilot`
- Junction 位置：`C:\Users\phoen\personal-agent` → `D:\code\copilot`
- 实测环境：VS Code **1.142.0**、内置 Copilot Chat 0.70.0、Windows 11
- Instructions 例外：`~/.claude/CLAUDE.md` 是一份必要的用户目录副本，VS Code 默认读取它；Skills 和 Agents 直读仓库。

> 版本说明：Agent Skills 在 VS Code 1.109 起正式可用并默认启用；本文用到的设置名、默认目录和 Agent 格式已在本机 1.142.0 的程序文件中核实。更早版本可能不支持部分字段。

---

## A. 首次配置

### 1. 获取仓库

**方式 A（最简单）**：把仓库放在 `C:\Users\<你的用户名>\personal-agent`。这样设置里的 `~/personal-agent/...` 直接指向仓库，不需要 junction。

**方式 B**：仓库放在其他目录（本文以 `D:\code\copilot` 为例），再用 junction 映射到 `~/personal-agent`。路径含空格也可以。

### 2. 确认 VS Code 版本

`帮助 > 关于` 查看版本。低于 1.109 时先升级，否则 Skills 功能不可用。

### 3. 仓库不在用户主目录时，创建 junction（只需一次）

普通用户即可执行，不需要管理员权限：

```powershell
New-Item -ItemType Junction -Path "$HOME\personal-agent" -Target "D:\code\copilot"
```

替换 `D:\code\copilot` 为你的实际仓库路径。验证：

```powershell
Get-Item "$HOME\personal-agent" | Select-Object FullName, LinkType, Target
```

应显示 `LinkType: Junction`，`Target` 指向仓库。

> 为什么需要 junction：VS Code 的这些位置设置只接受**相对路径或 `~/` 开头**的路径，不接受 `C:\...` 绝对路径。junction 让 `~/personal-agent/...` 解析到 D 盘仓库。仓库已经在 `~/personal-agent` 时跳过这一步。

### 4. 打开用户级 settings.json

命令面板（`Ctrl+Shift+P`）→ **Preferences: Open User Settings (JSON)**。

### 5. 注册路径

在用户设置中加入（已有同名键时合并，不要重复定义）：

```jsonc
"chat.agentSkillsLocations": {
  "~/personal-agent/personal-skills": true,
  "~/personal-agent/coding-skills": true
},
"chat.agentFilesLocations": {
  "~/personal-agent/agents": true
}
```

说明：

- 路径用正斜杠 `/`，不要用反斜杠。
- 值是对象映射：`true` 启用，`false` 禁用。
- 不要写文件名或通配符，写到**包含 Skill/Agent 的目录**即可，VS Code 会向下查找。
- `chat.useAgentSkills` 和 `chat.useClaudeMdFile` 在 1.142.0 中默认 `true`，不需要额外添加；只有你之前手动关闭过才要设回 `true`。后者负责读取 `~/.claude/CLAUDE.md`。
- 不需要添加 `chat.instructionsFilesLocations`：当前 Instructions 走 `~/.claude/CLAUDE.md`，它不属于 `*.instructions.md` 扫描规则。

### 6. 让 VS Code 重新发现

命令面板 → **Developer: Reload Window**。修改设置后通常需要重载窗口或至少新开一个聊天会话。

### 7. 确认生效

1. 打开 Chat 视图，输入 `/`，应能看到 `doc-coauthoring`。
2. 打开 Chat 的 Agent 下拉框，应能看到 **Coding Agent** 和 **Review Agent**。
3. 命令面板 → **Chat: Open Customizations**，在 Agents / Skills 分区能看到已加载的条目和错误信息。
4. 在默认 Agent 下问一个普通问题，回答应遵守 `~/.claude/CLAUDE.md` 中的全局规则（中文、先结论等）。

前 3 项出现、第 4 项行为正确，才算配置成功。

---

## B. 日常使用

1. 用 VS Code 打开任意业务项目。
2. 打开 Chat 视图（`Ctrl+Alt+I`）。
3. 直接输入问题，使用默认 Agent。**不需要选择 Agent、指定 Skill 或执行审查流程。**
4. 默认 Agent 会遵守 `~/.claude/CLAUDE.md` 中的全局规则，并按 Skill 的 `description` 自动加载相关 Skill。例如文档写作任务可能读取 `doc-coauthoring`。
5. 需要确定使用某个 Skill 时，在提示中点名，或输入 `/skill-name`。
6. 只有需要结构化编码流程时，才在 Agent 下拉框切换到 **Coding Agent**，然后直接给任务，例如：
   - `帮我给这个模块补充单元测试`
   - `把这段 COBOL 逻辑改写成 Java`
   它按“理解需求 → 阅读现状 → 最小修改 → 补测试 → 实际运行验证”执行。
7. 只有需要独立只读审查时，才使用 Review Agent：
   - 在 Coding Agent 回答后点击 **交由 Review Agent 审查** 按钮（handoff）；
   - 或切到 **Review Agent**，输入 `审查当前未提交的修改`。
8. 修复问题：点击 **交由 Coding Agent 修复**，或切回 Coding Agent 输入 `修复审查中确认的问题`。
9. 自定义 Agent 的工具白名单、子代理和 handoff 只在 Chat 顶部的 **Session Target** 为 **Local** 时生效；普通问答不需要关心。
10. 查看实际使用了哪些 Skills：展开工具调用明细；或打开 **Chat: Open Customizations** 查看已加载列表。

注意：

- 自动 Skill 选择和多 Skill 组合由模型判断，不保证每次触发。
- 默认 Agent 不会自动切换成 Coding / Review Agent；需要时手动选择。
- 用完 Coding / Review 后，确认 Agent 下拉框已切回默认 Agent，否则后续问题可能继续由当前 Agent 回答。
- 新建 Agent 不会自动成为默认，也不会自动委派。

---

## C. 修改和新增 Skills

### 1. 新建

```text
personal-skills/<name>/SKILL.md     # 个人通用
coding-skills/<name>/SKILL.md       # 编码类
```

可直接复制 `templates/SKILL.md`。

### 2. Frontmatter

```yaml
---
name: my-skill
description: 做什么，以及什么情况下使用。写上用户会说的关键词。
# 仅手动触发时加：
# disable-model-invocation: true
---
```

- `name` 必填，必须与目录名一致，只能小写字母、数字、连字符，≤64 字符。**名称非法会静默加载失败**。
- `description` 必填，≤1024 字符。自动调用主要依赖这句话。
- `disable-model-invocation: true`：只允许 `/` 手动调用，不自动触发。
- `user-invocable: false`：从 `/` 菜单隐藏，但仍可被模型自动调用。
- `argument-hint`：作为斜杠命令时的参数提示。

### 3. 验证是否被发现

输入 `/` 查看菜单，或 **Chat: Open Customizations** 的 Skills 分区。未出现时先重载窗口。

### 4. 更新

直接编辑 `SKILL.md`，新开聊天会话即生效，**不需要复制到任何地方**。

### 5. 删除

删除对应 Skill 目录。删除前确认目标路径在 `personal-skills/` 或 `coding-skills/` 下，避免误删仓库其他内容。

### 6. 是否需要重载

一般不需要；若修改后未生效，执行 **Developer: Reload Window** 并新开会话。

---

## D. 修改和新增 Agents

### 1. 修改 Coding Agent

编辑 `agents/coding.agent.md`。职责、工作流和输出格式在正文；工具权限在 `tools`；允许调用的子代理在 `agents`。

### 2. 修改 Review Agent

编辑 `agents/review.agent.md`。保持 `tools: ['read', 'search']`，不要加入 `edit`、`execute`；这是只读约束的工具白名单声明，运行时效果仍需在界面确认。

### 3. 新增 Agent

在 `agents/` 下创建 `xxx.agent.md`：

```yaml
---
name: My Agent
description: 这个 Agent 做什么、何时使用。
tools: ['read', 'search']
---
正文：角色、步骤、输出格式。
```

### 4. 让 VS Code 发现

已通过 `chat.agentFilesLocations` 指向 `~/personal-agent/agents`，保存文件后重载窗口即可。不要在文件名和 `name` 之间制造不一致。

### 5. 在 Chat 中选择

Chat 顶部的 Agent 下拉框。`name` 就是显示名。

### 6. 判断是否真正生效

- 下拉框能选中该 Agent；
- 选中后应只使用 `tools` 中列出的工具（例如 Review Agent 无法编辑文件）；实际运行时仍需确认；
- **Chat: Open Customizations** 中能看到该文件且无解析错误。

### 7. 新建 Agent 不会自动成为默认

VS Code 始终使用你当前选中的 Agent。新文件只是增加了一个可选项，必须手动选择，或被其他 Agent 作为子代理调用。它不会按问题内容自动切换，也不会自动接管日常问答。

---

## E. 故障排查

### Skills 没有出现在列表中

1. 确认 `~/personal-agent` 存在且指向仓库（junction 或直接目录）：`Get-Item "$HOME\personal-agent"`。
2. 确认设置键名和格式正确：`chat.agentSkillsLocations` 是对象，不是数组；路径以 `~/` 开头、用 `/`。
3. 确认 `chat.useAgentSkills` 为 `true`。
4. 检查 Skill 目录名与 `name` 是否一致、是否全小写连字符。非法名称会静默失败。
5. 确认每个 Skill 目录下是 `SKILL.md`（大写）。
6. 执行 **Developer: Reload Window**。
7. 打开 **Chat: Open Customizations** 查看错误详情。

### Skills 存在但没有自动调用

1. 检查是否设置了 `disable-model-invocation: true`（那样只能手动 `/` 调用）。
2. 检查 `description` 是否写清楚了“做什么 + 何时用”，以及用户可能使用的关键词。
3. 自动调用由模型判断，**不保证每次触发**。任务不相关、描述太泛或与其他 Skill 重叠时都可能不调用。
4. 需要确定执行时，在提示中明确点名：`使用 xxx skill`，或直接输入 `/xxx`。
5. 多个 Skill 的组合同样依赖模型判断，Agent 指令只能提高概率，不能强制。
6. 默认实现是让模型用 `read_file` 读取对应的 `SKILL.md`；Coding Agent 的 `read` 已覆盖。若手动启用了实验设置 `github.copilot.chat.skillTool.enabled`，模型会改用 `skill` 工具，此时需要在 Agent 的 `tools` 中加入 `'skill'`。本工程没有开启该设置。

### Instructions 没有生效

1. VS Code 默认读取 `~/.claude/CLAUDE.md`，要求 `chat.useClaudeMdFile` 为 `true`（1.142.0 默认值就是 `true`）。
2. 确认 `~/.claude/CLAUDE.md` 存在，且内容是你需要的语言版本。本机当前与 `language/claude.md` 逐字节一致。
3. 用户级 Instructions 的默认目录还有 `~/.copilot/instructions` 和 `~/.claude/rules`（后者扫描 `*.md`）；工作区级是 `.github/instructions`（`*.instructions.md`）和 `.claude/rules`（`*.md`）。
4. 仓库内的 `instructions/` 不会被直接扫描：VS Code 只识别 `*.instructions.md`，当前 `instructions/` 下没有这种文件。不要为了直读把三份语言版本同时放进用户目录，否则规则会叠加。
5. Copilot CLI / 云端读取 `~/.copilot/copilot-instructions.md`；本机当前不存在该文件，需要时从 `deploy/copilot/copilot-instructions.md` 手动复制。VS Code 默认 Agent 不依赖它。
6. 修改后新开会话；仍不生效时重载窗口。

### Agent 文件没有被发现

1. 文件名必须是 `*.agent.md`。
2. 确认在 `agents/` 下，且设置指向 `~/personal-agent/agents`。
3. 检查 frontmatter YAML 是否有语法错误（缩进、引号）。
4. **Chat: Open Customizations** 查看解析错误。
5. 重载窗口。

### Agent 无法调用另一个 Agent

1. 调用方的 `tools` 必须包含 `agent`（Coding Agent 已包含）。
2. 被调用方必须在调用方 `agents` 列表中（Coding Agent 已设为 `['Review Agent']`）。
3. 名称区分大小写，必须与对方 `name` 完全一致。
4. 仅 **Local harness** 支持子代理；Copilot harness 使用原生 `task` 工具，行为不同。
5. 普通聊天中需在工具配置里启用子代理（`agent` 工具）；自定义 Agent 已通过 `tools: ['agent']` 声明。
6. 嵌套子代理默认关闭：`chat.subagents.allowInvocationsFromSubagents` 默认 `false`。本工程只用一层（Coding Agent → Review Agent），不需要开启；超过一层的调用会被禁用。
7. 默认 Agent 不会自动切换成自定义 Agent；需要先手动选择 Coding Agent，或直接选择 Review Agent。

### 路径配置错误

- 只支持相对路径或 `~/` 开头；`C:\...`、`/...` 会被拒绝。
- 用正斜杠 `/`，不要用反斜杠。
- 值是 `{ "路径": true }` 对象，不是数组。
- 不要指向单个文件或写通配符。
- 修改后重载窗口。

### 修改文件后未生效

1. 保存文件。
2. **Developer: Reload Window**。
3. 新开一个 Chat 会话（旧会话可能持有启动时的 Skill 列表）。

### 自定义位置完全不生效时的 fallback

如果当前 VS Code 版本不识别 `chat.agentSkillsLocations` / `chat.agentFilesLocations`，改用默认用户目录（VS Code 会自动扫描）：

1. 默认 Skills 目录：`~/.agents/skills`、`~/.copilot/skills`、`~/.claude/skills`。选一个即可，本机这三个目录当前都没有这类 Skill（`~/.agents/skills` 只有无关的 `find-skills`、`microsoft-foundry`）。
2. 默认 Agents 目录：`~/.copilot/agents`、`~/.claude/agents`（本机当前都不存在）。
3. 把 `personal-skills/*`、`coding-skills/*` 复制到选定的 skills 目录，把 `agents/*.agent.md` 复制到选定的 agents 目录。
4. 如需避免重复，可把 `chat.agentSkillsLocations` 中的两个自定义路径和 `chat.agentFilesLocations` 中的自定义路径设为 `false`。
5. 这是手动副本，修改仓库后必须重新复制。

不要复制到 `~/.agents/skills/`：本机该目录已有两个无关的全局 Skill，混入后不易区分。`~/.claude/skills` 和 `~/.claude/agents` 也会被 Claude Code 读取；Skills 可共用，Agents 的 frontmatter 格式不同，不要直接把 `.agent.md` 当作 Claude Agent 使用。

### Copilot 与 Claude Code 表现不同

- VS Code（Local harness）通过本文的 `chat.*` 设置读取仓库；Copilot CLI / 云端不读取编辑器设置。
- Claude Code 读取 `~/.claude/skills/`、`~/.claude/agents/`、`~/.claude/CLAUDE.md`，格式与 VS Code 的 `.agent.md` 不完全相同。
- Claude Code 的 Agent 使用自己的 frontmatter（`name`、`description`、逗号分隔的 `tools`），不要直接把 `.agent.md` 当作 Claude Agent 使用。
- 需要同时支持两个工具时，Skills 可以共享同一份 `SKILL.md`；Agents 需按各自格式分别维护。

---

## 已知限制

1. 本文的设置名、默认目录、Agent 格式和路径规则已在 VS Code 1.142.0 中核实；其他版本可能不同，升级后建议重新确认。
2. 自动 Skill 发现和多 Skill 组合由模型判断，无法保证每次都按预期执行。
3. Review Agent 的只读性由 `tools: ['read', 'search']` 声明（Local harness）；换成其他 harness 时工具名和限制可能不同，仅靠提示词约束不等于安全隔离。运行时是否实际限制工具，仍需在界面验证。
4. `chat.agentSkillsLocations` 的自定义位置主要供 **Local agent harness** 使用；其他 Copilot 客户端不会继承编辑器设置。在 VS Code 编辑器内可正常使用。
5. Instructions 没有直接读取仓库：VS Code 只扫描 `*.instructions.md`（以及 `.claude/rules` 下的 `*.md`），且位置设置不支持绝对路径；Windows 文件符号链接默认需要管理员权限（或开发者模式），因此 `~/.claude/CLAUDE.md` 仍是必要的用户目录副本。`~/.copilot/copilot-instructions.md` 只供 Copilot CLI / 云端使用。
6. 如果同时把 Skills 复制到 `~/.claude/skills/`（供 Claude Code 使用），VS Code 默认也会扫描该目录，可能出现重复条目；可在 `chat.agentSkillsLocations` 中加 `"~/.claude/skills": false` 关闭该来源。`~/.claude/agents` 默认也会被扫描。
7. 自定义 Agent 不会被自动选中；默认 Agent 也不会按问题内容自动切换成 Coding / Review Agent。
8. `chat.subagents.allowInvocationsFromSubagents` 默认 `false`，嵌套子代理默认关闭；本工程只用一层。
