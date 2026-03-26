---
title: "常见问题"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Common Issues
    - Wordpress Org
    - Repos
---

# 常见问题

本文档整理了插件审核团队在审查插件时遇到的一些最常见问题。

此列表包含团队邮件内容的摘录，不应被视为完整或详尽；审核结果取决于团队执行的人工审查。

## 安全

### 数据净化

**输入数据必须经过净化、验证，并在输出时进行转义**

当您在插件中使用 POST/GET/REQUEST/FILE 调用时，对它们进行净化、验证和转义非常重要。这样做的目的是防止用户意外通过系统发送垃圾数据，并保护他们免受潜在的安全问题。

**净化**：输入的数据（无论是用户输入还是自动生成）必须尽快进行净化。这可以减少 XSS 漏洞和 MITM 攻击的可能性，这些攻击可能篡改提交的数据。

**验证**：所有数据都应进行验证，无一例外。即使您进行了净化，也要记住，当唯一有效值是数字时，您不希望有人输入“dog”。

**转义**：输出的数据在回显时必须正确转义，以防止其劫持管理界面。WordPress 提供了许多 esc_*() 函数，您可以使用它们来确保不会向用户显示错误的数据。

为了帮助您完成这些操作，WordPress 内置了许多净化和转义函数。您可以在此处阅读相关内容：

https://developer.wordpress.org/apis/security/sanitizing/
https://developer.wordpress.org/apis/security/escaping/

请记住：您必须根据上下文使用最合适的函数。如果您要净化电子邮件，请使用 sanitize_email()；如果您要输出 HTML，请使用 wp_kses_post()，依此类推。

一个简单的口诀是：

**尽早净化**  
**最后转义**  
**始终验证**

净化一切，检查一切，转义一切，永远不要相信用户总是会输入合理的数据。毕竟，用户来自各行各业。

#### 清理：关于转义和清理函数的混淆

**注意**：转义函数不能用于清理。它们用途不同。即使它们看起来非常适合此目的，大多数函数都是可过滤的，人们期望用它们来转义。因此，另一个插件可能会改变它们的功能，使您的代码面临风险并可能被利用。

如果您尝试输出变量，必须先清理再转义，例如：

```php
echo esc_html(sanitize_text_field($_POST['example']));
```

#### 清理：使用过滤函数进行数据清理

**注意**：当使用 `filter_var`、`filter_var_array`、`filter_input` 和/或 `filter_input_array` 等函数时，你需要[将 FILTER 参数设置为任何能清理输入的过滤器类型](https://www.php.net/manual/en/filter.filters.php)。

如果过滤器参数留空，PHP 默认会应用过滤器 "FILTER_DEFAULT"，**它完全不进行清理**。

```php
$post_id = filter_input(INPUT_GET, 'post_id', FILTER_SANITIZE_NUMBER_INT);
```

#### 清理：随机数

**注意**：使用 `wp_verify_nonce` 检查随机数时，需要使用 `wp_unslash` **和** `sanitize_text_field` 清理输入，[这是因为该函数是可插拔的，扩展者不应信任其输入值](https://developer.wordpress.org/news/2023/08/understand-and-use-wordpress-nonces-properly/#verifying-the-nonce)。

示例：
```php
if ( ! isset( $_POST['prefix_nonce'] ) || ! wp_verify_nonce( sanitize_text_field( wp_unslash ( $_POST['prefix_nonce'] ) ) , 'prefix_nonce' ) )
```

### 处理完整输入

我们强烈建议您**永远不要**尝试处理整个 $_POST/$_REQUEST/$_GET 堆栈。这会使您的插件变慢，因为您在不必要地遍历不需要的数据。相反，您应该只尝试处理插件运行所需的项目。

### 转义

**输出变量和选项时必须进行转义**

与全面清理数据密切相关，所有被输出的变量在输出时都需要进行转义，以防止其劫持用户界面或（更糟的）管理员界面。WordPress 提供了许多 esc_*() 函数，可确保您不会向用户显示错误数据，同时也有一些函数能让您安全地输出 HTML。

目前，我们要求您在**输出所有 $-变量、选项以及任何类型的生成数据时**进行转义。这意味着您不应在构建变量时转义，而应在最终输出时进行转义。我们称之为"延迟转义"。

除了保护自己免受潜在的 XSS 漏洞攻击外，延迟转义还能确保您为未来的自己提供安全保障。虽然当前您的代码可能只输出硬编码内容，但未来情况可能并非如此。通过在**输出时**花时间进行适当转义，您可以防止未来的错误演变为严重的安全问题。

对于已保存到数据库的选项，这一原则同样适用。即使您在保存时已进行适当清理，清理和转义的工具也不能互换使用。清理确保数据在处理和存储到数据库时是安全的，而转义则确保输出时的安全性。

还需注意，有时函数本应返回内容，却错误地进行了输出。这在返回 JSON 编码内容时是一个常见错误。实际上，您很少需要直接输出此类内容。输出是因为内容需要显示在屏幕上供人阅读，而返回（如 API 处理方式）可以进行 JSON 编码——但请记住在保存到 JSON 对象时要进行**清理**！

WordPress 提供了多种选项来保护各类内容（HTML、电子邮件等）。是的，即使是 HTML 也需要进行适当转义。

[https://developer.wordpress.org/apis/security/escaping/](https://developer.wordpress.org/apis/security/escaping/)

请记住：必须根据上下文使用最合适的函数。几乎每种可输出的内容都有对应的转义选项，即使是安全输出 HTML 也不例外。

#### 转义：esc_url_raw 的使用

我们知道这很令人困惑，`esc_url_raw` 函数并非转义函数，而是类似于 `sanitize_url` 的清理函数。具体来说，它用于清理 URL 以便在数据库或重定向中使用。

用于转义 URL 的合适函数是 `esc_url`。

#### 转义：__ 函数的使用

`__` 函数获取翻译文本时不进行转义，请选择以下任一方式：

- 使用能对结果值进行转义的其他函数，例如 `esc_html__` 或 `esc_attr__`。
- 或者用适当的转义函数包装 `__` 函数，例如 `esc_html`、`esc_attr`、`wp_kses_post` 等。

示例：
```php
<h2><?php echo esc_html__('Settings page', 'plugin-slug'); ?></h2>
<h2><?php echo esc_html(__('Settings page', 'plugin-slug')); ?></h2>
```

#### 转义：_e 和 _ex 的使用

函数 `_e` 和 `_ex` 会直接输出翻译内容而不进行转义，请使用能对输出进行转义的替代函数。

- `_e` 的替代方案可以是 `esc_html_e`、`esc_attr_e`，或者简单地在 `echo` 语句中使用 `__` 函数并用转义函数包裹。
- `_ex` 的替代方案可以是在 `echo` 语句中使用 `_x` 函数并用转义函数包裹。

示例：
```php
<h2><?php esc_html_e('Settings page', 'plugin-slug'); ?></h2>
<h2><?php echo esc_html(__('Settings page', 'plugin-slug')); ?></h2>
<h2><?php echo esc_html(_x('Settings page', 'Settings page title', 'plugin-slug')); ?></h2>
```

#### 转义：使用 json_encode

当你需要输出 JSON 时，最好使用 `wp_json_encode` 函数，同时确保在第二个参数中传递的选项不会绕过转义。

```php
echo wp_json_encode($array_or_object);
```

#### 转义：HTML

在转义时，有时您的插件需要输出 HTML。这可以使用函数 `wp_kses_post` 或 `wp_kses` 来完成。函数 `wp_kses_post` 将允许任何可以放在文章内容中的常见 HTML，`wp_kses` 将允许您使用其第二个和第三个参数设置的任何 HTML，请[参考其文档](https://developer.wordpress.org/reference/functions/wp_kses/)。

一个常见的错误是使用 `esc_html` 来转义 HTML。此函数并非用于此目的，它旨在转义将**放入** HTML 标签内的输出，因此它会去除任何 HTML 标签。

示例：
```php
echo wp_kses_post($html_content);
echo wp_kses($html_content, array( 'a', 'div', 'span' ));
```

#### 转义：使用净化函数

净化函数不能用于转义。它们用途不同。即使它们看起来非常适合此目的，但大多数函数是可过滤的，人们期望用它们来净化。因此，其他插件可能会改变它们的功能，使您的插件面临风险并可能被利用。

如果您要输出变量，必须先净化再转义，例如：

```php
echo esc_html(sanitize_text_field($_POST['example']));
```

### 文件

#### 文件：使用 WordPress 文件上传器

**请使用 WordPress 的文件上传器**

当插件使用 `move_uploaded_file()` 时，其上传的文件将绕过 WordPress 内置函数的检查与平衡机制。因此，您应当使用内置函数：

[https://developer.wordpress.org/reference/functions/wp_handle_upload/](https://developer.wordpress.org/reference/functions/wp_handle_upload/)

#### 文件：未过滤的上传

**不允许使用 ALLOW_UNFILTERED_UPLOADS。**

将此常量设置为 true 将允许用户上传任何类型的文件（包括 PHP 和其他可执行文件），从而造成严重的安全隐患。作为开发者，我们不应在任何逻辑中使用或允许使用此常量，即使在条件语句中也不行。

WordPress 包含一个安全文件列表，您可以在函数 [wp_get_mime_types](https://developer.wordpress.org/reference/functions/wp_get_mime_types) 中查看。

如果您需要添加列表中未包含且不会带来安全风险的特定文件，可以使用 [upload_mimes](https://developer.wordpress.org/reference/hooks/upload_mimes/) 过滤器来实现。

#### 文件：远程调用文件

禁止将图片、js、css 和其他脚本卸载到你的服务器或任何远程服务（如 Google、MaxCDN、jQuery.com 等）。当你调用远程数据时，会引入对另一个站点不必要的依赖。如果你调用的文件不是 WordPress 核心的一部分，那么你应该将其 -本地- 包含在你的插件中，而不是远程调用。如果该文件已包含在 WordPress 核心中，请直接调用它。

此规则的一个例外是，如果你的插件正在执行一项服务。我们将根据具体情况允许这样做。由于这可能令人困惑，我们提供了一些不被允许的示例：

- 将 jQuery CSS 文件卸载到 Google——你应该将 CSS 包含在你的插件中。
- 插入带有帮助文档的 iframe——首选链接或将文档包含在你的插件中。
- 从你自己的域名调用图片——它们应该包含在你的插件中。

以下是一些我们允许的示例：

- 从 Google 或其批准的 CDN 调用字体族（如果与 GPL 兼容）
- 向你的服务器进行 API 调用以处理可能的垃圾评论（如 Akismet）
- 将评论卸载到你自己的服务器（如 Disqus）
- 向服务提供商进行 oEmbed 调用（如 Twitter 或 YouTube）

请从你的插件中移除外部依赖，并尽可能将所有文件包含在插件内（而不是远程调用）。如果你认为你提供的是服务，请以解释服务、被调用的服务器以及是否需要账户连接的方式重写你的 readme.txt。

### 库

#### 库：使用开发版本

**使用库的 Beta / Alpha / 开发版本**

除非插件功能必需，否则我们不建议使用库的测试版本。您应当使用库最稳定的发布版本。

若因技术原因必须使用测试版本，请说明理由。否则，请将库更改为稳定版本。

#### 库：不再维护

**不再维护的库不允许使用**

我们不再接受使用任何开发者已停止支持或维护的库，因为它们会带来重大的安全风险。请考虑其他选项。

#### 库文件：已过时

您使用的至少一个第三方库已过时。请升级至最新稳定版本以获得更好的支持与安全性。我们不建议您使用测试版。

### WPDB：不安全的 SQL 调用

在进行数据库调用时，保护代码免受 SQL 注入漏洞的影响至关重要。您需要更新代码，在查询中使用 wpdb 调用和 prepare() 方法以增强安全性。

请查阅以下资料：

*   [https://developer.wordpress.org/reference/classes/wpdb/#protect-queries-against-sql-injection-attacks](https://developer.wordpress.org/reference/classes/wpdb/#protect-queries-against-sql-injection-attacks)
*   [https://codex.wordpress.org/Data\_Validation#Database](https://codex.wordpress.org/Data_Validation#Database)
*   [https://make.wordpress.org/core/2012/12/12/php-warning-missing-argument-2-for-wpdb-prepare/](https://make.wordpress.org/core/2012/12/12/php-warning-missing-argument-2-for-wpdb-prepare/)
*   [https://ottopress.com/2013/better-know-a-vulnerability-sql-injection/](https://ottopress.com/2013/better-know-a-vulnerability-sql-injection/)

#### WPDB：占位符数组

**注意**：使用占位符向 wpdb::prepare 传递单个值相当简单，但如果我们需要传递一个值数组呢？

你需要为数组中的每个项目创建一个占位符，并将所有对应的值传递给这些占位符，这看起来有些棘手，但这里有一个代码片段可以实现。

```php
$wordcamp_id_placeholders = implode( ', ', array_fill( 0, count( $wordcamp_ids ), '%d' ) );
$prepare_values = array_merge( array( $new_status ), $wordcamp_ids );
$wpdb->query( $wpdb->prepare( "
    UPDATE `$table_name`
    SET `post_status` = %s
    WHERE ID IN ( $wordcamp_id_placeholders )",
    $prepare_values
) );
```
有一个核心工单可能会在未来让这变得更容易：[https://core.trac.wordpress.org/ticket/54042](https://core.trac.wordpress.org/ticket/54042)

### 请勿使用 HEREDOC-NOWDOC

**请勿在您的插件中使用 HEREDOC 或 NOWDOC 语法**

虽然这两种语法完全有效，并且在许多方面是 PHP 中用于输出内容的理想特性，但对于大多数插件来说，其代价过高。

主要问题在于，当您使用 HEREDOC 或 NOWDOC 时，大多数（如果不是全部）代码检查工具将无法检测到代码中缺少转义的情况。尽管有解决方法，但最终结果会破坏所有可读性，留下一团混乱且无法被正确扫描的代码。

我们认为此处的风险远大于收益，因此不允许使用它们。

### 直接文件访问

**允许直接访问插件文件**

直接文件访问发生在有人直接查询 PHP 文件时。这可以通过在浏览器的 URL 栏中输入文件的完整路径，或直接向文件发送 POST 请求来实现。

对于仅包含类或函数定义的文件，直接访问时发生异常情况的风险很小。然而，对于包含可执行代码（例如，函数调用、类实例创建、类方法调用或包含其他 PHP 文件）的文件，安全问题的风险难以预测，因为它取决于具体情况，但风险可能存在且可能很高。

您可以通过在所有可能因直接访问而执行代码的 PHP 文件顶部添加以下代码来轻松防止这种情况：

```php
if ( ! defined( 'ABSPATH' ) ) exit; // 如果直接访问则退出
```

## 兼容性

### 前缀命名

**通用函数/类/定义/命名空间/选项名称**

所有插件必须具有唯一的函数名、命名空间、定义、类和选项名称。这能防止您的插件与其他插件或主题发生冲突。我们需要您更新插件，使用更独特且易于区分的名称。

一个很好的方法是使用前缀。例如，如果您的插件名为“Easy Custom Post Types”，那么可以使用以下名称：

- function ecpt_save_post()
- class ECPT_Admin{}
- namespace ECPT;
- update_option( 'ecpt_settings', $settings );
- define( 'ECPT_LICENSE', true );
- global $ecpt_options;

请不要再尝试使用两个（2）或三个（3）字母的前缀。仅在 WordPress.org 上我们就托管了近十万个插件，在我们服务器之外还有数万个。请相信我们，您会遇到冲突。

您还需要避免使用 __（双下划线）、wp_ 或 _（单下划线）作为前缀。这些是为 WordPress 本身保留的。您可以在类内部使用它们，但不能作为独立函数的前缀。

请记住，如果您使用 _n() 或 __() 进行翻译，这是可以的。我们**仅**讨论您为插件创建的函数，而不是 WordPress 的核心函数。事实上，正是因为这些核心功能，您才不应该在自己的插件中使用这些前缀！您不希望破坏用户的 WordPress 体验。

与此相关的是，在所有函数和类周围使用 if (!function_exists('NAME')) { 听起来是个好主意，直到您意识到致命的缺陷。如果其他代码有相同名称的函数并且其代码先加载，您的插件将会崩溃。if-exists 应仅用于共享库。

请记住：良好的前缀名称对您的插件来说是独特且易于区分的。这将有助于您和下一位开发人员进行调试，并防止冲突。

### PHP

#### PHP: Do not use Short Tags

The primary issue with PHP's short tags is that PHP managed to choose a tag (`<?`) that was used by another syntax: XML. This is a big issue when you consider how common XML parsing and management is.

We know as of PHP 5.4, `<?= ... ?>` tags are supported everywhere, regardless of short tags settings. This should mean they're safe to use in portable code but in reality that has proven not to be the case. Add on to that the fact that many codesniffers won't detect lack of escaping in code when you use short-tags, and it becomes not worth the headache for anyone.

Basically the risk here is way higher than the benefits, which is why we don't permit their use.

#### PHP：全局修改设置

**请勿强制全局设置 PHP 配置**

虽然许多插件可能需要 PHP 的最佳配置，但我们恳请您不要将其设为全局默认值。

在全局范围内运行类似 `ini_set('memory_limit', '-1');` 的定义（例如在 init 或代码的 `__construct()` 部分），意味着您将为网站上的所有内容运行此设置，这可能导致您的用户违反其主机上的任何限制或约束。

如果必须使用这些设置，您需要将其严格限制在仅确实需要的特定函数中。

#### PHP：设置默认时区

这通常不是一个好主意。用户应该能够在 WordPress 中定义自己的时区。

此外，WordPress 明确设置并期望默认时区为 UTC（在 settings.php 中），且日期/时间函数有时依赖于默认时区为 UTC 这一事实。例如，如果你执行 date_default_timezone_set(get_option('timezone_string'))，然后稍后尝试从 get_post_time() 或 get_post_modified_time() 获取 GMT 时间戳，它将无法给出正确的日期。

#### PHP：错误报告

**不要在生产代码中使用错误报告**

虽然 error_reporting() 是 PHP 中一个很棒的工具（[https://www.php.net/manual/en/function.error-reporting.php](https://www.php.net/manual/en/function.error-reporting.php)），但如果你在插件中永久设置它，就会给所有使用你代码的人带来麻烦。如果他们需要调试恰好使用了你代码的网站，他们将无法获得干净的测试结果，因为你在干扰输出。这在插件的日常功能中毫无用处。

### 插件标准

#### 主文件命名规范

**插件主文件的名称不符合规范。**

我们期望插件的主文件（包含插件头信息的文件）与插件文件夹同名，同时也应与插件的 slug / 固定链接同名。

例如，如果您的插件 slug 是 `ecpt-social-manager`，我们期望您的主插件文件名为 `ecpt-social-manager.php`。

请注意，使用某些常见名称作为主插件文件的文件名，在某些配置中可能会导致问题。

请查看我们关于如何[在插件中组织文件和文件夹](https://developer.wordpress.org/plugins/plugin-basics/best-practices/#folder-structure)的建议。

#### 不完整的文件头

您的插件文件头信息缺失或不完整。

请查阅[文件头要求](https://developer.wordpress.org/plugins/the-basics/header-requirements/)并相应更新您的插件，确保文件头信息仅存在于主文件中。

#### 不完整的自述文件

您的自述文件缺失或不完整。

在某些情况下，例如对于首个插件、有依赖项的插件或调用外部服务的插件，我们要求您提供完整的自述文件。这意味着您的自述文件必须包含标题、适当的描述以及关于其工作原理和使用方法的文档。

我们这样做的目标是确保每个人在安装前都清楚他们将要安装什么以及需要做什么。避免意外情况。如果您的插件正在调用其他服务器，这一点尤其重要。您需要在用户安装插件之前为他们提供所有必要的信息。

您的自述文件还必须通过 [Validator](https://wordpress.org/plugins/about/validator/) 验证，否则我们将拒绝它。请记住，我们不希望看到 readme.MD 文件。虽然它们可能有效，但 readme.txt 文件始终会被优先考虑，并且并非所有 Markdown 语法都能按预期工作。

我们请您基于此创建您的自述文件：[https://wordpress.org/plugins/readme.txt](https://wordpress.org/plugins/readme.txt)

#### 未声明 GPL 兼容许可证

必须声明此插件的许可证。您可以使用插件自述文件和插件头部中提供的字段来完成此操作。

请注意，所有代码、数据和图像——即存储在 WordPress.org 托管的插件目录中的任何内容——都必须符合 GPL 或 GPL 兼容许可证。包含的第三方库、代码、图像或其他内容也必须兼容。

有关兼容许可证的具体列表，[请阅读 gnu.org 上的 GPL 兼容许可证列表](https://www.gnu.org/licenses/license-list.html#GPLCompatibleLicenses)。

#### 错误的稳定标签

在您的 readme 文件中，'Stable Tag' 与主插件文件中指示的插件版本不匹配。

您的稳定标签应代表插件的稳定版本，而非 WordPress 版本。为了让您的插件能从 WordPress.org 正确下载，这些值必须保持一致。如果它们不同步，您的用户将无法获取正确的代码版本。

我们建议您使用语义化版本控制（即 SemVer）来管理版本：

- [软件版本控制](https://en.wikipedia.org/wiki/Software_versioning)
- [SemVer](https://semver.org/)

请注意：虽然目前在插件目录中使用 trunk 的稳定标签仍能工作，但这实际上并非受支持或推荐的指示新版本的方法，且已知会导致自动更新出现问题。

我们请您在发布新版本时正确使用标签并递增它们，就像更新主文件中的插件版本一样。保持两者匹配是实现完全向前兼容的最佳方式。

#### 声明的许可证不匹配

声明此插件的许可证时，必须保持一致。

请确保您在自述文件和插件头部声明的是相同的许可证。

此插件可以包含来自其他来源的代码，只要这些代码有完整文档记录且遵循 GPL 或 GPL 兼容许可证，但您的代码需要声明一个唯一的许可证。

### 使用 HTTP API

**使用 CURL 替代 HTTP API**

WordPress 自带一个功能全面的 HTTP API，应优先使用它而非自行创建 curl 调用。它既更快速又更全面。必要时它会回退到 curl，但会优先利用大量 WordPress 原生功能。

[https://developer.wordpress.org/plugins/http-api/](https://developer.wordpress.org/plugins/http-api/)

如有需要，您可以通过 [https://developer.wordpress.org/reference/hooks/http_api_curl/](https://developer.wordpress.org/reference/hooks/http_api_curl/) 使用 setopt。

请注意：如果您在第三方供应商库中使用 CURL，这是允许的。需要修正的是您插件专属代码（或任何专门的 WordPress 库）中的使用情况。

### 使用 wp_enqueue 命令

您的插件未能正确引入 JS 和/或 CSS。您应当使用内置函数来实现此功能：

当引入 **JavaScript 代码** 时，您可以使用：

- wp_register_script() 和 [wp_enqueue_script()](https://developer.wordpress.org/reference/functions/wp_enqueue_script/) 来从文件添加 JavaScript 代码。
- [wp_add_inline_script()](https://developer.wordpress.org/reference/functions/wp_add_inline_script/) 来为先前声明的脚本添加内联 JavaScript 代码。

当引入 **CSS** 时，您可以使用：

- wp_register_style() 和 [wp_enqueue_style()](https://developer.wordpress.org/reference/functions/wp_enqueue_style/) 来从文件添加 CSS。
- [wp_add_inline_style()](https://developer.wordpress.org/reference/functions/wp_add_inline_style/) 来为先前声明的 CSS 添加内联 CSS。

请注意，自 WordPress 5.7 起，您可以通过新的函数和过滤器传递如 async、nonce 和 type 等属性：[WordPress 5.7 中的脚本属性相关函数](https://make.wordpress.org/core/2021/02/23/introducing-script-attributes-related-functions-in-wordpress-5-7/)

如果您尝试在管理页面中加载资源，您需要使用管理端的加载钩子。

- [admin_enqueue_scripts](https://developer.wordpress.org/reference/hooks/admin_enqueue_scripts/)
- [admin_print_scripts](https://developer.wordpress.org/reference/hooks/admin_print_scripts/)
- [admin_print_styles](https://developer.wordpress.org/reference/hooks/admin_print_styles/)

### 包含核心中已有的库

您的插件包含了一个 WordPress 已包含的库的副本。

WordPress 包含了许多有用的库，例如 jQuery、Atom Lib、SimplePie、PHPMailer、PHPass 等。出于安全性和稳定性考虑，插件不应在其自身代码中包含这些库，而必须使用 WordPress 打包的版本。

您可以在此处查看 JS 库列表：

[WordPress 包含并注册的默认脚本和 JS 库](https://developer.wordpress.org/reference/functions/wp_enqueue_script/#default-scripts-and-js-libraries-included-and-registered-by-wordpress)

虽然我们目前还没有一个像样的公开页面来列出所有这些库，但我们在此处有一个列表：

[核心贡献者](https://meta.trac.wordpress.org/browser/sites/trunk/api.wordpress.org/public_html/core/credits/wp-59.php#L739)

本地包含核心中没有的附加组件是可以的，但请仅添加那些额外的文件。例如，您不需要为了一个文件而包含整个 jQuery UI 库。如果您的代码与内置的 jQuery 版本不兼容，很可能是一个 noConflict 问题。

### 国际化

#### 国际化：使用变量

**国际化：请勿将变量或定义用作文本、上下文或文本域参数。**

为了让插件中的字符串可翻译，您需要使用一组特殊函数。这些函数统称为"gettext"。

WordPress 社区中有一个[专门的团队](https://make.wordpress.org/polyglots/)负责翻译并帮助其他译者将 WordPress 核心、插件和主题的字符串翻译成其他语言。

为了让翻译团队能够翻译此插件，请**不要在 gettext 函数的文本、上下文或文本域参数中使用变量或函数调用**，所有这些参数**都必须是字符串**。请注意，翻译解析器读取代码时不会执行它，因此无法读取这些函数中非字符串的内容。

例如，如果您的 gettext 函数如下所示...

`esc_html__( $greetings , 'plugin-slug' );`

...翻译者将看不到任何可翻译的内容，因为 `$greetings` 不是字符串，无法翻译。
您需要提供要翻译的字符串，这样他们才能在翻译系统中看到并翻译它，正确的写法如下...

`esc_html__( 'Hello, how are you?' , 'plugin-slug' );`

这也适用于翻译域，以下是一个错误的调用：

`esc_html__( 'Hello, how are you?' , $plugin_slug );`

正确的写法应该是：

`esc_html__( 'Hello, how are you?' , 'plugin-slug' );`

同时请注意，翻译域必须与您的插件别名相同。

如果要在翻译中包含动态值怎么办？很简单，您需要添加一个占位符作为字符串的一部分，并在 gettext 函数完成其工作后替换它，可以使用 `printf` 来实现，如下所示：

```php
printf(
    /* translators: %s: 用户的名字 */
    esc_html__( 'Hello %s, how are you?', 'plugin-slug' ),
    esc_html( $user_firstname )
);
```

您可以在此处[阅读更多信息](https://developer.wordpress.org/plugins/internationalization/how-to-internationalize-your-plugin/#text-domains)。

## 合规性

### 更改插件激活状态

插件不允许更改其他插件的激活状态，此操作必须由用户执行。

同样不允许在用户激活或停用插件时进行干扰，除非是为了防止错误（例如：当您的插件依赖另一个插件时，若该插件未激活，则停用您自己的插件）。

WordPress 6.5 引入了[插件依赖](https://make.wordpress.org/core/2024/03/05/introducing-plugin-dependencies-in-wordpress-6-5/)，您可以使用它来管理依赖关系（尽管将其用作备用方案也是可以的）。

### 更新检查器

**包含更新检查器/更改更新功能**

请移除您插件中用于提供更新的检查代码。

我们不允许插件向其他服务器"拨号回家"检查更新，因为 WordPress.org 托管已为您提供此项服务。我们的指导原则之一是要求您实际使用我们的托管服务，因此需要您移除相关代码。

同时我们要求插件不要干扰内置更新器，否则在 WordPress 5.5 及以上版本中会导致用户遇到意外结果。

### 未记录的第三方服务

**未记录的第三方或外部服务使用**

我们允许插件要求使用第三方（即外部）服务，前提是这些服务已以清晰的方式妥善记录。

我们要求连接其他服务的插件以清晰易懂的语言披露此情况，以便用户了解数据被发送至何处。这使他们能够确保数据传输涉及的任何法律问题得到覆盖。即使您就是该第三方服务提供商，此要求同样适用。

为此，您必须更新您的自述文件以执行以下操作：

- 明确说明您的插件依赖第三方服务及其使用场景
- 提供该服务的链接
- 提供该服务的使用条款和/或隐私政策链接

请记住，这是为了您自身的法律保护。服务的使用必须公开透明并妥善记录。

### 包含非必需文件夹

此插件包含了一些看起来并非插件运行所必需的文件夹和文件。例如：

- 开发工具
- 生产环境中不需要的供应商文件夹（bower、node、grunt 等）
- 演示文件
- 单元测试

如果您试图包含自己代码的可读版本（以符合我们的指南），这是可以的，但请记住，我们也允许您在自述文件中提供这些文件的链接。

您还应保留和/或链接配置文件，例如 `composer.json` 文件，以便其他人可以审查、研究，甚至分叉此代码。

[详细插件指南](https://developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/#4-code-must-be-mostly-human-readable)

但是，您可以并且应该安全地从插件中删除那些其他非必需的文件夹。

### 不允许的文件

插件通常包含与功能相关的文件（php、js、css、txt、md），可能还有一些多媒体文件（png、svg、jpg）和/或数据文件（json、xml）。

我们检测到一些通常不会出现在插件中的文件，它们是必需的吗？如果不是，则这些文件将不被允许。

#### 包含“高级”来源的代码

某些高级库明确禁止被包含在免费（WordPress.org 托管的）插件中。这些代码必须被移除。

### GPL

#### GPL：包含非 GPL 兼容代码

所有代码必须兼容 GPLv2（或更高版本）许可证，才能被收录到我们的目录中。

您必须移除该代码并修改插件，使其不再依赖该代码。建议您寻找 GPL 兼容的代码作为替代。有关哪些类型的许可证与 GPL 兼容的更多信息，请查阅以下链接：

- [GNU 许可证 - 许可证列表](https://www.gnu.org/licenses/license-list.html)
- [GPL 常见问题 - 所有兼容性](https://www.gnu.org/licenses/gpl-faq.html#AllCompatibility)

#### GPL：无公开文档资源

**您的压缩内容缺少公开文档资源**

在审核您的插件时，我们未能找到您 JavaScript 和/或 CSS 相关源代码的非编译版本。

为遵循我们关于代码需具备人类可读性的指导原则，我们要求您提供源代码和/或指向插件中所包含的非压缩开发者库的链接。如果提供链接，可将其置于源代码中，但我们也要求您在 readme 文件中注明。

[详细插件指南](https://developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/#4-code-must-be-mostly-human-readable)

我们坚信，开源的优势之一在于能够审查、观察和改编代码。通过维护一个公开的免费代码目录，我们鼓励并欢迎未来的开发者参与 WordPress 并推动其发展。

话虽如此，随着使用更复杂库的大型插件不断涌现，人们正充分利用构建工具（如 composer 或 npm）来生成分发的生产代码。为了在保持插件体积较小的同时仍鼓励开源开发，我们要求插件通过在 readme 中记录的方式，将任何压缩文件的源代码公开在易于查找的位置。

例如，如果您开发了一个 Gutenberg 插件并使用 npm 和 webpack 进行压缩和最小化，您必须在发布的插件中包含源代码，或提供一个可供审查、研究乃至分叉的公开维护源代码库。

我们强烈建议您包含关于使用任何构建工具的说明，以鼓励未来的开发者。

#### GPL：使用 composer 但缺少 composer.json 文件

我们注意到您的插件正在使用 Composer 管理库依赖，这非常棒，因为它将有助于未来维护和更新您的插件，同时避免与使用相同库的其他插件发生冲突。

`composer.json` 文件描述了您项目的依赖关系，也可能包含其他元数据。这是一个通常可以在插件最顶层目录中找到的小文件。

作为开源优势之一，代码的可审查、可观察和可适应性至关重要，**我们恳请您将此文件包含在您的插件中**，即使它仅用于开发目的。这将使他人能够行使我们共同受益的开源自由。