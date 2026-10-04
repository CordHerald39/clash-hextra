---
{
  "title": "Clash 怎样记录缓存相关测试的时间间隔",
  "description": "Clash 怎样记录缓存相关测试的时间间隔。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.555817+00:00",
  "lastmod": "2026-10-04T20:32:58.555817+00:00",
  "type": "post"
}
---

## 为什么间隔必须写进记录

Clash DNS 提供 `cache-algorithm`（`lru` 为默认，`arc` 可选），并在 fake-ip 模式下提供 `fake-ip-ttl`。这两项都让何时查询成为实验变量。若不记录间隔，就无法区分第二次结果来自新的上游查询、尚未淘汰的缓存，还是尚未超过返回 TTL 的 fake-ip 映射。记录间隔不是为了声称某种算法会在固定秒数后清空，文档没有给出这种保证；而是为了让同一次验证里的每次查询都能对齐同一份配置快照。

适用条件：`dns.enable` 为 `true`，并且你正在验证 nameserver、fallback、policy、fake-ip 过滤或 hosts 相关改动。`enable` 为 `false` 时走系统 DNS，再记录 Clash 侧间隔没有对应对象。

## 一份间隔记录要写明的字段

缺一项就无法事后判断，至少包含：

1. 查询时刻：每次解析的开始时间，精确到秒，用于成对计算短间隔和长间隔。
2. 配置快照：`cache-algorithm`、`enhanced-mode`（`fake-ip` 或默认 `redir-host`）、是否出现 `fake-ip-ttl`、`fallback-lazy-query`（默认 `false`）。
3. 域名类别：普通域名、代理节点域名还是 direct 出口域名。后两类可能走 `proxy-server-nameserver` / `proxy-server-nameserver-policy`，或 `direct-nameserver`（`direct-nameserver-follow-policy` 默认不遵守 policy）。
4. 是否可能被 hosts 短路：`use-hosts`、`use-system-hosts` 默认 `true`。
5. 是否被 `nameserver-policy` 提前指定服务器。
6. `fallback-filter` 相关：`geoip`、`geoip-code`（默认 `CN`）、`ipcidr`、`domain`；不要把已废弃的 `geosite` 当成有效依据。
7. fake-ip 过滤：`fake-ip-filter-mode`（默认 `blacklist`，另有 `whitelist` 与 `rule`）以及该域名会得到 fake-ip 还是 real-ip。

判断依据：只有上述字段在两次查询之间保持不变，时间间隔才有资格作为自变量。若中间改了 YAML，间隔记录作废，必须重新取零点。`ipv6` 为 `false` 时还应注明 AAAA 为空解析，避免把空应答误记成缓存命中。

## 短间隔和长间隔怎么记

第一步，配置冻结后做第 0 次查询，记下时刻 T0 与结果（含是否 fake-ip、AAAA 是否为空）。

第二步，不改配置，立刻做第 1 次查询，时刻 T1，记录 Δ1 = T1 − T0。这一段称为短间隔，用来观察缓存仍可能命中时的表现。Δ1 的意义是短，不是已过期。

第三步，继续冻结配置，做第 2 次查询，时刻 T2，Δ2 = T2 − T0，称为长间隔。若使用 fake-ip 且显式配置了 `fake-ip-ttl`，把该 TTL 当作返回寿命的参考来选择 Δ2，但仍不要为了测试去改 TTL，文档要求非必要勿改。

第四步，把三次结果并排：若 Δ1 与 T0 相同而 Δ2 不同，记为短间隔命中旧结果、长间隔出现变化，缓存相关假设可以进入下一轮；若三次全同，记为间隔未解释差异，下一步转向 policy、hosts、过滤或 `default-nameserver`（必须为 IP）。

第五步，若开启了 `fallback` 且 `fallback-lazy-query` 为 `true`，额外记录每一次是否可能只完成了 nameserver 查询。懒查询改变的是这一次有没有发 fallback，和缓存间隔是不同维度，必须分列，不能写成一个时间数字。

## 记录失败时的下一步

常见失败是只写了等了一会儿而没有 T0/T1/T2，或在等待期间又改了 `nameserver-policy`。这两种记录都不能用来讨论缓存。下一步应丢弃该组数据，重新冻结配置后再取三个时刻。若节点域名和网页域名写在同一张表里，按文档拆成两张：节点只用 `proxy-server-nameserver` 通道的间隔，普通域名用 nameserver/fallback 通道的间隔。不要用修改 `fake-ip-ttl` 或切换 `cache-algorithm` 来加速验证，算法项只说明用 lru 还是 arc 做替换，并不提供可填写的过期秒数。

资料来源：https://wiki.metacubex.one/config/dns/
