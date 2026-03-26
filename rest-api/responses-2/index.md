---
title: "响应"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Responses 2
    - Rest Api
    - Repos
---

# 响应

## 概述

API 中的响应承载了我们所需的所有数据。如果我们的请求有误，响应数据也应告知我们发生了错误。WordPress REST API 中的响应应返回我们请求的数据或错误信息。API 中的响应由 `WP_REST_Response` 类处理，这是 API 的三个基础类之一。

## WP_REST_Response

`WP_REST_Response` 继承自 WordPress 的 `WP_HTTP_Response` 类，允许我们访问响应头、响应状态码和响应数据。

```
// 以下代码仅作演示，不会执行任何操作。
$response = new WP_REST_Response( '这是一些数据' );

// 要获取响应数据，我们可以使用此方法。它应等于'这是一些数据'。
$our_data = $response->get_data();

// 要访问 HTTP 状态码，我们可以使用此方法。最常见的状态码可能是 200，表示成功！
$our_status = $response->get_status();

// 要访问 HTTP 响应头，我们可以使用此方法。
$our_headers = $response->get_headers();
```

以上内容相当直接，展示了如何从响应中获取所需信息。`WP_REST_Response` 更进一步。您可以通过 `$response->get_matched_route()` 访问响应匹配的路由，以追溯响应来自哪个端点。`$response->get_matched_handler()` 将返回生成我们响应的端点所注册的选项。这些功能对于记录 API 日志等用途可能很有用。响应类还帮助我们进行错误处理。

### Error Handling

If something went terribly wrong in our request, we can return `WP_Error` objects in our endpoint callbacks explaining what went wrong, like this:

```
// Register our mock batch endpoint.
function prefix_register_broken_route() {
    register_rest_route( 'my-namespace/v1', '/broken', array(
        // Supported methods for this endpoint. WP_REST_Server::READABLE translates to GET.
        'methods' => WP_REST_Server::READABLE,
        // Register the callback for the endpoint.
        'callback' => 'prefix_get_an_error',
    ) );
}

add_action( 'rest_api_init', 'prefix_register_broken_route' );

/**
 * Our registered endpoint callback. Notice how we are passing in $request as an argument.
 * By default, the WP_REST_Server will pass in the matched request object to our callback.
 *
 * @param WP_REST_Request $request The current matched request object.
 */
function prefix_get_an_error( $request ) {
    return new WP_Error( 'oops', esc_html__( 'This endpoint is currently broken, try another endpoint, I promise the API is cool! EEEK!!!!', 'my-textdomain' ), array( 'status' => 400 ) );
}
```

That is kind of a silly example but it touches on some key things. The most important thing to understand is that the WordPress REST API will automatically handle changing the [WP\_Error](https://developer.wordpress.org/reference/classes/wp_error/) object into an HTTP Response containing your data. When you set the status code in the `WP_Error` object your HTTP response status code will take on that value. This comes in really handy when you need to use different error codes like 404 for content that wasn’t found, or 403 for forbidden access. All we have to do is have our endpoint callbacks return a request and the `WP_REST_Server` class will handle a lot of really important things for us.

There are other cool things the response class can help us with, like Linking.

## Linking

What if we wanted to get a post and the first comment for that post? Would we write a separate endpoint to handle this use case? If we did that, we would need to start adding a lot of endpoints to handle various small use cases and our API index would get bloated really fast. Response Linking provides us a way to form relations between our resources that the API can understand. The API implements a standard known as HAL for resource linking. Let’s look at our post and comment example, it would be better to have routes for each resource.

Let’s say we have post with ID = 1 and comment ID = 3. The comment is assigned to post 1, so realistically the two resources could live at the routes `/my-namespace/v1/posts/1` and `/my-namespace/v1/comments/3`. We would add links to the responses to create the relationships between them. Let’s look at this from the comment perspective first.

```
// Register our mock endpoints.
function prefix_register_my_routes() {
    register_rest_route( 'my-namespace/v1', '/posts/(?P<id>[\d]+)', array(
        // Supported methods for this endpoint. WP_REST_Server::READABLE translates to GET.
        'methods' => WP_REST_Server::READABLE,
        // Register the callback for the endpoint.
        'callback' => 'prefix_get_rest_post',
    ) );
    register_rest_route( 'my-namespace/v1', '/comments', array(
        // Supported methods for this endpoint. WP_REST_Server::READABLE translates to GET.
        'methods' => WP_REST_Server::READABLE,
        // Register the callback for the endpoint.
        'callback' => 'prefix_get_rest_comments',
        // Register the post argument to limit results to a specific post parent.
        'args' => array(
            'post' => array(
                'description' => esc_html__( 'The post ID that the comment is assigned to.', 'my-textdomain' ),
                'type'        => 'integer',
                'required'    => true,
            ),
        ),
    ) );
    register_rest_route( 'my-namespace/v1', '/comments/(?P<id>[\d]+)', array(
        // Supported methods for this endpoint. WP_REST_Server::READABLE translates to GET.
        'methods' => WP_REST_Server::READABLE,
        // Register the callback for the endpoint.
        'callback' => 'prefix_get_rest_comment',
    ) );
}

add_action( 'rest_api_init', 'prefix_register_my_routes' );

// Grab a post.
function prefix_get_rest_post( $request ) {
    $id = (int) $request['id'];
    $post = get_post( $id );

    $response = rest_ensure_response( array( $post ) );

    $response->add_links( prefix_prepare_post_links( $post ) );

    return $response;
}

// Prepare post links.
function prefix_prepare_post_links( $post ) {
    $links = array();

    $replies_url = rest_url( 'my-namespace/v1/comments' );
    $replies_url = add_query_arg( 'post', $post->ID, $replies_url );
    $links['replies'] = array(
		'href'         => $replies_url,
		'embeddable'   => true,
    );

    return $links;
}

// Grab a comments.
function prefix_get_rest_comments( $request ) {
    if ( ! isset( $request['post'] ) ) {
        return new WP_Error( 'rest_bad_request', esc_html__( 'You must specify the post parameter for this request.', 'my-text-domain' ), array( 'status' => 400 ) );
    }

    $data = array();

    $comments = get_comments( array( 'post__in' => $request['post'] ) );

    if ( empty( $comments ) ) {
        return array();
    }

    foreach( $comments as $comment ) {
        $response = rest_ensure_response( $comment );
        $response->add_links( prefix_prepare_comment_links( $comment ) );
        $data[] = prefix_prepare_for_collection( $response );
    }

    $response = rest_ensure_response( $data );
    return $response;
}

// Grab a comment.
function prefix_get_rest_comment( $request ) {
    $id = (int) $request['id'];
    $post = get_comment( $id );

    $response = rest_ensure_response( $comment );

    $response->add_links( prefix_prepare_comment_links( $comment ) );

    return $response;
}

// Prepare comment links.
function prefix_prepare_comment_links( $comment ) {
    $links = array();
    if ( 0 !== (int) $comment->comment_post_ID ) {
        $post = get_post( $comment->comment_post_ID );
        if ( ! empty( $post->ID ) ) {
        $links['up'] = array(
                'href'       => rest_url( 'my-namespace/v1/posts/' . $comment->comment_post_ID ),
                'embeddable' => true,
                'post_type'  => $post->post_type,
            );
        }
    }
    return $links;
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
```

如上方示例所示，我们使用链接来建立资源间的关系。若文章存在评论，端点回调将添加指向评论路由的链接，并通过 `post` 参数匹配当前文章ID。访问该路由即可获取关联此文章ID的评论。搜索评论时，每条评论均包含指向文章的 `up` 链接——在遵循HAL规范的链接中，`up` 具有特殊含义：跟随评论的向上链接将返回该评论的父级文章。链接机制已足够出色，但还有更强大的功能。

WordPress REST API 同时支持嵌入功能。请注意我们为两个链接设置的 `embeddable => true` 参数，这使得关联数据可嵌入响应中。例如要获取编号3的评论及其关联文章，可发起请求：`https://ourawesomesite.com/wp-json/my-namespace/v1/comments/3?_embed`。`_embed` 参数指示API将当前请求中所有可嵌入资源链接一并返回。嵌入功能通过单次HTTP请求处理多个资源，显著提升了性能表现。

巧妙运用嵌入与链接机制，使WordPress REST API在交互操作中展现出卓越的灵活性与强大功能。