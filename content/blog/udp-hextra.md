---
{
  "title": "Clash 怎样记录应用使用协议与接管方式",
  "description": "Clash 怎样记录应用使用协议与接管方式。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T21:03:35.930396+00:00",
  "lastmod": "2026-10-04T21:03:35.930396+00:00",
  "type": "post"
}
---

要记录某个应用实际走哪类连接、以及流量如何被内核接管，应依赖全局配置里的日志级别、进程匹配、运行模式、外部控制接口和选择是否被保存，而不是事后回忆一次界面状态。该页没有提供“按应用导出协议清单”的专用开关，因此记录方法是：把当时生效的字段与日志里能看到的信息写成可复核的条目，并标明哪些只能描述 TCP 行为。

## 适用条件与必须写入的接管上下文

适用条件是：需要留下可核对记录，说明某应用流量是否进入内核、以何种运行模式处理、进程是否被匹配、日志级别能否看到对应时间点。记录目标至少包括：`mode`、`find-process-mode`、`log-level`、`ipv6`、出站接口与路由标记、以及 API 侧策略选择是否保存。

`mode` 为 rule、global 或 direct，默认为规则模式。direct 表示全局直连；rule 表示规则匹配；global 表示全局代理，并且需要在 GLOBAL 策略组选择代理或策略。判断依据：记录中必须有模式字段，否则“谁接管”无法复述。global 时还要记下 GLOBAL 当时的选择对象，不能只写“已开代理”。

`ipv6` 为 true 或 false，决定内核是否接受 IPv6 流量。若应用使用 IPv6 而该项为 false，记录应写“内核不接受 IPv6”，不要写成应用自己的协议失败。`allow-lan`、`bind-address`、`lan-allowed-ips`、`lan-disallowed-ips` 决定其它设备能否进入代理端口，黑名单优先于白名单。本机应用与局域网应用要分开记录来源地址是否落在允许段。

## 操作步骤：日志、进程匹配与 API 选择

第一步，设定 `log-level`。要尽可能记录运行中信息时用 debug；只需一般运行内容用 info；关心不影响运行的错误用 warning；仅无法使用的错误用 error；silent 不输出，无法用于记录。日志仅在控制台和控制页面输出。判断依据：输出中应能对应到应用发起连接的时间点；级别导致没有条目时，不能填写“未出现即未发生”。

第二步，设定 `find-process-mode`。需要强制记录所有进程时用 always；默认 strict 表示由 Clash 判断是否开启；off 不匹配进程，文档推荐路由器使用。判断依据：只有在 always，或 strict 下实际已开启匹配时，记录里才适合填写应用进程名。off 时接管方式不能写成按进程，只能注明进程匹配已关闭，其余信息来自日志中的接受、拒绝或出站结果。

第三步，把 TCP 行为从“协议”栏拆出去。`keep-alive-interval`、`keep-alive-idle`、`disable-keep-alive` 只描述 TCP Keep Alive，Android 上禁用保活强制为 true。`tcp-concurrent` 只描述 TCP 并发连接：使用 DNS 解析出的所有 IP，并使用第一个成功的连接。判断依据：这些项只能作为 TCP 连接行为的附注，不能填成应用层协议名称，也不能填成 UDP 已确认。

第四步，用外部控制接口记录策略选择。`external-controller` 为 API 监听地址，`secret` 为访问密钥。Unix socket 与 Windows namedpipe 访问 API 不会验证 secret；HTTPS-API 还需要 TLS 证书与私钥，且使用 TLS 时仍必须填写 `external-controller`。`profile.store-selected` 为 true 时储存 API 对策略组的选择，供下次启动使用；`store-fake-ip` 储存 fakeip 映射，域名再次连接时使用原映射地址。判断依据：当前接管所用的策略组选择，应以 API 读到的结果和是否开启 store-selected 为准；DNS 侧若涉及 fakeip，记录中要写明映射是否被保存。

第五步，记录出站层。`interface-name` 为流量出站接口；`routing-mark` 为 Linux 出站默认流量标记。判断依据：接管方式不仅是“进了内核”，还包括从哪一个接口、带什么标记离开；这两项未记录时，不能把入站成功写成完整接管路径。

## 记不到信息时的下一步

若日志为 silent 或没有对应时间点：下一步把级别改为 debug 后复现一次，只保留与该应用时间窗口重合的输出。仍无进程名时，检查是否为 off；路由器上按文档本就不匹配进程，记录应改为“进程匹配关闭”，不要留空进程栏假装已识别。

若 API 读不到选择：下一步核对 `external-controller` 地址与 `secret`，并确认当前用的是需校验 secret 的通道，还是 Unix socket、namedpipe 这类不验证 secret 的通道。不要在记录里填写并未从 API 读到的结果。重启后接管方式变化时，查看 `store-selected` 是否为 true；为 false 时必须写明选择未持久化。

若只能确认 TCP 行为、无法确认应用是否还使用其它协议：下一步在记录中明确写上“该全局配置未提供按协议类型的专用开关，TCP Keep Alive 与 TCP 并发不能代表全部协议”。记录只反映当时字段与日志级别下能看到的信息，不构成对应用层协议识别的效果保证。

https://wiki.metacubex.one/config/general/
