---
title: "插件基础"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Plugin Basics
    - Repos
    - Data
---

# 插件基础

## 快速入门

最简单的 WordPress 插件就是一个带有 WordPress 插件头部注释的 PHP 文件。强烈建议您创建一个目录来存放插件，以便将所有插件文件整齐地组织在一个位置。

请按照以下步骤开始创建新插件。

1. 进入 WordPress 安装目录的 `wp-content/` 文件夹。
2. 打开 `plugins/` 目录。
3. 创建一个新目录，并以插件名称命名（例如 `plugin-name/`）。
4. 打开新建的插件目录。
5. 创建一个新的 PHP 文件（建议以插件名称命名此文件，例如 `plugin-name.php`）。

以下是在 Unix 命令行中执行此过程的示例：

```bash
$ cd wp-content/
$ cd plugins/
$ mkdir plugin-name/
$ cd plugin-name/
$ vim plugin-name.php
```

在上面的示例中，`vim` 是文本编辑器的名称。您可以使用任何您习惯的编辑器。

现在您正在编辑新插件的 PHP 文件，需要添加一个插件头部注释。这是一个特殊格式的 PHP 块注释，包含插件的元数据，例如名称、作者、版本、许可证等。插件头部注释必须符合[头部要求](https://developer.wordpress.org/plugins/plugin-basics/header-requirements/)，并且至少包含插件的名称。

插件文件夹中只能有一个文件包含头部注释——如果插件有多个 PHP 文件，则只有一个文件应包含头部注释。

保存文件后，您应该能在 WordPress 站点中看到您的插件。登录您的 WordPress 站点，在 WordPress 管理后台的左侧导航栏中点击 **插件**。此页面会显示 WordPress 站点拥有的所有插件列表。您的新插件现在应该出现在该列表中！

## 钩子：动作与过滤器

WordPress 钩子允许你在特定节点介入 WordPress，无需修改任何核心文件即可改变其运行方式。

WordPress 包含两种钩子类型：*动作*与*过滤器*。动作允许你添加或修改 WordPress 功能，而过滤器则让你能在内容加载和展示给网站用户时对其进行调整。

钩子并非仅为插件开发者设计；WordPress 核心自身就广泛使用钩子来提供默认功能。其余钩子则是预留的接入点，当你需要调整 WordPress 工作方式时可直接使用。这正是 WordPress 如此灵活的关键所在。

### 基础钩子

创建插件时所需的 3 个基础钩子是 [register_activation_hook()](https://developer.wordpress.org/reference/functions/register_activation_hook/)、[register_deactivation_hook()](https://developer.wordpress.org/reference/functions/register_deactivation_hook/) 和 [register_uninstall_hook()](https://developer.wordpress.org/reference/functions/register_uninstall_hook/)。

[激活钩子](https://developer.wordpress.org/plugins/plugin-basics/activation-deactivation-hooks/)在您*激活*插件时运行。您可以用它来提供设置插件的函数——例如，在 `options` 表中创建一些默认设置。

[停用钩子](https://developer.wordpress.org/plugins/plugin-basics/activation-deactivation-hooks/)在您*停用*插件时运行。您可以用它来提供清除插件存储的任何临时数据的函数。

这些[卸载方法](https://developer.wordpress.org/plugins/plugin-basics/uninstall-methods/)用于在通过 WordPress 后台*删除*插件后进行清理。您可以用它来删除插件创建的所有数据，例如添加到 `options` 表中的任何选项。

### 添加钩子

您可以使用 [do_action()](https://developer.wordpress.org/reference/functions/do_action/) 添加自定义钩子，这将使开发者能够通过您的钩子传递函数来扩展插件。

### 移除钩子

你也可以使用 [remove_action()](https://developer.wordpress.org/reference/functions/remove_action/) 来移除之前定义的函数。例如，如果你的插件是另一个插件的附加组件，你可以使用 [remove_action()](https://developer.wordpress.org/reference/functions/remove_action/) 并指定与前一个插件通过 [add_action()](https://developer.wordpress.org/reference/functions/add_action/) 添加的相同回调函数。在这些情况下，动作的优先级很重要，因为 [remove_action()](https://developer.wordpress.org/reference/functions/remove_action/) 需要在初始的 [add_action()](https://developer.wordpress.org/reference/functions/add_action/) 之后运行。

移除钩子上的动作或更改优先级时应格外小心，因为这些改动可能难以预见会如何影响同一钩子的其他交互。我们强烈建议频繁进行测试。

你可以在本手册的[钩子](https://developer.wordpress.org/plugins/hooks/)章节了解更多关于创建钩子及与其交互的信息。

## WordPress API

您知道 WordPress 提供了多种[应用程序编程接口（API）](https://make.wordpress.org/core/handbook/best-practices/core-apis/)吗？这些 API 可以极大地简化您在插件中需要编写的代码。您肯定不想重复造轮子，尤其是在许多人已经为您完成了大量工作和测试的情况下。

最常用的是[选项 API](https://codex.wordpress.org/Options_API)，它能让您轻松地将插件数据存储到数据库中。如果您考虑在插件中使用 [cURL](https://en.wikipedia.org/wiki/CURL)，那么[HTTP API](https://developer.wordpress.org/plugins/http-api/) 可能会引起您的兴趣。

既然我们讨论的是插件，您应该研究一下[插件 API](https://codex.wordpress.org/Plugin_API)。它提供了多种函数，将协助您开发插件。

## WordPress 如何加载插件

当 WordPress 在管理后台的插件页面加载已安装插件列表时，它会搜索 `plugins/` 文件夹（及其子文件夹）以查找包含 WordPress 插件头部注释的 PHP 文件。如果你的整个插件仅由一个 PHP 文件组成，例如 [Hello Dolly](https://wordpress.org/plugins/hello-dolly/ "Hello Dolly")，该文件可直接位于 `plugins/` 文件夹的根目录中。但更常见的情况是，插件文件会存放在以插件命名的专属文件夹内。

## 分享你的插件

有时你创建的插件仅用于自己的网站。但许多人喜欢与 WordPress 社区分享他们的插件。在分享插件之前，你需要做的一件事是[选择许可证](https://choosealicense.com/)。这能让插件的使用者了解他们被允许如何使用你的代码。为保持与 WordPress 核心的兼容性，建议选择与 GNU 通用公共许可证（GPLv2+）兼容的许可证。