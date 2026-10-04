---
{
  "title": "Clash 怎样记录游戏进程、协议和失败阶段",
  "description": "Clash 怎样记录游戏进程、协议和失败阶段。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T21:37:45.963264+00:00",
  "lastmod": "2026-10-04T21:37:45.963264+00:00",
  "type": "post"
}
---

记录游戏连不上时的进程、协议和失败阶段，目的是把现象钉在 TUN 文档已给出的字段上，而不是事后追述。Android 可用包名表达进程，Linux 可用 UID，协议用 TCP 与 UDP 对照堆栈，失败阶段用接管、DNS、NAT 和路由排除来分段。以下只依据 TUN 文档。

## 适用条件与进程如何落记录

适用条件是：需要把哪一个游戏和有没有被 TUN 路由写成可核对的记录。Android 应用规则仅在 Android 下被支持，并且需要 `auto-route`。应记录 `include-package` 与 `exclude-package` 中的包名：前者使列出的应用被 Tun 路由，未配置的应用包不会被路由；后者使指定包避免被路由。同时记录 `include-android-user`，文档给出常用用户 ID：机主 0、手机分身 10、应用多开 999。若游戏装在分身或应用多开用户下，只记录机主包名会把失败阶段误判到协议。

Linux 上 UID 规则仅在 Linux 被支持并且需要 `auto-route`。记录 `include-uid`、`include-uid-range`、`exclude-uid`、`exclude-uid-range`。未出现在包含列表中的用户不会被路由。接口层记录 `include-interface` 与 `exclude-interface`（二者冲突，不可一起配置）。Linux 在 `auto-route` 与 `auto-redirect` 同时启用时，还可记录 `include-mac-address` 与 `exclude-mac-address`。以上字段就是文档提供的进程与来源记录方式；不要改写成未在该页出现的操作名称。

## 协议如何对照堆栈来记

`stack` 可用 system、gvisor、mixed、mips。记录游戏使用的传输层后，再对照堆栈：mixed 下 TCP 使用 system、UDP 使用 gvisor，因此必须分开记录 TCP 阶段与 UDP 阶段。文档说明打开防火墙则无法使用 system 和 mixed，并分别给出 Windows、MacOS、Linux 的放行方式；若游戏失败发生在刚启用这两类堆栈之后，应把失败阶段记为防火墙未放行，而不是规则未命中。文档写明如无使用问题建议使用 mips，默认 mips；记录时应写下实际取值，而不是只写已开 TUN。

`dns-hijack` 记录解析阶段：匹配连接导入内部 dns 模块，不写协议则为 udp://。同时注明平台限制：MacOS 与 Windows 无法自动劫持发往局域网的 dns；Android 私人 dns 无法自动劫持。若记录显示游戏服务器名依赖 DNS 而网页已用固定地址，失败阶段应标在劫持，而不是标在堆栈。

UDP 游戏必须记录 `udp-timeout`（默认 300 秒）以及是否启用 `endpoint-independent-nat`（文档说明不需要时不建议开启）。TCP 且 `stack` 为 mips 时，可记录 `congestion-controller`，可选 cubic、reno、bbr、bbr3，默认 cubic，仅 mips 生效。这些才是文档里与协议相关的可记录项。`mtu`、Linux 上的 `gso` 与 `gso-max-size` 只在需要对照极限传输时附加，一般会话失败不必放在协议栏首位。

## 失败阶段划分与下一步

建议把一次失败写成四个阶段并逐项核对：一是接管阶段，记录 `enable`、`auto-route`，Linux 再记 `auto-redirect`，Android 注明是否仅本地 IPv4；二是来源阶段，记录包名、UID、用户 ID、接口或 MAC 是否把游戏排除在外；三是解析与路由阶段，记录 dns-hijack 是否受平台限制，以及 `route-exclude-address`、`route-address-set` 与 `route-exclude-address-set`（后两者仅 Linux，需 nftables 且 `auto-route` 与 `auto-redirect` 已启用，并与 `routing-mark` 冲突）；四是会话阶段，记录 `stack`、防火墙、`udp-timeout` 和 NAT。

`device` 在 MacOS 必须是 utun 开头，网卡名不符合时应记为设备阶段失败。`strict-route` 在 Windows 上可能影响部分应用程序，若失败与其同时出现，应单独记录该项是否启用。`inet6-address` 会在启动时检查系统其他网卡是否有 IPv6，不存在会禁用；游戏若走 IPv6，还要记下是否设置 `SKIP_SYSTEM_IPV6_CHECK=1` 以及顶层 `ipv6` 是否为 true。`iproute2-table-index` 与 `iproute2-rule-index` 可在 Linux 路由被占用时作为接管阶段的附注，默认 2022 与 9000。

失败后下一步：哪一阶段没有记录，就只补该阶段对应字段，不要跨阶段改堆栈或 NAT。来源阶段显示未被路由时，先调整 include 与 exclude，而不是增加 dns-hijack 条目。会话阶段超时则核对该 UDP 超时值。Linux 集合类路由与 `routing-mark` 冲突时，先消除冲突再重复记录一次通断。

资料：https://wiki.metacubex.one/config/inbound/tun/
