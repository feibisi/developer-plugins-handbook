---
title: "插件开发入门"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Intro
    - Repos
    - Data
---

# 插件开发入门

欢迎阅读《插件开发者手册》。无论您是在编写第一个插件还是第五十个插件，我们都希望这份资源能帮助您编写出最优秀的插件。

本手册涵盖多种主题——从插件头部应包含的内容，到安全最佳实践，再到可用于构建插件的工具。它也是一份持续完善的文档——如果您发现任何缺失或不完整之处，请在 Slack 中通知文档团队，我们将共同改进。

## 为何开发插件

WordPress 开发中有一条核心原则：**不要修改 WordPress 核心代码**。这意味着您不应通过编辑核心文件来为网站添加功能，因为每次更新时 WordPress 都会覆盖核心文件。任何您想要添加或修改的功能都应通过插件实现。

根据您的需求，WordPress 插件可以非常简单，也可以非常复杂。最简单的插件是单个 PHP 文件。[Hello Dolly](https://wordpress.org/plugins/hello-dolly/ "Hello Dolly 插件") 就是这类插件的典型示例。插件 PHP 文件仅需包含[插件头部信息](https://developer.wordpress.org/plugins/plugin-basics/header-requirements/)、若干 PHP 函数以及用于挂载函数的[钩子](https://developer.wordpress.org/plugins/hooks/)。

插件让您能够在不触及 WordPress 核心代码的前提下，极大地扩展其功能。