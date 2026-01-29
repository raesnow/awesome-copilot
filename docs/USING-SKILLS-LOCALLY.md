# 在本地 Copilot 开发中使用 Agent Skills 指南

本文档介绍如何将 Awesome GitHub Copilot 仓库中的 Skills 应用到你的本地 GitHub Copilot 插件开发中。

## 目录

- [什么是 Agent Skills](#什么是-agent-skills)
- [Skills 的结构](#skills-的结构)
- [如何在本地使用 Skills](#如何在本地使用-skills)
- [Skills 使用示例](#skills-使用示例)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)

---

## 什么是 Agent Skills

Agent Skills 是自包含的文件夹，包含指令和捆绑资源，用于增强 AI 在特定任务上的能力。每个 Skill 基于 [Agent Skills 规范](https://agentskills.io/specification)，包含一个 `SKILL.md` 文件，其中有详细的指令，供 Agent 按需加载。

**Skills 与其他资源的区别：**

| 资源类型 | 说明 | 使用场景 |
|---------|------|---------|
| **Prompts** | 一次性任务提示 | 单次代码生成或问题解决 |
| **Instructions** | 编码规范和最佳实践 | 应用于特定文件模式 |
| **Agents** | 专门化的 Copilot 角色 | 特定领域的对话助手 |
| **Skills** | 包含指令和资源的文件夹 | 复杂、可重复的工作流，需要捆绑资源 |

---

## Skills 的结构

每个 Skill 是 `skills/` 目录下的一个文件夹，结构如下：

```
skills/
└── skill-name/
    ├── SKILL.md          # 必需：包含 frontmatter 和指令
    ├── references/       # 可选：参考文档
    │   ├── guide.md
    │   └── examples.md
    ├── scripts/          # 可选：辅助脚本
    │   └── helper.ps1
    ├── examples/         # 可选：示例代码
    │   └── sample.bicep
    └── assets/           # 可选：其他资源
        └── template.md
```

### SKILL.md 文件格式

每个 SKILL.md 文件必须包含 frontmatter 元数据：

```markdown
---
name: skill-name
description: '技能的简短描述（10-1024 字符）'
license: MIT
allowed-tools: Bash, FileSystem
---

# Skill 标题

## 概述
描述这个 skill 的功能...

## 使用场景
说明何时使用这个 skill...

## 使用方法
具体的指令和步骤...
```

### 常用 Frontmatter 字段

| 字段 | 必需 | 说明 |
|-----|------|------|
| `name` | ✅ | Skill 名称，小写，连字符分隔，需与文件夹名匹配 |
| `description` | ✅ | 技能描述，10-1024 字符 |
| `license` | ❌ | 许可证类型（如 MIT） |
| `allowed-tools` | ❌ | 允许使用的工具列表 |

---

## 如何在本地使用 Skills

### 方法一：直接复制 Skill 文件夹

最简单的方式是将整个 skill 文件夹复制到你的项目中：

1. **选择需要的 Skill**
   
   浏览 `skills/` 目录，选择适合你需求的 skill。

2. **复制到本地**
   
   将整个 skill 文件夹复制到你项目的 `.github/copilot/skills/` 目录：

   ```bash
   # 创建目录结构
   mkdir -p .github/copilot/skills
   
   # 复制 skill（以 git-commit 为例）
   cp -r path/to/awesome-copilot/skills/git-commit .github/copilot/skills/
   ```

3. **在 Copilot Chat 中引用**
   
   在 VS Code 的 Copilot Chat 中，你可以直接引用 skill：
   
   ```
   @workspace 请使用 git-commit skill 帮我提交这些更改
   ```

### 方法二：使用 MCP Server（推荐）

使用 Awesome Copilot 提供的 MCP Server 可以直接搜索和安装 skills：

1. **配置 MCP Server**
   
   在 VS Code 的 `settings.json` 中添加：

   ```json
   {
     "github.copilot.chat.mcpServers": {
       "awesome-copilot": {
         "type": "stdio",
         "command": "docker",
         "args": [
           "run",
           "-i",
           "--rm",
           "ghcr.io/microsoft/mcp-dotnet-samples/awesome-copilot:latest"
         ]
       }
     }
   }
   ```

2. **通过 Copilot Chat 搜索和安装**
   
   ```
   @mcp awesome-copilot 搜索 git commit 相关的 skill
   ```

### 方法三：创建自定义 Skill

你也可以基于现有模板创建自己的 skill：

1. **使用 make-skill-template**
   
   参考 `skills/make-skill-template/SKILL.md` 中的指南创建新 skill。

2. **创建目录结构**
   
   ```bash
   mkdir -p .github/copilot/skills/my-skill
   touch .github/copilot/skills/my-skill/SKILL.md
   ```

3. **编写 SKILL.md**
   
   ```markdown
   ---
   name: my-skill
   description: '我的自定义 skill 描述'
   ---
   
   # My Skill
   
   ## 概述
   这个 skill 用于...
   
   ## 使用方法
   步骤...
   ```

---

## Skills 使用示例

### 示例 1：使用 git-commit Skill

`git-commit` skill 帮助你按照 Conventional Commits 规范创建提交：

**Skill 位置：** `skills/git-commit/`

**使用方式：**

```
@workspace 使用 git-commit skill，帮我分析当前的更改并生成合适的 commit message
```

**Skill 功能：**
- 自动检测变更类型（feat, fix, docs 等）
- 分析变更范围（scope）
- 生成符合规范的提交信息
- 支持智能文件暂存

### 示例 2：使用 refactor Skill

`refactor` skill 帮助你进行代码重构：

**Skill 位置：** `skills/refactor/`

**使用方式：**

```
@workspace 使用 refactor skill，帮我重构这个函数，它太长了需要拆分
```

**Skill 功能：**
- 识别代码异味（Code Smells）
- 提取方法/类
- 改进类型安全
- 应用设计模式

### 示例 3：使用 webapp-testing Skill

`webapp-testing` skill 用于测试 Web 应用：

**Skill 位置：** `skills/webapp-testing/`

**使用方式：**

```
@workspace 使用 webapp-testing skill，帮我测试这个登录功能
```

**Skill 功能：**
- 使用 Playwright 进行浏览器自动化
- 验证前端功能
- 捕获截图
- 查看浏览器日志

### 示例 4：使用 copilot-sdk Skill

`copilot-sdk` skill 帮助你使用 GitHub Copilot SDK 构建应用：

**Skill 位置：** `skills/copilot-sdk/`

**使用方式：**

```
@workspace 使用 copilot-sdk skill，帮我创建一个使用 Copilot SDK 的 TypeScript 应用
```

**Skill 功能：**
- 多语言支持（TypeScript, Python, Go, .NET）
- 自定义工具定义
- 流式响应处理
- MCP Server 集成

### 示例 5：使用带有捆绑资源的 Skill

某些 skills 包含额外的参考资料和脚本。例如 `appinsights-instrumentation` skill：

**Skill 位置：** `skills/appinsights-instrumentation/`

**目录结构：**
```
appinsights-instrumentation/
├── SKILL.md
├── LICENSE.txt
├── examples/
│   └── appinsights.bicep
├── references/
│   ├── ASPNETCORE.md
│   ├── AUTO.md
│   ├── NODEJS.md
│   └── PYTHON.md
└── scripts/
    └── appinsights.ps1
```

**使用方式：**

```
@workspace 使用 appinsights-instrumentation skill，帮我为我的 Node.js 应用添加 Azure Application Insights
```

---

## 仓库中可用的 Skills 列表

| Skill 名称 | 说明 | 捆绑资源 |
|-----------|------|---------|
| [agentic-eval](../skills/agentic-eval/) | AI 代理输出评估和改进模式 | 无 |
| [appinsights-instrumentation](../skills/appinsights-instrumentation/) | Azure App Insights 遥测配置 | 示例、参考、脚本 |
| [azure-deployment-preflight](../skills/azure-deployment-preflight/) | Bicep 部署预检验证 | 参考文档 |
| [azure-devops-cli](../skills/azure-devops-cli/) | Azure DevOps CLI 资源管理 | 无 |
| [azure-resource-visualizer](../skills/azure-resource-visualizer/) | Azure 资源架构图生成 | 资源、模板 |
| [azure-role-selector](../skills/azure-role-selector/) | Azure RBAC 角色选择指南 | 许可证 |
| [azure-static-web-apps](../skills/azure-static-web-apps/) | Azure Static Web Apps 部署 | 无 |
| [chrome-devtools](../skills/chrome-devtools/) | Chrome DevTools 浏览器自动化 | 无 |
| [copilot-sdk](../skills/copilot-sdk/) | GitHub Copilot SDK 应用开发 | 无 |
| [gh-cli](../skills/gh-cli/) | GitHub CLI 完整参考 | 无 |
| [git-commit](../skills/git-commit/) | Conventional Commits 智能提交 | 无 |
| [github-issues](../skills/github-issues/) | GitHub Issues 管理 | 模板参考 |
| [image-manipulation-image-magick](../skills/image-manipulation-image-magick/) | ImageMagick 图像处理 | 无 |
| [legacy-circuit-mockups](../skills/legacy-circuit-mockups/) | 面包板电路图生成 | 大量参考文档 |
| [make-skill-template](../skills/make-skill-template/) | 创建新 Skill 模板 | 无 |
| [mcp-cli](../skills/mcp-cli/) | MCP Server CLI 接口 | 无 |
| [microsoft-code-reference](../skills/microsoft-code-reference/) | Microsoft API 参考查询 | 无 |
| [microsoft-docs](../skills/microsoft-docs/) | Microsoft 官方文档查询 | 无 |
| [nuget-manager](../skills/nuget-manager/) | NuGet 包管理 | 无 |
| [plantuml-ascii](../skills/plantuml-ascii/) | PlantUML ASCII 图表生成 | 无 |
| [prd](../skills/prd/) | 产品需求文档生成 | 无 |
| [refactor](../skills/refactor/) | 代码重构指南 | 无 |
| [scoutqa-test](../skills/scoutqa-test/) | ScoutQA Web 应用测试 | 无 |
| [snowflake-semanticview](../skills/snowflake-semanticview/) | Snowflake 语义视图 | 无 |
| [vscode-ext-commands](../skills/vscode-ext-commands/) | VS Code 扩展命令开发 | 无 |
| [vscode-ext-localization](../skills/vscode-ext-localization/) | VS Code 扩展本地化 | 无 |
| [web-design-reviewer](../skills/web-design-reviewer/) | Web 设计审查和修复 | 参考文档 |
| [webapp-testing](../skills/webapp-testing/) | Web 应用 Playwright 测试 | 测试脚本 |
| [workiq-copilot](../skills/workiq-copilot/) | WorkIQ M365 Copilot 数据查询 | 无 |

---

## 最佳实践

### 1. 选择合适的 Skill

- 仔细阅读 skill 的 description，确保它符合你的需求
- 检查 skill 是否有特定的前置条件（如需要安装特定工具）
- 查看 skill 的 `allowed-tools` 字段了解它使用哪些工具

### 2. 组织项目中的 Skills

```
your-project/
├── .github/
│   └── copilot/
│       └── skills/
│           ├── git-commit/
│           ├── refactor/
│           └── your-custom-skill/
├── src/
└── ...
```

### 3. 定制化 Skill

如果现有 skill 不完全满足需求，可以：

1. 复制 skill 到本地
2. 修改 SKILL.md 中的指令
3. 添加或修改参考文档
4. 更新 description 以反映你的定制

### 4. 在团队中共享 Skills

- 将定制的 skills 提交到项目仓库
- 在 README 中说明可用的 skills
- 考虑为通用 skills 贡献回 awesome-copilot 仓库

### 5. 与其他资源配合使用

Skills 可以与 prompts、instructions 和 agents 配合使用：

```
# 示例：结合使用
@workspace 使用 git-commit skill 和我们项目的代码规范 instructions，
帮我提交当前的更改
```

---

## 常见问题

### Q: Skill 和 Instruction 有什么区别？

**Instruction** 是应用于特定文件模式的编码规范，会自动应用。

**Skill** 是更复杂的任务指南，包含工作流程、示例和可能的捆绑资源，需要明确引用。

### Q: 如何知道 Skill 是否有捆绑资源？

查看 skill 文件夹的目录结构，或者查看 [docs/README.skills.md](./README.skills.md) 中的 "捆绑资源" 列。

### Q: Skill 的 allowed-tools 字段是什么意思？

这表示该 skill 设计用于与特定工具配合使用。例如 `allowed-tools: Bash` 表示 skill 可能需要执行 shell 命令。

### Q: 如何创建自己的 Skill？

1. 参考 `skills/make-skill-template/SKILL.md`
2. 创建符合规范的目录结构
3. 编写 SKILL.md 文件，包含必需的 frontmatter
4. 可选：添加参考文档、脚本或其他资源

### Q: 如何让 Copilot 自动发现我的 Skills？

将 skills 放在项目的 `.github/copilot/skills/` 目录下，Copilot 会自动发现它们。

### Q: Skills 支持哪些编程语言？

Skills 本身是语言无关的。每个 skill 的适用范围取决于其内容。某些 skills（如 `copilot-sdk`）明确支持多种语言（TypeScript, Python, Go, .NET）。

---

## 更多资源

- [Agent Skills 规范](https://agentskills.io/specification)
- [Awesome GitHub Copilot README](../README.md)
- [Skills 完整列表](./README.skills.md)
- [贡献指南](../CONTRIBUTING.md)
- [MCP Server 文档](https://developer.microsoft.com/blog/announcing-awesome-copilot-mcp-server)

---

如有问题或建议，欢迎在项目的 GitHub Issues 中提出。
