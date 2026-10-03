---
{
  "title": "nameserver-policy 与 proxy-server-nameserver：网站和节点分别由谁解析",
  "description": "梳理默认解析、按域名指定解析与节点服务器解析，避免代理 DNS 依赖循环。",
  "draft": false,
  "date": "2026-10-04T06:25:42+08:00",
  "lastmod": "2026-10-04T06:25:42+08:00",
  "type": "blog",
  "author": "编辑部",
  "tags": [
    "nameserver-policy",
    "proxy-server-nameserver",
    "DNS 依赖循环"
  ]
}
---

网页域名和代理节点服务器域名都需要解析，但它们不一定走同一条路径。mihomo 的 `nameserver`、`nameserver-policy`、`proxy-server-nameserver` 各有职责；把所有 DNS 请求强制走一个尚未解析成功的代理，可能形成“先有节点才能查询，先有查询才能连节点”的依赖。

## 先分三类问题

普通网站域名通常按默认解析与域名策略处理。`nameserver-policy` 为匹配域名指定解析服务器，文档说明它优先于 nameserver/fallback。`proxy-server-nameserver` 专门用于节点服务器域名；不填写时，会遵循 nameserver-policy、nameserver、fallback 的配置。

另外，`default-nameserver` 用于解析 DNS 服务器自己的域名，并非“给所有网站用的第一 DNS”。`direct-nameserver` 则用于直连出口域名，其是否遵守策略还有独立选项。字段相似，不应一股脑写成同一地址。

## 把依赖关系画在自己的纸上

先写节点 server 是 IP 还是域名，再写节点域名由谁解析，以及到该解析器的连接是否依赖这个节点。若解析器经由同一节点访问，而节点域名还未获得 IP，就需要调整依赖链；继续重试不会自动补出起点。

文档要求使用 `respect-rules` 时配置 proxy-server-nameserver，并对它与 prefer-h3 同时使用给出不建议提示。是否启用应依据实际需求和版本，不能把网上一份 DNS 全选配置当作通用最佳方案。

## 如何判断修改是否有效

先只验证节点域名能否解析并建立连接，再验证网站解析与访问。节点能连、特定网站不通时，检查域名策略；节点本身就解析失败时，优先检查节点解析路径。日志中的域名、结果与时间有助于区分两类失败，但应脱敏公开信息。

改变 DNS 后重新建立连接，保留原配置便于回退。不要用浏览器自带加密 DNS 的单次成功证明内核 DNS 已正确，也不要通过关闭证书验证掩盖解析器身份错误。

相关：[连接与模式](/blog/modes/)。依据：[mihomo DNS 字段与附加参数](https://wiki.metacubex.one/config/dns/)。这里解释字段职责，不宣称任何公共 DNS 在所有网络中可达。
