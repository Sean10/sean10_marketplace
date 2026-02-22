---
name: pr-review-workflow
description: 一套完整的 PR 评审工作流方法论，包括多专家并行评审、结果汇总、问题追踪和交互式问答。支持代码质量、安全、性能、架构等多维度评审。
---

# PR 评审工作流

此工作流提供了一套完整的 GitHub PR 评审方法论，支持多专家并行评审和交互式问答。

## 何时使用此技能

当需要进行以下操作时，应使用此技能：
- 对 GitHub PR 进行多维度专业评审
- 识别代码中的质量、安全、性能、架构问题
- 生成结构化的评审报告
- 与用户进行评审结果的交互式讨论

## 工作流程概述

```
PR URL 输入
     │
     ▼
┌─────────────────┐
│  信息收集阶段    │  gh pr view, gh pr diff, gh pr files
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  并行专家评审    │  代码专家 / 安全专家 / 性能专家 / 架构专家
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  结果汇总阶段    │  分类、排序、生成报告
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  交互式问答      │  用户追问、持续讨论
└─────────────────┘
```

## 核心步骤

### 步骤 1: PR 信息获取

使用 `gh` CLI 工具获取 PR 相关信息：

```bash
# 获取 PR 基本信息
gh pr view <pr-url> --json title,body,author,state,baseRefName,headRefName

# 获取 PR diff
gh pr diff <pr-url>

# 获取变更文件列表
gh pr files <pr-url>

# 获取 PR 评论
gh pr comment list <pr-url>
```

### 步骤 2: 专家评审分配

根据评审需求，分配相应的专家子代理：

| 专家 | 评审维度 | 工具权限 |
|------|----------|----------|
| code-reviewer | 代码质量、逻辑正确性 | Read, Bash, Glob, Grep |
| security-expert | 安全漏洞、认证授权 | Read, Bash, Glob, Grep |
| performance-expert | 性能问题、资源管理 | Read, Bash, Glob, Grep |
| architecture-reviewer | 设计模式、架构一致性 | Read, Bash, Glob, Grep, WebFetch |

### 步骤 3: 并行评审执行

使用 `/agents` 命令启动并行评审：

```markdown
请使用以下专家对这个 PR 进行评审:
- code-reviewer: 重点关注代码质量和逻辑正确性
- security-expert: 重点关注安全漏洞
- performance-expert: 重点关注性能问题
- architecture-reviewer: 重点关注设计合理性

PR 信息:
- URL: {pr_url}
- 变更: {changes_summary}
```

### 步骤 4: 结果汇总

收集各专家的评审意见，按以下维度汇总：

1. **严重程度分类**: CRITICAL > HIGH > MEDIUM > LOW
2. **按文件排序**: 按问题数量和严重程度排序
3. **按类型分组**: 代码、安全、性能、架构
4. **生成综合报告**: 包含评分、问题清单、改进建议

## 输出格式

### 评审报告模板

```markdown
# PR 评审报告

## 基本信息
- **PR**: #{number} - {title}
- **作者**: {author}
- **状态**: {state}
- **变更**: {files_changed} 文件, {additions} 行增加, {deletions} 行删除

## 综合评分

| 维度 | 评分 (1-5) | 说明 |
|------|------------|------|
| 代码质量 | {score} | {reason} |
| 安全性 | {score} | {reason} |
| 性能 | {score} | {reason} |
| 架构 | {score} | {reason} |
| **综合** | **{overall}** | |

## 关键问题

### CRITICAL

1. **问题标题**
   - 位置: {file}:{line}
   - 影响: {impact}
   - 建议: {suggestion}

### HIGH

...

## 各维度详情

### 代码质量
{details}

### 安全
{details}

### 性能
{details}

### 架构
{details}

## 改进建议

### 必须修复
1. ...

### 强烈建议
1. ...

### 可选优化
1. ...
```

## 交互式问答

评审完成后，用户可以继续提问。问答模式支持：

1. **问题澄清**: "这个问题的具体原因是什么?"
2. **方案讨论**: "有没有其他更好的实现方式?"
3. **影响评估**: "这个问题的影响范围有多大?"
4. **学习请教**: "能否解释一下这个设计模式?"

### 问答处理流程

```
用户提问
     │
     ▼
┌─────────────────┐
│  问题分类       │  技术问题 / 评审细节 / 改进建议
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  上下文检索     │  读取原始代码、评审记录
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  答案生成       │  基于事实和专业知识回答
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  持续对话       │  支持多轮问答
└─────────────────┘
```

## 质量保证

### 评审质量检查清单

- [ ] 每个问题都有具体的代码位置
- [ ] 每个问题都有清晰的严重程度
- [ ] 每个问题都有具体的修复建议
- [ ] 评审意见客观、专业、有建设性
- [ ] 无遗漏重要的评审维度

### 报告质量检查清单

- [ ] 报告结构清晰、格式统一
- [ ] 评分有理有据
- [ ] 问题分类准确
- [ ] 建议可操作
- [ ] 无主观臆断

## 最佳实践

### 1. 渐进式评审

对于大型 PR，建议分阶段评审：

1. **快速概览**: 了解变更范围和目的
2. **核心评审**: 优先评审核心逻辑
3. **细节补充**: 评审边缘情况和辅助代码

### 2. 上下文感知

评审时需要了解：
- 项目的编码规范
- 相关的架构文档
- 之前的评审记录
- 业务背景和需求

### 3. 平衡建议

提出建议时需要平衡：
- 代码质量 vs 开发效率
- 理想方案 vs 现实约束
- 完美主义 vs 实用主义

## 常见问题

### Q1: 如何处理评审意见分歧?

当用户与评审意见不一致时：
1. 解释评审意见的技术依据
2. 听取用户的实际考虑
3. 寻求共识，必要时提供折中方案

### Q2: 如何处理大型 PR?

对于超过 10 个文件或 500 行变更的 PR：
1. 建议拆分为多个小 PR
2. 优先评审关键文件
3. 使用分层评审策略

### Q3: 如何处理紧急 PR?

对于需要紧急合并的 PR：
1. 明确标注紧急原因
2. 聚焦关键问题（CRITICAL/HIGH）
3. 允许标记为 "已知问题，后续优化"

## 参考资源

### 评审标准

- [Google Engineering Practices](https://google.github.io/eng-practices/review/)
- [Code Review Best Practices](https://github.com/marc作业/awesome-code-review)

### 安全评审

- [OWASP Code Review Guide](https://owasp.org/www-project-code-review-guide/)
- [CWE Top 25](https://cwe.mitre.org/top25/)

### 性能评审

- [Web Performance Rules](https://developers.google.com/web/fundamentals/performance)
- [Node.js Performance](https://nodejs.org/en/docs/guides/simple-profiling)
