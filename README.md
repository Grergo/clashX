<h1 align="center">
  <img src="https://github.com/Dreamacro/clash/raw/master/docs/logo.png" alt="Clash" width="200">
  <br>
  ClashX
  <br>
</h1>



A rule based proxy For Mac base on [Clash](https://github.com/Dreamacro/clash).

ClashX 旨在提供一个简单轻量化的代理客户端，如果需要更多的定制化，可以考虑使用 [CFW Mac 版](https://github.com/Fndroid/clash_for_windows_pkg/releases) 

**此版本支持Mitm、Vless**

## Features

- Local HTTP/HTTPS/SOCKS server with authentication support
- Built-in [fake-ip](https://www.rfc-editor.org/rfc/rfc3089) DNS server that aims to minimize DNS pollution attack impact. DoH/DoT upstream supported.
- Rules based off domains, GEOIP, GEOSITE, IP-CIDR or process names to route packets to different destinations
- Support Vmess/Shadowsocks/Socks5/Trojan/Vless/Snell/WireGuard/HTTPS
- Support for Netfilter TCP redirect
- Support Mitm
- Policy routing with Scripts

## Install

You can download from [Release](https://github.com/Grergo/clashX/releases) page


## Build

- Make sure have python3 and golang installed in your computer.

- Install Golang

  ```
  brew install golang
  
  or download from https://golang.org
  ```

- Download deps

  ```
  bash install_dependency.sh
  ```

- Build and run.

## Config



The default configuration directory is `$HOME/.config/clash`

The default name of the configuration file is `config.yaml`. You can use your custom config name and switch config in menu `Config` section.


Checkout [Clash-pro](https://github.com/yaling888/clash) or [SS-Rule-Snippet for Clash](https://github.com/Hackl0us/SS-Rule-Snippet/blob/master/LAZY_RULES/clash.yaml) or [lancellc's gitbook](https://lancellc.gitbook.io/clash/) for more detail.

## Advance Config

### IMPORTANT

Before use, you need to download [geosite.dat](https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat) and place it in the `~/.config/clash/` directory.

### Change your status menu icon

  Place your icon file in the `~/.config/clash/menuImage.png`  then restart ClashX

### Change default system ignore list.

- Change by menu -> Config -> Setting -> Bypass proxy settings for these Hosts & Domains

### URL Schemes.

- Using url scheme to import remote config.

  ```
  clash://install-config?url=http%3A%2F%2Fexample.com&name=example
  ```

- Using url scheme to reload current config.

  ```
  clash://update-config
  ```

### Get process name

You can add the follow config in your config file, and set your proxy mode to rule. Then open the log via help menu in ClashX.

```
script:
  code: |
    def main(ctx, metadata):
      # Log ProcessName
      ctx.log('Process Name: ' + ctx.resolve_process_name(metadata))
      return 'DIRECT'
```

### MITM configuration

A root CA certificate is required, the 
MITM proxy server will generate a CA certificate file and a CA private key file in your Clash home directory, you can use your own certificate replace it. 

Need to install and trust the CA certificate on the client device, open this URL [http://mitm.clash/cert.crt](http://mitm.clash/cert.crt) by the web browser to install the CA certificate, the host name 'mitm.clash' was always been hijacked.

NOTE: this feature cannot work on tls pinning

WARNING: DO NOT USE THIS FEATURE TO BREAK LOCAL LAWS

```yaml
# Port of MITM proxy server on the local end
mitm-port: 7894

# Man-In-The-Middle attack
mitm:
  hosts: # use for others proxy type. E.g: TUN, socks
    - +.example.com
  rules: # rewrite rules
    - '^https?://www\.example\.com/1 url reject' # The "reject" returns HTTP status code 404 with no content.
    - '^https?://www\.example\.com/2 url reject-200' # The "reject-200" returns HTTP status code 200 with no content.
    - '^https?://www\.example\.com/3 url reject-img' # The "reject-img" returns HTTP status code 200 with content of 1px png.
    - '^https?://www\.example\.com/4 url reject-dict' # The "reject-dict" returns HTTP status code 200 with content of empty json object.
    - '^https?://www\.example\.com/5 url reject-array' # The "reject-array" returns HTTP status code 200 with content of empty json array.
    - '^https?://www\.example\.com/(6) url 302 https://www.example.com/new-$1'
    - '^https?://www\.(example)\.com/7 url 307 https://www.$1.com/new-7'
    - '^https?://www\.example\.com/8 url request-header (\r\n)User-Agent:.+(\r\n) request-header $1User-Agent: haha-wriohoh$2' # The "request-header" works for all the http headers not just one single header, so you can match two or more headers including CRLF in one regular expression.
    - '^https?://www\.example\.com/9 url request-body "pos_2":\[.*\],"pos_3" request-body "pos_2":[{"xx": "xx"}],"pos_3"'
    - '^https?://www\.example\.com/10 url response-header (\r\n)Tracecode:.+(\r\n) response-header $1Tracecode: 88888888888$2'
    - '^https?://www\.example\.com/11 url response-body "errmsg":"ok" response-body "errmsg":"not-ok"'
```

### DNS configuration

Support resolve ip with a proxy tunnel or interface.

Support `geosite` with `fallback-filter`.

Use `curl -X POST controllerip:port/cache/fakeip/flush` to flush persistence fakeip

 ```yaml
dns:
  enable: true
  use-hosts: true
  ipv6: false
  # remote-dns-resolve: true # remote resolve DNS on handle UDP session, default value is true
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  listen: 127.0.0.1:6868
  default-nameserver:
    - 119.29.29.29
    - 114.114.114.114
  nameserver:
    - https://doh.pub/dns-query
    - tls://223.5.5.5:853
  fallback:
    - 'tls://8.8.4.4:853#proxy or interface'
    - 'https://1.0.0.1/dns-query#Proxy'  # append the proxy adapter name to the end of DNS URL with '#' prefix.
  fallback-filter:
    geoip: false
    geosite:
      - gfw  # `geosite` filter only use fallback server to resolve ip, prevent DNS leaks to untrusted DNS providers.
    domain:
      - +.example.com
    ipcidr:
      - 0.0.0.0/32
 ```

### TUN configuration

Simply add the following to the main configuration:

#### NOTE:

> auto-route and auto-detect-interface only available on macOS, Windows and Linux, receive IPv4 traffic

```yaml
tun:
  enable: true
  stack: system # or gvisor
  # device: tun://utun8 # or fd://xxx, it's optional
  # dns-hijack:
  #   - 8.8.8.8:53
  #   - tcp://8.8.8.8:53
  #   - any:53
  #   - tcp://any:53
  auto-route: true # auto set global route
  auto-detect-interface: true # conflict with interface-name
```

or

```yaml
interface-name: en0

tun:
  enable: true
  stack: system # or gvisor
  # dns-hijack:
  #   - 8.8.8.8:53
  #   - tcp://8.8.8.8:53
  auto-route: true # auto set global route
```

It's recommended to use fake-ip mode for the DNS server.

Clash needs elevated permission to create TUN device:

```sh
$ sudo ./clash
```

Then manually create the default route and DNS server. If your device already has some TUN device, Clash TUN might not work. In this case, fake-ip-filter may helpful.

Enjoy! :)

### Rules configuration

- Support rule `SCRIPT` shortcuts.
- Support rule `GEOSITE`.
- Support rule `USER-AGENT`.
- Support `multiport` condition for rule `SRC-PORT` and `DST-PORT`.
- Support `network` condition for all rules.
- Support `process` condition for all rules.
- Support source IPCIDR condition for all rules, just append to the end.

The `GEOIP` databases via [https://github.com/Loyalsoldier/geoip](https://raw.githubusercontent.com/Loyalsoldier/geoip/release/Country.mmdb).

The `GEOSITE` databases via [https://github.com/Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat).

```yaml
mode: rule

script:
  shortcuts: # `src_port` and `dst_port` are number
    quic: 'network == "udp" and dst_port == 443'
    privacy: '"analytics" in host or "adservice" in host or "firebase" in host or "safebrowsing" in host or "doubleclick" in host'
    BilibiliUdp: |
      network == "udp" and match_provider("bilibili")
    ParentalControls: |
      src_ip == "192.168.1.123" and now.hour < 8 and now.hour > 22
rules:
  # rule SCRIPT shortcuts
  - SCRIPT,quic,REJECT # Disable QUIC
  - SCRIPT,privacy,REJECT
  - SCRIPT,BilibiliUdp,REJECT
  - SCRIPT,ParentalControls,REJECT

  # network condition for all rules
  - DOMAIN-SUFFIX,example.com,DIRECT,tcp
  - DOMAIN-SUFFIX,example.com,REJECT,udp

  # process condition for all rules (add 'P:' prefix)
  - DOMAIN-SUFFIX,example.com,REJECT,P:Google Chrome Helper

  # multiport condition for rules SRC-PORT and DST-PORT
  - DST-PORT,123/136/137-139,DIRECT,udp

  # USER-AGENT payload cannot include the comma character, '*' meaning any character.
  - USER-AGENT,*example*,PROXY

  # rule GEOSITE
  - GEOSITE,category-ads-all,REJECT
  - GEOSITE,icloud@cn,DIRECT
  - GEOSITE,apple@cn,DIRECT
  - GEOSITE,apple-cn,DIRECT
  - GEOSITE,microsoft@cn,DIRECT
  - GEOSITE,facebook,PROXY
  - GEOSITE,youtube,PROXY
  - GEOSITE,geolocation-cn,DIRECT
  - GEOSITE,geolocation-!cn,PROXY

  # source IPCIDR condition for all rules in gateway proxy
  #- GEOSITE,geolocation-!cn,REJECT,192.168.1.88/32,192.168.1.99/32
  
  - GEOIP,telegram,PROXY,no-resolve
  - GEOIP,lan,DIRECT,no-resolve
  - GEOIP,cn,DIRECT

  - MATCH,PROXY
```

Script shortcut parameters

```ts
now: {
  year:       int
  month:      int
  day:        int
  hour:       int
  minute:     int
  second:     int
}
type:            string
network:         string
host:            string
process_name:    string
process_path:    string
user_agent:      string
special_proxy:   string
src_ip:          string
src_port:        int
dst_ip:          string // call resolve_ip(host) if empty
dst_port:        int
```

Script shortcut functions

```ts
type resolve_ip = (host: string) => string // ip string
type in_cidr = (ip: string, cidr: string) => boolean // ip in cidr
type geoip = (ip: string) => string // country code
type match_provider = (name: string) => boolean // in rule provider
type resolve_process_name = () => string // process name
type resolve_process_path = () => string // process path
```

### Script configuration

Script enables users to programmatically select a policy for the packets with more flexibility.

NOTE: If you want to use `ctx.geoip(ip)` you need to manually resolve ip first.

```yaml
mode: script

script:
  # path: ./script.star
  code: |
    def main(ctx, metadata):
      processName = ctx.resolve_process_name(metadata)
      if processName == 'apsd':
        return "DIRECT"

      if metadata["network"] == 'udp' and metadata["dst_port"] == 443:
        return "REJECT"

      host = metadata["host"]
      for kw in ['analytics', 'adservice', 'firebase', 'bugly', 'safebrowsing', 'doubleclick']:
        if kw in host:
          return "REJECT"

      # now = time.now()
      # if (now.hour < 8 or now.hour > 18) and metadata["src_ip"] == '192.168.1.99':
      #   return "REJECT"

      if ctx.rule_providers["category-ads-all"].match(metadata):
        return "REJECT"

      if ctx.rule_providers["youtube"].match(metadata):
        ctx.log('[Script] domain %s matched youtube' % host)
        return "Proxy"

      if ctx.rule_providers["geolocation-cn"].match(metadata):
        ctx.log('[Script] domain %s matched geolocation-cn' % host)
        return "DIRECT"

      ip = metadata["dst_ip"]
      if ip == "":
        ip = ctx.resolve_ip(host)
        if ip == "":
          return "Proxy"

      code = ctx.geoip(ip)
      if code == "TELEGRAM":
        ctx.log('[Script] matched telegram')
        return "Proxy"

      if code == "CN" or code == "LAN" or code == "PRIVATE":
        return "DIRECT"

      return "Proxy" # default policy for requests which are not matched by any other script
```

the context and metadata

```ts
interface Metadata {
  type: string // socks5、http
  network: string // tcp、udp
  host: string
  user_agent: string
  special_proxy: string
  src_ip: string
  src_port: string
  dst_ip: string
  dst_port: string
}

interface Context {
  resolve_ip: (host: string) => string // ip string
  resolve_process_name: (metadata: Metadata) => string
  resolve_process_path: (metadata: Metadata) => string
  geoip: (ip: string) => string // country code
  log: (log: string) => void
  proxy_providers: Record<string, Array<{ name: string, alive: boolean, delay: number }>>
  rule_providers: Record<string, { match: (metadata: Metadata) => boolean }>
}
```

### Proxies configuration

Support outbound protocol `VLESS`.

Support `Trojan` with XTLS.

Support userspace `WireGuard` outbound.

Support relay `UDP` traffic.

Support filtering proxy providers in proxy groups.

Support custom http request header, prefix name and V2Ray subscription URL in proxy providers.

```yaml
proxies:
  # VLESS
  - name: "vless-tls"
    type: vless
    server: server
    port: 443
    uuid: uuid
    network: tcp
    servername: example.com
    udp: true
    # skip-cert-verify: true
  - name: "vless-xtls"
    type: vless
    server: server
    port: 443
    uuid: uuid
    network: tcp
    servername: example.com
    flow: xtls-rprx-direct # or xtls-rprx-origin
    # flow-show: true # print the XTLS direction log
    # udp: true
    # skip-cert-verify: true

  # Trojan
  - name: "trojan-xtls"
    type: trojan
    server: server
    port: 443
    password: yourpsk
    network: tcp
    flow: xtls-rprx-direct # or xtls-rprx-origin
    # flow-show: true # print the XTLS direction log
    # udp: true
    # sni: example.com # aka server name
    # skip-cert-verify: true
  
  # WireGuard
  - name: "wg"
    type: wireguard
    server: 127.0.0.1
    port: 443
    ip: 127.0.0.1
    # ipv6: your_ipv6
    private-key: eCtXsJZ27+4PbhDkHnB923tkUn2Gj59wZw5wFA75MnU=
    public-key: Cr8hWlKvtDt7nrvf+f0brNQQzabAqrjfBvas9pmowjo=
    # preshared-key: base64
    # remote-dns-resolve: true # remote resolve DNS with `dns` field, default is true
    # dns: [1.1.1.1, 8.8.8.8]
    # mtu: 1420
    udp: true

proxy-groups:
  # Relay chains the proxies. proxies shall not contain a relay.
  # Support relay UDP traffic.
  # Traffic: clash <-> ss1 <-> trojan <-> vmess <-> ss2 <-> Internet
  - name: "relay-udp-over-tcp"
    type: relay
    proxies:
      - ss1
      - trojan
      - vmess
      - ss2

  - name: "relay-raw-udp"
    type: relay
    proxies:
      - ss1
      - ss2
      - ss3
        
  - name: "filtering-proxy-providers"
    type: url-test
    url: "http://www.gstatic.com/generate_204"
    interval: 300
    tolerance: 200
    # lazy: true
    filter: "XXX" # a regular expression
    use:
      - provider1

proxy-providers:
  provider1:
    type: http
    url: "url" # support V2Ray subscription URL
    # url-proxy: true # forward to tun if tun enabled
    interval: 3600
    path: ./providers/provider1.yaml
    # filter: "xxx"
    # prefix-name: "XXX-"
    header:  # custom http request header
      User-Agent:
        - "Clash/v1.10.6"
    #   Accept:
    #     - 'application/vnd.github.v3.raw'
    #   Authorization:
    #     - ' token xxxxxxxxxxx'
    health-check:
      enable: false
      interval: 1200
      # lazy: false # default value is true
      url: http://www.gstatic.com/generate_204
```

### Tunnels configuration

tunnels (like SSH local forwarding).

```yaml
tunnels:
  # one line config
  - tcp/udp,127.0.0.1:6553,114.114.114.114:53,proxy
  - tcp,127.0.0.1:6666,rds.mysql.com:3306,vpn
  # full yaml config
  - network: [tcp, udp]
    address: 127.0.0.1:7777
    target: target.com
    proxy: proxy
```

### FAQ

- Q: How to get shell command with external IP?  
  A: Click the clashX menu icon and then press `Option-Command-C`  

### 关闭ClashX的通知

1. 在系统设置中关闭 clashx 的推送权限
2. 在菜单栏->配置->更多设置中选中减少通知

Note：强烈不推荐这么做，这可能导致clashx的很多重要错误提醒无法显示。

### 全局快捷键

- 设置详情点击 [全局快捷键](
