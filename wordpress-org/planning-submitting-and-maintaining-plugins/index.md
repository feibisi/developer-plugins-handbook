---
title: "插件规划、提交与维护"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Planning Submitting And Maintaining Plugins
    - Wordpress Org
    - Repos
---

# 插件规划、提交与维护

您已经编写了下一个 [Hello Dolly](https://wordpress.org/plugins/hello-dolly/)，并希望全世界都能使用它。接下来该怎么做？

## 1. 反复测试

您的插件很可能会被许多用户在各种不同场景和托管环境中使用。您需要确保已充分测试插件，保证其在任何情况下都能正常运行，避免给用户带来困扰。

## 2. 选择一个好名字

插件名称应体现您和您作品的独特性。选择名称时，请确保不侵犯商标权或与他人产品名称冲突。例如，如果您并非为 FaceRange（一家虚构公司）工作，就不应将插件命名为“FaceRange 的跳舞松鼠”。更好的名称可以是“FaceRange 跳舞松鼠插件”。构思一个好名字可能很困难，请耐心斟酌。提交后插件 URL 无法更改，但显示名称可随时调整。

显示名称基于主插件文件的头部信息生成，因此请务必仔细检查。

## 3. 撰写优质文档

[README.txt](https://wordpress.org/plugins/developers/#readme) 文件是最好的起点，因为它是所有插件的标准参考点。你需要确保包含以下内容：

- 简明描述你的插件实际功能。如果功能过多，或许拆分成两个插件更好。
- 安装说明，特别是需要特殊配置时。如果用户需要注册你的服务，务必提供链接。
- 获取支持的指引，以及你提供和不提供哪些支持。

## 4. 提交你的插件

要提交插件，需要完成以下三个步骤：

1. 在 WordPress.org 上注册一个有效且定期检查的邮箱地址。如果你代表公司提交插件，请使用公司的**官方**邮箱进行验证。
2. 在你的邮件客户端中将 `plugins@wordpress.org` 加入白名单，以确保能收到邮件通知。
3. [提交你的插件](https://wordpress.org/plugins/developers/add/)，并附上简要的功能概述以及一个完整、可直接使用的插件 zip 包。该 zip 包必须是插件的完整版本，就像你通过插件安装器手动上传时一样。

插件进入审核队列后，我们将在 14 个工作日内审查代码是否存在问题。遵循指南可以避免大部分问题。如果发现问题，我们会联系开发者并共同寻求解决方案。审核通过后，你将收到一封邮件，其中包含如何访问[Subversion 仓库](https://developer.wordpress.org/plugins/wordpress-org/how-to-use-subversion/)的详细信息，你将在该仓库中存储你的插件。

当你通过 SVN 在该仓库中上传插件（以及[readme 文件](https://wordpress.org/plugins/developers/#readme)）后，它就会出现在[插件目录](https://wordpress.org/plugins/)中。

## 5. 发布首个版本

WordPress.org 插件目录是潜在用户下载和安装插件的最便捷途径。WordPress 与插件目录的集成意味着用户只需点击几下即可更新您的插件。

当您准备好发布第一个版本时，您需要[注册](https://wordpress.org/plugins/developers/add/)。审核流程成功完成后，您将获得一个用于存放代码的 Subversion 仓库。我们提供了关于[在 WordPress.org 上使用 SVN](https://developer.wordpress.org/plugins/wordpress-org/how-to-use-subversion/) 的文档，如果您习惯使用 GIT，其工作流程可能略有不同。

## 6. 拥抱开源

开源是我们这个时代最强大的理念之一，因为它促进了跨越国界的协作。通过鼓励贡献，你让他人也能像你一样热爱你的代码。开源你的代码有几种选择：

- [Github](https://github.com/) 让他人轻松参与你的项目。其他开发者和用户可以方便地提交错误修复或报告、功能请求或全新的贡献。Github 有一个[优秀的文档门户](https://support.github.com/)。
	- [Bitbucket](https://bitbucket.org/) 和 [Gitlab](https://about.gitlab.com/) 是具有类似功能的替代方案。
- WordPress.org 插件目录提供并要求你使用 [Subversion 仓库](https://developer.wordpress.org/plugins/wordpress-org/how-to-use-subversion/)。

## 7. 倾听用户的声音

你常会发现，用户对你的代码进行了远超你想象的多种测试。这是极其宝贵的反馈。

通过 WordPress.org 发布代码意味着你的插件会自动拥有一个支持论坛。善用它！你可以订阅邮件接收新帖子，并及时回应用户。他们只是希望像你一样热爱你的插件。

Jetpack 有一篇关于[如何撰写优质错误报告](https://jetpack.com/blog/how-to-write-a-great-bug-report/)的文章可供参考。

## 8. 定期推送新版本

优秀的插件会持续迭代，不断推出小幅更新。不要因等待过久而让心血之作变得陈旧。请注意，频繁升级可能导致“更新疲劳”，用户会停止更新。在更新过少与过多之间保持平衡至关重要。

## 9. 反复迭代

如同生活的其他方面，最好的成果源于耐心与努力。