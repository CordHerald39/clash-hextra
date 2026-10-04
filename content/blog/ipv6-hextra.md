---
{
  "title": "Clash 怎样比较同一目标的 IPv4 和 IPv6 表现",
  "description": "Clash 怎样比较同一目标的 IPv4 和 IPv6 表现。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.557956+00:00",
  "lastmod": "2026-10-04T20:32:58.557956+00:00",
  "type": "post"
}
---

## 适用条件

比较同一目标的 IPv4 与 IPv6，指同一域名的 A 与 AAAA、以及 fake-ip 两族映射是否被同等处理，而不是比快慢。适用条件：enable 为 true，能阅读 ipv6、fake-ip-range、fake-ip-range6 和 nameserver 附加参数。官方对称开关是 disable-ipv4 丢弃 A、disable-ipv6 丢弃 AAAA；全局 ipv6 为 false 时 AAAA 为空。比较时保持服务器集合与策略不变，只观察地址族相关项。

## 对照步骤与判断依据

第一步固定同一域名，看它是否命中 nameserver-policy、fake-ip-filter 或 fallback-filter 的 domain。判断依据：policy 优先于 nameserver 与 fallback，两族若落到不同服务器则不可比。节点域名看 proxy-server-nameserver 与 proxy-server-nameserver-policy；直连看 direct-nameserver。default-nameserver 必须为 IP，只解析 DNS 服务器域名，不是业务目标。

第二步对照开关。ipv6 为 false 时只有空 AAAA，两族并不同等。再看该服务器是否带 disable-ipv4 或 disable-ipv6。判断依据：二者分别丢弃 A 与 AAAA。若只观察一族，保存基线后只动其中一个。disable-qtype- 会丢弃特定类型，文档举例可屏蔽 HTTPS（TYPE65），比较 A 与 AAAA 时要避免误伤。附加参数用 # 附加、& 连接，写错会让某一族被丢弃。

第三步对照 fake-ip。enhanced-mode 为 fake-ip 时，IPv4 用 fake-ip-range，IPv6 用 fake-ip-range6。判断依据：只配一段则同一目标只在一个地址族得到伪造地址。blacklist 匹配则不下发 fake-ip；whitelist 仅匹配成功才返回；rule 模式给出 fake-ip 或 real-ip。redir-host 比较的是真实 A 与 AAAA 是否都返回。非必要不要改 fake-ip-ttl。

第四步对照 fallback-filter。配置 fallback 后默认启用，geoip-code 默认 CN；ipcidr 中网段视为污染。判断依据：A 被采用而 AAAA 被判污染（或相反）时，两族路径不同。respect-rules 开启需有 proxy-server-nameserver，且强烈不建议与 prefer-h3 同用，否则出站不同会使比较无效。hosts 两项若只写了一个地址族，比较前要记下。

结论只分三类：两族都有、只有 A、只有 AAAA。只有 A 时先核 ipv6 与 disable-ipv6；只有 AAAA 时核 disable-ipv4；两族都有仍不对称时再看两段网段和过滤。不要把 lru、arc 或 prefer-h3 说成地址族优劣。

## 比较无法完成时的下一步

全局 ipv6 为 false 时先记录该值，此时没有对照意义。没有 fake-ip-range6 应视为未配置 IPv6 伪造段，不能推断上游无 AAAA。policy 导致服务器不一致时，先记下差异。geosite 已废弃，勿用旧字段解释。仍无法比较时，按官方 DNS 页核对附加参数，避免一次改多个无关键。

资料：https://wiki.metacubex.one/config/dns/
