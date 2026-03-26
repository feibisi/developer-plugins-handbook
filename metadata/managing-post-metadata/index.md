---
title: "管理文章元数据"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Managing Post Metadata
    - Metadata
    - Repos
---

# 管理文章元数据

## 添加元数据

使用 [`add_post_meta()`](https://developer.wordpress.org/reference/functions/add_post_meta/) 可以非常轻松地添加元数据。该函数接受 `post_id`、`meta_key`、`meta_value` 和一个 `unique` 标志。

`meta_key` 是您的插件在代码其他地方引用元数据值的方式。像 `mycrazymetakeyname` 这样的名称也可以使用，但使用与您的插件或主题相关的前缀，后跟键的描述会更实用。例如 `wporg_featured_menu` 可能是一个不错的选择。需要注意的是，同一个 `meta_key` 可以多次使用以存储元数据的变体（请参阅下面的唯一标志）。

`meta_value` 可以是字符串、整数或数组。如果是数组，它将在存储到数据库之前自动序列化。

`unique` 标志允许您声明此键是否应唯一。**非**唯一键是文章可以拥有多个变体的键，例如价格。

如果您只希望文章有**一个**价格，则应将其标记为 `unique`，这样 `meta_key` 将只有一个值。

## 更新元数据

如果某个键已存在且您想要更新它，请使用 [`update_post_meta()`](https://developer.wordpress.org/reference/functions/update_post_meta/)。如果您使用此函数而该键**不**存在，那么它将创建该键，就像您使用了 [`add_post_meta()`](https://developer.wordpress.org/reference/functions/add_post_meta/) 一样。

与 [`add_post_meta()`](https://developer.wordpress.org/reference/functions/add_post_meta/) 类似，该函数接受 `post_id`、`meta_key` 和 `meta_value`。它还接受一个可选的 `prev_value` 参数——如果指定了该参数，将导致函数仅更新具有此值的现有元数据条目。如果未提供此参数，函数默认更新所有条目。

## 删除元数据

[`delete_post_meta()`](https://developer.wordpress.org/reference/functions/delete_post_meta/) 函数接收 `post_id`、`meta_key` 参数，以及可选的 `meta_value` 参数。其功能正如其名所示。

## 字符转义

文章元数据值在存储时会经过 [`stripslashes()`](https://www.php.net/manual/en/function.stripslashes.php) 函数处理，因此在传入可能包含转义字符的值（例如 JSON）时需要特别注意。

以 JSON 值 `{"key":"value with \"escaped quotes\""}` 为例：

```
$escaped_json = '{"key":"value with \"escaped quotes\""}';
update_post_meta( $id, 'escaped_json', $escaped_json );
$broken = get_post_meta( $id, 'escaped_json', true );
/*
$broken 经过 stripslashes() 处理后变得无法解析：
{"key":"value with "escaped quotes""}
*/
```

### 解决方法

通过使用函数 [`wp_slash()`](https://developer.wordpress.org/reference/functions/wp_slash/)（在 WP 3.6 中引入）添加额外一层转义，可以抵消对 [`stripslashes()`](https://www.php.net/manual/en/function.stripslashes.php) 的调用：

```
$escaped_json = '{"key":"value with \"escaped quotes\""}';
update_post_meta( $id, 'double_escaped_json', wp_slash( $escaped_json ) );
$fixed = get_post_meta( $id, 'double_escaped_json', true );
/*
$fixed 经过 stripslashes() 处理后，最终得到期望的结果：
{"key":"value with \"escaped quotes\""}
*/
```

## 隐藏自定义字段

如果您是插件或主题开发者，并计划使用自定义字段存储参数，请注意：WordPress 不会在文章编辑界面的自定义字段列表中显示 `meta_key` 以下划线 "_" 开头的字段，使用 [`get_post_meta()`](https://developer.wordpress.org/reference/functions/get_post_meta/) 模板函数时也不会显示。

这有助于通过 [`add_meta_box()`](https://developer.wordpress.org/reference/functions/add_meta_box/) 函数以特殊方式展示这些自定义字段。

以下示例将添加一个独特的自定义字段，其 `meta_key` 为 '_color'，`meta_value` 为 'red'，但该字段不会显示在文章编辑界面：

```
add_post_meta( 68, '_color', 'red', true );
```

### 隐藏数组

此外，如果 `meta_value` 是一个数组，即使你没有在 `meta_key` 名称前添加下划线，它也不会在页面编辑屏幕上显示。