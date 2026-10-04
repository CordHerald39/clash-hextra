---
{
  "title": "Clash 电脑端：怎样核对 Git 配置的作用范围",
  "description": "Clash 电脑端：怎样核对 Git 配置的作用范围。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T21:37:45.964194+00:00",
  "lastmod": "2026-10-04T21:37:45.964194+00:00",
  "type": "post"
}
---

## 适用条件

让 Git 经 Clash 本机入站前，或排查拉取异常时，需要核对代理相关键到底写在哪一级、对哪些仓库生效。Git 的系统级、全局、仓库本地配置互相覆盖，和 Clash 全局配置里的 mode、入站验证不是同一套作用域。本文只说明如何核对这些范围。

适用条件：本机已安装 Git；你关心 `http.proxy`、`https.proxy` 或 URL 改写项会不会作用于当前仓库；计划连接的是 http、https、socks 或 mixed 入站。SSH URL 不受这些 HTTP 代理键范围影响。核对范围不能改写 skip-auth-prefixes，也不能代替看内核日志。核对应以当前仓库目录为基准重复列出；换目录后 local 范围会变化，全局范围通常不变，这一差异本身就是判断依据。

## 用来源和范围同时列出

在出问题的仓库目录执行 `git config --list --show-origin --show-scope`。判断依据有两个。来源路径告诉你文件位置：系统级、用户全局、当前仓库。范围字段告诉你该键属于 system、global 还是 local。只做一次不带来源的读取，只能得到最终值，看不到是哪一级覆盖了其他级。

分别检查 `http.proxy`、`https.proxy`、带 URL 的更细代理键，以及 `url.*.insteadof`。判断冲突的依据是：同一逻辑键出现多行，且来源不是同一个文件。较窄范围覆盖较宽范围，仓库本地会盖掉用户全局。若当前目录不是仓库，local 不存在，此时只能看到全局与系统级，不能用这次输出证明某个仓库内部没有覆盖。列出结果应保存后再改文件，避免改完后无法对照原先范围。

## 把范围结论映射到入站行为

核对完成后，用最终生效的代理值对照 Clash 入站。若值为空，当前仓库的 Git 不会因为其他仓库的 local 配置而走入站，也不会因为内核正在运行就自动连接。若值指向本机，再看 authentication：需要验证时，该范围下的值必须带上可用凭据，除非连接源地址属于 skip-auth-prefixes 默认的 `127.0.0.1/8` 与 `::1/128`。写成非回环地址时，还要看 allow-lan、bind-address、lan-allowed-ips 与 lan-disallowed-ips，这些是入站访问范围，不是 Git 的 system 或 global。

mode 的作用范围是内核内部，对所有已经进入入站的连接生效，不会按 Git 的 system 或 global 再分一套。不要把 Git 的全局配置理解成 Clash 的 global 模式。find-process-mode 也不随 Git 作用范围变化：always、strict、off 只决定内核是否匹配进程。log-level 同样是内核输出范围，silent 时不要把没日志当成 Git 没走该范围的代理。

## 核对失败时的下一步

若带来源的列表显示已删除，但不带来源的读取仍有值，说明还有另一文件在提供该键，应继续列出直到每一来源都解释完毕。若在错误目录执行命令，local 范围会缺失或指向无关仓库，应先进入目标仓库再核。只改仓库内文件却看不到变化时，按来源路径去打开实际生效的那一层，而不是在 local 重复操作。

若范围已经清楚，Git 仍未访问入站，把 log-level 设为 info 或 debug 看有无连接；没有连接则是客户端没使用该代理值，或远程不是 HTTPS。有连接则再看 mode 是 rule、global 还是 direct。不要用改外部用户界面、geodata-mode 或 ETag 支持来刷新 Git 作用范围。ipv6 开关同样不改变 Git 配置写在哪一级。external-controller 与 secret 只约束接口，不能用来查询 Git 文件范围。确认范围后再决定删除或修改哪一个文件。

参考资料：
https://wiki.metacubex.one/config/general/
