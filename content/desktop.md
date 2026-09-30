---
{
  "title": "Clash 电脑版下载与 Windows 使用教程",
  "description": "核对 Windows x64 与 ARM64 安装包，导入订阅并检查系统代理。",
  "date": "2026-09-30T10:00:00+08:00",
  "lastmod": "2026-09-30T10:00:00+08:00",
  "type": "docs",
  "weight": 3
}
---

Windows 用户可以从 [Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev/releases)开始。打开系统“关于”查看架构，选择对应Windows附件。Mac和Linux用户也应在发行列表里分别核对平台标记。

## 安装程序与依赖

下载前阅读[项目安装文档](https://www.clashverge.dev/install.html)。EXE可能是安装器或主程序，按附件说明判断。便携包完整解压，源码包用于构建。[架构选择文章]({{< relref "blog/windows" >}})解释常见文件名。

## 导入并启用配置

打开配置或订阅界面，添加服务方兼容地址，更新后启用。选节点并开启系统代理，浏览器访问目标的同时查看连接记录。系统代理只影响遵循它的应用；TUN按文档单独设置。

## 退出后的网络

先关闭系统代理再退出。异常退出后若无法联网，检查Windows代理页面是否遗留已停止的本地地址。

继续阅读[规则与接管]({{< relref "blog/modes" >}})和[升级备份]({{< relref "blog/backup" >}})。
