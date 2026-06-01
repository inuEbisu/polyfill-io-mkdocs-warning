> 本文基于 2026 年 6 月对若干公开课程文档、个人笔记站和历史配置来源的排查。
> 
> TL;DR: 如果你的 MkDocs / Material for MkDocs 站点或其他站点引用了 `polyfill.io`，请尽快删除这行配置。

co-authored by [5dbwat4](https://github.com/5dbwat4); assisted by GPT-5.5

从 MkDocs 笔记中的 polyfill.io 弹窗看一次迟到的供应链风险清理。

## Abstract

近日笔者在访问一些课程实验文档和个人笔记站时，浏览器会弹出来自 `polyfill.io` 的 HTTP Basic Auth 用户名密码输入框。

<img width="1728" height="1084" alt="Image" src="https://github.com/user-attachments/assets/36549279-0d79-4ea3-b1a8-aa0bd21d65c1" />

弹窗出现的原因是页面引用了外部脚本：

```yaml
https://polyfill.io/v3/polyfill.min.js?features=es6
```

浏览器加载页面时会自动请求该脚本。近期该请求正返回 HTTP Basic Auth 认证响应，于是浏览器弹出密码框。对普通访问者而言，正确处理方式是：**直接取消，不要输入统一身份认证、GitHub、浏览器保存的任何密码；**对站点管理员而言，正确处理方式是：**删除对 `polyfill.io` 的引用。**

`polyfill.io` 在 2024 年已经发生过公开披露的供应链安全事件。Sansec 在 2024 年 6 月 25 日披露，`cdn.polyfill.io` 在域名和相关 GitHub 账号易主后，被用于向嵌入该脚本的网站注入恶意 JavaScript；Cloudflare 随后也发布公告称 `polyfill.io` 不再可信，建议所有网站移除相关引用。

追溯历史文档可以看到，MathJax 和 Material for MkDocs 曾经都在官方文档中推荐或展示过包含 `polyfill.io` 的配置片段。大量技术笔记、课程文档和开源项目文档站很可能是在 2019-2024 年间按照当时的官方示例配置搭建的，后来没有再回头清理。因此，2019-2024 年参考 MathJax 官方文档搭建的站点、2021 年末至 2024 年初使用 mkdocs-material 主题搭建的站点、参考其他站点的 `config.yaml` 搭建的 MkDocs 站点等均可能存在此风险。

## 观察到的现象

在若干公开 MkDocs 站点中，可以看到类似配置：

```yaml
extra_javascript:
  - javascripts/mathjax.js
  - https://polyfill.io/v3/polyfill.min.js?features=es6
  - https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js
```

或：

```yaml
extra_javascript:
  - javascripts/mathjax.js
  - https://polyfill.io/v3/polyfill.min.js?features=es6
  - https://unpkg.com/mathjax@3/es5/tex-mml-chtml.js
```

受影响页面通常具备几个共同点：

- 使用 MkDocs 或 Material for MkDocs 构建；
- 为支持 LaTeX 数学公式渲染，引入 `pymdownx.arithmatex` 与 MathJax；
- 在 `mkdocs.yml` 的 `extra_javascript` 中保留了旧示例里的 `polyfill.io`；
- 站点多年未主动审计第三方脚本依赖。

笔者最初在浙江大学相关课程文档和个人笔记中观察到这个问题；然而后续追溯显示，该文档生态遗留问题影响范围为全世界。

## 为什么 `polyfill.io` 不应继续使用

Polyfill 的原始用途是给旧浏览器补齐现代 JavaScript 或 Web API 特性。早年为了支持 IE11 等老浏览器，在页面中加入 ES6 polyfill 是常见实践；但 Polyfill 在 2024 年发生了供应链投毒事件。

公开披露的关键事实包括：

- Sansec 于 2024 年 6 月 25 日发布《Polyfill supply chain attack hits 100K+ sites》，称 Polyfill JS 项目由一家中国公司接手后，被用于向超过 10 万个嵌入该服务的网站注入恶意代码；报告还提到该代码会根据 HTTP headers 动态生成，并在特定移动设备、时间段和访问条件下触发跳转。
- Cloudflare 于 2024 年 6 月 26 日发布公告，明确表示 `polyfill.io` 已不再可信，应从网站中移除；Cloudflare 还为其代理的网站启用了将 `polyfill.io` 链接实时改写到其镜像的缓解措施。
- Sansec 在 2024 年 6 月 27 日的更新中提到，Namecheap 曾将该域名置为 hold 状态，这在当时暂时降低了风险，但 Sansec 仍然建议移除代码中的 `polyfill.io` 引用。

这会造成以下安全问题：

1. Basic Auth 弹窗会让访问者误以为课程文档、学校系统或 GitHub 要求登录。即使弹窗域名显示为 `polyfill.io`，非技术用户仍可能误输入常用密码、统一身份认证密码或浏览器保存的账号密码。
2. 第三方脚本执行风险：任何页面只要引用了 `polyfill.io`，就允许该域名返回的 JavaScript 在自己的站点上下文中运行。对公开课程文档而言，页面本身可能没有登录态或敏感业务逻辑，但恶意脚本仍可进行大量恶意操作，例如修改页面内容、跳转用户、加载更多脚本、诱导下载、伪造登录入口等。

## 这行配置是从哪里来的

笔者最初看到很多浙大系笔记带着同一行配置，一开始以为这是校内同学互相复制导致的局部问题；然而实际传播路径更长，实际传播范围要广得多。

### MathJax 文档中的历史推荐

MathJax 3.0 和 3.1 的历史文档曾在「Browser Compatibility」部分建议，为支持更早的浏览器版本，可以引入 polyfill library。2019 年 7 月 24 日，MathJax 文档仓库的提交 `397fb298db8fe5acbd0385ff27c95936b2c65d35` 中已经包含如下示例：

```html
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
```

这段文字的语境是让 MathJax 3 在 IE11 等旧浏览器中工作。站在 2019 年的时间点，这并不是奇怪的建议。但到了 2024 年，风险模型已经改变：老浏览器兼容性的收益变小，第三方动态脚本供应链风险变得不可接受。

2024 年 6 月 26 日，MathJax 文档提交 `58cf5ae96efc7199aefc7ed9ffb06a85c53122c2` 将示例切换为 Cloudflare 镜像；同日前后的文档更新也加入警告，说明原 `polyfill` 网站在 2024 年被收购后曾被用于向页面注入恶意代码，不应继续使用 `polyfill.io`。

### Material for MkDocs 文档中的历史示例

2021 年 10 月 3 日，Material for MkDocs 在文档中加入 `pymdownx.arithmatex` 与 MathJax 的配置示例，提交 `3ac2bd2df0c23566d2009ddcc5f701c798960030` 中出现了非常典型的配置：

```yaml
extra_javascript:
  - javascripts/config.js
  - https://polyfill.io/v3/polyfill.min.js?features=es6
  - https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js
```


2024 年 6 月 26 日，Material for MkDocs 提交 `12a8e82837fe400dd1f123e41d75b32987a11744` 移除了文档中的 `polyfill.io` 引用，并在同日加上了 Warning：

```
The original `polyfill` website was purchased by a 
Chinese company in 2024, and has been used to inject 
malware into pages that use it.  You should **NOT** use 
the ``polyfill.io`` library any longer, and should either remove 
the reference entirely, or switch to a link like the one above.  
See `this post <https://sansec.io/research/polyfill-supply-chain-attack>`__ for more details.
```
当前示例只保留 MathJax 配置脚本和 MathJax 运行时，例如：

```yaml
extra_javascript:
  - javascripts/mathjax.js
  - https://unpkg.com/mathjax@3/es5/tex-mml-chtml.js
```

但显然后续上游文档更新和安全事件披露并不会自动传导到已经发布的静态站点。换言之，如果在 2021 年末至 2024 年初使用 mkdocs-material 主题搭建的站点，其 `config.yaml` 配置及有可能受当时官方文档推荐或示例配置的影响而引入 `polyfill.io`。

## 处置建议

如果你是访问课程笔记的用户，不负责维护仓库：

- 看到 `polyfill.io` 的用户名密码弹窗，直接取消，不要输入任何账号密码；
- 可以把本帖链接发送给站点维护者，提醒他们删除 `polyfill.io`；
- 若熟悉 GitHub 开源协作流程，可直接提交 PR。

如果你是仓库维护者：

- 删除 `mkdocs.yml` 中对 `polyfill.io` 的引用。

## 时间线

- 2019-07-24：MathJax 文档提交 `397fb298db8fe5acbd0385ff27c95936b2c65d35` 中出现 `https://polyfill.io/v3/polyfill.min.js?features=es6` 示例，用于旧浏览器兼容。
- 2021-10-03：Material for MkDocs 文档提交 `3ac2bd2df0c23566d2009ddcc5f701c798960030` 中加入 Arithmatex / MathJax 配置示例，其中包含 `polyfill.io`。
- 2024-02：据 Sansec 和 Cloudflare 文章，`polyfill.io` 域名和相关 GitHub 账号易主，引发供应链风险担忧。
- 2024-06-25：Sansec 披露 `polyfill.io` 供应链攻击，称其影响超过 10 万个嵌入该服务的网站。
- 2024-06-26：Cloudflare 发布自动改写 `polyfill.io` 链接到 Cloudflare 镜像的缓解措施，并建议移除 `polyfill.io`。
- 2024-06-26：MathJax 文档提交 `58cf5ae96efc7199aefc7ed9ffb06a85c53122c2` 删除原 `polyfill.io` 示例；同日前后的文档更新加入相关警告。
- 2024-06-26：Material for MkDocs 文档提交 `12a8e82837fe400dd1f123e41d75b32987a11744` 移除 `polyfill.io` 引用。
- 2026-06：仍可在一些公开课程文档和个人笔记站中看到历史配置残留，并触发来自 `polyfill.io` 的 Basic Auth 弹窗。

## 参考资料

- Sansec: Polyfill supply chain attack hits 100K+ sites  
  <https://sansec.io/research/polyfill-supply-chain-attack>
- Cloudflare: Automatically replacing polyfill.io links with Cloudflare's mirror for a safer Internet
  <https://blog.cloudflare.com/automatically-replacing-polyfill-io-links-with-cloudflares-mirror-for-a-safer-internet/>
- MathJax docs, 2019-07-24 historical `polyfill.io` example  
  <https://github.com/mathjax/MathJax-docs/blob/397fb298db8fe5acbd0385ff27c95936b2c65d35/web/start.rst>
- MathJax docs, 2024-06-26 removal / warning commit  
  <https://github.com/mathjax/MathJax-docs/commit/58cf5ae96efc7199aefc7ed9ffb06a85c53122c2>
- Material for MkDocs, 2021-10-03 Arithmatex / MathJax example with `polyfill.io`  
  <https://github.com/squidfunk/mkdocs-material/commit/3ac2bd2df0c23566d2009ddcc5f701c798960030>
- Material for MkDocs, 2024-06-26 removal of `polyfill.io` from documentation  
  <https://github.com/squidfunk/mkdocs-material/commit/12a8e82837fe400dd1f123e41d75b32987a11744>
- 原 CC98 讨论：浙大众多笔记及实验文档遭 polyfill.io 供应链投毒而弹窗  
  <https://www.cc98.org/topic/6524251>
- 原 CC98 补充：那么，为什么这么多 mkdocs 笔记会使用 polyfill[.]io，相关代码是从哪抄的  
  <https://www.cc98.org/topic/6524323>
