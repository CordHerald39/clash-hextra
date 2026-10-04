---
{
  "title": "Clash 怎样验证浏览器到网关的访问路径",
  "description": "Clash 怎样验证浏览器到网关的访问路径。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T21:37:45.967635+00:00",
  "lastmod": "2026-10-04T21:37:45.967635+00:00",
  "type": "post"
}
---

验证浏览器到网关的访问路径，需要沿着“浏览器给出的目的 → 系统路由是否把该目的送进 Tun → 解析是否被劫持 → 接口与设备过滤是否把这次访问算进路由范围”对照官方 Tun 字段。Clash（mihomo）文档不提供图形化路径，结论只能来自这些开关的合成结果。

## 适用条件

适用于本机 Tun 已启用，需要确认访问网关管理地址时走的是连接路由器的网卡，还是 `device` 指定的虚接口。MacOS 上该名称只能使用 utun 开头。`auto-route` 为 true 时，全局流量默认进入 Tun；此时验证的核心是目的地址是否被 `route-exclude-address` 从自动路由中排除。`auto-route` 为 false 时，不存在这套自动全局路由，验证应观察系统原有默认路由，不要用排除列表解释路径。Linux 上 `auto-redirect` 在 `auto-route` 启用时可自动配置 iptables 或 nftables 重定向 TCP，文档称带有 `auto-route` 的 `auto-redirect` 可以在路由器上按预期工作。重定向改变的是 TCP 如何进入核心，不会把网关 IP 改写成公网地址。

## 用字段判断会不会进虚接口

第一步，记录浏览器访问的目的，能写成 IP 就不要停在名称。第二步，用该 IP 对比 `route-exclude-address`。示例中 `192.168.0.0/16` 覆盖该前缀内全部 IPv4；网关在其中，则在启用 `auto-route` 时应排除该自定义网段。不在其中，路径仍可能进入 Tun。`route-address` 若改为自定义集合，路径是否进 Tun 取决于“集合包含该目的且排除列表未包含”。Linux 的 `route-exclude-address-set` 是匹配则绕过，`route-address-set` 是不匹配则绕过，二者都需要 nftables 以及 `auto-route`、`auto-redirect`，并与 `routing-mark` 冲突；条件不成立时不能把规则集当作路径证据。旧的 `inet4-route-exclude-address` 与 `inet6-route-exclude-address` 即将废弃，核对时应标明用的是哪一组。

第三步，看接口。`include-interface` 限制被路由的接口，`exclude-interface` 排除接口，二者冲突、不可一起配置。流量从哪块物理网卡进入，决定它有没有资格被自动路由进 Tun。`auto-detect-interface` 自动选择流量出口接口，多出口网卡同时连接时建议手动指定出口网卡。访问网关通常应走连接路由器的那块网卡；自动检测影响的是进隧道之后的出站口，不要把两跳当成同一跳。

第四步，Linux 核 MAC 与 UID。`include-mac-address` 与 `exclude-mac-address` 需 `auto-route` 与 `auto-redirect`，按来源 MAC 限制或排除局域网设备。源 MAC 被排除时，路径不会按“被 Tun 路由”处理。`include-uid`、`exclude-uid` 及对应范围仅在 Linux 支持且需要 `auto-route`：未被包含的用户不会被 Tun 路由。Android 则核 `include-android-user`、`include-package`、`exclude-package`。浏览器所属用户或包名落在“不被路由”侧时，核心侧根本未进 Tun。

第五步，看解析。`dns-hijack` 将匹配连接导入内部 DNS 模块，不书写协议则为 UDP。MacOS 与 Windows 无法自动劫持发往局域网的 DNS 请求；Android 私人 DNS 无法自动劫持。使用域名验证时，解析可能在劫持范围外完成，随后连接的 IP 已不是预期网关。判断依据：同一管理页用 IP 访问与用名称访问若走向不同，视为解析路径与传输路径分裂，验证以 IP 为准。`strict-route` 会加强是否允许不经过 Tun 到达。Linux 上防止地址泄漏并把连接路由到 Tun；若目的是确认可以直连网关，必须同时看到排除网段已包含该网关，否则严格路由下的预期路径是进 Tun。

## 得不出路径结论时的下一步

若无法判断流量有没有进入指定虚接口，回到配置确认 `enable`、`auto-route`、排除 CIDR 三条，而不是调整 `udp-timeout`、`gso`、`congestion-controller` 等与路径选择无关的项。IPv6 路径还要确认启动检查未关闭 Tun 的 v6，顶层 `ipv6` 为 true；需要跳过系统检查时使用 `SKIP_SYSTEM_IPV6_CHECK=1`。防火墙导致 system 或 mixed 栈不可用时，路径会在协议栈层失败，表现为完全打不开，而不是改走另一块网卡。按文档在对应系统放行后再做一次目的 IP 对比。验证记录应写明：目的 IP、是否落在排除 CIDR、`auto-route` 是否开启、接口与设备过滤是否把本次访问算入 Tun。四项齐全才能给出路径结论。

参考资料：https://wiki.metacubex.one/config/inbound/tun/
