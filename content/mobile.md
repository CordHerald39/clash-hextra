---
{
  "title": "Clash 手机版下载与安卓配置",
  "description": "选择 Android APK，导入 CMFA 配置，完成 VPN 授权与首次连接。",
  "date": "2026-09-30T10:00:00+08:00",
  "lastmod": "2026-09-30T10:00:00+08:00",
  "type": "docs",
  "weight": 2
}
---

Android 用户可选择 [CMFA](https://github.com/MetaCubeX/ClashMetaForAndroid/releases)或[FlClash](https://github.com/chen08209/FlClash/releases)。先查看发行附件支持的 ABI，arm64-v8a、armeabi-v7a 与 x86_64 面向不同运行环境。

## 手机版本与安装权限

查设备系统信息和厂商说明，再选包。允许当前浏览器安装APK的权限与VPN授权分别出现，安装完成后可收回安装权限。iPhone使用不同应用分发方式，不能安装APK。

## CMFA 的首次使用

在配置页面从URL导入兼容地址，保存并激活，然后启动服务、确认系统VPN请求。运行后在代理页面选模式和节点，发起一次目标请求并看日志。完整过程见[安卓首次连接]({{< relref "blog/android" >}})。

## 部分应用不通

先查应用访问控制和连接记录，再看规则是否命中DIRECT。另一款VPN可能影响当前连接。有关系统授权与服务限制可查[Android官方VPN说明](https://developer.android.com/develop/connectivity/vpn)。

[返回下载目录]({{< relref "downloads" >}}) · [订阅排错]({{< relref "blog/subscription" >}})
