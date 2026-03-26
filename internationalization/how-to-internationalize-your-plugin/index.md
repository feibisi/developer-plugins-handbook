---
title: "如何实现插件国际化"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - How To Internationalize Your Plugin
    - Internationalization
    - Repos
---

# 如何实现插件国际化

要使应用程序中的字符串可翻译，您需要将原始字符串包裹在一组特殊函数调用中。这些函数统称为"gettext"。

## Gettext 简介

WordPress 使用 [gettext](https://www.gnu.org/software/gettext/) 库和工具进行国际化，但并非直接使用：为此专门创建了一组特殊函数来实现字符串翻译。这些函数如下所列。您应在插件中使用这些函数。

如需深入了解 gettext，请阅读 [gettext 在线手册](https://www.gnu.org/software/gettext/manual/html_node/)。

## 文本域

使用 _文本域_ 来标识属于您插件的所有文本。文本域是一个唯一标识符，确保 WordPress 能够区分所有已加载的翻译。这提高了可移植性，并能更好地与现有的 WordPress 工具配合使用。

文本域必须与插件的 `slug` 匹配。如果您的插件是名为 `my-plugin.php` 的单个文件，或者包含在名为 `my-plugin` 的文件夹中，则域名必须是 `my-plugin`。如果您的插件托管在 wordpress.org 上，则必须是插件 URL 的 slug 部分（`wordpress.org/plugins/my-plugin`）。

文本域名必须使用连字符而非下划线，全部小写，且不能包含空格。

文本域还需要添加到插件头部。WordPress 使用它来国际化您的插件元数据，即使插件处于禁用状态。文本域应与[加载文本域](https://developer.wordpress.org/plugins/internationalization/how-to-internationalize-your-plugin/#loading-text-domain)时使用的文本域相同。

### 标题示例

```
/* 
 * Plugin Name: My Plugin
 * Author: Plugin Author
 * Text Domain: my-plugin
 */
```

[info]再次提醒，将 "my-plugin" 更改为你的插件别名。[/info]

[info]自 WordPress 4.6 起，`Text Domain` 标题是可选的，因为它必须与插件别名相同。包含它没有坏处，但不是必需的。[/info]

## 域路径

域路径定义了插件翻译文件的位置。这有几个用途，特别是让 WordPress 在插件被禁用时仍能找到翻译文件。默认情况下，它指向插件所在的文件夹。

例如，如果翻译文件位于插件内名为 `languages` 的文件夹中，那么域路径就是 `/languages`，并且必须以斜杠开头：

### 标题示例

```
/*
 * Plugin Name: My Plugin
 * Author: Plugin Author
 * Text Domain: my-plugin
 * Domain Path: /languages
 */
```

[info]如果插件位于官方的 WordPress 插件目录中，可以省略 `Domain Path` 标题。[/info]

## 基础字符串

对于基础字符串（指不包含占位符或复数形式的字符串），请使用 [`__()`](https://developer.wordpress.org/reference/functions/__/)。该函数返回其参数的翻译：

```
__( 'Blog Options', 'my-plugin' );
```

[warning]请勿在 gettext 函数的文本域部分使用变量名或常量。例如：不要使用以下快捷方式：

`__( 'Translate me.' , $text_domain );`[/warning]

若要直接输出已检索的翻译，请使用 [`_e()`](https://developer.wordpress.org/reference/functions/_e/)。因此，无需编写：

```
echo __( 'WordPress is the best!', 'my-plugin' );
```

您可以使用：

```
_e( 'WordPress is the best!', 'my-plugin' );
```

## Variables

What if you have a string like the following:

```
echo 'Your city is $city.'
```

In this case, the `$city` is a variable and should not be part of the translation. The solution is to use placeholders for the variable, along with the `printf` family of functions. Especially helpful are [`printf`](https://www.php.net/manual/en/function.printf.php) and [`sprintf`](https://www.php.net/manual/en/function.sprintf). Here is what the right solution looks like:

```
printf(
  /* translators: %s: Name of a city */
  __( 'Your city is %s.', 'my-plugin' ),
  $city
);
```

Notice that here the string for translation is just the template `"Your city is %s."`, which is the same both in the source and at run-time.

Also note that there is a hint for translators so that they know the context of the placeholder.

If you have more than one placeholder in a string, it is recommended that you use [argument swapping](http://www.php.net/manual/en/function.sprintf.php). In this case, single quotes (`'`) around the string are mandatory because double quotes (`"`) will tell php to interpret the `$s` as the `s` variable, which is not what we want.

```
printf(
  /* translators: 1: Name of a city 2: ZIP code */
  __( 'Your city is %1$s, and your zip code is %2$s.', 'my-plugin' ),
  $city,
  $zipcode
);
```

Here the zip code is being displayed after the city name. In some languages displaying the zip code and city in opposite order would be more appropriate. Using %s prefix in the above example, allows for such a case. A translation can thereby be written:

```
printf(
  /* translators: 1: Name of a city 2: ZIP code */
  __( 'Your zip code is %2$s, and your city is %1$s.', 'my-plugin' ),
  $city,
  $zipcode
);
```

**Important!** The following code is incorrect:

```
// This is incorrect do not use.
_e( "Your city is $city.", 'my-plugin' );
```

The strings for translation are extracted from the sources, so the translators will get this phrase to translate: `"Your city is $city."`.

However in the application `_e` will be called with an argument like `"Your city is London."` and `gettext` won't find a suitable translation of this one and will return its argument: `"Your city is London."`. Unfortunately, it isn't translated correctly.

## 复数形式

### 基本复数化处理

如果字符串会随项目数量变化而改变，您需要在翻译中体现这一点。例如，英语中有 `"One comment"` 和 `"Two comments"`。在其他语言中可能存在多种复数形式。在 WordPress 中处理这种情况，请使用 [`_n()`](https://developer.wordpress.org/reference/functions/_n/) 函数。

```
printf(
  _n(
    '%s comment',
    '%s comments',
    get_comments_number(),
    'my-plugin'
  ),
  number_format_i18n( get_comments_number() )
);
```

`_n()` 接受 4 个参数：

- singular – 字符串的单数形式（请注意在某些语言中，单数形式可能用于非 1 的数字，因此应使用 `'%s item'` 而非 `'One item'`）。
- plural – 字符串的复数形式。
- count – 对象数量，用于确定应返回单数还是复数形式（有些语言的复数形式远超过 2 种）。
- text domain – 插件的文本域。

函数的返回值是根据给定数量对应的正确翻译形式。

请注意，有些语言会将单数形式用于其他数字（例如 21、31 等，类似于英语中的 "21st"、"31st"）。如果您想特别处理单数情况，请单独检查：

```
if ( 1 === $count ) {
  printf( esc_html__( 'Last thing!', 'my-text-domain' ), $count );
} else {
  printf( esc_html( _n( '%d thing.', '%d things.', $count, 'my-text-domain' ) ), $count );
}
```

另请注意，`$count` 参数通常被使用两次。首先将 `$count` 传递给 `_n()` 以确定使用哪个翻译字符串，然后将 `$count` 传递给 `printf()` 以将数字代入翻译后的字符串中。

### 稍后处理复数形式

首先使用 [`_n_noop()`](https://developer.wordpress.org/reference/functions/_n_noop/) 或 [`_nx_noop()`](https://developer.wordpress.org/reference/functions/_nx_noop/) 设置复数字符串。

```
$comments_plural = _n_noop(
  '%s 条评论。',
  '%s 条评论。'
);
```

然后在代码的后续位置，可以使用 [`translate_nooped_plural()`](https://developer.wordpress.org/reference/functions/translate_nooped_plural/) 来加载这些字符串。

```
printf(
  translate_nooped_plural(
    $comments_plural,
    get_comments_number(),
    'my-plugin'
  ),
  number_format_i18n( get_comments_number() )
);
```

## 通过上下文消除歧义

有时一个术语会在多个上下文中使用，尽管在英语中是同一个单词，但在其他语言中需要不同的翻译。例如，单词 `Post` 既可以用作动词 `"点击此处发布您的评论"`，也可以用作名词 `"编辑此文章"`。在这种情况下，应使用 `_x()` 或 `_ex()` 函数。它类似于 `__()` 和 `_e()`，但有一个额外的参数——上下文：

```
_x( 'Post', 'noun', 'my-plugin' );
_x( 'Post', 'verb', 'my-plugin' );
```

在这两种情况下使用此方法，原始版本将得到字符串 Comment，但翻译人员将看到两个用于翻译的 Comment 字符串，每个字符串处于不同的上下文中。

请注意，与 `__()` 类似，`_x()` 也有一个 `echo` 版本：`_ex()`。前面的示例可以写成：

```
_ex( 'Post', 'noun', 'my-plugin' );
_ex( 'Post', 'verb', 'my-plugin' );
```

使用您认为能增强可读性和编码便利性的任何一种。

## 说明

为了让翻译者知道如何翻译像 `__( 'g:i:s a' )` 这样的字符串，你可以在源代码中添加一个说明性注释。它必须以 `translators:` 开头，并且必须是 gettext 调用前的最后一个 PHP 注释。以下是一个示例：

```
/* translators: 草稿保存日期格式，参见 http://php.net/date */
$saved_date_format = __( 'g:i:s a' );
```

它也用于解释字符串中的占位符，例如 `_n_noop( '<strong>版本 %1$s</strong> 修复了 %2$s 个错误。','<strong>版本 %1$s</strong> 修复了 %2$s 个错误。' )`。

```
/* translators: 1: WordPress 版本号, 2: 错误数量的复数形式。 */
_n_noop( '<strong>版本 %1$s</strong> 修复了 %2$s 个错误。','<strong>版本 %1$s</strong> 修复了 %2$s 个错误。' );
```

## 换行符

Gettext 不欢迎可翻译字符串中出现 `r`（ASCII 码：13），请避免使用它，改用 `n`。

## 空字符串

空字符串被 Gettext 内部使用所保留，请勿尝试国际化空字符串。这也没有任何意义，因为翻译者将看不到任何上下文。

如果您有国际化空字符串的合理用例，请添加上下文，既帮助翻译者，又与 Gettext 系统和谐共处。

## 字符串转义

建议对所有字符串进行转义处理，这样翻译人员就无法运行恶意代码。国际化的函数中内置了几种转义函数。

## Localization functions

### Basic functions

- [`__()`](https://developer.wordpress.org/reference/functions/__/)
- [`_e()`](https://developer.wordpress.org/reference/functions/_e/)
- [`_x()`](https://developer.wordpress.org/reference/functions/_x/)
- [`_ex()`](https://developer.wordpress.org/reference/functions/_ex/)
- [`_n()`](https://developer.wordpress.org/reference/functions/_n/)
- [`_nx()`](https://developer.wordpress.org/reference/functions/_nx/)
- [`_n_noop()`](https://developer.wordpress.org/reference/functions/_n_noop/)
- [`_nx_noop()`](https://developer.wordpress.org/reference/functions/_nx_noop/)
- [`translate_nooped_plural()`](https://developer.wordpress.org/reference/functions/translate_nooped_plural/)

### Translate & Escape functions 

Strings that require translation and is used in attributes of html tags must be escaped.

- [`esc_html__()`](https://developer.wordpress.org/reference/functions/esc_html__/)
- [`esc_html_e()`](https://developer.wordpress.org/reference/functions/esc_html_e/)
- [`esc_html_x()`](https://developer.wordpress.org/reference/functions/esc_html_x/)
- [`esc_attr__()`](https://developer.wordpress.org/reference/functions/esc_attr__/)
- [`esc_attr_e()`](https://developer.wordpress.org/reference/functions/esc_attr_e/)
- [`esc_attr_x()`](https://developer.wordpress.org/reference/functions/esc_attr_x/)

### 日期与数字函数

- [`number_format_i18n()`](https://developer.wordpress.org/reference/functions/number_format_i18n/)
- [`date_i18n()`](https://developer.wordpress.org/reference/functions/date_i18n/)

## 字符串编写最佳实践

以下是编写字符串的最佳实践

- 使用得体的英语风格——尽量减少俚语和缩写
- 使用完整句子——大多数语言的语序与英语不同
- 按段落拆分——合并相关句子，但不要将整页文本放在一个字符串中
- 不要在可翻译短语中保留首尾空格
- 假设翻译后字符串长度可能翻倍
- 避免使用特殊标记和非常规控制字符——不要包含包围文本的标签
- 不要在翻译字符串中添加不必要的 HTML 标记
- 除非可能有其他语言版本，否则不要将 URL 留给翻译
- 在字符串中添加变量占位符，因为某些语言中占位符位置会变化

```
printf(
  __( '搜索结果：%s', 'my-plugin' ),
  get_search_query()
);
```

- 使用格式化字符串而非字符串连接——翻译短语而非单词——`printf( __( '您所在城市是 %1$s，邮政编码是 %2$s。', 'my-plugin' ), $city, $zipcode );` 总是优于：`__( '您所在城市是 ', 'my-plugin' ) . $city . __( '，邮政编码是 ', 'my-plugin' ) . $zipcode;`
- 尽量使用相同的词语和符号，避免多个字符串需要翻译，例如 `__( '文章：', 'my-plugin' );` 和 `__( '文章', 'my-plugin' );`

### 为字符串添加文本域

您必须将文本域作为参数添加到每个 `__()`、`_e()` 和 `__n()` 的 gettext 调用中，否则您的翻译将无法工作。

示例：

- `__( 'Post' )` 应改为 `__( 'Post', 'my-theme' )`。
- `_e( 'Post' )` 应改为 `_e( 'Post', 'my-theme' )`。
- `_n( '%s post', '%s posts', $count )` 应改为 `_n( '%s post', '%s posts', $count, 'my-theme' )`。

如果您的插件中有与 WordPress 核心相同的字符串（例如“Settings”），您仍然需要为其添加自己的文本域，否则当核心字符串更改时（这种情况会发生），这些字符串将无法翻译。

如果在编写代码时没有持续添加文本域，手动添加可能会很繁琐，这就是为什么您可以自动完成此操作：

- 将 [`add-textdomain.php`](https://develop.svn.wordpress.org/branches/5.2/tools/i18n/add-textdomain.php) 脚本下载到您要添加文本域的文件所在的文件夹中。
- 在命令行中，移动到该文件所在的目录。
- 运行以下命令以创建一个添加了文本域的新文件：

```
php add-textdomain.php my-plugin my-plugin.php > new-my-plugin.php
```

如果您希望将 `add-textdomain.php` 放在不同的文件夹中，只需在命令中定义其位置即可。

```
php /path/to/add-textdomain.php my-plugin my-plugin.php > new-my-plugin.php
```

如果您不想输出新文件，请使用此命令：

```
php add-textdomain.php -i my-plugin my-plugin.php
```

如果您想更改目录中的多个文件，也可以将目录传递给脚本：

```
php add-textdomain.php -i my-plugin my-plugin-directory
```

完成后，文本域将被添加到文件中所有 gettext 调用的末尾。如果已存在文本域，则不会被替换。

## 加载文本域

可以使用 `load_plugin_textdomain` 加载翻译，例如：

```
add_action( 'init', 'wpdocs_load_textdomain' );

function wpdocs_load_textdomain() {
  load_plugin_textdomain( 'wpdocs_textdomain', false, dirname( plugin_basename( __FILE__ ) ) . '/languages' );
}
```

## WordPress.org 上的插件

[info]自 WordPress 4.6 起，翻译现在优先使用 [translate.wordpress.org](https://translate.wordpress.org/)，因此通过 translate.wordpress.org 翻译的插件不再必须使用 `load_plugin_textdomain()`。如果您不想在插件中添加 `load_plugin_textdomain()` 调用，则必须在 readme.txt 中将 `Requires at least:` 字段设置为 4.6 或更高版本。[/info]

如果您仍想加载自己的翻译而非来自 translate 的翻译，则必须使用名为 `load_textdomain_mofile` 的钩子过滤器。  
**示例**：在插件的 `/languages/` 目录中放置 .mo 文件，并将以下代码插入主插件文件：

```
function my_plugin_load_my_own_textdomain( $mofile, $domain ) {
  if ( 'my-domain' === $domain && false !== strpos( $mofile, WP_LANG_DIR . '/plugins/' ) ) {
    $locale = apply_filters( 'plugin_locale', determine_locale(), $domain );
    $mofile = WP_PLUGIN_DIR . '/' . dirname( plugin_basename( __FILE__ ) ) . '/languages/' . $domain . '-' . $locale . '.mo';
  }
  return $mofile;
}
add_filter( 'load_textdomain_mofile', 'my_plugin_load_my_own_textdomain', 10, 2 );
```

## 处理 JavaScript 文件

请查阅 [通用 API 手册](https://developer.wordpress.org/apis/) 中的 [JavaScript 国际化](https://developer.wordpress.org/apis/internationalization/#internationalizing-javascript) 部分，了解如何正确加载翻译文件。另请参阅 [Gutenberg 插件文档页面](https://github.com/WordPress/gutenberg/blob/trunk/docs/how-to-guides/internationalization.md)。

## 语言包

如果您对语言包以及如何导入到 [translate.wordpress.org](https://translate.wordpress.org/) 感兴趣，请阅读 [Meta 手册中关于翻译的页面](https://make.wordpress.org/meta/handbook/tutorials-guides/translations/)。

同时，请参考 [Polyglots 手册中的插件/主题作者指南](https://make.wordpress.org/polyglots/handbook/plugin-theme-authors-guide/) 来获取项目的翻译支持。