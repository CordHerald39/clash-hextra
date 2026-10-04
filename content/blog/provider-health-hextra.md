---
{
  "title": "Clash 怎样记录测试 URL 与间隔以便复现",
  "description": "Clash 怎样记录测试 URL 与间隔以便复现。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.525713+00:00",
  "lastmod": "2026-10-04T20:32:58.525713+00:00",
  "type": "post"
}
---

要复现代理集合上的延迟测试，必须把健康检查用的地址和间隔，从更新集合用的地址和间隔里单独抄出来。配置里存在两套 url 与两套 interval：类型为 http 时，集合自身的 url 与 interval 负责下载更新；health-check.url 与 health-check.interval 负责延迟测试。适用条件是已经或准备启用 health-check.enable，并需要把一次成功或失败的检查按相同条件重复。记录只使用实际配置中的值。若必须提到订阅类参数，只保留片段，例如 token=示例，不编写完整下载地址，也不编造域名。

## 健康检查本身要抄全哪些键

除测试地址与间隔外，至少同时记录：enable 是否为 true；timeout，单位毫秒；lazy，默认 true，不使用该集合节点时不测试；expected-status。判断依据是：另一人只凭这些字段就能写出同一段 health-check，而不需要猜测单位或默认值。interval 在健康检查中单位为秒，与 timeout 的毫秒不可写反，否则复现会把“尚未到达超时”写成“必定失败”，或把“早已超时”写成“还在测”。检查地址必须标明位于 health-check 下，避免复现者填进集合下载 url。文档将健康检查称为延迟测试，因此记录的是这次探测条件，不是下载条件。

## 还要记下哪些上下文才不会测错对象

应注明集合的 name 与 type（http、file 或 inline），以及更新用 interval，以免把集合刷新周期当成测试周期。若存在 filter、exclude-filter、exclude-type，应原样记录表达式，因为它们决定哪些节点会被拿去测。文档说明多个正则可用反引号区分；exclude-type 用竖线分割且不支持正则。若检查时集合可能未被使用，必须写明当时是否正在使用该集合，否则懒惰状态会使复现变成“没有任何测试动作”。下载用 proxy、header、path、size-limit 与健康检查无替代关系，只有怀疑复现环境连集合都未加载时才需要附加。http 或 file 解析失败时可用 payload 作为备用代理，应注明当时名单是否来自 payload，否则他人会去复现一份并不存在的下载结果。

## 建议的记录步骤和复现失败时下一步

按固定顺序写成纯文本：集合名称与类型；health-check 的 enable、url、interval、timeout、lazy、expected-status；当时是否使用该集合；筛选三项原文；更新用 interval。操作步骤是从配置复制 health-check 子键，不要只从更新记录里抄一个间隔数字；核对检查地址是否位于 health-check 下；再并排核对两个 interval 的键名和单位。复现仍无测试时，下一步先对齐 lazy 与使用状态。复现必然超时时，对齐 timeout 单位与检查地址。复现对象数量不同时，对齐 filter 与 exclude-type。不要补充文档未给出的界面名称，也不要对一次复现的时延作保证。需要核对字段含义时，以对应资料中的代理集合说明为准。

资料：https://wiki.metacubex.one/config/proxy-providers/
