# Listeners (Inbound)

mihomo can act as a server by defining multiple inbound listeners. Listeners
coexist with port-based inbound (`mixed-port`, `socks-port`, `port`, etc.) —
they add additional endpoints, they do not replace them.

## Common Listener Fields

| Field    | Type   | Default   | Description                                       |
| -------- | ------ | --------- | ------------------------------------------------- |
| `name`   | string | required  | Unique listener name                              |
| `type`   | string | required  | Listener type (see below)                         |
| `port`   | int    | required  | Listen port                                       |
| `listen` | string | `0.0.0.0` | Bind address                                      |
| `rule`   | string | --        | Sub-rule name; falls back to `rules` if not found |
| `proxy`  | string | --        | Route all traffic to this proxy/group directly    |
| `udp`    | bool   | true      | Accept UDP traffic (where applicable)             |

## LAN Listener Types

These accept unencrypted traffic from local clients:

```yaml
listeners:
  - name: socks5-in-1
    type: socks
    port: 10808

  - name: http-in-1
    type: http
    port: 10809
    listen: 0.0.0.0

  - name: mixed-in-1
    type: mixed          # HTTP(S) + SOCKS combined
    port: 10810

  - name: redir-in-1
    type: redir          # Linux redirect transparent proxy
    port: 10811

  - name: tproxy-in-1
    type: tproxy         # Linux TProxy transparent proxy
    port: 10812
    # udp: true

  - name: tunnel-in-1
    type: tunnel          # Traffic forwarding tunnel
    port: 10816
    network: [tcp, udp]
    target: target.com    # Forward all traffic to this address

  - name: tun-in-1
    type: tun
    stack: system         # system / gvisor / mixed
    dns-hijack:
      - 0.0.0.0:53
    # auto-detect-interface: true
    # auto-route: true
    # mtu: 9000
    # strict-route: true
```

### Listener-Specific Notes

| Type     | Platform    | Notes                                          |
| -------- | ----------- | ---------------------------------------------- |
| `socks`  | All         | SOCKS5 proxy; supports UDP relay               |
| `http`   | All         | HTTP/HTTPS proxy (CONNECT method)              |
| `mixed`  | All         | Auto-detect HTTP or SOCKS5                     |
| `redir`  | Linux       | Requires iptables REDIRECT rules               |
| `tproxy` | Linux       | Requires iptables TPROXY rules + capabilities  |
| `tunnel` | All         | Fixed destination forwarding                   |
| `tun`    | All         | Virtual network device; see TUN section in main |

## Internet Listener Types (Encrypted Inbound)

These accept encrypted traffic, allowing mihomo to act as a proxy server:

```yaml
listeners:
  - name: ss-in-1
    type: shadowsocks
    port: 10813
    password: vlmpIPSyHH6f4S8WVPdRIHIlzmB+GIRfoH3aNJ/t9Gg=
    cipher: 2022-blake3-aes-256-gcm

  - name: vmess-in-1
    type: vmess
    port: 10814
    users:
      - username: 1
        uuid: 9d0cb9d0-964f-4ef6-897d-6c6b3ccf9e68
        alterId: 1

  - name: tuic-in-1
    type: tuic
    port: 10815
    # token:                         # TUIC v4 (mutually exclusive with users)
    #   - TOKEN
    # users:                         # TUIC v5 (mutually exclusive with token)
    #   00000000-0000-0000-0000-000000000000: PASSWORD-0
    # certificate: ./server.crt
    # private-key: ./server.key
    # congestion-controller: bbr
    # max-idle-time: 15000
    # authentication-timeout: 1000
    # alpn: [h3]
    # max-udp-relay-packet-size: 1500
```

## Legacy Entry Configs

These top-level shortcuts are equivalent to defining a listener but use a more
compact syntax. They coexist with the `listeners:` block:

```yaml
# Shadowsocks entry (URI format)
ss-config: ss://2022-blake3-aes-256-gcm:PASSWORD@:23456

# VMess entry (URI format)
vmess-config: vmess://1:UUID@:12345

# TUIC server entry (expanded format)
tuic-server:
  enable: true
  listen: 127.0.0.1:10443
  users:
    00000000-0000-0000-0000-000000000000: PASSWORD-0
  certificate: ./server.crt
  private-key: ./server.key
  congestion-controller: bbr
  max-idle-time: 15000
  authentication-timeout: 1000
  alpn: [h3]
  max-udp-relay-packet-size: 1500
```

Note: `proxy` on a listener routes all inbound traffic directly to the
specified proxy or proxy group, bypassing rules entirely. `rule` selects a
sub-rule chain; if the named sub-rule does not exist, the global `rules` list
is used instead.
