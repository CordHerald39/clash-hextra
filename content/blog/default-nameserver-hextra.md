---
{
  "title": "Clash 怎样追踪上游服务器域名是否可解析",
  "description": "Clash 怎样追踪上游服务器域名是否可解析。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.550830+00:00",
  "lastmod": "2026-10-04T20:32:58.550830+00:00",
  "type": "post"
}
---

追踪上游服务器域名是否可解析，并不是另做一套探测协议，而是顺着官方字段把依赖链核对清楚：该主机名要被谁解开、解开时是否还依赖代理、失败时卡在引导还是卡在建连策略。适用条件是 `dns.enable` 为 true，且上游以域名形式出现在 `nameserver`、`fallback`、policy 或需要主机名才能建连的配置中。`enable` 为 false 时走系统 DNS，内置引导链不作为判断依据。追踪时只依据字段约束，不引入手册未给出的操作界面或效果承诺。

## 用引导列表追踪主机名能否第一次被解开

手册写明 `default-nameserver` 用于解析 DNS 服务器的域名，必须为 IP，可为加密 DNS。追踪第一步：列出所有上游主机名，再看引导列表是否全是 IP，例如 `223.5.5.5`。判断依据：引导条目若仍是域名，则「上游是否可解析」在内置逻辑上无法自洽。加密 DNS 可作为引导，但表达必须是 IP。

针对「配置里写了上游域名，不确定它能不能被解开」的场景，步骤如下。枚举 `nameserver`、`fallback`、`nameserver-policy`、`direct-nameserver` 值中的主机名；核对这些名字是否都依赖 `default-nameserver`；把非 IP 从引导中移走；再确认引导列表非空。`ipv6` 为 false 只让 AAAA 成为空解析，不回答上游主机名有没有 IPv4。`listen` 只说明 DNS 服务监听，不参与这次追踪。

## 用节点解析字段追踪经代理时的第二跳

仅知道引导 IP 不够。若 DNS 连接遵守路由或需要经代理，手册要求配置 `proxy-server-nameserver`，以防鸡蛋问题。`respect-rules` 同样依赖该字段，且强烈不建议与 `prefer-h3` 共用。附加参数指定代理时，优先已有代理，不存在则指定接口；`#RULES` 表示遵守路由规则。`proxy-server-nameserver-policy` 仅当节点解析列表非空时生效。

追踪步骤：判断该上游连接是否会被路由到代理；若会，检查节点解析服务器是否存在、是否自身仍是无法引导的域名；再检查 policy 是否因列表为空而无效。判断依据是：节点域名解析失败时，表现为上游域名写在配置中但建连条件不成立，而不是 `fallback-filter` 的污染判断失败。`direct-nameserver` 的回退规则与节点解析相互独立，追踪时应分开记录，避免把直连出口的 DNS 列表误当成节点域名的可解析证明。

## 把结果筛选排除出「能否解析上游」

`fallback-filter` 的 geoip、ipcidr、domain 用于决定最终目标结果是否采用 fallback；匹配 domain 的访问目标会直接走 fallback。`geosite` 已废弃，应改用 `nameserver-policy`。它们追踪的是目标域名结果是否可信，不是上游主机名有没有 IP。`cache-algorithm`、`enhanced-mode`、`fake-ip-filter`、`fake-ip-filter-mode` 同样不回答上游域名是否可解析。`use-hosts` 与 `use-system-hosts` 只影响 hosts 是否参与回应。

若引导为 IP、节点解析已独立，仍认为上游不可解析，下一步检查 policy 是否把该主机名指向另一组循环依赖的服务器，或 `direct-nameserver` 为空时的回退是否指向不可用列表；同时核对 `direct-nameserver-follow-policy` 的默认不遵守行为，以免把政策未生效当成上游不可解析。每次只验证链上的一个依赖：先引导，再节点解析，最后才是目标查询。字段约束满足后，上游域名才具备被解开的前提。

https://wiki.metacubex.one/config/dns/
