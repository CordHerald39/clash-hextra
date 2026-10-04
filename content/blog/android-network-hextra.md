---
{
  "title": "Clash Meta 手机端（Android）：怎样记录两种网络下同一请求的差异",
  "description": "Clash Meta 手机端（Android）：怎样记录两种网络下同一请求的差异。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T19:24:25.034346+00:00",
  "lastmod": "2026-10-04T19:24:25.034346+00:00",
  "type": "post"
}
---

## 适用条件

本文只说明：在 Android 上使用 Clash Meta 手机端时，怎样记录同一次访问意图在两种承载网络（例如 Wi-Fi 与蜂窝）下的差异。适用条件是设备满足 Clash Meta for Android 的运行要求（最低 Android 5.0，建议 7.0 及以上），应用包名为 `com.github.metacubex.clash.meta`，用户已通过系统 VPN 连接请求对话框授权，并且可以在两种网络之间切换。记录对象是系统与 `VpnService` 能观察到的状态，而不是未在官方材料中出现的应用内菜单或第三方抓包步骤。Clash Meta for Android 建立在 Android VPN API 上，系统只允许每个用户或工作资料同时运行一个活动 VPN 服务。

## 每次切换前要固定下来的对照项

先固定“同一请求”的含义：同一目的主机名或地址、同一应用进程、同一是否走 VPN 的策略，只改变默认承载网络。每次测试前记录时间、网络类型（Wi-Fi 或蜂窝）、以及系统 VPN 是否活动。官方给出的活动连接指示包括：状态栏钥匙图标、快捷设置中的信息面板（点按后有更详细对话框并链到设置）、「设置 > 网络和互联网 > VPN」里该应用是否仍被接受且未被断开或忘记、以及服务活动时不可清除的通知。文档允许该通知展示连接状态或网络统计，因此同一请求前后应记下通知是否存在、其中若含统计量有无变化；通知在服务停止后应被移除，可作为“本轮是否仍在 VPN 会话内”的分界。

同时记录 always-on 是否开启（Android 7.0 起由系统维持服务生命周期），以及是否打开“阻止不使用 VPN 的连接”。后者开启时，未走 VPN 的流量会被系统拦截，设置应用会警告连通前可能没有互联网。这两种系统选项会让同一请求在“失败码、是否完全无连接”上出现差异，必须写入对照表，否则会把策略拦截误记成解析失败。

## 按 API 路径记录接口、DNS、路由与套接字差异

对每一侧网络，按文档中的连接顺序核对并记下结果，而不是只记最终能否打开页面：`VpnService.prepare()` 返回 null 还是授权 Intent（每次都要调用，因当前 VPN 应用可能已被更换）；`VpnService.protect()` 是否在连接网关套接字之前完成；`Builder.addAddress()`、`addRoute()`、`addDnsServer()` 在本次 `establish()` 前写入了哪些值；`establish()` 返回的 `ParcelFileDescriptor` 是否为 null。未准备或权限收回时 `establish()` 为 null，应记为“本侧未建立 TUN”，此时 DNS 与路由属于该侧系统网络，不能与另一侧的 VPN DNS 直接比较。

路由按目的地址过滤，开放路由如 `0.0.0.0/0` 或 `::/0` 表示系统把相应流量送进 VPN 接口。若某一侧未添加匹配路由，同一目的地址会留在系统网络，解析服务器也可能不同，这是必须记录的差异点。分应用允许列表或拒绝列表只能二选一，且必须在连接建立前设置；列表是否包含发起请求的应用，决定该请求走 TUN 还是“如同 VPN 未运行”。`allowBypass()` 以及应用对 `ConnectivityManager.bindProcessToNetwork()`、`Network.bindSocket()` 的绑定，会使同一主机名在两种网络上绑定到不同 `Network`，应记下是否旁路。系统调用 `onRevoke()` 时替代接口已在转发，若某一侧测试中途出现撤销，应记下撤销时刻，避免把两段不同平面的结果拼成一次对比。

## 如何形成可复核的记录，以及失败时下一步

建议为两种网络各留一行相同字段：时间、承载网络、钥匙图标/快捷设置/VPN 设置页/不可清除通知四项是否同时指示活动、always-on 与阻止非 VPN 连接的开关、`prepare()` 与 `establish()` 结果、地址/路由/DNS 是否在本次 Builder 中出现、发起请求的应用是否在允许或拒绝列表中、是否存在 bind 到特定网络、以及请求是完成、被系统拦截还是在撤销后改走替代接口。仓库记载的 `START_CLASH`、`STOP_CLASH`、`TOGGLE_CLASH`（发往 `com.github.kr328.clash.ExternalControlActivity`）可用于在记录前把服务放到已知启停状态，但对比结论仍以系统 VPN 界面与 `establish()` 是否成功为准。

若某一侧无法形成完整记录：先补系统层四项指示，缺任何一项就把该侧标为“VPN 未活动”，只比较系统网络差异。若 `prepare()` 返回授权 Intent，先完成授权再记接口参数。若 always-on 在 Android 8.0 及以上弹出无法连接的不可清除通知，把该事件单独记为系统维持失败，而不是记成应用内“同一请求结果”。不要在缺少 TUN 是否建立的记录时，把两种网络的页面结果直接当成 Clash Meta 路径差异。

资料：
https://developer.android.com/develop/connectivity/vpn
https://github.com/MetaCubeX/ClashMetaForAndroid
