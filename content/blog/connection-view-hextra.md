---
{
  "title": "Clash 怎样核对目标、策略和请求时间",
  "description": "Clash 怎样核对目标、策略和请求时间。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T19:24:25.021808+00:00",
  "lastmod": "2026-10-04T19:24:25.021808+00:00",
  "type": "post"
}
---

核对 Clash 的目标、策略和请求时间，应使用全局配置里已经定义的运行模式、策略组选择持久化、日志输出以及与时间相关的保活和延迟项，而不是臆造列表字段。官方文档把 `mode` 写成 rule、global、direct 三种；把 API 对策略组的选择能否在下次启动恢复写成 `profile.store-selected`；把可观察的运行信息写成 `log-level` 输出到控制台和控制页面。请求是否“发生在刚才那一次点击”，要用复现窗口和保活参数来约束，避免把旧映射或保活包当成当次业务请求。

## 适用条件：先固定模式再谈策略是否一致

`mode` 默认为 `rule`，按规则匹配目标。`global` 不走逐条规则决定出站，而是要求在 GLOBAL 策略组选择代理或策略；未选择时谈“目标对应哪条策略”没有依据。`direct` 全局直连，目标即使出现在连接观察中，策略核对的结论也应是直连，而不是某个代理节点。三种模式混用同一次记录，目标和策略会对不齐。

`profile.store-selected` 为 true 时，API 对策略组的选择会被储存，供下次启动使用。核对策略前要知道当前选择来自这次 API 操作还是上次落盘。为 false 时，重启后组选择可能回到配置默认，记录中的出站与你记忆中的节点不一致。`find-process-mode` 为 `always` 强制匹配进程，`strict` 由内核判断是否开启，`off` 不匹配。目标相同但进程匹配开关不同，规则里带进程条件时选出的策略会变，必须把该项与模式一起记录。

判断依据：写下当前 `mode`、GLOBAL 是否已选择（仅 global）、`store-selected` 是否启用、`find-process-mode` 取值。缺一项就不能断言“这个目标应该走某策略”。

## 怎样核对目标本身没有指错地址族或网卡

目标要同时核域名、地址族和出站口。`ipv6` 默认为 true；为 false 时内核不接受 IPv6 流量，此时记录里不应出现 IPv6 目标，若页面仍请求 IPv6，属于目标未进入内核，不是策略误判。`tcp-concurrent` 为 true 时，会对解析得到的全部 IP 连接并取第一个成功的，核对应列出本次使用的 IP，而不是只看域名。`interface-name` 指定出站网卡；Linux 的 `routing-mark` 给出站默认标记。目标正确但网卡或标记错误，策略名称对了，包仍可能从错误路径离开。

`profile.store-fake-ip` 为 true 会储存 Fake-IP 映射，域名再次连接使用原有映射地址。核目标时若发现 IP 与当前公共解析不一致，应先考虑这是持久化映射，而不是策略把流量送到了错误主机。`authentication` 与 `lan-disallowed-ips` 可能让某来源根本到不了该目标，此时没有可核对的策略结果。

## 请求时间的可用判据以及失败后的下一步

全局配置没有单独的“连接时间戳字段”说明，时间只能从控制台/控制页面日志的输出顺序，以及保活、延迟相关项来约束。`log-level` 为 `info` 或 `debug` 时才能看到一般运行或尽量完整的信息；`silent` 无法对时间。`keep-alive-interval` 与 `keep-alive-idle` 单位为秒，超时后的包不应算作同一次页面请求。`disable-keep-alive` 在 Android 上强制为 true。`unified-delay` 用 RTT 消除握手带来的延迟差异，可比较节点延迟，不能当请求发生时刻。`geo-update-interval` 是 GEO 更新间隔（小时），与单次请求时间无关，不要拿来对表。

失败时下一步：把 `mode` 固定为当前要验证的值，必要时查看 `store-selected` 是否把组选择恢复成旧值；打开 `log-level` 为 `info`/`debug` 后立即访问该目标，只采这一小段日志。IPv6 目标检查 `ipv6`；多 IP 域名检查 `tcp-concurrent`；进程条件检查 `find-process-mode`。出站网卡与标记用 `interface-name`、`routing-mark` 对照系统接口。若 API 看得到策略切换但日志对不上本次访问，检查 `external-controller` 是否连到另一实例。仍无法对齐时，先排除 Fake-IP 旧映射与保活包，再谈规则是否写错。

资料：https://wiki.metacubex.one/config/general/
