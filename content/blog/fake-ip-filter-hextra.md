---
{
  "title": "Clash 怎样用最小过滤项验证兼容性问题",
  "description": "Clash 怎样用最小过滤项验证兼容性问题。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.554057+00:00",
  "lastmod": "2026-10-04T20:32:58.554057+00:00",
  "type": "post"
}
---

验证某域名是否因 fake-ip 不兼容而出现连接问题时，应使用尽可能小的过滤项，避免一次写入过多通配导致范围失控。以下仅依据官方 DNS 配置说明最小验证的适用条件、具体写法、判断依据以及验证失败后的下一步。

## 最小验证的适用条件与准备
仅当 dns.enable 为 true 且 enhanced-mode 为 fake-ip 时才进行此项验证。fake-ip-range 必须已设置，否则虚假地址来源不明。适用场景是单个应用或单个域名出现异常，而其余流量正常。判断是否值得做最小验证的依据是：该域名在 fake-ip 下失败、在 redir-host 下成功，或该域名类似文档示例中的 '*.lan'。先确认 fake-ip-filter-mode 当前值。默认 blacklist 下添加一条即排除该名称的 fake-ip；若已是 whitelist，添加一条反而会让该名称开始使用 fake-ip，验证方向完全相反，必须先改回 blacklist 或明确使用 rule 模式。

ipv6 为 false 时只验证 A 记录即可。use-hosts 与 use-system-hosts 保持默认 true，避免 hosts 项干扰最小集合。respect-rules 建议在验证期间保持 false，防止路由规则改变 DNS 出口。

## 最小条目的具体写法与判断
普通模式最小项就是一条精确通配或单域名，例如文档示例风格的 '*.lan' 或单个完整名称。rule 模式则写一条 DOMAIN,具体域名,real-ip，最后必须补 MATCH,fake-ip 以保持其余域名行为不变。RULE-SET 即使 behavior 为 domain 也属于集合，体积大于单条，不适合最小验证。GEOSITE 同样范围过大。操作步骤：备份原 filter 列表，清空后只保留这一条，保存并重新加载核心，观察该域名连接是否恢复而其他域名是否仍使用 fake-ip。

判断成功的依据是文档行为：匹配项在 blacklist 或对应 real-ip 动作下不会下发 fakeip 映射。同时检查 nameserver-policy 是否已为该域名指定了独立服务器，若有，解析结果可能本来就不是 fake-ip 网段，验证会失去意义。fallback-filter 的 domain 或 ipcidr 若包含该名称，实际解析路径已变，应先移出再验证。direct-nameserver 只影响 DIRECT 出口，最小验证期间可保持为空。

## 验证失败后的缩小或切换
若单条加入后无改善，说明该域名并非不兼容原因，或语法未命中。下一步改用更精确的 DOMAIN 而不是 DOMAIN-SUFFIX，或在 rule 模式把动作写成 real-ip 再试。仍失败则把 mode 改为 whitelist 做反向验证：只让这一条使用 fake-ip，看问题是否出现。proxy-server-nameserver 必须已配置，否则节点域名解析可能循环，干扰对普通域名的判断。fake-ip-ttl 非必要勿改。验证结束后立即恢复原来的完整列表。全部步骤严格对应官方 fake-ip-filter、mode 及 rule 模式语法说明。

https://wiki.metacubex.one/config/dns/
