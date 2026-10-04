---
{
  "title": "Clash 问题反馈：怎样写清客户端、Mihomo 内核与系统版本",
  "description": "Clash 问题反馈：怎样写清客户端、Mihomo 内核与系统版本。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T18:34:22.748095+00:00",
  "lastmod": "2026-10-04T18:34:22.748095+00:00",
  "type": "post"
}
---

反馈 Clash 相关故障时，只写一句“最新版不行”几乎无法对照公开手册复现。有效描述至少要分开三层：外壳（常被叫作客户端）、mihomo 内核、以及系统/运行平台。全局配置说明并不提供现成的工单模板，但给出了别人用来核对行为的字段和平台限制，应按这些来写，而不是按某一教程的菜单名来写。

## 适用条件与三层信息

适用于准备向他人说明现象、需要对方能对照 MetaCubeX/mihomo 全局配置做判断的场景。本页覆盖的是内核配置与部分操作系统差异，不覆盖某一商业客户端的关于页。因此：

- 客户端：写你实际启动的封装名称与其自称版本，并明确“以下配置按 mihomo 手册核验”。不要把 `global-ua` 默认值 clash.meta 当成应用名。
- 内核：写你是否使用该手册中的全局项，以及是否依赖带版本门槛的行为。
- 系统：写 OS 与形态（桌面、Android、路由器等），因为同一键在不同平台含义不同。

缺少任一层，对方只能猜测是界面封装问题还是内核问题。

## 按可核验字段写内核，而不是写观感

内核部分应尽量贴手册键名，便于对照：

- 运行：`mode` 取值（rule / global / direct；默认规则；global 还需说明 GLOBAL 策略组中的选择）、`ipv6`、`log-level`。并写明日志是在控制台还是控制页面看到的——手册写明仅这两处输出。若你把级别设为 silent 却抱怨没有日志，反馈本身无效。
- 入站与局域网：`allow-lan`、`bind-address`、`lan-allowed-ips`、`lan-disallowed-ips`（黑名单优先于白名单）、`authentication` 与 `skip-auth-prefixes`。涉及“其他设备能否定上”的问题，必须写这些，而不是只写“开了局域网”。
- 外部控制：监听地址、是否 HTTPS-API、是否 Unix socket 或 Windows namedpipe、是否设置 `secret`。手册两次强调：Unix socket、namedpipe 以及 `external-doh-server` 不会验证 secret。若反馈涉及未授权访问或“没填密钥也能进”，必须写清用的是哪一种入口。
- 数据与指纹：`geodata-mode`、`geodata-loader`、`geo-auto-update`、`geox-url`；若仍使用全局 TLS 指纹，应写明手册已弃用、要求改到 proxy 内的 `client-fingerprint`，避免对方按新语义排查旧写法。

与版本直接相关的一句不要省：若问题涉及本地证书、私钥或 ECH 密钥是否自动重载，写清文件是否本地路径、内核是否达到 v1.19.18。未核验就不要写成“应该会自动重载”。

## 系统与平台必须单独成条

下列限制来自手册，反馈里应写成环境事实，而不是写成“偶发”：

- Android：`disable-keep-alive` 强制为 true；Keep Alive 相关项若被用来讨论耗电，需同时写间隔与 idle 配置。
- Linux：才涉及默认 `routing-mark` 以及 API socket 的 `external-controller-routing-mark`。
- Windows：才涉及 namedpipe 监听及“该通道不校验 secret”。
- 路由器：文档推荐 `find-process-mode: off`；若你在路由上开启 always/strict 后出现异常，应写明当前模式，而不是只写客户端版本。
- 工作目录与安全路径：`external-ui` 等可以为相对或绝对路径；路径不在工作目录时需设置 `SAFE_PATHS`（Windows 分号，其他系统冒号）。界面或证书读失败时，漏写该环境变量会让内核版本讨论跑偏。

## 写不清或对方无法复现时的下一步

先补最小复现集：当前 `mode` 与 `log-level`、问题发生时控制台/控制页面的对应级别日志、外部控制的访问方式（TCP / TLS / socket / pipe）、是否依赖 v1.19.18 的文件重载。不要用“界面和教程不一样”代替客户端名称；菜单差异应另标为外壳，内核仍用字段说话。涉及 API 跨域时写 `external-controller-cors`，不要只说浏览器打不开。若只能提供截图、不能提供键名，先把截图中的现象翻译成手册中的项再发送；翻译失败则注明“未在全局配置中找到对应项”，避免对方按不存在的按钮去找。

https://wiki.metacubex.one/config/general/
