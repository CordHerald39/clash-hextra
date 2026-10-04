---
{
  "title": "Clash 电脑端：怎样用同一网址比较代理设置前后",
  "description": "Clash 电脑端：怎样用同一网址比较代理设置前后。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T18:59:12.558253+00:00",
  "lastmod": "2026-10-04T18:59:12.558253+00:00",
  "type": "post"
}
---

用同一网址比较代理设置前后，目的是把差异归因到“请求有没有进入 Clash 入站”以及“进入后按哪种 mode 处理”，而不是同时改规则、节点、认证和系统代理。官方全局配置提供的对照手段是运行模式、日志级别和外部控制接口。文档没有指定必须使用哪一个检测站点，比较对象可以是你本来就要打开的同一 URL；结论应来自内核记录，而不是页面上的提示文字。

## 比较前要固定的变量

固定入站条件：http(s)/socks/mixed 的地址与端口、authentication、skip-auth-prefixes 在两次访问中保持不变。若比较过程中本机突然需要认证，前后差异可能来自验证失败，而不是代理策略。文档中 skip-auth-prefixes 的示例覆盖本机回环，比较时应保证浏览器源地址始终走同一套认证结果。

固定观察通道：两次访问的 log-level 同为 info 或 debug，才能对照控制台或控制页面输出。silent 无法比较。external-controller 与 secret 保持可访问，避免一次能读 API、另一次只能看页面。profile.store-selected 为 true 时会保存策略组选择；若对照的是 global，要先确认 GLOBAL 组选项没有在两次访问之间被改写。

unified-delay、tcp-concurrent、ipv6 会影响连接建立过程，但不是“代理是否开启”的判据。统一延迟用于计算 RTT、减弱握手带来的延迟观感差异；TCP 并发会对解析到的多个 IP 同时连接并使用最先成功的一条；ipv6 控制内核是否接受 IPv6 流量，默认 true。比较设置前后时这些项应保持不变，以免把握手时序变化误认为代理状态变化。

## 用运行模式做对照

mode 可选 rule、global、direct，默认 rule。对同一网址，第一种对照是：系统代理已指向 Clash 入站时，分别在 direct 与 rule（或 global）下访问。direct 全局直连，rule 按规则，global 走 GLOBAL 组所选代理或策略。若三种 mode 下页面表现和日志都没有差别，应优先怀疑浏览器请求没有进入入站，而不是规则失效。

第二种对照是：保持 mode 不变，只切换系统是否把 HTTP/HTTPS 代理指向该入站。开启后，访问该网址时应在日志或 API 中看到新连接；关闭后，同一网址的请求不应再进入 Clash 入站，浏览器仍可能以直连打开页面。页面能打开只说明目标可达，不能单独证明走了代理。

若比较的是另一台设备上的浏览器，还需要固定 allow-lan、bind-address、lan-allowed-ips 与 lan-disallowed-ips。否则前后差异可能来自绑定地址或黑名单，而不是代理策略。黑名单优先级高于白名单。本机回环比较不要把 allow-lan 当作变量。

## 如何读日志、API 以及失败后的下一步

判断依据必须同时看内核侧：访问同一 URL 时，日志是否新增对应连接或匹配信息；API 读到的 mode 是否为你设定的值；做 global 对照时 GLOBAL 组选择是否一致。只有“进入入站”这一前提成立，比较 rule 与 direct 的差异才有意义。API 监听示例为 127.0.0.1:9090；经 Unix socket 或 Windows namedpipe 访问时不会验证 secret，比较过程中不要把未鉴权通道暴露到不可控范围。

失败时按对照结果收窄。两次都无日志：检查系统代理主机与端口是否指向 http 或 mixed、浏览器是否忽略系统代理、认证是否拒绝。只有某一种 mode 有日志：说明入站正常，应去看规则或 GLOBAL 选择，而不是反复开关系统代理。出现认证错误：先对齐 skip-auth-prefixes 与 authentication。需要更细的握手信息时把 log-level 设为 debug。不要用 unified-delay 或 tcp-concurrent 的开关状态，代替“是否走代理”的结论。

参考资料：https://wiki.metacubex.one/config/general/
