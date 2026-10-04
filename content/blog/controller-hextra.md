---
{
  "title": "Clash 怎样核对面板连接的内核地址",
  "description": "Clash 怎样核对面板连接的内核地址。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T20:32:58.524920+00:00",
  "lastmod": "2026-10-04T20:32:58.524920+00:00",
  "type": "post"
}
---

面板连接的内核地址，以全局配置里的外部控制器为准。文档说明可以使用 RESTful API 控制 Clash 内核，监听项为 external-controller。把面板里填写的主机和端口，与这份配置逐项对照，才能判断连的是不是当前内核。

## 适用条件

适用于已填写 external-controller，并用浏览器、独立面板或静态外部用户界面访问该 API 的部署。文档示例为 127.0.0.1:9090。可以把 127.0.0.1 改成 0.0.0.0 以监听所有 IP；若仍只监听环回地址，其他设备上的面板无法连到该内核。

若实际使用的是 external-controller-unix 或 Windows 的 external-controller-pipe，核对对象是套接字路径或管道名，而不是主机加端口。文档写明从这两种通道访问 API 不会验证 secret，需要自行保证安全。若使用 external-controller-tls，必须同时填写 external-controller，并在 tls 中配置证书与私钥。

## 核对步骤与判断依据

1. 从配置抄写 external-controller 的主机与端口，把它当作内核 API 的权威地址。
2. 核对接面板或外部界面正在请求的主机与端口，必须与上一步一致，包括是否为环回、是否误用了 TLS 端口。
3. 若配置了 external-ui，静态网页资源运行在 Clash API 上，路径为 API 地址加 /ui。界面能打开只说明静态目录或下载地址可用；操作失败时，要看界面请求的 API 是否仍指向 external-controller。external-ui 可以为绝对路径或工作目录相对路径；路径不在工作目录时需按文档设置 SAFE_PATHS。
4. 抄写 secret。经 TCP 或 HTTPS 访问 API 使用该密钥，面板侧填写值必须一致。Unix socket 或 namedpipe 能通而 TCP 不通时，应优先查密钥，而不是假定端口写错。
5. 浏览器跨源访问时，核对 external-controller-cors 的 allow-origins 与 allow-private-network。源不匹配时，拦截发生在浏览器，不能当成内核未监听。
6. 走 HTTPS 时，核对 external-controller-tls 的主机端口以及 tls.certificate、tls.private-key（PEM 或路径）。文档说明使用 TLS 也必须填写 external-controller。自 v1.19.18 起，证书或私钥为本地文件时支持自动重载，但不等于可以省略监听地址。

判断依据：面板请求的主机、端口、是否 TLS，与配置中对应监听项一致，并且 secret 匹配，或走的是文档写明不验证 secret 的套接字或管道。全部一致才能认为面板连的是当前内核。

## 失败时下一步

连不上时按监听范围、密钥、跨源、TLS 分开查，不要改代理入站来代替。allow-lan 与 bind-address 作用于代理端口，不是 API 地址核对项。先确认进程加载的就是当前这份 external-controller。能打开 /ui 但不能调用 API，把 external-ui、external-ui-name、external-ui-url 与 API 主机端口分列。external-doh-server 开在 RESTful API 端口上且不验证 secret，不能代替面板的 API 地址核对。Linux 的 external-controller-routing-mark 只为监听 socket 设置标记，不改变端口号。日志只在控制台和控制页面按 log-level 输出，有日志不等于面板连对了地址。

资料：https://wiki.metacubex.one/config/general/
