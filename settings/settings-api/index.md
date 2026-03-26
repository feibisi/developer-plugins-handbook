---
title: "Settings API"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Settings Api
    - Settings
    - Repos
---

# Settings API

Settings API 于 WordPress 2.7 版本引入，允许以半自动方式管理包含设置表单的管理页面。您可以通过它定义设置页面、页面内的区块以及区块内的字段。

您可以注册新的设置页面及其内部的区块和字段。也可以通过向现有设置页面注册新的设置区块或字段来扩展其功能。

开发者仍需投入精力组织字段的注册与验证工作，但能避免大量底层选项管理的复杂调试。

[alert]使用 Settings API 时，表单将提交至 `wp-admin/options.php` 并执行严格的权限检查。用户需具备 `manage_options` 权限（在多站点环境下必须是超级管理员）才能提交表单。[/alert]

## 为何使用设置 API？

开发者*可以*忽略此 API 并自行编写设置页面。这就引出了一个问题：此 API 能带来什么好处？以下是部分优势的快速概述。

### 视觉一致性

使用 API 生成界面元素可确保您的设置页面与其余管理内容风格统一。您的界面将遵循相同的样式指南，完美融入系统，并且得益于 WordPress 设计师团队的卓越才华，视觉效果将非常出色！

### 健壮性（面向未来！）

由于该 API 是 WordPress 核心的一部分，任何更新都会自动考虑你的插件设置页面。如果你不使用设置 API 而自行创建界面，WordPress 核心更新更有可能破坏你的自定义功能。同时，有更广泛的用户群体在测试和维护该 API 代码，因此它往往更加稳定。

### 省时省力！

最直接的好处当然是 WordPress API 在幕后为你完成了大量工作。除了提供美观集成的设计外，设置 API 还具备以下功能：

- **处理表单提交**：让 WordPress 自动处理 `$_POST` 数据的获取与存储
- **内置安全措施**：免费获得随机数等额外安全防护机制
- **数据净化处理**：可使用与 WordPress 其他模块相同的字符串安全校验方法

## 函数参考

### 设置注册/注销

- [`register_setting()`](https://developer.wordpress.org/reference/functions/register_setting/)
- [`unregister_setting()`](https://developer.wordpress.org/reference/functions/unregister_setting/)

### 添加字段/区块

- [`add_settings_section()`](https://developer.wordpress.org/reference/functions/add_settings_section/)
- [`add_settings_field()`](https://developer.wordpress.org/reference/functions/add_settings_field/)

### Options Form Rendering

- [`settings_fields()`](https://developer.wordpress.org/reference/functions/settings_fields/)
- [`do_settings_sections()`](https://developer.wordpress.org/reference/functions/do_settings_sections/)
- [`do_settings_fields()`](https://developer.wordpress.org/reference/functions/do_settings_fields/)

### Errors

- [`add_settings_error()`](https://developer.wordpress.org/reference/functions/add_settings_error/)
- [`get_settings_errors()`](https://developer.wordpress.org/reference/functions/get_settings_errors/)
- [`settings_errors()`](https://developer.wordpress.org/reference/functions/settings_errors/)