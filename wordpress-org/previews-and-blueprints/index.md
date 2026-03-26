---
title: "预览与蓝图"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Previews And Blueprints
    - Wordpress Org
    - Repos
---

# 预览与蓝图

或许您尚未察觉，WordPress Playground 是一项令人惊叹的功能，它允许任何人在浏览器中安全地运行临时 WordPress 安装。它利用 WASM 技术，在您的网页浏览器内完整运行 WordPress 安装——包括 PHP、数据库等所有组件。无需服务器，无需安装任何软件。

目前 Playground 已支持从插件目录加载任意插件或主题；具体方法如下。

## 插件预览按钮

插件预览功能会在插件目录中为插件添加一个便捷按钮（需插件开发者启用）。点击该按钮将跳转至已安装该插件的 Playground 环境。该按钮位于下载按钮旁。

预览按钮默认不显示，必须由开发者明确启用。开发者可通过蓝图文件配置预览环境并安装依赖项（如其他插件和主题）。

## 启用插件预览

要使插件预览按钮对所有用户可见，需要满足两个条件：

1. 必须在插件 `assets` 文件夹的 `blueprints` 子目录中提供一个有效的 `blueprint.json` 文件。
2. 插件提交者必须在插件的“高级”视图中将插件预览设置为“公开”。

如果存在有效的 `blueprint.json` 文件，则预览按钮将仅对插件提交者显示。在这种情况下，它将如下所示：

![“测试预览”按钮允许插件作者一键展示其插件的功能。](https://i0.wp.com/developer.wordpress.org/files/2024/03/live-preview.png?resize=554%2C140&ssl=1)

它之所以被称为“测试预览”，是因为其目的就是允许插件提交者在 Playground 环境中测试其插件，并决定是否将其轻松提供给公众使用。

## 蓝图

蓝图是用于设置 WordPress Playground 实例的 JSON 文件。

它们可用于指定诸如 PHP 和 WordPress 版本、着陆页，以及（最重要的）一系列自动化步骤，例如登录、安装和激活插件与主题。

您插件的蓝图应通过 Subversion 提交到 `assets/blueprints/blueprint.json` 文件。最初仅支持一个蓝图文件，但我们预计未来将允许多个。

以下是一个简单的 `blueprint.json` 文件示例，您可以用作起点：

```
{
    "landingPage": "\/wp-admin\/edit.php",
    "preferredVersions": {
        "php": "7.4",
        "wp": "5.9"
    },
    "phpExtensionBundles": [
        "kitchen-sink"
    ],
    "steps": [
        {
            "step": "login",
            "username": "admin",
            "password": "password"
        }
    ]
}
```

此处使用的功能包括：

`landingPage`，用于指定预览加载时用户将着陆的页面 URL。

`preferredVersions`，用于指定 PHP 和 WordPress 的版本。

`phpExtensionBundles`，在此示例中指定我们希望提供最常见的 PHP 扩展（kitchen-sink）。

`steps`，用于告知 Playground 在显示着陆页之前要执行的操作。在此示例中，它将简单地让用户登录到 wp-admin。

以下是一个更高级的 `blueprint.json` 示例，展示了您可用于为用户创建丰富演示环境的更多功能：

```
{
    "landingPage": "/wp-admin/post.php?post=5&action=edit",
    "preferredVersions": {
        "php": "7.4",
        "wp": "5.9"
    },
    "phpExtensionBundles": [
        "kitchen-sink"
    ],
    "steps": [
        {
            "step": "login",
            "username": "admin",
            "password": "password"
        },
        {
            "step": "installPlugin",
            "pluginZipFile": {
                "resource": "wordpress.org\/plugins",
                "slug": "my-imaginary-plugin-dependency"
            },
            "options": {
                "activate": true
            }
        },
        {
            "step": "installPlugin",
            "pluginZipFile": {
                "resource": "wordpress.org\/plugins",
                "slug": "my-imaginary-plugin"
            },
            "options": {
                "activate": true
            }
        },
        {
            "step": "installTheme",
            "themeZipFile": {
                "resource": "wordpress.org\/themes",
                "slug": "my-imaginary-theme"
            }
        },
        {
            "step": "setSiteOptions",
            "options": {
                "some_required_option_1": "your_favorite_values",
                "some_required_option_2": "your_favorite_values"
            }
        },
        {
            "step": "runPHP",
            "code": "<?php require_once 'wordpress/wp-load.php'; wp_insert_post(array('post_title' => 'wp-load.php required for WP functionality', 'post_status' => 'publish')); ?>"
        }
    ]
}
```

# 使用生成的蓝图文件

您可能会在插件页面上看到类似这样的提示：

```
您的插件尚未包含用于用户预览的蓝图文件。如需启用预览功能，请按以下步骤创建蓝图：

1. 在 Playground 中测试您的插件
2. 修复插件中导致 Playground 运行异常的错误
3. 下载 blueprint.json 文件
4. 将蓝图文件提交至 SVN 仓库
```

__测试__ 链接将使用自动生成的蓝图文件在 Playground 中加载您的插件，其中包含默认配置值和步骤。__下载 blueprint.json__ 链接可让您下载该自动生成的 `blueprint.json` 文件，您可以根据需要修改此文件，并在插件准备就绪后提交至 Subversion 仓库以启用 Playground 预览功能。

# 提交蓝图到 Subversion

你必须将 blueprint.json 文件提交到插件的 assets 文件夹中，命名如下：

`/assets/blueprints/blueprint.json`