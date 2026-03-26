---
title: "操作"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Actions
    - Hooks
    - Repos
---

# 操作

**操作**是[钩子](https://developer.wordpress.org/plugins/hooks/)的两种类型之一。它们提供了一种在 WordPress 核心、插件和主题执行过程中的特定时刻运行函数的方式。操作的回调函数不会向调用它的操作钩子返回任何内容。它们是[过滤器](https://developer.wordpress.org/plugins/hooks/filters/)的对应物。这里回顾一下[操作与过滤器的区别](https://developer.wordpress.org/plugins/hooks/#actions-vs-filters)。

## 添加动作

添加动作的过程包含两个步骤：

### 创建 _回调函数_

首先，创建一个 _回调函数_。当它挂钩的动作运行时，此函数将被执行。

回调函数与普通函数类似：应添加前缀，并应位于 `functions.php` 或可调用的位置。它应接受的参数由您挂钩的动作定义；大多数钩子都有明确定义，因此请查阅钩子文档以了解所选动作将向您的函数传递哪些参数。

### Assign (_hook_) your callback function

Second, add your callback function to the action. This is called _hooking_ and tells the action to run your callback function when the action is run.

When your callback function is ready, use [add_action()](https://developer.wordpress.org/reference/functions/add_action/) to hook it to the action you have selected. At a minimum, `add_action()` requires two parameters:

1. `string $hook_name` which is the name of the action you're hooking to, and
2. `callable $callback` the name of your callback function.

The example below will run `wporg_callback()` when the `init` hook is executed:

```
function wporg_callback() {
  // do something
}
add_action( 'init', 'wporg_callback' );
```

You can refer to the [Hooks](https://developer.wordpress.org/plugins/hooks/) chapter for a list of available hooks.

As you gain more experience, looking through WordPress Core source code will allow you to find the most appropriate hook.

### 额外参数

`add_action()` 可接受两个额外参数：`int $priority` 用于指定回调函数的优先级，以及 `int $accepted_args` 用于设定传递给回调函数的参数数量。

#### 优先级

可以将多个回调函数挂载到同一个动作钩子上。例如 `init` 钩子就被广泛使用。在某些情况下，你可能需要确保你的回调函数在其他回调函数之前或之后运行，即使那些其他函数可能尚未被挂载。

WordPress 根据两个因素决定回调函数的运行顺序：第一种方式是通过手动设置**优先级**。这通过 `add_action()` 的第三个参数来实现。

以下是关于优先级的一些重要事实：

- 优先级是正整数，通常在 1 到 20 之间
- 默认优先级（即未手动提供 `priority` 值时分配的优先级）为 10
- 优先级值理论上没有上限，但实际使用中通常不超过 100

优先级为 11 的函数将在优先级为 10 的函数**之后**运行；优先级为 9 的函数将在优先级为 10 的函数**之前**运行。

决定回调函数顺序的第二种方式，是在**相同优先级值内**根据注册顺序确定。因此，如果两个回调函数以相同优先级注册到同一个钩子，它们将按照注册顺序依次运行。

例如，以下回调函数都注册到 `init` 钩子，但具有不同的优先级：

```
add_action( 'init', 'wporg_callback_run_me_late', 11 );
add_action( 'init', 'wporg_callback_run_me_normal' );
add_action( 'init', 'wporg_callback_run_me_early', 9 );
add_action( 'init', 'wporg_callback_run_me_later', 11 );
```

在上面的示例中：

- 首先运行的函数是 `wporg_callback_run_me_early()`，因为它的手动优先级为 9
- 接着是 `wporg_callback_run_me_normal()`，因为它未设置优先级，所以采用默认值 10
- 然后是 `wporg_callback_run_me_late()`，因为它的手动优先级为 11
- 最后是 `wporg_callback_run_me_later()`：它的优先级也是 11，但它在 `wporg_callback_run_me_late()` 之后被挂载

#### Number of Arguments

Sometimes it's desirable for a callback function to receive some extra data related to the action being hooked to.

For example, when WordPress saves a post and runs the [`save_post`](https://developer.wordpress.org/reference/hooks/save_post/) hook, it passes two parameters to the callback function: the ID of the post being saved, and the post object itself:

```
do_action( 'save_post', $post->ID, $post );
```

When a callback function is registered for the [`save_post`](https://developer.wordpress.org/reference/hooks/save_post/) hook, it can specify that it wants to receive those two parameters. It does so by telling `add_action` to expect them by (in this case) putting `2` as the fourth argument:

```
add_action( 'save_post', 'wporg_custom', 10, 2 );
```

In order to actually receive those parameters in your callback function, modify the parameters your callback function will accept, like this:

```
function wporg_custom( $post_id, $post ) {
  // do something
}
```

[tip]]It's good practice to give your callback function parameters the same name as the passed parameters, or as close as you can.[/tip]