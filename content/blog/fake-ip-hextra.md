---
{
  "title": "Clash 怎样区分合成地址与代理出口地址",
  "description": "Clash 怎样区分合成地址与代理出口地址。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.550830+00:00",
  "lastmod": "2026-10-04T20:32:58.550830+00:00",
  "type": "post"
}
---

区分 Clash 的合成地址与代理出口地址，关键是看 IP 是否落在 dns 段声明的 fake-ip 网段，以及该地址出现在连接的哪一跳。合成地址只存在于应用到 Clash 这一段，代理出口地址则是规则命中后真正发往远端的目标或节点 server。

## 用网段特征做第一层判断
合成地址来自 fake-ip-range，文档示例为 198.18.0.1/16，IPv6 对应 fake-ip-range6。该范围属于保留用途，正常网站不会把业务放在其上。代理出口地址是 proxies 里节点的 server 经 proxy-server-nameserver（或 nameserver-policy）解析得到的 IP，DIRECT 时则是网站真实解析结果，一般为公网单播。判断依据：连接五元组的目的 IP 若在配置的合成前缀内，即为 fake-ip；若为节点 IP 或站点真实 IP，则为出口侧地址。enhanced-mode 不是 fake-ip 时，应用侧也不会出现合成地址。

## 结合 DNS 与节点解析字段交叉验证
阅读 dns.enable、enhanced-mode、fake-ip-filter。filter 命中（视 mode 为 blacklist/whitelist/rule）会让部分域名改为 real-ip，此时应用侧看到的就不再是合成地址。proxy-server-nameserver 仅用于解析节点域名，且仅在该字段非空时 proxy-server-nameserver-policy 生效，避免与普通 nameserver 混淆。respect-rules 为 true 时 DNS 查询本身也走路由，但仍不改变合成地址只出现在客户端到内核这一跳的事实。适用条件是同时启用 fake-ip 与代理出站。

## 观察顺序与无法区分时的下一步
在连接或日志中分别查看应用侧 destination 与实际 remote。前者在 fake-ip 模式下常为 198.18.x.x，后者是规则选出的 proxy 或 DIRECT 目标。失败时两者看起来相同，可能是该域名被 filter 排除、应用未走 Clash DNS，或流量未进入 TUN。下一步：对同一域名分别向 listen 端口和公共 DNS 查询，确认是否返回合成前缀；再核对规则命中的出站。不要用合成地址做地理或延迟判断。文档同时指出 default-nameserver 必须为 IP，否则节点或上游域名都无法正确解析，会进一步混淆两类地址。

参考资料：https://wiki.metacubex.one/config/dns/
