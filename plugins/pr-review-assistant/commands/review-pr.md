# Review Pull Request

当用户要求 review GitHub PR 时使用此命令。

## 使用方式

```
/review-pr <pr-url> [--experts=<experts>] [--focus=<areas>]
```

### 参数说明

- **pr-url**: GitHub PR 链接，格式为 `https://github.com/owner/repo/pull/123`
- **experts**: (可选) 指定评审专家，逗号分隔
  - 可选值: `code`, `security`, `performance`, `architecture-reviewer`, `business`, `qa`
  - 默认: `code,security,performance,architecture-reviewer`
- **focus**: (可选) 重点关注领域，逗号分隔
  - 可选值: `logic`, `style`, `security`, `performance`, `testability`, `documentation`

### 示例

```
/review-pr https://github.com/owner/repo/pull/123
/review-pr https://github.com/owner/repo/pull/456 --experts=code,security
/review-pr https://github.com/owner/repo/pull/789 --focus=logic,security
```

## 工作流程

### 阶段 0: 上下文收集 (pr-context-gatherer)

**⭐ 首先执行**，使用 `pr-context-gatherer` agent 收集PR背景上下文：

1. 获取PR基本信息（标题、作者、标签、关联Issue等）
2. 从PR描述、评论、Issue中提取URL链接
3. 抓取并总结URL内容（设计文档、API规范、讨论等）
4. 提炼业务背景、用户场景和待解答问题

**输出**: 精简的上下文报告，包含：
- PR基本信息和变更统计（仅数字，不做分析）
- 关键链接的内容摘要（**核心价值**）
- 业务背景和用户场景
- 下游专家agent的评审建议

**注意**: pr-context-gatherer **不做代码分析**。代码分析由 code-reviewer 等专业 agent 负责，避免内容重复。

### 阶段 1: PR 信息获取

1. 解析 PR URL 提取 `owner/repo/pr-number`
2. 使用 `gh pr view <pr-number> --repo <owner>/<repo> --json <fields>` 获取 PR 基本信息
   - 需要字段: `title`, `author`, `state`, `body`, `additions`, `deletions`, `changedFiles`
3. 使用 `gh pr view <pr-number> --repo <owner>/<repo> --json files` 获取变更文件列表
   - **注意**: `gh pr files` 命令不存在，必须使用 `--json files`
4. 使用 `gh pr diff <pr-number> --repo <owner>/<repo>` 获取代码变更
   - 大 PR（>500KB）使用 `gh pr diff | head -1000` 预览前 1000 行
   - 使用 `grep -E "^diff --git" | wc -l` 统计文件数
   - 使用 `grep -E "^diff --git"` 列出所有变更文件

### 注意事项

1. **gh 命令限制**:
   - `gh pr files` 不存在，使用 `gh pr view --json files`
   - 评论获取使用 `gh pr view --json comments --jq '.comments |.[0:5]'`

2. **大文件处理**:
   - diff > 500KB 时，只预览前 1000 行
   - 关键文件使用 `grep -A <lines>` 分批读取

3. **网络问题处理**:
   - GraphQL API 失败时，尝试 REST API:
     ```bash
     gh api repos/ceph/ceph/pulls/56650
     gh api repos/ceph/ceph/pulls/56650/files
     ```
   - 添加重试机制（最多 3 次）

### 阶段 2: 并行专家评审

根据指定的专家列表，并行启动子代理进行专业评审：

| 专家 | 评审维度 |
|------|----------|
| pr-context-gatherer | 上下文收集（⭐ 第一个执行） |
| code-reviewer | 代码质量、逻辑正确性、可读性 |
| security-expert | 安全漏洞、敏感信息、认证授权 |
| performance-expert | 性能问题、资源泄漏、复杂度 |
| architecture-reviewer | 设计合理性、扩展性、模式使用 |
| business-analyst | 业务逻辑、需求覆盖、用户体验 |
| qa-expert | 测试覆盖、边界条件、错误处理 |

### 阶段 3: 结果汇总

1. 收集各专家的评审意见
2. 按严重程度分类（CRITICAL/HIGH/MEDIUM/LOW）
3. 生成综合评审报告

### 阶段 4: 持久化存储

**重要**: 评审完成后，必须将结果持久化保存到本地目录。

#### 目录结构

```
{project}/
└── {pr-number}_{pr-title-slug}/
    └── REVIEW.md
```

#### 目录命名规则

- `{project}`: 项目/仓库名（如 `ceph`, `nginx`）
- `{pr-number}`: PR 编号（如 `56650`）
- `{pr-title-slug}`: PR 标题的 slug 格式
  - 转小写
  - 空格替换为 `-`
  - 移除非字母数字字符（保留 `-`）
  - 截断到合理长度（最多 100 字符）

#### 示例

- PR #56650 "crimson/os/seastore: introduce logical bucket cache" 属于 ceph 项目
- 保存路径: `ceph/56650_crimson-os-seastore-introduce-logical-bucket-cache/REVIEW.md`

#### 持久化步骤

1. **提取 PR 信息和仓库名**:
   ```bash
   gh pr view <pr-url> --json number,title,repository --jq '.number, .title, .repository.name'
   ```

2. **生成目录名**:
   ```bash
   # 转小写，空格替换为 -，移除非字母数字
   echo "56650 crimson/os/seastore: introduce logical bucket cache" | \
     tr '[:upper:]' '[:lower:]' | tr ' ' '-' | tr -cd 'a-z0-9-' | head -c 100
   # 结果: 56650-crimson-os-seastore-introduce-logical-bucket-cache
   ```

3. **创建目录并保存**:
   ```bash
   mkdir -p "${project}/${pr_number}_${pr_title_slug}"
   ```

4. **写入评审报告**:
   使用 `Write` 工具将完整的评审报告写入 `{project}/{pr_number}_{pr_title_slug}/REVIEW.md`

#### 权限检查

确保 `.claude/settings.json` 中有以下权限:
- `Bash`: 执行 `mkdir`、`echo`、`tr` 等命令处理目录和文件名
- `Write`: 创建和写入评审报告文件
- `Read`: 读取已保存的评审记录（支持交互式问答）

## 输出格式

### 评审报告结构

```markdown
# PR 评审报告: {PR标题}

## 阶段 0: 上下文收集 (pr-context-gatherer)

{pr-context-gatherer 的输出内容}
- PR基本信息
- 关键链接汇总
- 代码变更分析
- 关键背景信息

## 基本信息
- **PR**: #{pr-number}
- **作者**: {author}
- **状态**: {status}
- **变更文件数**: {files-changed}

## 总体评分

| 维度 | 评分 | 说明 |
|------|------|------|
| 代码质量 | {score}/5 | {reason} |
| 安全性 | {score}/5 | {reason} |
| 性能 | {score}/5 | {reason} |
| 架构 | {score}/5 | {reason} |
| **综合** | **{overall}/5** | |

## 关键问题

### CRITICAL (必须修复)

1. [问题标题]
   - **位置**: {file}:{line}
   - **描述**: {description}
   - **建议**: {suggestion}

### HIGH (强烈建议修复)

...

### MEDIUM (建议考虑)

...

### LOW (可选改进)

...

## 各专家评审详情

### 代码质量专家

{详细评审意见}

### 安全专家

{详细评审意见}

### 性能专家

{详细评审意见}

...

## 改进建议

### 必须修复

1. ...

### 强烈建议

1. ...

### 可选优化

1. ...

## 相关文件

{files-list}
```

## 交互式问答

评审完成后，用户可以继续提问：

```
用户: "为什么这个函数没有被测试覆盖?"
用户: "这个安全问题的具体风险是什么?"
用户: "能否给出重构的建议?"
```

## 错误处理

### 常见错误

1. **无效 PR URL**
   - 提示: "无法解析 PR URL，请检查格式是否正确"
   - 示例: "正确格式: https://github.com/owner/repo/pull/123"

2. **PR 不存在**
   - 提示: "PR 不存在或您没有访问权限"

3. **认证失败**
   - 提示: "GitHub 认证失败，请运行 `gh auth login`"

4. **专家不存在**
   - 提示: "未知的专家类型: {expert}"
   - 可选专家: code, security, performance, architecture-reviewer, business, qa

## 工具权限

- `Bash`: 执行 `gh pr`/`gh api` 命令获取 PR 信息
- `Read`: 读取 PR 相关文件内容
- `Glob/Grep`: 搜索代码库相关上下文
- `TodoWrite`: 跟踪评审进度

### 常用命令速查

```bash
# PR 基本信息
gh pr view 56650 --repo ceph/ceph --json title,author,state,body,additions,deletions,changedFiles

# 变更文件列表
gh pr view 56650 --repo ceph/ceph --json files --jq '.files[:20]'

# 代码变更（预览前1000行）
gh pr diff 56650 --repo ceph/ceph | head -1000

# 统计变更文件数
gh pr diff 56650 --repo ceph/ceph | grep -E "^diff --git" | wc -l

# 列出所有变更文件
gh pr diff 56650 --repo ceph/ceph | grep -E "^diff --git"

# REST API 备选方案
gh api repos/ceph/ceph/pulls/56650
gh api repos/ceph/ceph/pulls/56650/files
```
