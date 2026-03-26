---
title: "什么是插件？"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - What Is A Plugin
    - Intro
    - Repos
---

# 什么是插件？

插件是用于扩展 WordPress 核心功能的代码包。WordPress 插件由 PHP 代码构成，并可包含其他资源，如图像、CSS 和 JavaScript。

通过创建自己的插件，你正在 _扩展_ WordPress，例如，在 WordPress 已有功能之上构建额外功能，或者编写一个显示网站最近十篇文章链接的插件。

或者，利用 WordPress 的自定义文章类型，你可以编写一个创建功能完整的支持工单系统的插件，包含邮件通知、自定义工单状态和面向客户的入口。可能性是 _无限的_！

大多数 WordPress 插件由多个文件组成，但插件实际上 _只需要_ 一个主文件，其头部包含特定格式的 [DocBlock](https://en.wikipedia.org/wiki/PHPDoc)。

[Hello Dolly](https://wordpress.org/plugins/hello-dolly/ "Hello Dolly") 是最早的插件之一，仅 [100 行](https://plugins.trac.wordpress.org/browser/hello-dolly/trunk/hello.php) 代码。它在 WordPress 后台显示 [著名歌曲](https://en.wikipedia.org/wiki/Hello,_Dolly!_(song)) 的歌词。PHP 文件中使用了一些 CSS 来控制歌词的样式。

作为 WordPress.org 的插件作者，你拥有绝佳的机会来创建一个将被数百万 WordPress 用户安装、调整和喜爱的插件。**你** 只需要将你的好想法转化为代码。本插件手册将帮助你实现这一目标。