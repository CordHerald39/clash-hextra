---
{
  "title": "Clash 怎样核对主机名与静态 IP 的对应关系",
  "description": "Clash 怎样核对主机名与静态 IP 的对应关系。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.554057+00:00",
  "lastmod": "2026-10-04T20:32:58.554057+00:00",
  "type": "post"
}
---

核对主机名与静态 IP 的对应关系，是要确认：查询该名字时，Clash DNS 是否按配置中的 hosts 或系统 hosts 给出预期地址。官方提供的核对入口包括：`use-hosts` 是否回应配置中的 hosts，`use-system-hosts` 是否查询系统 hosts，以及 `enable`、`listen`、`ipv6`、`enhanced-mode` 会不会改变实际下发的地址。核对的是当前配置与当前查询路径，而不是把其他解析器的结果拿来当证据。

## 何时需要核对对应关系

已经写入映射、验收变更、或怀疑记录与访问目标不一致时使用。判断依据：手里有主机名列表和预期 IP 列表；可以阅读当前 DNS 配置；能够发起会到达 Clash 的 DNS 查询。

`enable` 为 false 时，文档说明使用系统 DNS 解析，应先核对该前提，再谈 Clash 内的对应关系。映射在配置中但 `use-hosts` 为 false，或映射在系统文件但 `use-system-hosts` 为 false，则对应关系在 DNS 模块侧视为不存在。代理节点名应单独用 `proxy-server-nameserver` 路径核对，不要和业务主机名放在同一张对照表。

## 对照表与查询路径

先列出对照表：左列主机名，右列预期 IPv4 或 IPv6。只核对待查名字。`nameserver-policy` 的通配键用于指定解析服务器，不是静态 IP，不能写进这张表当作预期地址。

再核对开关与来源。配置中的 hosts 应对 `use-hosts` 为 true；系统 hosts 应对 `use-system-hosts` 为 true。默认值均为 true，但以实际配置为准。同时确认 `enable` 为 true。

查询必须发往文档中的 `listen`。该字段为 DNS 服务监听，支持 udp、tcp。发到其他解析器的结果，不能用来证明 Clash 的对应关系。`default-nameserver` 必须为 IP，只用于解析 DNS 服务器的域名，不要把它的应答误认为业务主机名的 hosts 记录。

地址类型要分开看。`ipv6` 为 false 时回应 AAAA 的空解析。对照表里有 IPv6 而该开关为 false 时，AAAA 表现为空，这不是 hosts 写错，而是 IPv6 解析被关闭。A 与 AAAA 必须分开核对。

还要声明你在核对哪一层结果。`enhanced-mode` 为 fake-ip 时，连接侧可能看到 `fake-ip-range`（文档示例 `198.18.0.1/16`）中的地址。`fake-ip-filter-mode` 为 rule 时，可对域名给出 fake-ip 或 real-ip。DNS 回应的静态 IP 与连接实际使用的 fake-ip 不一致，并不自动等于 hosts 写错。

若模块并未回应 hosts，查询结果可能来自 `nameserver-policy`（优先于 nameserver/fallback）或 `fallback-filter` 的 domain 匹配（匹配后只使用 fallback 解析）。用这类上游结果去对比 hosts 表会造成误判。判断依据：只有确认模块会回应 hosts 时，查询结果才应等于对照表。

direct 出口域名应核 `direct-nameserver` 与 `direct-nameserver-follow-policy`。节点名核 `proxy-server-nameserver-policy`，且该 policy 当且仅当 `proxy-server-nameserver` 不为空时生效。

## 结果不一致时的判断顺序

查询结果与对照表不一致时，按此顺序缩小范围：开关是否关闭；查询是否未达 `listen`；是否在看 AAAA 空回应；是否在看 fake-ip；是否命中 policy 或 fallback-filter；是否把节点解析和业务解析混用。

`cache-algorithm` 为 lru（默认）或 arc 时，旧对应关系可能仍被缓存。下一步是在缓存可能命中的前提下再次查询同一主机名，避免把过期缓存当成当前 hosts。

`respect-rules` 开启时 dns 连接遵守路由规则，需要 `proxy-server-nameserver`。核对过程中查询失败或得到意外地址，应先检查 DNS 查询自身是否按路由发出，而不是改对照表凑结果。官方强烈不建议 `respect-rules` 与 `prefer-h3` 一起使用，核对环境应避开这种组合，以免把连接方式问题当成映射错误。

https://wiki.metacubex.one/config/dns/
