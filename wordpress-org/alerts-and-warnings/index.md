---
title: "警报与警告"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Alerts And Warnings
    - Wordpress Org
    - Repos
---

# 警报与警告

当您访问 WordPress.org 上的插件页面时，可能会注意到特殊的警报或警告。这些提示旨在帮助访客了解各个插件的状态。

## 已批准与待处理数据

![蓝色背景 - 此插件已获批准并等待数据上传，但尚未对公众可见。一旦您进行首次提交，插件将变为公开状态。](https://developer.wordpress.org/files/2018/02/approved.jpg)

已获批准但尚未上传代码的插件将显示此消息：此信息**仅**对插件所有者显示，一旦通过 SVN 推送代码后便会消失。

## 已关闭

自 2017 年 11 月起，已关闭的插件会显示如下通知：

![红色背景：此插件已关闭，不再提供下载。](https://i3.wp.com/developer.wordpress.org/files/2018/02/closed.png)

所有访客均可看到此提示，表明插件已被关闭。2018 年 1 月之后关闭的插件将包含日期：

![红色背景：此插件已于 2018 年 2 月 7 日关闭，不再提供下载。](https://i3.wp.com/developer.wordpress.org/files/2018/02/closed-alt.jpg)

60 天后，提示信息将更新以说明插件被关闭的 _原因_：

![详细说明插件关闭原因的提示](https://i3.wp.com/developer.wordpress.org/files/2018/02/why-closed.png)

插件提交者将看到以下附加说明：

![蓝色背景：如果您未请求此变更，请联系 plugins@wordpress.org 了解状态。插件关闭时，所有具有提交权限的开发者都会收到通知及原因说明，请同时检查您的垃圾邮件。](https://i3.wp.com/developer.wordpress.org/files/2018/02/closed-owner.png)

### 插件被关闭的原因

- 作者要求 – 作者主动请求关闭插件。
- 违反准则 – 违反了任何一项指导原则。
- 许可/商标违规 – 使用了非 GPL 代码，或商标被滥用。
- 已并入核心 – 该插件现已成为核心的一部分（仅限功能项目）。
- 安全问题 – 在该插件中发现了安全隐患。

除非在极端情况下，否则不会向 WordPress.org 安全团队或插件作者之外的任何人提供插件被关闭的更多细节。

## 已过时

不支持最近 3 个主要 WordPress 版本的插件会显示以下通知：

![黄色背景：此插件尚未与最新 3 个主要 WordPress 版本进行测试。它可能不再被维护或支持，并且在较新版本的 WordPress 中使用时可能存在兼容性问题。](https://i3.wp.com/developer.wordpress.org/files/2018/02/old.jpg)

此前，此消息用于提醒用户插件在过去 2 年内未更新。2018 年，它被修改为依赖更相关的数据。由于 WordPress 每年发布 2 到 3 次主要更新，而一个维护中的插件应该与近期版本进行测试，因此当新版本的 WordPress 发布时，通过更新插件的 readme 文件可以避免此警报。

在每次 WordPress 主要版本发布前，开发者都会收到电子邮件，并被要求更新此值。他们_无需_发布新版本，只需[更新 readme 文件](https://developer.wordpress.org/plugins/wordpress-org/how-your-readme-txt-works/)并将 `Tested up to:` 的值编辑为最新的 WordPress 版本即可。