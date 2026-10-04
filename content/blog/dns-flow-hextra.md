---
{
  "title": "Clash 怎样分开记录解析结果与连接结果",
  "description": "Clash 怎样分开记录解析结果与连接结果。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T19:24:25.035175+00:00",
  "lastmod": "2026-10-04T19:24:25.035175+00:00",
  "type": "post"
}
---

## 适用条件：为何要把两类结果分开记

官方 DNS 配置把“查询由谁回答、回答是否被 filter 替换、给客户端 fake-ip 还是 real-ip”写在 `dns` 段，把随后的出站连接写在规则与代理。分开记录的适用条件是：需要判断故障发生在解析阶段还是连接阶段，例如 IP 可连而域名失败、仅节点域名失败、或 fallback 前后结果不一致。不适用于未开启 `enable`（走系统 DNS）或只改规则、不改 DNS 的场景。

记录时应固定一次故障的域名、当时 `dns` 全文、以及该域名命中的策略/过滤项，避免把后续连接失败写进“解析结果”。

## 解析结果应记录的字段与判定

按官方优先级记录解析路径：是否命中 `nameserver-policy`（键为通配或 geosite，值可为多服务器）；未命中则记录实际询问的 `nameserver`；若存在 `fallback`，记录 `fallback-filter` 是否生效。filter 判定包括：geoip 是否开启、geoip-code（默认 CN）是否把 nameserver 的 IP 视为应采用；是否命中 `ipcidr`；是否命中 `domain`（这些域名直接走 fallback、不用 nameserver）。geosite 在 filter 中已废弃，应改记 nameserver-policy。

同时记录 `enhanced-mode`：`redir-host` 下解析结果即真实地址；`fake-ip` 下记录是否落入 `fake-ip-range`，以及 `fake-ip-filter`/`fake-ip-filter-mode` 对该域名是 fake-ip 还是 real-ip。rule 模式下过滤语法与路由类似，需记下最后 MATCH 或具体 RULE-SET/GEOSITE/DOMAIN 条目。`ipv6: false` 记 AAAA 为空。`use-hosts` 与 `use-system-hosts` 为 true 时记是否被 hosts 抢答。

节点域名单独一行：`proxy-server-nameserver` 是否为空（空则回退 nameserver-policy/nameserver/fallback），以及 `proxy-server-nameserver-policy` 是否对该主机名生效。直连出口单独一行：`direct-nameserver` 与 `direct-nameserver-follow-policy`。`respect-rules` 为 true 时必须记下 DNS 查询是否按规则出站、以及是否已配 proxy-server-nameserver。附加参数（`#proxy`、ecs、disable-ipv4/ipv6、disable-qtype）记是否丢弃了某类记录。`default-nameserver` 只记录“DNS 服务器域名能否被解析”，不要当成业务域名结果。

## 连接结果如何单独写、失败时下一步

连接结果另起记录：在已得到地址（真实 IP 或 fake-ip 映射）之后，流量是否按规则选择 direct/代理并完成传输。不要把 fallback 换答案写成“连接失败”，也不要把 fake-ip 下发写成“网站已通”。判断依据仍是配置语义：解析记录回答“谁给了什么地址、是否被 filter/hosts/空 AAAA 改变”；连接记录回答“该地址之后的出站是否建立”。

若只有解析记录没有连接记录，无法区分污染过滤与出站故障。下一步：用同一域名对比“策略命中的服务器列表”与“filter 是否改用 fallback”；节点名只对比 proxy-server 两项；直连只对比 direct-nameserver。仍分不清时，将域名写入 nameserver-policy 做单变量，或临时 `enable: false` 对比系统 DNS。`listen` 仅表示本机 DNS 服务监听，用于确认查询是否进入核心，不能代替连接结果。修改配置前把上述两段记录与原 yaml 一并保存。

https://wiki.metacubex.one/config/dns/
