---
title: "顶级菜单"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Top Level Menus
    - Administration Menus
    - Repos
---

# 顶级菜单

## 添加顶级菜单

要向 WordPress 管理后台添加新的顶级菜单，请使用 [`add_menu_page()`](https://developer.wordpress.org/reference/functions/add_menu_page/) 函数。

```
add_menu_page(
  string $page_title,
  string $menu_title,
  string $capability,
  string $menu_slug,
  callable $function = '',
  string $icon_url = '',
  int $position = null
);
```

### Example

Lets say we want to add a new Top-level menu called "WPOrg".

**The first step** will be creating a function which will output the HTML. In this function we will perform the necessary security checks and render the options we've registered using the [Settings API](https://developer.wordpress.org/plugins/settings/).

[info]We recommend wrapping your HTML using a `<div>` with a class of `wrap`.[/info]

```
function wporg_options_page_html() {
?>
  <div class="wrap">
    <h1><?php echo esc_html( get_admin_page_title() ); ?></h1>
    <form action="options.php" method="post">
      <?php
      // output security fields for the registered setting "wporg_options"
      settings_fields( 'wporg_options' );
      // output setting sections and their fields
      // (sections are registered for "wporg", each field is registered to a specific section)
      do_settings_sections( 'wporg' );
      // output save settings button
      submit_button( __( 'Save Settings', 'textdomain' ) );
      ?>
    </form>
  </div>
<?php
}
```

**The second step** will be registering our WPOrg menu. The registration needs to occur during the `admin_menu` action hook.

```
add_action( 'admin_menu', 'wporg_options_page' );
function wporg_options_page() {
  add_menu_page(
    'WPOrg',
    'WPOrg Options',
    'manage_options',
    'wporg',
    'wporg_options_page_html',
    plugin_dir_url(__FILE__) . 'images/icon_wporg.png',
    20
  );
}
```

For a list of parameters and what each do please see the [`add_menu_page()`](https://developer.wordpress.org/reference/functions/add_menu_page/) in the reference.

### 使用 PHP 文件生成 HTML

对于可移植代码的最佳实践是创建一个需要/包含你的 PHP 文件的回调函数。

为了完整性和帮助你理解遗留代码，我们将展示另一种方式：将 `PHP 文件路径` 作为 `$menu_slug` 参数传递，并将 `$function` 参数设为 `null`。

```
add_action( 'admin_menu', 'wporg_options_page' );
function wporg_options_page() {
  add_menu_page(
    'WPOrg',
    'WPOrg Options',
    'manage_options',
    plugin_dir_path(__FILE__) . 'admin/view.php',
    null,
    plugin_dir_url(__FILE__) . 'images/icon_wporg.png',
    20
  );
}
```

## 移除顶级菜单

要从 WordPress 管理界面移除已注册的菜单，请使用 [`remove_menu_page()`](https://developer.wordpress.org/reference/functions/remove_menu_page/) 函数。

```
remove_menu_page(
  string $menu_slug
);
```

[warning]移除菜单不会阻止用户直接访问它们。
这绝不应作为限制[用户能力](https://developer.wordpress.org/plugins/users/roles-and-capabilities/)的方式。[/warning]

### 示例

假设我们想要移除"工具"菜单。

```
add_action( 'admin_menu', 'wporg_remove_options_page', 99 );
function wporg_remove_options_page() {
  remove_menu_page( 'tools.php' );
}
```

在尝试移除菜单前，请确保该菜单已通过 `admin_menu` 钩子注册，并为 [`add_action()`](https://developer.wordpress.org/reference/functions/add_action/) 指定更高的优先级数值。

## 提交表单

要在选项页面处理表单提交，你需要完成以下两件事：

1. 使用页面 URL 作为表单的 `action` 属性
2. 添加一个钩子，使用 `add_menu_page` 返回的别名

[info]仅当你在后端手动创建表单时才需遵循这些步骤。推荐使用 [Settings API](https://developer.wordpress.org/plugins/settings/) 实现此功能。[/info]

### 表单操作属性

将选项页面的 `$menu_slug` 参数用作 [`menu_page_url()`](https://developer.wordpress.org/reference/functions/menu_page_url/) 函数的第一个参数。该函数默认会自动转义 URL 并输出，因此你可以直接在 `<form>` 标签内使用它：

```
<form action="<?php menu_page_url( 'wporg' ) ?>" method="post">
```

### 处理表单

在添加页面时指定的 `$function` 仅在显示页面时才会被调用，因此当需要向浏览器发送标头（例如重定向）时，此方法并不适用。

`add_menu_page` 会返回一个 `$hookname`，而 WordPress 会在任何 HTML 输出之前触发 `"load-$hookname"` 动作。你可以利用此机制来分配一个函数，该函数可用于处理表单。

[info]`"load-$hookname"` 将在每次显示选项页面之前执行，即使表单未被提交时也是如此。[/info]

考虑到返回参数和动作，上面的示例将如下所示：

```
add_action( 'admin_menu', 'wporg_options_page' );
function wporg_options_page() {
  $hookname = add_menu_page(
    'WPOrg',
    'WPOrg Options',
    'manage_options',
    'wporg',
    'wporg_options_page_html',
    plugin_dir_url(__FILE__) . 'images/icon_wporg.png',
    20
  );

  add_action( 'load-' . $hookname, 'wporg_options_page_submit' );
}
```

你可以根据需求编写 `wporg_options_page_submit` 函数，但请记住必须手动执行所有必要的检查，包括：

1. 表单是否正在提交（`'POST' === $_SERVER['REQUEST_METHOD']`）。
2. [CSRF 验证](https://developer.wordpress.org/apis/security/nonces/)
3. 验证
4. 数据清理