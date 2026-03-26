---
title: "最佳实践"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Best Practices
    - Plugin Basics
    - Repos
---

# 最佳实践

以下是一些最佳实践，可帮助您组织代码，使其与 WordPress 核心和其他 WordPress 插件良好协作。

## 避免命名冲突

当您的插件为变量、函数或类使用了与其他插件相同的名称时，就会发生命名冲突。

幸运的是，您可以通过使用以下方法来避免命名冲突。

## 过程式编码方法

默认情况下，所有变量、函数和类都定义在**全局命名空间**中，这意味着您的插件可能会覆盖其他插件设置的变量、函数和类，反之亦然。在函数或类*内部*定义的变量不受此影响。

### 为所有内容添加前缀

所有全局可访问的代码都应使用一个*唯一*标识符作为前缀。前缀可以防止与其他插件发生冲突，避免它们覆盖你的变量或意外调用你的函数和类。

为了防止与其他插件冲突，你的前缀应至少为 4 个字母，但我们建议使用 5 个。你应该避免使用常见的英文单词，而是选择对你的插件来说独特的东西。仅在 WordPress.org 上我们就托管了数万个插件，在我们的服务器之外还有数十万个。你*必定*会遇到冲突。

一个好的方法是使用前缀。例如，如果你的插件名为 "Easy Custom Post Types"，那么你可以使用以下名称：

- `function ecpt_save_post()`
- `define( 'ECPT_LICENSE', true );`
- `class ECPT_Admin{}`
- `namespace EasyCustomPostTypes;`
- `update_option( 'ecpt_settings', $settings );`

[info]由于你是作为 **WordPress** 项目的一部分编写代码，因此必须避免使用极有可能与 WordPress 核心发生冲突的前缀。这包括但不限于：`__`（双下划线）、`wp_`、`WordPress` 或 `_`（单下划线）。

如果你正在为“子”插件（例如 WooCommerce 扩展）编写代码，同样需要避免使用它们通常/常见的前缀（例如 Woo、WooCommerce）。

你可以在类或命名空间*内部*使用它们，但不能作为独立的函数/命名空间/类。[/info]

如果你使用 `_n()` 或 `__()` 进行翻译，那没问题。我们**仅**讨论你为插件创建的函数，而不是 WordPress 的核心函数。事实上，这些核心功能正是你*不应该*在自己的插件中使用这些前缀的原因！你肯定不希望破坏用户的 WordPress。

请记住：好的前缀名称对你的插件来说是唯一且独特的。这将有助于你和下一个人在调试时，同时防止冲突。

**必须**添加前缀的代码包括：

- 函数（除非已命名空间化）
- 类、接口和特征（除非已命名空间化）
- 命名空间
- 全局变量
- 选项和瞬态数据

### 检查现有实现

PHP 提供了多种函数来验证变量、函数、类和常量是否存在。如果实体存在，所有这些函数都将返回 true。

- **变量**: [isset()](https://www.php.net/manual/en/function.isset.php)（包括数组、对象等）
- **函数**: [function_exists()](https://www.php.net/manual/en/function.function-exists.php)
- **类**: [class_exists()](https://www.php.net/manual/en/function.class-exists.php)
- **常量**: [defined()](https://www.php.net/manual/en/function.defined.php)

请注意，在所有函数和类周围使用 `(!function_exists('NAME')) {` 听起来是个好主意，直到你意识到其致命缺陷。如果其他代码中存在同名函数且其代码先加载，你的插件将会崩溃。使用 if-exists 来替换/覆盖函数或类应仅用于**共享**库。

### Example

```
// Create a function called "wporg_init" if it doesn't already exist
if ( ! function_exists( 'wporg_init' ) ) {
  function wporg_init() {
    register_setting( 'wporg_settings', 'wporg_option_foo' );
  }
}

// Create a function called "wporg_get_foo" if it doesn't already exist
if ( ! function_exists( 'wporg_get_foo' ) ) {
  function wporg_get_foo() {
    return get_option( 'wporg_option_foo' );
  }
}
```

## 面向对象编程方法

解决命名冲突问题的一种更简便方法是使用[类](https://www.php.net/manual/en/language.oop5.php)来组织插件代码。

您仍需注意检查所需类名是否已被占用，其余部分将由 PHP 自动处理。

### Example

```
if ( ! class_exists( 'WPOrg_Plugin' ) ) {
  class WPOrg_Plugin {
    public static function init() {
      register_setting( 'wporg_settings', 'wporg_option_foo' );
    }

    public static function get_foo() {
      return get_option( 'wporg_option_foo' );
    }
  }

  WPOrg_Plugin::init();
  WPOrg_Plugin::get_foo();
}
```

## 文件组织

插件目录的根层级应包含您的 `plugin-name.php` 文件，并可选择性包含 [uninstall.php](https://developer.wordpress.org/plugins/plugin-basics/uninstall-methods/) 文件。所有其他文件应尽可能组织到子文件夹中。

### 文件夹结构

清晰的文件夹结构有助于您和其他参与插件开发的人员将相关文件归类存放。

以下是一个可供参考的示例文件夹结构：

```
/plugin-name
  plugin-name.php
  uninstall.php
  /languages
  /includes
  /admin
    /js
    /css
    /images
  /public
    /js
    /css
    /images
```

## 插件架构

您为插件选择的架构或代码组织方式，很可能取决于插件的大小。

对于与 WordPress 核心、主题或其他插件交互有限的小型单一用途插件，设计复杂的类结构益处不大；除非您知道该插件后续会大幅扩展。

对于包含大量代码的大型插件，请从一开始就考虑使用类。分离样式文件、脚本文件，甚至构建相关文件。这将有助于代码组织和插件的长期维护。

### Conditional Loading

It's helpful to separate your admin code from the public code. Use the conditional [is_admin()](https://developer.wordpress.org/reference/functions/is_admin/). You must still perform capability checks as this doesn't indicate the user is authenticated or has Administrator-level access. See [Checking User Capabilities](https://developer.wordpress.org/plugins/security/checking-user-capabilities/).

For example:

```
if ( is_admin() ) {
  // we are in admin mode
  require_once __DIR__ . '/admin/plugin-name-admin.php';
}
```

### Avoiding Direct File Access

As a security precaution, it's a good practice to disallow access if the `ABSPATH` global is not defined. This is only applicable to files which contain code outside of class or function definitions, such as the main plugin file.

You can implement this by including this code at the top of the file:

```
if ( ! defined( 'ABSPATH' ) ) {
  exit; // Exit if accessed directly
}
```

### 架构模式

虽然存在多种可能的架构模式，但大致可分为三种变体：

- [单一插件文件，包含函数](https://github.com/GaryJones/move-floating-social-bar-in-genesis/blob/master/move-floating-social-bar-in-genesis.php)
- [单一插件文件，包含类、实例化对象及可选函数](https://github.com/norcross/wp-comment-notes/blob/master/wp-comment-notes.php)
- [主插件文件，附带一个或多个类文件](https://github.com/DevinVinson/WordPress-Plugin-Boilerplate)

### 架构模式详解

上述较为复杂的代码组织形式已有具体实现方案，并整理为教程和演示文稿：

- [Slash – 单例模式、加载器、动作、屏幕、处理器](https://jjj.blog/2012/12/slash-architecture-my-approach-to-building-wordpress-plugins/)
- [在 WordPress 插件中实现 MVC 模式](https://iandunn.name/content/presentations/wp-oop-mvc/mvc.php)

## 样板起点

与其为每个新插件从头开始编写，您可能希望从一个**样板**开始。使用样板的一个优势是能保持您自己插件之间的一致性。如果您使用其他人已经熟悉的样板，也更容易让他们为您的代码做出贡献。

这些样板也作为不同但可比较架构的进一步示例。

- [WordPress 插件样板](https://github.com/DevinVinson/WordPress-Plugin-Boilerplate)：一个 WordPress 插件开发的基础，旨在为构建您的插件提供清晰一致的指南。
- [WordPress 插件引导程序](https://github.com/claudiosanches/wordpress-plugin-boilerplate)：使用 Grunt、Compass、GIT 和 SVN 开发 WordPress 插件的基本引导程序。
- [WP 骨架插件](https://github.com/ptahdunbar/wp-skeleton-plugin)：专注于单元测试和使用 composer 进行开发的骨架插件。
- [WP CLI 脚手架](https://developer.wordpress.org/cli/commands/scaffold/plugin/)：WP CLI 的 Scaffold 命令创建一个带有 CI 配置文件等选项的骨架插件。

当然，您可以结合这些以及其他样板的不同方面，创建您自己的自定义样板。