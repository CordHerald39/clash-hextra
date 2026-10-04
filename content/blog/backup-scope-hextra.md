---
{
  "title": "Clash 怎样核对备份是否含敏感链接与密钥",
  "description": "Clash 怎样核对备份是否含敏感链接与密钥。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T21:37:45.970753+00:00",
  "lastmod": "2026-10-04T21:37:45.970753+00:00",
  "type": "post"
}
---

备份若进入他人之手或公共存储，应先核对其中是否带有可接管内核或代理端口的密钥，以及可追踪的外部链接。下面只说明如何按全局配置字段逐项核对、怎样判断通过或不通过，以及核对无法完成时下一步。依据为全局配置说明。

## 适用条件

适用于已经生成配置副本、准备外传或归档之前。核对对象是全局配置中会出现的明文口令、证书、监听地址和下载 URL。部分 API 入口写明不会验证 secret，这类开关一旦出现在备份里，即使 `secret` 为空或未设置，仍应按敏感项处理。核对以即将外传的那份文本为准，而不是以内存中的运行值为准。

## 密钥与控制通道如何判定敏感

用户验证：`authentication` 使用 `user1:pass1` 这类用户对，作用于 http(s) / socks / mixed 代理。备份中只要出现该键，即含代理端口口令。`skip-auth-prefixes` 记录可跳过验证的 IP 段，说明中的示例包括 `127.0.0.1/8` 与 `::1/128`；若被改成过大网段，备份等于声明了谁能无口令接入，也应视为敏感。

API：`secret` 为 RESTful API 的访问密钥。`external-controller` 若监听所有地址，备份会暴露控制面可达范围。`external-controller-unix`、`external-controller-pipe` 从 Unix socket 或 Windows namedpipe 访问 API 不会验证 secret；`external-doh-server` 在 RESTful API 端口上开启 DOH，该路径不会验证 secret。备份里一旦启用这些键，即使没有 `secret` 字符串，也含有无密钥控制通道的描述。使用 `external-controller-tls` 时还依赖 `tls` 段；`certificate`、`private-key`、`ech-key` 可能是 PEM 正文或文件路径。内联 PEM 等于直接含私钥材料；若为路径，还要核对该路径文件是否被打进同一份归档。自 v1.19.18 起本地证书文件支持自动重载，路径型私钥同样不能随备份扩散。

绑定与跨域：`allow-lan`、`bind-address`、`lan-allowed-ips`、`lan-disallowed-ips` 本身不是口令，但会描述允许哪些地址进入代理端口，与 `authentication` 一起出现时足以复现入口。`lan-disallowed-ips` 黑名单优先级高于白名单，名单过宽或过空都应在核对记录里写明。`external-controller-cors` 的 `allow-origins` 与 `allow-private-network` 描述浏览器跨域，过宽的 `*` 应标明风险。`SAFE_PATHS` 与 `external-ui` 路径可能暴露主机目录结构，也应列入记录。

## 链接类字段、核对步骤与失败处理

外部资源 URL 可能暴露 GEO 源与界面源。`external-ui-url` 的说明示例为 `https://github.com/MetaCubeX/metacubexd/archive/refs/heads/gh-pages.zip`。`geox-url` 可含 geoip、geosite、mmdb、asn，说明中包括 `https://testingcf.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geoip.dat`、`https://testingcf.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geosite.dat`、`https://testingcf.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/country.mmdb`、`https://github.com/xishang0128/geoip/releases/download/latest/GeoLite2-ASN.mmdb`。`global-ua` 会用于外部资源下载，默认为 `clash.meta`。核对时应列出这些完整 URL，判断是否仍指向说明中的公共地址，或已被改成内网及其他带凭证的地址。订阅类完整网址不得臆造；若只有参数，只保留类似 `token=示例` 的片段。

步骤：在副本中定位 `authentication`、`secret`、`tls`、unix/pipe/DOH、`skip-auth-prefixes`、`external-controller`、`bind-address`、`external-ui-url`、`geox-url`。每项记录是否存在、是否明文、是否路径、是否监听非本机、是否声明不校验 secret。证书为路径时，打开归档看私钥文件是否同包。CORS 与局域网名单记录范围是否大于生产需要。`etag-support` 只影响外部资源下载校验，不替代密钥核对。

判断通过：上述密钥类键均不存在，或密钥已移除且不存在无校验监听，URL 仅为已登记的公共地址。判断不通过：任何用户口令、非空 `secret`、内联私钥、unix/pipe/DOH 已开，或 `allow-lan` 为 true 且绑定过宽却无说明。

失败时下一步：不要外传该副本。从工作副本删除或改写密钥与 PEM，关闭不校验 secret 的监听后重新生成并再核对一遍。路径型证书若必须保留，应与 YAML 分权存储，并在核对表写明私钥未进入同一文件。手册不提供自动脱敏开关，核对只能按字段完成。

https://wiki.metacubex.one/config/general/
