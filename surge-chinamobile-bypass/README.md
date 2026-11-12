# 中国移动直连绕过 Surge 模块

让中国移动 App 在 Surge 开启时能正常打开（已穷尽 Surge 内方案，最终结论见下）。

## 最终结论（2026-11-12，真机实测穷尽）

| 方案 | 结果 |
|---|---|
| 规则直连（默认模式 3）：DIRECT + 真实 IP + 真证书透传 | ✗ 闪退（抓包证实网络侧全部正常，App 发出首个请求后 25ms 内自退） |
| compatibility-mode=5（VIF 不设默认路由） | ✗ 仍无法打开 |
| compatibility-mode=2 写在模块里（ProxyOnly） | ✗ 仍无法打开 |
| **compatibility-mode=2 写进配置本体（Test-Mode2.conf）** | **✗ 仍无法打开（2026-11-12 20:46 用户确认）** |

即：只要 Surge 的 VPN 处于连接状态（哪怕不接管任何流量、只剩代理设置），该 App 就拒绝运行。它查的是 VPN/代理存在本身，Surge 配置层面无解，证书路线也已排除（见下方抓包摘要）。

**终点方案：快捷指令自动化**——打开中国移动时自动停 Surge、关闭时自动启：

1. 快捷指令 → 自动化 → 新建 → App → 选「中国移动」→「已打开」+「立即运行」
   动作：「打开 URL」填 `surge:///stop?autoclose=true`，其后再加「打开 App」选中国移动（跳回 App 本身）
2. 再建一条：触发改「已关闭」，动作「打开 URL」填 `surge:///start?autoclose=true`

收尾：切回主配置 Surge-Full.conf（Test-Mode2.conf 是静态快照、节点不更新，勿长期使用），两个 ChinaMobile 模块与测试配置可删。

## 抓包证据摘要（2026-11-12）

模块已生效（移动域名解析到真实 IP 而非 198.18 Fake-IP、TLS 握手完整、收到 SHECA 签发的 `*.moazq.chinamobile.com` 真实证书链、无 MITM 干扰、服务器未要求客户端证书），App 向 `zj.moazq.chinamobile.com`（111.22.67.169:443）发出首个加密请求后 25ms、服务器未回复，客户端即 RST 断开，始终未连 10086.cn 主接口。证书路线已排除：新制证书骗不过本机检测，Surge CA 现签证书冒充会撞证书绑定。

## 模块内容存档

覆盖域名：`chinamobile.com`、`10086.cn`、`cmpassport.com`、`139.com`。做法：`[Rule]` 置顶全 `DIRECT`；`[General]` 用 `%APPEND%` 追加 `skip-proxy` 与 `always-real-ip`；`[MITM]` 用 `-域名` 排除解密。这些规则本身无错（对付查出口 IP 的 App 有效），只是对付不了本 App 的本机存在性检测。
