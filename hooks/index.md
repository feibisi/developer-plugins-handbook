---
title: "钩子"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Hooks
    - Repos
    - Data
---

# 钩子

钩子是一种让代码在特定预定义位置与其他代码交互/修改的方式。它们构成了插件和主题与 WordPress 核心交互的基础，同时核心自身也广泛使用钩子。

钩子有两种类型：[动作](https://developer.wordpress.org/plugins/hooks/actions/)和[过滤器](https://developer.wordpress.org/plugins/hooks/filters/)。要使用其中任何一种，您需要编写一个称为“回调函数”的自定义函数，然后将其注册到 WordPress 的特定动作或过滤器钩子。

[动作](https://developer.wordpress.org/plugins/hooks/actions/)允许您添加数据或改变 WordPress 的运行方式。动作会在 WordPress 核心、插件和主题执行的特定时刻运行。动作的回调函数可以执行某些任务，例如向用户输出内容或将数据插入数据库。动作的回调函数不会向调用它的动作钩子返回任何内容。

[过滤器](https://developer.wordpress.org/plugins/hooks/filters/)让您能够在 WordPress 核心、插件和主题执行期间修改数据。过滤器的回调函数会接收一个变量，修改它，然后将其返回。它们应以独立的方式工作，不应产生[副作用](https://en.wikipedia.org/wiki/Side_effect_(computer_science))，例如影响全局变量或输出。过滤器期望有内容返回给它们。

WordPress 提供了许多可用的钩子，但您也可以[创建自己的钩子](https://developer.wordpress.org/plugins/hooks/custom-hooks/)，以便其他开发者可以扩展和修改您的插件或主题。

## 动作与过滤器的区别

动作和过滤器的主要区别可以这样概括：

- 动作接收信息，对其进行某些操作，然后不返回任何内容。换句话说：它对某物执行操作，然后退出，不向调用钩子返回任何内容。
- 过滤器接收信息，以某种方式修改它，然后将其返回。换句话说：它过滤某物并将其传回钩子供后续使用。

另一种表述：

- 动作中断代码流程以执行某些操作，然后返回正常流程，不修改任何内容；
- 过滤器用于以特定方式修改某物，以便后续代码使用修改后的结果。

这里提到的“某物”是指通过钩子定义传递的参数列表。更多细节将在后续章节中介绍。

## 更多资源

- [过滤器参考](https://codex.wordpress.org/Plugin_API/Filter_Reference)
- [动作参考](https://codex.wordpress.org/Plugin_API/Action_Reference)