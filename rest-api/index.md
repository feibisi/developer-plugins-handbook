---
title: "REST API"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Rest Api
    - Repos
    - Data
---

# REST API

WordPress 4.4 引入了 REST API 的基础架构。REST API 提供了一种简便的方式来获取和存储 WordPress 数据。通过向 REST API 服务器发送 HTTP 请求，可以检索和存储数据。REST API 利用了不同的 HTTP 方法。

- `GET` 应用于从 API 检索数据。
- `POST` 应用于创建新资源（例如用户、文章、分类法）。
- `PUT` 应用于更新资源。
- `DELETE` 应用于删除资源。
- `OPTIONS` 应用于提供资源的上下文信息。

资源是任何单一的实体或对象。WordPress 中一个很好的资源示例是文章。文章具有不同的属性，如标题和内容。API 的响应可以将标题和内容作为响应中的字段显示。REST API 使我们能够以新的方式与文章和其他 WordPress 资源进行交互。REST API 使得与网络其他部分共享内容变得更加容易，并提供了处理 WordPress 内复杂交互的结构化方式。

在本插件手册的章节中，我们将探讨 API 的工作原理，以及如何利用其功能在 WordPress 中实现出色的功能！