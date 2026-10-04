---
{
  "title": "Clash 怎样核对一个端口是否承担代理入口",
  "description": "Clash 怎样核对一个端口是否承担代理入口。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T18:59:12.560260+00:00",
  "lastmod": "2026-10-04T18:59:12.560260+00:00",
  "type": "post"
}
---

要核对某端口是不是 Clash 的代理入口，必须把它从外部控制 API 里排除，再确认它属于 http(s)、socks 或 mixed 这一类入站。官方全局配置明确区分两套东西：`external-controller` 等是 REST 控制面；`authentication` 覆盖的是 http(s)/socks/mixed 代理。`allow-lan` 的对象也写的是「代理端口」。只看「端口能连上」不够，能连上的也可能是 API、Unix socket 或 DOH 路径。

## 适用条件

本方法适用于：你手里有一份正在使用的全局配置，能读到监听地址；你怀疑应用、脚本或系统代理填错了端口；需要区分本机回环、局域网与控制面。不适用于：把运行模式当成端口用途——`mode` 的 `rule`/`global`/`direct` 只决定选路；也不适用于：仅凭控制页面能打开就断定该端口是代理，因为外部用户界面是挂在 API 地址 `/ui` 上的静态资源。

准备信息包括：`external-controller`、`external-controller-tls`、`external-controller-unix`、`external-controller-pipe`、`external-doh-server`；http(s)/socks/mixed 各自的端口；`bind-address`；`allow-lan`；`authentication` 与 `skip-auth-prefixes`；`secret`。缺任一项，就只能得出「还不能判定」，不能用猜测补全。

## 核对步骤

第一步，做排除。若待查端口与 `external-controller` 或 TLS API 一致，判定为控制面，不是代理入口。若它是 Unix socket 或 Windows namedpipe，文档写明从这些通道访问 API 不会验证 secret，仍属控制面。若只是 REST 上的 DOH 路径，同样不是 http(s)/socks/mixed 代理。

第二步，做归类。待查端口若对应 mixed、独立 HTTP 或 socks 入站，则属于代理入口。mixed 可同时作为 HTTP(S) 与 SOCKS 的入口；独立 HTTP 只覆盖 http(s) 语义。归类后，再用访问控制核对：`allow-lan` 为 false 时，其他设备不应经过该代理端口出网；为 true 时，还要看 `bind-address`、`lan-allowed-ips` 与 `lan-disallowed-ips`（黑名单优先）。本机 `127.0.0.1/8` 与 `::1/128` 默认在可跳过验证前缀里，这只能说明验证策略，不能单独证明端口角色。

第三步，做协议核对。用「客户端发出的是代理握手还是 REST」来交叉验证。代理入口应接受 HTTP 代理或 SOCKS；向该端口发送 API 风格请求成功，反而说明你可能仍打在控制面上。鉴权字段也要对应：代理看 `authentication`，API 看 `secret`。

## 判断依据、常见误判与下一步

可以判定「是代理入口」的依据：配置中该端口归属于 http(s)/socks/mixed；访问控制项按代理端口解释得通；客户端协议与入站类型一致。可以判定「不是」的依据：命中外部控制器及其 unix/pipe/tls/doh 变体；唯一能工作的口令是 `secret`；或该地址实际提供的是 `/ui` 一类 API 附属资源。

日志是辅助而不是替代。`log-level` 为 `silent` 时无法靠输出核对；`error` 只保留无法使用级别；需要观察一般连接应使用 `info`，需要尽量多的运行信息再用 `debug`。IPv6 待查地址还要看 `ipv6` 是否允许内核接受 IPv6 流量，避免把「未监听 IPv6」误判成「不是代理端口」。

若仍无法判定：不要改 `mode` 来「试出」端口用途；不要把 `bind-address: "*"` 理解成该端口一定是代理；不要因为本机免验证成功就断定角色正确。回到配置原文逐项对照，先移开所有与 `external-controller` 相同的填写，再只保留一个 http(s)/socks/mixed 端口做协议测试。局域网结果与本机不一致时，优先查 `allow-lan` 与地址段，而不是增加新的监听。

资料来源：https://wiki.metacubex.one/config/general/
