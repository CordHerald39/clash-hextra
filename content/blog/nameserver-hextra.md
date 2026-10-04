---
{
  "title": "Clash 怎样固定域名对比不同上游结果",
  "description": "Clash 怎样固定域名对比不同上游结果。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.523922+00:00",
  "lastmod": "2026-10-04T20:32:58.523922+00:00",
  "type": "post"
}
---

要在 Clash（mihomo）中固定某一域名并对比不同上游的结果，应使用官方的按域名指定解析服务器机制，而不是只改全局 `nameserver` 并假设该域名一定走第一项。对应字段是 `nameserver-policy`：指定域名查询的解析服务器，可使用 geosite，优先于 `nameserver`/`fallback` 查询。

## 适用条件

`dns.enable` 须为 `true`，否则使用系统 DNS 解析，无法固定到配置中的上游。查询须进入 `listen` 的 DNS 服务，支持 udp、tcp。

对比的是上游返回的记录，因此要排除抢答与过滤。`use-hosts`、`use-system-hosts` 默认 `true`，命中 hosts 时比较的不是上游。`ipv6` 为 `false` 会回应 AAAA 空解析，不能用来比较 IPv6。`enhanced-mode` 为 fake-ip 时，`fake-ip-filter` 匹配地址不会下发 fakeip 映射；`fake-ip-filter-mode` 为 blacklist、whitelist 或 rule 时逻辑不同，rule 模式下语法与路由 rules 一致。`cache-algorithm` 为 lru 或 arc，缓存可能让相邻两次查询看到同一应答，对比时要意识到缓存存在。`fake-ip-ttl` 非必要情况下请勿修改。

节点域名应改用 `proxy-server-nameserver-policy`，当且仅当 `proxy-server-nameserver` 不为空时生效，不能用业务政策去对比节点名。direct 出口受 `direct-nameserver` 与 `direct-nameserver-follow-policy` 约束，后者默认不遵守 `nameserver-policy`。

## 固定域名到指定上游的步骤

场景：需要把同一测试域名先后指向两台上游，或用两个仅用于对比的域名分别指向两台上游，观察记录差异。

第一步，在 `nameserver-policy` 中用域名通配作为键，值写上游地址，值支持字符串或数组。要固定走上游甲，键匹配该域名且值只有甲。对比上游乙时，只改该值，或另设仅用于对比的另一域名键指向乙。不要把甲乙都放进全局 `nameserver` 且不做政策，官方并未保证此时该域名固定询问哪一台。

第二步，避免 `fallback-filter` 改写最终采用的结果。配置 `fallback` 后默认启用 filter，`geoip-code` 为 cn。`geoip` 判断下，该国结果直接采用，其他视为污染并采用 fallback；`domain` 匹配则直接使用 fallback、不去使用 nameserver；`ipcidr` 污染同理。`geosite` 已废弃，请使用 `nameserver-policy`。目标若是对比单一上游，应让该域名不要落入这些强制路径。`fallback-lazy-query` 默认 `false`，为 `true` 会先判断 nameserver 结果是否满足 filter 后再发起查询，对比时序会变。

第三步，保持附加参数一致再比记录。公网 DNS 用 `#` 附加、`&` 连接。`ecs` 与 `ecs-override` 会改变 subnet 视图；`disable-ipv4`、`disable-ipv6`、`disable-qtype-<int>` 会丢弃记录，例如 `disable-qtype-65` 屏蔽 HTTPS（TYPE65）。指定代理或 `#RULES` 只改变 DNS 连接路径。`h3` 强制 HTTP/3，与 `prefer-h3` 不冲突，但服务器不支持时该上游会失败，对比会失真。`respect-rules` 需配置 `proxy-server-nameserver`，以防鸡蛋问题。

第四步，确认 `default-nameserver` 为 IP，否则连上游服务器域名都解析不了，对比无法开始。

## 判断依据与失败时下一步

判断已经固定的依据：该域名命中 `nameserver-policy` 且该字段优先；未走 fallback 的 domain 等强制路径；不是节点专用或 direct 在不遵循政策时的专用列表；hosts 未抢答；`enable` 与 `listen` 有效。对比时应每次只改变政策的值或使用两个测试域名，避免同时改全局列表与 filter。

若结果对不上，下一步检查政策键通配是否其实没匹配；检查 `direct-nameserver-follow-policy` 是否让 direct 流量忽略政策；检查证书相关 `skip-cert-verify`、`name-cert-verify` 是否导致某一侧加密 DNS 失败。不要用节点列表充当上游来做对比。字段以官方 DNS 配置为准。

参考资料：https://wiki.metacubex.one/config/dns/
