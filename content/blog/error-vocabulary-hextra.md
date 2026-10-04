---
{
  "title": "Clash 怎样保留完整错误类型同时隐藏敏感值",
  "description": "Clash 怎样保留完整错误类型同时隐藏敏感值。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T22:13:15.044166+00:00",
  "lastmod": "2026-10-04T22:13:15.044166+00:00",
  "type": "post"
}
---

## 适用条件与必须保留的类型信息

对外提供 Clash 日志或配置片段时，要同时做到：错误类型仍可对照官方全局配置归类，敏感取值不能被直接复用。适用条件是这些文字来自内核在控制台或控制页面的输出，或来自全局配置本身。官方用 `log-level` 约束输出：`silent` 不输出；`error` 仅输出发生错误至无法使用的日志；`warning` 含不影响运行的错误以及 error；`info` 再含一般运行内容；`debug` 尽可能输出运行中所有信息。级别词本身属于类型信息，必须保留，因为它区分「已无法使用」和「仍在运行」。

除此之外还应保留：`mode` 的类别（rule / global / direct）；失败所属配置域（入站验证、出站保活、外部控制、GEO 或缓存）；原文中的英文错误类别词。判断依据是：读到这些键名和级别后，可以回到全局配置对应小节继续查，而不需要账号、口令、证书 PEM 或精确监听地址。

## 敏感值清单和替换步骤

下列字段在官方文档中以凭据或可定位信息出现，对外时键名保留、取值替换。

`authentication` 使用 `user1:pass1` 形式的用户验证，应改成「用户名已替换:口令已删除」这样的占位，但保留这是 http(s)/socks/mixed 用户验证这一类型。`secret` 是 API 访问密钥，只保留「已填写」或「空」，不留原文。`tls` 下的 certificate、private-key、ech-key 为 PEM 或路径，只保留「已配置证书 / 私钥 / ECH」以及是否为本地文件（官方注明自 v1.19.18 起，本地文件支持自动重载），删除 PEM 块本身。`skip-auth-prefixes`、`lan-allowed-ips`、`lan-disallowed-ips`、`bind-address` 含具体地址或地址段，对外改成「回环 / 单 IPv4 / 单 IPv6 / 自定义段」分类；若地址能画出内网拓扑，则不要原文。`external-controller`、`external-controller-tls`、Unix socket 名、Windows namedpipe 名属于控制面入口，保留「本机监听或监听全部 IP」以及是否启用 TLS，不写完整套接字字符串。

操作顺序如下。先复制原文。再用同一套占位符替换所有口令、PEM、具体 IP 和路径中的用户目录。然后检查是否仍能看见 error/warning/info/debug 以及配置键名。最后查看 `global-ua`：官方默认值为 clash.meta，若已被改成可识别组织或环境的字符串，按敏感值处理。`external-ui` 路径若写在工作目录外，官方要求设置 `SAFE_PATHS` 环境变量（语法同本操作系统 PATH：Windows 下分号分割，其他系统下冒号分割）；对外只写「路径在工作目录内」或「已加入安全路径」，不写绝对路径。

## 用日志级别和监听模型控制暴露面

隐藏敏感值不是把 `log-level` 改成 `silent`，那样错误类型会一并消失。排障时用 `warning` 或 `info`，保证无法使用的 error 与不影响运行的 warning 仍在；需要更完整类型时再短时使用 `debug`，抓取结束后降回。官方写明日志仅在控制台和控制页面输出，因此不要把控制页面连同 `secret` 一起放到不可信网络。若把 `external-controller` 从 `127.0.0.1` 改为监听全部 IP，属于扩大控制面；对外说明时可保留 CORS、`allow-private-network` 以及「Unix socket / namedpipe 不验证 secret」这些类型说明，但仍然不写出 secret 原文。开启 `external-doh-server` 时，官方同样写明该 URL 不会验证 secret，对外只保留「已开启且不验证 secret」这一类型，不提供可访问入口。

若替换后只剩「有问题」而看不出类型，说明删过了：回退到仍含级别词和配置键名的版本，只继续遮盖取值。不确定某字段是否敏感时，对照它是否出现在用户验证、API 密钥、TLS、绑定地址或跳过验证前缀小节；是则遮盖取值，不是则保留键与类型。不要用关闭 `allow-lan` 或改 `mode` 当作隐藏日志的手段，那会改变流量路径，不是脱敏。若控制面必须给他人看，下一步是把监听改回本机地址、确认 `secret` 非空，并避免使用不验证 secret 的 socket、pipe 或 DOH 路径来传递完整日志。

资料：
https://wiki.metacubex.one/config/general/
