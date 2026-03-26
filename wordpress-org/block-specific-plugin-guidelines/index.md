---
title: "区块特定插件指南"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Block Specific Plugin Guidelines
    - Wordpress Org
    - Repos
---

# 区块特定插件指南

[info]所有区块特定插件还必须遵守[整体插件指南](https://developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/)。这些额外指南是区块特定插件独有的。[/info]

## 提交插件至区块目录指南

[区块目录](https://wordpress.org/plugins/browse/block/)的目标是为 WordPress 用户提供一个安全的空间，用于查找和安装新的区块。

### 用户期望

- 用户将能在 WordPress 管理后台和 WordPress.org 上找到可下载安装区块的专区，这些区块已通过重大安全问题的预先审查。
- 用户可在管理后台一键安装区块，每次安装一个。
- 区块在安装并激活后将显示在区块库中。
- 区块将无缝即时运行，不会出现侵扰性广告或升级推销。

### 开发者期望

- 开发者在为区块目录编写区块时，将拥有一套明确的需求和指南可供遵循。
- 遵循这些指南将有助于确保他们开发的区块能够无缝安装到编辑器中。
- 提交到区块目录的区块，相比一般的 WordPress 插件，需要遵守更严格的技术和政策规则。
- 包含不符合区块目录指南的区块的插件，仍可提交到插件目录。

### 定义

针对区块目录，我们区分两种类型的插件：

1. 仅用于分发区块的插件。
2. 所有其他插件，包括捆绑多个独立区块的插件、除区块外还包含其他功能的插件，以及完全不包含区块的插件。

这些指南专门适用于第一种类型，称为区块插件。如果插件属于第二种类型，则不适用进一步的指南和限制。所有插件，无论是否仅为区块插件，都必须遵守[详细插件指南](https://developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/)。

### 区块插件与区块目录

区块目录仅收录区块插件，即仅包含单个独立顶级区块及最少支持代码的插件。区块插件具有以下特征：

1. 属于特定类型的 WordPress 插件，具有相同结构（包含 `readme.txt` 文件）
2. 仅包含区块（通常为单个顶级区块）
3. 仅包含最少的服务端（如 PHP）代码
4. 自包含，在文章编辑器外无独立界面

除适用于所有 WordPress 插件的规范外，提交至区块目录的区块插件必须遵循以下准则：

### 指南

#### 1. 区块插件适用于区块编辑器。

一个区块插件必须包含一个区块，以及最少量的其他支持代码。它不能包含编辑器之外的任何用户体验，例如 WordPress 选项或 `wp-admin` 菜单。服务器端代码应保持在最低限度。

目前，仅扩展或为其他现有区块提供样式的插件不符合纳入区块目录的资格。目前，它们不受区块编辑器无缝安装过程的支持。目前仅支持注册新区块的区块插件。

#### 2. 区块插件是独立的区块。

区块插件旨在是单一用途、独立、分开的区块，而非多个区块的集合或汇编。在大多数情况下，一个区块插件应仅包含一个顶级区块。区块目录不会收录那些可以合理拆分为独立区块的区块集合。

当存在明确必要的父子或容器/内容依赖关系时，区块插件可以包含多个区块；例如，一个包含列表项区块的列表区块。

#### 3. 插件和区块名称应反映区块的用途。

插件标题和区块标题应描述区块的功能，以帮助用户轻松理解其用途。在大多数情况下，插件标题和区块标题应相同或非常相似。一些良好的插件和区块名称示例如下：

`Rainbow Block`
`Sepia Image Grid`
`Business Hours Block`

请避免使用与区块本身无关或难以与核心区块区分的插件和区块标题。一些无益的插件和区块名称示例如下：

`PluginCo Inc Block`
`Widget`
`Image Block`

区块标题和名称同样适用插件标题的商标限制。这意味着，除非由 Facerange 的合法代表开发，否则区块不得命名为“Facerange Block”。

#### 3a. 区块名称应唯一且正确命名空间化。

区块名称（指 [`registerBlockType()` 的 `name` 参数](https://developer.wordpress.org/block-editor/developers/block-api/block-registration/#block-name) 和 [`block.json` 中的 `name`](https://github.com/WordPress/gutenberg/blob/master/docs/rfc/block-registration.md#name)）必须对该区块是唯一的。与标题一样，请尊重商标和其他项目常用的名称，以免产生冲突。

区块名称的命名空间前缀应反映插件作者或插件别名。例如：

`name: "my-rainbow-block-plugin/rainbow"`，或
`name: "john-doe/rainbow"`，或
`name: "pluginco/rainbow"`。

命名空间不能是保留的，例如 `core` 或 `wordpress`。

#### 4. 区块插件必须包含 `block.json` 文件。

区块注册 RFC 概述了 `block.json` 文件的格式：[https://github.com/WordPress/gutenberg/blob/master/docs/designers-developers/developers/block-api/block-metadata.md](https://github.com/WordPress/gutenberg/blob/master/docs/designers-developers/developers/block-api/block-metadata.md)

区块插件必须包含一个有效的 `block.json` 文件。除了 RFC 中规定的要求外，`block.json` 还必须包含以下属性：

- `name`
- `title`
- 至少包含一项：`script`、`editorScript`
- 至少包含一项：`style`、`editorStyle`

#### 5. 区块插件必须能独立工作。

区块插件必须能够独立运行，无需依赖任何外部组件，例如其他 WordPress 插件或主题。

区块插件可以使用其他 WordPress 插件或主题中的代码或钩子，但前提是该区块插件在未安装该插件或主题的情况下仍可使用。例如，一个食谱区块插件可以应用滑块插件中实现的过滤器来改善其图像显示，只要该食谱区块插件在没有滑块插件的情况下仍能正常使用。

#### 6. 区块插件应无缝运行。

区块插件从编辑器安装后，应能即时无缝运行。这意味着它们不应给用户带来额外步骤或前提条件，例如安装其他插件或主题、注册账户、登录或手动连接外部服务。

使用区块插件不允许任何形式的付费。虽然允许在插件的 readme 文件中使用捐赠链接功能，或从完整插件列表页面链接，但区块插件本身必须免费使用。确实需要付费服务或包含升级销售和高级功能的区块插件，只要符合其指导原则，仍可在 WordPress 插件主目录中发布。

区块插件可在必要时使用外部/云 API，前提是无需登录或激活密钥即可无缝完成。

它们不应依赖外部 API 或云服务来执行可在本地完成的功能。

#### 7. 应尽量减少服务器端代码。

由于区块插件是 WordPress 插件，它们必然包含用于元数据和初始化的 PHP 代码。它们应尽可能避免包含额外的 PHP 代码或服务器端库。WordPress REST API 应是首选的 WordPress 接口，而非自定义的服务器端代码。

REST API 存在限制，某些情况下服务器端 PHP 是实现功能的唯一高效方式。在这些情况下，可以包含 PHP 代码，前提是代码清晰、尽可能少用且文档完善。

#### 8. 不得包含广告或推广通知。

区块插件不得包含显示警告、仪表盘通知或与区块功能无关的类似干扰性消息的代码。