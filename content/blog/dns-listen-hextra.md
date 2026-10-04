---
{
  "title": "Clash 怎样确认客户端实际查询了预期入口",
  "description": "Clash 怎样确认客户端实际查询了预期入口。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.522926+00:00",
  "lastmod": "2026-10-04T20:32:58.522926+00:00",
  "type": "post"
}
---

在 Clash（mihomo）中，确认客户端是否查询了预期入口，只能依据官方 DNS 字段的启用条件、对象分类和优先级，而不能依靠未记载的界面或主观感受。预期入口是指该次查询按配置应使用的解析服务器类别，包括 `nameserver-policy` 指定值、默认 `nameserver`、`fallback`、`proxy-server-nameserver` 与 `direct-nameserver`。

## 适用条件

仅当 `dns.enable` 为 `true` 时讨论才有意义。官方写明：是否启用，如为 `false`，则使用系统 DNS 解析。此时即便列表里写了预期服务器，客户端也不会向其查询。

查询对象必须分类。解析 DNS 服务器自身域名时看 `default-nameserver`，必须为 IP，可为加密 DNS。解析代理节点域名时看 `proxy-server-nameserver`，仅用于该用途；如果不填则遵循 `nameserver-policy`、`nameserver` 和 `fallback`。`proxy-server-nameserver-policy` 格式同 `nameserver-policy`，当且仅当 `proxy-server-nameserver` 不为空时生效。解析 direct 出口域名时看 `direct-nameserver`；`direct-nameserver-follow-policy` 是否遵循 `nameserver-policy`，默认为不遵守，仅当 `direct-nameserver` 不为空时生效。

`respect-rules` 表示 dns 连接遵守路由规则，需配置 `proxy-server-nameserver`，强烈不建议和 `prefer-h3` 一起使用。`listen` 是 DNS 服务监听，支持 udp、tcp。客户端解析器必须指向该监听，查询才会进入 Clash。

## 具体场景下的对照步骤

场景：浏览器访问普通业务域名，配置希望走 `nameserver` 中某一加密 DNS，需要确认没有落到系统 DNS、政策指定服务器或 fallback。

第一步，读 `nameserver-policy`。该字段指定域名查询的解析服务器，可使用 geosite，优先于 `nameserver`/`fallback` 查询。键支持域名通配，值支持字符串或数组。若域名匹配某一键，预期入口就是该值，而不是 `nameserver` 里的那一台。

第二步，读 `fallback` 与 `fallback-filter`。配置 `fallback` 后默认启用 `fallback-filter`，`geoip-code` 为 cn。满足条件的将使用 fallback 结果或只使用 fallback 解析。`geoip` 为是否启用 geoip 判断；除 `geoip-code` 国家 IP 外其他 IP 视为污染，该国结果直接采用，否则采用 fallback。`domain` 中域名被视为已污染，匹配后直接使用 fallback、不去使用 nameserver。`ipcidr` 网段结果视为污染，nameserver 解析出这些结果时采用 fallback。`geosite` 已废弃，请使用 `nameserver-policy`。若域名落入上述强制路径，预期入口是 fallback 而非 nameserver。

第三步，读条目附加参数。对公网 DNS 使用 `#` 附加、`&` 连接。优先使用已有代理，不存在该名称的代理则指定接口连接。`#RULES` 等同于 `respect-rules`。如需经过代理查询，应配置 `proxy-server-nameserver`，以防出现鸡蛋问题。附加参数会改变实际连接的入口，读配置时必须把 `#` 前后拆开。

第四步，排除 hosts 与 fake-ip 过滤。`use-hosts` 是否回应配置中的 hosts，默认 `true`；`use-system-hosts` 是否查询系统 hosts，默认 `true`。`enhanced-mode` 为 fake-ip 时，`fake-ip-filter` 内地址不会下发 fakeip 映射；`fake-ip-filter-mode` 可选 blacklist、whitelist、rule。

## 判断依据与失败时下一步

判断实际会查询预期入口的依据是同时成立：`enable` 为真；查询打到 `listen`；按政策优先、再 fallback 条件、再默认 `nameserver` 能唯一确定一类服务器；对象不是应走节点专用或 direct 专用列表的域名；hosts 未抢答。`cache-algorithm` 仅为 lru 或 arc，不改变入口选择。`ipv6` 为 `false` 时回应 AAAA 空解析，不能据此判断没查上游。

若仍无法对应，下一步检查 `default-nameserver` 是否写成非 IP，导致连加密 DNS 的域名都无法解析；再检查 `respect-rules` 与鸡蛋问题是否使查询连不出去。不要用节点列表替代 DNS 入口来验证。完整含义见官方 DNS 配置说明。

参考资料：https://wiki.metacubex.one/config/dns/
