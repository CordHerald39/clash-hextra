---
{
  "title": "Clash 怎样确认应用填写的是本地 HTTP 入口",
  "description": "Clash 怎样确认应用填写的是本地 HTTP 入口。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T18:59:12.559349+00:00",
  "lastmod": "2026-10-04T18:59:12.559349+00:00",
  "type": "post"
}
---

确认应用填写的是“本地 HTTP 入口”，是指：流量送到本机 Clash 的 HTTP（或 mixed）入站，而不是局域网里另一台主机，也不是本机的外部控制 API。主机、端口、协议三项里任何一项指向别处，都不能称为本地 HTTP 入口。

## 适用条件

适用于与 Clash 运行在同一台设备上的浏览器、开发工具或支持 HTTP 代理的应用。官方全局配置用 `bind-address` 说明可绑定 `"*"`、单个 IPv4 或单个 IPv6；用 `skip-auth-prefixes` 指定可跳过 http(s)/socks/mixed 验证的 IP 段，默认包含 `127.0.0.1/8` 与 `::1/128`；用 `allow-lan` 控制其他设备能否经过代理端口访问互联网。`ipv6` 决定是否接受 IPv6 流量。`external-controller` 示例为 `127.0.0.1:9090`，以及可选的 `external-controller-unix`、`external-controller-pipe`、`external-controller-tls`，都属于 API 监听，不是 HTTP 入口。

应用若运行在另一台设备，应改用被绑定且被 `allow-lan`、`lan-allowed-ips` 允许、又不在 `lan-disallowed-ips` 中的地址，不能再按“本地入口”来判断。

## 确认步骤

第一步，从生效配置中抄下 HTTP 或 mixed 入站端口，只把该端口填进应用的 HTTP 代理端口。不要填写 API 端口，也不要填写仅用于 SOCKS 的入站端口（应用明确走 mixed 所支持协议时除外）。

第二步，主机使用本机回环：IPv4 用 `127.0.0.1`；若应用走 IPv6 且 `ipv6` 为 true，可用 `::1`。`bind-address` 为 `"*"` 时回环通常可用；若只绑定了某个单地址，则必须填那个地址。把当前局域网 IP 填进本机应用，不能证明走的是本地入口，地址变化后还会失效。

第三步，核对鉴权是否与“本地”来源一致。默认情况下，来自 `127.0.0.1/8` 与 `::1/128` 的连接可跳过 `authentication`。若你缩小了 `skip-auth-prefixes`，本机应用也必须填写对应用户名和密码。不要把 API 的 `secret` 当作 HTTP 代理密码。

第四步，确认应用协议是 HTTP 代理。系统或浏览器里若选成 SOCKS，而入站只提供 HTTP，则即使地址是 `127.0.0.1`，也不是正确的本地 HTTP 入口用法。

## 判断依据与失败后的处理

可认定为本地 HTTP 入口的依据：主机是回环或 `bind-address` 明确提供给本机的地址；端口是 HTTP/mixed 入站端口；协议是 HTTP 代理；日志中的来源为 `127.0.0.1` 或 `::1`，而不是其他设备的局域网地址。`allow-lan` 为 false 时，其他设备不能用代理端口，但本机回环仍可作为本地入口，二者不要当成同一个开关。

若应用连不上：先排除误填 `127.0.0.1:9090` 这类外部控制地址。将 `log-level` 设为 `info` 或 `debug`，看入站来源 IP。来源是局域网地址，说明填写的不是本地入口，应改回回环并检查该应用是否走了系统级局域网代理。`ipv6` 为 false 时不要坚持使用 `::1`。绑定失败时，应调整 `bind-address` 或入站本身，而不是在应用里改成未在配置中出现的端口。

https://wiki.metacubex.one/config/general/
