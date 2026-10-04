---
{
  "title": "Clash Meta 手机端（Android）：怎样记录当前 VPN 应用和授权状态",
  "description": "Clash Meta 手机端（Android）：怎样记录当前 VPN 应用和授权状态。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T19:24:25.031165+00:00",
  "lastmod": "2026-10-04T19:24:25.031165+00:00",
  "type": "post"
}
---

排查 Clash Meta for Android 与其他 VPN 的关系时，需要把“当前哪个应用是系统认定的 VPN”以及“该应用是否仍持有用户授权”记录下来，否则后续无法对比启动前后的变化。记录应依据系统提供的设置界面与 prepare 机制，而不是应用内自定义状态。

## 适用条件与需要记录的对象

适用条件：设备运行 Android 且已安装至少一个声明了 VpnService 的应用（含 Clash Meta for Android）。每个用户或工作资料只有一个活动服务，也只有一个当前 prepared 应用。需要记录的对象包括：已接受过连接请求的应用列表、当前是否显示活动图标/面板、Always-on 是否开启、以及是否启用阻止非 VPN 连接。

系统在应用首次活动前显示连接请求对话框。Settings > Network & Internet > VPN 列出已接受请求的应用，并提供配置系统选项或忘记该 VPN 的入口。快捷设置托盘在连接活动时显示信息面板。状态栏用钥匙图标表示有活动连接。这些都是可在当时截取或抄写的官方界面事实。Clash Meta 的包名为 com.github.metacubex.clash.meta，外部控制 Activity 为 com.github.kr328.clash.ExternalControlActivity，记录包名有助于以后用 Intent 对照。

## 按顺序采集授权与活动状态

第一步记录状态栏是否有 VPN 钥匙图标，以及快捷设置中 VPN 面板点击后对话框指向的应用名称。第二步打开 VPN 设置屏幕，逐项记下列出的应用、是否能忘记、Always-on 开关状态（Android 7.0+），以及“阻止不使用 VPN 的连接”是否打开。Android 8.0+ 在 Always-on 断开时还有不可关闭通知，也应记下是否出现及指向谁。

第三步理解授权与 prepared 的区别。prepare() 在未授权时返回启动系统对话框的 intent，已准备则返回 null。列表中出现某应用只说明曾经接受过请求，不保证它现在仍是唯一 prepared 者。若要对 Clash Meta 重新确认，应在其服务启动流程中再次 prepare，因为用户可能已改选其他应用。第四步如需留下可复现操作，可记录是否使用过 START_CLASH / STOP_CLASH / TOGGLE_CLASH 这些仓库公开的 action，但 Intent 本身不能替代设置屏幕上的授权记录。

## 记录是否完整及失败时的下一步

记录完整的判断依据：能对应到设置里的应用列表、钥匙图标有无、快捷设置对话框内容、Always-on 与拦截开关、以及 Clash Meta 包名。工作资料与个人用户要分开记，因为两者可运行不同 VPN 应用。per-app 允许/禁止列表属于接口建立时的 Builder 配置，系统设置屏幕不一定逐项显示包名，若 Clash Meta 使用了列表，需在其自身配置侧另行注明，否则记录会遗漏“并非全设备流量进入 TUN”。

若设置中看不到预期应用，可能尚未接受过连接请求、权限已用“忘记”清除，或服务未用 BIND_VPN_SERVICE 与 android.net.VpnService 过滤器正确声明。下一步：不要反复猜测菜单，而是再次触发会调用 prepare() 的启动流程，看是否弹出系统对话框；弹出则把“未授权”记下来。establish() 返回 null 时应记录为未取得接口。完成书面记录后再做启动或停止，才能把后续变化与这份基线对比。

https://developer.android.com/develop/connectivity/vpn
https://github.com/MetaCubeX/ClashMetaForAndroid
