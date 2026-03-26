---
title: "卸载方法"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Uninstall Methods
    - Plugin Basics
    - Repos
---

# 卸载方法

当您的插件从网站卸载时，可能需要执行一些清理操作。

如果用户已停用插件，然后在 WordPress 管理后台点击删除链接，该插件即被视为已卸载。

当您的插件被卸载时，您需要清除所有插件特定的选项和/或设置，以及/或其他数据库实体（如表）。

经验不足的开发人员有时会错误地为此目的使用停用钩子。

下表说明了停用与卸载之间的区别。

| 场景 | 停用钩子 | 卸载钩子 |
| --- | --- | --- |
| 清除缓存/临时文件 | 是 | 否 |
| 刷新固定链接 | 是 | 否 |
| 从 {[$wpdb](https://developer.wordpress.org/reference/classes/wpdb/)→prefix}_options 表中移除选项 | 否 | 是 |
| 从 [$wpdb](https://developer.wordpress.org/reference/classes/wpdb/) 中移除表 | 否 | 是 |

## 方法一：register_uninstall_hook

要设置卸载钩子，请使用 [register\_uninstall\_hook()](https://developer.wordpress.org/reference/functions/register_uninstall_hook/) 函数：

```
register_uninstall_hook(
  __FILE__,
  'pluginprefix_function_to_run'
);
```

## 方法二：uninstall.php

要使用此方法，您需要在插件的根文件夹中创建一个 `uninstall.php` 文件。当用户删除插件时，此特殊文件会自动运行。

例如：`/plugin-name/uninstall.php`

[alert]在 `uninstall.php` 中执行任何操作之前，请务必检查常量 `WP_UNINSTALL_PLUGIN` 是否存在。这可以防止直接访问。

该常量将在 `uninstall.php` 调用期间由 WordPress 定义。

当通过 [register\_uninstall\_hook()](https://developer.wordpress.org/reference/functions/register_uninstall_hook/) 执行卸载时，该常量**不会**被定义。[/alert]

以下是一个删除选项条目和删除数据库表的示例：

```
// 如果 uninstall.php 不是由 WordPress 调用，则退出
if ( ! defined( 'WP_UNINSTALL_PLUGIN' ) ) {
  die;
}

$option_name = 'wporg_option';

delete_option( $option_name );

// 针对多站点中的站点选项
delete_site_option( $option_name );

// 删除自定义数据库表
global $wpdb;
$wpdb->query( "DROP TABLE IF EXISTS {$wpdb->prefix}mytable" );
```

[info]在多站点环境中，遍历所有博客以删除选项可能会非常消耗资源。[/info]