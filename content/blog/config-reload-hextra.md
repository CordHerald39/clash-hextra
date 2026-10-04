---
{
  "title": "Clash 如何记录一次配置重载的前后差异",
  "description": "Clash 如何记录一次配置重载的前后差异。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T18:59:12.555212+00:00",
  "lastmod": "2026-10-04T18:59:12.555212+00:00",
  "type": "post"
}
---

本页没有提供配置差异记录器。一次「重载」前后要记什么，只能按该页已经列出的全局键来做清单，并单独记下与 YAML 平行的缓存和外部文件事件。混在一起的话，你会把策略组选择、fake-ip 映射、GEO 定时更新和 TLS 文件自动重载，误写成同一次配置重载的结果。

## 适用条件

适用于改动前能打开当前 YAML、改动后仍能看到控制台或控制页面的场合。适用于记录 mode、日志等级、入站与验证、外部控制、profile、tls、GEO 相关项。不适用于记录本页未出现的接口返回体或未记载的重载时间戳。日志仅在控制台和控制页面输出，记录观察通道时不要写成其他位置。

## 按字段清单记录 YAML 前后值

先抄录改前取值，再抄录改后取值，键名以本页为准，例如：allow-lan、bind-address、lan-allowed-ips、lan-disallowed-ips、authentication、skip-auth-prefixes、mode、log-level、ipv6、keep-alive-interval、keep-alive-idle、disable-keep-alive、find-process-mode、external-controller 及其 cors/unix/pipe/tls 变体、secret、external-ui、external-ui-name、external-ui-url、profile 下两项、unified-delay、tcp-concurrent、interface-name、routing-mark、tls 中证书与私钥和 ech-key、geodata-mode、geodata-loader、geo-auto-update、geo-update-interval、geox-url、global-ua、etag-support。

对照时注明缺省：mode 默认为规则模式，find-process-mode 默认为 strict，ipv6 默认为 true，geodata-mode 默认为 false，lan-allowed-ips 默认为 0.0.0.0/0 与 ::/0，lan-disallowed-ips 默认为空。删除某键后的「新值」应记为缺省，而不是记为空白。黑名单优先于白名单，记录入站差异时要写清优先级，否则无法解释为何白名单仍被挡住。

## 把缓存和外部事件写成另一栏

store-selected 与 store-fake-ip 不是 YAML 里某一代理的字段，而是 API 选择和映射表。若二者为 true，启动后策略组选择或域名映射与改前相同，应记为缓存仍在，而不是记为 YAML 重载失败或成功。tls 本地文件自 v1.19.18 可自动重载：文件内容变而 YAML 路径未变，应记为证书材料事件。geo-auto-update 按小时间隔工作，etag-support 影响外部资源是否视为有更新，这两类都不要记进「本次 YAML 重载」。全局 TLS 指纹已弃用，即使文件里还能看到，也不应记为有效差异。

日志栏只记等级是否与 log-level 一致：silent 应无输出，error 仅无法使用的错误，warning 含不影响运行的错误，info 含一般运行内容，debug 尽可能全部。等级与输出长期不符，只能记为「无法证明新字段已作用于当前进程」。

## 失败时下一步

若清单上 YAML 已变、日志与缓存均无对应变化，本页不能补写一条重载成功记录。保留改前文本作为对照原件，不要用 GEO 下载或外部界面 zip 更新来填充空白。Unix socket 与 namedpipe 不验证 secret，经它们看到的状态不能单独用来证明 YAML 中的 secret 前后差异已生效。路径离开工作目录时，先把 SAFE_PATHS 是否已设置记入同一份对照，再判断是配置差异还是路径策略拒绝加载。

资料来源：
https://wiki.metacubex.one/config/general/
