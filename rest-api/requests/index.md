---
title: "请求"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Requests
    - Rest Api
    - Repos
---

# 请求

## 概述

REST API 在许多方面都非常简单。存在输入，即请求。服务器解释输入并生成输出，即响应。在某些方面，您可以将对 WordPress REST API 的请求视为一组应由 API 执行和解释的指示或指令。默认情况下，WordPress REST API 旨在使用 HTTP 请求作为其请求媒介。HTTP 是互联网上数据通信的基础，这使得 WordPress REST API 成为一个影响深远的 API。API 中的请求利用了 HTTP 请求中的许多不同方面，如 URI、HTTP 方法、标头和参数。请求的数据结构由 `WP_REST_Request` 类方便地处理。

## WP_REST_Request

该类是 WordPress 4.4 引入的三个核心基础设施类之一。当向 API 端点发起 HTTP 请求时，API 会自动创建 `WP_REST_Request` 类的实例，并与提供的数据进行匹配。响应对象在 `WP_REST_Server` 的 `serve_request()` 方法中自动生成。请求创建并完成身份验证后，请求被分派，我们注册的端点回调函数开始执行。存储在 `WP_REST_Request` 对象中的所有数据都会传递给我们注册端点的回调函数。因此，`permissions_callback` 和 `callback` 都会接收到传入的请求对象。这使得我们能够在回调函数中访问各种请求属性，从而根据期望的输出定制响应内容。

## 请求属性

请求对象具有许多不同的属性，每个属性都可以以各种方式使用。主要属性包括请求方法、路由、标头、参数和属性。让我们逐一分析这些属性在请求中的作用。如果您要自己创建请求对象，它将如下所示：

```
$request = new WP_REST_Request( 'GET', '/my-namespace/v1/examples' );
```

在上面的代码示例中，我们仅指定请求对象方法为 `GET`，并且应匹配路由 `/my-namespace/v1/examples`，在整个 URL 上下文中将如下所示：`https://ourawesomesite.com/wp-json/my-namepsace/v1/examples`。`WP_REST_Request` 构造函数的 method 和 route 参数用于将请求映射到所需的端点。如果请求发送到未注册的端点，则响应中将返回有用的 404 错误消息。让我们更深入地了解各种属性。

### 方法

请求对象的 method 属性默认与 HTTP 请求方法匹配。在大多数情况下，方法将是 `GET`、`POST`、`PUT`、`DELETE`、`OPTIONS` 或 `HEAD` 之一。这些方法将用于匹配注册到路由的各个端点。当 API 找到方法与路由的匹配项时，它将触发该端点的回调函数。

以下约定是匹配 HTTP 方法的最佳实践：`GET` 用于只读任务，`POST` 用于创建，`PUT` 用于更新，`DELETE` 用于删除。请求方法充当端点预期功能的指示器。当您向路由发出 `GET` 请求时，应期望返回只读数据。

### 路由

请求的路由默认会匹配服务器环境变量中的路径信息：`$_SERVER['PATH_INFO']`。当你向 WordPress REST API 的某个路由发起 HTTP 请求时，生成的 `WP_REST_Request` 对象将与该路径匹配，并期望最终匹配到有效的端点。简而言之，请求的路由就是你在 API 中希望定位的目标位置。

如果我们使用 `GET` 方法注册了一个书籍端点，它可能位于 `https://ourawesomesite.com/wp-json/my-namespace/v1/books`。在浏览器中访问该 URL，我们将看到以 JSON 格式呈现的书籍集合。WordPress 会自动为我们生成请求对象，并处理所有路由匹配端点的逻辑。因此，既然我们无需自行处理路由，理解如何在请求中传递额外数据就成了更重要的学习内容。

### 请求头

HTTP 请求头本质上是关于我们 HTTP 请求的附加数据。请求头可以指定缓存策略、请求内容、请求来源等诸多信息。请求头不一定直接与我们的端点交互，但其中的信息能帮助 WordPress 确定如何处理请求。若需传递端点需要交互的数据，我们应当使用参数。

### 参数

向 WordPress REST API 发起请求时，传递的大部分附加数据都将以参数形式呈现。什么是参数？在 API 上下文中存在四种不同类型：路由参数、查询参数、正文参数和文件参数。让我们逐一深入探讨。

#### URL Params

URL parameters are automatically generated in a `WP_REST_Request` from the path variables in the requested route. What does that mean? Let’s look at this route, which grabs individual books by id: `/my-namespace/v1/books/(?P\d+)`. The odd looking `(?P\d+)` is a path variable. The name of the path variable is ‘`id`‘.

If we were to make a request like `GET https://ourawesomesite.com/wp-json/my-namespace/v1/books/5`, `5` will become the value for our `id` path variable. The `WP_REST_Request` object will automatically take that path variable and store it as a URL parameter. Now inside of our endpoint callbacks we can interact with that URL parameter really easily. Let’s look at an example.  

```
// Register our individual books endpoint.
function prefix_register_book_route() {
    register_rest_route( 'my-namespace/v1', '/books/(?P<id>\d+)', array(
        // Supported methods for this endpoint. WP_REST_Server::READABLE translates to GET.
        'methods' => WP_REST_Server::READABLE,
        // Register the callback for the endpoint.
        'callback' => 'prefix_get_book',
    ) );
}

add_action( 'rest_api_init', 'prefix_register_book_route' ); 

/**
 * Our registered endpoint callback. Notice how we are passing in $request as an argument.
 * By default, the WP_REST_Server will pass in the matched request object to our callback.
 *
 * @param WP_REST_Request $request The current matched request object.
 */
function prefix_get_book( $request ) {
    // Here we are accessing the path variable 'id' from the $request.
    $book = prefix_get_the_book( $request['id'] );
    return rest_ensure_response( $book );
}

// A simple function that grabs a book title from our books by ID.
function prefix_get_the_book( $id ) {
    $books = array(
        'Design Patterns',
        'Clean Code',
        'Refactoring',
        'Structure and Interpretation of Computer Programs',
    );

    $book = '';
    if ( isset( $books[ $id ] ) ) {
        // Grab the matching book.
        $book = $books[ $id ];
    } else {
        // Error handling.
        return new WP_Error( 'rest_not_found', esc_html__( 'The book does not exist', 'my-text-domain' ), array( 'status' => 404 ) );
    }

    return $book;
}
```

In the example above we see how path variables are stored as URL parameters in the request object. We can then access those parameters in our endpoint callbacks. The above example is a pretty common use case for using URL params. Adding too many path variables to a route can slow down the matching of routes and it can also over complicate registering endpoints, it is advised to use URL parameters sparingly. If we aren’t supposed to use parameters directly in our URL path, then we need another way to pass in extra information to our request. This is where query and body parameters come in, they will typically do most of the heavy lifting in your API.

#### 查询参数

查询参数存在于 URI 的查询字符串部分。在 `https://ourawesomesite.com/wp-json/my-namespace/v1/books?per_page=2&genre=fiction` 中，URI 的查询字符串部分是 `?per_page=2&genre=fiction`。查询字符串以 '`?`' 字符开始，查询字符串中的不同值由 '`&`' 字符分隔。我们在查询字符串中指定了两个参数：`per_page` 和 `fiction`。在我们的端点中，我们可能只想获取属于小说类别的两本书。我们可以在回调函数中这样访问这些值：`$request['per_page']` 和 `$request['genre']`（假设 `$request` 是我们使用的参数名称）。如果你熟悉 PHP，你可能已经在你的 Web 应用程序中使用过查询参数。

在 PHP 中，查询参数存储在超全局变量 `$_GET` 中。需要注意的是，你永远不应该直接在端点中访问任何超全局变量或服务器变量。最好始终使用 `WP_REST_Request` 类提供的内容。另一种向端点传递变量的常见方法是使用正文参数。

#### 请求体参数

请求体参数是以键值对形式存储在请求体中的数据。如果你曾通过 HTML 表单、cURL 或其他方式发送过 `POST` 请求，那么你就已经使用过请求体参数。请求体参数可以通过不同的内容类型进行传递。`POST` 请求默认的 `Content-Type` 请求头是 `x-www-form-urlencoded`。当使用 `x-www-form-urlencoded` 时，参数会以查询字符串的形式发送，例如 `per_page=2&genre=fiction`。默认情况下，HTML 表单会将各种输入字段打包，并发送符合 `x-www-form-urlencoded` 模式的 `POST` 请求。

需要注意的是，虽然 HTTP 规范并未禁止在 `GET` 请求中使用请求体参数，但建议不要在 `GET` 请求中使用。请求体参数可以且应该用于 `POST`、`PUT` 和 `DELETE` 请求。

#### 文件参数

当请求使用特殊内容类型标头 `multipart/form-data` 时，`WP_REST_Request` 对象中的文件参数会被存储。随后可通过 `$request->get_file_params()` 从请求对象访问文件数据。文件参数等同于 PHP 超全局变量：`$_FILES`。请记住，不要直接访问超全局变量，仅使用 `WP_REST_Request` 对象提供的方法。

在端点回调中，我们可以使用 `wp_handle_upload()` 将所需文件添加到 WordPress 的媒体上传目录。文件参数仅适用于处理文件数据，切勿将其用于任何其他目的。

### 属性

`WP_REST_Request` 同样支持请求属性。请求属性是指注册到匹配路由的属性。如果我们向 `my-namespace/v1/books` 发起 `GET` 请求，然后在端点回调中调用 `$request->get_attributes()`，我们将获得 `my-namespace/v1/books` 端点的所有注册选项。如果我们向同一路由发起 `POST` 请求，且端点回调同样返回 `$request->get_attributes()`，则会收到注册到 `POST` 端点回调的另一组端点选项。

在属性中，我们将获得包含以下内容的响应：支持的方法、选项、是否在索引中显示此端点、端点已注册参数列表以及已注册的回调函数。其返回内容可能如下所示：

```
{
  "methods": {
    "GET": true
  },
  "accept_json": false,
  "accept_raw": false,
  "show_in_index": true,
  "args": {
    "context": {
      "description": "Scope under which the request is made; determines fields present in response.",
      "type": "string",
      "sanitize_callback": "sanitize_key",
      "validate_callback": "rest_validate_request_arg",
      "enum": [
        "view",
        "embed",
        "edit"
      ],
      "default": "view"
    },
    "page": {
      "description": "Current page of the collection.",
      "type": "integer",
      "default": 1,
      "sanitize_callback": "absint",
      "validate_callback": "rest_validate_request_arg",
      "minimum": 1
    },
    "per_page": {
      "description": "Maximum number of items to be returned in result set.",
      "type": "integer",
      "default": 10,
      "minimum": 1,
      "maximum": 100,
      "sanitize_callback": "absint",
      "validate_callback": "rest_validate_request_arg"
    },
    "search": {
      "description": "Limit results to those matching a string.",
      "type": "string",
      "sanitize_callback": "sanitize_text_field",
      "validate_callback": "rest_validate_request_arg"
    },
    "after": {
      "description": "Limit response to resources published after a given ISO8601 compliant date.",
      "type": "string",
      "format": "date-time",
      "validate_callback": "rest_validate_request_arg"
    },
    "author": {
      "description": "Limit result set to posts assigned to specific authors.",
      "type": "array",
      "default": [],
      "sanitize_callback": "wp_parse_id_list",
      "validate_callback": "rest_validate_request_arg"
    },
    "author_exclude": {
      "description": "Ensure result set excludes posts assigned to specific authors.",
      "type": "array",
      "default": [],
      "sanitize_callback": "wp_parse_id_list",
      "validate_callback": "rest_validate_request_arg"
    },
    "before": {
      "description": "Limit response to resources published before a given ISO8601 compliant date.",
      "type": "string",
      "format": "date-time",
      "validate_callback": "rest_validate_request_arg"
    },
    "exclude": {
      "description": "Ensure result set excludes specific ids.",
      "type": "array",
      "default": [],
      "sanitize_callback": "wp_parse_id_list"
    },
    "include": {
      "description": "Limit result set to specific ids.",
      "type": "array",
      "default": [],
      "sanitize_callback": "wp_parse_id_list"
    },
    "offset": {
      "description": "Offset the result set by a specific number of items.",
      "type": "integer",
      "sanitize_callback": "absint",
      "validate_callback": "rest_validate_request_arg"
    },
    "order": {
      "description": "Order sort attribute ascending or descending.",
      "type": "string",
      "default": "desc",
      "enum": [
        "asc",
        "desc"
      ],
      "validate_callback": "rest_validate_request_arg"
    },
    "orderby": {
      "description": "Sort collection by object attribute.",
      "type": "string",
      "default": "date",
      "enum": [
        "date",
        "relevance",
        "id",
        "include",
        "title",
        "slug"
      ],
      "validate_callback": "rest_validate_request_arg"
    },
    "slug": {
      "description": "Limit result set to posts with a specific slug.",
      "type": "string",
      "validate_callback": "rest_validate_request_arg"
    },
    "status": {
      "default": "publish",
      "description": "Limit result set to posts assigned a specific status; can be comma-delimited list of status types.",
      "enum": [
        "publish",
        "future",
        "draft",
        "pending",
        "private",
        "trash",
        "auto-draft",
        "inherit",
        "any"
      ],
      "sanitize_callback": "sanitize_key",
      "type": "string",
      "validate_callback": [
        {},
        "validate_user_can_query_private_statuses"
      ]
    },
    "filter": {
      "description": "Use WP Query arguments to modify the response; private query vars require appropriate authorization."
    },
    "categories": {
      "description": "Limit result set to all items that have the specified term assigned in the categories taxonomy.",
      "type": "array",
      "sanitize_callback": "wp_parse_id_list",
      "default": []
    },
    "tags": {
      "description": "Limit result set to all items that have the specified term assigned in the tags taxonomy.",
      "type": "array",
      "sanitize_callback": "wp_parse_id_list",
      "default": []
    }
  },
  "callback": [
    {},
    "get_items"
  ],
  "permission_callback": [
    {},
    "get_items_permissions_check"
  ]
}
```

如您所见，我们已将所有注册到端点的信息准备就绪，随时可用！请求属性通常在较低层级使用，由 `WP_REST_Server` 类处理，但在端点回调内部仍可进行一些有趣的操作，例如将接受的参数限制为与注册参数匹配。

WP REST API 的设计旨在让您无需处理任何内部细节，因此这些与 `WP_REST_Request` 交互的高级方法并不常用。使用 WP REST API 的核心在于注册路由和端点。请求是我们用来告知 API 要访问哪个端点的工具，这通常通过 HTTP 完成，但我们也可以在内部使用 `WP_REST_Request`。

## Internal Requests

The key to making internal requests is using `rest_do_request()`. All you need to do is pass in a request object and you will be returned a response. Because the request is never served by the `WP_REST_Server`, the response data is never encoded into json, meaning we have our response object as a PHP object. This is pretty awesome and enables us to do a lot of interesting things. For one, we can create efficient batch endpoints. From a performance perspective, one of the hurdles is minimizing HTTP requests. We can create batch endpoints that will use `rest_do_request()` to serve all of our requests internally all in one HTTP request. Here is a very simplistic batch endpoint for read only data, so you can see `rest_do_request()` in action.

```
// Register our mock batch endpoint.
function prefix_register_batch_route() {
    register_rest_route( 'my-namespace/v1', '/batch', array(
        // Supported methods for this endpoint. WP_REST_Server::READABLE translates to GET.
        'methods' => WP_REST_Server::READABLE,
        // Register the callback for the endpoint.
        'callback' => 'prefix_do_batch_request',
        // Register args for the batch endpoint.
        'args' => prefix_batch_request_parameters(),
    ) );
}

add_action( 'rest_api_init', 'prefix_register_batch_route' );

/**
 * Our registered endpoint callback. Notice how we are passing in $request as an argument.
 * By default, the WP_REST_Server will pass in the matched request object to our callback.
 *
 * @param WP_REST_Request $request The current matched request object.
 */
function prefix_do_batch_request( $request ) {
    // Here we initialize the array that will hold our response data.
    $data = array();
	$data = prefix_handle_batch_requests( $request['requests'] );
	return $data;
}

/**
 * This handles the building of the response for the batch requests we make.
 *
 * @param array $requests An array of data to build WP_REST_Request objects from.
 * @return WP_REST_Response A collection of response data for batch endpoints.
 */
function prefix_handle_batch_requests( $requests ) {
	$data = array();

	// Foreach request specified in the requests param run the endpoint.
	foreach ( $requests as $request_params ) {
		$response = prefix_handle_request( $request_params );
		$key = $request_params['method'] . ' ' . $request_params['route'];
		$data[ $key ] = prefix_prepare_for_collection( $response );
	}

	return rest_ensure_response( $data );
}

/**
 * This handles the building of the response for the batch requests we make.
 *
 * @param array $request_params Data to build a WP_REST_Request object from.
 * @return WP_REST_Response Response data for the request.
 */
function prefix_handle_request( $request_params ) {
	$request = new WP_REST_Request( $request_params['method'], $request_params['route'] );

// Add specified request parameters into the request.
	if ( isset( $request_params['params'] ) ) {
		foreach ( $request_params['params'] as $param_name => $param_value ) {
			$request->set_param( $param_name, $param_value );
		}
	}
	$response = rest_do_request( $request );
	return $response;
}

/**
 * Prepare a response for inserting into a collection of responses.
 *
 * This is lifted from WP_REST_Controller class in the WP REST API v2 plugin.
 *
 * @param WP_REST_Response $response Response object.
 * @return array Response data, ready for insertion into collection data.
 */
function prefix_prepare_for_collection( $response ) {
	if ( ! ( $response instanceof WP_REST_Response ) ) {
		return $response;
	}

	$data = (array) $response->get_data();
	$server = rest_get_server();

	if ( method_exists( $server, 'get_compact_response_links' ) ) {
		$links = call_user_func( array( $server, 'get_compact_response_links' ), $response );
	} else {
		$links = call_user_func( array( $server, 'get_response_links' ), $response );
	}

	if ( ! empty( $links ) ) {
		$data['_links'] = $links;
	}

	return $data;
}

/**
 * Returns the JSON schema data for our registered parameters.
 *
 * @return array $params A PHP representation of JSON Schema data.
 */
function prefix_batch_request_parameters() {
    $params = array();

    $params['requests'] = array(
		'description'        => esc_html__( 'An array of request objects arguments that can be built into WP_REST_Request instances.', 'my-text-domain' ),
		'type'               => 'array',
		'required'           => true,
		'validate_callback'  => 'prefix_validate_requests',
		'items'              => array(
			array(
				'type' => 'object',
				'properties' => array(
					'method' => array(
						'description' => esc_html__( 'HTTP Method of the desired request.', 'my-text-domain' ),
						'type'        => 'string',
						'required'    => true,
						'enum'        => array(
							'GET',
							'POST',
							'PUT',
							'DELETE',
							'OPTIONS',
						),
					),
					'route' => array(
						'description' => esc_html__( 'Desired route for the request.', 'my-text-domain' ),
						'required'    => true,
						'type'        => 'string',
						'format'      => 'uri',
					),
					'params' => array(
						'description' => esc_html__( 'Key value pairs of desired request parameters.', 'my-text-domain' ),
						'type' => 'object',
					),
				),
			),
		),
	);

	return $params;
}

function prefix_validate_requests( $requests, $request, $param_key ) {
	// If requests isn't an array of requests then we don't process the batch.
	if ( ! is_array( $requests ) ) {
		return new WP_Error( 'rest_invald_param', esc_html__( 'The requests parameter must be an array of requests.' ), array( 'status' => 400 ) );
	}

foreach ( $requests as $request ) {
		// If the method or route is not set then we do not run the requests.
		if ( ! isset( $request['method'] ) || ! isset( $request['route'] ) ) {
			return new WP_Error( 'rest_invald_param', esc_html__( 'You must specify the method and route for each request.' ), array( 'status' => 400 ) );
		}

		if ( isset( $request['params'] ) && ! is_array( $request['params'] ) ) {
			return new WP_Error( 'rest_invald_param', esc_html__( 'You must specify the params for each request as an array of named key value pairs.' ), array( 'status' => 400 ) );
		}
	}

	// This is a black listing approach to data validation.
	return true;
}
```

That is quite a decent chunk of code that covers a number of topics, but everything centers around what happens in `prefix_handle_request()`. Here we are passing in an array that tells us a HTTP method, a route, and a set of parameters we want to turn into a request. We then build the request object for the method and route. If any parameters were specified we use the `WP_REST_Request::set_param()` method to add in the desired parameters. Once our `WP_REST_Request` is ready to go we use `rest_do_request` to internally match that endpoint and the response is returned to our batch endpoint response collection. Using a batch endpoint like this can net you huge performance gains, as you will only make one HTTP request to get a response for multiple endpoints. The implementation of this is not necessarily the best and serves as an example; not the only way to do this.