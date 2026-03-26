---
title: "过滤器"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Filters
    - Hooks
    - Repos
---

# 过滤器

**过滤器**是两种[钩子](https://developer.wordpress.org/plugins/hooks/)类型之一。

它们为函数提供了一种在 WordPress 核心、插件和主题执行期间修改数据的方式。它们是[动作](https://developer.wordpress.org/plugins/hooks/actions/)的对应部分。

与[动作](https://developer.wordpress.org/plugins/hooks/actions/)不同，过滤器应以独立方式工作，且不应产生[副作用](https://en.wikipedia.org/wiki/Side_effect_(computer_science))，例如影响全局变量和输出。过滤器期望有返回值。

## 添加过滤器

添加过滤器的过程包括两个步骤。

首先，您需要创建一个回调函数，该函数将在过滤器运行时被调用。其次，您需要将回调函数添加到一个将执行该函数调用的钩子中。

您将使用 [`add_filter()`](https://developer.wordpress.org/reference/functions/add_filter/) 函数，并至少传递两个参数：

1. `string $hook_name`：您要挂钩的过滤器名称，以及
2. `callable $callback`：您的回调函数名称。

以下示例将在 `the_title` 过滤器执行时运行。

```
function wporg_filter_title( $title ) {
  return 'The ' . $title . ' was filtered';
}
add_filter( 'the_title', 'wporg_filter_title' );
```

假设我们有一个文章标题 "Learning WordPress"，上述示例将把它修改为 "The Learning WordPress was filtered"。

您可以参考[钩子](https://developer.wordpress.org/plugins/hooks/)章节以获取可用钩子的列表。

随着经验的积累，查看 WordPress 核心源代码将帮助您找到最合适的钩子。

### 附加参数

[`add_filter()`](https://developer.wordpress.org/reference/functions/add_filter/) 可以接受两个附加参数：`int $priority` 用于指定回调函数的优先级，以及 `int $accepted_args` 用于指定将传递给回调函数的参数数量。

有关这些参数的详细说明，请阅读关于[动作](https://developer.wordpress.org/plugins/hooks/actions/)的文章。

### 示例

在满足特定条件时为 `<body>` 标签添加 CSS 类：

```
function wporg_css_body_class( $classes ) {
  if ( ! is_admin() ) {
    $classes[] = 'wporg-is-awesome';
  }
  return $classes;
}
add_filter( 'body_class', 'wporg_css_body_class' );
```