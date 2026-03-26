---
title: "用户管理"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Working With Users
    - Users
    - Repos
---

# 用户管理

## 添加用户

您可以使用 `wp_create_user()` 或 `wp_insert_user()` 来添加用户。

`wp_create_user()` 仅使用用户名、密码和邮箱参数创建用户，而 `wp_insert_user()` 接受描述用户及其属性的数组或对象。

### 创建用户

`wp_create_user()` 允许您创建新的 WordPress 用户。

[info]它使用 [`wp_slash()`](https://developer.wordpress.org/reference/functions/wp_slash/) 对值进行转义，使用 PHP 的 compact() 函数创建包含这些值的数组，并通过 [`wp_insert_user()`](https://developer.wordpress.org/reference/functions/wp_insert_user/) 执行插入操作。[/info]

请查阅关于 `wp_create_user()` 的函数参考以获取所用参数的完整说明。

#### Example Create

```
// check if the username is taken
$user_id = username_exists( $user_name );

// check that the email address does not belong to a registered user
if ( ! $user_id && email_exists( $user_email ) === false ) {
  // create a random password
  $random_password = wp_generate_password( 12, false );
  // create the user
  $user_id = wp_create_user(
    $user_name,
    $random_password,
    $user_email
  );
}
```

## 插入用户

```
wp_insert_user( $userdata );
```

[info]该函数会对大多数预定义属性调用过滤器。

创建用户时（用户 ID 不存在），该函数会执行 `user_register` 操作。

更新用户时（用户 ID 存在），该函数会执行 `profile_update` 操作。[/info]

关于所用参数的完整说明，请参阅 `wp_insert_user()` 的函数参考文档。

### Example Insert

Below is an example showing how to insert a new user with the website profile field filled in.

```
$username  = $_POST['username'];
$password  = $_POST['password'];
$website   = $_POST['website'];
$user_data = [
  'user_login' => $username,
  'user_pass'  => $password,
  'user_url'   => $website,
];

$user_id = wp_insert_user( $user_data );

// success
if ( ! is_wp_error( $user_id ) ) {
  echo 'User created: ' . $user_id;
}
```

## 更新用户

`wp_update_user()` 用于更新数据库中的单个用户。更新数据通过 `$userdata` 数组/对象传递。

如需更新单个用户元数据，请改用 `update_user_meta()`。如需创建新用户，请改用 `wp_insert_user()`。

[info]如果当前用户的密码被更新，cookies 将被清除！[/info]

关于所用参数的完整说明，请参阅 `wp_update_user()` 的函数参考文档。

### Example Update

Below is an example showing how to update a user's website profile field.

```
$user_id = 1;
$website = 'https://wordpress.org';

$user_id = wp_update_user(
  array(
    'ID'       => $user_id,
    'user_url' => $website,
  )
);

if ( is_wp_error( $user_id ) ) {
  // error
} else {
  // success
}
```

## 删除用户

`wp_delete_user()` 用于删除用户，并可选择将关联实体重新分配给另一个用户 ID。

[info]该函数在用户被删除后执行 `deleted_user` 操作。[/info]

[alert]如果 `$reassign` 参数未设置为有效的用户 ID，则属于被删除用户的所有实体都将被删除！[/alert]

请参阅关于 `wp_delete_user()` 的函数参考，以获取所用参数的完整说明。