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

安装：Surge → 修改 → 模块 → 安装新模块，粘贴永久地址：

https://raw.githubusercontent.com/Hero2022-2023/Private/main/surge-chinamobile-bypass/ChinaMobile-Bypass.sgmodule

注意：

- 必须在「规则模式」下使用；全局代理模式下规则不生效。
- 若 App 检测的是本机 utun VPN 接口本身（把 Surge 切全局直连仍闪退），任何规则都藏不掉，只能在用它时暂停 Surge，或用快捷指令自动化在打开该 App 时自动停 Surge、关闭时自动启。
- 未加 `tun-excluded-routes`：该项只能按 IP 段写，移动 IP 段分散，大段排除在 iOS 上有已知内存问题；拿到具体服务器 IP 后再按 /32 补。
- 未在 iOS 真机实测，效果以真机为准。
