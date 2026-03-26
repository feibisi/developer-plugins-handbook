---
title: "渲染文章元数据"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Rendering Post Metadata
    - Metadata
    - Repos
---

# 渲染文章元数据

以下是不完整的用于获取和渲染文章元数据的函数和[模板标签](https://developer.wordpress.org/themes/basics/template-tags/)列表：

- [`get_post_meta()`](https://developer.wordpress.org/reference/functions/get_post_meta/) – 自动列出文章所有自定义字段的模板标签。
- [`get_post_custom()`](https://developer.wordpress.org/reference/functions/get_post_custom/) 和 [`get_post_meta()`](https://developer.wordpress.org/reference/functions/get_post_meta/) – 检索文章的一个或所有元数据。
- [`get_post_custom_values()`](https://developer.wordpress.org/reference/functions/get_post_custom_values/) – 检索自定义文章字段的值。