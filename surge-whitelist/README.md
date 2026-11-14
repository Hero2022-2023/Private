# 直连白名单 Surge 模块 v1.0.0

把三个服务的流量强制直连、不走任何代理节点，解决「挂着 Surge 时这些服务用不了或很慢」的问题：

| 服务 | 域名 | 为什么必须直连 |
|---|---|---|
| 瞬影 PRO 线路 | `syemby.com` | PRO 服只允许中国大陆 IP 访问，走节点出口会被拒；官方也明令禁止代理工具，多地异地 IP 登录会封号 |
| 139 云盘接口 | `139.com` | 国内服务，走代理绕远且易触发风控 |
| 移动云盘对象存储 | `cmecloud.cn` | 139 云盘的文件直链实际走 `*.eos.*.cmecloud.cn` 这类对象存储域名，走代理下载慢、易断 |

## 实现方式（三层，和之前中国移动模块同一套，但不带 compatibility-mode）

1. `[Rule]` 三条 `DOMAIN-SUFFIX,…,DIRECT`：模块规则优先于主配置规则，最终策略组选了什么都不影响这三个域名。
2. `[General]` `skip-proxy` + `always-real-ip` 用 `%APPEND%` 追加这三个域名：拿真实 IP，不走 Fake-IP，CDN 按你的真实位置调度最近节点，瞬影那边看到的也是你的真实大陆 IP。
3. `[MITM]` 负向排除这三个域名：Surge 不解密它们的流量，证书环节零干扰。

**不包含** `compatibility-mode`：中国移动那次已实测，改接管模式与直连无关且容易误事，本模块只做直连白名单这一件事。

## 一个需要你核对的假设（瞬影 PRO 线路域名）

你给的第一条写的是 `DOMAIN-SUFFIX,瞬影PRO线路,DIRECT`，「瞬影PRO线路」不是合法域名、没法直接当后缀用。我按瞬影的官方域名落成了 `syemby.com`（用户面板就是 `ad.syemby.com`，线路一般是它的子域名）。

装好后请核对一次：Surge → 最近请求，播一条瞬影 PRO 的片，看请求域名是不是 `*.syemby.com`；另外播放真正出数据的域名也要看一眼（302 直连模式下，媒体流可能跳到另一个 CDN 域名）。如果线路或媒体流走的是别的域名，把域名告诉我，我加进模块再推一次，你在 Surge 里点更新就行。

## 安装

Surge → 修改 → 模块 → 安装新模块，粘贴永久地址（托管在主人 GitHub 仓库，`surge-whitelist/` 目录）：

```
https://raw.githubusercontent.com/Hero2022-2023/Private/main/surge-whitelist/Whitelist-Direct.sgmodule
```

启用即可，不需要改主配置。已经装了「中国移动直连绕过」模块的话可以停用/删掉：本模块已覆盖 139.com，且不带那个模块里实测无效的模式设置；chinamobile.com / 10086.cn 不在本模块范围内（中国移动 App 那个问题已由快捷指令自动化结案，不靠规则）。

## 测试结果（2026-11-14，本地逐项模拟）

通过 35/35：结构与三段配置 19 项、三条规则原文逐条核对、DOMAIN-SUFFIX 语义模拟命中 9 项（含 ad.syemby.com、yun.139.com、eos-beijing-1.cmecloud.cn 等真实形态）、不命中 7 项（含 notsyemby.com、syemby.com.evil.com、139.com.cn 等防误伤形态），并确认模块内没有 compatibility-mode 设置行。

未在真机验证：模块已推送并回拉核对与本地一致，但这三个服务在你 iPhone 上是否全部恢复正常，要以你实际播放/打开的结果为准；瞬影线路域名假设见上文，尤其需要这一次核对。
