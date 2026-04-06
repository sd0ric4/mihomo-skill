# mihomo-skill

[中文](README.zh-CN.md) | English

A Claude Code Plugin Marketplace that distributes **mihomo-config** -- a reference skill that helps AI assistants understand and generate [mihomo](https://github.com/MetaCubeX/mihomo) (Clash.Meta) YAML proxy configuration files.

## What It Does

The `mihomo-config` skill gives your AI assistant deep knowledge of mihomo configuration, enabling it to:

- **Generate** complete, production-ready mihomo YAML configs from scratch
- **Debug** configuration issues (DNS leaks, routing problems, TUN failures)
- **Explain** any section of a mihomo config in detail
- **Convert** Clash configs to mihomo format

Coverage includes 20+ proxy protocols (Shadowsocks, VMess, VLESS, Trojan, Hysteria2, TUIC, WireGuard, and more), DNS configuration (fake-ip / redir-host), TUN mode transparent proxying, proxy groups with region filtering, 30+ routing rule types, and proxy/rule providers.

## Skill Contents

| File | Description |
|------|-------------|
| `SKILL.md` | Main entry point -- config structure, global settings, TUN, sniffer, common gotchas |
| `reference/proxies.md` | 20+ proxy protocol configurations (SS, VMess, VLESS, Trojan, Hysteria2, TUIC, WireGuard, etc.) |
| `reference/dns.md` | DNS configuration (fake-ip, redir-host, fallback-filter, nameserver-policy) |
| `reference/proxy-groups.md` | Proxy group types and region filtering patterns |
| `reference/rules-and-providers.md` | 30+ rule types, rule providers, proxy providers |
| `examples/complete-config.yaml` | Production-ready example configuration |

All skill files are located under `plugins/mihomo-config-plugin/skills/mihomo-config/`.

## Installation

### Claude Code (CLI, Desktop, and Web)

From the Claude Code prompt:

```bash
# Add marketplace
/plugin marketplace add sd0ric4/mihomo-skill
# Install plugin
/plugin install mihomo-config-plugin@mihomo-skills
```

Or from your terminal:

```bash
claude plugin marketplace add sd0ric4/mihomo-skill
claude plugin install mihomo-config-plugin@mihomo-skills
```

### Cursor

Copy the skill files into your project or global Cursor rules:

```bash
# Clone the repo
git clone https://github.com/sd0ric4/mihomo-skill.git
# Copy skill to your project
cp -r mihomo-skill/plugins/mihomo-config-plugin/skills/mihomo-config/ .cursor/skills/mihomo-config/
```

Or reference in `.cursorrules`:

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

Add to your custom instructions or `.clinerules`:

```bash
git clone https://github.com/sd0ric4/mihomo-skill.git
# Reference SKILL.md in your custom instructions
```

### Generic (aider, opencode, Continue, etc.)

```bash
git clone https://github.com/sd0ric4/mihomo-skill.git
# Copy the skill directory to your tool's rules/instructions directory
# Or reference SKILL.md in your system prompt / custom instructions
```

### Manual Installation (any AI tool)

For tools that support custom system prompts or knowledge files:

1. Clone or download this repository
2. Add the contents of `SKILL.md` and relevant `reference/*.md` files to your AI tool's context or knowledge base

## Usage Examples

After installation, try these prompts with your AI assistant:

- "Generate a mihomo config with TUN mode, fake-ip DNS, and proxy subscriptions for HK/JP/US regions"
- "Debug my mihomo config -- DNS keeps leaking"
- "Add a Hysteria2 node with port hopping to my config"
- "Convert this Clash config to mihomo format"

## Contributing

Contributions are welcome. Please open an issue or submit a pull request on [GitHub](https://github.com/sd0ric4/mihomo-skill).

## License

[MIT](LICENSE)

## Credits

Built with [Claude Code](https://claude.ai/code).
