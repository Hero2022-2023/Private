# 中国移动直连绕过 Surge 模块

让 `*.chinamobile.com` 及中国移动 App 常用域名在 Surge 开启时强制直连，缓解 App 检测代理 / 出口 IP 后闪退或登录失败。

覆盖域名：

- `chinamobile.com`（用户指定）
- `10086.cn`（中国移动 App 主要接口，如 `client.app.coc.10086.cn`）
- `cmpassport.com`（中国移动通行证 / 一键登录）
- `139.com`（中国移动相关服务）

做法：

- `[Rule]` 规则置顶，全部 `DIRECT`（模块规则只能使用 DIRECT 等内置策略）
- `[General]` 用 `%APPEND%` 追加 `skip-proxy` 与 `always-real-ip`，返回真实 IP，不给 Fake-IP
- `[MITM]` 用 `-域名` 排除解密，避免证书绑定（Certificate Pinning）失败

## 两个版本（只能启用一个）

| 文件 | compatibility-mode | 说明 |
|---|---|---|
| `ChinaMobile-Bypass.sgmodule`（v1.1.0） | 5 | VIF 不声明默认路由、改用多条小路由接管；按「默认路由是否为 VPN」判断的 App 在此模式下会认为 VPN 未开启（官方以此绕 HomeKit 摄像头等兼容问题）。先试这个 |
| `ChinaMobile-Bypass-ProxyOnly.sgmodule`（v1.0.0） | 2 | 仅代理接管，VIF 不接管任何流量；不遵循系统代理的 App 不会被 Surge 接管。主版本无效时再试这个 |

安装：Surge → 修改 → 模块 → 安装新模块，粘贴地址。**切换/启用模块后必须彻底停止再启动 Surge**，工作模式才生效。

- 主版本：https://raw.githubusercontent.com/Hero2022-2023/Private/main/surge-chinamobile-bypass/ChinaMobile-Bypass.sgmodule
- 仅代理版：https://raw.githubusercontent.com/Hero2022-2023/Private/main/surge-chinamobile-bypass/ChinaMobile-Bypass-ProxyOnly.sgmodule

## 实测记录

- 2026-11-12 v1.0.0 真机测试仍闪退。抓包分析：模块已生效（移动域名解析到真实 IP、TLS 握手完整、收到的是 SHECA 签发的真实证书、无 MITM 干扰），App 向 `zj.moazq.chinamobile.com` 发出首个加密请求后 25ms 内、服务器未回复即由客户端 RST 断开，未连 10086.cn 主接口，判定为本机检测后自退。
- 因此 v1.1.0 起改走 compatibility-mode 方向做判别测试：模式 5 能过 = 它查的是默认路由；模式 2 能过 = 它查的是 VIF 路由；两个都不过 = 它查的是 VPN 描述文件 / utun 接口是否存在本身，Surge 内无解，只能用快捷指令在打开该 App 时自动停 Surge、关闭时自动启。

注意：

- 规则类设置必须在「规则模式」下使用；全局代理模式下规则不生效（compatibility-mode 不受此限）。
- 未加 `tun-excluded-routes`：该项只能按 IP 段写，移动 IP 段分散，大段排除在 iOS 上有已知内存问题；拿到具体服务器 IP 后再按 /32 补。
