---
{
  "title": "Clash 怎样验证同域名解析与连接两个阶段",
  "description": "Clash 怎样验证同域名解析与连接两个阶段。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.554057+00:00",
  "lastmod": "2026-10-04T20:32:58.554057+00:00",
  "type": "post"
}
---

同一域名在 Clash 中会经过解析与连接两个阶段。解析阶段由 DNS 配置决定问哪台上游、是否先命中 hosts、返回真实地址还是 fake-ip；连接阶段由路由规则决定应用访问目标的出站，而 Clash 访问 DNS 上游的连接在 `respect-rules` 开启时也会进入路由。验证时必须把“查到了什么、问的是谁”和“这条连接从哪出去”分开对照官方字段，否则会把上游选错、污染过滤、节点域名循环和分流失配混成一个问题。文档没有把验证绑定到某一操作界面，因此以配置语义逐项核对。

## 适用条件

适用于已经启用 `dns.enable`，需要确认某一域名的查询是否命中 `nameserver-policy`、是否落入 `nameserver` 或 `fallback`，以及随后连接是否按预期出站。`enhanced-mode` 可选 fake-ip 或 redir-host，默认 redir-host。fake-ip 下还会使用 `fake-ip-range`、`fake-ip-filter` 等，这些只改变返回给客户端的映射及是否用于连接，不改变“应使用哪台上游”的策略语义。若 `enable` 为 false，解析交给系统 DNS，无法按 Clash 字段验证上游选择。具体场景例如：域名已写入策略，但既怀疑问错上游，又怀疑随后没有走到对应出站。

## 解析阶段怎样核对

第一步，确认该查询由 Clash DNS 处理，而不是系统解析或应用内置 DNS。第二步，对照域名是否命中 `nameserver-policy`。该项指定域名查询的解析服务器，可使用 geosite，优先于 `nameserver` 与 `fallback`；键支持域名通配，值支持字符串或数组。文档示例中的 `+.arpa`、`rule-set:cn` 代表通配与集合两类键。第三步，未命中则看 `nameserver`；若配置了 `fallback`，再按 `fallback-filter` 判断：`geoip` 为 true 时，非 `geoip-code` 国家的结果视为污染并采用 fallback 结果；`domain` 名单内域名直接使用 `fallback`、不去使用 `nameserver`；`ipcidr` 命中的结果视为污染。其中 `geosite` 字段已废弃，应改用 `nameserver-policy`。第四步，若该字符串同时是代理节点域名，核对 `proxy-server-nameserver`；不填则遵循 `nameserver-policy`、`nameserver` 和 `fallback`。`proxy-server-nameserver-policy` 仅当前者非空时生效。第五步，若随后连接将走 `direct`，核对 `direct-nameserver` 与 `direct-nameserver-follow-policy`（默认不遵守策略，且仅当 `direct-nameserver` 非空时生效）。第六步，核对容易误判的旁路：`use-hosts`、`use-system-hosts` 会先按 hosts 回应；`default-nameserver` 只解析 DNS 服务器域名且必须为 IP；`ipv6` 为 false 时回应 AAAA 空解析。这些都不等于策略键选错，但会让解析阶段的观察结果与预期不一致。

## 连接阶段核对、判断依据与失败下一步

连接阶段要分清两类连接。第一类是应用访问该域名目标的连接，出站由路由规则决定，DNS policy 不代替分流。第二类是 Clash 访问 DNS 上游的连接：`respect-rules` 为 true 时遵守路由规则，且需配置 `proxy-server-nameserver`；附加参数 `#RULES` 语义相同。经代理查询应配置 `proxy-server-nameserver`，避免鸡蛋问题。文档强烈不建议 `respect-rules` 与 `prefer-h3` 一起使用。`#proxy` 优先使用已有代理，名称不存在则走指定接口。`fake-ip-filter` 只决定哪些地址不发 fakeip 映射用于连接，不能证明解析上游正确。

判断依据：解析阶段看该域名命中了哪一类 DNS 字段；连接阶段看应用连接的路由，以及 DNS 查询连接是否被 `respect-rules` 或附加参数带入代理。两阶段都通过，才能认为“同域名”配置闭环；只看到 fake-ip 或真实 IP，不足以得出上游或出站正确的结论。

失败时下一步：解析来源不对，检查 `enable`、策略键、废弃的 `fallback-filter.geosite`、direct 与节点专用 DNS、hosts。DNS 上游连不上，补 `proxy-server-nameserver` 并检查 `respect-rules`。应用连接出站不对，去查路由规则，而不是继续增加 nameserver。需要改变某条 DoH 的传输或 ECS 行为时，使用文档所列附加参数，而不是用分流规则去模拟上游选择。

资料：https://wiki.metacubex.one/config/dns/
