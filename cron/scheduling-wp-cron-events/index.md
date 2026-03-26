---
title: "调度 WP Cron 事件"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Scheduling Wp Cron Events
    - Cron
    - Repos
---

# 调度 WP Cron 事件

WP Cron 系统使用钩子来添加新的定时任务。

## 添加钩子

为了让你的任务能够运行，你必须创建自己的自定义钩子，并给这个钩子指定要执行的函数名。这是非常重要的一步。如果忘记，你的任务将永远不会运行。

以下示例将创建一个钩子。第一个参数是你正在创建的钩子名称，第二个是要调用的函数名称。

```
add_action( 'bl_cron_hook', 'bl_cron_exec' );
```

[info]请记住，函数名中的 `bl_` 部分是一个 _函数前缀_。你可以[在此](https://developer.wordpress.org/plugins/plugin-basics/best-practices/#prefix-everything)了解为什么前缀很重要。[/info]

[info]你可以[在此](https://developer.wordpress.org/plugins/hooks/actions/)阅读更多关于动作的信息。[/info]

## 任务调度

一个重要注意事项是 WP-Cron 是一个简单的任务调度器。我们知道，任务是通过创建的钩子来添加的，该钩子用于调用运行所需任务的函数。但是，如果您多次调用 `wp_schedule_event()`，即使使用相同的钩子名称，事件也会被多次调度。如果您的代码在每次页面加载时都添加任务，这可能导致任务被调度数千次。这并非您想要的结果。

WordPress 提供了一个便捷的函数 [`wp_next_scheduled()`](https://developer.wordpress.org/reference/functions/wp_next_scheduled/) 来检查特定钩子是否已被调度。`wp_next_scheduled()` 接受一个参数，即钩子名称。它将返回一个包含下次执行时间戳的字符串，或者返回 false，表示任务尚未调度。其用法如下：

```
wp_next_scheduled( 'bl_cron_hook' );
```

调度重复任务是通过 [`wp_schedule_event()`](https://developer.wordpress.org/reference/functions/wp_schedule_event/) 完成的。此函数接受三个必需参数，以及一个额外的参数，该参数是一个数组，可以传递给执行 wp-cron 任务的函数。我们将重点关注前三个参数。参数如下：

1. `$timestamp` – 此任务首次执行的 UNIX 时间戳。
2. `$recurrence` – 任务重复执行的时间间隔名称（以秒为单位）。
3. `$hook` – 要调用的自定义钩子名称。

我们将使用在[此处](https://developer.wordpress.org/plugins/cron/understanding-wp-cron-scheduling/)创建的 5 秒间隔以及上面创建的钩子，如下所示：

```
wp_schedule_event( time(), 'five_seconds', 'bl_cron_hook' );
```

请记住，我们首先需要确保任务尚未被调度。因此，我们将调度代码包装在如下检查中：

```
if ( ! wp_next_scheduled( 'bl_cron_hook' ) ) {
  wp_schedule_event( time(), 'five_seconds', 'bl_cron_hook' );
}
```

## 取消任务调度

当你不再需要某个已调度的任务时，可以使用 [`wp_unschedule_event()`](https://developer.wordpress.org/reference/functions/wp_unschedule_event/) 来取消任务调度。该函数接受以下两个参数：

1. `$timestamp` – 任务下一次执行的时间戳。
2. `$hook` – 要调用的自定义钩子名称。

此函数不仅会取消指定时间戳对应的任务，还会取消该任务所有未来的执行计划。由于你可能不知道下一个任务的时间戳，另一个实用函数 [`wp_next_scheduled()`](https://developer.wordpress.org/reference/functions/wp_next_scheduled/) 可以帮你找到它。`wp_next_scheduled()` 接受一个参数（我们关心的）：

1. `$hook` – 执行任务时调用的钩子名称。

整合起来的代码如下所示：

```
$timestamp = wp_next_scheduled( 'bl_cron_hook' );
wp_unschedule_event( $timestamp, 'bl_cron_hook' );
```

在不再需要任务时取消调度非常重要，因为 WordPress 会继续尝试执行这些任务，即使它们已不再使用（甚至在你的插件已停用或删除后）。一个需要记住取消任务调度的重要时机是在插件停用时。

遗憾的是，WordPress.org 插件目录中有许多插件没有自行清理。如果你发现这样的插件，请告知作者更新他们的代码。WordPress 提供了一个名为 [`register_deactivation_hook()`](https://developer.wordpress.org/reference/functions/register_deactivation_hook/) 的函数，允许开发者在插件停用时运行特定函数。设置非常简单，如下所示：

```
register_deactivation_hook( __FILE__, 'bl_deactivate' ); 

function bl_deactivate() {
  $timestamp = wp_next_scheduled( 'bl_cron_hook' );
  wp_unschedule_event( $timestamp, 'bl_cron_hook' );
}
```

[info]你可以在此处了解更多关于激活和停用钩子的信息[此处](https://developer.wordpress.org/plugins/plugin-basics/activation-deactivation-hooks/)。[/info]