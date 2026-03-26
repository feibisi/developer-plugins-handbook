---
title: "自定义钩子"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Custom Hooks
    - Hooks
    - Repos
---

# 自定义钩子

一项重要但常被忽视的做法是在插件中使用自定义钩子，以便其他开发者能够扩展和修改它。

自定义钩子的创建和调用方式与 WordPress 核心钩子相同。

## 创建钩子

要创建自定义钩子，请对[操作](https://developer.wordpress.org/plugins/hooks/actions/)使用 [`do_action()`](https://developer.wordpress.org/reference/functions/do_action/)，对[过滤器](https://developer.wordpress.org/plugins/hooks/filters/)使用 [`apply_filters()`](https://developer.wordpress.org/reference/functions/apply_filters/)。

[info]我们建议对任何输出到浏览器的文本使用 [`apply_filters()`](https://developer.wordpress.org/reference/functions/apply_filters/)，尤其是在前端。

这使插件能更轻松地根据用户需求进行修改。[/info]

## 为钩子添加回调函数

要为自定义钩子添加回调函数，对[操作](https://developer.wordpress.org/plugins/hooks/actions/)使用[`add_action()`](https://developer.wordpress.org/reference/functions/add_action/)，对[过滤器](https://developer.wordpress.org/plugins/hooks/filters/)使用[`add_filter()`](https://developer.wordpress.org/reference/functions/add_filter/)。

## 命名冲突

当两位开发者将完全不同的功能使用相同的钩子名称时，就会发生命名冲突（"碰撞"）。这会导致难以排查的错误。因此，为你的钩子名称添加唯一的前缀非常重要，以避免与其他插件发生钩子名称冲突。

例如，名为 `email_body` 的过滤器过于通用，两位或更多开发者可能会在不同的插件中为不同目的使用此钩子。为避免这种情况，需要添加前缀。例如，本手册中的示例函数使用 `wporg_` 作为前缀。

选择前缀时，你可以使用公司名称、你的 WordPress 用户名、插件名称，或任何你喜欢的名称。目标是使其具有唯一性，因此请明智选择。

## 示例

### 可扩展操作：设置表单

如果您的插件向管理面板添加了设置表单，您可以使用操作（Actions）来允许其他插件向其中添加它们自己的设置。

```
do_action( 'wporg_after_settings_page_html' );
```

现在，另一个插件可以为 `wporg_after_settings_page_html` 钩子注册一个回调函数并注入新的设置：

```
add_action( 'wporg_after_settings_page_html', 'myprefix_add_settings' );
```

请注意，因为这是一个操作（action），所以不会返回值。同时请注意，由于没有指定优先级，它将以默认优先级 10 运行。

### 可扩展过滤器：自定义文章类型

在此示例中，当注册新的文章类型时，定义它的参数会通过过滤器传递，因此其他插件可以在文章类型创建之前修改这些参数。

```
function wporg_create_post_type() {
  $post_type_params = [/* ... */];

  register_post_type(
    'post_type_slug',
    apply_filters( 'wporg_post_type_params', $post_type_params )
  );
}
```

现在，另一个插件可以为 `wporg_post_type_params` 钩子注册一个回调函数并修改文章类型参数：

```
function myprefix_change_post_type_params( $post_type_params ) {
  $post_type_params['hierarchical'] = true;
  return $post_type_params;
}
add_filter( 'wporg_post_type_params', 'myprefix_change_post_type_params' );
```

请注意，过滤器接收数据、修改数据并返回数据。因此被调用的代码（`myprefix_change_post_type_params`）不会使用 echo 或 html 等方式直接向屏幕输出任何内容。同时注意，返回值直接被 `register_post_type` 使用，而无需先赋值给变量。这样可以简单地跳过那个额外（且不必要）的步骤。

还需注意，由于未指定优先级，它将以默认优先级 10 运行。并且由于未指定期望的参数数量，因此假定默认值为一个。