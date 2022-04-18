---
lang: zh-Hans
---

# 修复个人网站失效链接

- 发布于：2022-04-18
- [Markdown][raw]

[raw]: https://raw.githubusercontent.com/liolok/liolok.com/master/zhs/fix-broken-urls-of-my-site/index.md

## 起因

链接本应该是永久的（permanent URL），但是由于各种各样的原因失效之后，被其他网站引用的链接打开会是 404 页面甚至更糟。

我个人的情况是几年中反复用过各种风格的链接格式，比如中文页面的链接是包含中文还是全部英文、字母大小写（title case）等等；后来更是一不做二不休直接从 GitHub Pages 子域名换了自己的自定义域名。

在上网查资料的时候，偶尔也会遇到某些页面甚至整个网站已经打不开了的情况，或许是关停不维护了，但也有可能像我自己一样是把本来的文章或者说内容迁移到了新的链接。

一想到自己点开失效链接时的失望感，我自省己所不欲勿施于人：网友觉得我的文章值得引用，我不应该辜负人家，让链接一直就那样失效下去。有两种解决方案：

1. 通知对方，我的文章已经迁移，请求更新其引用的链接：
    - 优点：访问体验完全无缝
    - 缺点：如何联系得上站长？就算联系得上，也不见得好意思由于自己的问题而麻烦人家。
2. 在旧的失效链接页面进行自动重定向：
    - 优点：不需要对引用进行更改
    - 缺点：访问体验相对有割裂感、可能会令人困惑。

## 寻找被引用的失效链接

更现实的情况是：我写的文章并没有被引用，这种情况下似乎也没必要进行修复，反正没人会点开不是嘛。

需要关心的是那些已经被外部站点引用的失效链接，我之前尝试过在搜索引擎直接对旧域名进行搜索，有一定效果但还是不太满意。

于是我找到了这个在线工具：[ahrefs.com/backlink-checker](https://ahrefs.com/backlink-checker)。

不注册帐号、免费使用的情况下，可以针对：

- 某个的精确 URL
- 某个域名（强制包含子域名）下面的所有 URL

进行反向查询，只显示前 100 个查询结果。

![](../../fix-broken-urls-of-my-site/ahrefs-backlink-checker-example.webp)

## 旧链接自动重定向

重定向也可以通过 HTTP 状态码 301/302 来实现，但是不符合我的需求场景所以没有深入了解。

在[旧链接][old_url]写一个 HTML 来实现自动跳转到迁移之后的[新链接][new_url]：

[old_url]: https://liolok.github.io/zh-CN/Bézier-曲线及其-deCasteljau-剖分算法/
[new_url]: https://liolok.com/zhs/bezier-curve-and-de-casteljau-algorithm/

> 注意：如果像我这样链接是目录形式（pretty URL）的话，HTML 文件应该对应目录下的 `index.html`。

```html
<!DOCTYPE html>
<html lang="zh-Hans">

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta http-equiv="refresh" content="3; url=https://liolok.com/zhs/bezier-curve-and-de-casteljau-algorithm/">
    <link rel="canonical" href="https://liolok.com/zhs/bezier-curve-and-de-casteljau-algorithm/">
    <title>网站已迁移</title>
</head>

<body>
    <h1>网站已迁移</h1>
    <p><a href="https://liolok.com/zhs/bezier-curve-and-de-casteljau-algorithm/">正在跳转至新站点...</a></p>
</body>

</html>
```

核心代码就是那行 [meta refresh](https://en.wikipedia.org/wiki/Meta_refresh)。`content` 参数第一项是跳转的延迟，比如三秒或者五秒；第二项 `url=` 后面写新链接即可。

## 感言

想要维持永久链接一直有效，基本思路如下：

1. 不要给页面换路径
2. 不要给网站换域名
3. 不要弄丢域名
4. 寄希望于人类文明不衰落到网络瘫痪的程度
5. 尽量永远活着来维护网站

最后，祝大家身体健康，一生长寿。
