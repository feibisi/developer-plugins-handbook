---
title: "Index"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Adding The Personal Data Exporter To Your Plugin
    - Privacy
    - Repos
---

## 为插件添加个人数据导出功能

在 WordPress 4.9.6 版本中，新增了工具以简化对欧盟《通用数据保护条例》（简称 GDPR）等法律的合规流程。新增工具中包含个人数据导出功能，支持将指定用户的所有个人数据以 ZIP 文件格式导出。除了存储在 WordPress 评论等模块中的个人数据外，插件也可以通过挂接导出器功能来导出其收集的个人数据，无论这些数据是存储在 postmeta 中还是全新的自定义文章类型（CPT）中。

所有导出的"关键标识"是用户的电子邮件地址——选择此标识是因为它既支持导出已注册用户的个人数据，也支持导出未注册用户（例如未登录的评论者）的数据。

但由于整理个人数据导出可能涉及大量处理过程且可能包含敏感信息，我们不会未经确认就直接生成并发送给请求者。因此，面向管理员的操作界面会要求管理员先输入请求者的用户名或邮箱地址，然后向请求者发送确认链接。

确认请求后，管理员可以生成并下载或直接通过邮件发送个人数据导出 ZIP 文件，也可在需要时随时执行导出操作。用户收到的 ZIP 文件中包含一个"微型网站"，其中的 HTML 索引页面将个人数据按组分类呈现（例如评论组等）。

无论管理员选择下载还是直接发送导出文件，个人数据的汇编方式完全一致——都依赖于挂接"导出器"回调函数来执行数据收集工作。当管理员点击下载或邮件链接时，系统会通过 AJAX 循环依次调用所有已注册的导出器。除了核心内置的导出器，插件也可以注册自己的导出器回调函数。

导出器回调接口的设计力求简洁。导出器回调函数会接收待处理的邮箱地址和页码参数。页码参数（从 1 开始）用于避免插件因一次性导出所有收集数据而导致超时。规范的插件会限制每页处理的数据量（例如 100 篇文章、200 条评论等）。

The exporter callback replies with whatever data it has for that email address and page and whether it is done or not. If a exporter callback reports that it is not done, it will be called again (in a separate request) with the page parameter incremented by 1. Exporter callbacks are expected to return an array of items for the export. Each item contains an a group identifier for the group of which the item is a part (e.g. comments, posts, orders, etc.), an optional group label (translated), an item identifier (e.g. comment-133) and then an array of name, value pairs containing the data to be exported for that item.

It is noteworthy that the value could be a media path, in which case a link to the media file will be added to the index HTML page in the export.

When all the exporters have been called to completion, WordPress first assembles an "index" HTML document that serves as the heart of the export report. If a plugin reports additional data for an item that WordPress or another plugin has already added, all the data for that item will be presented together.

Exports are cached on the server for 3 days and then deleted.

A plugin can register one or more exporters, but most plugins will only need one. Let's work on a hypothetical plugin which adds location data for the commenter to comments.

First, let's assume the plugin has used [`add_comment_meta`](https://developer.wordpress.org/reference/functions/add_comment_meta/) to add location data using `meta_key`'s of `latitude` and `longitude`.

The first thing the plugin needs to do is to create an exporter function that accepts an email address and a page, e.g.:

```
/**
 * Export user meta for a user using the supplied email.
 *
 * @param string $email_address   email address to manipulate
 * @param int    $page            pagination
 *
 * @return array
 */
function wporg_export_user_data_by_email( $email_address, $page = 1 ) {
  $number = 500; // Limit us to avoid timing out
  $page   = (int) $page;

  $export_items = array();

  $comments = get_comments(
    array(
      'author_email' => $email_address,
      'number'       => $number,
      'paged'        => $page,
      'order_by'     => 'comment_ID',
      'order'        => 'ASC',
    )
  );

  foreach ( (array) $comments as $comment ) {
    $latitude  = get_comment_meta( $comment->comment_ID, 'latitude', true );
    $longitude = get_comment_meta( $comment->comment_ID, 'longitude', true );

    // Only add location data to the export if it is not empty.
    if ( ! empty( $latitude ) ) {
      // Most item IDs should look like postType-postID. If you don't have a post, comment or other ID to work with,
      // use a unique value to avoid having this item's export combined in the final report with other items
      // of the same id.
      $item_id = "comment-{$comment->comment_ID}";

      // Core group IDs include 'comments', 'posts', etc. But you can add your own group IDs as needed
      $group_id = 'comments';

// 可选的组标签。核心功能为核心组提供这些标签。如果你定义自己的组，第一个
      // 包含标签的导出器将被用作最终导出报告中的组标签。
      $group_label = __( 'Comments', 'text-domain' );

      // 插件可以在项目数据数组中添加任意数量的项目。
      $data = array(
        array(
          'name'  => __( 'Commenter Latitude', 'text-domain' ),
          'value' => $latitude,
        ),
        array(
          'name'  => __( 'Commenter Longitude', 'text-domain' ),
          'value' => $longitude,
        ),
      );

      $export_items[] = array(
        'group_id'    => $group_id,
        'group_label' => $group_label,
        'item_id'     => $item_id,
        'data'        => $data,
      );
    }
  }

  // 告知核心是否还有更多评论需要处理。
  $done = count( $comments ) > $number;
  return array(
    'data' => $export_items,
    'done' => $done,
  );
}
```

插件接下来需要做的是通过使用 [`wp_privacy_personal_data_exporters`](https://developer.wordpress.org/reference/hooks/wp_privacy_personal_data_exporters/) 过滤器过滤导出器数组来注册回调函数。

注册时，你需要为导出提供一个友好的名称（用于辅助调试——目前此友好名称不会向任何人显示）以及回调函数，例如：

```
/**
 * 注册所有数据导出器。
 *
 * @param array $exporters
 *
 * @return mixed
 */
function wporg_register_user_data_exporters( $exporters ) {
  $exporters['my-plugin-slug'] = array(
    'exporter_friendly_name' => __( 'Comment Location Plugin', 'text-domain' ),
    'callback'               => 'my_plugin_exporter',
  );
  return $exporters;
}

add_filter( 'wp_privacy_personal_data_exporters', 'wporg_register_user_data_exporters' );
```

这样就完成了！你的插件现在将为导出提供数据！