# AI 额度页部署指南

把 AI 额度页（Token Quota）部署到一台新主机上的操作指南。页面本身是
单文件静态页 + 两张 hero 素材，没有后端、没有数据库、不用容器。
迁移不难，主要工作是把写死的地址和额度卡换成新主机的。

英文版：[README.md](README.md)

## 1. 这个目录里有什么

| 文件 | 是什么 | 用法 |
|---|---|---|
| `README-zh.md` | 本指南 | 按着做 |
| `README.md` | 本指南（英文版） | 与 `README-zh.md` 内容相同 |
| `token-quota.html.template` | 额度页模板 | 填占位符后去 `.template`，放到新主机上当 `token-quota.html` |
| `images/hero-video.mp4` | Hero 视频 | 跟模板一起放到新主机上（`images/` 目录） |
| `images/forest-poster.jpg` | Hero 海报 | 同上；同时是 `.hero` 自己的背景层 |

模板里的 `__占位符__` 都要填成新主机的真实值，见第 3 节。填完之前
这个文件不能直接用。模板里没有任何真实 IP、邮箱和域名，可以放心放着。

注意：`token-quota.html.template` 是旧主机 `token-quota.html` 在某
一时刻的快照。如果页面后来改了，要重新生成本模板（或直接拷新版
`token-quota.html`，按第 3 节的表把相关值改掉）。

## 2. 页面是怎么工作的

- 单文件静态页：HTML + CSS + JS 全在一个文件里，只依赖 `images/` 下
  的两张素材。放到任何能 serving 静态文件的 web 服务器上就能跑。
- 额度卡：`CARDS` 数组里每行一张卡，`{ software, ai, start, cycle,
  expires? }`。`start` 是你知道的任意一次重置时刻（北京时间），
  `cycle` 是重置周期（`7d` / `24h` / `30d`），`expires` 可选，到期
  时刻。页面按当前北京时间自动算出下一次重置，不用手动改日期。
- 时钟：服务器时间（HEAD 请求的 Date 头取回，UTC+8），60 秒重同步、
  1 秒一跳。
- Hero 视频：加载时静止在毛玻璃后；悬停或点按播放，再点一次或移开
  鼠标必须 STOP（pause + 回到第 0 帧），不是暂停。
- 主题：**没有切换按钮**，只跟系统 `prefers-color-scheme`，和必应的
  “System default”一样；系统变它就变。
- 计时器 60 秒一刷；最后 10 分钟改 31 毫秒级倒计时；到零芯片渐变为
  🎉 10 秒，随后周期推进、卡片重排；过期卡变灰（⌛）沉底。

## 3. 占位符表

填模板时按这个表换。第三列是旧主机（OL8）的现值，只当例子看，
新主机按第四列填。敏感值已隐去，不写真实值。

| 占位符 | 是什么 | OL8 现值（例子） | 新主机填什么 |
|---|---|---|---|
| `__PAGE_URL__` | 页面地址 | `https://example.com/token-quota.html` | 新主机上这个页面的完整 URL（canonical 和 og:url 用） |
| `__META_DESCRIPTION__` | 页面描述 | （见模板） | 一句话描述这个页面（description / og:description / twitter:description 三处共用） |
| `__SOFTWARE_1__` | 第 1 张卡的软件名 | （隐私，已隐去） | 如 `MuseAI`、`ChatGPT` |
| `__AI_1__` | 第 1 张卡的 AI 名 | （隐私，已隐去） | 如 `Yang`、`Work` |
| `__START_1__` | 第 1 张卡的起算时刻 | （隐私，已隐去） | 你知道的任意一次重置时刻，北京时间，格式 `YYYY-MM-DD HH:mm:ss`，如 `2027-01-01 00:00:00` |
| `__CYCLE_1__` | 第 1 张卡的周期 | `7d` | `7d` / `24h` / `30d` |
| `__SOFTWARE_2__` | 第 2 张卡的软件名 | （隐私，已隐去） | 同上 |
| `__AI_2__` | 第 2 张卡的 AI 名 | （隐私，已隐去） | 同上 |
| `__START_2__` | 第 2 张卡的起算时刻 | （隐私，已隐去） | 同上 |
| `__CYCLE_2__` | 第 2 张卡的周期 | `7d` | 同上 |
| `__SOFTWARE_3__` | 第 3 张卡的软件名 | （隐私，已隐去） | 同上 |
| `__AI_3__` | 第 3 张卡的 AI 名 | （隐私，已隐去） | 同上 |
| `__START_3__` | 第 3 张卡的起算时刻 | （隐私，已隐去） | 同上 |
| `__CYCLE_3__` | 第 3 张卡的周期 | `7d` | 同上 |
| `__EXPIRES_3__` | 第 3 张卡的到期时刻 | （隐私，已隐去） | 可选；没有到期就把 `, expires: "__EXPIRES_3__"` 整段删掉 |
| `__FOOTER__` | 页脚 | （隐私，已隐去） | 如 `© 2026 Your Name` |

加卡：把 `CARDS` 里的一行复制一份，按上表改值即可；删卡：删掉对应
的一行。`start` 只要是你知道的任意一次重置时刻就行，页面会自动
往后推算，不用算下一次是哪天。

Hero 素材：`images/hero-video.mp4` 和 `images/forest-poster.jpg`
是随模板一起带的，换成自己的也行——文件名不变直接覆盖，或改模板
里 `images/hero-video.mp4?v=5` / `images/forest-poster.jpg?v=5`
的路径（`?v=5` 是缓存号，换素材时改一下数字，绕开浏览器缓存）。

## 4. 部署步骤（按顺序）

1. 从这个目录把 `token-quota.html.template` 和 `images/` 拷到新主机，
   模板填好第 3 节的占位符，去掉 `.template` 后缀，当
   `token-quota.html` 放到站点的根目录（`images/` 跟它放一起）。
2. 确认 `images/hero-video.mp4` 和 `images/forest-poster.jpg` 能
   访问（直接打开这两个地址能看到东西）。
3. 用浏览器打开新地址，确认三件事：时钟在走（时间对得上）、额度卡
   在倒计时、点一下 hero 视频能播、再点能停。
4. 切系统深浅色各看一次，确认页面跟着变、毛玻璃没丢。

## 5. 注意事项

- 页面是纯静态的，不用配后端；但时钟取的是服务器时间的 Date 头，
  所以必须走 HTTP(S) 访问，直接双击本地文件打开时时钟取不到时间。
- `?v=5` 是素材的缓存号；换了视频或海报后把数字 +1，不然访客的
  浏览器可能还显示旧的。
- 模板是快照：`token-quota.html` 改了之后要重新生成模板（或直接拷
  新版再改占位符），不要拿旧模板去套新版页面。
- 防翻译：页面带 `<meta name="google" content="notranslate">` +
  `translate="no"`，不要删。
