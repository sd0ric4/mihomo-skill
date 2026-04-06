# Proxy Groups Reference

## Group Types

| Type | Description |
|------|-------------|
| `select` | Manual proxy selection |
| `url-test` | Auto-select lowest latency proxy |
| `fallback` | Use first available proxy in order |
| `load-balance` | Distribute traffic across proxies |
| `relay` | **Deprecated** -- use `dialer-proxy` instead |

## Common Fields

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `name` | Yes | - | Group name (quote if contains special chars) |
| `type` | Yes | - | Group type (see table above) |
| `proxies` | No | - | List of proxy names or other group names |
| `use` | No | - | List of proxy-provider names |
| `url` | No | - | Health check URL (only checks `proxies`, not `use`) |
| `interval` | No | 0 | Health check interval in seconds (0 = disabled) |
| `lazy` | No | true | Skip health check when group is not active |
| `timeout` | No | 5000 | Health check timeout in ms |
| `max-failed-times` | No | 5 | Force health check after N consecutive failures |
| `include-all` | No | false | Include all proxies + all providers (sorted by name) |
| `include-all-proxies` | No | false | Include all proxies (sorted by name) |
| `include-all-providers` | No | false | Include all providers (overrides `use`) |
| `filter` | No | - | Regex to include matching nodes (applies to providers and include-all) |
| `exclude-filter` | No | - | Regex to exclude matching nodes |
| `exclude-type` | No | - | Exclude by type, `\|` separated (e.g. `"Shadowsocks\|Http"`) |
| `expected-status` | No | * | Expected HTTP status: `200`, `200/302`, `400-503` |
| `hidden` | No | false | Hide group from API/dashboard |
| `icon` | No | - | Icon string for API/dashboard |
| `disable-udp` | No | false | Disable UDP for this group |

## Type-Specific Fields

### url-test: tolerance

```yaml
proxy-groups:
- name: "Auto"
  type: url-test
  proxies:
    - node-1
    - node-2
  url: 'https://www.gstatic.com/generate_204'
  interval: 300
  tolerance: 50
```

`tolerance` (ms): Only switch to a new node when it is faster than the current node by at least this amount. Prevents constant switching between nodes with similar latency.

### load-balance: strategy

```yaml
proxy-groups:
- name: "LB"
  type: load-balance
  proxies:
    - node-1
    - node-2
  url: 'https://www.gstatic.com/generate_204'
  interval: 300
  strategy: consistent-hashing
```

| Strategy | Description |
|----------|-------------|
| `consistent-hashing` | Same destination domain routes to same proxy |
| `round-robin` | Distribute all requests across proxies evenly |
| `sticky-sessions` | Same source+destination uses same proxy (10min cache) |

> `consistent-hashing` matches on top-level domain when destination is a domain name.

## Region Filtering Patterns

Standard regex patterns for filtering proxy nodes by region:

```yaml
proxy-groups:
# Hong Kong
- name: "HK"
  type: url-test
  include-all-providers: true
  filter: "(?i)港|hk|hongkong|hong kong"

# Taiwan
- name: "TW"
  type: url-test
  include-all-providers: true
  filter: "(?i)台|tw|taiwan"

# Japan
- name: "JP"
  type: url-test
  include-all-providers: true
  filter: "(?i)日|jp|japan"

# USA
- name: "US"
  type: url-test
  include-all-providers: true
  filter: "(?i)美|us|unitedstates|united states"

# Singapore
- name: "SG"
  type: url-test
  include-all-providers: true
  filter: "(?i)(新|sg|singapore)"

# Other regions (exclude all above)
- name: "Other"
  type: url-test
  include-all-providers: true
  filter: "(?i)^(?!.*(?:港|hk|hongkong|台|tw|taiwan|日|jp|japan|新|sg|singapore|美|us|unitedstates)).*"
```

## Complete Example

```yaml
proxy-groups:
- name: "Proxy"
  type: select
  proxies:
    - Auto
    - HK
    - JP
    - US
    - SG
    - DIRECT

- name: "Auto"
  type: url-test
  include-all-providers: true
  url: 'https://www.gstatic.com/generate_204'
  interval: 300
  tolerance: 50
  lazy: true

- name: "HK"
  type: url-test
  include-all-providers: true
  filter: "(?i)港|hk|hongkong|hong kong"
  url: 'https://www.gstatic.com/generate_204'
  interval: 300

- name: "Fallback"
  type: fallback
  proxies:
    - HK
    - JP
    - US
  url: 'https://www.gstatic.com/generate_204'
  interval: 300

- name: "LB"
  type: load-balance
  include-all-providers: true
  strategy: consistent-hashing
  url: 'https://www.gstatic.com/generate_204'
  interval: 300
```
