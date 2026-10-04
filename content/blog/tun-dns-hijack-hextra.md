---
{
  "title": "Clash 怎样区分域名解析失败与目标连接失败",
  "description": "Clash 怎样区分域名解析失败与目标连接失败。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T19:24:25.027673+00:00",
  "lastmod": "2026-10-04T19:24:25.027673+00:00",
  "type": "post"
}
---

TUN 文档没有单独列出日志关键字，但给出了 dns-hijack、auto-route、strict-route 以及平台例外。区分“域名解析失败”与“目标连接失败”，可以依据这些字段是否把 53 端口导入内部 DNS，以及 IP 直连是否仍然成功来判断。

## 适用条件
当 tun.enable 为 true 且 auto-route 打开时，流量（含 DNS）进入 TUN。dns-hijack 将匹配项导入内部模块；未写协议视为 UDP。若劫持因平台限制未发生——MacOS/Windows 不能自动劫持局域网 DNS，Android 私人 DNS 不能劫持——域名会继续由系统解析。此时若系统解析器已被全局路由切断，就会表现为解析失败。反之，若劫持已生效而目标 IP 仍不通，则更接近连接或协议栈问题。strict-route 用于收紧路径并在 Android 上使劫持工作、在 Windows 上抑制多宿主泄漏，因此它同时影响解析路径是否完整。

## 判断步骤与依据
第一步用同一目标分别测试域名与字面 IP。IP 可通而域名不通，说明 mtu、device、stack 和出站接口大体可用，问题落在解析：检查 dns-hijack 是否包含 any:53 与 tcp://any:53。第二步看 auto-route 与 strict-route 是否按文档组合；缺少前者则 DNS 包到不了 TUN，缺少后者则 Android/Windows 上可能泄漏或劫持无效。第三步核对其余过滤：route-exclude-address、exclude-interface、exclude-uid、exclude-package 是否把 DNS 服务器或发起进程排除。第四步对照平台例外，确认不是局域网 DNS 或私人 DNS 导致系统解析。连接失败的判断依据则是：域名与 IP 均失败，或仅特定协议（TCP/UDP）失败，此时应转向 stack 选择（防火墙下 system/mixed 不可用）、Linux auto-redirect、以及文档给出的防火墙放行方式。udp-timeout 过短可能影响 UDP DNS 会话，可与解析失败交叉验证。

## 失败后的下一步
若判断为解析失败，按文档补全 dns-hijack，打开 strict-route，关闭 Android 私人 DNS，并避免依赖无法劫持的局域网 DNS。若判断为连接失败，检查 Windows 是否允许内核通过防火墙、Linux 是否对 TUN 网卡出站放行，MacOS 网卡名是否为 utun 开头。可暂时把 enable 设为 false，确认系统本身域名与 IP 均正常，再只启用 auto-route 测试 IP，最后加回 dns-hijack 测试域名。旧 inet6-route-exclude-address 等废弃写法若残留，可能造成 IPv6 DNS 或连接被错误排除，应改为现行 route-exclude-address。完成字段对照后仍无法区分，再检查 auto-detect-interface 是否选错出口，导致解析结果或回程连接失败。

https://wiki.metacubex.one/config/inbound/tun/
