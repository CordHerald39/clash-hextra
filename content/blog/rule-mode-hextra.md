---
{
  "title": "Clash 怎样固定同一个请求比较规则命中结果",
  "description": "Clash 怎样固定同一个请求比较规则命中结果。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T18:59:12.556304+00:00",
  "lastmod": "2026-10-04T18:59:12.556304+00:00",
  "type": "post"
}
---

要比较两次规则命中是否一致，先保证两次看到的是同一条请求、同一套全局条件。本页没有提供「规则命中结果」字段，也没有规定用哪条 API 路径打印命中。能做的是：把运行模式固定为规则匹配，并冻结会影响解析地址、进程匹配、出站网卡和 GEO 数据的项，避免两次请求因全局配置不同而不可比。

## 适用条件

适用于已经使用 `mode: rule`（或默认规则模式），需要前后对比同一次访问在规则匹配下表现是否变化的场合。适用于刚改过 `ipv6`、`tcp-concurrent`、`find-process-mode`、GEO 更新或策略组选择缓存的场合。若两次之间把 `mode` 改成了 `global` 或 `direct`，比较的就不再是规则命中。

## 需要固定的项与判断依据

1. 固定运行模式。两次比较都保持 `rule`，不要用 `global`（走 `GLOBAL` 策略组所选出口）或 `direct`（全局直连）混进对照。判断：配置中的 `mode` 在两次请求之间没有变化。
2. 固定地址与并发。`ipv6` 保持同一取值，避免一次接受 IPv6、一次不接受。`tcp-concurrent` 为 true 时会用 dns 解析出的所有 IP 连接并取第一个成功者，同一域名两次可能落到不同地址。比较命中前应固定该开关。`profile.store-fake-ip` 为 true 时储存 fakeip 映射表，域名再次连接使用原有映射地址，有利于同一域名对上同一映射；为 false 时映射不必保持。判断：对照前后的地址族、并发开关和 fakeip 是否储存，三者都不变，请求才更接近「同一个」。
3. 固定进程匹配与出站。`find-process-mode` 在 `always` / `strict` / `off` 之间切换，会改变是否匹配进程。`interface-name` 与 `routing-mark`（Linux）改变出站接口和标记。判断：比较窗口内不要改这三项，否则即使模式仍是 `rule`，匹配条件和出站路径已经不是同一实验。
4. 固定 GEO 数据版本。`geo-auto-update` 为 true 时按 `geo-update-interval`（小时）更新；`geox-url` 和 `geodata-mode`（mmdb 或 dat）决定文件来源与类型。比较过程中若文件被自动换成另一份，按 GEO 数据作出的匹配可以变化。`etag-support` 默认 true，影响外部资源是否按 ETag 下载；`global-ua` 默认 `clash.meta`，影响下载所用 UA。判断：比较前记下当前 GEO 模式、加载器、下载地址和是否自动更新，两次之间不要换文件。
5. 固定策略组选择缓存。`profile.store-selected` 为 true 时储存 API 对策略组的选择供下次启动使用。比较命中时应避免一次启动读到旧选择、另一次没有。日志方面，`log-level` 调到 `debug` 可在控制台和控制页面看到尽可能多的运行信息，但资料未说明其中包含规则命中名称；两次须用同一日志等级，避免一次 `silent`、一次 `debug` 造成「有无输出」的假差异。

从其他设备复测时，还要固定 `allow-lan`、`bind-address`、地址段黑白名单和 `authentication`，否则入口都不是同一请求。

## 失败时下一步

两次结果对不上，先列出上述固定项是否被改动，尤其是 `mode`、`tcp-concurrent`、`store-fake-ip` 和 GEO 自动更新。需要更多输出时只在控制台和控制页面提高 `log-level`，不要假定存在本页未记载的命中对比界面。本页也未给出 RESTful API 的具体路径，不能把 `external-controller` 的监听地址直接写成命中查询接口。

资料来源：
https://wiki.metacubex.one/config/general/
