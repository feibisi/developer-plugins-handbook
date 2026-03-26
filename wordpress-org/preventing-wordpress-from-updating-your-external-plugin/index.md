---
title: "防止 WordPress 更新你的外部插件"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Preventing Wordpress From Updating Your External Plugin
    - Wordpress Org
    - Repos
---

# 防止 WordPress 更新你的外部插件

[warning]本页信息**仅**适用于未托管在 WordPress.org 上的插件。请勿对托管在官方目录中的代码使用此方法。[/warning]

如果你的插件托管在 WordPress.org 上，我们会为你处理所有更新。因此，本文档中的步骤**严禁**用于提交到官方目录的任何插件。任何托管于此的插件若被发现包含此类代码，将被关闭，开发者必须移除该代码才能恢复插件。

我们选择在此记录此方法，旨在教育开发者，并确保全球 WordPress 社区更加安全。

## 始终使用规范的文件夹名称

在深入代码之前，我们必须强调：确保插件不会被 WordPress.org 的更新覆盖的最佳方法就是使用规范的名称。如果您为公司开发插件，请使用类似 `companyname-function-plugin` 的文件夹命名方式——例如，如果您为 FaceRange 工作并开发状态插件，可将其命名为 `facerange-status-plugin`

这不仅因为使用了禁止词汇“plugin”而无法通过审核，插件团队还会验证插件所有者是否**合法**代表 FaceRange。

## 更新 URI

自 WordPress 5.8 起，我们为 WordPress.org API 的更新检查机制新增了一项功能，并允许通过一个新的头部字段来阻止更新：更新 URI。

假设你为自己网站制作了一个插件，并为其指定了文件夹名称 `my-plugin`。这是一个通用的文件夹名称，很可能被其他人使用。由于其通用性，我们也不会允许你在我们的系统中阻止此名称。

更新 URI 头部可以添加到插件头部信息中。请在你的主插件文件中查找以下部分：

```
/**
 * 插件名称：我的酷炫插件
 * 插件 URI：https://example.com/my-plugin/
 * 描述：我的插件能实现酷炫功能。
 * 版本：1.0
 * 作者：团队
 * 作者 URI：https://example.com/
 * 文本域：my-plugin
 * 许可证：GPLv2
 * 许可证 URI：https://opensource.org/licenses/gpl-2.0.php
 */
```

要应用此功能，请添加一个新的 **更新 URI** 头部，并在其值中填入一个**非** WordPress.org 的 URI：

```
 * 更新 URI：https://example.com/my-updater/
```

你也可以根据需要将其设置为 `更新 URI：false`。只要其中不包含 `wordpress.org/plugins` 或 `w.org/plugins`，它就会受到保护。

## Filtering Updates

Another method, which is supported on older versions of WordPress, is to filter external API requests and discard any for your plugin.

This code, which was written by [Mark Jaquith](https://markjaquith.wordpress.com/2009/12/14/excluding-your-plugin-or-theme-from-update-checks/), can be added to your own plugin:

```
function example_hidden_plugin_12345( $r, $url ) {
    if ( 0 !== strpos( $url, 'https://api.wordpress.org/plugins/update-check' ) )
        return $r; // Not a plugin update request. Bail immediately.

    $plugins = unserialize( $r['body']['plugins'] );
    unset( $plugins->plugins[ plugin_basename( __FILE__ ) ] );
    unset( $plugins->active[ array_search( plugin_basename( __FILE__ ), $plugins->active ) ] );
    $r['body']['plugins'] = serialize( $plugins );
    return $r;
}

add_filter( 'http_request_args', 'example_hidden_plugin_12345', 5, 2 );
```

What that does is check if the update request is from the WordPress.org api, and if it matches the plugin folder and file name of _this_ plugin. If it does, the plugin is removed from the list of plugins to check for updates.