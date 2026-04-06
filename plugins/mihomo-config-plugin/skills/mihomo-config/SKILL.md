---
name: mihomo-config
description: "Use when generating, debugging, or explaining mihomo (Clash.Meta / Clash / MetaCubeX) YAML proxy configuration -- protocols like Shadowsocks, VMess, VLESS, Trojan, Hysteria2, TUIC, WireGuard; fake-ip DNS; TUN mode; proxy-groups; routing rules; or proxy/rule providers."
---

## Overview

mihomo (formerly Clash.Meta) is a rule-based network proxy tool that uses YAML
configuration files. It supports multiple proxy protocols (Shadowsocks, VMess,
VLESS, Trojan, Hysteria2, TUIC, WireGuard, etc.), advanced DNS with fake-ip,
TUN-mode transparent proxying, and flexible rule-based traffic routing.

## Config File Structure

A mihomo config file is a single YAML document. Top-level keys should appear in
roughly this order:

| Section              | Purpose                                        |
| -------------------- | ---------------------------------------------- |
| `mixed-port`         | HTTP+SOCKS5 combined listen port               |
| `port`               | HTTP proxy listen port                         |
| `socks-port`         | SOCKS5 proxy listen port                       |
| `redir-port`         | Redirect (Linux redir) transparent proxy port  |
| `tproxy-port`        | TProxy (Linux) transparent proxy port          |
| `allow-lan`          | Allow LAN devices to use the proxy             |
| `bind-address`       | Address to bind for LAN access                 |
| `lan-allowed-ips`    | IP whitelist for LAN access                    |
| `lan-disallowed-ips` | IP blacklist for LAN access (higher priority)  |
| `authentication`     | User:pass credentials for HTTP/SOCKS/mixed     |
| `skip-auth-prefixes` | CIDRs that bypass proxy authentication         |
| `mode`               | Running mode: rule / global / direct           |
| `log-level`          | Log verbosity: silent/error/warning/info/debug |
| `ipv6`               | Accept IPv6 traffic                            |
| `external-controller`| RESTful API listen address                     |
| `secret`             | API access secret                              |
| `external-ui`        | Path to web dashboard static files             |
| `unified-delay`      | Calculate RTT to normalize latency measurement |
| `tcp-concurrent`     | Dial all resolved IPs concurrently             |
| `find-process-mode`  | Process matching: always/strict/off            |
| `geodata-mode`       | Use .dat instead of .mmdb for GeoIP            |
| `geox-url`           | Custom download URLs for geo databases         |
| `geo-auto-update`    | Auto-update geo databases                      |
| `profile`            | Persistence: store-selected, store-fake-ip     |
| `ntp`                | NTP time synchronization                       |
| `experimental`       | Experimental feature flags                     |
| `sniffer`            | Protocol sniffing configuration                |
| `tun`                | TUN device transparent proxy                   |
| `tunnels`            | Traffic forwarding tunnels (TCP/UDP)           |
| `listeners`          | Advanced inbound listener definitions          |
| `dns`                | DNS resolver configuration                     |
| `proxies`            | Manually defined proxy nodes                   |
| `proxy-groups`       | Proxy selection strategy groups                |
| `rules`              | Traffic routing rules (ordered list)           |
| `proxy-providers`    | Remote/file-based proxy sources                |
| `rule-providers`     | Remote/file-based rule sources                 |
| `sub-rules`          | Sub-rule chains for advanced routing           |

## Global Settings Quick Reference

| Key                    | Type    | Default    | Description                                           |
| ---------------------- | ------- | ---------- | ----------------------------------------------------- |
| `mixed-port`           | int     | --         | Combined HTTP/SOCKS5 port                             |
| `port`                 | int     | --         | HTTP-only proxy port                                  |
| `socks-port`           | int     | --         | SOCKS5-only proxy port                                |
| `redir-port`           | int     | --         | Linux redirect transparent proxy port                 |
| `tproxy-port`          | int     | --         | Linux TProxy transparent proxy port                   |
| `allow-lan`            | bool    | false      | Allow other devices on the LAN to connect             |
| `bind-address`         | string  | `"*"`      | Bind address for LAN access (`"*"` = all)             |
| `lan-allowed-ips`      | list    | `0.0.0.0/0,::/0` | IP whitelist for LAN access                    |
| `lan-disallowed-ips`   | list    | --         | IP blacklist for LAN (higher priority than whitelist) |
| `authentication`       | list    | --         | User:pass pairs for HTTP/SOCKS/mixed proxy            |
| `skip-auth-prefixes`   | list    | --         | CIDRs that bypass proxy authentication                |
| `mode`                 | string  | `rule`     | `rule` / `global` / `direct`                          |
| `log-level`            | string  | `info`     | `silent`/`error`/`warning`/`info`/`debug`             |
| `ipv6`                 | bool    | true       | Accept IPv6 traffic                                   |
| `external-controller`  | string  | --         | RESTful API address, e.g. `127.0.0.1:9090`            |
| `secret`               | string  | `""`       | API access key                                        |
| `external-ui`          | string  | --         | Path to web UI folder (e.g. metacubexd)               |
| `external-ui-url`      | string  | --         | URL to auto-download web UI zip                       |
| `external-ui-name`     | string  | --         | Custom subfolder name for web UI                      |
| `external-controller-cors` | object | --     | CORS config for API (allow-origins, allow-private-network) |
| `external-controller-tls`  | string | --     | HTTPS API address (requires TLS cert config)          |
| `external-controller-unix` | string | --     | Unix socket API address (no secret verification)      |
| `external-controller-pipe` | string | --     | Windows named pipe API address                        |
| `external-doh-server`  | string  | --         | DOH endpoint on API port (no secret verification)     |
| `unified-delay`        | bool    | false      | Compute RTT to normalize latency across protocols     |
| `tcp-concurrent`       | bool    | false      | Try all DNS-resolved IPs concurrently                 |
| `find-process-mode`    | string  | `strict`   | `always`/`strict`/`off` - process name matching       |
| `geodata-mode`         | bool    | false      | `true` = use .dat files; `false` = use .mmdb          |
| `geo-auto-update`      | bool    | false      | Automatically update GEO databases                    |
| `geo-update-interval`  | int     | 24         | GEO update interval in hours                          |
| `keep-alive-interval`  | int     | 15         | TCP keep-alive interval in seconds                    |
| `keep-alive-idle`      | int     | 15         | TCP keep-alive idle timeout in seconds                |
| `disable-keep-alive`   | bool    | false      | Disable TCP keep-alive (forced true on Android)       |
| `interface-name`       | string  | --         | Global outbound network interface                     |
| `routing-mark`         | int     | --         | Linux routing mark for outbound connections           |
| `global-ua`            | string  | clash.meta | User-Agent for external resource downloads            |
| `profile.store-selected` | bool  | false      | Persist proxy group selections across restarts        |
| `profile.store-fake-ip`  | bool  | false      | Persist fake-ip mappings across restarts              |

## Proxy Authentication

HTTP/SOCKS/mixed inbound proxies can require credentials:

```yaml
authentication:
  - "user1:pass1"
  - "user2:pass2"
skip-auth-prefixes:        # CIDRs that bypass authentication
  - 127.0.0.1/8
  - ::1/128
```

## TUN Configuration

TUN mode creates a virtual network interface to capture all system traffic
transparently. Recommended stack is `mixed` (TCP via system stack, UDP via
gvisor) for the best balance of compatibility and performance.

```yaml
tun:
  enable: true
  stack: mixed          # system / gvisor / mixed
  dns-hijack:
    - "any:53"
    - "tcp://any:53"
  auto-route: true      # auto-configure system routes
  auto-redirect: true   # auto iptables redirect (Linux only)
  auto-detect-interface: true
```

Key fields:

| Field                  | Type   | Default  | Description                                     |
| ---------------------- | ------ | -------- | ----------------------------------------------- |
| `enable`               | bool   | false    | Enable TUN device                               |
| `stack`                | string | gvisor   | Protocol stack: `system`/`gvisor`/`mixed`       |
| `dns-hijack`           | list   | --       | Hijack DNS queries to internal resolver          |
| `auto-route`           | bool   | false    | Automatically set up system routes               |
| `auto-redirect`        | bool   | false    | Auto iptables/nftables redirect (Linux only)     |
| `auto-detect-interface`| bool   | false    | Auto-select outbound network interface           |
| `device`               | string | --       | TUN device name (macOS: must start with `utun`)  |
| `mtu`                  | int    | 9000     | Maximum transmission unit                        |
| `strict-route`         | bool   | false    | Strict routing to prevent leaks                  |
| `udp-timeout`          | int    | 300      | UDP NAT expiry in seconds                        |
| `endpoint-independent-nat` | bool | false  | Enable endpoint-independent NAT                  |
| `gso`                  | bool   | false    | Generic Segmentation Offload (Linux only)        |
| `gso-max-size`         | int    | 65536    | Max data block length for GSO (Linux only)       |

Routing control (require `auto-route`):

| Field                       | Type | Default | Description                                   |
| --------------------------- | ---- | ------- | --------------------------------------------- |
| `route-address`             | list | --      | Route only these network segments              |
| `route-exclude-address`     | list | --      | Exclude these network segments from routing    |
| `route-address-set`         | list | --      | Add ruleset IP CIDRs to firewall (Linux+nftables+auto-redirect) |
| `route-exclude-address-set` | list | --      | Exclude ruleset IP CIDRs (Linux+nftables+auto-redirect) |
| `iproute2-table-index`      | int  | 2022    | iproute2 routing table index (Linux only)      |
| `iproute2-rule-index`       | int  | 9000    | iproute2 rule starting index (Linux only)      |

Note: `route-address-set` / `route-exclude-address-set` conflict with `routing-mark`.

Interface and user filtering (require `auto-route`):

| Field                  | Type | Platform | Description                                    |
| ---------------------- | ---- | -------- | ---------------------------------------------- |
| `include-interface`    | list | All      | Limit routed interfaces (conflicts with exclude)|
| `exclude-interface`    | list | All      | Exclude interfaces from routing                 |
| `include-uid`          | list | Linux    | Include users for TUN routing                   |
| `include-uid-range`    | list | Linux    | Include user ranges (e.g. `1000:9999`)          |
| `exclude-uid`          | list | Linux    | Exclude users from TUN routing                  |
| `exclude-uid-range`    | list | Linux    | Exclude user ranges                             |
| `include-android-user` | list | Android  | Include Android users (0=owner, 10=clone, 999=multi) |
| `include-package`      | list | Android  | Include Android app packages for TUN routing    |
| `exclude-package`      | list | Android  | Exclude Android app packages from TUN routing   |

Deprecated fields (use `route-address` / `route-exclude-address` instead):
`inet4-route-address`, `inet6-route-address`, `inet4-route-exclude-address`,
`inet6-route-exclude-address`.

Notes:
- On macOS/Windows, `dns-hijack` cannot capture LAN-destined DNS.
- On Android with Private DNS enabled, DNS hijacking will not work.
- If a firewall is active, `system` and `mixed` stacks may need the mihomo
  binary to be explicitly allowed through the firewall.
- `include-interface` and `exclude-interface` are mutually exclusive.

## NTP Configuration

Synchronize system time via NTP. `write-to-system` requires root/admin.

```yaml
ntp:
  enable: true
  write-to-system: true
  server: time.apple.com
  port: 123
  interval: 30             # sync interval in minutes
```

| Field             | Type   | Default        | Description                        |
| ----------------- | ------ | -------------- | ---------------------------------- |
| `enable`          | bool   | false          | Enable NTP service                 |
| `write-to-system` | bool   | false          | Sync to system time (requires root)|
| `server`          | string | time.apple.com | NTP server address                 |
| `port`            | int    | 123            | NTP server port                    |
| `interval`        | int    | 30             | Sync interval in minutes           |

## Sniffer Configuration

The sniffer inspects traffic to determine the real destination domain, which is
essential when using fake-ip mode or when clients do not send SNI.

```yaml
sniffer:
  enable: true
  sniff:
    HTTP:
      ports: [80, 8080-8880]
      override-destination: true
    TLS:
      ports: [443, 8443]
    QUIC:
      ports: [443, 8443]
  skip-domain:
    - "Mijia Cloud"
    - "+.push.apple.com"
```

| Field                  | Type   | Description                                      |
| ---------------------- | ------ | ------------------------------------------------ |
| `force-dns-mapping`    | bool   | Force sniff on redir-host resolved traffic        |
| `parse-pure-ip`        | bool   | Force sniff on traffic without a domain           |
| `override-destination` | bool   | Use sniffed domain as actual destination (global) |
| `skip-domain`          | list   | Domains to skip sniffing (supports wildcards)     |

## Tunnels

Traffic forwarding tunnels for TCP/UDP. Optionally routed through a proxy.

```yaml
tunnels:
  # Single-line format: network,address,target,proxy
  - tcp/udp,127.0.0.1:6553,8.8.8.8:53,proxy

  # Multi-line format (equivalent)
  - network: [tcp, udp]
    address: 127.0.0.1:6553
    target: 8.8.8.8:53
    proxy: proxy             # optional: route through this proxy/group
```

| Field     | Type   | Description                                    |
| --------- | ------ | ---------------------------------------------- |
| `network` | list   | Protocols to listen on: `tcp`, `udp`, or both  |
| `address` | string | Local listen address (host:port)               |
| `target`  | string | Forward destination (host:port)                |
| `proxy`   | string | Optional proxy/group name to route traffic through |

## Listeners (Inbound)

mihomo can define multiple inbound listeners that coexist with port-based
inbound (`mixed-port`, `socks-port`, etc.). Listener types include `socks`,
`http`, `mixed`, `redir`, `tproxy`, `tunnel`, `tun` for LAN traffic, and
`shadowsocks`, `vmess`, `tuic` for encrypted server-side inbound.

See `reference/listeners.md` for the full listener configuration reference.

## Experimental

```yaml
experimental:
  quic-go-disable-gso: false     # disable QUIC GSO
  quic-go-disable-ecn: false     # disable QUIC ECN
  dialer-ip4p-convert: false     # enable IP4P address conversion
```

## DNS

mihomo has a built-in DNS resolver with two enhanced modes:

- **`fake-ip`**: Returns fake IPs from a private range, resolves the real IP
  when the connection is made. Faster and recommended for most setups.
- **`redir-host`**: Resolves the real IP immediately and maps it back to the
  domain for rule matching.

See `reference/dns.md` for the complete DNS configuration guide including
nameserver setup, fallback filters, fake-ip-filter, and nameserver-policy.

## Proxies

Each proxy node is defined with at minimum: `name`, `type`, `server`, `port`.
Protocol-specific fields vary by type (ss, vmess, vless, trojan, hysteria2,
tuic, wireguard, etc.). Set `udp: true` explicitly on nodes that should carry
UDP traffic.

See `reference/proxies.md` for the full proxy protocol reference including all
supported types and their fields.

## Proxy Groups

Proxy groups define selection/fallback strategies for routing decisions:

- **`select`**: Manual selection from the UI
- **`url-test`**: Auto-select the lowest-latency node
- **`fallback`**: Use the first available node in order
- **`load-balance`**: Distribute traffic across nodes (consistent-hashing or round-robin)

Use `include-all: true` to pull in all nodes from proxies and proxy-providers.
Use `filter` with regex to narrow down by node name.

See `reference/proxy-groups.md` for the full proxy group reference.

## Rules & Providers

Rules are evaluated top-to-bottom. Format:

```
RULE-TYPE,PAYLOAD,TARGET[,no-resolve]
```

Common rule types: `DOMAIN`, `DOMAIN-SUFFIX`, `DOMAIN-KEYWORD`, `IP-CIDR`,
`IP-CIDR6`, `GEOIP`, `GEOSITE`, `RULE-SET`, `PROCESS-NAME`, `MATCH`.

`rule-providers` fetch rule lists from remote URLs or local files with behaviors
`domain`, `ipcidr`, or `classical`, in formats `yaml`, `text`, or `mrs`.

See `reference/rules-and-providers.md` for the full rules and providers reference.

## YAML Anchor Patterns

YAML anchors (`&name` to define, `*name` to reference, `<<: *name` to merge)
are heavily used in mihomo configs to reduce repetition. Three common patterns:

- **Proxy-provider anchor**: `p: &p {type: http, interval: 3600, health-check: {...}}` then `<<: *p` in each provider
- **Rule-provider anchor**: `rule-anchor:` block with `&ip` / `&domain` anchors, then `<<: *domain` per provider
- **Proxy-group anchor**: `pr: &pr {type: select, proxies: [...]}` then `{name: Google, <<: *pr}`

See `examples/complete-config.yaml` for all three patterns in a working config.

## Complete Example

See `examples/complete-config.yaml` for a full, annotated, production-ready
configuration using RULE-SET mode with MetaCubeX MRS rule-providers, TUN,
fake-ip DNS, YAML anchors, regional proxy groups, and service-based routing.

## Common Mistakes / Gotchas

1. **UDP not proxied**: `udp: true` must be explicitly set on Shadowsocks,
   VMess, and Trojan nodes. Without it, UDP traffic (DNS, QUIC, VoIP) bypasses
   the proxy entirely.

2. **`relay` is deprecated**: Do not use `relay` proxy groups for chaining.
   Use `dialer-proxy` on individual proxy nodes instead:
   ```yaml
   proxies:
     - name: "exit-node"
       type: ss
       server: exit.example.com
       port: 443
       dialer-proxy: "entry-node"
   ```

3. **fake-ip-filter must include wildcards**: Always include at least `"*"`,
   `"+.lan"`, and `"+.local"` in `fake-ip-filter`. The `"*"` entry matches
   names without a dot (e.g. `localhost`). Without these, local network
   discovery and mDNS will break.

4. **DNS circular dependency**: `proxy-server-nameserver` must resolve to a
   direct IP address or use a DNS server that does not go through the proxy.
   If the DNS server used to resolve proxy server addresses itself needs the
   proxy, you get an infinite loop and the connection hangs.

5. **`include-all: true` scope**: This only pulls in nodes from `proxies` and
   `proxy-providers`. It does NOT include other proxy groups. If you want to
   nest groups, list them explicitly in `proxies:` within the group definition.

6. **Case-insensitive filter regex**: Use `(?i)` prefix for case-insensitive
   matching in proxy group `filter` fields:
   ```yaml
   filter: "(?i)港|hk|hongkong|hong kong"
   ```

7. **`behavior` must match data format**: In `rule-providers`, the `behavior`
   field (`domain`, `ipcidr`, `classical`) must match the actual content of
   the data file. A mismatch causes silent rule failures. MRS files from
   MetaCubeX use `domain` for geosite and `ipcidr` for geoip.

8. **`MATCH` must be last**: The `MATCH` (catch-all) rule must be the final
   entry in the `rules` list. Any rules after `MATCH` are never evaluated.

9. **Quote special characters in proxy names**: Proxy names containing `:`,
   `#`, `[`, `]`, `{`, `}`, or commas must be wrapped in quotes:
   ```yaml
   - name: "US - Node #1 [Fast]"
   ```

10. **`port` must be a number**: The `port` field on proxy nodes must be an
    unquoted integer. Writing `port: "443"` (string) can cause parse errors
    or unexpected behavior in some versions.

11. **`exclude-type: direct`**: When using `include-all: true` in proxy groups,
    add `exclude-type: direct` to prevent the built-in direct proxy from
    appearing in the node list.

12. **Rule-provider `format` field**: When using MetaCubeX meta-rules-dat,
    use `format: mrs` for `.mrs` files, `format: text` for `.list` files,
    and `format: yaml` for `.yaml` files. The format must match the file
    extension / actual content.

13. **`listeners` vs port-based inbound**: When using `listeners:` alongside
    `mixed-port`/`socks-port`/`port`, both are active simultaneously.
    `listeners:` adds additional inbound endpoints — it does not replace
    port-based inbound.
