# 中国移动直连绕过 Surge 模块

让中国移动 App 在 Surge 开启时能正常打开（进行中，未结案）。

## 状态更正（2026-11-12 20:40）

此前 README 标注的「模式 2 已验证有效、本案已结案」是基于用户误点选项的误报，**已撤回**。用户实际反馈：用更新后的模块仍无法打开。当前真实状态：

| 方案 | 状态 |
|---|---|
| 规则直连（v1.0.0，默认模式 3） | ✗ 已试，闪退 |
| compatibility-mode=5（主模块） | ✗ 已试，仍无法打开 |
| compatibility-mode=2 写在模块里（ProxyOnly） | ？未确认——且模块覆盖这个标量设置是否真的生效本身存疑 |
| **compatibility-mode=2 写进配置本体的 `Test-Mode2.conf`** | **待测（决定性复核）** |

## 决定性复核：Test-Mode2.conf

`Test-Mode2.conf` = 当前托管主配置 Surge-Full.conf 的快照（67 个测通节点），只改三处：`[General]` 加 `compatibility-mode = 2` 和四个域名的 `skip-proxy` / `always-real-ip`；`[Rule]` 在 FINAL 前加四条 DIRECT。静态快照、节点不自动更新，只用于这一次测试。

安装：https://raw.githubusercontent.com/Hero2022-2023/Private/main/surge-chinamobile-bypass/Test-Mode2.conf

步骤：

1. 在 Surge 里**停用两个 ChinaMobile 模块**（主模块和 ProxyOnly 都停）
2. 配置列表 → + → 从 URL 下载配置，粘贴上面地址，切换到这个测试配置
3. **彻底停止再启动 Surge**
4. 打开中国移动 App

判读：

- **能打开** → 模式 2 本身有效，之前是模块没把模式切过去；正式方案改为把 compatibility-mode=2 写进主配置生成流程，模块只留直连规则。
- **还是不行** → 模式 2 也救不了，它查的是 VPN 描述文件/代理存在本身；Surge 内无解，终点方案为快捷指令自动化（打开该 App 时 surge:///stop、关闭时 surge:///start?autoclose=true）。
- 测完切回原来的 Surge-Full.conf 配置即可。

## 直连规则部分（各模块相同）

覆盖域名：

- `chinamobile.com`（含 App 启动先连的 `*.moazq.chinamobile.com`）
- `10086.cn`（中国移动 App 主要接口，如 `client.app.coc.10086.cn`）
- `cmpassport.com`（中国移动通行证 / 一键登录）
- `139.com`（中国移动相关服务）

做法：`[Rule]` 置顶全 `DIRECT`；`[General]` 用 `%APPEND%` 追加 `skip-proxy` 与 `always-real-ip`（真实 IP，不给 Fake-IP）；`[MITM]` 用 `-域名` 排除解密，避免证书绑定失败。

## 抓包证据摘要（2026-11-12）

模块已生效（移动域名解析到真实 IP 而非 198.18 Fake-IP、TLS 握手完整、收到 SHECA 签发的 `*.moazq.chinamobile.com` 真实证书链、无 MITM 干扰、服务器未要求客户端证书），App 向 `zj.moazq.chinamobile.com`（111.22.67.169:443）发出首个加密请求后 25ms、服务器未回复，客户端即 RST 断开，始终未连 10086.cn 主接口。证书路线已排除：新制证书骗不过本机检测，Surge CA 现签证书冒充会撞证书绑定。
