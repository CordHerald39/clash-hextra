---
{
  "title": "Clash 怎样记录 TUN 前后到同一目标的表现",
  "description": "Clash 怎样记录 TUN 前后到同一目标的表现。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T19:24:25.028695+00:00",
  "lastmod": "2026-10-04T19:24:25.028695+00:00",
  "type": "post"
}
---

要记录 Clash TUN 开启前后到同一目标的表现，只能依据官方对 auto-route 如何改变流量路径、不同协议栈以及回环测试的说明来设计对比，而不能添加文档未给出的测试命令、按钮或效果保证。开启前后路径是否切换，取决于流量是否被导向 tun 网卡。

## 适用条件

对比的前提是能够在同一目标、同一平台上分别观察 tun.enable 或 auto-route 关闭与开启两种状态。auto-route 为 true 时自动将全局流量路由进入 tun，路径相对系统原路由会改变。stack 可选 system、gvisor、mixed、mips，文档建议无使用问题则使用 mips，并提供协议栈网络回环测试（system/gvisor/lwip 顺序，仅供参考，平台为 Linux，Windows 与 macOS 可能有差异）。route-address、route-exclude-address 会改变该目标是否被纳入路由。strict-route、auto-redirect、dns-hijack 也会改变实际路径或 DNS。gso 仅 Linux。未满足 auto-route 依赖时，开启 TUN 并不会按文档把该目标导入 tun，对比无意义。

## 具体记录步骤

固定目标地址与端口，先在 auto-route 为 false（或 enable 为 false）时记录系统原路径下的连通与解析情况。再改为 auto-route true，确认该目标 IP 是否落在 route-address 内、是否未被 route-exclude-address 排除（文档排除示例为 192.168.0.0/16 等）。Linux 可同时记下 iproute2-table-index（默认 2022）和 iproute2-rule-index（默认 9000）是否生成指向 tun 的规则。切换 stack 时，对照文档中的协议栈网络回环测试说明，分别记录 system、gvisor、mixed、mips 下到同一目标的差异，注意仅作参考且跨平台不可直接比数值。查看 dns-hijack 是否把 53 端口导入内部 DNS，从而改变“同一目标”的解析结果。记录 device、mtu、udp-timeout（默认 300 秒）是否在两次测试中保持一致。Android 注意私人 DNS，Windows/macOS 注意局域网 DNS 无法自动劫持。

## 判断依据

若 auto-route 启用后该目标不再走系统默认路由而进入 tun 网卡，即符合“自动将全局流量路由进入 tun 网卡”的描述，前后路径差异来自捕获而非应用层。若目标落在 route-exclude-address 中，开启后仍应绕过，表现应接近关闭状态。strict-route 为 true 时 Linux 会将所有连接路由到 tun，前后差异会更大。文档中的回环测试用于比较不同栈，不能当作开启前后的性能保证。dns-hijack 在部分平台对局域网无效，若目标依赖局域网 DNS，前后解析路径可能不一致。endpoint-independent-nat 可能影响 UDP 到同一目标的表现。用“是否进入 tun、是否被排除、栈是否改变”这三项对照文档，即可判断记录到的差异是否由 TUN 路由引起。

## 失败时下一步

前后结果无法对应时，确认目标地址完全相同，且两次测试未改 route-address 或 exclude 列表。检查防火墙是否在开启后拦截（Windows 需放行内核，Linux 可放行 TUN 网卡出站）。尝试只改 enable 或只改 auto-route 单项对比。Linux 核对实际路由表是否使用文档默认索引。IPv6 目标需满足 inet6-address 启用条件（系统有 IPv6 或 SKIP_SYSTEM_IPV6_CHECK=1，且顶层 ipv6 为 true）。不要引入文档未记载的外部基准数值。若仅栈不同导致差异，回到文档回环测试说明，限定在同一操作系统内比较。

https://wiki.metacubex.one/config/inbound/tun/
