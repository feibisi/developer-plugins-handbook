---
title: "为您的插件添加个人数据擦除功能"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Adding The Personal Data Eraser To Your Plugin
    - Privacy
    - Repos
---

# 为您的插件添加个人数据擦除功能

在 WordPress 4.9.6 版本中，新增了工具以简化对欧盟《通用数据保护条例》（简称 GDPR）等法规的合规操作。新增工具包括个人数据移除工具，支持擦除/匿名化指定用户的个人数据。该工具**不会**删除已注册的用户账户——这仍是管理员可选择是否执行的独立步骤。

除了存储在 WordPress 评论等内容中的个人数据外，插件还可以通过钩子接入擦除功能，以清除它们收集的个人数据，无论这些数据是存储在 postmeta 中还是全新的自定义文章类型（CPT）中。

与导出器类似，所有擦除器的“密钥”是用户的电子邮件地址——选择此方式是因为它既支持擦除已注册用户的个人数据，也支持擦除未注册用户（例如未登录的评论者）的数据。

然而，由于执行个人数据擦除是破坏性操作，我们不能未经确认就直接执行。因此，面向管理员的用户界面会要求管理员输入提出请求的用户名或电子邮件地址来启动所有请求，然后向该地址发送确认链接。请求确认后，管理员可以启动用户的个人数据擦除，或在需要时强制执行。

个人数据导出的擦除方式与个人数据导出器类似——依赖于挂接“擦除器”回调函数来执行数据擦除的实际工作。当管理员点击移除个人数据链接时，系统会启动一个 AJAX 循环，逐个遍历系统中注册的所有擦除器。除了核心内置的擦除器，插件也可以注册自己的擦除器回调函数。

擦除器回调接口的设计力求简洁。擦除器回调函数接收待处理的电子邮件地址和页码参数。页码参数（从 1 开始）用于避免插件因尝试一次性擦除所有收集的个人数据而导致超时。规范的插件会限制每页尝试擦除的数据量（例如 100 篇文章、200 条评论等）。

擦除器回调函数会返回：是否已擦除包含个人数据的项目、是否保留了任何包含个人数据的项目、向管理员显示的消息数组（解释保留项目的原因）以及操作是否完成。如果擦除器回调报告未完成，系统将以页码参数加 1 的方式再次调用该回调函数（通过独立请求）。

当所有导出器都完成处理后，管理员界面将更新显示是否已擦除所有发现的个人数据，以及解释个人数据保留原因的相关消息。

Let's work on a hypothetical plugin which adds commenter location data to comments. Let's assume the plugin has used [`add_comment_meta`](https://developer.wordpress.org/reference/functions/add_comment_meta/) to add location data using `meta_key`'s of `latitude` and `longitude`.

The first thing the plugin needs to do is to create an eraser function that accepts an email address and a page, e.g.:

```
/**
 * Removes any stored location data from a user's comment meta for the supplied email address.
 *
 * @param string $email_address   email address to manipulate
 * @param int    $page            pagination
 *
 * @return array
 */
function wporg_remove_location_meta_from_comments_for_email( $email_address, $page = 1 ) {
  $number = 500; // Limit us to avoid timing out
  $page   = (int) $page;

  $comments = get_comments(
    array(
      'author_email' => $email_address,
      'number'       => $number,
      'paged'        => $page,
      'order_by'     => 'comment_ID',
      'order'        => 'ASC',
    )
  );

  $items_removed = false;

  foreach ( (array) $comments as $comment ) {
    $latitude  = get_comment_meta( $comment->comment_ID, 'latitude', true );
    $longitude = get_comment_meta( $comment->comment_ID, 'longitude', true );

    if ( ! empty( $latitude ) ) {
      delete_comment_meta( $comment->comment_ID, 'latitude' );
      $items_removed = true;
    }

    if ( ! empty( $longitude ) ) {
      delete_comment_meta( $comment->comment_ID, 'longitude' );
      $items_removed = true;
    }
  }

  // Tell core if we have more comments to work on still
  $done = count( $comments ) < $number;
  return array(
    'items_removed'  => $items_removed,
    'items_retained' => false, // always false in this example
    'messages'       => array(), // no messages in this example
    'done'           => $done,
  );
}
```

The next thing the plugin needs to do is to register the callback by filtering the eraser array using the [`wp_privacy_personal_data_erasers`](https://developer.wordpress.org/reference/hooks/wp_privacy_personal_data_erasers/) filter.

When registering you provide a friendly name for the eraser (to aid in debugging – this friendly name is not shown to anyone at this time) and the callback, e.g.

```
/**
 * Registers all data erasers.
 *
 * @param array $exporters
 *
 * @return mixed
 */
function wporg_register_privacy_erasers( $erasers ) {
  $erasers['my-plugin-slug'] = array(
    'eraser_friendly_name' => __( 'Comment Location Plugin', 'text-domain' ),
    'callback'             => 'wporg_remove_location_meta_from_comments_for_email',
  );
  return $erasers;
}

add_filter( 'wp_privacy_personal_data_erasers', 'wporg_register_privacy_erasers' );
```

And that's all there is to it! Your plugin will now clean up its personal data!