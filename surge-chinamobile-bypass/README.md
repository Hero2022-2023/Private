# 中国移动直连绕过 Surge 模块

让中国移动 App 在 Surge 开启时能正常打开（2026-11-12 真机已验证）。

## 结论（已结案）

**有效的是仅代理版 `ChinaMobile-Bypass-ProxyOnly.sgmodule`（compatibility-mode=2）。**

判别测试结果：

| 模式 | 结果 | 说明 |
|---|---|---|
| 规则直连（v1.0.0，默认模式 3） | ✗ 闪退 | 抓包证实直连/真实 IP/真证书都已生效，App 仍在首个请求后自退 |
| compatibility-mode=5（主模块 v1.1.x） | ✗ 闪退 | VIF 不设默认路由也没用，文件已标注勿启用 |
| **compatibility-mode=2（ProxyOnly v1.1.0）** | **✓ 能打开** | **VIF 完全不接管流量后通过；它查的是 VIF 接管，不是 VPN 是否存在（此模式下 VPN 描述文件仍连接、状态栏图标仍在），也不是系统代理设置（此模式下代理设置照样存在）** |

安装（保持长期启用）：

https://raw.githubusercontent.com/Hero2022-2023/Private/main/surge-chinamobile-bypass/ChinaMobile-Bypass-ProxyOnly.sgmodule

Surge → 修改 → 模块 → 安装新模块，粘贴启用，**彻底停止再启动 Surge**。模式 5 的主模块请删除或保持停用，两个不要同时启用。

## 代价（compatibility-mode=2 是全局模式）

- 仅代理接管意味着：不遵循 iOS 系统代理的 App 不会被 Surge 接管，其流量直走物理网络、不经节点（部分游戏、自研网络栈 App、UDP/QUIC 流量常见）。
- 遵循系统代理的 App（浏览器、大多数主流 App）照常按 Surge 规则与节点走，不受影响。
- 建议启用后抽查自己最常用的几个走代理的 App（如 X、Spotify、浏览器查出口 IP）；若某个 App 因此不走代理了且不可接受，备选是回到默认模式 + 快捷指令自动化（打开中国移动时 surge:///stop、关闭时 surge:///start?autoclose=true）。

## 直连规则部分（两个模块相同）

覆盖域名：

- `chinamobile.com`（用户指定，含 App 启动先连的 `*.moazq.chinamobile.com`）
- `10086.cn`（中国移动 App 主要接口，如 `client.app.coc.10086.cn`）
- `cmpassport.com`（中国移动通行证 / 一键登录）
- `139.com`（中国移动相关服务）

做法：`[Rule]` 置顶全 `DIRECT`；`[General]` 用 `%APPEND%` 追加 `skip-proxy` 与 `always-real-ip`（真实 IP，不给 Fake-IP）；`[MITM]` 用 `-域名` 排除解密，避免证书绑定失败。未加 `tun-excluded-routes`（只能按 IP 段写、移动段分散、大段排除在 iOS 上有内存问题；且模式 2 下 VIF 不接管，此项无意义）。

## 抓包证据摘要（2026-11-12）

模块已生效（移动域名解析到真实 IP 而非 198.18 Fake-IP、TLS 握手完整、收到 SHECA 签发的 `*.moazq.chinamobile.com` 真实证书链、无 MITM 干扰、服务器未要求客户端证书），App 向 `zj.moazq.chinamobile.com`（111.22.67.169:443）发出首个加密请求后 25ms、服务器未回复，客户端即 RST 断开，始终未连 10086.cn 主接口。证书路线已排除：新制证书骗不过本机检测，Surge CA 现签证书冒充会撞证书绑定。
