---
{
  "title": "Clash 怎样记录浏览器协议而不随意改全局设置",
  "description": "Clash 怎样记录浏览器协议而不随意改全局设置。了解适用条件、操作步骤与常见问题的排查方法。",
  "date": "2026-10-04T21:03:35.930396+00:00",
  "lastmod": "2026-10-04T21:03:35.930396+00:00",
  "type": "post"
}
---

要记录浏览器走哪类连接，优先用日志与外部控制接口做观察，而不是改运行模式、IPv6、出站网卡或 GEO。官方全局配置把日志级别、API 监听、策略选择缓存和 fakeip 缓存写在同一页。能只读观察的，就不要改会改变流量路径的项。

## 适用条件

适用于需要留下一次浏览器访问的日志或策略选择记录，且当前代理已能工作的场景。`mode` 保持原值：`rule`、`global` 或 `direct` 会彻底改变是否走规则。为了「多看一点信息」去改模式，记录到的已不是原来的协议路径。

`log-level` 只影响控制台与控制页面输出：`silent` 不输出，`error`/`warning`/`info`/`debug` 逐级变细。把级别调到 `debug` 是资料允许的观测手段，它不改变入站范围，也不改变是否接受 IPv6。相比之下，改 `allow-lan`、`bind-address`、`tcp-concurrent` 或 `find-process-mode` 都会改变匹配或连接行为，不属于「只记录」。

## 具体操作

1. 只调整观测口径：将 `log-level` 设为 `debug`，其余全局项保持文件原值。记录开始与结束时间，便于和浏览器操作对应。观察结束后若需减少输出，再改回原来的 `info` 或 `warning`，不要顺手改别的键。
2. 用已有外部控制做只读查看。资料中 API 监听示例为 `external-controller: 127.0.0.1:9090`，并用 `secret` 作为访问密钥。不要为了记录协议把监听改成 `0.0.0.0`，那会扩大控制面。Unix socket 与 Windows namedpipe 访问 API 不会验证 secret，资料要求自行保证安全，记录浏览器行为时没有必要新开它们。同样，RESTful API 上的 DOH（`external-doh-server`）也不会验证 secret，不应为记录协议而打开。
3. 打开缓存中与「可复现」有关、但不改路径的项：`profile.store-selected` 为 true 时储存 API 对策略组的选择，供下次启动使用；`store-fake-ip` 为 true 时储存 fakeip 映射。它们记录的是选择与映射，不是改规则。对照期间不要用 API 改组，以免记录被新的选择覆盖。
4. 不要动这些项来「帮忙记录」：`ipv6`、`keep-alive-interval`、`keep-alive-idle`、`disable-keep-alive`、`tcp-concurrent`、`unified-delay`、`interface-name`、`routing-mark`、`find-process-mode`。进程匹配从 `strict` 改到 `always` 会强制匹配所有进程，记录对象已经变了；路由器上资料推荐 `off`，更不应为记录临时打开。
5. 全局 TLS 指纹已弃用，资料要求在 proxy 内设置 `client-fingerprint`。不要在全局层加指纹来标记浏览器协议，那既不是记录方法，也不再是本页推荐做法。

判断依据：操作前后，除 `log-level` 外的全局键未变，且 API 监听地址与 `secret` 未变，则日志与 `store-selected` 记录的是原条件。若同时改了模式或进程匹配，记录不能代表原浏览器路径。

示例中仅展示观测相关键，其他键保持你文件中的原值：

```yaml
log-level: debug
external-controller: 127.0.0.1:9090
secret: ""
profile:
  store-selected: true
  store-fake-ip: true
```

## 失败时下一步

若 `debug` 仍看不到与浏览器相关的内容：先确认不是 `silent` 被别处覆盖，再确认控制页面连的是当前 `external-controller`，而不是旧地址。不要因此改 `mode` 或打开 `allow-lan`。

若 API 无法访问，检查是否误开了必须配置证书的 `external-controller-tls` 却未准备 TLS 段。资料写明 TLS 目前仅用于 API 的 https，且使用 TLS 时仍必须填写 `external-controller`。记录协议不需要先上 HTTPS-API。

外部 UI 只是把静态网页挂到 API 的 `/ui` 路径，路径可以是绝对路径或工作目录相对路径；路径不在工作目录时需设置 `SAFE_PATHS`。装界面不是记录浏览器协议的前提。仍不足时，维持现有全局项，仅保留 `debug` 与策略选择缓存，避免把 GEO 自动更新、出站接口或 TCP 并发加进这次记录。

https://wiki.metacubex.one/config/general/
