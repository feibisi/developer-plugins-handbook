---
title: "Options API"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Options Api
    - Settings
    - Repos
---

# Options API

Options API 自 WordPress 1.0 版本引入，允许对 WordPress 选项进行创建、读取、更新和删除操作。结合 [Settings API](https://developer.wordpress.org/plugins/settings/settings-api/) 使用，可以控制设置在页面中定义的选项。

## 选项存储位置？

选项存储在 `{$wpdb->prefix}_options` 表中。`$wpdb->prefix` 由 `wp-config.php` 文件中设置的 `$table_prefix` 变量定义。

## 选项如何存储？

选项在 WordPress 数据库中可以两种方式之一存储：作为单个值或作为值数组。

### 单个值

当保存为单个值时，选项名称对应单个值。

```
// 添加新选项
add_option('wporg_custom_option', 'hello world!');
// 获取选项
$option = get_option('wporg_custom_option');
```

### 值数组

当保存为值数组时，选项名称对应一个数组，该数组本身可能包含键/值对。

```
// 选项数组
$data_r = array('title' => 'hello world!', 1, false );
// 添加新选项
add_option('wporg_custom_option', $data_r);
// 获取选项
$options_r = get_option('wporg_custom_option');
// 输出标题
echo esc_html($options_r['title']);
```

如果您需要处理大量相关选项，将它们存储为数组可以对整体性能产生积极影响。

[info]以独立选项形式访问数据可能导致多次独立的数据库事务，通常数据库事务是开销较大的操作（在时间和服务器资源方面）。当您存储或检索选项数组时，它会在单次事务中完成，这是理想的方式。[/info]

## 函数参考

### 添加选项

- [`add_option()`](https://developer.wordpress.org/reference/functions/add_option/)
- [`add_site_option()`](https://developer.wordpress.org/reference/functions/add_site_option/)

### 获取选项

- [`get_option()`](https://developer.wordpress.org/reference/functions/get_option/)
- [`get_site_option()`](https://developer.wordpress.org/reference/functions/get_site_option/)

### 更新选项

- [`update_option()`](https://developer.wordpress.org/reference/functions/update_option/)
- [`update_site_option()`](https://developer.wordpress.org/reference/functions/update_site_option/)

### 删除选项

- [`delete_option()`](https://developer.wordpress.org/reference/functions/delete_option/)
- [`delete_site_option()`](https://developer.wordpress.org/reference/functions/delete_site_option/)