# Claude Code插件示例

**官方插件市场文档**：
* [创建和分发 plugin marketplace](https://code.claude.com/docs/zh-CN/plugin-marketplaces)
* [Claude Code创建插件](https://code.claude.com/docs/zh-CN/plugins)
* [Plugin 清单架构](https://code.claude.com/docs/zh-CN/plugins-reference#plugin-%E6%B8%85%E5%8D%95%E6%9E%B6%E6%9E%84)

## 插件的清单文件定义结构

```sh
enterprise-plugin/
├── .claude-plugin/           # 元数据目录（可选）
│   └── plugin.json             # plugin 清单
├── skills/                   # Skills
│   ├── code-reviewer/
│   │   └── SKILL.md
│   └── pdf-processor/
│       ├── SKILL.md
│       └── scripts/
├── commands/                 # Skills 作为平面 .md 文件
│   ├── status.md
│   └── logs.md
├── agents/                   # Subagent 定义
│   ├── security-reviewer.md
│   ├── performance-tester.md
│   └── compliance-checker.md
├── output-styles/            # 输出样式定义
│   └── terse.md
├── themes/                   # 颜色主题定义
│   └── dracula.json
├── monitors/                 # 后台 monitor 配置
│   └── monitors.json
├── hooks/                    # Hook 配置
│   ├── hooks.json           # 主 hook 配置
│   └── security-hooks.json  # 其他 hooks
├── bin/                      # 添加到 PATH 的 plugin 可执行文件
│   └── my-tool               # 在 Bash tool 中可作为裸命令调用
├── settings.json            # plugin 的默认设置
├── .mcp.json                # MCP server 定义
├── .lsp.json                # LSP server 配置
├── scripts/                 # Hook 和实用脚本
│   ├── security-scan.sh
│   ├── format-code.py
│   └── deploy.js
├── LICENSE                  # 许可证文件
└── CHANGELOG.md             # 版本历史
```

`xxx插件/.claude-plugin/plugin.json`：

```sh
{
  "name": "plugin-name",
  "displayName": "Plugin Name",
  "version": "1.2.0",
  "description": "Brief plugin description",
  "author": {
    "name": "Author Name",
    "email": "author@example.com",
    "url": "https://github.com/author"
  },
  "homepage": "https://docs.example.com/plugin",
  "repository": "https://github.com/author/plugin",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "skills": "./custom/skills/",
  "commands": ["./custom/commands/special.md"],
  "agents": ["./custom/agents/reviewer.md"],
  "hooks": "./config/hooks.json",
  "mcpServers": "./mcp-config.json",
  "outputStyles": "./styles/",
  "lspServers": "./.lsp.json",
  "experimental": {
    "themes": "./themes/",
    "monitors": "./monitors.json"
  },
  "dependencies": [
    "helper-lib",
    { "name": "secrets-vault", "version": "~2.1.0" }
  ]
}
```