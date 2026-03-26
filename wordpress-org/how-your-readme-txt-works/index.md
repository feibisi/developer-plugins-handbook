---
title: "插件自述文件"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - How Your Readme Txt Works
    - Wordpress Org
    - Repos
---

# 插件自述文件

本页将解释插件目录的一些方面，并说明许多用户容易忽略的明显要点。

为使插件浏览器中的条目发挥最大效用，每个插件都应包含一个名为 `readme.txt` 的自述文件，且需符合 [WordPress 插件自述文件标准](https://wordpress.org/plugins/readme.txt)。该文件控制目录前端页面的显示内容。在自述文件中编写的描述将直接决定 `wordpress.org/plugins/Your-Plugin` 页面的显示内容。

您可以使用 [插件自述文件生成器](https://generatewp.com/plugin-readme/) 创建内容，并通过 [官方自述文件验证器](https://wordpress.org/plugins/developers/readme-validator/) 检查结果。如需更直观的辅助工具，可使用 [wpreadme.com](https://wpreadme.com/)。

自 WordPress 5.8 起，系统[不再解析自述文件中的运行要求](https://core.trac.wordpress.org/ticket/48520)。这意味着 `Requires PHP` 和 `Requires at least` 标头信息将改由插件主 PHP 文件解析。

## 章节详情

所有插件都包含一个主 PHP 文件，并且几乎所有插件都有一个 `readme.txt` 文件。`readme.txt` 文件应使用 Markdown 的子集来编写。

### 插件说明文件头部信息

插件说明文件的头部包含以下信息：

```
=== 插件名称 ===
贡献者: (此处应为 WordPress.org 用户 ID 列表)
捐赠链接: https://example.com/
标签: tag1, tag2
最低 WordPress 版本要求: 4.7
测试至: 5.4
稳定版本: 4.3
最低 PHP 版本要求: 7.0
许可证: GPLv2 或更高版本
许可证链接: https://www.gnu.org/licenses/gpl-2.0.html
这里是插件的简短描述。不应超过 150 个字符。此处不使用任何标记语言。
```

- **贡献者** – 一个区分大小写、以逗号分隔的列表，包含所有为代码做出贡献的 WordPress.org 用户名。通常，包含分叉项目贡献者的名字被视为一种尊重。有些开发者会要求从列表中移除他们的名字，因为他们不希望其他插件出现在他们的个人资料页面上。最好尊重这些请求。请记住，*仅*使用 WordPress.org 用户名 – 其他任何内容都将显示为无个人资料链接和 Gravatar 头像。要更改某人的显示名称（显示在插件面向用户的页面上），请编辑个人资料 `https://wordpress.org/support/users/YOURID/edit/` 并更改显示名称。
- **捐赠链接** – (可选) 在侧边栏生成“捐赠给此插件”链接。如果没有链接，则不显示任何内容。
- **标签** – 1 到 5 个以逗号分隔的术语，用于描述插件。插件必须避免使用竞争对手的插件名称作为标签。插件不应使用该插件独有的标签，因为这些标签将不会显示。
- **测试至** – 插件已测试兼容的 WordPress 版本。此字段*忽略*次要版本，因为插件不应因次要更新而失效。这意味着插件只需定义其测试过的主要版本，WordPress.org 插件目录将自动添加次要版本。此处应*仅*使用数字，例如‘4.9’而非‘WP 4.9’。
- **最低 PHP 版本要求** – (可选) 使用此插件所需的 PHP *最低*版本。此处应*仅*使用数字，例如‘7.0’而非‘PHP 7.0’。
- **稳定版本** – 插件的稳定版本。这*不是* WordPress 的版本，而是插件本身的版本。仅使用数字和句点，推荐使用[语义化版本格式](https://semver.org/)。
- **许可证** – 插件使用的 GPLV2（或更高版本）兼容许可证。
- **许可证链接** – (可选) 许可证的链接。这是可选的，但如果插件使用较罕见的许可证，则强烈推荐提供。

在头部部分的末尾是用于插件 _简短_ 描述的位置。示例建议不超过 150 个字符且不使用标记。该行文本是插件的单行描述，会直接显示在插件名称下方。如果超过 150 个字符，它会被截断，因此请保持简短。

### 安装

如果您的插件没有自定义安装设置，可以省略此部分。如果插件安装后需要特殊配置说明，此处是放置相关信息的理想位置。

### 自定义区块

虽然允许并支持使用自定义区块，但请适度使用。用户已习惯其他插件的呈现方式，若您的插件样式过于特殊，可能导致用户遗漏重要信息。

## 技术细节

虽然自述文件的大部分内容不言自明，但有几个部分容易让人困惑。

### 自述文件的解析方式

WordPress.org 插件目录的运行基于自述文件中 **Stable Tag** 字段的信息。当 WordPress.org 解析 `readme.txt` 时，它首先查看 `/trunk` 目录中的 `readme.txt`，读取其中的 "Stable Tag" 行。

当 Stable Tag 正确设置时，WordPress.org 会去 `/tags/` 目录中查找引用的版本。因此，Stable Tag 设为 "1.2.3" 将使其查找 `/tags/1.2.3/`。

[tip]标签文件夹中的 readme.txt 也必须正确更新以包含正确的 "Stable Tag" —— 否则可能导致您的插件无法更新。[/tip]

如果 Stable Tag 是 1.2.3 且 `/tags/1.2.3/` 存在，那么系统将不再读取 trunk 中的任何内容进行解析。如果您尝试更改 `/trunk/readme.txt` 中的插件描述，您的更改将不会在插件页面上生效。所有内容都来自 Stable Tag 指向的文件中的 `readme.txt`。

WordPress.org 插件目录会读取主插件 PHP 文件以获取插件名称、插件 URI 以及最重要的版本号等信息。在插件页面上，您会看到下载按钮显示 "下载版本 1.2.3" 或类似内容。该版本号来自插件的主 PHP 文件，_而非_ 自述文件！

Stable Tag 指向 `/tags` 目录中的一个子目录。但插件的版本实际上并非由该文件夹名称决定。相反，插件 PHP 文件中列出的版本决定了名称。如果您将 Stable Tag 更改为 1.4，而 PHP 文件中插件版本仍为 1.3，则列出的版本将是 1.3。

[warning]虽然在插件目录中仍可使用 **stable tag** 设为 `trunk`（而非版本号），但这既不受支持也不推荐作为指示新版本的方法，并且已知会导致自动更新问题。我们目前积极反对使用 "Stable Tag: trunk"，并禁止新插件使用此设置。[/warning]

### 视频

您可以在文档中嵌入来自 YouTube、Vimeo 以及任何其他 [WordPress 默认支持](https://wordpress.org/support/article/embeds/)平台的视频。您只需将视频 URL 单独粘贴到 readme 文件的一行中即可。

我们建议您**不要**将视频放在 FAQ 部分的最后一行，因为有时格式会变得很奇怪。

### Markdown

这些自述文件使用定制版的 [Markdown](https://daringfireball.net/projects/markdown/)。大多数 Markdown 语法都能按预期工作。

Markdown 也允许在 readme.txt 中轻松添加链接。只需这样书写即可将词语链接到 URL：

\[WordPress\](http://wordpress.org)

视频也可以放入 readme.txt 中。单独一行的 YouTube 或 Vimeo 链接会自动嵌入。还可以使用 wpvideo 短代码嵌入托管在 VideoPress 上的视频。

### 字段详情

对于想要确切了解解析对应关系的人员：

- **作者**
  插件头部的作者字段和自述文件中的贡献者字段。
- **版本**
  插件头部的版本字段。
- **标签**（即分类）
  自述文件中的标签字段。
- **插件名称**
  自述文件中的插件名称，若缺失则回退到插件头部指定的插件名称。
- **作者与插件主页**
  插件头部的作者 URI 和插件 URI 字段。插件 URI 应对每个插件保持**唯一**。请**勿**为免费版和付费版插件使用相同的 URI，否则会导致不良后果。
- **最后更新时间**
  版本号变更后，在相应目录中最后一次提交的时间。
- **创建时间**
  **首次**提交的时间。

### 文件大小

虽然自述文件是简单的文本文件，但文件大小超过 10k 可能会导致错误。您的自述文件应简洁明了。描述不应是推销辞令，而应侧重于说明插件功能、用途及使用方法。安装指南应直接明了。常见问题解答应切实解决问题。

至于更新日志，我们建议将当前版本信息保留在自述文件中，其余内容可拆分至独立文件——例如 `changelog.txt`。通过将历史更新日志存储在该文件中，既能保持自述文件精简，又便于需要详细更新记录的用户自行查阅。

同理，若需包含内嵌图片等深度文档，建议引导用户访问您的独立网站。

## 另请参阅

- [插件头部信息](https://developer.wordpress.org/plugins/plugin-basics/header-requirements/)（位于插件的主文件中）