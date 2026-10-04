---
{
  "title": "Clash 怎样记录 DoH 服务地址和错误类型",
  "description": "Clash 怎样记录 DoH 服务地址和错误类型。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.550830+00:00",
  "lastmod": "2026-10-04T20:32:58.550830+00:00",
  "type": "post"
}
---

手册没有单独规定“DoH 日志面板”，记录服务地址和错误类型，指在变更说明里按字段抄写 HTTPS 上游及其附加参数，并用文档里的机制给失败分类。这样下次修改时能回到同一条依据，而不是凭记忆合并所有上游。

## 按字段分开抄写 DoH 地址

适用条件：配置中有多处 HTTPS DNS。应分别登记：`nameserver`（默认域名解析服务器）、`fallback`（后备）、`nameserver-policy`（键加值，值可能是数组）、`proxy-server-nameserver`（仅节点域名）、`proxy-server-nameserver-policy`（仅当上一字段非空时生效）、`direct-nameserver` 与 `direct-nameserver-follow-policy`、以及 `default-nameserver`（必须为 IP，可为加密 DNS）。判断依据：字段职责不同，混成一行会无法判断“哪一类查询在用这条 DoH”。

对每一条地址再抄：`#` 后的代理或接口名、是否为 `#RULES`、是否 `h3`、是否 `skip-cert-verify`、是否 `name-cert-verify`、ecs 与 `ecs-override`、以及 disable 类参数。全局还要抄 `enable`、`prefer-h3`、`respect-rules`、`ipv6`、`enhanced-mode`。步骤：一张表对应一个字段，数组拆成多行；代理名与 `proxies` 中的 `name` 对照。失败时下一步：若登记时发现 `respect-rules` 为 true 却没有 `proxy-server-nameserver`，先把这个缺口写成待修项，不要只存 URL。

## 按手册机制给错误类型命名

适用条件：更换或填写 DoH 后查询不符合预期。建议只用文档已出现的机制命名错误类型，避免自造日志短语。可区分：未启用（`enable` 为 `false`，使用系统 DNS）；引导解析类型（`default-nameserver` 不是 IP，或无法解析 DoH 主机名）；鸡蛋问题类型（经代理查询或 `respect-rules` 却未配置 `proxy-server-nameserver`）；HTTP/3 类型（`prefer-h3` 或 `h3` 而服务器不支持 HTTP/3）；证书类型（`skip-cert-verify`、`name-cert-verify` 只改 DNSName 不改 SNI）；应答丢弃类型（`ipv6` 为 false 导致 AAAA 空解析，或 disable-ipv4/ipv6/qtype）；分流类型（`nameserver-policy` 优先命中旧值）；过滤类型（`fallback-filter` 的 geoip、ipcidr、domain；`geosite` 已废弃）；假 IP 类型（`fake-ip-filter` 与 mode）。判断依据：每类都对应可回看的字段，而不是主观“网络不好”。

`cache-algorithm` 只说明 lru 或 arc，不要把缓存算法登记成 DoH 错误。`listen` 异常属于本机服务监听，与上游 DoH 地址分列。步骤：一次失败只标一个主类型，并附上当时的字段摘录。失败时下一步：主类型是引导或鸡蛋问题，禁止先改证书项；主类型是过滤，去改 policy 或 fallback-filter，而不是更换同一条 DoH 的路径。

## 用记录决定修改顺序

适用条件：已有地址清单和错误类型。修改顺序应与依赖一致：先 `enable`，再 `default-nameserver` 的 IP，再 `proxy-server-nameserver`（当查询走代理或遵守路由时），再决定是否同时使用 `prefer-h3`（文档强烈不建议其与 `respect-rules` 一起使用），最后才改具体 DoH URL 和 `#` 参数。判断依据：前序依赖不成立时，后序地址抄得再完整也不会进入手册描述的 DOH 建连。

登记变更时写明：旧 URL、新 URL、保留了哪些 bool 参数、是否仍强制 HTTP/3、证书策略是否变化。`fallback` 若存在，注明默认会启用 `fallback-filter`，且 `geoip-code` 默认与污染判断有关。步骤：每次只改一类错误对应的字段，把结果写回同一张表。失败时下一步：若类型未变，说明记录里的主因没改到，回到该类型的字段，而不是追加更多 HTTPS 条目造成来源混乱。完整字段含义以手册为准。

资料：https://wiki.metacubex.one/config/dns/
