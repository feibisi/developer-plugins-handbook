---
title: "将您的插件转移给新所有者"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Transferring Your Plugin To A New Owner
    - Wordpress Org
    - Repos
---

# 将您的插件转移给新所有者

虽然任何插件都可以拥有无限数量的提交者和支持代表，但任何时候插件只有一个官方所有者。这类似于 WordPress 中的文章只能有一个官方文章作者。

## 适用于用户数少于 10,000 的插件

如果您要将插件转移给新所有者，必须执行以下两个步骤。

首先，将新用户添加为插件的**提交者**：

- 访问 `https://wordpress.org/plugins/YOURPLUGIN/advanced`，将他们的用户名添加为提交者
- 更新 `readme.txt` 文件，将他们的用户 ID 添加为作者

接下来，转到“高级”选项卡并向下滚动到“危险区域”。您将看到**转移您的插件**部分。从下拉菜单中选择新所有者，然后单击按钮。

如果没有其他提交者，插件将无法转移，因此您必须先添加提交者。

![转移此插件界面，包含新所有者选择器和“请转移 -插件名称-”按钮](https://i0.wp.com/developer.wordpress.org/files/2020/04/transfer.jpeg?resize=1024%2C558&ssl=1)

## 适用于用户数超过 10,000（或测试版/精选）的插件

为防止滥用，用户量较大的插件以及被官方认定为精选/测试版的插件禁止进行此类更改。

在这种情况下转移插件，您需要从**当前**所有者的邮箱发送电子邮件至 `plugins@wordpress.org`，并提供以下信息：

1. 转移原因的_简要_说明
2. 新所有者的用户 ID
3. 如果适用，说明精选/测试版插件状态的任何变更

大多数请求都会顺利处理，但如果确定某个插件对 WordPress.org 项目至关重要，或有理由认为请求无效（例如，不是从当前所有者的邮箱发送，或无法确认关联的邮箱地址），请求可能会被拒绝或延迟处理。