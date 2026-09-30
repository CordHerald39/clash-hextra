---
{
  "title": "Windows 安装包该选 x64 还是 ARM64",
  "description": "从系统类型、文件后缀到安装组件，逐项核对 Clash Verge Rev 下载包。",
  "date": "2026-09-30T10:00:00+08:00",
  "lastmod": "2026-09-30T10:00:00+08:00",
  "type": "blog",
  "author": "Clash 编辑组",
  "tags": [
    "Clash",
    "使用教程"
  ]
}
---

打开 Windows“设置 → 系统 → 关于”，查看系统类型，然后进入 [Clash Verge Rev 发行页](https://github.com/clash-verge-rev/clash-verge-rev/releases)。x64 与 ARM64 是架构标记，Windows 版本名称本身不能替你决定架构。

## 看清附件名称

| 标记 | 说明 |
| --- | --- |
| x64、amd64 | 64位 x86 环境常见标记 |
| arm64、aarch64 | 64位 ARM 架构，仍需核对文件面向 Windows |
| setup.exe、.msi | 常见安装器形式，以发行说明为准 |
| Source code | 源码归档 |

同一发行里的 ARM64 文件可能对应不同操作系统。.dmg 是 macOS 格式，不能因为架构相同就下载到 Windows。便携压缩包应完整解压，保留主程序旁的资源和辅助文件。

## 缺少组件时怎么处理

Clash Verge Rev 安装文档介绍了带 fix_webview2 标记的包，用于需要内置 WebView2 环境的情况。先核对系统与项目要求，再按开发者说明处理依赖。遇到程序打不开，保存错误原文比反复换客户端更有用。

## 安装后的检查

打开关于页记下实际版本，与刚下载的版本核对。导入配置并选中节点后，先开启系统代理，用浏览器请求和连接记录验证。需要 TUN 时再按文档设置服务与权限，避免第一次就同时启用多种接管路径。

退出时先关闭系统代理。若强制结束进程后网络异常，在 Windows 代理设置中检查是否留下了已经停止的本地端口。

资料：[开发者安装说明](https://www.clashverge.dev/install.html)、[快速入门](https://www.clashverge.dev/guide/quickstart.html)。更多入口见[电脑版下载]({{< relref "desktop" >}})。
