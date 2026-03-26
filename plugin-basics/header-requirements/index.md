---
title: "头部要求"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Header Requirements
    - Plugin Basics
    - Repos
---

# 头部要求

如[入门指南](https://developer.wordpress.org/plugins/plugin-basics/#getting-started)所述，主 PHP 文件应包含头部注释，以告知 WordPress 该文件是一个插件并提供插件相关信息。

## 最低要求字段

至少，头部注释必须包含插件名称：

```
/*
 * Plugin Name: 你的插件名称
 */
```

## 头部字段

可用的头部字段：

- **Plugin Name:** (_必需_) 你的插件名称，将显示在 WordPress 管理后台的插件列表中。
- **Plugin URI:** 插件的主页，应是一个唯一的 URL，最好在你自己的网站上。这 _必须是你的插件独有的_。你不能在此处使用 WordPress.org 的 URL。
- **Description:** 插件的简短描述，显示在 WordPress 管理后台的插件部分。请将此描述控制在 140 个字符以内。
- **Version:** 插件的当前版本号，例如 1.0 或 1.0.3。
- **Requires at least:** 插件能正常运行所需的最低 WordPress 版本。
- **Requires PHP:** 所需的最低 PHP 版本。
- **Author:** 插件作者的名字。多个作者可以使用逗号分隔列出。
- **Author URI:** 作者的网站或在其他网站（如 WordPress.org）上的个人资料页。
- **License:** 插件许可证的简称（slug），例如 GPLv2。更多关于许可证的信息可以在 [WordPress.org 指南](https://developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/#1-plugins-must-be-compatible-with-the-gnu-general-public-license) 中找到。
- **License URI:** 指向许可证全文的链接（例如 [https://www.gnu.org/licenses/old-licenses/gpl-2.0.html](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html)）。
- **Text Domain:** 插件的 [gettext](https://www.gnu.org/software/gettext/) 文本域。更多信息可以在 [如何国际化你的插件](https://developer.wordpress.org/plugins/internationalization/how-to-internationalize-your-plugin/) 页面的 [文本域](https://developer.wordpress.org/plugins/internationalization/how-to-internationalize-your-plugin/#text-domains) 部分找到。
- **Domain Path:** 域路径让 WordPress 知道在哪里可以找到翻译文件。更多信息可以在 [如何国际化你的插件](https://developer.wordpress.org/plugins/internationalization/how-to-internationalize-your-plugin/) 页面的 [域路径](https://developer.wordpress.org/plugins/internationalization/how-to-internationalize-your-plugin/#domain-path) 部分找到。
- **Network:** 插件是否只能在整个网络范围内激活。只能设置为 _true_，不需要时应省略。
- **Update URI:** **(_重要：切勿用于托管在 WordPress.org 插件目录中的插件_)** 允许非 WordPress.org 插件避免被 WordPress.org 插件目录中同名插件的更新意外覆盖。更多信息请阅读相关的 [开发说明](https://make.wordpress.org/core/2021/06/29/introducing-update-uri-plugin-header-in-wordpress-5-8/)。

一个包含头部注释的有效 PHP 文件可能如下所示：

```
/*
 * Plugin Name:       My Basics Plugin
 * Plugin URI:        https://example.com/plugins/the-basics/
 * Description:       Handle the basics with this plugin.
 * Version:           1.10.3
 * Requires at least: 5.2
 * Requires PHP:      7.2
 * Author:            John Smith
 * Author URI:        https://author.example.com/
 * License:           GPL v2 or later
 * License URI:       https://www.gnu.org/licenses/gpl-2.0.html
 * Text Domain:       my-basics-plugin
 * Domain Path:       /languages
 */
```

Here's another example which allows file-level PHPDoc DocBlock as well as WordPress plugin file headers:

```
/**
 * Plugin Name
 *
 * @package           PluginPackage
 * @author            Your Name
 * @copyright         2019 Your Name or Company Name
 * @license           GPL-2.0-or-later
 *
 * @wordpress-plugin
 * Plugin Name:       Plugin Name
 * Plugin URI:        https://example.com/plugin-name
 * Description:       Description of the plugin.
 * Version:           1.0.0
 * Requires at least: 5.2
 * Requires PHP:      7.2
 * Author:            Your Name
 * Author URI:        https://example.com
 * Text Domain:       plugin-slug
 * License:           GPL v2 or later
 * License URI:       https://www.gnu.org/licenses/gpl-2.0.txt
 */
```

## 注意事项

[alert]为项目分配版本号时，请注意 WordPress 使用 PHP `version_compare()` 函数来比较插件版本号。因此，在发布插件新版本前，应确保此 PHP 函数认为新版本"大于"旧版本。例如，1.02 实际上大于 1.1。[/alert]