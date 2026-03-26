---
title: "将您的插件添加到区块目录"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Add Your Plugin To The Block Directory
    - Wordpress Org
    - Repos
---

# 将您的插件添加到区块目录

区块相关插件主要有两种目录列表：

- **仅包含**区块的插件
- 包含区块的插件

一个插件可以同时出现在两个列表中，但这不会对您的插件 SEO 或排名带来任何好处。成为仅包含区块的插件的唯一好处是，您可以在区块编辑器本身的区块目录中添加插件，以便立即安装。

## 仅区块插件

仅区块插件是指**仅**包含区块的插件。

区块插件需要比常规 WordPress 插件更小巧、更精简，以确保能够安全地一键安装。这意味着除了遵守常规的[插件指南](https://developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/)外，您还需要遵循一些额外的规则。特别是，您应主要使用 JavaScript 代码，将 PHP 保持在最低限度；并且不要在古腾堡编辑器之外添加任何用户界面或其他代码。

如果您是某个区块插件的提交者，并且该插件已通过检查工具确认符合添加到[区块目录](https://wordpress.org/plugins/browse/block/)的标准，那么您可以[使用区块检查工具](https://wordpress.org/plugins/developers/block-plugin-validator/)自行添加：

![区块检查工具界面，带有“将插件名称添加到区块目录”按钮](https://i0.wp.com/developer.wordpress.org/files/2020/08/Screen-Shot-2020-07-10-at-1.29.25-pm.png?resize=1024%2C308&ssl=1)

同样，如果您发现问题或希望不将其包含在内，也可以随时使用同一工具将其移除。

**有用的工具：**

- [区块创建教程](https://github.com/WordPress/gutenberg/pull/22831/files?short_path=c4d2c28#diff-c4d2c286eac33acdc7571032a984e0ca) – 如何编写区块插件
- [区块提交教程](https://github.com/WordPress/gutenberg/pull/23545/files?short_path=555f1c3#diff-555f1c31856d86ed5ff0d492b5a127c1) – 确保您的区块已准备好加入区块目录的技巧和建议
- [区块检查工具](https://make.wordpress.org/plugins/2020/07/11/you-can-now-add-your-own-plugins-to-the-block-directory/) – 验证插件是否符合收录条件

## 包含区块的插件

许多旧版插件以及更大型、更复杂的插件可能包含区块。它们通常也包含其他功能，例如小工具。这类插件的典型代表是 Jetpack 或 Yoast SEO。虽然它们拥有大量功能，但也包含一些区块。

如果您编写的插件引入或改进了区块，或知道有此类插件，**请发送邮件至 [plugins@wordpress.org](wordpress-org/add-your-plugin-to-the-block-directory/mailto:plugins@wordpress.org)** 并申请将您的插件加入列表。届时我们将审核您的插件以确认请求，同时确保其符合所有现行规范标准。

发送邮件前，请务必确认您的插件与最新版 WordPress 兼容，且不存在任何安全问题。如果插件存在安全或规范问题，可能会在您修正前被暂停处理。