---
title: "本地化"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Localization
    - Internationalization
    - Repos
---

# 本地化

## 什么是本地化？

本地化描述的是国际化插件后续的翻译过程。本地化常被缩写为 `l10n`（因为 l 和 n 之间有 10 个字母）。

## 本地化文件

### POT（可移植对象模板）文件

此文件包含您插件中的原始字符串（英文）。

### PO（可移植对象）文件

每位译者将获取 `POT` 文件，并将 `msgstr` 部分翻译为自己的语言。最终生成与 `POT` 格式相同但包含译文及特定标头的 `PO` 文件。每种语言对应一个 `PO` 文件。

### MO（机器对象）文件

每个翻译后的 `PO` 文件都会生成一个 `MO` 文件。这些是机器可读的二进制文件，gettext 函数实际使用它们（它们不关心 `.POT` 或 `.PO` 文件），是 `PO` 文件的“编译”版本。转换通过 `msgfmt` 命令行工具完成。通常，一个应用程序可能使用多个大型逻辑可翻译模块，并相应地使用不同的 `MO` 文件。文本域是每个模块的句柄，对应不同的 `MO` 文件。

## 生成 POT 文件

`POT` 文件是您需要交给翻译人员以便他们开展工作的文件。`POT` 和 `PO` 文件可以轻松地通过重命名互换，从而更改文件类型，不会出现问题。

[info]建议将 POT 文件与您的插件一同提供，这样翻译人员就不必专门向您索要。[/info]

为您的插件生成 `POT` 文件有几种方法：

### WP-CLI

安装 [WP-CLI](https://make.wordpress.org/cli/handbook/guides/installing/)，并根据[文档](https://developer.wordpress.org/cli/commands/i18n/make-pot/)使用 `wp i18n make-pot` 命令。

### Poedit

你也可以在本地翻译时使用 [Poedit](https://poedit.net/)。这是一款适用于所有主流操作系统的开源工具。免费的 Poedit 默认版本支持手动扫描所有包含 Gettext 函数的源代码。其专业版还提供了一键扫描 WordPress 插件的功能。生成 `PO` 文件后，你可以将其重命名为 `POT`。如果生成了 `MO` 文件，可以将其删除，因为它不是必需的。如果你没有专业版，可以轻松获取 [Blank POT](https://github.com/fxbenard/Blank-WordPress-Pot) 并将其用作 `POT 文件` 的基础。将空白 `POT` 文件放入语言文件夹后，你可以在 Poedit 中点击“更新”来用你的字符串更新 `POT` 文件。

### Grunt 任务

甚至还有一些可用于创建 POT 文件的 Grunt 任务：[grunt-wp-i18n](https://github.com/cedaro/grunt-wp-i18n) 和 [grunt-pot](https://www.npmjs.com/package/grunt-pot)。设置 Grunt 的步骤超出了本文档的范围，但请注意这是可行的。这里有一个[示例 Grunt.js 和 package.json](https://gist.github.com/grappler/10187003)，你可以将其放在插件的根目录中。

## 翻译 PO 文件

将翻译后的文件保存为 `my-plugin-{locale}.mo`。locale 是语言代码和/或国家代码。例如，德语的 locale 是 `de_DE`。根据上面的代码示例，文本域是 'my-plugin'，因此德语的 MO 和 PO 文件应命名为 `my-plugin-de_DE.mo` 和 `my-plugin-de_DE.po`。有关语言和国家代码的更多信息，请参阅[以您的语言安装 WordPress](https://developer.wordpress.org/advanced-administration/before-install/in-your-language/)。

翻译 `PO` 文件有多种方法。

### 手动翻译

您可以使用文本编辑器输入翻译内容。在文本编辑器中，其格式如下所示。

```
#: plugin-name.php:123
msgid "Page Title"
msgstr ""
```

您需要在引号之间输入翻译内容。例如，德语翻译将如下所示。

```
#: plugin-name.php:123
msgid "Page Title"
msgstr "Seitentitel"
```

### Poedit

翻译时也可以使用 [Poedit](https://poedit.net/)。免费的 Poedit 默认版本支持手动扫描所有包含 Gettext 函数的源代码。其专业版还提供一键扫描 WordPress 插件和主题的功能。

### 在线服务

第三种选择是使用在线翻译服务。基本思路是上传 `POT` 文件，然后授权用户或翻译人员翻译你的插件。这能让你追踪变更、始终获取最新翻译，并避免重复翻译工作。

以下是一些可用于在线翻译 PO 文件的工具：

- [Transifex](https://www.transifex.com/)
- [WebTranslateIt](https://webtranslateit.com/)
- [Poeditor](https://poeditor.com/)
- [GlotPress](https://wordpress.org/plugins/glotpress/)

## 生成 MO 文件

### 命令行

`msgfmt` 用于创建 MO 文件。`msgfmt` 是 Gettext 软件包的一部分。也可以使用命令行。典型的 `msgfmt` 命令如下所示：

#### Unix 操作系统

```
msgfmt -o filename.mo filename.po
```

#### Windows 操作系统

```
msgfmt -o filename.mo filename.po
```

如果你需要批量转换多个 `PO` 文件，可以将其作为批处理任务运行。例如，使用 `bash` 命令：

#### Unix 操作系统（多文件）

```
# 查找 PO 文件，使用 msgfmt 处理每个文件，并将结果重命名为 MO
for file in `find . -name "*.po"` ; do msgfmt -o ${file/.po/.mo} $file ; done
```

#### Windows 操作系统（多个文件）

对于 Windows 系统，你需要先安装 [Cygwin](https://www.cygwin.com/)。

创建一个名为 `potomo.sh` 的文件，并将以下内容放入其中：

```
#! /bin/sh
# 查找 PO 文件，用 msgfmt 处理每个文件，并将结果重命名为 MO
for file in `/usr/bin/find . -name '*.po'` ; do /usr/bin/msgfmt -o ${file/.po/.mo} $file ; done
```

你可以在命令行中运行此命令。

```
cd C:/path/to/language/folder/my-plugin/languages & C:/cygwin/bin/bash -c /cygdrive/c/path/to/script/directory/potomo.sh
```

### Poedit

`msgfmt` 也集成在 [Poedit](https://poedit.net/) 中，允许你使用它来生成 MO 文件。在首选项中有一项设置，你可以启用或禁用它。

![国际化与本地化](https://i3.wp.com/developer.wordpress.org/files/2014/10/internationalization-localization-04.jpg)

### Grunt 任务

[grunt-po2mo](https://www.npmjs.com/package/grunt-po2mo) 可以转换所有文件。

## 优质翻译技巧

### 不要逐字翻译，要有机翻译

作为多语言使用者，你肯定知道不同语言在结构、节奏、语调和变化上各有差异。翻译信息无需与英文保持相同结构：理解原文表达的理念，用目标语言自然流畅的方式重新组织信息。这相当于创造对等信息而非相同信息：不要复制，而要替换。即使是信息中结构性较强的内容，如果认为调整后更符合逻辑或更适合目标受众，你也有权进行创造性改编。

### 尽量保持相同的正式程度（或非正式程度）

每条信息都有不同的正式或非正式程度。具体在目标语言中每条信息应使用何种程度的正式或非正式，需要您自行（或与团队共同）确定，但 WordPress 信息（尤其是通知类信息）在英语中通常采用礼貌的非正式语气。请尝试在您的文化背景下，在目标语言中实现同等效果。

### 避免使用俚语或特定受众术语

博客中难免会出现专业术语，但应避免使用只有"圈内人"才懂的俗语。如果一位新手博主用您的语言安装 WordPress，他们能理解这些术语的含义吗？像 pingback、trackback 和 feed 这类词汇是例外规则；这些术语通常难以翻译，许多译者选择保留英文原词。

### 阅读其他软件的本地化版本

如果你遇到困难或需要指导，可以尝试阅读其他流行软件工具的翻译版本，了解常用术语、正式程度等。当然，WordPress 有其独特的风格和感觉，在阅读其他本地化版本时请记住这一点，但可以自由挖掘 UI 术语等内容，以保持与你所用语言的其他软件的一致性。

## 使用本地化文件

将本地化文件放置在语言文件夹中，可以放在插件的 `languages` 文件夹内，或者从 WordPress 3.7 开始，通常放在 `wp-content` 下的插件 `languages` 文件夹中。完整路径为 `wp-content/languages/plugins/my-plugin-fr_FR.mo`。

您可以在“常规设置”中更改语言。如果看不到此选项，或者您想切换的语言未列出，请手动操作：

- 在 `wp-config.php` 中定义 `WPLANG` 为您选择的语言。例如，要使用法语，应添加：`define ('WPLANG', 'fr_FR');`
- 访问 `wp-admin/options-general.php` 或“设置” -> “常规”
- 在“站点语言”下拉菜单中选择您的语言
- 访问 `wp-admin/update-core.php`
- 点击“更新翻译”（如果可用）
- 核心翻译文件将被下载（如果可用）

## Resources

- [Creating .pot file for your theme or plugin](https://foxland.fi/creating-pot-file-for-your-theme-or-plugin/)
- [How To Internationalize WordPress Plugins](https://tommcfarlin.com/internationalize-wordpress-plugins/)
- [Translating Your Theme](https://code.tutsplus.com/translating-your-theme--wp-25014t)
- [Blank WordPress POT](https://github.com/fxbenard/Blank-WordPress-Pot)
- [Improved i18n WordPress tools](https://github.com/grappler/i18n)
- [How to update translations quickly](https://ulrich.pogson.ch/update-translations-quickly)
- [Gist: Complete Localization Grunt task](https://gist.github.com/grappler/10187003)
- WordPress.tv
  - [i18n](https://wordpress.tv/tag/i18n/)
  - [internationalization](https://wordpress.tv/tag/internationalization/)
  - [translation](https://wordpress.tv/tag/translation/)