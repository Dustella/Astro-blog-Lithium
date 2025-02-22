---
title: 简中独立开发者流量密码
date: 2023-01-15 13:08:00
description: 简中独立开发者的流量密码，一是翻墙，二是博客。整个“简中开发圈”获得的 star，有 80% 以上被浪费在了这些东西上面，而这与它们的质量完全无关。
tag:
  - 杂谈
  - 分析
---

简中独立开发者的流量密码，一是翻墙，二是博客。整个“简中开发圈”[^1]获得的 star，有 80% 以上被浪费在了这些东西上面，而这与它们的质量完全无关。

## 博客

先挑危害没那么大的“博客”讲讲。“有个自己的博客/网站”是一件既能满足好奇心，又能让人很有成就感的事情。小学时我就很想开个自己的网站，尽管一直到高中才如愿以偿。然而这样旺盛的需求自然也就培养出了大量没什么能力的开发者，他们写的博客系统或主题可谓“怎么灵车怎么来”，代码方面基本维持在“恰好能跑起来”的程度。更恶劣者，代码从一处抄来，又被另一个地方抄去，几经转手，却完全不知道原理。出了 bug 怎么修？只能大眼瞪小眼。大量博客主（包括曾经的我）陷在各式各样的营销词汇中，什么“ajax”“pjax”啦，什么“免刷新加载”[^2]啦，“cdn”啦，“图片优化”啦，却对博客的核心——内容产出，没有任何关注。这之后，通过互加友链，形成一个个小圈子，在里面干着名为折腾技术实则如小孩子玩玩具一般的事情。从这个角度上说，既然这些博客本质不过青年人的玩具，再加上写博客的人也越来越少，那么这个流量密码倒也没什么进一步批评的必要了。

## 翻墙

“翻墙”，则是危害更大、更该被批评的那个流量密码。它作为所有在墙外活动的中国大陆人的共同需求，市场不可谓不小。很可惜，这样的需求却并没有带来与之匹配的技术水平。一个很典型的例子是 V2Ray（VMess 协议），它的设计者没有最基本的安全常识。V2Ray 崛起时，SS 和 SSR 正在相互指责对方不安全，当时双方的用户都大批量转而使用号称更加安全的 V2Ray。五年后的今天，我们可以认为：事实上，V2Ray 的崛起是且仅是 SS 和 SSR 相争的结果，与它自身的安全性和代码质量没有任何关系。而与之相比，SS 至少**实现**了自己的*设计目标*：单纯的加密、没有任何特征[^3]，而 V2Ray 却没有实现那些被自己宣传的特性。[^4]

以下列表按我获知的时间顺序排序，请注意：你的内容依然是安全的，墙没法在中间解密。

| 严重性 | 标准                                                                 |
| ------ | -------------------------------------------------------------------- |
| 轻微   | 不会造成用户翻墙行为被发现                                           |
| 中等   | 需要墙的主动探测                                                     |
| 严重   | 墙只需要检查流量（尽管想利用文中的两个严重漏洞都需要不小的性能开销） |

1. （中等，已修）最早的一个洞，非 AEAD 加密套件下，V2Ray（VMess）可以被主动探测。与之相比，SS 虽然被 SSR 的作者找到了主动探测的方法，并且给出了程序，但 SS 协议进行了与 VMess 一样的修改（加上 AEAD）就能按设计目标一直沿用到现在。[^5]

2. （轻微，未修）VMess 协议会试图对传输内容用同一个算法加密两次。从密码学出发，这不会带来任何额外的安全性，而只是空耗性能。

3. （严重，未修）如果你使用了 WebSocket/http2 等 TLS 加密，那么由于 V2Ray 使用了 gotls 作为自己的 tls 加密套件，而这个套件的 ClientHello 握手包特征与浏览器的不同，可以被识别。这条不属于 VMess 协议的缺陷，而是 V2ray 的 TLS 实现的问题。

4. （严重，已修）VMess 协议的数据包末尾有一个 padding 字段，这个字段在设计上是被随机数填充的，然而 V2Ray 却使用了 go 的 math.random 作为随机数生成器。这并不是一个密码学安全的随机数生成器——它的种子总共只有 $2^{32}-1$ 个，理论上可以从这些种子出发，生成所有可能的随机数列，并对数据包进行匹配，然后一眼丁真。在别的很多地方都会有这样的 padding/salt 设计，但 salt 都是要跟着内容一起加密的，而 V2Ray 却是明文！前面两次加密浪费性能，现在又想着省性能了？

只出一个低级错误还可以说是疏忽，多个呢？漏洞 2 和 4 反映出，V2Ray/VMess 协议的作者就是缺乏最最基础的安全/密码学常识，并且根本没有能力去设计一个拿来对抗全世界最强大[^6]审查机构的协议。V2Ray 的唯一优势在于它的可扩展性和可配置性，但这一点在 Clash 这样的后起之秀面前就不值一提了。[^7]

V2Ray 绝不是个案：同样是前些日子，V2Board 被爆出[硬编码](https://github.com/v2board/v2board/blob/08653fb2cd0a823f6c56999019917b7de071e2da/resources/views/admin.blade.php#L45)了一段谷歌统计代码[^8]，这种无异于主动投毒的行为[^9]同样是在几年后才被发现。此外我还必须指出，[V2Board 发生了数据泄露](https://t.me/XueXiNmsland/48290)，而作为一个翻墙面板，它本应对安全问题有足够的敏感。

“翻墙”作为流量密码的危害更大，这一点无需置疑。如果说“博客圈”只是在自嗨和相互炫耀新玩具，那么“翻墙界”则是在把事实上并没有什么安全性的工具向大众推荐，并且让大众以为它们很安全。获得高 star 的项目几乎无一例外依赖于营销，而非技术。[^10]

写到这里，本该拿一些号召作为结尾，但我发现其实没什么好说的。所有“流量密码”都有其背后的原因，而它们无一例外都不是简单号召能改变的（建议阅读《[抢购的理性](/posts/panicPurchasing/)》）。不过我仍然希望读到这篇文章的人能自觉对流量密码加以警惕，有志于开发的人则不应在流量密码上消耗过多本该用于自我提升的时间与精力。

> 本文的目的只是指出流量密码并且向那些名不副实的项目狠狠地吐口水，而“找出那些真正有质量的项目”超过了本文作者的能力范围。目前而言，我认为 [shadow-tls](https://github.com/ihciah/shadow-tls) 是值得推荐的。

---

[^1]: 指由生活在中国大陆的独立开发者构成的集合。
[^2]: 和 pjax 指的是同一个技术。
[^3]: 尽管这个目标不一定是正确的，毕竟“没有特征就是最大的特征”。
[^4]: [V2fly.org](https://guide.v2fly.org/#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98-q-a) “白话文教程”宣称，V2Ray“一开始就致力于让大家更好更快的科学上网”。
[^5]: SSR 作者实现的是对 cfb 的重放攻击，使用 AEAD 可以规避，但 V2Ray 也有类似问题，并且是和 SS 差不多同时换上 AEAD 的，也就是说单纯的 VMess 的安全性从未高于 SS。
[^6]: 或者第二，视标准而定。
[^7]: 只是说 Clash 的可扩展性和可配置性好，而不是说它和它衍生产物的[安全性](https://t.me/XueXiNmsland/49983)好。
[^8]: [前端编译产物中也存在](https://github.com/v2board/v2board-admin/blob/50b6058bf67fa9cea13f8a6cb9d0cd8d3c30a42b/index.html#L9)，可以相互验证。
[^9]: 硬编码的这段谷歌统计是为开发者而非部署者收集数据，且没有展示任何相关选项或说明，性质极其恶劣。甚至这段统计代码被作为特征去[申请了专利](https://patents.google.com/patent/CN113505323A/zh)，可谓粗心大意 / 不安好心 vs. 沽名钓誉。
[^10]: 把流量加密以后套个 WebSocket 发出去，这件事情（无论要不要再套一层 TLS）在 NodeJS 上 100 行代码以内就可以实现，技术要求并没有特别高。大概在 C++ 和 [Go](https://github.com/v2fly/v2ray-core) 上写起来会更难吧。
