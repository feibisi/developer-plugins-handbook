---
title: "Cron"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Cron
    - Repos
    - Data
---

# Cron

## 什么是 WP-Cron

WP-Cron 是 WordPress 处理基于时间的任务调度的方式。WordPress 核心的多个功能，例如检查更新和发布定时文章，都使用了 WP-Cron。名称中的 "Cron" 部分来源于 UNIX 系统中可用的基于时间的任务调度系统 cron。

WP-Cron 的工作原理是：在每次页面加载时，检查预定的任务列表，以确定需要运行哪些任务。任何计划运行的任务都会在该页面加载期间被调用。

[info]WP-Cron 不像系统 cron 那样持续运行；它仅在页面加载时触发。[/info]

如果你计划在下午 2:00 运行一个任务，但直到下午 5:00 才有页面加载，则可能会出现调度错误。

## 为什么使用 WP-Cron

- WordPress 核心和许多插件需要一个调度系统来执行基于时间的任务。然而，许多托管服务是共享的，不提供对系统调度器的访问。
- 使用 WordPress API 来设置定时任务，比脱离 WordPress 去使用系统调度器更简单。
- 对于系统调度器，如果时间已过而任务未运行，它将不会重新尝试。而对于 WP-Cron，所有预定的任务都会被放入队列，并在下一次有机会时（即下一次页面加载时）运行。因此，虽然你无法 100% 确定任务**何时**运行，但你可以 100% 确定它**最终**会运行。