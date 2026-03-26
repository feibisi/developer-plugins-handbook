---
title: "发布确认邮件"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Release Confirmation Emails
    - Wordpress Org
    - Repos
---

# 发布确认邮件

为了更好地保护插件免遭意外发布，我们为插件新增了一项**可选**功能：发布确认。

当插件启用此功能后，要发布新版本，您需要像往常一样在 SVN 中标记新版本，然后在[发布管理](https://wordpress.org/plugins/developers/releases/)面板上进行确认。

发布管理面板的访问权限仅限于插件提交者，所有操作都需要通过令牌化链接进行确认，该链接会根据需要通过电子邮件发送给您。

当 WordPress.org 检测到 SVN 中标记了新版本时，所有提交者都会自动收到一封电子邮件，通知他们有新版本待发布、发布者是谁，以及一个指向确认发布的面板链接。
插件提交者还将在其 WordPress.org 插件页面顶部看到一个横幅，引导您前往发布管理面板。

一旦确认，发布将照常进行，并应在几分钟内更新。

## 要求

-   插件必须使用标签来标记新版本，不能从主干（trunk）发布插件。
-   您仍然必须更新主干/readme.txt 文件中的 `Stable Tag:` 标头。
-   一旦发布，已标记的版本将无法进行任何修改。这包括更改 readme 文件（例如“测试至”版本）。
-   提交者必须能够直接接收电子邮件，而不是将其发送到共享收件箱或支持工单系统。

## 需要双重批准

发布确认是可选的，但只需要一位提交者确认发布即可。
如果某个团队希望要求多位提交者批准发布，请向插件团队提出您的请求。