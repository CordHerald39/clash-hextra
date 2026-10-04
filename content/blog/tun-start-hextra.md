---
{
  "title": "Clash 怎样验证启用 TUN 后目标应用产生了连接",
  "description": "Clash 怎样验证启用 TUN 后目标应用产生了连接。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T19:24:25.024874+00:00",
  "lastmod": "2026-10-04T19:24:25.024874+00:00",
  "type": "post"
}
---

启用 Tun 后，要验证的是目标应用的包是否经虚网卡进入内核，而不是应用界面是否仍显示“已连接”。适用条件是：`tun.enable` 为 `true`，并且按平台打开了 `auto-route`（Linux 重定向 TCP 时还需要 `auto-redirect`）。判断依据来自手册：流量应被路由或重定向到指定 `device`，匹配 `dns-hijack` 的查询应进入内部 DNS；若配置了 UID、包名、MAC 或接口过滤，则只有命中范围的来源才会被接管。

## 先确认 Tun 设备与路由已经接管

查看配置中的 `device`。MacOS 只能使用 `utun` 开头的网卡名，名称不合规则时谈不上应用连接验证。`auto-route` 会自动把全局流量导入 tun；若改用 `route-address`/`route-exclude-address`，则只验证这些网段，而不是默认全局。Linux 上可再核对 `iproute2-table-index`（默认 2022）与 `iproute2-rule-index`（默认 9000）是否已生成。

目标应用若在过滤范围外，不会产生经 Tun 的连接：Linux 未列入 `include-uid` 的用户默认不被路由；Android 未列入 `include-package` 的包不会被路由，`exclude-package` 会排除；`include-interface`/`exclude-interface` 会限制来源网卡。验证前先确认该应用、用户、网卡未被排除。Android 常用用户 ID 在手册中给出：机主 `0`，手机分身 `10`，应用多开 `999`。未包含对应 `include-android-user` 时，不要用这些用户里的应用做验证。

## 再核对 DNS 劫持与协议栈是否真正工作

`dns-hijack` 会把匹配连接导入内部 DNS，未写协议时视为 UDP。若目标应用的解析仍走局域网 DNS，在 MacOS/Windows 上属于手册写明的无法自动劫持；Android 若开启私人 DNS，同样无法自动劫持。这两种情况只能说明 DNS 入口未进内部模块，不能单独证明 Tun 入站失败，需要结合是否已有 IP 流量进入虚网卡来判断。

手册提供协议栈网络回环测试，测试对象为 `system`/`gvisor`/`lwip`，并注明仅供参考，平台为 Linux，Windows 与 MacOS 可能有差异。该测试用于判断所选 `stack` 的协议栈能否完成回环，不能当成跨平台性能结论。若打开防火墙，`system` 与 `mixed` 不可用，除非已按文档放行内核；此时即使应用在发请求，也可能根本进不了 Tun。`mixed` 为 TCP 走 `system`、UDP 走 `gvisor`，验证 UDP 应用与 TCP 应用时应分开看，不要用一种协议的结果推断另一种。

## 按应用类型取证与失败时下一步

对普通客户端：在过滤条件允许的前提下发起连接，观察是否出现经 tun 网卡的四层会话，且目的地址不属于 `route-exclude-address` 或 `route-exclude-address-set`。对 Android 指定包名：只对 `include-package` 内应用取证，对 `exclude-package`（文档示例包含门户登录类包名）应看到其不进 Tun。对 Linux 局域网设备：在已启用 `auto-route` 与 `auto-redirect` 时，用 `include-mac-address`/`exclude-mac-address` 核对来源 MAC 是否被路由。

若应用有请求但虚网卡无会话：先查 `auto-route` 是否开启；Linux 还要查 `auto-redirect` 是否在非 Linux 误配，或是否与 `routing-mark` 冲突；再查防火墙是否拦截 `system`/`mixed`。若只有 DNS 无 TCP/UDP：按平台核对局域网 DNS 与私人 DNS 限制，并确认 `dns-hijack` 写法（`any:53` 与 `tcp://any:53`）。若 IPv6 应用无连接：确认系统其他网卡是否已有 IPv6，否则 tun v6 会被禁用；强制开启需 `SKIP_SYSTEM_IPV6_CHECK=1` 且顶层 `ipv6: true`。仍无法证明接管时，缩小为最小 Tun 配置后复测单一应用，再加回过滤项，避免把规则集绕过误判成应用未连接。

资料：https://wiki.metacubex.one/config/inbound/tun/
