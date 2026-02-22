# Sean10 Marketplace

Claude Code plugin marketplace for e-commerce development tools.

## Structure

```
sean10_marketplace/
├── .claude-plugin/
│   ├── marketplace.json     # Marketplace 清单
│   └── plugin.json          # 自身 plugin 配置
├── plugins/
│   └── marketplace-assistant/
│       ├── .claude-plugin/
│       │   └── plugin.json  # Plugin 清单
│       ├── skills/
│       │   └── marketplace-assistant/
│       │       └── SKILL.md
│       ├── agents/
│       │   └── marketplace-developer.md
│       ├── commands/
│       │   └── generate-product.md
│       ├── hooks/
│       │   └── hooks.json
│       ├── mcp-servers/
│       │   └── config.json
│       └── scripts/
│           └── format-check.sh
└── .claude.json            # 项目本地 MCP 配置
```

## 安装使用

### 1. 本地测试

```bash
# 添加 marketplace（本地路径）
/plugin marketplace add ./sean10-marketplace

# 安装 plugin
/plugin install marketplace-assistant@sean10_marketplace
```

### 2. 推送到 GitHub 后使用

```bash
# 添加 marketplace
/plugin marketplace add sean10/sean10-marketplace

# 安装 plugin
/plugin install marketplace-assistant@sean10-marketplace
```

## 迭代开发

在 `plugins/marketplace-assistant/` 目录下修改：

- **Skills**: `skills/marketplace-assistant/SKILL.md`
- **Agents**: `agents/marketplace-developer.md`
- **Commands**: `commands/generate-product.md`
- **Hooks**: `hooks/hooks.json`
- **MCP**: `mcp-servers/config.json`

修改后重新安装：

```bash
/plugin uninstall marketplace-assistant@sean10-marketplace
/plugin install marketplace-assistant@sean10-marketplace
```

## 验证

```bash
# 验证 marketplace
/plugin validate ./sean10-marketplace

# 列出已安装插件
/plugin list
```

## 更新发布

1. 修改 `plugins/marketplace-assistant/.claude-plugin/plugin.json` 中的版本号
2. 提交并推送到 GitHub
3. 用户运行 `/plugin update marketplace-assistant@sean10_marketplace`

## 可用组件

| 类型 | 路径 | 说明 |
|------|------|------|
| Skill | `skills/marketplace-assistant/` | 代码生成技能 |
| Agent | `agents/marketplace-developer.md` | 市场开发专家代理 |
| Command | `commands/generate-product.md` | `/generate-product` 命令 |
| Hook | `hooks/hooks.json` | 写入后自动格式化检查 |
| MCP | `mcp-servers/config.json` | 内存和文件系统服务器 |
