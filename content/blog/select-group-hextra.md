---
{
  "title": "Clash 怎样从规则一路追踪到最终节点",
  "description": "Clash 怎样从规则一路追踪到最终节点。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T21:03:35.922099+00:00",
  "lastmod": "2026-10-04T21:03:35.922099+00:00",
  "type": "post"
}
---

从规则一路追踪到最终节点，指的是：从一个策略名出发，沿着 proxy-groups 的 name、type、proxies、use 和筛选字段，走到不能再展开的出站代理。组名不等于节点名。官方代理组字段里，每一层都可能改写下一跳，应按字段记录，而不是在第一层名字上停下。

## 适用条件：什么时候必须整链核对

当 proxies 里出现其他策略组，同时使用 use 或 include-all 系列，或配置了 filter、exclude-filter、exclude-type 时，命中得到的名字只是链路入口。default-selected、组中第一个节点、empty-fallback 也会在未选择或组空时改写终点。此时必须从规则给出的策略名往下走，直到成员是出站代理为止。

判断依据：当前名字能否在 proxy-groups 的 name 中找到；若能找到，它就还不是终点。name 含特殊符号时，确认配置里使用了引号，避免追踪到「看起来同名、实际未定义」的一层。

## 按 name、type、proxies 与 use 向下展开

定位到组后先记录 type。成员列表优先看 proxies：其中每一项可能是 DIRECT 这类出站，也可能是另一个组的 name。use 指向代理集合。include-all 按名称排序引入全部出站代理和代理集合，但不含策略组；include-all-proxies 只引入全部出站代理；include-all-providers 只引入全部代理集合，且会使「引入代理集合」失效。追踪时把这三类开关展开成实际名单，再与 proxies 合并理解——策略组只能来自 proxies。

若当前选择（或 default-selected，或默认的第一个节点）指向另一组，对那一组重复上述步骤。url 健康检查只覆盖 proxies 里的代理，不覆盖 use 引入的集合，因此不能用检查结果代替「集合里到底有谁」。lazy 默认为 true，未选择到当前策略组时不进行测试，未测试也不等于该层不存在。

## 筛选、空组回退、UDP 对终点的修改和失败处理

filter 与 exclude-filter 用关键词或正则限制集合与「引入所有出站代理」的结果，多段正则可用反引号区分。exclude-type 不用正则，用 | 分割类型，仅排除引入的出站代理，无视大小写。追踪记录里应写下裁剪后的名单，否则会把已被排除的节点当成终点。

组为空时走 empty-fallback，默认 COMPATIBLE，且不能填代理组。此时终点是该 proxy 名，而不是组名本身。disable-udp 为 true 则该组不提供 UDP，UDP 请求的终点不能按 TCP 选择来记录。hidden、icon 只影响 api 展示，追踪转发时应忽略。组上的 interface-name、routing-mark 已弃用，最终出口的网卡与标记要在节点上核对，优先级为代理节点大于代理策略大于全局。

失败时下一步：若某层 name 找不到对应组，检查拼写与引号。若展开后名单为空，检查三条筛选和 empty-fallback 是否为合法 proxy。若停在集合名上，改查 use 或 include-all-providers，而不是看 url 探测。若只对 proxies 做了健康检查，不要推断集合内节点的状态。每一层记下 type、选择项、是否组空、UDP 是否被禁用，直到成员不能再指向 proxy-groups 为止。

https://wiki.metacubex.one/config/proxy-groups/
