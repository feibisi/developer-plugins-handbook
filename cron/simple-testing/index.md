---
title: "WP-Cron 测试"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Simple Testing
    - Cron
    - Repos
---

# WP-Cron 测试

## WP-CLI

可以使用 [WP-CLI](https://wp-cli.org/) 测试定时任务。它提供了诸如 `wp cron event list` 和 `wp cron event run {job name}` 等命令。[查看文档](https://developer.wordpress.org/cli/commands/cron/) 获取更多详情。

## WP-Cron 管理插件

WordPress.org 插件目录中提供了多个[插件](https://wordpress.org/plugins/tags/cron/)，用于查看、编辑和控制您网站上的计划定时任务及可用计划。

## `_get_cron_array()`

[`_get_cron_array()`](https://developer.wordpress.org/reference/functions/_get_cron_array/) 函数返回所有当前计划定时任务的数组。如果您需要检查原始事件列表，请使用此函数。

## `wp_get_schedules()`

[`wp_get_schedules()`](https://developer.wordpress.org/reference/functions/wp_get_schedules/) 函数返回可用事件重复计划的数组。如果您需要检查原始可用计划列表，请使用此函数。