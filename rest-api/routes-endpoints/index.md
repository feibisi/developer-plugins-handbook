---
title: "Routes & Endpoints"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Routes Endpoints
    - Rest Api
    - Repos
---

# Routes & Endpoints

The REST API provides a way to match URIs to various resources in our WordPress installation. By default, if you have pretty permalinks enabled, the WordPress REST API “lives” at `/wp-json/`. At our WordPress site `https://ourawesomesite.com`, we can access the REST API’s index by making a `GET` request to `https://ourawesomesite.com/wp-json/`. The index provides information regarding what routes are available for that particular WordPress install, along with what HTTP methods are supported and what endpoints are registered.

If we wanted to create an endpoint that would return the phrase “Hello World, this is the WordPress REST API”, we would first need to register the route for that endpoint. To register routes you should use the `register_rest_route()` function. It needs to be called on the `rest_api_init` action hook. `register_rest_route()` handles all of the mapping for routes to endpoints. Let’s try to create a “Hello World, this is the WordPress REST API” route.

```
/**
 * This is our callback function that embeds our phrase in a WP_REST_Response
 */
function prefix_get_endpoint_phrase() {
    // rest_ensure_response() wraps the data we want to return into a WP_REST_Response, and ensures it will be properly returned.
    return rest_ensure_response( 'Hello World, this is the WordPress REST API' );
}

/**
 * This function is where we register our routes for our example endpoint.
 */
function prefix_register_example_routes() {
    // register_rest_route() handles more arguments but we are going to stick to the basics for now.
    register_rest_route( 'hello-world/v1', '/phrase', array(
        // By using this constant we ensure that when the WP_REST_Server changes our readable endpoints will work as intended.
        'methods'  => WP_REST_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP_REST_Server class.
        'callback' => 'prefix_get_endpoint_phrase',
    ) );
}

add_action( 'rest_api_init', 'prefix_register_example_routes' );
```

The first argument passed into `register_rest_route()` is the namespace, which provides us a way to group our routes. The second argument passed in is the resource path, or resource base. For our example, the resource we are retrieving is the “Hello World, this is the WordPress REST API” phrase. The third argument is an array of options. We specify what methods the endpoint can use and what callback should happen when the endpoint is matched (more things can be done but these are the fundamentals).

The third argument also allows us to provide a permissions callback, which can restrict access for the endpoint to only certain users. The third argument also offers a way to register arguments for the endpoint so that requests can modify the response of our endpoint. We will get into those concepts in the endpoints section of this guide.

当我们访问 `https://ourawesomesite.com/wp-json/hello-world/v1/phrase` 时，现在可以看到我们的 REST API 友好地向我们致意。让我们更深入地了解一下路由。

## 路由

REST API 中的路由由 URI 表示。路由本身是附加在 `https://ourawesomesite.com/wp-json` 末尾的部分。API 的索引路由是 `'/'`，这就是为什么 `https://ourawesomesite.com/wp-json/` 会返回 API 的所有可用信息。所有路由都应构建在此路由之上，`wp-json` 部分可以更改，但通常建议保持不变。

我们需要确保路由是唯一的。例如，我们可以为书籍设置这样的路由：`/books`。我们的书籍路由现在将位于 `https://ourawesomesite.com/wp-json/books`。然而，这不是一个好的做法，因为我们会最终污染 API 的潜在路由。如果另一个插件也想注册一个书籍路由怎么办？那样我们会遇到大麻烦，因为两个路由会相互冲突，只能使用其中一个。`register_rest_field()` 的第四个参数是一个布尔值，用于指示路由是否应覆盖现有路由。

覆盖参数也不能真正解决我们的问题，因为两个路由都可能覆盖，或者我们可能希望将两个路由用于不同目的。这就是为路由使用命名空间的原因。

### 命名空间

为路由添加命名空间至关重要。那些等待被合并到 WordPress 核心的“核心”端点使用 `/wp/v2` 命名空间。

[info]**除非你创建端点是为了将其合并到核心中，否则请勿将任何内容放入 `/wp` 命名空间。**[/info]

核心端点命名空间中有一些关键点需要注意。命名空间的第一部分是 `/wp`，代表供应商名称，即 WordPress。对于我们的插件，我们需要为命名空间的供应商部分想一个独特的名称。在上面的例子中，我们使用了 `hello-world`。

供应商部分之后是命名空间的版本部分。“核心”端点使用 `v2` 来代表 WordPress REST API 的版本 2。如果你正在编写一个插件，可以通过简单地创建新端点并提升你提供的版本号来保持 REST API 端点的向后兼容性。这样，原始的 `v1` 和 `v2` 端点都可以被访问。

路由中命名空间之后的部分是资源路径。

### 资源路径

资源路径应体现端点关联的资源类型。在前述示例中，我们使用 `phrase` 一词表示当前交互的资源是短语。为避免命名冲突，每个注册的资源路径在其命名空间内应保持唯一性。资源路径应用于定义特定命名空间内的不同资源路由。

假设某插件处理基础电商功能，我们将涉及订单和商品两类核心资源。订单是对商品的请求，但并非商品本身。同理，商品资源虽与订单相关却本质不同，应分别置于独立的资源路径中。最终电商插件的路由将呈现为：`/my-shop/v1/orders` 与 `/my-shop/v1/products`。

采用此类路由结构时，每个端点应返回对应资源的集合。若需通过ID获取特定商品，则需在路由中使用路径变量。

### Path Variables

Path variables enable us to add dynamic routes. To expand on our eCommerce routes, we could register a route to grab individual products.

```
/**
 * This is our callback function to return our products.
 *
 * @param WP_REST_Request $request This function accepts a rest request to process data.
 */
function prefix_get_products( $request ) {
    // In practice this function would fetch the desired data. Here we are just making stuff up.
    $products = array(
        '1' => 'I am product 1',
        '2' => 'I am product 2',
        '3' => 'I am product 3',
    );

    return rest_ensure_response( $products );
}

/**
 * This is our callback function to return a single product.
 *
 * @param WP_REST_Request $request This function accepts a rest request to process data.
 */
function prefix_get_product( $request ) {
    // In practice this function would fetch the desired data. Here we are just making stuff up.
    $products = array(
        '1' => 'I am product 1',
        '2' => 'I am product 2',
        '3' => 'I am product 3',
    );

    // Here we are grabbing the 'id' path variable from the $request object. WP_REST_Request implements ArrayAccess, which allows us to grab properties as though it is an array.
    $id = (string) $request['id'];

    if ( isset( $products[ $id ] ) ) {
        // Grab the product.
        $product = $products[ $id ];

        // Return the product as a response.
        return rest_ensure_response( $product );
    } else {
        // Return a WP_Error because the request product was not found. In this case we return a 404 because the main resource was not found.
        return new WP_Error( 'rest_product_invalid', esc_html__( 'The product does not exist.', 'my-text-domain' ), array( 'status' => 404 ) );
    }

    // If the code somehow executes to here something bad happened return a 500.
    return new WP_Error( 'rest_api_sad', esc_html__( 'Something went horribly wrong.', 'my-text-domain' ), array( 'status' => 500 ) );
}

/**
 * This function is where we register our routes for our example endpoint.
 */
function prefix_register_product_routes() {
    // Here we are registering our route for a collection of products.
    register_rest_route( 'my-shop/v1', '/products', array(
        // By using this constant we ensure that when the WP_REST_Server changes our readable endpoints will work as intended.
        'methods'  => WP_REST_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP_REST_Server class.
        'callback' => 'prefix_get_products',
    ) );
    // Here we are registering our route for single products. The (?P<id>[\d]+) is our path variable for the ID, which, in this example, can only be some form of positive number.
    register_rest_route( 'my-shop/v1', '/products/(?P<id>[\d]+)', array(
        // By using this constant we ensure that when the WP_REST_Server changes our readable endpoints will work as intended.
        'methods'  => WP_REST_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP_REST_Server class.
        'callback' => 'prefix_get_product',
    ) );
}

add_action( 'rest_api_init', 'prefix_register_product_routes' );
```

The above example covers a lot. The important part to note is that in the second route we register, we add on a path variable `/(?P[\d]+)` to our resource path `/products`. The path variable is a regular expression. In this case it uses `[\d]+` to signify that should be any numerical character at least once. If you are using numeric IDs for your resources, then this is a great example of how to use a path variable. When using path variables, we now have to be careful around what can be matched as it is user input.

The regex luckily will filter out anything that is not numerical. However, what if the product for the requested ID doesn’t exist. We need to do error handling. You can see the basic way we are handling errors in the code example above. When you return a `WP_Error` in your endpoint callbacks the API server will automatically handle serving the error to the client.

Although this section is about routes, we have covered quite a bit about endpoints. Endpoints and routes are interrelated, but they definitely have distinctions.

## Endpoints

Endpoints are the destination that a route needs to map to. For any given route, you could have a number of different endpoints registered to it. We will expand on our fictitious eCommerce plugin, to better show the distinction between routes and endpoints. We are going to create two endpoints that exist at the `/wp-json/my-shop/v1/products/` route. One endpoint uses the HTTP verb `GET` to get products, and the other endpoint uses the HTTP verb `POST` to create a new product.

```
/**
 * This is our callback function to return our products.
 *
 * @param WP_REST_Request $request This function accepts a rest request to process data.
 */
function prefix_get_products( $request ) {
    // In practice this function would fetch the desired data. Here we are just making stuff up.
    $products = array(
        '1' => 'I am product 1',
        '2' => 'I am product 2',
        '3' => 'I am product 3',
    );

    return rest_ensure_response( $products );
}

/**
 * This is our callback function to return a single product.
 *
 * @param WP_REST_Request $request This function accepts a rest request to process data.
 */
function prefix_create_product( $request ) {
    // In practice this function would create a product. Here we are just making stuff up.
   return rest_ensure_response( 'Product has been created' );
}

/**
 * This function is where we register our routes for our example endpoint.
 */
function prefix_register_product_routes() {
    // Here we are registering our route for a collection of products and creation of products.
    register_rest_route( 'my-shop/v1', '/products', array(
        array(
            // By using this constant we ensure that when the WP_REST_Server changes, our readable endpoints will work as intended.
            'methods'  => WP_REST_Server::READABLE,
            // Here we register our callback. The callback is fired when this endpoint is matched by the WP_REST_Server class.
            'callback' => 'prefix_get_products',
        ),
        array(
            // By using this constant we ensure that when the WP_REST_Server changes, our create endpoints will work as intended.
            'methods'  => WP_REST_Server::CREATABLE,
            // Here we register our callback. The callback is fired when this endpoint is matched by the WP_REST_Server class.
            'callback' => 'prefix_create_product',
        ),
    ) );
}

add_action( 'rest_api_init', 'prefix_register_product_routes' );
```

Depending on what HTTP Method we use for the route `/wp-json/my-shop/v1/products`, we are matched to a different endpoint and a different callback is fired. When we use `POST` we trigger the `prefix_create_product()` callback, and when we use `GET` we trigger the `prefix_get_products()` callback.

There are a number of different HTTP methods and the REST API can make use of any of them.

### HTTP 方法

HTTP 方法有时被称为 HTTP 动词。它们只是通过 HTTP 进行通信的不同方式。WordPress REST API 主要使用以下几种：

- `GET` 应用于从 API 检索数据。
- `POST` 应用于创建新资源（例如用户、文章、分类法）。
- `PUT` 应用于更新资源。
- `DELETE` 应用于删除资源。
- `OPTIONS` 应用于提供关于资源的上下文信息。

需要注意的是，并非所有客户端都支持这些方法，因为它们是在 HTTP 1.1 中引入的。幸运的是，API 为这些情况提供了变通方案。如果你想删除一个资源但无法发送 `DELETE` 请求，那么你可以在请求中使用 `_method` 参数或 `X-HTTP-Method-Override` 请求头。其工作原理是：你将发送一个 `POST` 请求到 `https://ourawesomesite.com/wp-json/my-shop/v1/products/1?_method=DELETE`。这样，即使你的客户端无法在请求中发送正确的 HTTP 方法，或者存在防火墙阻止了 DELETE 请求，你也能成功删除 1 号产品。

HTTP 方法与路由和回调函数相结合，构成了端点的核心。

### 回调函数

REST API 目前仅支持两种端点回调类型：`callback` 和 `permissions_callback`。主回调函数负责处理与资源的交互，权限回调函数则管理用户对端点的访问权限。您可以在注册端点时添加额外信息来扩展回调功能，随后通过挂载 `rest_pre_dispatch`、`rest_dispatch_request` 或 `rest_post_dispatch` 钩子来触发自定义回调。

#### 端点回调

删除端点的主回调应仅删除资源并在响应中返回其副本。创建端点的主回调应仅创建资源并返回与新创建数据匹配的响应。更新回调应仅修改实际存在的资源。读取回调应仅检索已存在的数据。必须考虑幂等性概念。

在 REST API 中，幂等性意味着如果向端点发送相同请求，服务器将以相同方式处理请求。假设我们的读取端点不具备幂等性，那么每次请求时服务器状态都会被修改，即使我们只是尝试获取数据。这可能是灾难性的——每当有人从服务器获取数据时，内部状态都会发生变化。必须确保读取、更新和删除端点不会产生有害副作用，严格遵循其设计功能。

在 REST API 中，幂等性概念与 HTTP 方法而非端点回调绑定。任何使用 `GET`、`HEAD`、`TRACE`、`OPTIONS`、`PUT` 或 `DELETE` 的回调都不应产生副作用。`POST` 请求不具备幂等性，通常用于创建资源。如果创建幂等的创建方法，则只会生成一个资源，因为重复相同请求不会对服务器产生额外副作用。对于创建操作，重复发送相同请求时服务器应每次都生成新资源。

为限制端点使用，我们需要注册权限回调。

#### Permissions Callback

Permissions callbacks are extremely important for security with the WordPress REST API. If you have any private data that should not be displayed publicly, then you need to have permissions callbacks registered for your endpoints. Below is an example of how to register permissions callbacks.

```
/**
 * This is our callback function that embeds our resource in a WP_REST_Response
 */
function prefix_get_private_data() {
    // rest_ensure_response() wraps the data we want to return into a WP_REST_Response, and ensures it will be properly returned.
    return rest_ensure_response( 'This is private data.' );
}

/**
 * This is our callback function that embeds our resource in a WP_REST_Response
 */
function prefix_get_private_data_permissions_check() {
    // Restrict endpoint to only users who have the edit_posts capability.
    if ( ! current_user_can( 'edit_posts' ) ) {
        return new WP_Error( 'rest_forbidden', esc_html__( 'OMG you can not view private data.', 'my-text-domain' ), array( 'status' => 401 ) );
    }

    // This is a black-listing approach. You could alternatively do this via white-listing, by returning false here and changing the permissions check.
    return true;
}

/**
 * This function is where we register our routes for our example endpoint.
 */
function prefix_register_example_routes() {
    // register_rest_route() handles more arguments but we are going to stick to the basics for now.
    register_rest_route( 'my-plugin/v1', '/private-data', array(
        // By using this constant we ensure that when the WP_REST_Server changes our readable endpoints will work as intended.
        'methods'  => WP_REST_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP_REST_Server class.
        'callback' => 'prefix_get_private_data',
        // Here we register our permissions callback. The callback is fired before the main callback to check if the current user can access the endpoint.
        'permissions_callback' => 'prefix_get_private_data_permissions_check',
    ) );
}

add_action( 'rest_api_init', 'prefix_register_example_routes' );
```

If you try out this endpoint without any Authentication enabled then you will also be returned the error response, preventing you from seeing the data. Authentication is a huge topic and eventually a portion of this chapter will be created to show you how to create your own authentication processes.

### Arguments

When making requests to an endpoint you might need to specify extra parameters to change the response. These extra parameters can be added while registering endpoints. Let’s look at an example of how to use arguments with an endpoint.

```
/**
 * This is our callback function that embeds our resource in a WP_REST_Response
 */
function prefix_get_colors( $request ) {
    // In practice this function would fetch the desired data. Here we are just making stuff up.
    $colors = array(
        'blue',
        'blue',
        'red',
        'red',
        'green',
        'green',
    );

    if ( isset( $request['filter'] ) ) {
       $filtered_colors = array();
       foreach ( $colors as $color ) {
           if ( $request['filter'] === $color ) {
               $filtered_colors[] = $color;
           }
       }
       return rest_ensure_response( $filtered_colors );
    }
    return rest_ensure_response( $colors );
}

/**
 * We can use this function to contain our arguments for the example product endpoint.
 */
function prefix_get_color_arguments() {
    $args = array();
    // Here we are registering the schema for the filter argument.
    $args['filter'] = array(
        // description should be a human readable description of the argument.
        'description' => esc_html__( 'The filter parameter is used to filter the collection of colors', 'my-text-domain' ),
        // type specifies the type of data that the argument should be.
        'type'        => 'string',
        // enum specified what values filter can take on.
        'enum'        => array( 'red', 'green', 'blue' ),
    );
    return $args;
}

/**
 * This function is where we register our routes for our example endpoint.
 */
function prefix_register_example_routes() {
    // register_rest_route() handles more arguments but we are going to stick to the basics for now.
    register_rest_route( 'my-colors/v1', '/colors', array(
        // By using this constant we ensure that when the WP_REST_Server changes our readable endpoints will work as intended.
        'methods'  => WP_REST_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP_REST_Server class.
        'callback' => 'prefix_get_colors',
        // Here we register our permissions callback. The callback is fired before the main callback to check if the current user can access the endpoint.
        'args' => prefix_get_color_arguments(),
    ) );
}

add_action( 'rest_api_init', 'prefix_register_example_routes' );
```

在此示例中，我们已为 `filter` 参数进行了定义。我们可以在请求端点时将该参数作为查询参数传递。如果向 `https://ourawesomesitem.com/my-colors/v1/colors?filter=blue` 发起 `GET` 请求，将仅返回集合中的蓝色数据。您也可以将这些参数作为请求体参数传递，而非置于查询字符串中。要理解查询参数与请求体参数的区别，请查阅 HTTP 规范说明。查询参数位于 URL 附加的查询字符串中，而请求体参数则直接嵌入 HTTP 请求的正文内。

我们已为端点创建了参数，但如何验证该参数是否为字符串，并判断其是否匹配红、绿或蓝的取值？为此，我们需要为该参数指定验证回调函数。

#### Validation

Validation and sanitization are extremely important for security in the API. The validate callback (in WP 4.6+), fires before the sanitize callback. You should use the `validate_callback` for your arguments to verify whether the input you are receiving is valid. The `sanitize_callback` should be used to transform the argument input or clean out unwanted parts out of the argument, before the argument is processed by the main callback.

In the example above, we need to verify that the `filter` parameter is a string, and it matches the value red, green, or blue. Let’s look at what the code looks like after adding in a `validate_callback`.

```
/**
 * This is our callback function that embeds our resource in a WP_REST_Response
 */
function prefix_get_colors( $request ) {
    // In practice this function would fetch more practical data. Here we are just making stuff up.
    $colors = array(
        'blue',
        'blue',
        'red',
        'red',
        'green',
        'green',
    );

    if ( isset( $request['filter'] ) ) {
       $filtered_colors = array();
       foreach ( $colors as $color ) {
           if ( $request['filter'] === $color ) {
               $filtered_colors[] = $color;
           }
       }
       return rest_ensure_response( $filtered_colors );
    }
    return rest_ensure_response( $colors );
}
/**
 * Validate a request argument based on details registered to the route.
 *
 * @param  mixed            $value   Value of the 'filter' argument.
 * @param  WP_REST_Request  $request The current request object.
 * @param  string           $param   Key of the parameter. In this case it is 'filter'.
 * @return WP_Error|boolean
 */
function prefix_filter_arg_validate_callback( $value, $request, $param ) {
    // If the 'filter' argument is not a string return an error.
    if ( ! is_string( $value ) ) {
        return new WP_Error( 'rest_invalid_param', esc_html__( 'The filter argument must be a string.', 'my-text-domain' ), array( 'status' => 400 ) );
    }

    // Get the registered attributes for this endpoint request.
    $attributes = $request->get_attributes();

    // Grab the filter param schema.
    $args = $attributes['args'][ $param ];

    // If the filter param is not a value in our enum then we should return an error as well.
    if ( ! in_array( $value, $args['enum'], true ) ) {
        return new WP_Error( 'rest_invalid_param', sprintf( __( '%s is not one of %s' ), $param, implode( ', ', $args['enum'] ) ), array( 'status' => 400 ) );
    }
}

/**
 * We can use this function to contain our arguments for the example product endpoint.
 */
function prefix_get_color_arguments() {
    $args = array();
    // Here we are registering the schema for the filter argument.
    $args['filter'] = array(
        // description should be a human readable description of the argument.
        'description' => esc_html__( 'The filter parameter is used to filter the collection of colors', 'my-text-domain' ),
        // type specifies the type of data that the argument should be.
        'type'        => 'string',
        // enum specified what values filter can take on.
        'enum'        => array( 'red', 'green', 'blue' ),
        // Here we register the validation callback for the filter argument.
        'validate_callback' => 'prefix_filter_arg_validate_callback',
    );
    return $args;
}

/**
 * This function is where we register our routes for our example endpoint.
 */
function prefix_register_example_routes() {
    // register_rest_route() handles more arguments but we are going to stick to the basics for now.
    register_rest_route( 'my-colors/v1', '/colors', array(
        // By using this constant we ensure that when the WP_REST_Server changes our readable endpoints will work as intended.
        'methods'  => WP_REST_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP_REST_Server class.
        'callback' => 'prefix_get_colors',
        // Here we register our permissions callback. The callback is fired before the main callback to check if the current user can access the endpoint.
        'args' => prefix_get_color_arguments(),
    ) );
}

add_action( 'rest_api_init', 'prefix_register_example_routes' );
```

#### Sanitizing

In the above example, we do not need to use a `sanitize_callback`, because we are restricting input to only values in our enum. If we did not have strict validation and accepted any string as a parameter, we would definitely need to register a `sanitize_callback`. What if we wanted to update a content field and the user entered something like `alert('ZOMG Hacking you');`. The field value could potentially be a executable script. To strip out unwanted data or to transform data into a desired format we need to register a `sanitize_callback` for our arguments. Here is an example of how to use WordPress’s `sanitize_text_field()` for a sanitize callback:

```
/**
 * This is our callback function that embeds our resource in a WP_REST_Response.
 *
 * The parameter is already sanitized by this point so we can use it without any worries.
 */
function prefix_get_item( $request ) {
    if ( isset( $request['data'] ) ) {
        return rest_ensure_response( $request['data'] );
    }

    return new WP_Error( 'rest_invalid', esc_html__( 'The data parameter is required.', 'my-text-domain' ), array( 'status' => 400 ) );
}

/**
 * Validate a request argument based on details registered to the route.
 *
 * @param  mixed            $value   Value of the 'filter' argument.
 * @param  WP_REST_Request  $request The current request object.
 * @param  string           $param   Key of the parameter. In this case it is 'filter'.
 * @return WP_Error|boolean
 */
function prefix_data_arg_validate_callback( $value, $request, $param ) {
    // If the 'data' argument is not a string return an error.
    if ( ! is_string( $value ) ) {
        return new WP_Error( 'rest_invalid_param', esc_html__( 'The filter argument must be a string.', 'my-text-domain' ), array( 'status' => 400 ) );
    }
}

/**
 * Sanitize a request argument based on details registered to the route.
 *
 * @param  mixed            $value   Value of the 'filter' argument.
 * @param  WP_REST_Request  $request The current request object.
 * @param  string           $param   Key of the parameter. In this case it is 'filter'.
 * @return WP_Error|boolean
 */
function prefix_data_arg_sanitize_callback( $value, $request, $param ) {
    // It is as simple as returning the sanitized value.
    return sanitize_text_field( $value );
}

/**
 * We can use this function to contain our arguments for the example product endpoint.
 */
function prefix_get_data_arguments() {
    $args = array();
    // Here we are registering the schema for the filter argument.
    $args['data'] = array(
        // description should be a human readable description of the argument.
        'description' => esc_html__( 'The data parameter is used to be sanitized and returned in the response.', 'my-text-domain' ),
        // type specifies the type of data that the argument should be.
        'type'        => 'string',
        // Set the argument to be required for the endpoint.
        'required'    => true,
        // We are registering a basic validation callback for the data argument.
        'validate_callback' => 'prefix_data_arg_validate_callback',
        // Here we register the validation callback for the filter argument.
        'sanitize_callback' => 'prefix_data_arg_sanitize_callback',
    );
    return $args;
}

/**
 * This function is where we register our routes for our example endpoint.
 */
function prefix_register_example_routes() {
    // register_rest_route() handles more arguments but we are going to stick to the basics for now.
    register_rest_route( 'my-plugin/v1', '/sanitized-data', array(
        // By using this constant we ensure that when the WP_REST_Server changes our readable endpoints will work as intended.
        'methods'  => WP_REST_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP_REST_Server class.
        'callback' => 'prefix_get_item',
        // Here we register our permissions callback. The callback is fired before the main callback to check if the current user can access the endpoint.
        'args' => prefix_get_data_arguments(),
    ) );
}

add_action( 'rest_api_init', 'prefix_register_example_routes' );
```

## 总结

我们已经介绍了为 WordPress REST API 注册端点的基本知识。路由是端点所在的 URI。端点是一组回调函数、方法、参数和其他选项的集合。使用 `register_rest_route()` 时，每个端点都会映射到一个路由。默认情况下，一个端点可以支持多种 HTTP 方法、一个主回调函数、一个权限回调函数以及已注册的参数。我们可以注册端点来满足与 WordPress 交互的任何用例。端点是 REST API 的核心交互点，但要充分利用这个强大的 API，还有许多其他主题需要探索和理解。