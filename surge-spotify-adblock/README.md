# Spotify 去广告 Surge 模块 v1.0.0

去除 Spotify iOS App 的播放音频广告，附带恢复歌手/专辑列表正常展示、去除强制随机播放。

## 它是怎么工作的（两层）

1. **客户端配置补丁（核心）**：Spotify 的广告开关不在某个广告接口里，而在登录时下发的客户端配置里。模块用脚本改写 `bootstrap` / `user-customization-service` 两个接口的 protobuf 响应，把配置里的 `ads` 关掉并写入 premium 客户端属性，App 就不再插播音频广告，首页/搜索的赞助内容也随之消失。核心脚本是社区长期维护的开源脚本 [app2smile/rules](https://github.com/app2smile/rules) 的 `spotify-proto.js`（脚本内版本标记 2026-02-25）和 `spotify-json.js`（歌手/专辑页把 `platform=iphone` 改成 `ipad` 以恢复列表展示），模块以远程地址引用它们，本模块没有复制或改动其代码。
2. **广告端点拦截（本模块自加）**：用 URL Rewrite 把 `pendragon/`（播放广告决策）、`ad-logic/`、`/ads` 三类明确的广告路径返回空字典，并删掉 customize 请求的 `if-none-match` 头，防止服务器返回 304 让补丁脚本拿不到内容。

## 在线更新（这个模块的重点）

- **脚本层每天自动更新**：两个脚本都设了 `script-update-interval=86400`，Surge 每天重新拉一次远程脚本。Spotify 改协议、上游修好后，你这边不用重装模块就会跟上。
- **模块层不过期**：模块文件托管在主人自己的 GitHub 仓库（Hero2022-2023/Private 的 `surge-spotify-adblock/` 目录，公开可读），安装地址永久有效。2026-11-10 首次推送时 GitHub 应用只有读权限、写入被拒，用户重新授权放出写权限后推送成功并已验证 raw 地址返回内容正确。以后本模块有新版时，仍从这个地址在 Surge 模块页执行更新即可。
- Surge 首次下载后会缓存脚本，断网时用缓存版本，不影响播放。

## 安装

1. Surge 里开启 MitM 和脚本功能，生成并安装 Surge CA 证书，并在 iPhone「设置 → 通用 → 关于本机 → 证书信任设置」里打开完全信任。
2. **先登录 Spotify，再启用模块**（上游脚本说明：登录后再开脚本更稳），然后把 Spotify 从后台划掉重开，等一两分钟让配置刷新。若已登录状态下没效果，退出账号重新登录一次。
3. Surge → 修改 → 模块 → 安装新模块，粘贴这个永久地址后启用：

   `https://raw.githubusercontent.com/Hero2022-2023/Private/main/surge-spotify-adblock/Spotify-AdBlock.sgmodule`

   也可以用本地版：把 `Spotify-AdBlock.sgmodule` 文件放进 Surge 配置目录（iCloud Drive 的 Surge 文件夹），本地模块会出现在模块列表里。本地版的脚本同样是远程地址、同样每天自动更新，与远程安装没有区别。
4. 如果装有其它处理 Spotify 的脚本模块（如别的 Spotify 解锁模块），先停用：同一个响应两个脚本会互相顶掉，只会执行先命中的一个。

## 已知限制（先说清）

- **音质不能设为「超高 / Very High」**，那需要真实 Premium 订阅，补丁做不到。
- 不提供真实的离线下载、账号层面的会员身份；播客里的口播/植入广告是音频内容本身的一部分，拦不住。
- Spotify 大版本更新改了协议时，可能短暂失效，等上游脚本更新（每天自动拉）或本模块更新。

## 覆盖范围与测试结果（2026-11-10，本地模拟逐项测试）

通过（35/35，本地用 Node 按模块文件实际提取的正则逐条跑样本 URL）：

- 结构检查 15 项：`#!name`/`#!version`、四个段落齐全、MITM 主机、protobuf 脚本的 `binary-body-mode=1`、两个脚本的每日更新间隔与远程地址。
- 应命中 12 项：bootstrap / customize（主站、区域站 `gew1-spclient`、带 `:443` 三种形态）、歌手页 / 专辑页、pendragon（主站+区域站）、ad-logic、`/ads/` 与 `/ads?query`、customize 的删头规则。
- 不应命中 8 项：bootstrap 子路径、connect-state、api.spotify.com、搜索接口、pendragonx、`adsense`（路径边界防误伤）、metadata 正常接口、bootstrap 不触发删头规则。

未验证（需你在真机上确认）：

- iOS 真机 + 当前版本 Spotify App 的实际去广告效果。这里没有你的手机和 Spotify 账号，无法实测；上面是通过的只是模块结构与 URL 匹配逻辑，不等于真机效果承诺。
- pendragon 拦截在你所用 Spotify 版本上的表现：若出现播放卡顿或跳歌异常，先在模块里删掉 pendragon 那一行 URL Rewrite 再试，并告诉我，我出 v1.0.1。

## 生效检查

重开 Spotify 后正常播放几首歌（免费账号原本每几首插一段）。同时看 Surge「最近请求」：应能看到 `spclient.wg.spotify.com` 的 bootstrap/customize 请求且脚本无报错。完全没变化时，先确认 MitM 证书信任、模块启用，再退出 Spotify 账号重新登录一次。
