AI 时代已经来临。

---

### 提示技巧

- 提供上下文
- 清晰的文档，注视，示例代码，命名
- 需求明确，需求拆分，分步骤
- 定义输入输出
- 期望使用的框架
- 反复迭代

---

### Agent


### Agent Skills
- 专业：针对领域特定任务定制功能，无需重复上下文
- 减少重复：创建一次，所有会话中自动使用
- 组合能力：整合多项技能以构建复杂工作流
- 高效加载：仅在需要时加载相关内容至上下文

### MCP
> 让AI模型通过统一的界面使用外部工具和服务。比如，文件操作、数据库或与外部API交互等任务提供工具。

---

- Agent chat
> 理解高层级需求并将其转化为可运行的代码。它们非常适合实现新功能、重构大量代码，或从零开始构建完整应用程序。
- Inline chat
> 适合进行小范围、有针对性的修改，而不会影响整个代码库，例如添加错误处理、重构单个函数或修复错误。
- 自定义说明 at .github/copilot-instructions.md
```
# Project general coding guidelines

## Code Style
- Use semantic HTML5 elements (header, main, section, article, etc.)
- Prefer modern JavaScript (ES6+) features like const/let, arrow functions, and template literals

## Naming Conventions
- Use PascalCase for component names, interfaces, and type aliases
- Use camelCase for variables, functions, and methods
- Prefix private class members with underscore (_)
- Use ALL_CAPS for constants

## Code Quality
- Use meaningful variable and function names that clearly describe their purpose
- Include helpful comments for complex logic
- Add error handling for user inputs and API calls
```
- 自定义 agent at .github/agents
> Code Reviewer.md
```
---
description: 'Review code for quality and adherence to best practices.'
tools: ['usages', 'vscodeAPI', 'problems', 'fetch', 'githubRepo', 'search']
---
# Code Reviewer agent

You are an experienced senior developer conducting a thorough code review. Your role is to review the code for quality, best practices, and adherence to [project standards](../copilot-instructions.md) without making direct code changes.

When reviewing code, structure your feedback with clear headings and specific examples from the code being reviewed.

## Analysis Focus
- Analyze code quality, structure, and best practices
- Identify potential bugs, security issues, or performance problems
- Evaluate accessibility and user experience considerations

## Important Guidelines
- Ask clarifying questions about design decisions when appropriate
- Focus on explaining what should be changed and why
- DO NOT write or suggest specific code changes directly
```

---

### 编写有效提示的诀窍
- 具体明确：详细说明目标任务、所需技术及预期输出格式。
- 添加上下文：使用#提及功能引用文件、符号或上下文变量，如#codebase、#changes或#problems。
- 迭代优化：从简单提示开始，根据响应逐步完善。通过追问提升结果质量。
- 分解复杂任务：避免一次性提出所有要求，将大型任务拆解为可控的小步骤。


---
参考
[Get started with GitHub Copilot in VS Code](https://code.visualstudio.com/docs/copilot/getting-started)
[Prompt examples for chat in VS Code](https://code.visualstudio.com/docs/copilot/chat/prompt-examples)
