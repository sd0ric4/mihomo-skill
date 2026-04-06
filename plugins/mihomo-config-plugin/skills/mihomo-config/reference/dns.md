# DNS Configuration Reference

## Complete Annotated DNS Config

```yaml
dns:
  enable: true                    # 启用 DNS 模块（false 则使用系统 DNS）
  cache-algorithm: arc            # 缓存算法: lru (默认) | arc (自适应替换缓存)
  prefer-h3: false                # DoH 优先使用 HTTP/3
  use-hosts: true                 # 使用配置文件中的 hosts 映射
  use-system-hosts: true          # 查询系统 /etc/hosts
  respect-rules: false            # DNS 连接遵守路由规则（需配合 proxy-server-nameserver）
  listen: 0.0.0.0:1053            # DNS 监听地址，支持 UDP/TCP
  ipv6: false                     # 是否解析 AAAA 记录（false 则返回空）

  # --- 默认 DNS ---
  default-nameserver:             # 用于解析 DNS 服务器本身的域名，必须为 IP
    - 223.5.5.5
    - 119.29.29.29

  # --- enhanced-mode ---
  enhanced-mode: fake-ip          # fake-ip | redir-host
  fake-ip-range: 198.18.0.1/16   # fake-ip 地址池（也作为 TUN 默认网段）
  # fake-ip-range6: fdfe:dcba:9876::1/64  # fake-ip IPv6 地址池
  fake-ip-filter-mode: blacklist  # blacklist | whitelist | rule
  fake-ip-filter:                 # blacklist 模式下匹配的域名返回真实 IP
    - "*.lan"
    - "*.local"
    - "*.direct"
    - "time.*.com"
    - "ntp.*.com"
    - "+.msftconnecttest.com"
    - "+.msftncsi.com"
    - "localhost.ptlogin2.qq.com"
  # fake-ip-ttl: 1                # fake-ip TTL，非必要勿改

  # --- 域名解析服务器 ---
  nameserver:                     # 主 DNS 服务器
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query

  fallback:                       # 后备 DNS（一般用境外 DNS）
    - tls://8.8.4.4
    - tls://1.1.1.1

  # --- 域名策略 ---
  nameserver-policy:              # 指定域名使用特定 DNS，优先于 nameserver/fallback
    "+.arpa": "10.0.0.1"
    "rule-set:cn":
      - https://doh.pub/dns-query
      - https://dns.alidns.com/dns-query
    "geosite:cn":
      - https://doh.pub/dns-query

  # --- 代理节点 DNS ---
  proxy-server-nameserver:        # 解析代理节点域名的 DNS（避免循环依赖）
    - https://doh.pub/dns-query
    - tls://223.5.5.5
  proxy-server-nameserver-policy: # 特定代理节点的 DNS 策略
    "www.yournode.com": "114.114.114.114"

  # --- 直连 DNS ---
  direct-nameserver:              # direct 出口的域名解析服务器
    - system
  direct-nameserver-follow-policy: false  # 是否遵循 nameserver-policy

  # --- fallback 过滤 ---
  fallback-filter:
    geoip: true                   # 启用 GeoIP 判断
    geoip-code: CN                # 非 CN IP 视为污染，使用 fallback
    geosite:
      - gfw                       # 匹配的域名只用 fallback
    ipcidr:
      - 240.0.0.0/4               # 这些 IP 段视为污染
    domain:
      - "+.google.com"            # 这些域名直接用 fallback
      - "+.facebook.com"
      - "+.youtube.com"
```

## DNS Server Types

| 协议 | 格式 | 示例 |
|------|------|------|
| UDP | `ip:port` | `223.5.5.5` |
| TCP | `tcp://ip:port` | `tcp://223.5.5.5` |
| DoT (DNS over TLS) | `tls://host:port` | `tls://dns.alidns.com` |
| DoH (DNS over HTTPS) | `https://host/path` | `https://doh.pub/dns-query` |
| DoQ (DNS over QUIC) | `quic://host:port` | `quic://dns.alidns.com` |
| System DNS | `system` | `system` |
| DHCP | `dhcp://interface` | `dhcp://en0` |
| RCode | `rcode://code` | `rcode://success`, `rcode://refused` |

## enhanced-mode Decision Guide

| | fake-ip | redir-host |
|---|---------|------------|
| 工作方式 | 返回假 IP，通过映射表关联真实域名 | 返回真实解析 IP |
| DNS 泄漏 | 低 - 不向上游暴露真实查询 | 较高 - 需额外防护 |
| 速度 | 快 - 无需等待真实 DNS 解析 | 较慢 - 需等待解析完成 |
| 兼容性 | 部分应用需加入 fake-ip-filter | 兼容性好 |
| 推荐场景 | TUN/透明代理模式（推荐） | 需要真实 IP 的特殊场景 |
| 默认值 | 否 | 是 (redir-host 为默认) |

**建议**: 大多数场景使用 `fake-ip`，配合 `fake-ip-filter` 排除不兼容的域名。

## fake-ip-filter 与 fake-ip-filter-mode

### blacklist (默认)

匹配的域名**不返回** fake-ip，而是返回真实 IP。适合大部分域名走 fake-ip、少数例外的场景。

### whitelist

只有匹配的域名**才返回** fake-ip。适合只需要少量域名走 fake-ip 的场景。

### rule (路由规则模式)

使用与路由 rules 一致的匹配语法，自上而下匹配，最终值为 `fake-ip` 或 `real-ip`:

```yaml
dns:
  fake-ip-filter-mode: rule
  fake-ip-filter:
    - RULE-SET,reject-domain,fake-ip    # behavior 须为 domain/classical
    - RULE-SET,proxy-domain,fake-ip
    - GEOSITE,gfw,fake-ip
    - DOMAIN,www.baidu.com,real-ip
    - DOMAIN-SUFFIX,qq.com,real-ip
    - DOMAIN-SUFFIX,jd.com,fake-ip
    - MATCH,fake-ip                     # 兜底规则
```

### 常用 fake-ip-filter 列表 (blacklist 模式)

```yaml
fake-ip-filter:
  # 裸名（无点号的名称如 localhost）
  - "*"
  # 本地域名
  - "*.lan"
  - "*.local"
  - "*.direct"
  # NTP 时间同步
  - "time.*.com"
  - "ntp.*.com"
  - "time.*.gov"
  # Windows 网络检测
  - "+.msftconnecttest.com"
  - "+.msftncsi.com"
  # QQ 登录
  - "localhost.ptlogin2.qq.com"
  # 游戏平台
  - "+.stun.*.*"
  - "+.stun.*.*.*"
  # 音乐/流媒体（需要真实 IP 判断地区）
  - "+.music.163.com"
  - "+.126.net"
```

## nameserver-policy

指定域名使用特定 DNS 服务器，优先于 nameserver/fallback。

**键格式** - 支持域名通配:
- `+.example.com` - 匹配 example.com 及其所有子域名
- `geosite:cn` - 匹配 GeoSite 中国域名集合
- `rule-set:cn` - 匹配 rule-set 中的域名

**值格式** - 支持字符串或数组:

```yaml
nameserver-policy:
  # 单个 DNS
  "+.internal.company.com": "10.0.0.1"
  # 多个 DNS
  "geosite:cn":
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
  # 带附加参数
  "+.google.com": "https://dns.google/dns-query#proxy"
```

## proxy-server-nameserver -- 解决循环依赖

当代理节点地址为域名时，需要 DNS 解析该域名。但如果 DNS 查询本身也需要经过代理，就会形成**循环依赖（鸡蛋问题）**。

`proxy-server-nameserver` 专门用于解析代理节点域名，**必须**满足:
- 使用 IP 地址（不依赖域名解析），或
- 使用可直连的加密 DNS

```yaml
proxy-server-nameserver:
  - tls://223.5.5.5            # 阿里 DNS（IP 直连）
  - https://doh.pub/dns-query  # DNSPod DoH（可直连）
```

`proxy-server-nameserver-policy` 格式同 nameserver-policy，仅在 proxy-server-nameserver 不为空时生效:

```yaml
proxy-server-nameserver-policy:
  "www.yournode.com": "114.114.114.114"
```

## fallback-filter 逻辑

配置 `fallback` 后自动启用 fallback-filter。filter 决定何时使用 fallback 结果:

```
nameserver 和 fallback 并发查询
          |
    fallback-filter 检查 nameserver 结果
          |
    +----- geosite 匹配? -----> 域名在列表中 --> 只用 fallback
    |
    +----- domain 匹配? ------> 域名在列表中 --> 只用 fallback
    |
    +----- geoip 检查? -------> IP 非 geoip-code 国家 --> 用 fallback
    |
    +----- ipcidr 检查? ------> IP 在污染段内 --> 用 fallback
    |
    +--- 通过所有检查 ---------> 使用 nameserver 结果
```

各字段说明:

| 字段 | 作用 | 示例 |
|------|------|------|
| `geoip: true` | 启用 GeoIP 判断 | - |
| `geoip-code: CN` | 该国家的 IP 直接采用 nameserver 结果，其余用 fallback | `CN` |
| `geosite` | 列表内域名只用 fallback，不查询 nameserver | `[gfw]` |
| `ipcidr` | 这些网段视为 DNS 污染，nameserver 解析出则用 fallback | `[240.0.0.0/4]` |
| `domain` | 这些域名视为已污染，直接用 fallback | `["+.google.com"]` |

## DNS 附加参数

通过 `#` 附加到 DNS 地址后，多个参数用 `&` 连接:

```
https://8.8.8.8/dns-query#proxy&ecs=1.1.1.1/24&ecs-override=true
```

| 参数 | 值类型 | 说明 |
|------|--------|------|
| `#proxy` 或 `#代理名` | 代理名/接口名 | 通过指定代理或网络接口查询 DNS |
| `#RULES` | - | DNS 连接遵守路由规则（等同 respect-rules） |
| `&h3=true` | bool | 强制使用 HTTP/3（需 DoH 服务器支持） |
| `&skip-cert-verify=true` | bool | 跳过 TLS 证书验证 |
| `&ecs=1.1.1.1/24` | IP/掩码 | 指定 EDNS Client Subnet |
| `&ecs-override=true` | bool | 强制覆盖 ECS subnet |
| `&disable-ipv4=true` | bool | 丢弃 A 记录回应 |
| `&disable-ipv6=true` | bool | 丢弃 AAAA 记录回应 |
| `&disable-qtype-65=true` | bool | 丢弃 HTTPS (TYPE65) 记录 |

**注意**: 使用 `#proxy` 通过代理查询时，必须配置 `proxy-server-nameserver` 以避免循环依赖。

## hosts 配置

```yaml
hosts:
  "*.clash.dev": 127.0.0.1        # 通配符域名
  "alpha.clash.dev": "::1"        # 完整域名（优先级高于通配符）
  test.com: [1.1.1.1, 2.2.2.2]   # 多 IP（数组）
  baidu.com: google.com           # 域名重定向（不支持数组）
```

- 通过 `use-hosts: true` 启用（默认开启）
- 完整域名优先级 > 通配符域名: `foo.example.com` > `*.example.com` > `.example.com`
- `use-system-hosts: true` 同时查询系统 hosts 文件
