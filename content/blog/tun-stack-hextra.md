---
{
  "title": "Clash 怎样设计只改变协议栈的一次对照",
  "description": "Clash 怎样设计只改变协议栈的一次对照。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T19:24:25.026407+00:00",
  "lastmod": "2026-10-04T19:24:25.026407+00:00",
  "type": "post"
}
---

要验证 Clash（mihomo）TUN 协议栈本身的影响，对照必须只改变 `stack`，其余入站项保持不变。官方可用值是 `system`、`gvisor`、`mixed`、`mips`。一次对照只在这四个值之间替换其中一个，并记录系统、防火墙和「仅某栈生效」的字段是否被误触。

## 固定对照范围：只改 stack，不改路由与劫持

先写出一份基线：`enable`、`auto-route`、`auto-redirect`、`auto-detect-interface`、`dns-hijack`、`device`、`mtu`、`strict-route`、`gso` 等保持原样，仅替换 `stack`。这样结果才能对应文档中的栈差异：`system` 为系统协议栈，`gvisor` 为用户空间协议栈，`mixed` 为 TCP 走 system、UDP 走 gvisor，`mips` 为自研 IP 协议栈。

判断依据：若同时改了 `auto-route` 或 `dns-hijack`，失败或恢复都无法归因到栈。`mixed` 尤其需要在记录里分开 TCP 与 UDP 观察，因为它本来就不是单一实现。文档对默认值的表述是默认 mips，如无使用问题建议使用 mips 栈，因此基线优先落在 `mips`，再向其他值做单次切换。

## 把防火墙和操作系统写成对照的前置条件

打开防火墙时无法使用 `system` 和 `mixed`。若对照包含这两项，必须先按文档完成放行，否则比较的是过滤策略而不是协议栈。Windows：设置 → Windows 安全中心 → 允许应用通过防火墙 → 选中内核。MacOS 一般无需配置；若开启防火墙无法使用，可尝试系统设置 → 网络 → 防火墙 → 选项 → 添加 mihomo app。Linux 一般无需配置；必要时 `sudo iptables -A OUTPUT -o Mihomo -j ACCEPT`（示例网卡名 Mihomo）。

操作系统也要固定。页面中的协议栈网络回环测试注明仅供参考，平台为 linux，Windows 和 MacOS 可能会有差异。因此一次对照不要跨系统更换主机。MacOS 上 `device` 只能使用 utun 开头的网卡名，这项属于环境约束，对照期间不要改名。

## 从对照中剔除不会随栈一起变化的项

`congestion-controller` 仅在 mips 时生效。基线若是 `mips` 且配置了该字段，改到其他栈后该项应视为停用，而不是再调它来「补对照」。`gso` 仅 Linux；`auto-redirect` 仅 Linux 且依赖 `auto-route`。`endpoint-independent-nat` 文档写明不需要时不建议开启。这些字段在一次只改栈的实验里应保持关闭或保持原值，避免引入第二种变量。

步骤：保存基线配置 → 确认防火墙与系统满足拟测栈的条件 → 只改 `stack` 并重载 → 按 TCP/UDP 与能否建立入站记录结果 → 改回基线后再测下一个值。失败时下一步：拟测 `system`/`mixed` 时先核防火墙；结果与 UDP 相关时优先对比 `mixed` 与单一栈；附属字段有无效果与文档生效条件不符时，把该字段移出本次对照，另开实验，而不是在同一次修改里同时动栈和路由。

资料：https://wiki.metacubex.one/config/inbound/tun/
