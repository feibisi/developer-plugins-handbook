---
title: "自定义元框"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Custom Meta Boxes
    - Metadata
    - Repos
---

# 自定义元框

## 什么是元框？

当用户编辑文章时，编辑界面由多个默认框组成：编辑器、发布、分类、标签等。这些框就是元框。插件可以向任何文章类型的编辑界面添加自定义元框。

自定义元框的内容通常是 HTML 表单元素，用户在其中输入与插件功能相关的数据，但其内容实际上可以是您需要的任何 HTML。

## 为何使用元数据框？

元数据框是便捷、灵活、模块化的编辑屏幕元素，可用于收集与正在编辑的文章相关的信息。您的自定义元数据框将与所有其他文章相关信息显示在同一屏幕上，从而建立清晰的关系。

元数据框可以轻松对不需要查看的用户隐藏，而对需要查看的用户显示。用户可以在编辑屏幕上自由排列元数据框，从而控制其编辑环境。

[alert]本页所有示例仅用于说明目的。代码不适用于生产环境。

诸如[输入安全处理](https://developer.wordpress.org/apis/security/sanitizing/)、[用户权限](https://developer.wordpress.org/plugins/security/checking-user-capabilities/)、[随机数](https://developer.wordpress.org/apis/security/nonces/)和[国际化](https://developer.wordpress.org/plugins/internationalization/)等操作已被有意省略。请务必始终处理这些重要操作。[/alert]

## Adding Meta Boxes

To create a meta box use the [`add_meta_box()`](https://developer.wordpress.org/reference/functions/add_meta_box/) function and plug its execution to the [`add_meta_boxes`](https://developer.wordpress.org/reference/hooks/add_meta_boxes/) action hook.

The following example is adding a meta box to the `post` edit screen and the `wporg_cpt` edit screen.

```
function wporg_add_custom_box() {
  $screens = [ 'post', 'wporg_cpt' ];
  foreach ( $screens as $screen ) {
    add_meta_box(
      'wporg_box_id',                 // Unique ID
      'Custom Meta Box Title',      // Box title
      'wporg_custom_box_html',  // Content callback, must be of type callable
      $screen                            // Post type
    );
  }
}
add_action( 'add_meta_boxes', 'wporg_add_custom_box' );
```

The `wporg_custom_box_html` function will hold the HTML for the meta box.

The following example is adding form elements, labels, and other HTML elements.

```
function wporg_custom_box_html( $post ) {
  ?>
  <label for="wporg_field">Description for this field</label>
  <select name="wporg_field" id="wporg_field" class="postbox">
    <option value="">Select something...</option>
    <option value="something">Something</option>
    <option value="else">Else</option>
  </select>
  <?php
}
```

[info]**Note there are no submit buttons in meta boxes.** The meta box HTML is included inside the edit screen's form tags, all the post data including meta box values are transfered via `POST` when the user clicks on the Publish or Update buttons.[/info]

The example shown here only contains one form field, a drop down list. You may create as many as needed in any particular meta box. If you have a lot of fields to display, consider using multiple meta boxes, grouping similar fields together in each meta box. This helps keep the page more organized and visually appealing.

### Getting Values

To retrieve saved user data and make use of it, you need to get it from wherever you saved it initially. If it was stored in the `postmeta` table, you may get the data with [`get_post_meta()`](https://developer.wordpress.org/reference/functions/get_post_meta/).

The following example enhances the previous form elements with pre-populated data based on saved meta box values. You will learn how to save meta box values in the next section.

```
function wporg_custom_box_html( $post ) {
  $value = get_post_meta( $post->ID, '_wporg_meta_key', true );
  ?>
  <label for="wporg_field">Description for this field</label>
    <select name="wporg_field" id="wporg_field" class="postbox">
      <option value="">Select something...</option>
      <option value="something" <?php selected( $value, 'something' ); ?>>Something</option>
      <option value="else" <?php selected( $value, 'else' ); ?>>Else</option>
    </select>
    <?php
}
```

More on the [`selected()`](https://developer.wordpress.org/reference/functions/selected/) function.

### 保存数值

当文章类型被保存或更新时，会触发多个操作，其中任何一个都可能适合挂钩以保存输入的值。在本示例中，我们使用 `save_post` 动作钩子，但在某些情况下，其他钩子可能更合适。请注意，对于单个更新事件，`save_post` 可能会触发多次。请相应地构建保存数据的方法。

您可以将输入的数据保存在任何地方，甚至可以在 WordPress 之外。由于您可能处理与文章相关的数据，`postmeta` 表通常是存储数据的好地方。

以下示例将把 `wporg_field` 字段值保存在隐藏的元键 `_wporg_meta_key` 中。

```
function wporg_save_postdata( $post_id ) {
  if ( array_key_exists( 'wporg_field', $_POST ) ) {
    update_post_meta(
      $post_id,
      '_wporg_meta_key',
      $_POST['wporg_field']
    );
  }
}
add_action( 'save_post', 'wporg_save_postdata' );
```

在生产代码中，请记住遵循信息框中概述的安全措施！

## 幕后机制

通常您无需关心幕后的运行机制。本节内容是为了保持文档完整性而添加。

当文章编辑界面需要显示所有已添加的元数据框时，它会调用 [`do_meta_boxes()`](https://developer.wordpress.org/reference/functions/do_meta_boxes/) 函数。该函数会遍历所有元数据框，并触发每个框关联的 `callback` 回调函数。

在每次调用之间，系统会插入相应的标记元素（例如 div 容器、标题等）。

## 移除元框

要从编辑屏幕移除现有的元框，请使用 [`remove_meta_box()`](https://developer.wordpress.org/reference/functions/remove_meta_box/) 函数。传递的参数必须与使用 [`add_meta_box()`](https://developer.wordpress.org/reference/functions/add_meta_box/) 添加元框时所用的参数完全匹配。

要移除默认的元框，请检查源代码中使用的参数。默认的 [`add_meta_box()`](https://developer.wordpress.org/reference/functions/add_meta_box/) 调用位于 `wp-includes/edit-form-advanced.php` 文件中。

## Implementation Variants

So far we've been using the procedural technique of implementing meta boxes. Many plugin developers find the need to implement meta boxes using various other techniques.

### OOP

Adding meta boxes using OOP is easy and saves you from having to worry about naming collisions in the global namespace.

To save memory and allow easier implementation, the following example uses an abstract Class with static methods.

```
abstract class WPOrg_Meta_Box {

  /**
   * Set up and add the meta box.
   */
  public static function add() {
    $screens = [ 'post', 'wporg_cpt' ];
    foreach ( $screens as $screen ) {
      add_meta_box(
        'wporg_box_id',          // Unique ID
        'Custom Meta Box Title', // Box title
        [ self::class, 'html' ],   // Content callback, must be of type callable
        $screen                  // Post type
      );
    }
  }

  /**
   * Save the meta box selections.
   *
   * @param int $post_id  The post ID.
   */
  public static function save( int $post_id ) {
    if ( array_key_exists( 'wporg_field', $_POST ) ) {
      update_post_meta(
        $post_id,
        '_wporg_meta_key',
        $_POST['wporg_field']
      );
    }
  }

  /**
   * Display the meta box HTML to the user.
   *
   * @param WP_Post $post   Post object.
   */
  public static function html( $post ) {
    $value = get_post_meta( $post->ID, '_wporg_meta_key', true );
    ?>
    <label for="wporg_field">Description for this field</label>
    <select name="wporg_field" id="wporg_field" class="postbox">
      <option value="">Select something...</option>
      <option value="something" <?php selected( $value, 'something' ); ?>>Something</option>
      <option value="else" <?php selected( $value, 'else' ); ?>>Else</option>
    </select>
    <?php
  }
}

add_action( 'add_meta_boxes', [ 'WPOrg_Meta_Box', 'add' ] );
add_action( 'save_post', [ 'WPOrg_Meta_Box', 'save' ] );
```

### AJAX

由于元框的 HTML 元素位于编辑屏幕的 `form` 标签内，默认行为是在用户提交页面后从 `$_POST` 超级全局变量中解析元框值。

您可以使用 AJAX 增强默认体验；这允许根据用户输入和行为执行操作，无论他们是否已提交页面。

#### Define a Trigger

First, you must define the trigger, this can be a link click, a change of a value or any other JavaScript event.

In the example below we will define `change` as our trigger for performing an AJAX request.

```
/*jslint browser: true, plusplus: true */
(function ($, window, document) {
  'use strict';
  // execute when the DOM is ready
  $(document).ready(function () {
    // js 'change' event triggered on the wporg_field form field
    $('#wporg_field').on('change', function () {
      // our code
    });
  });
}(jQuery, window, document));
```

#### Client Side Code

Next, we need to define what we want the trigger to do, in other words we need to write our client side code.

In the example below we will make a `POST` request, the response will either be success or failure, this will indicate wither the value of the `wporg_field` is valid.

```
/*jslint browser: true, plusplus: true */
(function ($, window, document) {
  'use strict';
  // execute when the DOM is ready
  $(document).ready(function () {
    // js 'change' event triggered on the wporg_field form field
    $('#wporg_field').on('change', function () {
      // jQuery post method, a shorthand for $.ajax with POST
      $.post(wporg_meta_box_obj.url,                        // or ajaxurl
        {
          action: 'wporg_ajax_change',                // POST data, action
          wporg_field_value: $('#wporg_field').val(), // POST data, wporg_field_value
          post_ID: jQuery('#post_ID').val()           // The ID of the post currently being edited
        }, function (data) {
          // handle response data
          if (data === 'success') {
            // perform our success logic
          } else if (data === 'failure') {
            // perform our failure logic
          } else {
            // do nothing
          }
        }
      );
    });
  });
}(jQuery, window, document));
```

We took the WordPress AJAX file URL dynamically from the `wporg_meta_box_obj` JavaScript custom object that we will create in the next step.

[info]If your meta box only requires the WordPress AJAX file URL; instead of creating a new custom JavaScript object you could use the `ajaxurl` predefined JavaScript variable.

**Available only in the WordPress Administration.** Make sure it is not empty before performing any logic.[/info]

#### Enqueue Client Side Code

Next step is to put our code into a script file and enqueue it on our edit screens.

In the example below we will add the AJAX functionality to the edit screens of the following post types: post, `wporg_cpt`.

The script file will reside at `/plugin-name/admin/meta-boxes/js/admin.js`, `plugin-name` being the main plugin folder, `/plugin-name/plugin.php` the file calling the function.

```
function wporg_meta_box_scripts() {
  // get current admin screen, or null
  $screen = get_current_screen();
  // verify admin screen object
  if (is_object($screen)) {
    // enqueue only for specific post types
    if (in_array($screen->post_type, ['post', 'wporg_cpt'])) {
      // enqueue script
      wp_enqueue_script('wporg_meta_box_script', plugin_dir_url(__FILE__) . 'admin/meta-boxes/js/admin.js', ['jquery']);
      // localize script, create a custom js object
      wp_localize_script(
        'wporg_meta_box_script',
        'wporg_meta_box_obj',
        [
          'url' => admin_url('admin-ajax.php'),
        ]
      );
    }
  }
}
add_action('admin_enqueue_scripts', 'wporg_meta_box_scripts');
```

#### Server Side Code

The last step is to write our server side code that is going to handle the request.

```
// The piece after `wp_ajax_`  matches the action argument being sent in the POST request.
add_action( 'wp_ajax_wporg_ajax_change', 'my_ajax_handler' );

/**
 * Handles my AJAX request.
 */
function my_ajax_handler() {
  // Handle the ajax request here
  if ( array_key_exists( 'wporg_field_value', $_POST ) ) {
    $post_id = (int) $_POST['post_ID'];
    if ( current_user_can( 'edit_post', $post_id ) ) {
      update_post_meta(
        $post_id,
        '_wporg_meta_key',
        $_POST['wporg_field_value']
      );
    }
  }

  wp_die(); // All ajax handlers die when finished
}
```

As a final reminder, the code illustrated on this page lacks important operations that take care of security. Be sure your production code includes such operations.

See [Handbook's AJAX Chapter](https://developer.wordpress.org/plugins/javascript/ajax/) and the [Codex](https://codex.wordpress.org/AJAX_in_Plugins) for more on AJAX.

## More Information

- [Complex Meta Boxes in WordPress](https://www.wproots.com/complex-meta-boxes-in-wordpress/)
- [How To Create Custom Post Meta Boxes In WordPress](https://www.smashingmagazine.com/2011/10/create-custom-post-meta-boxes-wordpress/)
- [WordPress Meta Boxes: a Comprehensive Developer's Guide](https://themefoundation.com/wordpress-meta-boxes-guide/)