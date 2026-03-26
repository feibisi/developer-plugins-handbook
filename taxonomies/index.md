---
title: "Index"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Taxonomies
    - Repos
    - Data
---

## 分类法

**分类法** 是对事物进行分类/分组的专业术语。分类法可以是层级结构（包含父级/子级）或扁平结构。

WordPress 将分类法存储在 `term_taxonomy` 数据库表中，允许开发者注册自定义分类法，与现有分类法并存。

分类法包含 **术语**，这些术语作为对事物进行分类/分组的话题。它们存储在 `terms` 表中。

例如：一个名为 "艺术" 的分类法可以包含多个术语，例如 "现代" 和 "18世纪"。

本章将展示如何注册自定义分类法，如何从数据库中检索其内容，以及如何向公众展示它们。

[info]WordPress 3.4 及更早版本有一个名为 "链接" 的分类法，该分类法在 WordPress 3.5 中已被弃用。[/info]