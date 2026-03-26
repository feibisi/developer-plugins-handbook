---
title: "确定插件和内容目录"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Determining Plugin And Content Directories
    - Plugin Basics
    - Repos
---

# 确定插件和内容目录

在编写 WordPress 插件时，您经常需要引用 WordPress 安装目录以及您的插件或主题中的各种文件和文件夹。

WordPress 提供了多个函数来轻松确定特定文件或目录的位置。在您的插件中应始终使用这些函数，而不是硬编码对 wp-content 目录的引用或使用 WordPress 内部常量。

[tip]WordPress 允许用户将 wp-content 目录放置在任意位置并随意重命名。切勿假设插件一定位于 wp-content/plugins、上传文件一定位于 wp-content/uploads，或主题一定位于 wp-content/themes。[/tip]

PHP 的 `__FILE__` 魔术常量会自动解析符号链接，因此如果 `wp-content/`、`wp-content/plugins/` 甚至单个插件目录被符号链接，硬编码的路径将无法正常工作。

## 常用方法

如果您的插件包含 JavaScript 文件、CSS 文件或其他外部文件，您可能需要这些文件的 URL 以便将其加载到页面中。为此，您应该使用 [plugins_url()](https://developer.wordpress.org/reference/functions/plugins_url/) 函数，如下所示：

```
plugins_url( 'myscript.js', __FILE__ );
```

这将返回 myscript.js 的完整 URL，例如 `example.com/wp-content/plugins/myplugin/myscript.js`。

要将插件的 JavaScript 或 CSS 加载到页面中，您应分别使用 [`wp_enqueue_script()`](https://developer.wordpress.org/reference/functions/wp_enqueue_script/) 或 [`wp_enqueue_style()`](https://developer.wordpress.org/reference/functions/wp_enqueue_style/)，并将 `plugins_url()` 的结果作为文件 URL 传入。

## 可用函数

WordPress 包含许多其他函数，用于确定插件、主题和 WordPress 自身内部文件或目录的路径和 URL。有关其用法的完整信息，请参阅每个函数的独立 DevHub 页面。

### Plugins

- [`plugins_url()`](https://developer.wordpress.org/reference/functions/plugins_url/)
- [`plugin_dir_url()`](https://developer.wordpress.org/reference/functions/plugin_dir_url/)
- [`plugin_dir_path()`](https://developer.wordpress.org/reference/functions/plugin_dir_path/)
- [`plugin_basename()`](https://developer.wordpress.org/reference/functions/plugin_basename/)

### Themes

- [`get_template_directory_uri()`](https://developer.wordpress.org/reference/functions/get_template_directory_uri/)
- [`get_stylesheet_directory_uri()`](https://developer.wordpress.org/reference/functions/get_stylesheet_directory_uri/)
- [`get_stylesheet_uri()`](https://developer.wordpress.org/reference/functions/get_stylesheet_uri/)
- [`get_theme_root_uri()`](https://developer.wordpress.org/reference/functions/get_theme_root_uri/)
- [`get_theme_root()`](https://developer.wordpress.org/reference/functions/get_theme_root/)
- [`get_theme_roots()`](https://developer.wordpress.org/reference/functions/get_theme_roots/)
- [`get_stylesheet_directory()`](https://developer.wordpress.org/reference/functions/get_stylesheet_directory/)
- [`get_template_directory()`](https://developer.wordpress.org/reference/functions/get_template_directory/)

### Site Home

- [`home_url()`](https://developer.wordpress.org/reference/functions/home_url/)
- [`get_home_path()`](https://developer.wordpress.org/reference/functions/get_home_path/)

### WordPress

- [`admin_url()`](https://developer.wordpress.org/reference/functions/admin_url/)
- [`site_url()`](https://developer.wordpress.org/reference/functions/site_url/)
- [`content_url()`](https://developer.wordpress.org/reference/functions/content_url/)
- [`includes_url()`](https://developer.wordpress.org/reference/functions/includes_url/)
- [`wp_upload_dir()`](https://developer.wordpress.org/reference/functions/wp_upload_dir/)

### Multisite

- [`get_admin_url()`](https://developer.wordpress.org/reference/functions/get_admin_url/)
- [`get_home_url()`](https://developer.wordpress.org/reference/functions/get_home_url/)
- [`get_site_url()`](https://developer.wordpress.org/reference/functions/get_site_url/)
- [`network_admin_url()`](https://developer.wordpress.org/reference/functions/network_admin_url/)
- [`network_site_url()`](https://developer.wordpress.org/reference/functions/network_site_url/)
- [`network_home_url()`](https://developer.wordpress.org/reference/functions/network_home_url/)

## 常量

WordPress 在确定内容和插件目录路径时会使用以下常量。插件或主题不应直接使用这些常量，但为保持完整性，此处列出它们。

```
WP_CONTENT_DIR  // 无尾部斜杠，仅完整路径
WP_CONTENT_URL  // 完整 URL
WP_PLUGIN_DIR  // 完整路径，无尾部斜杠
WP_PLUGIN_URL  // 完整 URL，无尾部斜杠

// 多站点默认可用，单站点安装中未设置
// 可在单站点安装中使用（通常：风险自负）
UPLOADS // （若设置，上传文件夹，相对于 ABSPATH）（例如：/wp-content/uploads）
```

## Related

### WordPress Directories

| Function | | Directory |
| --- | --- | --- |
| [`home_url()`](https://developer.wordpress.org/reference/functions/home_url/) | Home URL | `https://www.example.com` |
| [`site_url()`](https://developer.wordpress.org/reference/functions/site_url/) | Site directory URL | `https://www.example.com` or `https://www.example.com/wordpress` |
| [`admin_url()`](https://developer.wordpress.org/reference/functions/admin_url/) | Admin directory URL | `https://www.example.com/wp-admin` |
| [`includes_url()`](https://developer.wordpress.org/reference/functions/includes_url/) | Includes directory URL | `https://www.example.com/wp-includes` |
| [`content_url()`](https://developer.wordpress.org/reference/functions/content_url/) | Content directory URL | `https://www.example.com/wp-content` |
| [`plugins_url()`](https://developer.wordpress.org/reference/functions/plugins_url/) | Plugins directory URL | `https://www.example.com/wp-content/plugins` |
| [`wp_upload_dir()`](https://developer.wordpress.org/reference/functions/wp_upload_dir/) | Upload directory URL (returns an array) | `https://www.example.com/wp-content/uploads` |