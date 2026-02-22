# PR Review Assistant

多专家并行 PR 评审插件，支持代码质量、安全、性能、架构等维度的专业审查。

## 功能

- **多专家并行评审**：代码质量、安全、性能、架构
- **PR 上下文收集**：自动提取 PR 描述、Issue、链接等背景信息
- **结构化评审报告**：按严重程度分类（CRITICAL/HIGH/MEDIUM/LOW）
- **结果持久化**：自动保存评审报告到本地目录
- **交互式问答**：评审后继续讨论

## 前置要求

1. **GitHub CLI**：已安装并完成认证

   ```bash
   brew install gh
   gh auth login
   gh auth refresh -s repo
   ```

2. **Claude Code**：已安装

   ```bash
   npm install -g @anthropic-ai/claude-code
   ```

## 权限配置

安装插件后，需在项目根目录的 `.claude/settings.json` 中增加以下权限：

```json
{
  "permissions": {
    "allow": [
      "Bash(gh pr view:*)",
      "Bash(gh pr diff:*)",
      "Bash(gh pr view:*)",
      "Bash(gh api:*)",
      "Bash(mkdir:*)",
      "Bash(echo:*)",
      "Bash(tr:*)",
      "Bash(grep:*)",
      "Bash(head:*)",
      "Bash(wc:*)",
      "Read",
      "Write",
      "Edit",
      "Glob",
      "Grep",
      "TodoWrite"
    ]
  }
}
```

## 使用方式

```
/review-pr <pr-url> [--experts=<experts>] [--focus=<areas>]
```

### 参数

| 参数    | 说明                 | 默认值                                      |
|---------|----------------------|---------------------------------------------|
| pr-url  | GitHub PR 链接       | 必需                                        |
| experts | 评审专家列表         | code,security,performance,architecture-reviewer |
| focus   | 重点关注领域         | 全部                                        |

### 示例

```
/review-pr https://github.com/owner/repo/pull/123
/review-pr https://github.com/owner/repo/pull/456 --experts=code,security
/review-pr https://github.com/owner/repo/pull/789 --focus=logic,security
```

## 输出

评审报告保存到 `{project}/{pr-number}_{title-slug}/REVIEW.md`，其中 `{project}` 为仓库名。

## License

MIT
