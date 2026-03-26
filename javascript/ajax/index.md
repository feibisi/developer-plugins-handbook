---
title: "AJAX"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Ajax
    - Javascript
    - Repos
---

# AJAX

## 什么是 AJAX？

AJAX 是 Asynchronous JavaScript And XML（异步 JavaScript 和 XML）的缩写。XML 是一种数据交换格式，而 UX 是开发者对用户体验的简称。Ajax 是一种互联网通信技术，允许用户浏览器中显示的网页向服务器请求特定信息，并在同一页面显示新信息，而无需重新加载整个页面。您已经可以想象这如何改善用户体验。

虽然 XML 是传统使用的数据交换格式，但实际交换可以采用任何方便的格式。在使用 PHP 代码时，许多开发者偏爱 JSON，因为从传输数据流创建的内部数据结构更易于对接。

要查看 AJAX 的实际效果，请进入您的 WordPress 管理区域并添加一个分类或标签。当您点击“添加新”按钮时请仔细观察，注意页面发生变化但并未实际重新加载。不相信吗？检查浏览器的后退历史记录，如果页面已重新加载，您会看到该页面的两个条目。

AJAX 甚至不需要用户操作即可工作。Google Docs 每隔几分钟就会通过 AJAX 自动保存您的文档，而无需您手动发起保存操作。

## 为何使用 AJAX？

显然，它能提升用户体验。AJAX 让你能够呈现动态、响应迅速且用户友好的体验，而非枯燥的静态页面。用户能即时获得操作正确与否的反馈，无需等到提交整个表单后才发现某个字段有误。重要字段可以在数据输入后立即验证，或在用户输入时提供建议。

AJAX 能显著减少前后端传输的数据量。只需交换相关数据，而非重新加载页面时传输的全部页面内容。

特别对于 WordPress 插件而言，AJAX 是目前实现独立于 WordPress 内容流程的最佳方式。如果你曾编写过 PHP 程序，可能会通过链接到新 PHP 页面来实现。用户点击链接即可触发流程。但问题在于，链接到外部 PHP 页面时无法调用任何 WordPress 函数。过去开发者通过在新 PHP 页面中包含核心文件 `wp-load.php` 来访问 WordPress 函数，但如今已无法确定该文件的正确路径。WordPress 架构现已足够灵活，`/wp-content/` 目录和插件文件可以从常规位置移动到安装根目录的下一级。你既无法确定 `wp-load.php` 相对于插件文件的位置，也无法获知安装文件夹的绝对路径。

而 AJAX 请求的发送位置是可知的，因为它定义在全局 JavaScript 变量中。你的 PHP AJAX 处理脚本实际上是一个动作钩子，因此与外部 PHP 文件不同，所有 WordPress 函数都能自动为其所用。

## 如何使用 AJAX？

如果你是 WordPress 新手，但在其他环境中使用过 AJAX，你需要重新学习一些知识。WordPress 实现 AJAX 的方式很可能与你习惯的方式不同。如果这对你来说都是全新的，也没关系。你将在这里学到基础知识。一旦你开发了一个基本的 AJAX 交互，在此基础上扩展并开发出具有出色用户界面的杀手级应用就轻而易举了！

WordPress 中的任何 AJAX 交互都有两个主要组成部分：客户端 JavaScript 或 jQuery，以及服务器端 PHP。所有 AJAX 交互都遵循以下事件序列。

1. 某种页面事件触发 JavaScript 或 jQuery 函数。该函数从页面收集一些数据，并通过 HTTP 请求将其发送到服务器。由于用 JavaScript 处理 HTTP 请求很麻烦，而且 WordPress 已经捆绑了 jQuery，因此从现在开始我们将只关注 jQuery 代码。使用纯 JavaScript 实现 AJAX 是可能的，但当 jQuery 可用时，这样做并不值得。
2. 服务器接收请求并对数据进行处理。它可能会组装相关数据，并以 HTTP 响应的形式将其发送回客户端浏览器。这不是必须的，但由于让用户了解正在发生的情况是可取的，因此不发送某种响应的情况非常罕见。
3. 发送初始 AJAX 请求的 jQuery 函数接收服务器响应并对其进行处理。它可能会更新页面上的某些内容，和/或通过某种方式向用户显示消息。

## 在 jQuery 中使用 AJAX

现在我们将定义[关于 jQuery 的文章片段](https://developer.wordpress.org/plugins/javascript/jquery/#selector-and-event)中的“执行操作”部分。我们将使用 [`$.post()`](https://api.jquery.com/jQuery.post/ "jQuery 参考") 方法，它接受 3 个参数：发送 POST 请求的 URL、要发送的数据以及处理服务器响应的回调函数。但在开始之前，我们需要做一些预先规划。我们进行以下赋值，以便稍后在回调函数中使用。其目的在[回调部分](https://developer.wordpress.org/plugins/javascript/ajax/#callback "页面章节")会更加明确。

### URL

所有 WordPress AJAX 请求都必须发送到 `wp-admin/admin-ajax.php`。正确且完整的 URL 必须通过 PHP 获取，jQuery 无法自行确定此值，并且您不能在 jQuery 代码中硬编码 URL 并期望其他人在其网站上使用您的插件。如果页面来自管理区域，WordPress 会在全局 JavaScript 变量 `ajaxurl` 中设置正确的 URL。对于公共区域的页面，您需要自行建立正确的 URL，并使用 [`wp_localize_script()`](https://developer.wordpress.org/reference/functions/wp_localize_script/) 将其传递给 jQuery。这将在 [PHP 部分](https://developer.wordpress.org/plugins/javascript/enqueuing/ "Server Side PHP and Enqueuing")中更详细地介绍。目前只需知道，适用于前端和后端的 URL 将作为您在 PHP 部分定义的全局对象的一个属性可用。在 jQuery 中，它像这样引用：

```
my_ajax_obj.ajax_url
```

### Data

All data that needs to be sent to the server is included in the data array. Besides any data needed by your app, you must send an action parameter. For requests that could result in a change to the database you need to send a nonce so the server knows the request came from a legitimate source. Our example data array provided to the `.post()` method looks like this:

```
{
  _ajax_nonce: my_ajax_obj.nonce, // nonce
  action: "my_tag_count", // action
  title: this.value // data
}
```

Each component is explained below.

### Nonce

[Nonce](https://developer.wordpress.org/apis/security/nonces/) 是 "Number used ONCE" 的合成词。它本质上是一个分配给每个表单实例的唯一序列号。Nonce 通过 PHP 脚本创建，并以与 URL 相同的方式传递给 jQuery，作为全局对象的一个属性。在此例中，它被引用为 my\_ajax\_obj.nonce。

[info]一个真正的 nonce 每次使用后都需要刷新，以便下一次 AJAX 调用有一个新的、未使用的 nonce 作为验证发送。实际上，WordPress 的 nonce 实现并非真正的 nonce。同一个 nonce 在 24 小时内可以多次使用，除非你登出。使用相同的种子短语生成 nonce 在 12 小时内总是产生相同的数字，之后才会生成新的数字。

如果你的应用需要严格的安全性，请实现一个真正的 nonce 系统，其中服务器在响应 Ajax 请求时发送一个新的 nonce，供脚本用于验证下一个请求。[/info]

最简单的方法是将此 nonce 值键设为 \_ajax\_nonce。如果与验证 nonce 的 PHP 代码协调一致，你也可以使用不同的键，但使用默认值更容易，无需担心协调问题。以下是此键值对的声明方式：

```
_ajax_nonce: my_ajax_obj.nonce
```

### 操作

所有 WordPress AJAX 请求都必须在数据中包含一个操作参数。该值是一个任意字符串，用于部分构建操作标签以挂载 AJAX 处理程序代码。建议此值能简要描述 AJAX 调用的用途。不出所料，该值的键名为 _‘action’_。在本示例中，我们将使用 `my_tag_count` 作为操作值。该键值对的声明如下所示：

```
action: "my_tag_count"
```

服务器执行任务所需的其他数据也包含在此数组中。如果需要传输大量字段，通常有两种格式可将数据字段合并为单个字符串以便传输：XML 和 JSON。使用这些格式是可选的，但无论采用何种方式，都需要与服务器端的 PHP 脚本协调配合。有关这些格式的更多信息，请参阅后续的回调部分。接收这种格式的数据比发送更常见，但两种方式均可使用。

在我们的示例中，服务器只需要一个值，即所选书名的单个字符串，因此我们将使用键名 _‘title’_。在 jQuery 中，触发事件的对象始终包含在变量 this 中。因此，所选元素的值为 this.value。该键值对的声明如下所示：

```
title: this.value
```

### 回调函数

回调处理程序是在请求发送后，服务器返回响应时执行的函数。通常我们在这里会看到一个匿名函数。该函数接收一个参数，即服务器响应。响应内容可以是简单的"是"或"否"，也可以是庞大的XML数据库。JSON格式数据也是常用的数据格式。响应甚至不是必需的——如果没有响应，则无需指定回调函数。为了用户体验考虑，最好让用户知道请求的处理结果，因此建议始终提供响应并给出操作已执行的提示。

在我们的示例中，我们用服务器响应替换了单选按钮后的当前文本，该响应包含按书名标记的帖子数量。以下是我们的匿名回调函数：

```
function( data ) {
  this2.nextSibling.remove();
  $( this2 ).after( data );
}
```

`data` 包含完整的服务器响应。之前我们通过 `var this2 = this;` 将触发变更事件的对象（引用为 `this`）赋值给 `this2`。这是因为闭包中的变量作用域仅延伸一层。通过在事件处理程序（最初仅包含 `/* do stuff */` 的部分）中赋值 `this2`，我们能够在回调函数中使用它，而在此处直接使用 `this` 会超出作用域范围。

服务器响应可以采用任何形式。大量数据应编码为数据流以便处理。XML和JSON是两种常见的编码方案。

#### XML

XML 是 AJAX 的旧式数据交换格式，毕竟 AJAX 中的 "X" 就代表它。尽管使用原生 PHP 函数处理 XML 可能较为困难，但它仍是一种可行的交换格式。正因如此，许多 PHP 程序员更偏好 JSON 交换格式。若使用 XML，解析方法取决于所用浏览器：Internet Explorer 使用 Microsoft.XMLDOM ActiveX，其他浏览器则使用 DOMParser。请注意 [WordPress 自 5.8 版本起已不再支持 Internet Explorer](https://make.wordpress.org/core/2021/04/22/ie-11-support-phase-out-plan/)。

#### JSON

JSON 因其轻量和易用性而备受青睐。实际上你可以使用 `eval()` 解析 JSON，但千万别这么做！使用 `eval()` 会带来严重的安全风险。请改用专用解析器，这样也更高效。使用全局解析器对象 `JSON` 的实例。[特定函数可提供简便方式](https://developer.wordpress.org/plugins/javascript/enqueuing/#json)，让你的 AJAX 调用获得 JSON 格式的响应。

#### Other

As long as the data format is coordinated with the PHP handler, it can be any format you like, such as comma delimited, tab delimited, or any kind of structure that works for you.

#### Client Side Summary

Now that we've added our callback as the final parameter for the $.post() function, we've completed our sample jQuery Ajax script. All the pieces put together look like this:

```
jQuery(document).ready(function($) {         //wrapper
  $(".pref").change(function() {          //event
    var this2 = this;                  //use in callback
    $.post(my_ajax_obj.ajax_url, {      //POST request
      _ajax_nonce: my_ajax_obj.nonce, //nonce
      action: "my_tag_count",         //action
      title: this.value               //data
      }, function(data) {            //callback
        this2.nextSibling.remove(); //remove current title
        $(this2).after(data);       //insert server response
      }
    );
  } );
} );
```

This script can either be output into a `block` on the web page or contained in its own file. This file can reside anywhere on the Internet, but most plugin developers place it in a `/js/` subfolder of the plugin's main folder. Unless you have reason to do otherwise, you may as well follow convention. For this example we will name our file `myjquery.js`.