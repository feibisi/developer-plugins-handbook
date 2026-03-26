---
title: "用户"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Users
    - Repos
    - Data
---

# 用户

_用户_ 是 WordPress 安装中具有相应权限的访问账户。每个 WordPress 用户至少拥有用户名、密码和电子邮件地址。

用户账户创建后，该用户即可通过 WordPress 管理后台（或以编程方式）登录，以访问 WordPress 功能和数据。WordPress 将用户存储在 `users` 表中。

## 角色与权限

用户被分配[角色](https://developer.wordpress.org/plugins/users/roles-and-capabilities/#roles)，每个角色拥有一组[权限](https://developer.wordpress.org/plugins/users/roles-and-capabilities/#capabilities)。

您可以创建具有自定义权限集的新角色。也可以创建自定义权限，并将其分配给现有角色或新角色。

在 WordPress 中，开发者可以利用用户角色来限制账户可执行的操作集合。

## 最小权限原则

WordPress 遵循最小权限原则，即仅授予用户执行所需工作所必需的权限。您应尽可能遵循此原则，在适当时创建角色，并在执行敏感任务前检查权限。