---
title: "高级主题"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Advanced Topics
    - Hooks
    - Repos
---

# 高级主题

## 移除动作与过滤器

有时您需要从其他插件、主题甚至 WordPress 核心已注册的钩子中移除某个回调函数。

若要从钩子中移除回调函数，您需要调用 `remove_action()` 或 `remove_filter()`，具体取决于该回调函数是作为动作还是过滤器添加的。

传递给 `remove_action()` / `remove_filter()` 的参数必须与注册时传递给 `add_action()` / `add_filter()` 的参数完全一致，否则移除操作将无法生效。

[alert]要成功移除回调函数，必须在回调函数注册之后执行移除操作。执行顺序至关重要。[/alert]

## Example

Let's say we want to improve the performance of a large theme by removing unnecessary functionality.

Let's analyze the theme's code by looking into `functions.php`.

```
function wporg_setup_slider() {
  // ...
}
add_action( 'template_redirect', 'wporg_setup_slider', 9 );
```

The `wporg_setup_slider` function is adding a slider that we don't need, which probably loads a huge CSS file followed by a JavaScript initialization file which uses a custom written library the size of 1MB. We can can get rid of that.

Since we want to hook into WordPress after the `wporg_setup_slider` callback function was registered (`functions.php` executed) our best chance would be the [`after_setup_theme`](https://developer.wordpress.org/reference/hooks/after_setup_theme/) hook.

```
function wporg_disable_slider() {
  // Make sure all parameters match the add_action() call exactly.
  remove_action( 'template_redirect', 'wporg_setup_slider', 9 );
}
// Make sure we call remove_action() after add_action() has been called.
add_action( 'after_setup_theme', 'wporg_disable_slider' );
```

## Removing All Callbacks

You can also remove all of the callback functions associated with a hook by using `remove_all_actions()` / `remove_all_filters()`.

## Determining the Current Hook

Sometimes you want to run an Action or a Filter on multiple hooks, but behave differently based on which one is currently calling it.

You can use the `current_action()` / `current_filter()` to determine the current Action / Filter.

```
function wporg_modify_content( $content ) {
  switch ( current_filter() ) {
    case 'the_content':
      // Do something.
      break;
    case 'the_excerpt':
      // Do something.
      break;
  }
  return $content;
}

add_filter( 'the_content', 'wporg_modify_content' );
add_filter( 'the_excerpt', 'wporg_modify_content' );
```

## Checking How Many Times a Hook Has Run

Some hooks are called multiple times in the course of execution, but you may only want your callback function to run once.

In this situation, you can check how many times the hook has run with the [`did_action()`](https://developer.wordpress.org/reference/functions/did_action/).

```
function wporg_custom() {
  // If save_post has been run more than once, skip the rest of the code.
  if ( did_action( 'save_post' ) !== 1 ) {
    return;
  }
  // ...
}
add_action( 'save_post', 'wporg_custom' );
```

## 使用 "all" 钩子进行调试

如果你希望某个回调函数在每一个钩子上都触发，可以将其注册到 `all` 钩子。这在调试场景中有时很有用，可以帮助确定特定事件何时发生或页面何时崩溃。

```
function wporg_debug() {
  echo '<p>' . current_action() . '</p>';
}
add_action( 'all', 'wporg_debug' );
```