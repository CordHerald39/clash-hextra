---
{
  "title": "Clash 怎样用一条请求验证排除范围",
  "description": "Clash 怎样用一条请求验证排除范围。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T19:24:25.029981+00:00",
  "lastmod": "2026-10-04T19:24:25.029981+00:00",
  "type": "post"
}
---

## 先固定“一条请求”对应的排除维度

Clash（mihomo）官方 TUN 文档没有给出独立的探测命令，但排除语义是按维度写死的：目的网段、规则集 CIDR、入接口、UID、MAC、Android 包名。用一条请求验证排除范围，是指只选一个维度、一个目标，核对该请求是否属于“应绕过路由、不进 TUN”的集合，而不是同时改多条列表。

适用条件仍是文档中的依赖：`tun.enable` 为 true；网段排除需要 `auto-route`；Linux 上 `route-exclude-address-set` 还需要 nftables 以及 `auto-redirect`；UID 需要 Linux + `auto-route`；MAC 需要 Linux + `auto-route` + `auto-redirect`；包名需要 Android + `auto-route`。不满足时，验证结果只能说明平台条件不足，不能说明 CIDR 写错。

操作上先写出这一条请求的五元组中与排除字段对应的那一项。例如验证 `route-exclude-address` 时，只关心目的 IP 是否落在已声明网段（文档示例为 `192.168.0.0/16`、`fc00::/7`）；验证 `exclude-interface` 时只关心请求从哪块网卡出去；验证 `exclude-package` 时只关心发起应用的包名。不要用域名规则或 DIRECT 作为这一步的判据，那些不是 TUN 排除字段。

## 用文档语义做通过/失败判断

`route-exclude-address-set` 的官方说明是：将指定规则集中的目标 IP CIDR 加入防火墙，匹配的流量将绕过路由。因此“一条请求验证”的判断依据是：该请求的目的 IP 是否属于规则集里的 CIDR。属于则应绕过路由；不属于则仍可能被 `auto-route` 送入 TUN。`route-address-set` 相反：不匹配的流量绕过路由，匹配的才进入防火墙放行路径，验证时不要和 exclude 搞反。

`route-exclude-address` 在启用 `auto-route` 时排除自定义网段。判断依据是：目的地址落在列表内，则该请求不应再按默认全局路由进 TUN。`route-address` 是自定义“要路由的网段”，一般无需配置，不能拿来当排除验证。`include-*` 与 `exclude-*` 成对出现时，包含未配置即不路由，排除是使已可能被路由的对象避开；Android 上未配置的包不会按包含逻辑被 TUN 路由。一条请求若来自未包含的 UID/用户/包名，结论是“不在包含范围”，不是“排除列表命中”。

接口、MAC、UID 的验证同样只改一个变量。`include-interface` 与 `exclude-interface` 冲突，同时配置时无法用一条请求得出有效结论，必须先删掉一方。`route-*-address-set` 与 `routing-mark` 冲突时同样不要做范围验证。IPv6 请求还要确认启动时系统是否已有 IPv6、顶层 `ipv6` 是否为 true，否则 v6 排除范围会被直接禁用。

## 平台限制会让“一条请求”失真时怎么处理

DNS 类请求不能当作通用排除验证：`dns-hijack` 在 macOS/Windows 无法自动劫持发往局域网的 DNS，Android 开启私人 DNS 时也无法自动劫持。对 `any:53` 的验证会混入劫持失败，而不是排除范围错误。`strict-route` 在 Linux 会让不支持的网络无法到达并把连接路由到 TUN；在 Windows 会加防火墙规则抑制多宿主 DNS 泄露，也可能影响 VirtualBox。这类请求失败应先记下 `strict-route` 是否开启。

防火墙与协议栈也会干扰。打开防火墙时无法使用 system 和 mixed 栈，需按文档放行内核后再验证。macOS 的 TUN 名必须 `utun` 开头，设备名不对时任何一条请求都不能代表排除范围。Android 的 `auto-redirect` 仅转发本地 IPv4，用热点对端的一条请求不能验证本机排除列表。

## 验证失败时的下一步

若目的 IP 确在 `route-exclude-address` 或规则集 CIDR 内仍进 TUN：先查 OS 是否支持该字段、nftables/`auto-redirect` 是否缺失、是否与 `routing-mark` 冲突、是否还在用即将废弃的 `inet4-route-exclude-address` 等旧键。若请求维度与字段不一致（用包名请求去验证网段排除），改选匹配该字段的一条请求，而不是扩大列表。平台不支持该维度时停止验证，回到支持项核对，不要把规则 DIRECT 的命中当成 TUN 排除范围已证实。

资料来源：https://wiki.metacubex.one/config/inbound/tun/
