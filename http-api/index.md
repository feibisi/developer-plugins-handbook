---
title: "HTTP API"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Http Api
    - Repos
    - Data
---

# HTTP API

## 简介

HTTP 代表超文本传输协议，是整个互联网的基础通信协议。即使这是您首次接触 HTTP，您可能已经理解得比您意识到的更多。在最基本的层面上，HTTP 的工作方式如下：

- “你好，服务器 XYZ，我可以获取文件 abc.html 吗？”
- “你好，小客户端，当然可以，给你。”

在 PHP 中有许多不同的方法来发送 HTTP 请求。WordPress HTTP API 的目的是支持尽可能多的这些方法，并为特定请求选择最合适的一种。

WordPress HTTP API 还可用于与其他 API（如 Twitter API 或 Google Maps API）进行通信和交互。

## HTTP 方法

HTTP 拥有多种方法（或称动词）来描述特定类型的操作。虽然存在更多方法，但 WordPress 已为三种最常用的方法预置了函数。每当发起 HTTP 请求时，都会附带一个方法，以帮助服务器判断客户端请求的是何种操作。

### GET

GET 用于获取数据。这是目前最常用的动词。每次你浏览网站或从 API 拉取数据时，你看到的都是 GET 请求的结果。实际上，你的浏览器向承载本文的服务器发送了一个 GET 请求，并请求了用于构建本文的数据。

### POST

POST 用于向服务器发送数据，以便服务器以某种方式处理。例如，联系表单。当您在表单字段中输入数据并点击提交按钮时，浏览器会获取数据，并将您输入表单的文本作为 POST 请求发送到服务器。随后，服务器将处理该联系请求。

### HEAD

HEAD 远不如另外两种方法知名。HEAD 本质上与 GET 请求相同，只是它不获取数据本身，仅获取数据的相关信息。这些信息描述了诸如数据最后更新时间、客户端是否应缓存数据、数据类型等。现代浏览器常向您先前访问过的页面发送 HEAD 请求，以判断是否有更新。如果没有，您实际看到的可能是先前下载的页面副本，从而避免不必要地消耗带宽重复获取相同内容。

所有优秀的 API 客户端都会在执行 GET 请求前先使用 HEAD，以节省带宽。虽然如果 HEAD 指示有新数据时需要进行两次独立的 HTTP 请求，但 GET 请求的数据量可能非常庞大。仅在 HEAD 指示数据是新的或不应缓存时才使用 GET，有助于节省宝贵的带宽并减少加载时间。

### 自定义方法

还存在其他 HTTP 方法，例如 PUT、DELETE、TRACE 和 CONNECT。本文不会涵盖这些方法，因为 WordPress 中没有预置的方法来使用它们，且 API 实现这些方法也尚不普遍。

根据服务器配置方式，您还可以实现自己的额外 HTTP 方法。使用非标准方法总是存在风险，并且会对其他开发者创建客户端来使用您的网站或 API 带来巨大限制，但在 WordPress 中确实可以使用任何您希望的方法。本文将简要介绍如何实现这一点。

## 响应码

HTTP 使用数字和字符串响应码。与其逐一详细解释，这里列出标准响应码。创建 API 时可以自定义响应码，但除非需要支持特定响应类型，否则最好遵循标准码。自定义码通常位于 1xx 范围。

### Code Classes

The type of response can quickly be seen by the leftmost digit of the three digit codes.

| Status Code | Description |
| --- | --- |
| 2xx | Request was successful |
| 3xx | Request was redirected to another URL |
| 4xx | Request failed due to client error. Usually invalid authentication or missing data |
| 5xx | Request failed due to a server error. Commonly missing or misconfigured configuration files |

**Common Codes**

These are the most common codes you will encounter.

| Status Code | Description |
| --- | --- |
| 200 | OK – Request was successful |
| 301 | Resource was moved permanently |
| 302 | Resource was moved temporarily |
| 403 | Forbidden – Usually due to an invalid authentication |
| 404 | Resource not found |
| 500 | Internal server error |
| 503 | Service unavailable |

## 从 API 获取数据

[GitHub](https://github.com/) 提供了一个出色的 API，其许多公开功能无需注册应用即可使用。因此，为了演示其中一些方法，示例将针对 GitHub API。

在 WordPress 中，通过 [`wp_remote_get()`](https://developer.wordpress.org/reference/functions/wp_remote_get/) 函数可以极其简单地获取数据。该函数接受以下两个参数：

1. `$url` – 要从中检索数据的资源。必须是标准的 HTTP 格式。
2. `$args` – 可选 – 您可以在此传递一个参数数组来更改行为和请求头，例如 cookies、跟随重定向等。

以下为默认设置，但可以通过 `$args` 参数进行更改：

- `method` – GET
- `timeout` – 5 – 放弃请求前的等待时间
- `redirection` – 5 – 跟随重定向的次数
- `httpversion` – 1.0
- `blocking` – true – 页面其余部分是否应等待此操作完成后再继续加载？
- `headers` – array()
- `body` – null
- `cookies` – array()

让我们使用一个 GitHub 用户账户的 URL，看看能获取到什么样的信息。

```
$response = wp_remote_get( 'https://api.github.com/users/blobaugh' );
```

`$response` 将包含我们请求的所有响应头、内容和其他元数据。

```
Array(
  [headers] => Array(
    [server] => nginx
    [date] => Fri, 05 Oct 2012 04:43:50 GMT
    [content-type] => application/json; charset=utf-8
    [connection] => close
    [status] => 200 OK
    [vary] => Accept
    [x-ratelimit-remaining] => 4988
    [content-length] => 594
    [last-modified] => Fri, 05 Oct 2012 04:39:58 GMT
    [etag] => "5d5e6f7a09462d6a2b473fb616a26d2a"
    [x-github-media-type] => github.beta
    [cache-control] => public, s-maxage=60, max-age=60
    [x-content-type-options] => nosniff
    [x-ratelimit-limit] => 5000
  )
  [body] => {"type":"User","login":"blobaugh","gravatar_id":"f25f324a47a1efdf7a745e0b2e3c878f","public_gists":1,"followers":22,"created_at":"2011-05-23T21:38:50Z","public_repos":31,"email":"ben@lobaugh.net","hireable":true,"blog":"http://ben.lobaugh.net","bio":null,"following":30,"name":"Ben Lobaugh","company":null,"avatar_url":"https://secure.gravatar.com/avatar/f25f324a47a1efdf7a745e0b2e3c878f?d=https://a248.e.akamai.net/assets.github.com%2Fimages%2Fgravatars%2Fgravatar-user-420.png","id":806179,"html_url":"https://github.com/blobaugh","location":null,"url":"https://api.github.com/users/blobaugh"}
  [response] => Array(
    [preserved_text 5237511b45884ac6db1ff9d7e407f225 /] => 200
    [message] => OK
  )
  [cookies] => Array()
  [filename] =>
)
```

与此函数配合使用的所有辅助函数与前两个函数相同。唯一的例外是 HEAD 请求从不返回响应体，因此该元素将始终为空。

### 获取你梦寐以求的响应体

仅使用 [`wp_remote_retrieve_body()`](https://developer.wordpress.org/reference/functions/wp_remote_retrieve_body/) 即可获取响应体。此函数仅接受一个参数，即来自任何其他 [`wp_remote_X`](https://developer.wordpress.org/?s=wp_remote_&post_type%5B%5D=wp-parser-function) 函数的响应，其中 `retrieve` 不是紧随其后的值。

```
$response = wp_remote_get( 'https://api.github.com/users/blobaugh' );
$body     = wp_remote_retrieve_body( $response );
```

继续使用前面示例中的 GitHub 资源，`$body` 将是：

```
{"type":"User","login":"blobaugh","public_repos":31,"gravatar_id":"f25f324a47a1efdf7a745e0b2e3c878f","followers":22,"avatar_url":"https://secure.gravatar.com/avatar/f25f324a47a1efdf7a745e0b2e3c878f?d=https://a248.e.akamai.net/assets.github.com%2Fimages%2Fgravatars%2Fgravatar-user-420.png","public_gists":1,"created_at":"2011-05-23T21:38:50Z","email":"ben@lobaugh.net","following":30,"name":"Ben Lobaugh","company":null,"hireable":true,"id":806179,"html_url":"https://github.com/blobaugh","blog":"http://ben.lobaugh.net","location":null,"bio":null,"url":"https://api.github.com/users/blobaugh"}
```

如果除了获取响应体之外，你不需要对响应执行任何其他操作，则可以将代码简化为一行：

```
$body = wp_remote_retrieve_body( wp_remote_get( 'https://api.github.com/users/blobaugh' ) );
```

许多类似的辅助函数都可以这样在一行中使用。

### 获取响应代码

您可能需要检查响应代码以确保检索成功。这可以通过 [`wp_remote_retrieve_response_code()`](https://developer.wordpress.org/reference/functions/wp_remote_retrieve_response_code/) 函数完成：

```
$response = wp_remote_get( 'https://api.github.com/users/blobaugh' );
$http_code = wp_remote_retrieve_response_code( $response );
```

如果成功，`$http_code` 将包含 `200`。

### 获取特定响应头

若您希望获取特定响应头，例如 last-modified，可以使用 [`wp_remote_retrieve_header()`](https://developer.wordpress.org/reference/functions/wp_remote_retrieve_header) 函数。此函数接受两个参数。

1. `$response` – 来自 GET 调用的响应。
2. `$header` – 要获取的响应头名称。

获取 last-modified 响应头的示例：

```
$response      = wp_remote_get( 'https://api.github.com/users/blobaugh' );
$last_modified = wp_remote_retrieve_header( $response, 'last-modified' );
```

`$last_modified` 将包含 `[last-modified] => Fri, 05 Oct 2012 04:39:58 GMT`

您也可以使用 `wp_remote_retrieve_headers( $response )` 以数组形式获取所有响应头。

### 使用基本认证的 GET 请求

受保护的 API 通常会提供多种不同类型的认证方式。HTTP 基本认证是一种常见但安全性不高的认证方法。在 WordPress 中，可以通过将 'Authorization' 传递给 [`wp_remote_get()`](https://developer.wordpress.org/reference/functions/wp_remote_get/) 函数的第二个参数（以及其他 HTTP 方法函数）来使用它。

```
$args = array(
  'headers' => array(
    'Authorization' => 'Basic ' . base64_encode( YOUR_USERNAME . ':' . YOUR_PASSWORD )
  )
);
wp_remote_get( $url, $args );
```

## 向 API 发送 POST 数据

所有 HTTP 方法调用都使用相同的辅助方法（[`wp_remote_retrieve_body()`](https://developer.wordpress.org/reference/functions/wp_remote_retrieve_body/) 等），并且使用方式也完全相同。

发送 POST 数据使用 [`wp_remote_post()`](https://developer.wordpress.org/reference/functions/wp_remote_post/) 函数，其参数与 [`wp_remote_get()`](https://developer.wordpress.org/reference/functions/wp_remote_get/) 完全一致。这里需要注意的是，您必须为第二个参数传入数组中的所有元素。Codex 提供了默认的可接受值。目前您只需关注要发送的数据，其他值将使用默认设置。

要向服务器发送数据，您需要构建一个关联数组。该数据将被赋值给 `'body'` 键。从服务器端来看，该值将如预期那样出现在 `$_POST` 变量中。例如，如果 `body => array( 'myvar' => 5 )`，那么在服务器端 `$_POST['myvar'] = 5`。

由于 GitHub 不允许向之前示例中使用的 API 发送 POST 请求，本示例将假设它可以。通常，如果您想向 API 发送 POST 数据，您需要联系 API 的维护者并获取 API 密钥或其他形式的身份验证令牌。这仅用于证明您的应用程序有权操作 API 上的数据，就像用户登录网站后对网站的操作一样。

假设我们正在提交一个包含以下字段的联系表单：姓名、电子邮件、主题、评论。要设置请求体，我们这样做：

```
$body = array(
  'name'    => 'Jane Smith',
  'email'   => 'some@email.com',
  'subject' => 'Checkout this API stuff',
  'comment' => 'I just read a great tutorial. You gotta check it out!',
);
```

现在我们需要设置将传递给 [`wp_remote_post()`](https://developer.wordpress.org/reference/functions/wp_remote_post/) 第二个参数的其他值。

```
$args = array(
  'body'        => $body,
  'timeout'     => '5',
  'redirection' => '5',
  'httpversion' => '1.0',
  'blocking'    => true,
  'headers'     => array(),
  'cookies'     => array(),
);
```

然后当然就是进行调用。

```
$response = wp_remote_post( 'https://example.com', $args );
```

## 避免带宽消耗

在获取资源前使用 HEAD 方法检查其状态非常重要，有时甚至是 API 的要求。对于高流量 API，GET 请求通常被限制为每分钟或每小时一定次数。除非 HEAD 请求显示 API 数据已更新，否则甚至无需尝试 GET 请求。

如前所述，HEAD 响应包含数据是否已更新、是否应缓存数据、缓存副本何时过期等信息，有时还会包含 API 请求的速率限制。

回到 GitHub 示例，以下是一些需要关注的响应头。大多数这些头部是标准化的，但您应始终查阅 API 文档以确保理解各头部的名称和用途。

- `x-ratelimit-limit` – 时间段内允许的请求数。
- `x-ratelimit-remaining` – 时间段内剩余的可用请求数。
- `content-length` – 内容大小（字节）。若内容较大，可用于提醒用户。
- `last-modified` – 资源最后修改时间。对缓存工具极为有用。
- `cache-control` – 客户端应如何处理缓存。

以下代码将检查我的 GitHub 用户账户的 HEAD 值：

```
$response = wp_remote_head( 'https://api.github.com/users/blobaugh' );
```

`$response` 应类似于：

```
Array(
  [headers] => Array(
    [server] => nginx
    [date] => Fri, 05 Oct 2012 05:21:26 GMT
    [content-type] => application/json; charset=utf-8
    [connection] => close
    [status] => 200 OK
    [vary] => Accept
    [x-ratelimit-remaining] => 4982
    [content-length] => 594
    [last-modified] => Fri, 05 Oct 2012 04:39:58 GMT
    [etag] => "5d5e6f7a09462d6a2b473fb616a26d2a"
    [x-github-media-type] => github.beta
    [cache-control] => public, s-maxage=60, max-age=60
    [x-content-type-options] => nosniff
    [x-ratelimit-limit] => 5000
  )
  [body] =>
  [response] => Array(
    [preserved_text 39a8515bd2dce2aa06ee8a2a6656b1de /] => 200
    [message] => OK
  )
  [cookies] => Array(
  )
  [filename] =>
)
```

与此前两种方法相同，所有辅助函数均可用于此函数。例外在于 HEAD 请求从不返回响应体，因此该元素将始终为空。

## 发起任意类型的请求

如果你需要使用上述函数不支持的 HTTP 方法发起请求，请不要慌张。优秀的 WordPress 开发者早已考虑到这一点，并贴心地提供了 [`wp_remote_request()`](https://developer.wordpress.org/reference/functions/wp_remote_request/)。此函数与 [`wp_remote_get()`](https://developer.wordpress.org/reference/functions/wp_remote_get/) 接收相同的两个参数，并允许你指定 HTTP 方法。需要传递哪些数据则取决于你的方法。

要发送 DELETE 方法请求，可以参考以下示例：

```
$args     = array(
  'method' => 'DELETE',
);
$response = wp_remote_request( 'http://some-api.com/object/to/delete', $args );
```

## 缓存简介

缓存是一种将常用对象或构建耗时较长的对象保存到快速对象存储中，以便后续请求快速检索的做法。这避免了再次花费时间获取和构建对象。缓存是一个广泛的主题，属于网站优化的一部分，本身就可以写成一个完整的系列文章。以下只是对缓存的介绍，以及一种简单而有效的方法来快速设置 API 响应缓存。

为什么要缓存 API 响应？嗯，房间里的大象是因为外部 API 会拖慢你的网站。许多顾问会告诉你，接入外部 API 可以通过减少连接和处理量以及昂贵的带宽来提高网站性能，但有时这并不正确。

这需要在服务器发送数据的速度与远程服务器处理请求、构建数据并返回所需时间之间进行微妙的平衡。第二个明显的问题是，许多 API 在特定时间段内的请求数量有限，并且可能限制应用程序同时的连接数。缓存通过将数据副本存储在服务器上直到需要刷新，有助于解决这些难题。

## 何时应该缓存？

直觉的回答是**总是缓存**，但同样存在不应缓存的情况。如果您处理的是实时数据，或者 API 在响应头中明确声明不应缓存，那么您可能不希望缓存；但在所有其他情况下，缓存从 API 获取的资源通常是个好主意。

## WordPress 瞬态数据

WordPress 瞬态数据提供了一种便捷的方式来存储和使用缓存对象。瞬态数据会存活指定的时间，或在 API 资源更新后按需过期。使用 WordPress 的瞬态功能可能是您遇到过的最简单的缓存系统，仅需三个函数即可完成所有繁重工作。

### 缓存对象（设置瞬态数据）

使用 [`set_transient()`](https://developer.wordpress.org/reference/functions/set_transient/) 函数可以缓存对象。该函数接受以下三个参数：

1. `$transient` – 用于后续引用的瞬态数据名称。
2. `$value` – 瞬态数据的值。
3. `$expiration` – 从保存瞬态数据到其过期的时间（以秒为单位）。

例如，将上述 GitHub 用户信息响应缓存一小时：

```
$response = wp_remote_get( 'https://api.github.com/users/blobaugh' );
set_transient( 'prefix_github_userinfo', $response, 60 * 60 );
```

### Get a cached object ( Get a transient )

Getting a cached object is quite a bit more complex than setting a transient. You need to request the transient, but then you also need to check to see if that transient has expired and if so fetch updated data. Usually the `set_transient()` call is made inside of the `get_transient()` call. Here is an example of getting the transient data for the GitHub user profile:

```
$github_userinfo = get_transient( 'prefix_github_userinfo' );
if ( false === $github_userinfo ) {
  // Transient expired, refresh the data
  $response = wp_remote_get( 'https://api.github.com/users/blobaugh' );
  set_transient( 'prefix_github_userinfo', $response, HOUR_IN_SECONDS );
}
// Use $github_userinfo as you will
```

## 删除缓存对象（删除瞬态数据）

删除缓存对象是所有瞬态函数中最简单的操作，只需传入瞬态名称参数即可完成。

要删除 Github 用户信息：

```
delete_transient( 'blobaugh_github_userinfo' );
```

更多关于瞬态数据的信息可查阅[此文档](https://developer.wordpress.org/apis/transients/)。