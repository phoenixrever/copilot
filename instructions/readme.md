### url

https://github.com/github/awesome-copilot/blob/main/skills/github-copilot-starter/SKILL.md

###

0. ~/.copilot/copilot-instructions.md
1. .github/copilot-instructions.md
2. .github/instructions/ Directory
3. .github/skills/ Directory
4. .github/agents/ Directory
5. .github/workflows/ Directory (only if user uses GitHub Actions)

## 8. 用户级 Instructions（VS Code）

不在仓库里，而是在个人 VS Code 配置中。

配置项：

JSON
github.copilot.chat.codeGeneration.instructions

或

JSON
Github > Copilot > Chat: Instructions File

可以指定一个全局 Markdown 文件。

例如：

Plain Text
~/copilot/my-instructions.md

所有项目都会生效。
