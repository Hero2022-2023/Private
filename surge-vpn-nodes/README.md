# Surge 免费节点（自动更新 + 有效性测试）

后台约每 6 小时自动执行：抓取 5 个来源 → 去重 → 逐个节点做真实延迟测试 → 只保留测通的节点 → 推送到本目录。

## 在 Surge 里怎么装（重要）

要让配置自动更新，必须装成「托管配置」：Surge → 配置列表 → + → 从 URL 下载配置，
粘贴 Surge-Full.conf 的 raw 地址。用本地文件导入的配置不会自动更新。

## 文件

- Surge-Full.conf：主配置，只含延迟测试通过的 Surge 官方协议节点（SS / VMess / Hysteria2 / Trojan）。
  默认 FINAL 走 DIRECT，在 FreeNodes 组里选节点或 Auto 自动测速。
- Surge-All.conf：全部节点（未过滤），主配置节点太少时备用。
- proxies-include.conf：与主配置相同的有效节点，供已有配置 #!include / 粘贴。
- Surge-VLESS.conf / vless-links.txt：测试通过的 VLESS Reality 节点单列（不在 Surge 官方协议列表，仅新版可能解析；链接版可用于 Shadowrocket / sing-box 等）。
- mihomo.yaml：原始合并输出（含重复），供 Mihomo / Clash 系客户端使用。

## 测试说明

- 测试在本机沙箱里经出口代理双跳完成（sing-box 逐节点请求 generate_204，超时 15 秒），
  与手机直连的网络路径不同：测不通不等于手机上一定不通，测通的节点可信度较高。
- 若某轮测通的节点少于 5 个，系统会回退为发布未过滤列表，并汇报异常。
- iPoW 来源因新注册钱包订阅未激活（接口 402），目前出 0 条，后台每轮仍会重试。
- 免费共享节点短命、不可信，不要用于银行/支付等重要账号。
