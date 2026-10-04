---
{
  "title": "Clash 电脑端：怎样在保留设置的前提下做扩展对照测试",
  "description": "Clash 电脑端：怎样在保留设置的前提下做扩展对照测试。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T21:37:45.962530+00:00",
  "lastmod": "2026-10-04T21:37:45.962530+00:00",
  "type": "post"
}
---

对照测试的目的是：在不丢掉已保存的策略组选择和其他全局项的前提下，判断浏览器扩展是否改变了入站路径。手册提供的可保留项主要是 `profile` 中的储存开关；可临时改、事后改回的是 `mode`、`log-level` 和 `find-process-mode`。适用条件是电脑端已有一份稳定的全局配置，只想比较启用扩展与不启用扩展，而不是重写整份配置。不要在对照中改手册未记载的客户端入口名称，也不要把 `secret` 改成空字符串，除非你本来就没有访问密钥。

## 先锁住会在重启后还原的选择

`profile.store-selected` 为 true 时，会储存 API 对策略组的选择，供下次启动使用。对照测试前确认该项为 true，这样即使你通过 API 临时切换 GLOBAL 或其他策略组，重启仍可回到已储存选择。`store-fake-ip` 为 true 时储存 fakeip 映射表，域名再次连接时使用原有映射地址；对照扩展时不要关闭它，以免把解析差异算进扩展头上。判断依据：测试前后策略组选择一致，且不是一次启动就丢失。

不要为了测试去改 `external-controller` 监听地址或 Unix socket、Windows namedpipe。文档写明从后两种通道访问 API 不会验证 secret。对照扩展与入站无关的 API 面应保持原样，以免把安全边界变化当成浏览器差异。

## 每次只改一类可逆变量

第一组对照：保持内核配置不动，只启用或停用浏览器扩展，观察流量是否进入 http(s)/socks/mixed。此时 `authentication` 与 `skip-auth-prefixes` 应保持原值，以便判断扩展是否仍能通过验证。文档中跳过验证示例如 `127.0.0.1/8`、`::1/128`。判断依据：扩展开关变化时，控制台或控制页面的入站记录出现或消失。

第二组对照：扩展状态固定，只改 `mode`。在 `rule`、`global`、`direct` 之间切换。`global` 需要在 GLOBAL 策略组选择代理或策略；因已打开 `store-selected`，该选择可在测试后恢复。判断依据：扩展若已进核，浏览器行为应随 `mode` 变化；若 `mode` 改变而该浏览器完全不动，说明扩展可能连到了非 Clash 地址。

第三组对照：仅调整 `find-process-mode`（`always`、`strict`、`off`）观察该浏览器进程是否被匹配。手册建议路由器使用 `off`，桌面对照结束后应改回你原来的值，默认一般是 `strict`。不要同时改 `allow-lan` 与绑定地址，除非扩展明确使用了局域网 IP。

## 用日志收束并处理失败

将 `log-level` 从日常的 `info` 临时改为 `debug` 以便看到更多运行信息，测试结束改回。`silent` 无法用于对照。`ipv6` 保持原值，除非扩展只在某一协议上失败。`unified-delay` 与 `tcp-concurrent` 影响延迟计算和并发建连，与扩展入口无关，对照期间不要动，以免引入节点侧差异。

失败时下一步：若停用扩展后仍无法回到原行为，检查是否把 `mode` 留在了 `direct` 或 `global`；若策略组选择丢失，检查 `store-selected` 是否被改成 false。若只有扩展开启时出现认证失败，恢复 `authentication` 与 `skip-auth-prefixes`，不要为了测试清空验证。全部对照结束后，确认 `log-level`、`find-process-mode`、`mode` 已回到测试前取值。字段定义以全局配置文档为准。

资料来源：https://wiki.metacubex.one/config/general/
