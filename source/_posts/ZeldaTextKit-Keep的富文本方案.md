---
title: ZeldaTextKit - Keep 的富文本方案
date: 2018-01-13 22:02:46
tags: iOS
---

# 背景
对于 Keep 社交部分的业务来说，PM 很想灵活的展示一些图文信息。这些信息可以图片文字混合排列，并且也要灵活。方便后端配置。对于单页面场景，使用 WebView 是一个不错的方案。但是如果是要放在列表当中，对于性能要求很高。WebView 就不太适用。所以就需要一个高性能的显示富文本的控件。  

当时项目里面其实已经引入了很多的富文本控件。包括 [RTLabel](https://github.com/honcheng/RTLabel)，[YYText](https://github.com/ibireme/YYText) ，[TTTAttributedLabel](https://github.com/TTTAttributedLabel/TTTAttributedLabel)  三个不同的富文本控件，这些控件或多或少都有些小问题。比如 RTLabel 已经很久没人维护，不支持 AutoLayout。另外两个不支持 HTML等。所以我们就想自己造个轮子。把项目里面的所有的控件统一。

# 调研以及实现
基于上面的背景，这个控件需要支持 HTML，@ #hashtag# 解析也不能少。对于 AutoLayout 也要友好。 于是前期调研了一些现有的控件：
- [DTCoreText](https://github.com/Cocoanetics/DTCoreText) 强大的富文本控件，可以解析 HTML + CSS
- [YYText](https://github.com/ibireme/YYText) 支持图文混排，支持异步绘制，基于 CoreText 实现。支持简单的 Markdown/表情解析


## HTML 解析

对于 HTML 的支持， [DTCoreText](https://github.com/Cocoanetics/DTCoreText) 是非常强大的，文本的绘制基于 CoreText。HTML 的解析是单独的一套逻辑。渲染与逻辑分离。封装了不同的接口：DTAttributedLabel，DTAttributedTextCell，DTAttributedTextView。对于我们来说这个控件有点重，而且就目前的情况来说我们不需要对富文本进行编辑。只需要简单的显示就可以了。也不需要支持丰富的 CSS。所以我们参考了很多  [DTCoreText](https://github.com/Cocoanetics/DTCoreText) 的实现思路，使用了类似的 SAX 解析方式。封装了一套 HTML 解析。具体的实现思路可以参考这篇文章：http://blog.cnbang.net/tech/2630/  我在这里就不再赘述。



## UI渲染
对于 UI 的渲染我们没有使用 CoreText， 而是使用了更友好的 TextKit。相比CoreText 的 c 接口，TextKit 更加现代，而且概念也比较少。我们使用了 TextKit 重新实现一个  ZeldaLabel。对于 TextKit 这篇文章写的很好：https://objccn.io/issue-5-1/

TextKit 分为三部分：
- `NSTextStorage` 用于保存和管理富文本字符串。继承于 `NSAttributedString`

- `NSTextContainer` 定义文本绘制的区域。

- `NSLayoutManager` 将所有的组件粘合在一起：监听 Text Storage 中文本或属性改变的通知，一旦接收到通知就触发布局进程。它将所有的Text Storage 提供的文本翻译为字形。字形生成后，这个管理器向它的 Text Containers 查询文本可用以绘制的区域。然后开始逐行填充。填充完后就会显示在对应的控件上。

ZeldaLabel 使用 TextKit 重新实现了渲染，继承于 UILabel，首先在 attributedText 的 didSet 方法处理加工 AttributedString, 处理完成后把处理完的文本设置给 textStorage。
重写了 drawText 的绘制逻辑，使用 layoutManager 进行排版渲染:
```textContainer.size = rect.size
let newOrigin = textOrigin(inRect: rect)
let glyphRange = layoutManager.glyphRange(for: textContainer)
layoutManager.drawBackground(forGlyphRange: glyphRange, at: newOrigin)
layoutManager.drawGlyphs(forGlyphRange: glyphRange, at: newOrigin)```

这样文本就可以显示在 UIlabel 上面了

## 点击实现
通过设置 attributedText 解析富文本信息。 遍历所有的`NSLinkAttributeName` 属性后保存一个数组。重写 touchesBegan 等事件实现点击判断对应的文本所在的 range ，如果是在之前保存的数组中，就触发点击，响应事件。将当前点击的 link 回调出去。  
为了支持 @ #hashtag# 我们扩展了 `NSAttributedStringKey` 添加了`NSAttributedStringKeyMention` 和 `NSAttributedStringKeyHashTag` 这样就可以区分不同的点击类型，实现不同的回调和设置。



## AutoLayout 支持

使用了TextKit 对 AutoLayout 支持就变得很简单。只需要重写 `intrinsicContentSize` 将 `NSTextContainer` 重新绘制一下文本就可以知道具体的 size 了

# 总结

ZeldaTextKit 在大部分场景下性能还是不错的。但是如果是混入了很多的图片的话，性能依然不是很好。未来可能也会提供使用异步的方式实现 UI 的渲染等。

在做这个控件的时候当然也遇到了坑。对于 iOS 10.0 - 10.2 系统上面 emoji 的字体排版会居下一点点。造成被 emoji 被切断。对于我们的需求来说就是图片和 emoji 混排的时候，图片会更靠上一点。具体的bug 地址：https://openradar.appspot.com/radar?id=4998540401049600 苹果建议将 emoji 的字体替换，但是效果不是很好。无奈只能试了很多次调整对应的图片 `NSTextAttachment` 的位置。


