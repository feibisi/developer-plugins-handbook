---
title: "使用用户元数据"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Working With User Metadata
    - Users
    - Repos
---

# 使用用户元数据

## 简介

WordPress 的 `users` 表最初设计为仅包含用户的基本信息。

[info]截至 WP 4.7，该表包含：`ID`、`user_login`、`user_pass`、`user_nicename`、`user_email`、`user_url`、`user_registered`、`user_activation_key`、`user_status` 和 `display_name`。[/info]

因此，为了存储额外数据，引入了 `usermeta` 表，它可以存储关于用户的任意数量数据。

这两个表通过基于 `users` 表中 `ID` 的一对多关系相互关联。

## 操作用户元数据

主要有两种方式操作用户元数据。

1. 用户资料屏幕中的表单字段。
2. 通过函数调用以编程方式。

### 通过表单字段

表单字段选项适用于用户可以访问 WordPress 管理后台的情况，用户将能够在其中查看和编辑个人资料。

在深入示例之前，了解此过程中涉及的钩子及其存在的原因非常重要。

#### `show_user_profile` 钩子

当用户编辑**其自身**用户资料时，会触发此操作钩子。

**请注意，** 不具备编辑自身资料权限的用户不会触发此钩子。

#### `edit_user_profile` 钩子

当用户编辑**其他用户**的个人资料时，会触发此操作钩子。

**请注意，** 不具备编辑第三方资料权限的用户不会触发此钩子。

#### Example Form Field

In the example below we will be adding a birthday field to the all profile screens. Saving it to the database on profile updates.

```
/**
 * The field on the editing screens.
 *
 * @param $user WP_User user object
 */
function wporg_usermeta_form_field_birthday( $user ) {
  ?>
  <h3>It's Your Birthday</h3>
  <table class="form-table">
    <tr>
      <th>
        <label for="birthday">Birthday</label>
      </th>
      <td>
        <input type="date"
          class="regular-text ltr"
          id="birthday"
          name="birthday"
          value="<?= esc_attr( get_user_meta( $user->ID, 'birthday', true ) ) ?>"
          title="Please use YYYY-MM-DD as the date format."
          pattern="(19[0-9][0-9]|20[0-9][0-9])-(1[0-2]|0[1-9])-(3[01]|[21][0-9]|0[1-9])"
          required>
        <p class="description">
          Please enter your birthday date.
        </p>
      </td>
    </tr>
  </table>
  <?php
}

/**
 * The save action.
 *
 * @param $user_id int the ID of the current user.
 *
 * @return bool Meta ID if the key didn't exist, true on successful update, false on failure.
 */
function wporg_usermeta_form_field_birthday_update( $user_id ) {
  // check that the current user have the capability to edit the $user_id
  if ( ! current_user_can( 'edit_user', $user_id ) ) {
    return false;
  }

  // create/update user meta for the $user_id
  return update_user_meta(
    $user_id,
    'birthday',
    $_POST['birthday']
  );
}

// Add the field to user's own profile editing screen.
add_action(
  'show_user_profile',
  'wporg_usermeta_form_field_birthday'
);

// Add the field to user profile editing screen.
add_action(
  'edit_user_profile',
  'wporg_usermeta_form_field_birthday'
);

// Add the save action to user's own profile editing screen update.
add_action(
  'personal_options_update',
  'wporg_usermeta_form_field_birthday_update'
);

// Add the save action to user profile editing screen update.
add_action(
  'edit_user_profile_update',
  'wporg_usermeta_form_field_birthday_update'
);
```

### 以编程方式

此选项适用于构建自定义用户区域和/或计划禁用 WordPress 管理区域访问权限的情况。

可用于操作用户元数据的函数包括：[`add_user_meta()`](https://developer.wordpress.org/reference/functions/add_user_meta/)、[`update_user_meta()`](https://developer.wordpress.org/reference/functions/update_user_meta/)、[`delete_user_meta()`](https://developer.wordpress.org/reference/functions/delete_user_meta/) 和 [`get_user_meta()`](https://developer.wordpress.org/reference/functions/get_user_meta/)。

#### 添加

```
add_user_meta(
  int $user_id,
  string $meta_key,
  mixed $meta_value,
  bool $unique = false
);
```

关于所用参数的完整说明，请参阅关于 [`add_user_meta()`](https://developer.wordpress.org/reference/functions/add_user_meta/) 的函数参考。

#### 更新

```
update_user_meta(
  int $user_id,
  string $meta_key,
  mixed $meta_value,
  mixed $prev_value = ''
);
```

关于所用参数的完整说明，请参阅 [`update_user_meta()`](https://developer.wordpress.org/reference/functions/update_user_meta/) 的函数参考。

#### 删除

```
delete_user_meta(
  int $user_id,
  string $meta_key,
  mixed $meta_value = ''
);
```

关于所用参数的完整说明，请参阅关于 [`delete_user_meta()`](https://developer.wordpress.org/reference/functions/delete_user_meta/) 的函数参考。

#### 获取

```
get_user_meta(
  int $user_id,
  string $key = '',
  bool $single = false
);
```

关于所用参数的完整说明，请参阅 [`get_user_meta()`](https://developer.wordpress.org/reference/functions/get_user_meta/) 的函数参考文档。

请注意，如果仅传递 `$user_id` 参数，该函数将检索所有元数据并以关联数组形式返回。

您可以在插件或主题的任何位置渲染用户元数据。