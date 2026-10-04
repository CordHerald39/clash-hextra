---
{
  "title": "Clash 怎样检查每个 DNS 上游是否又指回入口",
  "description": "Clash 怎样检查每个 DNS 上游是否又指回入口。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.557267+00:00",
  "lastmod": "2026-10-04T20:32:58.557267+00:00",
  "type": "post"
}
---

入口在手册里对应 `listen` 所宣布的 DNS 服务监听，支持 udp、tcp。检查每个 DNS 上游是否又指回入口，就是核对所有可能发出查询的服务器字段，是否把目标写成该监听，或写成必须再经该监听才能解析的名字。检查通过之前，不能把转发当成已闭合。

## 适用条件

在 `enable` 为 true 且已设置 listen 时进行。enable 为 false 时使用系统 DNS，不按这套上游清单检查。清单包括 default-nameserver、nameserver、fallback、nameserver-policy 的值、proxy-server-nameserver、proxy-server-nameserver-policy、direct-nameserver，以及用 `#` 指定的代理或接口、`#RULES`。`fake-ip-range` 是 fakeip 地址段，不是上游，不要拿来和入口比对。`fallback-filter` 的 geosite 已废弃，检查结论不要依赖它。

## 逐项检查步骤

第一步记下 listen 的地址与端口，作为入口指纹。第二步检查 default-nameserver：用于解析 DNS 服务器的域名，必须是 IP（可为加密 DNS），不得是还要回到入口才能解析的域名，也不得写成入口自身。第三步展开 nameserver 与 fallback。纯 IP、字面 IP 的 tls，或手册示例中的 `https://doh.pub/dns-query`、`https://dns.alidns.com/dns-query`、`https://8.8.8.8/dns-query`，要判断主机是否会解析到入口，以及附加参数是否把连接送回依赖本 DNS 的出站。附加参数用 `#` 追加、`&` 连接；优先使用已有代理，不存在该名称则指定接口；`#RULES` 等同 respect-rules。第四步检查 nameserver-policy 每个值，规则同第三步；该字段优先于 nameserver/fallback，漏检一条就等于漏检入口回流。第五步看 proxy-server-nameserver 是否为空：为空则节点域名回头遵循 policy、nameserver、fallback，等价于可能指回入口解析链；不为空时再检查 proxy-server-nameserver-policy。第六步检查 direct-nameserver 与 direct-nameserver-follow-policy（默认不遵守 policy，仅当 direct-nameserver 不为空时生效）。第七步看 respect-rules：为 true 时 DNS 连接遵守路由，若规则把 DNS 送进需要本入口解析的代理，即指回入口。手册要求此时配置 proxy-server-nameserver，并强烈不建议与 prefer-h3 共用。

## 判断依据与失败下一步

指回入口的判断依据是：上游目标等于 listen，或上游主机名只能由本 listen 解析，或出站选择 `#proxy`、`#RULES`、respect-rules 后节点名仍要本 DNS 才能得到。use-hosts 为 true 且配置 hosts 已写死上游 IP 时，可视为名字不经入口递归；未写则不能这样认定。use-system-hosts 同理。h3、skip-cert-verify、ecs 只改变连接或查询内容，不证明上游离开了入口。失败下一步：把 default 改为 IP；为经代理的查询单独配置 proxy-server-nameserver；删除指向 listen 的 nameserver 项；拆开 respect-rules 与 prefer-h3。cache-algorithm 与 fake-ip-ttl 不参与这项检查，非必要不要改 TTL。未通过检查就不要认为转发已经离开入口。

资料：https://wiki.metacubex.one/config/dns/
