---
title: "管理菜单"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Administration Menus
    - Repos
    - Data
---

# 管理菜单

管理菜单是 WordPress 后台显示的界面。它们允许您为插件添加选项页面。

[info]关于管理导航菜单的信息，请参阅《主题开发者手册》中的导航菜单章节。[/info]

## 顶级菜单与子菜单

顶级菜单显示在 WordPress 后台的左侧。每个菜单可以包含一组子菜单。

在选择使用[顶级菜单](https://developer.wordpress.org/plugins/administration-menus/top-level-menus/)还是[子菜单](https://developer.wordpress.org/plugins/administration-menus/sub-menus/)时，请仔细考虑您的插件需求以及最终用户的需求。

[alert]我们建议只有一个选项页面的开发者将其作为子菜单添加到现有的顶级菜单中，例如设置或工具。[/alert]