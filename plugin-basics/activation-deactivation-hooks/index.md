---
title: "激活/停用钩子"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Activation Deactivation Hooks
    - Plugin Basics
    - Repos
---

# 激活/停用钩子

激活和停用钩子提供了在插件激活或停用时执行操作的方法。

- 在**激活**时，插件可以运行例程来添加重写规则、添加自定义数据库表或设置默认选项值。
- 在**停用**时，插件可以运行例程来删除临时数据，例如缓存和临时文件及目录。

[alert]停用钩子有时会与[卸载钩子](https://developer.wordpress.org/plugins/plugin-basics/uninstall-methods/)混淆。卸载钩子最适合**永久删除所有数据**，例如删除插件选项和自定义表等。[/alert]

## 激活

要设置激活钩子，请使用 [register\_activation\_hook()](https://developer.wordpress.org/reference/functions/register_activation_hook/) 函数：

```
register_activation_hook(
  __FILE__,
  'pluginprefix_function_to_run'
);
```

## 停用

要设置停用钩子，请使用 [register_deactivation_hook()](https://developer.wordpress.org/reference/functions/register_deactivation_hook/) 函数：

```
register_deactivation_hook(
  __FILE__,
  'pluginprefix_function_to_run'
);
```

这些函数的第一个参数均指向你的主插件文件，即你放置了[插件头部注释](https://developer.wordpress.org/plugins/plugin-basics/header-requirements/)的文件。通常这两个函数会在主插件文件中触发；但如果函数被放置在其他文件中，你必须更新第一个参数以正确指向主插件文件。

## Example

One of the most common uses for an activation hook is to refresh WordPress permalinks when a plugin registers a custom post type. This gets rid of the nasty 404 errors.

Let's look at an example of how to do this:

```
/**
 * Register the "book" custom post type
 */
function pluginprefix_setup_post_type() {
  register_post_type( 'book', ['public' => true ] ); 
} 
add_action( 'init', 'pluginprefix_setup_post_type' );

/**
 * Activate the plugin.
 */
function pluginprefix_activate() { 
  // Trigger our function that registers the custom post type plugin.
  pluginprefix_setup_post_type(); 
  // Clear the permalinks after the post type has been registered.
  flush_rewrite_rules(); 
}
register_activation_hook( __FILE__, 'pluginprefix_activate' );
```

If you are unfamiliar with registering custom post types, don't worry – this will be covered later. This example is used simply because it's very common.

Using the example from above, the following is how to reverse this process and deactivate a plugin:

```
/**
 * Deactivation hook.
 */
function pluginprefix_deactivate() {
  // Unregister the post type, so the rules are no longer in memory.
  unregister_post_type( 'book' );
  // Clear the permalinks to remove our post type's rules from the database.
  flush_rewrite_rules();
}
register_deactivation_hook( __FILE__, 'pluginprefix_deactivate' );
```

For further information regarding activation and deactivation hooks, here are some excellent resources:

- [register\_activation\_hook()](https://developer.wordpress.org/reference/functions/register_activation_hook/) in the WordPress function reference.
- [register\_deactivation\_hook()](https://developer.wordpress.org/reference/functions/register_deactivation_hook/) in the WordPress function reference.