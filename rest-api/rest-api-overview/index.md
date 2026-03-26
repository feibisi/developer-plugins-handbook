---
title: "REST API 概述"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Rest Api Overview
    - Rest Api
    - Repos
---

# REST API 概述

WordPress REST API 为 WordPress 带来了许多新功能。REST API 使用 JSON（JavaScript 对象表示法）作为其数据格式。JSON 是一种开放标准数据格式，在整个网络乃至通用软件中正得到越来越广泛的应用。它轻量且人类可读，看起来像 JavaScript 中的对象；因此得名。当你向 API 发出请求时，响应将以 JSON 格式返回。这使得开发者能够使用 PHP 以外的语言来使用 WordPress，从而允许以新颖而激动人心的方式使用 WordPress。

## 为何使用 WordPress REST API

WordPress REST API 拥有众多应用场景。其中最主要的用途之一是在 WordPress 之上构建单页应用程序。您可以为 WordPress 创建全新的管理界面体验，也可以打造全新的前端交互体验。甚至无需使用 PHP 编写应用程序——任何能够发起 HTTP 请求并解析 JSON 的编程语言都可用于在 WordPress 平台上进行开发。

WordPress REST API 也可作为核心中 admin-ajax API 的强大替代方案。通过使用 REST API，您可以更灵活地构建数据进出 WordPress 的传输结构。借助 REST API 可大幅简化 AJAX 调用，从而在工作中提供更优质的用户体验。

其应用场景远不止于此，真正限制我们的是想象力。关键在于，如果您需要一种结构化、可扩展且简单的方式来传输 WordPress 数据，那么 REST API 很可能是理想选择。尽管该 API 设计简洁，但初期可能显得复杂，我们将尝试将其拆解为小型组件，以便逐步拼凑出完整的应用图景。

## 核心概念

要开始使用 WordPress REST API，我们先来解析与该 API 相关的一些核心概念和术语：

- 路由/端点
- 请求
- 响应
- 模式
- 控制器类

这些概念在理解和使用 WordPress REST API 中都扮演着至关重要的角色。让我们先简要解析它们，以便后续能更深入地探讨每个概念。

### 路由与端点

在 WordPress REST API 中，路由是指可映射到不同 HTTP 方法的 URI。将单个 HTTP 方法映射到路由的过程称为端点。具体来说：当我们向 `http://oursite.com/wp-json/` 发起 `GET` 请求时，会收到 JSON 响应，显示可用路由及每个路由下的可用端点。`/wp-json/` 本身就是一个路由，当发起 `GET` 请求时，会匹配到显示 WordPress REST API 索引的端点。我们将在后续章节学习如何注册自定义路由和端点。

### 请求

在 WordPress REST API 架构中，`WP_REST_Request` 是核心类之一。该类用于存储和检索当前请求的信息，也可在 PHP 内部发起请求以避免使用 HTTP。当向已注册路由发起 HTTP 请求时，系统会自动生成 `WP_REST_Request` 对象。请求中指定的数据将直接影响 API 返回的响应内容。通过请求类可实现诸多实用功能，后续章节将对此展开详细说明。

### 响应

响应是从 API 获取的数据。`WP_REST_Response` 提供了一种与端点返回的响应数据进行交互的方式。响应可以返回所需的数据，也可用于返回错误信息。

### 模式

当我们需要处理不同类型数据的响应和请求时，必须能够识别所交互的数据类型。模式为我们提供了一种结构化数据的方式。模式还为 API 带来了安全优势，因为它使我们能够验证向 API 发出的请求。模式是一个庞大的主题，我们将在模式章节中深入探讨。

### 控制器类

正如您所见，WordPress REST API 包含许多需要协同工作的动态组件。控制器类使我们能够在一个统一的位置整合所有这些元素。通过控制器类，我们将能够管理路由和端点的注册、处理请求、利用模式并生成响应。

## 后续步骤

让我们深入了解如何为 REST API 注册路由和端点。