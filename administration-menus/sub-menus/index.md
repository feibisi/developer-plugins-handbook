---
title: "子菜单"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Sub Menus
    - Administration Menus
    - Repos
---

# 子菜单

## 添加子菜单

要向 WordPress 管理后台添加新的子菜单，请使用 `add_submenu_page()` 函数。

```
add_submenu_page(
  string $parent_slug,
  string $page_title,
  string $menu_title,
  string $capability,
  string $menu_slug,
  callable $function = ''
);
```

### Example

Lets say we want to add a Sub-menu "WPOrg Options" to the "Tools" Top-level menu.

**The first step** will be creating a function which will output the HTML. In this function we will perform the necessary security checks and render the options we’ve registered using the [Settings API](https://developer.wordpress.org/plugins/settings/).

[info]We recommend wrapping your HTML using a `<div>` with a class of `wrap`.[/info]

```
function wporg_options_page_html() {
  // check user capabilities
  if ( ! current_user_can( 'manage_options' ) ) {
    return;
  }
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

**The second step** will be registering our WPOrg Options Sub-menu. The registration needs to occur during the `admin_menu` action hook.

```
function wporg_options_page() {
  add_submenu_page(
    'tools.php',
    'WPOrg Options',
    'WPOrg Options',
    'manage_options',
    'wporg',
    'wporg_options_page_html'
  );
}
add_action( 'admin_menu', 'wporg_options_page' );
```

For a list of parameters and what each do please see the [`add_submenu_page()`](https://developer.wordpress.org/reference/functions/add_submenu_page/) in the reference.

## Predefined Sub-Menus

Wouldn’t it be nice if we had helper functions that define the `$parent_slug` for WordPress built-in Top-level menus and save us from manually searching it through the source code?

Below is a list of parent slugs and their helper functions:

- [`add_dashboard_page()`](https://developer.wordpress.org/reference/functions/add_dashboard_page/) – `index.php`
- [`add_posts_page()`](https://developer.wordpress.org/reference/functions/add_posts_page/) – `edit.php`
- [`add_media_page()`](https://developer.wordpress.org/reference/functions/add_media_page/) – `upload.php`
- [`add_pages_page()`](https://developer.wordpress.org/reference/functions/add_pages_page/) – `edit.php?post_type=page`
- [`add_comments_page()`](https://developer.wordpress.org/reference/functions/add_comments_page/) – `edit-comments.php`
- [`add_theme_page()`](https://developer.wordpress.org/reference/functions/add_theme_page/) – `themes.php`
- [`add_plugins_page()`](https://developer.wordpress.org/reference/functions/add_plugins_page/) – `plugins.php`
- [`add_users_page()`](administration-menus/sub-menus/hhttps:/developer.wordpress.org/reference/functions/add_users_page) – `users.php`
- [`add_management_page()`](https://developer.wordpress.org/reference/functions/add_management_page/) – `tools.php`
- [`add_options_page()`](https://developer.wordpress.org/reference/functions/add_options_page/) – `options-general.php`
- [`add_options_page()`](https://developer.wordpress.org/reference/functions/add_options_page/) – `settings.php`
- [`add_links_page()`](https://developer.wordpress.org/reference/functions/add_links_page/) – `link-manager.php` – requires a plugin since WP 3.5+
- Custom Post Type – `edit.php?post_type=wporg_post_type`
- Network Admin – `settings.php`

## 移除子菜单

移除子菜单的流程与[移除顶级菜单](https://developer.wordpress.org/plugins/administration-menus/top-level-menus/#remove-a-top-level-menu)完全相同。

## 提交表单

在子菜单中处理表单提交的过程与[顶级菜单中的表单提交](https://developer.wordpress.org/plugins/administration-menus/top-level-menus/#submitting-forms)完全相同。

`add_submenu_page()` 以及所有预定义子菜单的函数（`add_dashboard_page`、`add_posts_page` 等）都会返回一个 `$hookname`，你可以将其用作 `add_action` 的第一个参数，以便在自定义页面中处理表单提交：

```
function wporg_options_page() {
  $hookname = add_submenu_page(
    'tools.php',
    'WPOrg Options',
    'WPOrg Options',
    'manage_options',
    'wporg',
    'wporg_options_page_html'
    );

  add_action( 'load-' . $hookname, 'wporg_options_page_html_submit' );
}

add_action( 'admin_menu', 'wporg_options_page' );
```

一如既往，不要忘记检查表单是否正在提交，进行 CSRF 验证、[数据验证](https://developer.wordpress.org/apis/security/data-validation/)和[数据清理](https://developer.wordpress.org/apis/security/sanitizing/)。