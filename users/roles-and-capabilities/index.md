---
title: "角色与权限"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Roles And Capabilities
    - Users
    - Repos
---

# 角色与权限

角色与权限是 WordPress 中两个重要的概念，它们允许您控制用户特权。

WordPress 将角色及其对应的权限存储在 `options` 表中，键名为 `user_roles`。

## 角色

角色定义了用户的一组权限。例如，用户在其仪表板中可以看到和操作的内容。

**默认情况下，WordPress 有六个角色：**

- 超级管理员
- 管理员
- 编辑
- 作者
- 投稿者
- 订阅者

可以添加更多角色，也可以移除默认角色。

![用户角色](https://i3.wp.com/developer.wordpress.org/files/2014/09/wp-roles.png)

### 添加角色

使用 [`add_role()`](https://developer.wordpress.org/reference/functions/add_role/) 添加新角色并为其分配权限。

```
function wporg_simple_role() {
  add_role(
    'simple_role',
    'Simple Role',
    array(
      'read'         => true,
      'edit_posts'   => true,
      'upload_files' => true,
    ),
  );
}

// 添加 simple_role。
add_action( 'init', 'wporg_simple_role' );
```

[alert]首次调用 [`add_role()`](https://developer.wordpress.org/reference/functions/add_role/) 后，角色及其权限将被存储到数据库中！

后续调用将不会产生任何效果：包括修改权限列表，这可能不符合你的预期行为。[/alert]

[info]要批量修改权限列表：请使用 [`remove_role()`](https://developer.wordpress.org/reference/functions/remove_role/) 移除角色，然后使用 [`add_role()`](https://developer.wordpress.org/reference/functions/add_role/) 重新添加并指定新的权限。

请确保仅在权限与预期不符时才执行此操作（即添加条件判断），否则会显著降低性能！[/info]

### 移除角色

使用 [`remove_role()`](https://developer.wordpress.org/reference/functions/remove_role/) 移除角色。

```
function wporg_simple_role_remove() {
  remove_role( 'simple_role' );
}

// 移除 simple_role。
add_action( 'init', 'wporg_simple_role_remove' );
```

[alert]首次调用 [`remove_role()`](https://developer.wordpress.org/reference/functions/remove_role/) 后，该角色及其权限将从数据库中移除！

后续调用将不会产生任何效果。[/alert]

[info]如果你要移除默认角色：

- 我们**不建议**移除管理员和超级管理员角色！
- 请务必将相关代码保留在你的插件/主题中，因为未来的 WordPress 更新可能会重新添加这些角色。
- 运行 `update_option('default_role', YOUR_NEW_DEFAULT_ROLE)`，因为你将删除 WordPress 的默认角色 `subscriber`。[/info]

## Capabilities

Capabilities define what a **role** can and can not do: edit posts, publish posts, etc.

[info]Custom post types can require a certain set of Capabilities.[/info]

### Adding Capabilities

You may define new capabilities for a role.

Use [`get_role()`](https://developer.wordpress.org/reference/functions/get_role/) to get the role object, then use the `add_cap()` method of that object to add a new capability.

```
function wporg_simple_role_caps() {
  // Gets the simple_role role object.
  $role = get_role( 'simple_role' );

  // Add a new capability.
  $role->add_cap( 'edit_others_posts', true );
}

// Add simple_role capabilities, priority must be after the initial role definition.
add_action( 'init', 'wporg_simple_role_caps', 11 );
```

[info]It's possible to add custom capabilities to any role.

Under the default WordPress admin, they would have no effect, but they can be used for custom admin screen and front-end areas.[/info]

### 移除权限

您可以从角色中移除权限。

实现方式与添加权限类似，区别在于使用角色对象的 `remove_cap()` 方法。

## 使用角色与权限

### 获取角色

通过 [`get_role()`](https://developer.wordpress.org/reference/functions/get_role/) 获取包含其所有权限的角色对象。

### 用户权限检查

使用 [`user_can()`](https://developer.wordpress.org/reference/functions/user_can/) 可检查用户是否拥有指定的**角色**或**权限**。

```
user_can( $user, $capability );
```

[warning]存在未记录的第三个参数 $args，可包含用于权限测试的目标对象。

例如：传递文章 ID 以测试对该特定文章的权限。[/warning]

### 当前用户权限检查

[`current_user_can()`](https://developer.wordpress.org/reference/functions/current_user_can/) 是 [`user_can()`](https://developer.wordpress.org/reference/functions/user_can/) 的封装函数，它将当前用户对象作为 `$user` 参数传入。

当后台和前台区域需要特定权限级别才能访问或修改时，可使用此函数。

```
current_user_can( $capability );
```

### 示例

以下是一个在模板文件中添加编辑链接的实际示例，前提是用户具备相应权限：

```
if ( current_user_can( 'edit_posts' ) ) {
  edit_post_link( esc_html__( 'Edit', 'wporg' ), '<p>', '</p>' );
}
```

## 多站点

[`current_user_can_for_blog()`](https://developer.wordpress.org/reference/functions/current_user_can_for_blog/) 函数用于测试当前用户在特定博客上是否拥有某个**角色**或**权限**。

```
current_user_can_for_blog( $blog_id, $capability );
```

## 参考

[用户角色与权限](https://wordpress.org/documentation/article/roles-and-capabilities/)的 Codex 参考文档。