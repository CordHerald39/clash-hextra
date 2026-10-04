---
{
  "title": "Clash 怎样核对配置采用的是哪种 DNS 协议",
  "description": "Clash 怎样核对配置采用的是哪种 DNS 协议。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.552925+00:00",
  "lastmod": "2026-10-04T20:32:58.552925+00:00",
  "type": "post"
}
---

## 适用条件

核对 Clash 当前采用哪种 DNS 协议，对象是配置文件中所有会发出查询的服务器字段，而不是本机 `listen`。`listen` 支持 udp、tcp，描述的是 DNS 服务监听，不能用来推断上游是 DoH 还是 DoT。`enable` 为 false 时使用系统 DNS，结论应写成“未启用配置内上游”，不要再按 `nameserver` 的 URI 下判断。

需要逐项打开的字段包括：`nameserver`、`fallback`、`nameserver-policy`、`default-nameserver`、`proxy-server-nameserver`、`proxy-server-nameserver-policy`、`direct-nameserver`。`nameserver-policy` 优先于 `nameserver`/`fallback`；`proxy-server-nameserver-policy` 仅当 `proxy-server-nameserver` 非空时生效；`direct-nameserver` 为空则遵循前几项，`direct-nameserver-follow-policy` 默认不遵守 policy。漏读策略键时，核对结论会只覆盖默认列表。

## 按 URI 与开关核对协议

第一，看方案前缀。手册示例里，HTTPS 查询路径表示 DoH；`tls://` 表示 DoT；`direct-nameserver` 可以写 `system`，表示系统解析。只出现 IP、且没有上述加密方案前缀时，不能把它标成文档中的 DoH 或 DoT 写法。`default-nameserver` 必须为 IP，但文档允许其为加密 DNS，因此“是 IP”不等于“一定是明文”，还要看该条目是否仍带加密方案。

第二，看 DoH 专有传输开关。`prefer-h3` 为 true 表示 DoH 优先 HTTP/3；条目上的 `#h3` 强制 HTTP/3 建立 DoH 连接。这两项只能作为“DoH 还走了 HTTP/3”的附加结论，不能单独证明存在 DoT。

第三，看列表职责是否改变协议含义。协议由该条 URI 决定，不因放进 `fallback` 就变成另一种封装。`fallback` 配置后默认启用 `fallback-filter`，只影响何时采用后备结果。`nameserver-policy` 的键支持域名通配，值支持字符串或数组，数组里也可以出现不同方案，需要逐条核对。

第四，看附加参数是否干扰判断。`#` 后的代理名、接口、`RULES`、`skip-cert-verify`、`name-cert-verify`、`ecs` 等不改变方案前缀。证书参数出现在 DoH 或 DoT 上都可能，不能凭 `skip-cert-verify` 判定协议种类。

## 判断依据与核对失败时下一步

结论应分字段给出，例如：默认 `nameserver` 为 DoH；`fallback` 为 DoT；某策略命中为指定服务器；节点域名走 `proxy-server-nameserver`；直连走 `direct-nameserver` 或回退到前几项。全局 `prefer-h3` 或条目 `h3` 只附加在 DoH 上。

若同一域名在 policy 与默认列表方案不一致，以 policy 为准。若 `enable` 为 false，下一步是先打开 DNS 配置再核对。若只有主机名、没有方案，无法判断为手册中的 DoH 或 DoT，应补全 HTTPS 方案或 `tls://` 后再核。主机名无法解析时检查 `default-nameserver` 是否为 IP。经代理查询缺少 `proxy-server-nameserver` 时，节点域名所用协议可能并不是你在 `nameserver` 里看到的那一条。`geosite` 在 fallback-filter 中已废弃，核对协议时不要把废弃字段当成服务器方案。`ipv6`、`disable-qtype-<int>` 等只影响回应内容，不改变协议种类。

参考资料：https://wiki.metacubex.one/config/dns/
