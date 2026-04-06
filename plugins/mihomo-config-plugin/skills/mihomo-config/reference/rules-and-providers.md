# Rules and Providers Reference

## Rule Syntax

```
RULE-TYPE,PAYLOAD,TARGET[,no-resolve]
```

- 规则从上到下匹配，**第一个匹配生效**
- TARGET 为代理组名、`DIRECT`、`REJECT` 等
- 如请求为 UDP 但代理节点不支持 UDP，会继续向下匹配

## Rule Types

### Domain Rules

| 类型 | 说明 | 示例 |
|------|------|------|
| `DOMAIN` | 完整域名匹配 | `DOMAIN,ad.com,REJECT` |
| `DOMAIN-SUFFIX` | 域名后缀匹配 | `DOMAIN-SUFFIX,google.com,PROXY` |
| `DOMAIN-KEYWORD` | 域名关键字匹配 | `DOMAIN-KEYWORD,google,PROXY` |
| `DOMAIN-WILDCARD` | 通配符匹配 (`*`, `?`) | `DOMAIN-WILDCARD,*.google.com,PROXY` |
| `DOMAIN-REGEX` | 正则表达式匹配 | `DOMAIN-REGEX,^abc.*com,PROXY` |
| `GEOSITE` | GeoSite 域名集合 | `GEOSITE,youtube,PROXY` |

`DOMAIN-SUFFIX` 示例: `google.com` 匹配 `www.google.com`、`mail.google.com`、`google.com`，但不匹配 `content-google.com`。

`DOMAIN-WILDCARD` 中 `*` 匹配零个或多个字符，`?` 匹配一个字符。注意这里的通配符与配置文件其他地方的 Clash 格式通配符不同。

### IP Rules

| 类型 | 说明 | 示例 |
|------|------|------|
| `IP-CIDR` | 目标 IP 地址范围 | `IP-CIDR,127.0.0.0/8,DIRECT,no-resolve` |
| `IP-CIDR6` | IPv6 地址范围（IP-CIDR 别名） | `IP-CIDR6,2620:0:2d0:200::7/32,PROXY` |
| `IP-SUFFIX` | IP 后缀范围 | `IP-SUFFIX,8.8.8.8/24,PROXY` |
| `IP-ASN` | ASN 自治系统号 | `IP-ASN,13335,DIRECT` |
| `GEOIP` | 目标 IP 国家代码 | `GEOIP,CN,DIRECT` |

### Source IP Rules

| 类型 | 说明 | 示例 |
|------|------|------|
| `SRC-GEOIP` | 来源 IP 国家代码 | `SRC-GEOIP,cn,DIRECT` |
| `SRC-IP-ASN` | 来源 IP ASN | `SRC-IP-ASN,9808,DIRECT` |
| `SRC-IP-CIDR` | 来源 IP 地址范围 | `SRC-IP-CIDR,192.168.1.201/32,DIRECT` |
| `SRC-IP-SUFFIX` | 来源 IP 后缀范围 | `SRC-IP-SUFFIX,192.168.1.201/8,DIRECT` |

### Port Rules

| 类型 | 说明 | 示例 |
|------|------|------|
| `DST-PORT` | 目标端口（支持范围） | `DST-PORT,80,DIRECT` |
| `SRC-PORT` | 来源端口 | `SRC-PORT,7777,DIRECT` |
| `IN-PORT` | 入站监听端口 | `IN-PORT,7890,PROXY` |

### Inbound Rules

| 类型 | 说明 | 示例 |
|------|------|------|
| `IN-TYPE` | 入站类型 | `IN-TYPE,SOCKS/HTTP,PROXY` |
| `IN-USER` | 入站用户名（`/` 分隔多个） | `IN-USER,mihomo,PROXY` |
| `IN-NAME` | 入站名称 | `IN-NAME,ss,PROXY` |

### Process Rules

| 类型 | 说明 | 示例 |
|------|------|------|
| `PROCESS-NAME` | 进程名（Android 可匹配包名） | `PROCESS-NAME,curl,PROXY` |
| `PROCESS-NAME-WILDCARD` | 进程名通配符 | `PROCESS-NAME-WILDCARD,*telegram*,PROXY` |
| `PROCESS-NAME-REGEX` | 进程名正则 | `PROCESS-NAME-REGEX,(?i)Telegram,PROXY` |
| `PROCESS-PATH` | 完整进程路径 | `PROCESS-PATH,/usr/bin/wget,PROXY` |
| `PROCESS-PATH-WILDCARD` | 进程路径通配符 | `PROCESS-PATH-WILDCARD,/usr/*/wget,PROXY` |
| `PROCESS-PATH-REGEX` | 进程路径正则 | `PROCESS-PATH-REGEX,.*bin/wget,PROXY` |
| `UID` | Linux 用户 ID | `UID,1001,DIRECT` |

### Other Rules

| 类型 | 说明 | 示例 |
|------|------|------|
| `NETWORK` | 网络协议 tcp/udp | `NETWORK,udp,DIRECT` |
| `DSCP` | DSCP 标记（仅 tproxy UDP） | `DSCP,4,DIRECT` |
| `RULE-SET` | 引用规则集合 | `RULE-SET,providername,PROXY` |
| `AND` | 逻辑与 | 见下文 |
| `OR` | 逻辑或 | 见下文 |
| `NOT` | 逻辑非 | 见下文 |
| `SUB-RULE` | 匹配至子规则 | 见下文 |
| `MATCH` | 匹配所有（兜底规则） | `MATCH,DIRECT` |

## Logic Rules (AND / OR / NOT)

语法: `LOGIC_TYPE,((payload1),(payload2)),TARGET`

```yaml
rules:
  # AND: 所有条件都满足才匹配
  - AND,((DOMAIN,baidu.com),(NETWORK,UDP)),DIRECT

  # OR: 任一条件满足即匹配
  - OR,((NETWORK,UDP),(DOMAIN,baidu.com)),REJECT

  # NOT: 条件不满足才匹配
  - NOT,((DOMAIN,baidu.com)),PROXY
```

注意**括号**的使用：外层双括号包裹所有 payload，每个 payload 用单括号包裹。

## SUB-RULE

将匹配的流量送入子规则集进一步分流:

```yaml
rules:
  - SUB-RULE,(NETWORK,tcp),sub-rule-1
  - SUB-RULE,(NETWORK,udp),sub-rule-2
  - MATCH,DIRECT

sub-rules:
  sub-rule-1:
    - DOMAIN-SUFFIX,baidu.com,DIRECT
    - MATCH,PROXY
  sub-rule-2:
    - IP-CIDR,1.1.1.1/32,REJECT
    - IP-CIDR,8.8.8.8/32,PROXY
    - DOMAIN,dns.alidns.com,REJECT
```

## no-resolve 参数

仅支持**目标 IP** 类规则（`IP-CIDR`、`IP-CIDR6`、`IP-SUFFIX`、`IP-ASN`、`GEOIP` 等）。

- 匹配 IP 类规则时，mihomo 会触发 DNS 解析以获取目标 IP
- 添加 `no-resolve` 跳过 DNS 解析，仅对已知 IP 的连接生效
- 如果更早的规则已触发 DNS 解析，即使加了 `no-resolve` 也会用已缓存的 IP 匹配

```yaml
rules:
  - IP-CIDR,127.0.0.0/8,DIRECT,no-resolve
  - GEOIP,CN,DIRECT,no-resolve
```

### src 参数

仅支持目标 IP 类规则，将目标 IP 匹配转为来源 IP 匹配:

```yaml
rules:
  - IP-CIDR,192.168.1.0/24,DIRECT,src    # 匹配来源 IP 而非目标 IP
```

---

## Rule Providers

### 基本配置

```yaml
rule-providers:
  provider-name:
    type: http              # http | file | inline
    behavior: classical     # domain | ipcidr | classical
    format: yaml            # yaml | text | mrs
    url: "https://example.com/rules.yaml"
    path: ./rules/provider.yaml   # 本地存储路径（可选）
    interval: 86400         # 更新间隔（秒）
    proxy: DIRECT           # 下载时使用的代理
    size-limit: 0           # 文件大小限制（字节，0=不限）
```

### type

| 类型 | 说明 |
|------|------|
| `http` | 从 URL 下载（需配置 url） |
| `file` | 本地文件 |
| `inline` | 内联规则（通过 payload 字段指定） |

### behavior

| 行为 | 说明 | 文件内容格式 |
|------|------|-------------|
| `domain` | 域名规则集 | 每行一个域名 |
| `ipcidr` | IP 地址规则集 | 每行一个 CIDR |
| `classical` | 经典规则集 | 完整规则语法（如 `DOMAIN-SUFFIX,google.com`） |

### format

| 格式 | 说明 |
|------|------|
| `yaml` | YAML 格式（默认） |
| `text` | 纯文本格式 |
| `mrs` | 二进制格式（更小更快，behavior 仅支持 domain/ipcidr） |

转换命令: `mihomo convert-ruleset domain/ipcidr yaml/text input.yaml output.mrs`

### inline 类型

```yaml
rule-providers:
  my-rules:
    type: inline
    behavior: classical
    payload:
      - "DOMAIN-SUFFIX,google.com"
      - "DOMAIN-SUFFIX,youtube.com"
```

### YAML Anchor Pattern (推荐)

使用 YAML 锚点避免重复配置:

```yaml
rule-anchor:
  ip: &ip {type: http, interval: 86400, behavior: ipcidr, format: mrs}
  domain: &domain {type: http, interval: 86400, behavior: domain, format: mrs}
  classical: &classical {type: http, interval: 86400, behavior: classical, format: text}

rule-providers:
  # 域名规则集
  google_domain:
    <<: *domain
    url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/google.mrs"
  github_domain:
    <<: *domain
    url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/github.mrs"
  cn_domain:
    <<: *domain
    url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/cn.mrs"

  # IP 规则集
  google_ip:
    <<: *ip
    url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geoip/google.mrs"
  cn_ip:
    <<: *ip
    url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geoip/cn.mrs"
  private_ip:
    <<: *ip
    url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geoip/private.mrs"
```

在 rules 中引用:

```yaml
rules:
  - RULE-SET,google_domain,PROXY
  - RULE-SET,cn_domain,DIRECT
  - RULE-SET,google_ip,PROXY
  - RULE-SET,cn_ip,DIRECT
  - RULE-SET,private_ip,DIRECT
  - MATCH,PROXY
```

---

## Proxy Providers

### 基本配置

```yaml
proxy-providers:
  provider1:
    type: http                # http | file | inline
    url: "https://your-subscription-url"
    path: ./proxy_providers/provider1.yaml  # 本地缓存路径（可选）
    interval: 86400           # 更新间隔（秒）
    proxy: DIRECT             # 下载时使用的代理
    size-limit: 0             # 文件大小限制（字节）
    header:                   # 自定义请求头
      User-Agent:
        - "mihomo/1.18.3"

    health-check:             # 健康检查（延迟测试）
      enable: true
      url: "https://www.gstatic.com/generate_204"
      interval: 300           # 检查间隔（秒）
      timeout: 5000           # 超时（毫秒）
      lazy: true              # 不使用时不检测（默认 true）
      expected-status: 204    # 期望的 HTTP 状态码

    override:                 # 覆写节点属性
      additional-prefix: "[provider1] "   # 节点名前缀
      additional-suffix: " | provider1"   # 节点名后缀
      udp: true
      skip-cert-verify: true
      ip-version: ipv4-prefer
      proxy-name:             # 正则替换节点名
        - pattern: "IPLC-(.*?)x"
          target: "iplc $1"

    filter: "(?i)港|hk|hongkong|hong kong"  # 正则筛选节点
    exclude-filter: "xxx"     # 正则排除节点
    exclude-type: "ss|http"   # 按类型排除（| 分隔，非正则）
```

### type

| 类型 | 说明 |
|------|------|
| `http` | 从 URL 下载订阅（需配置 url） |
| `file` | 本地文件 |
| `inline` | 内联节点（通过 payload 字段指定） |

### health-check

| 字段 | 说明 | 默认值 |
|------|------|--------|
| `enable` | 是否启用 | - |
| `url` | 测试地址 | - |
| `interval` | 检查间隔（秒） | - |
| `timeout` | 超时（毫秒） | - |
| `lazy` | 不使用时不检测 | `true` |
| `expected-status` | 期望 HTTP 状态码 | - |

推荐测试地址:
- `https://www.gstatic.com/generate_204` (Google)
- `https://cp.cloudflare.com` (Cloudflare)

### override

覆写订阅中的节点属性:

| 字段 | 说明 |
|------|------|
| `additional-prefix` | 为节点名添加前缀 |
| `additional-suffix` | 为节点名添加后缀 |
| `proxy-name` | 正则替换节点名（pattern/target） |
| `tfo` | TCP Fast Open |
| `mptcp` | 多路径 TCP |
| `udp` | 启用 UDP |
| `udp-over-tcp` | UDP over TCP |
| `skip-cert-verify` | 跳过证书验证 |
| `dialer-proxy` | 前置代理 |
| `interface-name` | 绑定网络接口 |
| `routing-mark` | 路由标记 |
| `ip-version` | IP 版本偏好 |
| `up` / `down` | Hysteria 带宽限制 |

### filter / exclude-filter / exclude-type

```yaml
proxy-providers:
  # 只保留香港节点
  hk-nodes:
    type: http
    url: "https://your-subscription-url"
    filter: "(?i)港|hk|hongkong|hong kong"
    exclude-filter: "premium"     # 排除包含 premium 的节点
    exclude-type: "ss|http"       # 排除 ss 和 http 类型的节点

  # 只保留日本节点
  jp-nodes:
    type: http
    url: "https://your-subscription-url"
    filter: "(?i)日|jp|japan|tokyo"
```

### inline 类型

当 http/file 解析失败时，payload 也可作为备用:

```yaml
proxy-providers:
  fallback-provider:
    type: inline
    payload:
      - name: "ss1"
        type: ss
        server: server
        port: 443
        cipher: chacha20-ietf-poly1305
        password: "password"
```

### 在 proxy-groups 中引用

```yaml
proxy-groups:
  - name: "Auto"
    type: url-test
    use:                     # 引用 proxy-provider
      - provider1
      - provider2
    proxies:                 # 也可混合直接定义的代理
      - DIRECT
```
