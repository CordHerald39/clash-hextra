---
{
  "title": "Clash 怎样记录应用专用代理与系统代理设置",
  "description": "Clash 怎样记录应用专用代理与系统代理设置。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T21:03:35.935143+00:00",
  "lastmod": "2026-10-04T21:03:35.935143+00:00",
  "type": "post"
}
---

这里的“记录”指把可复查的配置键、日志级别和缓存策略固定下来，用来区分两类事实：某一应用自己填写的代理主机、端口和账号，以及本机或其他设备经同一内核端口进入后的处理方式。适用条件是使用全局配置中的入站、鉴权、运行模式、进程匹配和 `profile` 缓存，而不是依赖未在文档出现的界面名称。记录时应同时写下应用侧参数和内核侧对应项，避免只保存其中一侧，导致重启或换设备后无法对照。

## 把入站可达性和鉴权写成可核对清单

需要记录的内核项包括：`allow-lan` 是否允许其他设备使用代理端口，`bind-address` 绑定 `*` 还是单个 IPv4 或 IPv6，`lan-allowed-ips` 与 `lan-disallowed-ips`（禁止段优先级高于允许段），以及 `authentication`、`skip-auth-prefixes`。判断依据：应用专用代理通常表现为该应用连接特定地址并可能携带用户名密码；其他应用若只是使用同一端口，则差异会出现在来源 IP、是否命中跳过验证前缀、是否被禁止段拒绝。步骤：把上述键值写入变更说明；注明每个应用连接的绑定地址；标明哪些来源依赖账号、哪些依赖跳过前缀。失败时下一步：若按记录仍无法复现差异，检查是否只记下了账号而漏了来源 IP 段，或漏了黑名单。

## 同步保存模式、进程匹配和出站相关键

`mode` 应记录为 `rule`、`global` 或 `direct`；若为 `global`，还要记录 `GLOBAL` 策略组所选代理或策略。`find-process-mode` 记录 `always`、`strict` 或 `off`，这决定内核会不会匹配进程，从而决定“按应用区分”在内核里有没有依据。`ipv6`、`interface-name`、`routing-mark`、`tcp-concurrent`，以及 `keep-alive-interval`、`keep-alive-idle`、`disable-keep-alive`，影响入站后的传输与出站路径，应与模式一并保存。判断依据：若所谓应用专用处理依赖进程匹配，则记成 `off` 会使这份记录失去进程侧意义；路由器场景按文档推荐记录为 `off`。步骤：以一份配置快照同时保存这些键，不要只记住模式名称。失败时下一步：对照快照查看是否有人把 `mode` 改成 `direct`，或把进程匹配关掉，再决定是否继续查日志。

## 用日志级别和 profile 留下重启后仍能解释的状态

`log-level` 可选 `silent`、`error`、`warning`、`info`、`debug`，仅在控制台和控制页面输出。排查期应记下当时级别；需要一般运行信息时用 `info`，需要尽量完整的运行信息时用 `debug`，结束后改回，避免长期停留在 `debug`。`profile` 中 `store-selected` 为 true 时储存 API 对策略组的选择，供下次启动使用；`store-fake-ip` 为 true 时储存 fakeip 映射表，域名再次发生连接时使用原有映射地址。判断依据：策略组选择和映射表属于可被内核记住的状态，不写入记录就无法解释某一应用在重启后的表现变化。外部控制器的监听地址和 `secret` 决定如何用 API 读取这些状态；从 Unix socket 或 Windows namedpipe 访问 API 不会验证 secret，记录时必须注明是否启用。步骤：保存日志级别、`store-selected` 与 `store-fake-ip`、API 监听地址和是否设置 secret，变更前后各留一份。失败时下一步：若重启后策略组选择丢失，检查 `store-selected` 是否为 false；若域名映射变化，检查 `store-fake-ip`；不要把 API 只读查看与应用入站端口混为一谈。

全局配置说明：https://wiki.metacubex.one/config/general/
