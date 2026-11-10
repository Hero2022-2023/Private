# Surge 免费节点（自动更新）

自动抓取 5 个来源并转换为 Surge 格式，由 Muse 后台约每 6 小时重新生成并推送到本目录。
节点是免费共享节点，短命、不可信，不要用于银行/支付等重要账号。

## 用法

主配置（推荐）：Surge → 配置 → 从 URL 下载配置，填 Surge-Full.conf 的 raw 地址。
它只含 SS / VMess / Hysteria2 / Trojan（Surge 官方支持的协议），默认 FINAL 走 DIRECT，
在 FreeNodes 策略组里手动选节点或选 Auto 自动测速即可，不会一导入就劫持全部流量。

已有自己的配置：把 proxies-include.conf 的内容粘进现有配置，或用 #!include 引用它的 raw 地址。

VLESS Reality 17 条在 Surge-VLESS.conf / vless-links.txt 里单列：VLESS 不在 Surge 官方协议列表，
仅新版 Surge 可能解析；链接版可用于 Shadowrocket / Streisand / sing-box。

mihomo.yaml 是原始合并输出（含重复节点），供 Mihomo / Clash 系客户端使用。

## 本轮说明

- iPoW 来源本轮 0 条：新注册钱包的订阅未激活，接口返回 402 配额耗尽；
  需要有效的 IPOW_TOKEN / IPOW_SUB_URL 才能出节点，后台每轮仍会重试。
- CNC07 的 Russia SS 节点密码被源脚本解析为单字符，疑似截断，可能不可用。
- 节点可用性无法在 Muse 沙箱内端到端验证（出站被强制走出口代理），以 Surge 真机测速为准。
