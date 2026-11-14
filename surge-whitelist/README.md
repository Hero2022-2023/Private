# 瞬影PRO直连白名单 Surge 模块 v1.0.1

只针对瞬影一家、与其他模块无关：把瞬影 PRO 线路域名 `yd.emby.syemby.com` 的流量强制直连，不走任何代理节点。

为什么必须直连：瞬影 PRO 服只允许中国大陆 IP 访问，走代理节点出口会被拒；官方也明令禁止代理工具，账号出现多地异地 IP 登录会直接封号。所以这条线路在 Surge 里必须绕开所有节点、用你的真实 IP 出去。

## 实现方式（三层）

1. `[Rule]` 一条 `DOMAIN-SUFFIX,yd.emby.syemby.com,DIRECT`：模块规则优先于主配置规则，最终策略组选了什么都不影响它；后缀匹配同时覆盖该域名本身和它的子域名。
2. `[General]` `skip-proxy` + `always-real-ip` 用 `%APPEND%` 追加该域名（裸域名与 `*.` 通配各一份）：拿真实 IP，不走 Fake-IP，服务端看到的是你的真实大陆 IP。
3. `[MITM]` 负向排除该域名：Surge 不解密它的流量，证书环节零干扰。

不包含 `compatibility-mode`，不改 Surge 的接管方式，本模块只做这一件事。

## 版本

- v1.0.1（2026-11-14）：按要求收窄为瞬影专用——线路域名定为 `yd.emby.syemby.com`（用户确认），删除 v1.0.0 里另两条非瞬影条目，模块内不再包含任何其他服务的域名。
- v1.0.0（2026-11-14）：初版，三域名直连白名单。

## 安装

Surge → 修改 → 模块 → 安装新模块，粘贴永久地址（托管在主人 GitHub 仓库，`surge-whitelist/` 目录）：

```
https://raw.githubusercontent.com/Hero2022-2023/Private/main/surge-whitelist/Whitelist-Direct.sgmodule
```

已装 v1.0.0 的，在 Surge 模块页对它点更新即可升到 v1.0.1（地址不变）。启用后正常打开瞬影 PRO 播放即可；若播放时媒体流跳到别的域名（在 Surge「最近请求」里能看到实际域名），把那个域名告诉我，我加上再推一版。

## 测试结果（2026-11-14，本地逐项模拟）

v1.0.1 通过 28/28：结构与版本 6 项、模块内不含已删域名 2 项、唯一规则原文核对、skip-proxy / always-real-ip / MITM 对裸域名与通配各一份共 6 项、DOMAIN-SUFFIX 语义模拟命中 2 项（域名本身及其子域名）、不命中 10 项（含 syemby.com 本身、面板 ad.syemby.com、其他服务域名与仿冒形态）。

未在真机验证：模块已推送并回拉核对与本地一致；瞬影在你 iPhone 上能否正常播放，以你的实际结果为准。
