---
title: "简码"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Shortcodes
    - Repos
    - Data
---

# 简码

出于安全考虑，WordPress 内容中禁止运行 PHP；为了允许与内容进行动态交互，WordPress 2.5 版本引入了简码。

简码是一种宏，可用于与内容进行动态交互，例如从文章附件中创建相册或渲染视频。

## 为何使用简码？

简码是一种有价值的方式，既能保持内容简洁和语义化，又能让最终用户在一定程度上通过编程方式改变内容的呈现方式。

当最终用户使用简码在文章中添加照片库时，他们使用尽可能少的数据来指示相册应如何呈现。

优点：

- 不会向文章内容添加标记，这意味着标记和样式可以轻松地即时或稍后修改。
- 简码还可以接受参数，允许用户根据每个实例修改简码的行为。

## 内置简码

默认情况下，WordPress 包含以下简码：

- `[caption]` – 允许在内容周围添加标题
- `[gallery]` – 允许显示图片库
- `[audio]` – 允许嵌入和播放音频文件
- `[video]` – 允许嵌入和播放视频文件
- `[playlist]` – 允许显示音频或视频文件集合
- `[embed]` – 允许包装嵌入项目

## 简码最佳实践

开发简码的最佳实践包括[插件开发最佳实践](https://developer.wordpress.org/plugins/plugin-basics/best-practices/)以及以下列表：

- **始终返回！** 简码本质上是过滤器，因此创建“[副作用](https://en.wikipedia.org/wiki/Side_effect_(computer_science))”会导致意外错误。
- 为简码名称添加前缀，以避免与其他插件冲突。
- 清理输入并转义输出。
- 向用户提供所有简码属性的清晰文档。

## 快速参考

查看使用[基本简码结构、处理自闭合和包含场景、简码嵌套以及保护输出](https://developer.wordpress.org/plugins/shortcodes/shortcodes-with-parameters/#complete-example)的完整示例。

## 外部资源

- [WordPress 简码生成器](https://generatewp.com/shortcodes/)