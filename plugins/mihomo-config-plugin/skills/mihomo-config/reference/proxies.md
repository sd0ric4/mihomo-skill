# Proxies Reference

## Common Fields

All proxy types share these fields:

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `name` | Yes | - | Proxy name, must be unique |
| `type` | Yes | - | Protocol type: ss, ssr, vmess, vless, trojan, hysteria, hysteria2, tuic, wireguard, ssh, http, socks5, snell, direct, dns, anytls, masque, mieru, sudoku, trusttunnel |
| `server` | Yes | - | Server address (domain or IP) |
| `port` | Yes | - | Server port |
| `udp` | No | false | Allow UDP (default true for TUIC, direct, dns) |
| `ip-version` | No | dual | `dual`/`ipv4`/`ipv6`/`ipv4-prefer`/`ipv6-prefer` |
| `tfo` | No | false | TCP Fast Open |
| `mptcp` | No | false | TCP Multi Path |
| `interface-name` | No | - | Bind outgoing interface |
| `routing-mark` | No | - | Routing mark for outgoing connections |
| `dialer-proxy` | No | - | Route through another proxy/group (see [dialer-proxy](#dialer-proxy-pattern)) |

## TLS Fields

Applicable to protocols using TLS (vmess, vless, trojan, hysteria2, tuic, http, socks5, anytls):

| Field | Default | Description |
|-------|---------|-------------|
| `tls` | false | Enable TLS (trojan forces true) |
| `sni` / `servername` | server | Server Name Indication. VMess/VLESS use `servername`, others use `sni` |
| `skip-cert-verify` | false | Skip certificate verification |
| `fingerprint` | - | Certificate SHA-256 fingerprint for pinning |
| `alpn` | - | ALPN list, e.g. `[h2, http/1.1]` |
| `client-fingerprint` | - | uTLS fingerprint: `chrome`/`firefox`/`safari`/`ios`/`android`/`edge`/`360`/`qq`/`random` |
| `reality-opts` | - | REALITY settings (see below) |
| `ech-opts` | - | ECH settings (see below) |

### reality-opts

```yaml
reality-opts:
  public-key: xxxx        # Server public key
  short-id: xxxx          # Server short ID
```

### ech-opts

```yaml
ech-opts:
  enable: true
  config: base64_encoded  # ECH config, empty = resolve via DNS
  # query-server-name: xxx.com  # DNS query domain when config is empty
```

## Transport Layers

Set via `network` field. Supported: `ws`, `http`, `h2`, `grpc`, `xhttp` (VLESS only).

### WebSocket (ws)

```yaml
network: ws
ws-opts:
  path: /path
  headers:
    Host: example.com
  max-early-data: 2048
  early-data-header-name: Sec-WebSocket-Protocol
  v2ray-http-upgrade: false
  v2ray-http-upgrade-fast-open: false
```

### HTTP/2 (h2)

```yaml
network: h2
h2-opts:
  host:
    - example.com
  path: /
```

### gRPC

```yaml
network: grpc
grpc-opts:
  grpc-service-name: example
```

### HTTP

```yaml
network: http
http-opts:
  method: "GET"
  path:
    - '/'
  headers:
    Connection:
      - keep-alive
```

---

## Protocol Configs

### Shadowsocks (ss)

```yaml
proxies:
- name: "ss"
  type: ss
  server: 1.2.3.4
  port: 443
  cipher: aes-128-gcm
  password: "password"
  udp: true
```

**Cipher table:**

| Category | Ciphers |
|----------|---------|
| 2022 Blake3 | `2022-blake3-aes-128-gcm`, `2022-blake3-aes-256-gcm`, `2022-blake3-chacha20-poly1305` |
| AES-GCM | `aes-128-gcm`, `aes-192-gcm`, `aes-256-gcm` |
| CHACHA | `chacha20-ietf-poly1305`, `xchacha20-ietf-poly1305` |
| AES-CTR | `aes-128-ctr`, `aes-192-ctr`, `aes-256-ctr` |
| AES-CFB | `aes-128-cfb`, `aes-192-cfb`, `aes-256-cfb` |
| Other | `rc4-md5`, `none` |

Optional fields:
- `udp-over-tcp`: false - Enable UDP over TCP
- `plugin`: `obfs`/`v2ray-plugin`/`shadow-tls`/`restls`/`kcptun`

### VMess

```yaml
proxies:
- name: "vmess"
  type: vmess
  server: 1.2.3.4
  port: 443
  uuid: a1b2c3d4-e5f6-7890-abcd-ef1234567890
  alterId: 0
  cipher: auto
  udp: true
  tls: true
  servername: example.com
  client-fingerprint: chrome
  network: ws
  ws-opts:
    path: /ws
    headers:
      Host: example.com
```

- `uuid`: required, VMess user ID
- `alterId`: required, set 0 for AEAD
- `cipher`: required, `auto`/`none`/`zero`/`aes-128-gcm`/`chacha20-poly1305`
- `packet-encoding`: `packetaddr` (v2ray 5+) / `xudp` (xray)

### VLESS

```yaml
proxies:
- name: "vless-reality"
  type: vless
  server: 1.2.3.4
  port: 443
  uuid: a1b2c3d4-e5f6-7890-abcd-ef1234567890
  flow: xtls-rprx-vision
  udp: true
  tls: true
  servername: www.microsoft.com
  client-fingerprint: chrome
  reality-opts:
    public-key: xxxx
    short-id: xxxx
```

- `uuid`: required, VLESS user ID
- `flow`: `xtls-rprx-vision` (optional, for XTLS)
- `packet-encoding`: `xudp` / `packetaddr`
- Supports `xhttp` transport (VLESS only)

### Trojan

```yaml
proxies:
- name: "trojan"
  type: trojan
  server: 1.2.3.4
  port: 443
  password: "yourpassword"
  udp: true
  sni: example.com
  client-fingerprint: chrome
  skip-cert-verify: false
```

- `password`: required
- Transport: supports `ws`/`grpc` (add `network` + `ws-opts`/`grpc-opts`)

### Hysteria2

```yaml
proxies:
- name: "hysteria2"
  type: hysteria2
  server: 1.2.3.4
  port: 443
  password: "yourpassword"
  up: "30 Mbps"
  down: "200 Mbps"
  sni: example.com
  skip-cert-verify: false
```

With port hopping and obfs:

```yaml
proxies:
- name: "hysteria2-full"
  type: hysteria2
  server: 1.2.3.4
  port: 443
  ports: 2000-3000      # Port hopping range, overrides port
  hop-interval: 30       # Hop interval in seconds (default 30)
  password: "yourpassword"
  obfs: salamander       # Only supported obfs type
  obfs-password: "obfs-password"
  up: "30 Mbps"
  down: "200 Mbps"
  sni: example.com
```

- `up`/`down`: Brutal congestion control bandwidth (default unit: Mbps)
- `ports`: port hopping range, format e.g. `"2000-3000"`, ignores `port` when set
- `obfs`: only `salamander` supported

### TUIC

```yaml
proxies:
- name: "tuic"
  type: tuic
  server: 1.2.3.4
  port: 443
  uuid: a1b2c3d4-e5f6-7890-abcd-ef1234567890
  password: "yourpassword"
  congestion-controller: bbr
  udp-relay-mode: native
  reduce-rtt: true
  sni: example.com
  skip-cert-verify: false
  alpn:
    - h3
```

- `uuid` + `password`: required (TUIC v5)
- `congestion-controller`: `bbr`/`cubic`/`new_reno`
- `udp-relay-mode`: `native`/`quic`
- `reduce-rtt`: enable QUIC 0-RTT handshake
- UDP is enabled by default

### WireGuard

```yaml
proxies:
- name: "wg"
  type: wireguard
  private-key: eCtXsJZ27+4PbhDkHnB923tkUn2Gj59wZw5wFA75MnU=
  ip: 172.16.0.2
  ipv6: fd01:5ca1:ab1e:80fa:ab85:6eea:213f:f4a5
  udp: true
  mtu: 1408
  peers:
    - server: 162.159.192.1
      port: 2480
      public-key: Cr8hWlKvtDt7nrvf+f0brNQQzabAqrjfBvas9pmowjo=
      allowed-ips: ['0.0.0.0/0']
      reserved: [209, 98, 59]
```

- `private-key`: required, base64 client private key
- `peers`: list of peer configs (server, port, public-key, allowed-ips, reserved)
- Simplified single-peer form: put `server`, `port`, `public-key` at top level instead of `peers`
- `remote-dns-resolve` + `dns`: resolve DNS through WireGuard tunnel

### SSH

```yaml
proxies:
- name: "ssh"
  type: ssh
  server: 1.2.3.4
  port: 22
  username: root
  password: "password"
  # Or use key auth:
  # private-key: /path/to/key
  # private-key-passphrase: key_password
  host-key-algorithms:
    - rsa
```

- `username`: SSH user
- `password` or `private-key`: authentication method
- `host-key`: server host keys (empty = accept all)
- `host-key-algorithms`: allowed host key algorithms

### HTTP

```yaml
proxies:
- name: "http-proxy"
  type: http
  server: 1.2.3.4
  port: 8080
  username: user
  password: pass
  tls: true
  sni: example.com
  headers:
    X-Custom: value
```

### SOCKS5

```yaml
proxies:
- name: "socks5"
  type: socks5
  server: 1.2.3.4
  port: 1080
  username: user
  password: pass
  tls: true
  udp: true
  skip-cert-verify: false
```

### Snell

```yaml
proxies:
- name: "snell"
  type: snell
  server: 1.2.3.4
  port: 44046
  psk: "yourpsk"
  version: 3
  obfs-opts:
    mode: http
    host: bing.com
```

- `psk`: required, pre-shared key
- `version`: 1/2/3 (default 1, only v3 supports UDP)
- `obfs-opts.mode`: `http`/`tls`

### Direct

```yaml
proxies:
- name: "direct-out"
  type: direct
  udp: true
  ip-version: ipv4
  interface-name: eth0
  routing-mark: 1234
```

Uses common fields only. UDP is enabled by default.

### AnyTLS

```yaml
proxies:
- name: "anytls"
  type: anytls
  server: 1.2.3.4
  port: 443
  password: "yourpassword"
  client-fingerprint: chrome
  udp: true
  sni: example.com
  idle-session-check-interval: 30
  idle-session-timeout: 30
  min-idle-session: 0
```

- `password`: required
- `idle-session-check-interval`: check interval for idle sessions (default 30s)
- `idle-session-timeout`: close sessions idle longer than this (default 30s)
- `min-idle-session`: keep at least N idle sessions open (default 0)
- Note: AnyTLS does NOT support Reality. Use ECH to hide SNI instead.

### MASQUE

```yaml
proxies:
- name: "masque"
  type: masque
  server: 1.2.3.4
  port: 443
  private-key: "BASE64_ECDSA_PRIVATE_KEY"
  public-key: "BASE64_ECDSA_PUBLIC_KEY"
  ip: 172.16.0.2/32
  ipv6: fd00::2/128
  mtu: 1280
  udp: true
```

- `private-key`: required, base64 ECDSA private key
- `public-key`: required, base64 ECDSA server public key
- `ip`/`ipv6`: local addresses in CIDR format
- `remote-dns-resolve` + `dns`: resolve DNS through MASQUE tunnel

---

## SMUX (sing-mux)

Multiplexing over TCP-based protocols. Add to any TCP proxy:

```yaml
proxies:
- name: "ss-smux"
  type: ss
  server: 1.2.3.4
  port: 443
  cipher: aes-128-gcm
  password: "password"
  smux:
    enabled: true
    protocol: h2mux       # smux / yamux / h2mux (default)
    max-connections: 4     # conflicts with max-streams
    min-streams: 4         # conflicts with max-streams
    padding: true
    brutal-opts:
      enabled: true
      up: 50               # Upload Mbps
      down: 100             # Download Mbps
```

| Field | Default | Description |
|-------|---------|-------------|
| `enabled` | false | Enable multiplexing |
| `protocol` | h2mux | `smux` / `yamux` / `h2mux` |
| `max-connections` | - | Max connections (conflicts with max-streams) |
| `min-streams` | - | Min streams before opening new connection |
| `max-streams` | - | Max streams per connection (conflicts with max-connections + min-streams) |
| `statistic` | false | Show underlying connections in dashboard |
| `only-tcp` | false | Only multiplex TCP, UDP uses default transport |
| `padding` | false | Enable padding |
| `brutal-opts.enabled` | false | Enable TCP Brutal congestion control |
| `brutal-opts.up/down` | - | Bandwidth in Mbps |

---

## dialer-proxy Pattern

Chain proxies by routing one proxy's connection through another:

```yaml
proxies:
- name: "vps"
  type: ss
  server: vps.example.com
  port: 443
  cipher: aes-128-gcm
  password: "password"

- name: "final"
  type: vmess
  server: final.example.com
  port: 443
  uuid: xxxx
  alterId: 0
  cipher: auto
  dialer-proxy: "vps"   # Connect to "final" through "vps"
```

Traffic flow: `Client --> vps --> final --> Target`

Result:
- Target site sees `final`'s IP
- ISP sees connection to `vps` only
- `vps` server only knows it connects to `final`, not the target

`dialer-proxy` can also reference a proxy-group name, allowing relay through subscription nodes.

> **Tip:** For transit targets, prefer simple TCP protocols (SS AEAD, VMess). Avoid UDP-based (hy2/tuic/wg) or TLS-disguise (reality/shadow-tls) as transit -- they may not work reliably.
