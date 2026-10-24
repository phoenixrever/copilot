# personal-agent

个人 Agent 配置工程。仓库是唯一维护源；日常在 VS Code 的 Chat 里直接提问即可，默认 Agent 会遵守全局 Instructions，并按需加载相关 Skill。Coding Agent 和 Review Agent 是可选项，只在需要结构化编码或独立只读审查时手动选择。

**本仓库的维护流程不会写入你的 VS Code 配置。** 首次配置按下文由你自己修改用户 `settings.json`。本文的设置项、默认目录和 Agent 格式已在 VS Code 1.142.0 / 内置 Copilot Chat 0.70.0 中核实。

## 目录结构

```text
.
├── personal-skills/     个人通用 Skills（源文件）
├── coding-skills/       编码与技术类 Skills（源文件）
├── agents/              自定义 Agents（VS Code 格式）
│   ├── coding.agent.md    编码代理（可选）
│   └── review.agent.md    只读审查代理（可选）
├── instructions/        Instructions 源文件与笔记（VS Code 不直接扫描）
├── language/            Instructions 多语言版本
│   ├── claude.md          英文版（当前生效源）
│   ├── claude-zh.md       中文版
│   └── claude-jp.md       日文版
├── templates/SKILL.md   新建 Skill 的模板
├── docs/VSCODE-SETUP.md VS Code 全局使用手顺与排错
├── THIRD-PARTY-SKILLS.md  第三方 Skill 来源与许可
└── deploy/              需要放进用户目录的客户端副本
    ├── claude/
    │   ├── CLAUDE.md            → ~/.claude/CLAUDE.md
    │   └── skills/              → ~/.claude/skills/
    └── copilot/
        ├── copilot-instructions.md → ~/.copilot/copilot-instructions.md
        └── skills/                 → ~/.copilot/skills/
```

## 设计原则

- **一个默认入口**：VS Code Chat 的默认 Agent。普通问题直接问，不需要先选 Agent。
- **一个维护源**：Skills、Agents、Instructions 的源文件都在本仓库。
- **按需调用**：Skills 由模型根据 `description` 自动选择；Coding / Review Agent 只在需要时手动选择。
- **尽量不复制**：Skills 和 Agents 由 VS Code 直接读取仓库（`~/personal-agent`，可用 junction）；只有 Instructions 因 VS Code 的文件名和权限限制保留一份用户目录副本。
- **配置自理**：本仓库的维护流程不写入任何 VS Code 配置文件。

## 日常使用（默认入口）

1. 用 VS Code 打开任意项目。
2. 按 `Ctrl+Alt+I` 打开 Chat。
3. 直接输入问题。不需要选择 Agent，也不需要指定 Skill。
4. 默认 Agent 会：
   - 遵守 `~/.claude/CLAUDE.md` 中的全局规则（`chat.useClaudeMdFile` 默认开启）。
   - 按 Skill 的 `description` 自动加载相关 Skill；自动选择由模型判断，不保证每次触发。
5. 需要确定使用某个 Skill 时：
   - 在问题中点名，例如 `使用 doc-coauthoring skill 帮我写设计文档`；
   - 或直接输入 `/doc-coauthoring`。
6. 只有以下情况才需要切换 Agent：
   - 需要结构化编码流程（理解需求 → 最小修改 → 补测试 → 实际验证）：选择 **Coding Agent**。
   - 需要独立只读审查：选择 **Review Agent**，或在 Coding Agent 回答后点 **交由 Review Agent 审查**。
7. Coding Agent / Review Agent 的工具限制、子代理和 handoff 只在 Chat 的 **Local** session target 生效；日常普通问答不需要关心这一点。

> 默认 Agent 不会自动切换成 Coding Agent 或 Review Agent。VS Code 使用你当前选中的 Agent；自定义 Agent 只是下拉框里的可选项。用完 Coding / Review 后，确认下拉框已切回默认 Agent，否则后续问题可能继续由当前 Agent 回答。

## 首次配置（由你自己修改）

### 1. 决定仓库位置

两种方式二选一，不要同时做。

**方式 A（最简单，推荐）**：把仓库放在用户主目录下：

```text
C:\Users\<你的用户名>\personal-agent
```

这样设置里的 `~/personal-agent/...` 直接指向仓库，不需要 junction。

**方式 B（仓库在别处）**：当前仓库在 `D:\code\copilot`，用目录 junction 把它映射到 `~/personal-agent`。普通用户即可执行，不需要管理员权限：

```powershell
New-Item -ItemType Junction -Path "$HOME\personal-agent" -Target "D:\code\copilot"
```

验证：

```powershell
Get-Item "$HOME\personal-agent" | Select-Object FullName, LinkType, Target
```

应显示 `LinkType: Junction`，`Target` 指向实际仓库。

> 为什么需要这一步：VS Code 的 `chat.agentSkillsLocations` / `chat.agentFilesLocations` 只接受相对路径或 `~/` 开头的路径，不接受 `C:\...` 绝对路径。junction 让 `~/personal-agent/...` 解析到实际仓库。

### 2. 修改用户 settings.json

命令面板（`Ctrl+Shift+P`）→ **Preferences: Open User Settings (JSON)**。

加入以下两项（已有同名键时合并，不要重复定义）：

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

- 路径用正斜杠 `/`，以 `~/` 开头；不要用反斜杠、盘符或通配符。
- 值是对象映射 `{ "路径": true }`，不是数组。
- 写到**包含 Skill / Agent 的目录**，VS Code 会向下查找；不要写到单个文件。
- `chat.useAgentSkills` 和 `chat.useClaudeMdFile` 在 VS Code 1.142.0 中默认就是 `true`，**不需要额外添加**；只有你之前手动关闭过，才需要把它们设回 `true`。
- 不需要添加 `chat.instructionsFilesLocations`：当前 Instructions 走 `~/.claude/CLAUDE.md`（见下一节）。
- VS Code 还会自动扫描默认 Skills 目录。本机 `~/.agents/skills` 已有两个无关 Skill；如果不想在 `/` 菜单看到它们，可在同一对象中加 `"~/.agents/skills": false`（可选）。

保存后执行 **Developer: Reload Window**，再新开一个 Chat 会话。

### 3. 验证是否生效

1. Chat 输入 `/`，应出现 `doc-coauthoring`。
2. Agent 下拉框应出现 **Coding Agent** 和 **Review Agent**。
3. 命令面板 → **Chat: Open Customizations**，Skills / Agents 分区能看到条目且没有解析错误。
4. 在默认 Agent 下问一个普通问题，回答应遵守中文、先结论等全局规则（来自 `~/.claude/CLAUDE.md`）。
5. 问一个文档写作任务，展开工具调用明细，观察是否读取了 `coding-skills/doc-coauthoring/SKILL.md`。自动加载不保证每次触发。

任一项不生效时，先看 [docs/VSCODE-SETUP.md](docs/VSCODE-SETUP.md) 的排查部分。

## Instructions 如何生效

- **默认入口**：`~/.claude/CLAUDE.md`。VS Code 的 `chat.useClaudeMdFile` 默认为 `true`，默认 Agent 会读取该文件；不需要选择 Agent。
- **源文件**：`language/claude.md`。本机 `~/.claude/CLAUDE.md` 与它当前逐字节一致。
- **修改流程**：改 `language/claude.md`，手动复制到 `~/.claude/CLAUDE.md`，新开 Chat 会话生效。Windows 不区分大小写，实际文件名 `claude.md` 也可。
- **语言版本**：`language/claude-zh.md`（中文）、`language/claude-jp.md`（日文）。只把需要的一份放到 `~/.claude/CLAUDE.md`，不要同时放多份，否则规则会叠加。
- **Copilot CLI / 云端**：读取 `~/.copilot/copilot-instructions.md`。本机当前没有这个文件；需要时从 `deploy/copilot/copilot-instructions.md` 手动复制。VS Code 的默认 Agent 不依赖它。
- **`instructions/`**：Instructions 源文件和笔记，VS Code 不会直接扫描。VS Code 只识别 `*.instructions.md`；当前仓库没有配置该目录。保留原内容，不影响日常使用。

为什么 Instructions 没有像 Skills 一样直读：VS Code 的用户级 Instructions 原生位置是 `~/.claude/CLAUDE.md`、`~/.copilot/instructions`、`~/.claude/rules`（以及工作区 `.github/instructions`、`.claude/rules`）。仓库里的 `language/claude.md` 不是 `*.instructions.md`，无法通过 `chat.instructionsFilesLocations` 直接扫描；Windows 文件符号链接默认又需要管理员权限（或开发者模式）。因此保留一份 `~/.claude/CLAUDE.md` 副本，这是本工程唯一的必要副本。

## Skills

### 分类与发现

- `personal-skills/`：个人通用。
- `coding-skills/`：编码、技术、文档类。
- VS Code 通过 `chat.agentSkillsLocations` 直接扫描这两个目录，不需要复制。
- 每个 Skill 一个目录，入口文件必须是 `SKILL.md`（大写）。

### 新建 Skill

1. 复制 `templates/SKILL.md` 到 `personal-skills/<name>/SKILL.md` 或 `coding-skills/<name>/SKILL.md`。
2. `name` 必须与目录名一致，只能小写字母、数字、连字符，≤64 字符；非法名称会静默加载失败。
3. `description` 写清楚“做什么 + 什么时候用 + 用户可能说的关键词”，自动选择主要依赖这句话。
4. 保存后新开 Chat 会话；必要时重载窗口。
5. 需要供 Claude Code / Copilot CLI 使用时，再复制到 `deploy/claude/skills/`、`deploy/copilot/skills/`。

### 触发方式

| 模式 | frontmatter | 效果 |
| --- | --- | --- |
| 自动 / 混合 | 不写 `disable-model-invocation` | 可由模型自动调用，也可 `/name` 手动调用 |
| 仅手动 | `disable-model-invocation: true` | 只能 `/name` 调用，不自动触发 |

自动调用由模型判断，不保证每次触发。需要确定执行时，在提示中点名或直接输入 `/name`。

### 更新与删除

- 更新：直接编辑 `SKILL.md`，新开会话生效；不需要复制到任何地方。
- 删除：删除对应 Skill 目录。删除前确认路径在 `personal-skills/` 或 `coding-skills/` 下；这是破坏性操作，先确认。

### 第三方 Skills

- 现有 `coding-skills/doc-coauthoring` 来源：<https://github.com/anthropics/skills>，许可证 Apache 2.0；记录在 `THIRD-PARTY-SKILLS.md`。
- 第三方内容视为不可信；不要运行其中未经审查的脚本。
- 安装：浅克隆来源仓库，复制对应目录到 `coding-skills/`，再复制到 `deploy/` 两个目录；更新 `THIRD-PARTY-SKILLS.md`。

## 可选 Agents

### Coding Agent

- 文件：`agents/coding.agent.md`。
- 用途：需要结构化编码流程时手动选择；负责需求理解、最小修改、补测试、实际运行验证。
- 工具：`read`、`search`、`edit`、`execute`、`agent`、`todos`、`web`。
- 复杂或高风险修改时，可按正文规则调用 Review Agent 子代理；简单修改可跳过。

### Review Agent

- 文件：`agents/review.agent.md`。
- 用途：独立只读审查，手动选择，或由 Coding Agent / handoff 触发。
- 只读约束：`tools: ['read', 'search']`，没有编辑和执行权限。这是 Local harness 的工具白名单声明，不只是提示词；实际界面效果仍需验证。
- 输出：按严重程度排列，区分已确认缺陷、潜在风险、一般建议，不虚构问题。

### 协作方式

- Coding Agent 回答后，点击 **交由 Review Agent 审查**（handoff，`send: false`，发送前可确认）。
- Review Agent 回答后，点击 **交由 Coding Agent 修复**。
- 也可以直接在 Agent 下拉框切换。

### 是否自动委派

没有配置“按问题内容自动切换 Agent”。VS Code 不会自动把默认 Agent 换成 Coding / Review；自定义 Agent 只是可选项。Coding Agent 在自身流程中调用 Review Agent 子代理属于可选 Agent 内部行为，只在已选择 Coding Agent 时发生；嵌套子代理默认关闭（`chat.subagents.allowInvocationsFromSubagents` 默认 `false`），本工程只用一层。

### 新增 Agent

1. 在 `agents/` 下创建 `xxx.agent.md`。
2. frontmatter 至少写 `name`、`description`、`tools`。
3. 保存后重载窗口，在 Agent 下拉框选择；新 Agent 不会自动成为默认。
4. 验证：能选中；应只使用 `tools` 中列出的工具；**Chat: Open Customizations** 无解析错误。

## Claude Code / Copilot CLI

VS Code 用户不需要复制。以下仅用于读取固定用户目录的客户端：

| 复制 `deploy/` 中的 | 到 |
| --- | --- |
| `claude/CLAUDE.md` | `~/.claude/CLAUDE.md` |
| `claude/skills/*` | `~/.claude/skills/` |
| `copilot/copilot-instructions.md` | `~/.copilot/copilot-instructions.md` |
| `copilot/skills/*` | `~/.copilot/skills/` |

- `deploy/` 是副本，不要直接编辑；修改源文件后重新复制。
- 默认部署副本：`language/claude.md` → `deploy/claude/CLAUDE.md`（英语版）；中文版用 `language/claude-zh.md`。
- `deploy/copilot/copilot-instructions.md` 是日文版；本机 `~/.copilot/copilot-instructions.md` 尚未创建。
- Claude Code 的 Agent 格式与 VS Code 的 `.agent.md` 不同；Agents 需按各自格式分别维护。

## 实际验证结果

已在 VS Code **1.142.0**、内置 Copilot Chat **0.70.0**、Windows 11 上核实：

- `chat.useAgentSkills` 默认 `true`；`chat.useClaudeMdFile` 默认 `true`。
- 默认 Skills 目录：工作区 `.agents/skills`、`.github/skills`、`.claude/skills`；用户 `~/.agents/skills`、`~/.copilot/skills`、`~/.claude/skills`。
- 默认 Agents 目录：工作区 `.github/agents`、`.claude/agents`；用户 `~/.copilot/agents`、`~/.claude/agents`。
- 默认 Instructions 目录：工作区 `.github/instructions`、`.claude/rules`；用户 `~/.copilot/instructions`、`~/.claude/rules`；另有 `~/.claude/CLAUDE.md`。
- 自定义位置只接受相对路径或 `~/` 路径，拒绝盘符、前导 `/`、反斜杠和通配符。
- `agents/*.agent.md` 的 frontmatter 可解析；`tools` 工具声明和 `handoffs` 字段完整。
- `~/.claude/CLAUDE.md` 与 `language/claude.md` 逐字节一致。
- `deploy/` 中的 Claude/Copilot 指令和 Skill 副本与源文件逐字节一致。
- junction `C:\Users\phoen\personal-agent` → `D:\code\copilot` 存在，三个配置路径可达。
- 本仓库的 `README.md`、`docs/VSCODE-SETUP.md` 内部链接可达。

**未实际执行**（需要你在界面操作）：

- 重载窗口后的 `/` 菜单是否列出 `doc-coauthoring`。
- Agent 下拉框是否显示 Coding Agent / Review Agent。
- 默认 Agent 是否真的自动加载了某个 Skill。
- Review Agent 的工具白名单是否在运行时生效。
- handoff 按钮点击、Coding Agent 调用 Review Agent 子代理的实际效果。

这些行为依赖界面和模型，配置层无法代替实际点按验证。

## 故障排查

1. **Skill 不出现**：确认 `~/personal-agent` 指向仓库；`chat.agentSkillsLocations` 是对象、路径 `~/` 开头、正斜杠；`chat.useAgentSkills` 为 true；目录名与 `name` 一致；入口是 `SKILL.md`；重载窗口；**Chat: Open Customizations** 看错误。
2. **Skill 不自动调用**：检查是否 `disable-model-invocation: true`；`description` 是否写清触发条件；自动调用不保证，必要时点名或 `/name`。
3. **Instructions 不生效**：确认 `~/.claude/CLAUDE.md` 存在且 `chat.useClaudeMdFile` 为 true；新开会话；检查是否被其他规则覆盖。
4. **Agent 不出现**：文件名 `*.agent.md`；在 `agents/` 下；frontmatter YAML 合法；**Chat: Open Customizations** 看错误；重载窗口。
5. **Agent 不能调用另一个**：调用方 `tools` 包含 `agent`；被调用方在 `agents` 列表中；名称大小写一致；仅 Local harness；嵌套子代理默认关闭。
6. **改了没生效**：保存 → 重载窗口 → 新开会话。
7. **不想用自定义位置**：把 Skills 复制到 `~/.copilot/skills` 或 `~/.claude/skills`，Agents 复制到 `~/.copilot/agents` 或 `~/.claude/agents`，由 VS Code 默认扫描；这是手动副本，修改后要重新复制。注意 `~/.agents/skills` 已有无关 Skill，不要混入。

完整排错见 [docs/VSCODE-SETUP.md](docs/VSCODE-SETUP.md)。

## 安全边界

以下操作执行前必须人工确认，Agent 自动触发不等于自动授权：

- 删除、覆盖 Skill、Agent 或 Instruction 文件（尤其用户目录中已手动修改的文件）。
- 运行第三方 Skill 中的脚本、安装依赖。
- 提交、推送、部署，除非已明确授权。

Review Agent 的只读性在 VS Code Local harness 中由工具白名单声明；实际运行时是否完全限制仍需验证，不要只依赖提示词。

## 已知限制

1. 自动 Skill 选择和多 Skill 组合由模型判断，无法保证每次都按预期执行。
2. 自定义 Agent 不会被自动选中；需要手动选择，或由已选中的 Coding Agent 调用一层子代理。
3. 自定义 Agent 的工具限制、子代理和 handoff 仅在 VS Code Local harness 生效。
4. `chat.subagents.allowInvocationsFromSubagents` 默认 `false`，嵌套子代理默认关闭；本工程只用一层。
5. Instructions 保留一份 `~/.claude/CLAUDE.md` 副本；修改 `language/claude.md` 后需手动复制。
6. `deploy/` 与源文件是手动副本关系，修改后必须重新复制。
7. Copilot CLI / 云端不读取 VS Code 的 `chat.*` 设置；行为可能不同。
8. `instructions/readme.md` 是旧笔记，其中 `github.copilot.chat.codeGeneration.instructions` 在 VS Code 1.102 起已弃用。
