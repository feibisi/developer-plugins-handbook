---
title: "Debug Bar 和插件"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Debug Bar And Add Ons
    - Developer Tools
    - Repos
---

# Debug Bar 和插件

Debug Bar 是一个插件，它会在管理栏中添加一个调试菜单，显示查询、缓存和其他有用的调试信息。

## Debug Bar

这是主插件，提供了基础功能，本页列出的其他插件会在此基础上进行扩展。

当启用 `WP_DEBUG` 时，它还会跟踪 PHP 警告和通知，以便更容易找到它们。

当启用 `SAVEQUERIES` 时，会跟踪并显示 MySQL 查询。

[访问 Debug Bar](https://wordpress.org/plugins/debug-bar/)

## Debug Bar Console

此插件添加了一个控制台，您可以在其中运行任意 PHP 代码。这对于测试变量内容等众多用途来说非常出色。

[访问 Debug Bar Console](https://wordpress.org/plugins/debug-bar-console/)

## Debug Bar Shortcodes

此插件为 Debug Bar 添加一个新面板，显示当前请求中已注册的短代码。

此外，它还会显示：

- 短代码调用的函数/方法
- 短代码是否在当前文章/页面/文章类型中使用及使用方式（仅在单篇文章页面显示）
- 短代码的任何附加可用信息，例如描述、参数说明、是否为自闭合标签
- 查找使用特定短代码的所有页面/文章等内容

[访问 Debug Bar Shortcodes](https://wordpress.org/plugins/debug-bar-shortcodes/)

## 调试栏常量

此插件为调试栏添加三个新面板，显示当前请求中可供开发者使用的已定义常量：

- WP 常量
- WP 类常量
- PHP 常量

[访问调试栏常量](https://wordpress.org/plugins/debug-bar-constants/)

## Debug Bar 文章类型

此插件为 Debug Bar 添加一个新面板，显示您站点已注册文章类型的详细信息。

[访问 Debug Bar Post Types](https://wordpress.org/plugins/debug-bar-post-types/)

## Debug Bar Cron

此插件在 Debug Bar 中添加一个新面板，用于显示 WordPress 计划任务的相关信息。

安装后，您将可以访问以下信息：

- 计划任务的数量。
- 当前是否正在执行 cron。
- 下一个任务的时间。
- 当前时间。
- 自定义计划任务列表。
- 核心计划任务列表。
- 计划列表。

[访问 Debug Bar Cron](https://wordpress.org/plugins/debug-bar-cron/)

## Debug Bar 操作与过滤器插件

此插件在 Debug Bar 中新增两个标签页，用于显示当前请求所关联的钩子（操作与过滤器）。操作标签页展示当前请求所挂载的操作，过滤器标签页则显示过滤器标签及其关联函数与相应优先级。

[访问 Debug Bar 操作与过滤器插件](https://wordpress.org/plugins/debug-bar-actions-and-filters-addon/)

## 调试栏瞬态数据

此插件将 WordPress 瞬态数据信息添加至调试栏的新面板中。

安装后，您将可访问以下信息：

- 现有瞬态数据数量
- 自定义瞬态数据列表
- 核心瞬态数据列表
- 自定义站点瞬态数据列表
- 核心站点瞬态数据列表
- 删除瞬态数据的选项

[访问调试栏瞬态数据](https://wordpress.org/plugins/debug-bar-transients/)

## 调试栏脚本与样式依赖项列表

此插件可列出已加载的脚本和样式，显示其加载顺序及存在的依赖关系。

[访问调试栏脚本与样式依赖项列表](https://wordpress.org/plugins/debug-bar-list-dependencies/)

## Debug Bar 远程请求

此插件将为 Debug Bar 添加一个新面板，用于显示和分析通过 HTTP API 发起的远程请求。

安装后，您将可以访问以下信息：

- 请求方法（GET、POST 等）
- URL
- 每个请求的耗时
- 所有请求的总耗时
- 请求总数

您还可以在 URL 中添加 `?dbrr_full=1` 来获取额外信息，包括所有请求参数以及包含响应头的完整响应转储。

[访问 Debug Bar 远程请求](https://wordpress.org/plugins/debug-bar-remote-requests/)