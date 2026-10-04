---
{
  "title": "Clash 电脑端：怎样记录唤醒前后的网络和连接状态",
  "description": "Clash 电脑端：怎样记录唤醒前后的网络和连接状态。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T21:37:45.959282+00:00",
  "lastmod": "2026-10-04T21:37:45.959282+00:00",
  "type": "post"
}
---

要比较唤醒前后的网络和连接状态，必须在同一组全局配置字段上各留一份记录，而不是只记「能上网」或「不能上网」。可记录的内容来自运行模式、日志级别、出站接口、IPv6、TCP Keep Alive、外部控制 API、局域网绑定和 profile 缓存开关。下面只说明怎样记录：适用条件、睡眠前应写下什么、唤醒后如何对照，以及记录失败时下一步。

## 适用条件与记录原则

适用于电脑端计划进入睡眠或待机，并且唤醒后需要对比连接状态的场景。原则有三条。第一，睡眠前与唤醒后使用同一 `log-level`，否则两段输出不能对照。第二，记录的是配置值加上当时的系统接口和地址，而不是只抄配置文件。第三，控制面与数据面分开记：前者看 `external-controller` 能否访问，后者看 `mode`、`interface-name`、`ipv6` 是否仍成立。

`mode` 为 `rule`、`global` 或 `direct`，默认规则模式。API 监听示例为 `127.0.0.1:9090`，也可监听全部地址；访问时注意 `secret`。Unix socket 与 Windows namedpipe 访问不会验证 `secret`，记录时必须写明通道类型，避免把「不校验的通道可连」写成「密钥条件下 HTTP API 可连」。`log-level` 仅在控制台和控制页面输出；`silent` 不能作为基线。

## 唤醒前应记下的字段和系统状态

在睡眠前按原样记录下列配置：当前 `mode`；是否设置 `interface-name` 及其值；`ipv6` 为 true 还是 false（默认 true）；`keep-alive-interval`、`keep-alive-idle`、`disable-keep-alive`；`tcp-concurrent` 与 `unified-delay`；`allow-lan`、`bind-address`、`lan-allowed-ips`、`lan-disallowed-ips`；是否启用 `authentication` 以及 `skip-auth-prefixes`；`find-process-mode`（`always`/`strict`/`off`）；`profile.store-selected` 与 `store-fake-ip`；`external-controller` 实际监听，必要时加上 unix、pipe 或 TLS 监听。Linux 上同时记下 `routing-mark` 和 `external-controller-routing-mark`。

仅有配置不够。系统侧应附带：出站接口是否存在、IPv4 与 IPv6 地址、默认路由是否就绪。没有这些，唤醒后即使字段不变，也无法解释条件是否变化。再用当时的 `log-level` 留下一段 `info` 或 `debug` 输出作为基线。`error` 表示已无法使用，`warning` 表示出错但不影响运行，基线里如果已经有 warning，唤醒后应单独标注是新增还是重复。

`store-selected` 为 true 时，记录当前策略组选择，以便对照唤醒后 API 读到的选择是否仍在。`store-fake-ip` 为 true 时，记录关键域名的映射关系，以便对照唤醒后是命中旧映射还是重新分配。这两项只说明是否保存供下次启动使用，记录时不要把它们写成「唤醒不会打断连接」的保证。

## 唤醒后对照顺序与失败时下一步

唤醒后立即再采同一组字段，并再取一段同级别日志。对照顺序建议固定为：API 是否仍可达 → `mode` 是否仍为原值 → `interface-name` 是否仍对应存在的网卡 → `ipv6` 与系统地址是否仍对称 → `bind-address` 是否仍能绑到当前地址 → Keep Alive 参数未变但旧连接是否已失效。参数未变不表示会话还在。`tcp-concurrent` 若开启，记录实际成功的 IP，以便发现解析或路径切换。局域网相关项按当前地址是否落在白名单、是否命中黑名单来对照；黑名单优先。

判断依据：任一项在唤醒后读不到，记为采集失败，而不是「该项无变化」。`mode` 改变与接口消失要分成两行。日志按时间顺序保留原文。`unified-delay` 只影响 RTT 口径，对照表里可以记开关，不要用它代替连通性。`find-process-mode` 在唤醒后若进程表重建，应在记录中注明「进程匹配条件可能已变」，再去看规则是否依赖进程。

失败时下一步：若唤醒后 API 失败，记录监听地址、`secret` 是否仍适用、是否误用了不校验 `secret` 的 unix 或 pipe，TLS 监听则核对证书与私钥是否仍可读。若日志为空，检查是否为 `silent`，改为 `info` 或 `debug` 后重新做一轮「前记录—睡眠—后记录」。若接口名变化，以系统当前出站接口更新对照表，再决定是否需要改 `interface-name`。GEO 自动更新间隔、外部用户界面下载地址、全局 UA 与唤醒对比无直接关系，不要填进对照表，以免把一次唤醒问题稀释成整份全局配置抄写。

资料：
https://wiki.metacubex.one/config/general/
