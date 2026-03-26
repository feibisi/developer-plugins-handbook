---
title: "控制器类"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Controller Classes
    - Rest Api
    - Repos
---

# 控制器类

## 概述

编写端点时，使用控制器类来处理端点功能会很有帮助。控制器类将提供与 API 交互的标准方式，以及更易于维护的交互方式。WordPress 当前最低 PHP 版本为 5.2，如果您开发的端点将被整个 WordPress 生态系统使用，应考虑支持 WordPress 的最低要求。

PHP 5.2 本身不支持命名空间。这意味着您声明的每个函数都将位于全局作用域中。如果您决定为端点使用通用函数名（如 `get_items()`），而另一个插件也注册了该函数，PHP 将因致命错误而失败。这是因为函数 `get_items()` 被重复声明。通过封装端点，我们可以避免这些命名冲突，并保持与 API 交互的一致性。

## Controllers

Controllers typically do one thing; they receive input, and generate output. For the WordPress REST API our controllers will handle request input as `WP_REST_Request` objects and generate response output as `WP_REST_Response` objects. Let’s look at an example controller class:

```
class My_REST_Posts_Controller {

	// Here initialize our namespace and resource name.
	public function __construct() {
		$this->namespace     = '/my-namespace/v1';
		$this->resource_name = 'posts';
	}

	// Register our routes.
	public function register_routes() {
		register_rest_route( $this->namespace, '/' . $this->resource_name, array(
			// Here we register the readable endpoint for collections.
			array(
				'methods'   => 'GET',
				'callback'  => array( $this, 'get_items' ),
				'permission_callback' => array( $this, 'get_items_permissions_check' ),
			),
			// Register our schema callback.
			'schema' => array( $this, 'get_item_schema' ),
		) );
		register_rest_route( $this->namespace, '/' . $this->resource_name . '/(?P<id>[\d]+)', array(
			// Notice how we are registering multiple endpoints the 'schema' equates to an OPTIONS request.
			array(
				'methods'   => 'GET',
				'callback'  => array( $this, 'get_item' ),
				'permission_callback' => array( $this, 'get_item_permissions_check' ),
			),
			// Register our schema callback.
			'schema' => array( $this, 'get_item_schema' ),
		) );
	}

	/**
	 * Check permissions for the posts.
	 *
	 * @param WP_REST_Request $request Current request.
	 */
	public function get_items_permissions_check( $request ) {
		if ( ! current_user_can( 'read' ) ) {
			return new WP_Error( 'rest_forbidden', esc_html__( 'You cannot view the post resource.' ), array( 'status' => $this->authorization_status_code() ) );
		}
		return true;
	}

	/**
	 * Grabs the five most recent posts and outputs them as a rest response.
	 *
	 * @param WP_REST_Request $request Current request.
	 */
	public function get_items( $request ) {
		$args = array(
			'post_per_page' => 5,
		);
		$posts = get_posts( $args );

		$data = array();

		if ( empty( $posts ) ) {
			return rest_ensure_response( $data );
		}

		foreach ( $posts as $post ) {
			$response = $this->prepare_item_for_response( $post, $request );
			$data[] = $this->prepare_response_for_collection( $response );
		}

		// Return all of our comment response data.
		return rest_ensure_response( $data );
	}

	/**
	 * Check permissions for the posts.
	 *
	 * @param WP_REST_Request $request Current request.
	 */
	public function get_item_permissions_check( $request ) {
		if ( ! current_user_can( 'read' ) ) {
			return new WP_Error( 'rest_forbidden', esc_html__( 'You cannot view the post resource.' ), array( 'status' => $this->authorization_status_code() ) );
		}
		return true;
	}

/**
	 * Grabs the five most recent posts and outputs them as a rest response.
	 *
	 * @param WP_REST_Request $request Current request.
	 */
	public function get_item( $request ) {
		$id = (int) $request['id'];
		$post = get_post( $id );

		if ( empty( $post ) ) {
			return rest_ensure_response( array() );
		}

		$response = prepare_item_for_response( $post );

		// Return all of our post response data.
		return $response;
	}

	/**
	 * Matches the post data to the schema we want.
	 *
	 * @param WP_Post $post The comment object whose response is being prepared.
	 */
	public function prepare_item_for_response( $post, $request ) {
		$post_data = array();

		$schema = $this->get_item_schema( $request );

		// We are also renaming the fields to more understandable names.
		if ( isset( $schema['properties']['id'] ) ) {
			$post_data['id'] = (int) $post->ID;
		}

		if ( isset( $schema['properties']['content'] ) ) {
			$post_data['content'] = apply_filters( 'the_content', $post->post_content, $post );
		}

		return rest_ensure_response( $post_data );
	}

	/**
	 * Prepare a response for inserting into a collection of responses.
	 *
	 * This is copied from WP_REST_Controller class in the WP REST API v2 plugin.
	 *
	 * @param WP_REST_Response $response Response object.
	 * @return array Response data, ready for insertion into collection data.
	 */
	public function prepare_response_for_collection( $response ) {
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
	 * Get our sample schema for a post.
	 *
	 * @param WP_REST_Request $request Current request.
	 */
	public function get_item_schema( $request ) {
		$schema = array(
			// This tells the spec of JSON Schema we are using which is draft 4.
			'$schema'              => 'http://json-schema.org/draft-04/schema#',
			// The title property marks the identity of the resource.
			'title'                => 'post',
			'type'                 => 'object',
			// In JSON Schema you can specify object properties in the properties attribute.
			'properties'           => array(
				'id' => array(
					'description'  => esc_html__( 'Unique identifier for the object.', 'my-textdomain' ),
					'type'         => 'integer',
					'context'      => array( 'view', 'edit', 'embed' ),
					'readonly'     => true,
				),
				'content' => array(
					'description'  => esc_html__( 'The content for the object.', 'my-textdomain' ),
					'type'         => 'string',
				),
			),
		);

		return $schema;
	}

// 为授权设置正确的 HTTP 状态码。
	public function authorization_status_code() {

		$status = 401;

		if ( is_user_logged_in() ) {
			$status = 403;
		}

		return $status;
	}
}

// 用于从控制器注册新路由的函数。
function prefix_register_my_rest_routes() {
	$controller = new My_REST_Posts_Controller();
	$controller->register_routes();
}

add_action( 'rest_api_init', 'prefix_register_my_rest_routes' );
```

## 概述与未来展望

控制器类在开发端点时为我们解决了两个重大问题：命名空间缺失和结构不一致。需要注意的是，不应滥用端点的继承机制。例如：如果您为文章端点编写了控制器类（如上例所示），并希望同时支持自定义文章类型，**不应**像这样扩展 `My_REST_Posts_Controller`：`class My_CPT_REST_Controller extends My_REST_Posts_Controller`。

正确的做法是：要么创建完全独立的控制器类，要么让 `My_REST_Posts_Controller` 处理所有可用的文章类型。当您开始深入继承机制的黑暗深渊时，必须明白：如果父类在任何时候需要修改，而您的子类又依赖于它们，将会带来巨大麻烦。多数情况下，建议创建基础控制器类作为 `接口` 或 `抽象类`，供各个端点控制器实现或扩展。WP REST API 团队正采用 `抽象类` 方案开发 `WP_REST_Controller` 类，以期未来能纳入核心代码。

目前，支持文章、文章类型、文章状态、修订版本、分类法、术语、用户、评论及附件/媒体资源的“核心端点”正在功能插件中开发，有望在未来融入 WordPress 核心。该插件内包含一个提案中的 `WP_REST_Controller` 类，可用于为您的端点构建专属控制器。`WP_REST_Controller` 具备诸多优势，为 API 端点创建提供了统一规范的方法。