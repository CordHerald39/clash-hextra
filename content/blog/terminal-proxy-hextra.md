---
{
  "title": "Clash 电脑端：怎样记录单条命令是否读取代理变量",
  "description": "Clash 电脑端：怎样记录单条命令是否读取代理变量。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T21:37:45.961184+00:00",
  "lastmod": "2026-10-04T21:37:45.961184+00:00",
  "type": "post"
}
---

## 记录目标与适用条件

要记录单条命令是否读取代理变量，适用对象是当前终端会话中即将执行的那一条命令，而不是整个图形客户端。全局配置没有名为“记录环境变量读取”的开关。文档能提供的旁证是：入站为 http(s)/socks/mixed，可启用 authentication；skip-auth-prefixes 可让 127.0.0.1/8 与 ::1/128 跳过验证；bind-address 与 allow-lan 决定命令所连主机是否有效；log-level 决定控制台能看到多少连接信息；find-process-mode 决定是否匹配进程。记录应写成可核对的事实：命令启动前会话里有哪些变量、运行期间内核是否出现对应入站、认证是否成功、mode 是否把流量按直连处理。不要把 SAFE_PATHS 的有无写成“已经读取代理变量”；该变量只约束外部用户界面路径，语法同 PATH，Windows 用分号，其他系统用冒号。

## 单条命令的观察步骤

第一步，仅在将要执行该命令的同一会话中打印环境，保存一份文本快照，内容包括是否存在指向入站主机的代理类变量。第二步，读取当前配置中的 bind-address、authentication、skip-auth-prefixes、allow-lan，把快照里的主机、是否含 user:pass 与这些字段对照，预先写下“若命令读取了这些变量，应当连到何处”。第三步，将 log-level 设为 debug 或 info，避免 silent。第四步，只执行那一条命令，不要同时打开浏览器或其他会占用同一入站的操作，以便时间戳能对应到这一条。第五步，在控制台查找该时间窗口内是否出现连入当前绑定地址的记录。若有连接且源地址为回环、配置了 authentication 但未带密码，可依据 skip-auth-prefixes 判断免密是否符合文档；若主机不在 bind-address 范围，则即使命令读取了变量，内核也不会在该地址接受。第六步，结合 mode：direct 时即使读取了变量并连上入站，表现仍可能像直连，记录中应单独注明 mode。find-process-mode 为 always 时可增加进程匹配信息，为 off 时不要把“没有进程名”理解成“没有读变量”。

判断“读了变量”不能只看命令成功，必须同时具备会话快照与入站日志两份依据。只有成功结果、没有对照配置，不能作为记录结论。

## 无法判断时的下一步

若日志完全没有该命令的入站记录，存在两种不能互相替代的解释：命令未读取代理变量而走了直连；或读取了变量但主机指向未监听地址。此时不要改 tcp-concurrent、unified-delay、keep-alive-idle 等无关项。应先确认 bind-address 与 ipv6，去掉快照中指向未绑定 IPv6 或未绑定网卡的值后，仅对这一条命令再观察一次。若出现认证错误，说明命令尝试了入站但凭证与 authentication 不一致，这与“完全没读变量”不同，记录时应分开写。不要把 external-controller、external-doh-server、Unix socket、namedpipe 的访问记成命令走了代理；这些路径不验证或另用 secret，用途是 API。profile 里 store-selected 与 store-fake-ip 只保存策略选择和 fakeip 映射，不能证明单条命令读过变量。若仍要缩小范围，保持 mode、allow-lan、authentication 不变，只改变该会话变量后再跑同一条命令，用两次日志对比，作为是否读取变量的判断依据。

https://wiki.metacubex.one/config/general/
