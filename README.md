# instagram scraper：免费工具、开源脚本和付费 API 怎么选，抓一次 Instagram 数据到底花多少钱

搜 "instagram scraper" 的人通常卡在同一个地方：手动复制粘贴几条帖子还行，一旦要批量拿几百个账号的资料、帖子、点赞数，Instagram 的反爬机制就会让脚本三天两头失效。这篇文章把目前实际可用的三条路讲清楚：免费开源工具能做什么、做不到什么；专门的 Instagram API 长什么样；以及像 ScraperAPI 这类通用抓取 API 在 Instagram 场景下的真实成本怎么算。所有价格和限制都来自官网和官方文档的当前页面。

> **选购速览**：如果你要的是长期、批量、可编程的抓取，ScraperAPI 的 Hobby 套餐（$49/月，年付折后 $44.10/月，10 万 credits）是最常见的起点，注册后有 7 天免费试用、含 5,000 credits、不需要信用卡，先试再买不会花冤枉钱。👉 [领取 7 天免费试用，含 5,000 credits](https://www.scraperapi.com/?fp_ref=coupons)

## 抓 Instagram 数据，你现在有三种选择

三条路分别是：GitHub 上的免费开源工具、带专用 Instagram 端点的商业 Scraper API、以及通用抓取 API。选哪条取决于两件事：你要的数据量，以及你愿意花多少时间维护脚本。

开源工具适合一次性、小批量的任务。比如想下载某个公开账号的帖子图片，或者偶尔看一眼竞品的发帖频率。商业 API 适合持续运行的项目：监控竞品价格类帖子、追踪 hashtag 表现、批量收集主页数据。

下面这张表先给一个整体印象，后面每一类单独展开。

| 方案 | 典型代表 | 要不要写代码 | 反爬由谁处理 | 适合 |
| --- | --- | --- | --- | --- |
| 开源工具 | GitHub 上的 Python/Node.js 项目 | 要，且要持续维护 | 你自己 | 一次性小任务 |
| 专用 Instagram API | Bright Data 的 Instagram Scraper API | 少量，SDK 调用 | 服务商 | 只抓 Instagram、要结构化 JSON |
| 通用抓取 API | ScraperAPI | 会发 HTTP 请求即可 | 服务商 | Instagram 之外还要抓很多其他网站 |

## 为什么你的 Python 脚本跑三天就挂了

因为 Instagram 的反爬设计专门针对自建脚本，而且几个坑肉眼几乎看不出来。

ChocoData 在它的开源仓库里记录过几个真实踩坑点，值得原样搬过来。第一个坑最阴险：被拦截时 Instagram 返回的页面 HTTP 状态码照样是 200，没有重定向、没有报错，只是一个没有数据的空壳。天真的脚本会正常解析，然后给你记下 "0 followers"，日志里看起来一片祥和。真正贵的失败从来都是这种——连续三周的空数据，你以为是账号没发帖。

第二个坑是检测逻辑会自我欺骗。`captcha` 和 `login` 这两个字符串在完全正常的页面里也存在（它们在 JS 代码包里），用子字符串匹配做拦截检测，成功页面也会触发报警。

第三个坑说明为什么换 headers 没用：把浏览器 User-Agent 换进去，状态码不变、字节数不变、页面内容不变。没有哪组请求头能让一个普通的 HTTP 客户端看起来像浏览器。这是基础设施层面的问题。

最后，AWS、GCP 和 Azure 的数据中心 IP 段早就被拉黑了。这就是为什么同一个脚本在你笔记本上能跑、放到 CI 里就挂。干净的 IP 供应是一笔持续的开销，这也是付费 API 存在的根本原因——它们把代理池和维护这件事商品化了。

## 免费开源工具的真实能力边界

开源工具能完成基础的下载和数据提取，但每一个都有明确写进 README 的限制。

Cheeky Davy 的 instagram-scraper 项目基于 FastAPI，提供 Web 界面和 REST API，可以不登录下载公开帖子、Reels 和轮播图，内置缓存和磁盘自动清理。它的限制项目作者自己也写了：Instagram 的反爬限流（rate limiting）可能随时生效，而且 Instagram 的页面结构一变，工具就得跟着更新。

InstaScrapper 是另一个路线，用 Playwright 抓个人主页图片，支持会话持久化和 S3 上传。它的 README 里有一条很实在的原话："Without authentication, Instagram limits the number of downloadable images. Logging in ensures access to the complete image set." 也就是说不登录Instagram 会限制可下载数量，要拿全套就得交出自己的登录凭据——而且这个工具目前不支持两步验证。用它之前想清楚账号安全这一层。

想批量拿数据而不是下载图片的话，ChocoData 的仓库提供一套免费脚本加一个两端点的 API，免费额度是 1,000 次请求、不要信用卡，但只面向公开的商业账号（它的示例全是 nike、natgeo、cocacola 这类品牌号），明确不做个人画像类数据。

这三个项目都挂着同一句免责声明：仅供教育用途，使用时请遵守 Instagram 的服务条款。这不是客套话。批量抓取 Instagram 数据与平台条款之间的边界问题，是你在选择任何工具（包括付费的）之前都应该自己确认的事。

## 通用抓取 API 是怎么工作的：以 ScraperAPI 为例

ScraperAPI 的用法是把目标 URL 当参数发给它的接口，代理轮换、JS 渲染、验证码处理都在服务端完成，你拿到的是处理过的返回结果。它没有专门的 "Instagram Scraper API" 产品页，Instagram 在它这里就是一个普通的抓取目标 URL——这既意味着灵活性（同一个账号、同一套代码，今天抓 Instagram、明天抓 Amazon），也意味着你需要自己搞清楚 Instagram 的单次请求成本。

计费按 credits 走。官方文档列出的域名价目表是：普通页面 1 credit；Amazon、Walmart、eBay 这类电商 5 credits；Google、Bing 全部子域名 25 credits；社交媒体目前只列了 LinkedIn，30 credits。Instagram 没有单独列出——官方的理由是"其他域名也可能适用（定制爬虫），请先用 API Playground 确认成本"，而且网站可能随时更换防护措施。

用 `premium=true` 或 `render=true` 这类参数，每次请求各加 10 credits；`ultra_premium=true` 加 30。如果 Instagram 触发了绕过机制（Cloudflare、Datadome、PerimeterX 等），每次再加 10 credits。好在有几个办法把成本 uncertainty 压到最低：

1. Dashboard 里的 Domain Multiplier 可以在抓取前查任意 URL 的 credit 消耗
2. 调用 `api.scraperapi.com/account/urlcost` 这个端点，支持所有参数，能在正式跑之前精确算出"开渲染之后到底多少钱"
3. 每个响应头里的 `sa-credit-cost` 会告诉你这次请求实际花了几个 credits
4. `max_cost` 参数可以给单次请求设消耗上限

计费口径也值得记住：只对成功请求（200 和 404 状态码）收费，以及你在 70 秒完成时限内主动取消的请求。被拦截导致失败的请求不扣钱，这一点对 Instagram 这种目标尤其重要——如果某次尝试没抓到东西，它大概率不计费（404 除外）。

## ScraperAPI 全套餐价格表

这是官网 Pricing 页当前展示的全部七个档位，月付价与年付价（10% off 折算到每月）都列出来了。所有付费套餐功能一致：JS 渲染、住宅与移动代理、高级反爬绕过、DataPipeline、无限带宽、99.9% 在线率。

| 套餐 | 月付价 | 年付折后/月 | Credits/月 | 并发线程 | 购买链接 |
| --- | --- | --- | --- | --- | --- |
| Hobby | $49 | $44.10 | 100,000 | 20 | [查看 Hobby 价格](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [查看 Startup 价格](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [查看 Business 价格](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [查看 Scaling 价格](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [查看 Professional 价格](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [查看 Advanced 价格](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | 定制报价 | — | 22,000,000+ | 500+ | [联系获取定制方案](https://www.scraperapi.com/?fp_ref=coupons) |

两个不影响价格、但影响使用的档位差异要知道：Geotargeting 在 Hobby 和 Startup 上只开放美国和欧盟节点，国家级定位从 Business 开始；分析面板的数据保留期在前两档只有最近 30 天。

还有两个容易忽略的机制。一个是 Auto-upgrade：额度用完时自动把你移到下一档套餐，需要主动开启，Hobby 到 Professional 都支持。另一个是 Pay-as-you-go：超出套餐额度后按固定单价继续抓，不用换套餐，但有你自己设定的月度消费上限——这个只在 Scaling 及以上档位可用。Hobby 用户额度用完只有两条路：升级，或者等下个周期。

## 抓 Instagram 的真实成本：一笔可以自己算的账

Hobby 套餐 $49 对应 100,000 credits，单次成本完全取决于 Instagram 请求实际消耗多少 credits，而这正是需要你用官方工具自己确认的部分。

两种边界情况可以帮你建立预期。最乐观的情况：Instagram 请求按普通页面计费，1 credit 一次，100,000 credits 够跑 10 万次请求，约合每千次 $0.49。比较谨慎的情况：如果你的用法需要 `premium=true`（住宅代理），单次变 10 credits，额度只够 1 万次请求，折合每千次 $4.90。如果再叠加渲染或绕过机制，还会更高。

这不是 ScraperAPI 贵，而是这类服务的通行计费逻辑——Bright Data 那边虽然是"每帖 1 credit"的明码标价，但它的 credits 是另一套单价体系，两边不能直接比数字。真正该做的动作很简单：注册试用后，拿你要抓的真实 Instagram URL 去 API Playground 跑一遍，看 `sa-credit-cost` 返回多少，然后乘以你的目标量。十分钟就能把"每月到底要花多少钱"从猜测变成算术。

顺带一提，所有套餐都支持设置 credit 消耗上限，额度用尽前也能在 Dashboard 提前续订，不会出现脚本半夜把额度烧穿还继续扣费的意外。

## 和专用 Instagram API 比，通用 API 差在哪

Bright Data 的 Instagram Scraper API 是目前对 Instagram 需求最"对口"的产品，它把两者最关键的差异摆在了台面上。

| 维度 | Bright Data Instagram API | ScraperAPI |
| --- | --- | --- |
| 产品定位 | 专用 Instagram 端点，输出结构化 JSON | 通用抓取 API，Instagram 是普通目标 |
| 单次成本 | 每帖 1 credit（官方明确） | 未单列 Instagram，需用 Playground 确认 |
| 免费额度 | 新账号每月 5,000 credits | 7 天试用 5,000 credits + 免费层 1,000 credits |
| 返回速度 | 异步作业，1–3 分钟等待 | 同步请求，实时返回 |
| 数据保存 | 快照可下载保留 30 天 | 按请求即时返回 |
| 适用范围 | 仅 Instagram | 任意网站，同一套 key |

按帖计费和结构化输出是 Bright Data 的明显优势：它的 SDK 返回每帖 34 个字段，连点赞数、hashtag、评论数都是现成的，还支持按日期窗口（MM-DD-YYYY）筛选。如果你的项目永远只抓 Instagram，且能接受每次作业等 1–3 分钟，专用端点确实更省心。

ScraperAPI 的位置在另一侧：它的价值在"同一把钥匙开多把锁"。做竞品监控的人很少只盯 Instagram 一个平台——今天要 Instagram 的帖子数据，明天要 Amazon 的价格页，后天要 Google 搜索结果。这时一套 API key、一种接入方式、一份账单，维护成本比每个平台接一套专用工具低得多。代价就是 Instagram 的单次成本没有明码标价，需要自己测。

另外用 Bright Data 的 Python SDK 有一个官方仓库里都标了的坑：每次都要传 `auto_create_zones=False`，否则 SDK 启动时自动创建 zones，没绑支付方式的账号会直接报错。

## 常见问题

**ScraperAPI 抓 Instagram 一次请求消耗多少 credits？**

官方价目表没有单列 Instagram。普通域名 1 credit，带 `premium=true` 或 `render=true` 各加 10，触发反爬绕过再加 10。准确数字请用 API Playground 或 `urlcost` 端点针对你的目标 URL 实测，响应头 `sa-credit-cost` 会给出每次请求的真实消耗。

**不登录能抓多少 Instagram 数据？**

取决于工具。开源下载类工具不登录只能拿到受限数量的内容，InstaScrapper 的作者明确说登录后才能拿完整图片集。商业 API 走代理池抓公开数据，一般不需要你提供 Instagram 账号，但能拿到什么字段以各家产品文档为准。

**免费方案够用来评估吗？**

ScraperAPI 的组合是：7 天试用含 5,000 credits（不要信用卡），加上免费层的 1,000 credits 和 5 个并发连接。对评估"我的目标 URL 一次花几个 credits、返回的数据结构长什么样"来说，6,000 credits 的测试空间绰绰有余。👉 [注册后直接在 API Playground 测试你的目标 URL](https://www.scraperapi.com/?fp_ref=coupons)

**开源方案是不是完全免费？**

工具本身免费，但配套成本不是零：要么承担频繁失效的维护时间，要么自己买干净 IP（数据中心 IP 已被预拉黑），要么把 Instagram 账号凭据交出去换完整数据。项目量小的时候这些成本不明显，量一大就会超过一个 Hobby 套餐的年费（$528/年）。

## 最后的选择建议

判断标准其实就两条：数据量，以及 Instagram 是不是你唯一的目标。

只需要偶尔下载几条公开帖子或图片 → 开源工具够用，挑一个活跃维护的 GitHub 项目，接受限流和结构变动带来的维护成本。

只抓 Instagram、要结构化 JSON、能接受异步等待 → 专用 Instagram API（比如 Bright Data 那套，每帖 1 credit、每月 5,000 免费 credits）对口性最好。

Instagram 只是数据源之一，或者用量会持续增长 → ScraperAPI 这类通用 API 更划算，年付再省 10%。下单前就做一件事：拿你的真实目标 URL 去 Playground 确认单次 credit 消耗，然后用 `目标请求数 × 单次消耗` 对照套餐额度，选第一个够用的档位就行，不用一步到位买大套餐。

👉 [查看全部套餐价格并开始 7 天免费试用](https://www.scraperapi.com/pricing/?fp_ref=coupons)
