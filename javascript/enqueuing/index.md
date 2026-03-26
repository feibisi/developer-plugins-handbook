---
title: "服务器端 PHP 与脚本入队"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Enqueuing
    - Javascript
    - Repos
---

# 服务器端 PHP 与脚本入队

实现 AJAX 通信所需的服务器端 PHP 脚本包含两个部分。首先需要在网页中入队 jQuery 脚本，并本地化 jQuery 脚本所需的 PHP 值。其次是实际处理 AJAX 请求的部分。

## 脚本入队

本节涵盖 WordPress 中 AJAX 的两个主要特性，常使经验丰富但不熟悉 WordPress 的开发者困惑。一是需要将脚本入队，以便元链接正确显示在页面头部区域。二是**所有** AJAX 请求都必须通过 `wp-admin/admin-ajax.php` 发送，切勿直接向插件页面发送请求。

### 入队

使用函数 [`wp_enqueue_script()`](https://developer.wordpress.org/reference/functions/wp_enqueue_script/) 让 WordPress 在页面的 `<head>` 部分插入指向你脚本的链接。切勿在头部模板中硬编码此类链接。作为插件开发者，你可能无法直接访问头部模板，但这条规则仍然值得强调。

入队函数接受五个参数如下：

- `$handle` 是脚本的名称。
- `$src` 定义脚本的位置。为了可移植性，请使用 `plugins_url()` 构建正确的 URL。如果你入队的脚本不是用于插件，请使用相关函数创建正确的 URL——切勿硬编码。
- `$deps` 是一个数组，可以处理你的新脚本所依赖的任何脚本，例如 jQuery。由于我们使用 jQuery 发送 AJAX 请求，你至少需要在数组中列出 `'jquery'`。
- `$ver` 允许你列出版本号。
- `$args` 是一个参数数组，用于定义页脚打印（通过 `in_footer` 键）和脚本加载策略（通过 `strategy` 键），例如 `defer` 或 `async`。从 WordPress 6.3 版本开始，这取代/重载了 `$in_footer` 参数。

```
wp_enqueue_script(
  'ajax-script',
  plugins_url( '/js/myjquery.js', __FILE__ ),
  array( 'jquery' ),
  '1.0.0',
  array(
    'in_footer' => true,
  )
);
```

你不能在插件代码页面加载时直接入队脚本。脚本必须通过几个动作钩子之一入队——具体使用哪个钩子取决于脚本需要链接到哪种页面。对于管理页面，使用 `admin_enqueue_scripts`。对于前端页面，使用 `wp_enqueue_scripts`，但登录页面除外，此时应使用 `login_enqueue_scripts`。

`admin_enqueue_scripts` 钩子会将当前页面文件名传递给回调函数。利用此信息，仅在需要的页面上入队你的脚本。前端版本不传递任何参数。在这种情况下，使用模板标签如 `is_home()`、`is_single()` 等，以确保仅在需要的地方入队脚本。这是我们示例的完整入队代码：

```
add_action( 'admin_enqueue_scripts', 'my_enqueue' );
function my_enqueue( $hook ) {
  if ( 'myplugin_settings.php' !== $hook ) {
    return;
  }
  wp_enqueue_script(
    'ajax-script',
    plugins_url( '/js/myjquery.js', __FILE__ ),
    array( 'jquery' ),
    '1.0.0',
    array(
      'in_footer' => true,
    )
  );
}
```

为什么我们在这里使用命名函数，而在 jQuery 中使用匿名函数？因为闭包直到最近才被 PHP 支持。jQuery 支持闭包已经有一段时间了。由于有些人可能仍在运行旧版本的 PHP，我们总是使用命名函数以确保最大兼容性。如果你使用的是较新的 PHP 版本，并且只为你自己的安装进行开发，那么可以随意使用闭包。

#### 注册与入队

您会在其他教程中看到严格使用 [`wp_register_script()`](https://developer.wordpress.org/reference/functions/wp_register_script/) 的示例。这没问题，但它的使用是可选的。而 `wp_enqueue_script()` 则是必需的。必须调用此函数，您的脚本文件才能正确链接到网页上。那么为什么要注册脚本呢？它会创建一个有用的标签或句柄，您可以根据需要在代码的各个部分轻松引用该脚本。如果您只需要加载脚本且不在代码的其他地方引用它，则无需注册。

#### 延迟脚本加载

WordPress 通过 `wp_register_script()` 和 `wp_enqueue_script()` 函数提供对脚本加载策略的支持，这是通过 WordPress 6.3 中引入的新 `$args` 数组参数中的 `strategy` 键实现的。

支持的策略如下：

- **defer**
  - 通过向 `$args` 参数添加数组键值对 `'strategy' => 'defer'` 来指定。
  - 标记为延迟执行的脚本（通过 `defer` 脚本属性）仅在 DOM 树完全加载后执行（但在 `DOMContentLoaded` 和窗口加载事件之前）。与异步脚本不同，延迟脚本按照它们在 DOM 中打印/添加的顺序执行。
- **async**
  - 通过向 `$args` 参数添加数组键值对 `'strategy' => 'async'` 来指定。
  - 标记为异步执行的脚本（通过 `async` 脚本属性）在浏览器加载完成后立即执行。异步脚本没有保证的执行顺序，因为脚本 B（尽管在脚本 A 之后添加到 DOM 中）可能先执行，前提是它可能在脚本 A 之前完成加载。此类脚本可能在 DOM 完全构建之前或 `DOMContentLoaded` 事件之后执行。

以下是在我们的插件中为额外脚本入队指定加载策略的示例：

```
wp_register_script(
  'ajax-script-two',
  plugins_url( '/js/myscript.js', __FILE__ ),
  array( ajax-script ),
  '1.0.,0',
  array(
    'strategy' => 'defer',
  )
);
```

使用 `wp_enqueue_script()` 时也适用相同的方法。在上面的示例中，我们表明我们打算以延迟方式加载 `'ajax-script-two'` 脚本。

在指定延迟脚本加载策略时，系统会考虑脚本的依赖关系树（其依赖项和/或被依赖项）来决定“合格策略”，以避免应用对某个脚本有效但对树中其他脚本有害的策略，从而导致意外的执行顺序错误。由于这种逻辑，您通过 `$args` 参数传递的预期加载策略可能不是最终（选定）的策略，但它永远不会比预期策略更有害（或更严格）。

### 随机数

你需要创建一个随机数，以便 jQuery AJAX 请求能够被验证为合法请求，而非来自未知恶意方的潜在有害请求。只有你的 PHP 脚本和 jQuery 脚本会知道这个值。当收到请求时，你可以验证它是否与此处创建的值相同。以下是为我们的示例创建随机数的方法：

```
$title_nonce = wp_create_nonce( 'title_example' );
```

参数 `title_example` 可以是任意字符串。建议该字符串与随机数的用途相关，但实际上可以是任何适合你的内容。

### 本地化

如果您还记得 [jQuery 章节](https://developer.wordpress.org/plugins/javascript/jquery/) 的内容，PHP 创建供 jQuery 使用的数据是通过一个名为 `my_ajax_obj` 的全局对象传递的。在我们的示例中，这些数据是一个随机数（nonce）以及 `admin-ajax.php` 的完整 URL。分配对象属性并创建全局 jQuery 对象的过程称为**本地化**。以下是我们示例中使用的本地化代码，它使用了 [`wp_localize_script()`](https://developer.wordpress.org/reference/functions/wp_localize_script/) 函数。

```
wp_localize_script(
  'ajax-script',
  'my_ajax_obj',
  array(
    'ajax_url' => admin_url( 'admin-ajax.php' ),
    'nonce'    => $title_nonce,
  )
);
```

请注意，我们使用了脚本句柄 `ajax-script`，以确保全局对象被分配给正确的脚本。该对象仅对我们的脚本是全局的，而不是对所有脚本。本地化也可以从用于入队脚本的同一个钩子中调用。创建随机数也是如此，尽管该特定函数几乎可以在任何地方调用。所有这些组合在一个钩子回调函数中，看起来像这样：

```
add_action( 'admin_enqueue_scripts', 'my_enqueue' );

/**
 * 入队我的脚本和资源。
 *
 * @param $hook
 */
function my_enqueue( $hook ) {
  if ( 'myplugin_settings.php' !== $hook ) {
    return;
  }
  wp_enqueue_script(
    'ajax-script',
    plugins_url( '/js/myjquery.js', __FILE__ ),
    array( 'jquery' ),
    '1.0.0',
    true
  );

  wp_localize_script(
    'ajax-script',
    'my_ajax_obj',
    array(
      'ajax_url' => admin_url( 'admin-ajax.php' ),
      'nonce'    => wp_create_nonce( 'title_example' ),
    )
  );
}
```

[info]请记住，仅将随机数本地化添加到需要的页面，不要向不应使用它的人显示随机数。并且记得使用 `current_user_can()` 配合权限或角色来完善安全性。[/info]

## AJAX Action

The other major part of the server side PHP code is the actual AJAX handler that receives the POSTed data, does something with it, then sends an appropriate response back to the browser. This takes on the form of a WordPress [action hook](https://developer.wordpress.org/plugins/hooks/actions/). Which hook tag you use depends on whether the user is logged in or not and what value your jQuery script passed as the _action:_ value.

[info]`$_GET`, `$_POST`, and `$_COOKIE` vs. `$_REQUEST`

You've probably used one or more of the PHP super globals such as `$_GET` or `$_POST` to retrieve values from forms or cookies (using `$_COOKIE`). Maybe you prefer `$_REQUEST` instead, or at least have seen it used. It's kind of cool – regardless of the request method, `POST` or `GET`, it will have the form values. Works great for pages that use both methods. On top of that, it has cookie values as well. One stop shopping! Therein lies its tragic flaw. In the case of a name conflict, the cookie value will override any form values. Thus it is ridiculously easy for a bad actor to craft a counterfeit cookie on their browser, which will overwrite any form value you might be expecting from the request. `$_REQUEST` is an easy route for hackers to inject arbitrary data into your form values. To be extra safe, stick to the specific variables and avoid the one size fits all.[/info]

Since our AJAX exchange is for the plugin's settings page, the user must be logged in. If you recall from the [jQuery section](https://developer.wordpress.org/plugins/javascript/jquery/), the `action:` value is `"my_tag_count"`. This means our action hook tag will be `wp_ajax_my_tag_count`. If our AJAX exchange were to be utilized by users who were not currently logged in, the action hook tag would be `wp_ajax_nopriv_my_tag_count` The basic code used to hook the action looks like this:

```
add_action( 'wp_ajax_my_tag_count', 'my_ajax_handler' );

/**
 * Handles my AJAX request.
 */
function my_ajax_handler() {
  // Handle the ajax request here

  wp_die(); // All ajax handlers die when finished
}
```

The first thing your AJAX handler should do is verify the nonce sent by jQuery with [`check_ajax_referer()`](https://developer.wordpress.org/reference/functions/check_ajax_referer/), which should be the same value that was localized when the script was enqueued.

```
check_ajax_referer( 'title_example' );
```

The provided parameter must be identical to the parameter provided [earlier](https://developer.wordpress.org/plugins/javascript/enqueuing/#php-nonce) to `wp_create_nonce()`. The function simply dies if the nonce does not check out. If this were a true nonce, now that it was used, the value is no longer any good. You would then generate a new one and send it to the callback script so that it can be used for the next request. But since WordPress nonces are good for twenty-four hours, you needn't do anything but check it.

### 数据

处理完 nonce 后，我们的处理器就可以处理 jQuery 脚本通过 `$_POST['title']` 发送的数据了。首先，我们通过 [`wp_unslash()`](https://developer.wordpress.org/reference/functions/wp_unslash/) 函数运行该值以移除任何意外的引号，然后将其赋值给一个新变量。

```
$title = wp_unslash( $_POST['title'] );
```

我们可以使用 [`update_user_meta()`](https://developer.wordpress.org/reference/functions/update_user_meta/) 将用户的选择保存到用户元数据中。

```
update_user_meta( get_current_user_id(), 'title_preference', sanitize_post_title( $title ) );
```

接着，我们构建一个查询以获取所选标题标签的文章数量。

```
$args      = array(
  'tag' => $title,
);
$the_query = new WP_Query( $args );
```

最后，我们可以将响应发送回 jQuery 脚本。传输数据有几种方式。在深入我们示例的具体细节之前，先来看看一些可选的方案。

#### XML

PHP 对 XML 的支持尚有不足。幸运的是，WordPress 提供了 [`WP_Ajax_Response`](https://developer.wordpress.org/reference/classes/wp_ajax_response/) 类来简化这一任务。该类会生成 XML 格式的响应，设置正确的头部内容类型，输出响应 XML 后终止执行，从而确保返回正确的 XML 响应。

#### JSON

这种格式轻量且易于使用，WordPress 提供了 [`wp_send_json`](https://developer.wordpress.org/reference/functions/wp_send_json/) 函数来对响应进行 JSON 编码、输出并终止执行——这有效地替代了 [`WP_Ajax_Response`](https://developer.wordpress.org/reference/classes/wp_ajax_response/)。WordPress 还提供了 [`wp_send_json_success`](https://developer.wordpress.org/reference/functions/wp_send_json_success/) 和 [`wp_send_json_error`](https://developer.wordpress.org/reference/functions/wp_send_json_error/) 函数，它们允许在 JavaScript 中触发相应的 done() 或 fail() 回调。

#### 其他

您可以使用任何喜欢的方式传输数据，只要发送方和接收方协调一致即可。逗号分隔或制表符分隔等文本格式只是众多可能性之一。对于少量数据，直接发送原始数据流可能就足够了。这就是我们将在示例中采用的方式——我们将仅发送实际的替换 HTML，不包含其他内容。

```
echo esc_html( $title ) . ' (' . $the_query->post_count . ') ';
```

在实际应用中，您必须考虑到操作可能因某些原因失败的可能性——例如，数据库服务器可能宕机。响应应考虑到这种意外情况，接收响应的 jQuery 脚本也应相应处理，例如告知用户稍后重试。

### 终止处理

当处理器完成所有任务后，需要终止运行。如果使用 [`WP_Ajax_Response`](https://developer.wordpress.org/reference/classes/wp_ajax_response/) 或 `wp_send_json*` 系列函数，系统会自动处理此步骤。否则，直接调用 WordPress 的 [`wp_die()`](https://developer.wordpress.org/reference/functions/wp_die/) 函数即可。

### AJAX Handler Summary

The complete AJAX handler for our example looks like this:

```
/**
 * AJAX handler using JSON
 */
function my_ajax_handler__json() {
  check_ajax_referer( 'title_example' );
  $title = wp_unslash( $_POST['title'] );

  update_user_meta( get_current_user_id(), 'title_preference', sanitize_post_title( $title ) );

  $args      = array(
    'tag' => $title,
  );
  $the_query = new WP_Query( $args );
  wp_send_json( esc_html( $title ) . ' (' . $the_query->post_count . ') ' );
}

/**
 * AJAX handler not using JSON.
 */
function my_ajax_handler() {
  check_ajax_referer( 'title_example' );
  $title = wp_unslash( $_POST['title'] );

  update_user_meta( get_current_user_id(), 'title_preference', sanitize_post_title( $title ) );

  $args      = array(
    'tag' => $title,
  );
  $the_query = new WP_Query( $args );
  echo esc_html( $title ) . ' (' . $the_query->post_count . ') ';
  wp_die(); // All ajax handlers should die when finished
}
```