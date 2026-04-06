# mihomo-skill

中文 | [English](README.md)

一个 Claude Code 插件市场（Plugin Marketplace），分发 **mihomo-config** 技能 -- 帮助 AI 助手理解并生成 [mihomo](https://github.com/MetaCubeX/mihomo) (Clash.Meta) YAML 代理配置文件的参考技能。

## 功能介绍

`mihomo-config` 技能为你的 AI 助手注入 mihomo 配置的专业知识，使其能够：

- **生成**完整的、可直接用于生产环境的 mihomo YAML 配置
- **调试**配置问题（DNS 泄漏、路由异常、TUN 故障）
- **解释** mihomo 配置中任意章节的细节
- **转换** Clash 配置为 mihomo 格式

覆盖内容包括 20+ 种代理协议（Shadowsocks、VMess、VLESS、Trojan、Hysteria2、TUIC、WireGuard 等）、DNS 配置（fake-ip / redir-host）、TUN 模式透明代理、策略组与地区过滤、30+ 种路由规则类型，以及代理集/规则集。

## Skill 内容

| 文件 | 说明 |
|------|------|
| `SKILL.md` | 主入口 -- 配置结构、全局设置、TUN、嗅探器、常见错误 |
| `reference/proxies.md` | 20+ 种代理协议配置（SS、VMess、VLESS、Trojan、Hysteria2、TUIC、WireGuard 等） |
| `reference/dns.md` | DNS 配置（fake-ip、redir-host、fallback-filter、nameserver-policy） |
| `reference/proxy-groups.md` | 策略组类型与地区过滤模式 |
| `reference/rules-and-providers.md` | 30+ 种规则类型、规则集、代理集 |
| `examples/complete-config.yaml` | 生产级示例配置 |

所有技能文件位于 `plugins/mihomo-config-plugin/skills/mihomo-config/` 目录下。

## 安装

### Claude Code（CLI、桌面版、Web）

在 Claude Code 提示符中：

```bash
# 添加 marketplace
/plugin marketplace add sd0ric4/mihomo-skill
# 安装插件
/plugin install mihomo-config-plugin@mihomo-skills
```

或从终端执行：

```bash
claude plugin marketplace add sd0ric4/mihomo-skill
claude plugin install mihomo-config-plugin@mihomo-skills
```

### Cursor

将技能文件复制到项目或全局 Cursor 规则目录：

```bash
# 克隆仓库
git clone https://github.com/sd0ric4/mihomo-skill.git
# 复制技能到项目目录
cp -r mihomo-skill/plugins/mihomo-config-plugin/skills/mihomo-config/ .cursor/skills/mihomo-config/
```

或在 `.cursorrules` 中引用：

```
@mihomo-skill/plugins/mihomo-config-plugin/skills/mihomo-config/SKILL.md
```

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

在自定义指令或 `.clinerules` 中引用：

```bash
git clone https://github.com/sd0ric4/mihomo-skill.git
# 在自定义指令中引用 SKILL.md
```

### 通用方式（aider、opencode、Continue 等）

```bash
git clone https://github.com/sd0ric4/mihomo-skill.git
# 将 skill 目录复制到你所用工具的规则/指令目录
# 或在系统提示词/自定义指令中引用 SKILL.md
```

### 手动安装（任意 AI 工具）

适用于支持自定义系统提示词或知识库的工具：

1. 克隆或下载本仓库
2. 将 `SKILL.md` 和相关 `reference/*.md` 文件的内容添加到你的 AI 工具的上下文或知识库中

## 使用示例

安装完成后，可以试试以下提示词：

- "生成一个带 TUN 模式、fake-ip DNS 和港日美地区代理订阅的 mihomo 配置"
- "帮我调试 mihomo 配置，DNS 一直泄漏"
- "给我的配置添加一个带端口跳跃的 Hysteria2 节点"
- "把这个 Clash 配置转换为 mihomo 格式"

## 贡献

欢迎贡献。请在 [GitHub](https://github.com/sd0ric4/mihomo-skill) 上提交 Issue 或 Pull Request。

## 许可证

[MIT](LICENSE)

## 致谢

使用 [Claude Code](https://claude.ai/code) 构建。
