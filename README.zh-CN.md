# mihomo-skill

mihomo (Clash.Meta) 配置文件生成技能 -- Claude Code 插件市场

中文 | [English](README.md)

## 项目简介

这是一个 Claude Code 插件市场（Plugin Marketplace），提供名为 `mihomo-config` 的 AI 技能（Skill）。安装后，AI 助手能够理解并生成 mihomo (Clash.Meta) 的 YAML 代理配置文件。

覆盖内容包括：

- 20+ 种代理协议（Shadowsocks、VMess、VLESS、Trojan、Hysteria2、TUIC、WireGuard 等）
- DNS 配置（fake-ip / redir-host、fallback-filter、nameserver-policy）
- TUN 模式透明代理
- 策略组与自动选择
- 30+ 种路由规则类型
- 代理集（proxy-providers）与规则集（rule-providers）

## Skill 内容

```
plugins/mihomo-config-plugin/skills/mihomo-config/
├── SKILL.md                          # 主入口：配置结构、全局设置、TUN、嗅探器、常见错误
├── reference/
│   ├── proxies.md                    # 20+ 种代理协议详细配置
│   ├── dns.md                        # DNS 配置（fake-ip、redir-host、fallback-filter）
│   ├── proxy-groups.md               # 策略组类型与地区过滤模式
│   └── rules-and-providers.md        # 30+ 种规则类型，规则集与代理集
└── examples/
    └── complete-config.yaml          # 生产级完整示例配置
```

## 安装方法

### Claude Code（CLI / 桌面版 / Web）

```bash
# 添加 marketplace
/plugin marketplace add sd0ric4/mihomo-skill
# 安装插件
/plugin install mihomo-config-plugin@mihomo-skills
```

也可以在终端直接执行：

```bash
claude plugin marketplace add sd0ric4/mihomo-skill
claude plugin install mihomo-config-plugin@mihomo-skills
```

### Cursor

```bash
git clone https://github.com/sd0ric4/mihomo-skill.git
cp -r mihomo-skill/plugins/mihomo-config-plugin/skills/mihomo-config/ .cursor/skills/mihomo-config/
```

也可以在 `.cursorrules` 中引用 `SKILL.md` 的内容。

### Windsurf

```bash
git clone https://github.com/sd0ric4/mihomo-skill.git
cp -r mihomo-skill/plugins/mihomo-config-plugin/skills/mihomo-config/ .windsurf/skills/mihomo-config/
```

### Trae

```bash
git clone https://github.com/sd0ric4/mihomo-skill.git
cp -r mihomo-skill/plugins/mihomo-config-plugin/skills/mihomo-config/ .trae/skills/mihomo-config/
```

### Cline / Roo Code

克隆仓库后，在自定义指令或 `.clinerules` 中引用 `SKILL.md` 的内容。

### 通用方式（aider、opencode、Continue 等）

```bash
git clone https://github.com/sd0ric4/mihomo-skill.git
# 将 skill 目录复制到你所用工具的规则/指令目录
# 或在系统提示词中引用 SKILL.md
```

### 手动安装（任意 AI 工具）

1. 克隆或下载本仓库
2. 将 `SKILL.md` 和 `reference/*.md` 的内容添加到你的 AI 工具的上下文或知识库中

## 使用示例

安装完成后，可以试试以下提示词：

- "生成一个带 TUN 模式、fake-ip DNS 和港日美新地区代理订阅的 mihomo 配置"
- "帮我调试 mihomo 配置，DNS 一直泄漏"
- "给我的配置添加一个带端口跳跃的 Hysteria2 节点"
- "把这个 Clash 配置转换为 mihomo 格式"

## 贡献

欢迎提交 Issue 和 Pull Request。如果你发现配置参考有误或缺少某个协议的支持，请直接提 PR。

## 许可证

[MIT](LICENSE)

## 致谢

使用 [Claude Code](https://claude.ai/code) 构建。
