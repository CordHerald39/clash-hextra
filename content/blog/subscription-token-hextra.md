---
{
  "title": "Clash 分享导入报错截图前怎样检查敏感字段",
  "description": "Clash 分享导入报错截图前怎样检查敏感字段。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T18:34:22.757136+00:00",
  "lastmod": "2026-10-04T18:34:22.757136+00:00",
  "type": "post"
}
---

分享 Clash 导入报错截图前，应先把画面和贴出的片段当成会进入公共频道的文本，检查其中是否出现代理集合里的下载与节点凭据。官方代理集合字段里，真正危险的是能用来再次拉取或登录节点的值，而不是 `interval` 这类时间数字。

## 适用条件

适用于导入或解析 `proxy-providers` 失败、准备把报错界面或附近 YAML 发给他人的情况。`http` 类型必有 `url`；`file` 类型有 `path`；`inline` 类型有 `payload`。`http`/`file` 解析失败时也可用 `payload` 作备用代理，报错上下文里可能同时出现远程地址与本地节点口令。命令行 `-age-secret-key`、环境变量 `CLASH_AGE_SECRET_KEY` 以及 `SAFE_PATHS` 若出现在终端截图中，同样要处理。健康检查地址、过滤正则一般不是登录凭据，但不要和 `url` 抄在同一未遮挡区域。

## 截图前按字段检查

先找 `url:`。`type` 为 `http` 时该行是订阅下载地址，路径和查询参数都可能是令牌，必须改成无访问权的占位或裁掉取值。再找 `header`：文档示例含 `User-Agent` 与 `Authorization`，后者是请求凭据；若使用 age，还可能出现 `X-Age-Public-Key`。接着找 `age-secret-key`，其值形如文档中的 `AGE-SECRET-KEY-…`，属于解密密钥，不能出现在图中。

然后检查 `payload` 与报错附近的节点块：`server`、`port`、`cipher`、`password` 足以描述一条可连接代理，分享前应删除或改成明显假值。`path` 会暴露 HomeDir 下的文件布局；未写 `path` 时文件名来自 `url` 的 MD5，一般不还原链接，但完整配置若仍含 `url` 则必须遮挡。`proxy`（下载时使用的出站名）、`dialer-proxy`、`interface-name`、`routing-mark` 可能暴露本机网络布局，可按最小必要原则替换。`filter` / `exclude-filter` / `exclude-type` 不含口令，若截图只为说明语法可以保留。`override` 里的前缀、后缀、`skip-cert-verify` 等不是订阅令牌，但不要让它们把视线从上面的密钥字段上带走而漏看。

判断标准：把截图放大到能读文字后，第三者是否能复制出仍可访问的 `url`、`Authorization`、`age-secret-key` 或 `password`。只要能，就尚未检查完。

## 失败时的下一步

若检查后仍无法确定哪一行触发导入错误，只保留 `name`、`type`、不含取值的键名、以及报错原文，用占位符代替所有上述敏感值再截图。若图已发出，按泄露处理：轮换订阅 `url` 与头中令牌，更换节点口令和 `age-secret-key`，并停止补发未打码版本。本地再导入时，确认 `path` 仍在 HomeDir 或已用 `SAFE_PATHS` 声明，避免为了让对方“看见路径”而把目录结构拍进下一张图。解析失败需要对照 `payload` 备用节点时，只分享结构，不分享真实 `password`。

https://wiki.metacubex.one/config/proxy-providers/
