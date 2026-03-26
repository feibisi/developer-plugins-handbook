---
title: "使用设置 API"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Using Settings Api
    - Settings
    - Repos
---

# 使用设置 API

## 添加设置

您必须使用 [`register_setting()`](https://developer.wordpress.org/reference/functions/register_setting/) 定义新设置，它将在 `{$wpdb->prefix}_options` 表中创建一个条目。

您可以使用 [`add_settings_section()`](https://developer.wordpress.org/reference/functions/add_settings_section/) 在现有页面上添加新部分。

您可以使用 [`add_settings_field()`](https://developer.wordpress.org/reference/functions/add_settings_field/) 向现有部分添加新字段。

[alert][`register_setting()`](https://developer.wordpress.org/reference/functions/register_setting/) 以及提到的 `add_settings_*()` 函数都应添加到 `admin_init` 操作钩子中。[/alert]

### 添加设置项

```
register_setting(
  string $option_group,
  string $option_name,
  callable $sanitize_callback = ''
);
```

关于所用参数的完整说明，请参阅 [`register_setting()`](https://developer.wordpress.org/reference/functions/register_setting/) 函数参考文档。

### 添加设置区块

```
add_settings_section(
  string $id,
  string $title,
  callable $callback,
  string $page
);
```

区块是您在 WordPress 设置页面上看到的、具有共享标题的设置组。在您的插件中，您可以向现有设置页面添加新区块，而无需创建全新的页面。这使您的插件更易于维护，并为用户减少了需要学习的新页面数量。

有关所用参数的完整说明，请参阅关于 [`add_settings_section()`](https://developer.wordpress.org/reference/functions/add_settings_section/) 的函数参考。

### 添加字段

```
add_settings_field(
  string $id,
  string $title,
  callable $callback,
  string $page,
  string $section = 'default',
  array $args = []
);
```

关于所用参数的完整说明，请参阅 [`add_settings_field()`](https://developer.wordpress.org/reference/functions/add_settings_field/) 的函数参考。

### Example

```
function wporg_settings_init() {
  // register a new setting for "reading" page
  register_setting('reading', 'wporg_setting_name');

  // register a new section in the "reading" page
  add_settings_section(
    'wporg_settings_section',
    'WPOrg Settings Section', 'wporg_settings_section_callback',
    'reading'
  );

  // register a new field in the "wporg_settings_section" section, inside the "reading" page
  add_settings_field(
    'wporg_settings_field',
    'WPOrg Setting', 'wporg_settings_field_callback',
    'reading',
    'wporg_settings_section'
  );
}

/**
 * register wporg_settings_init to the admin_init action hook
 */
add_action('admin_init', 'wporg_settings_init');

/**
 * callback functions
 */

// section content cb
function wporg_settings_section_callback() {
  echo '<p>WPOrg Section Introduction.</p>';
}

// field content cb
function wporg_settings_field_callback() {
  // get the value of the setting we've registered with register_setting()
  $setting = get_option('wporg_setting_name');
  // output the field
  ?>
  <input type="text" name="wporg_setting_name" value="<?php echo isset( $setting ) ? esc_attr( $setting ) : ''; ?>">
  <?php
}
```

## 获取设置

```
get_option(
  string $option,
  mixed $default = false
);
```

获取设置通过 [`get_option()`](https://developer.wordpress.org/reference/functions/get_option/) 函数完成。

该函数接受两个参数：选项的名称和该选项的可选默认值。

### Example

```
// Get the value of the setting we've registered with register_setting()
$setting = get_option('wporg_setting_name');
```