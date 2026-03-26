---
title: "Heartbeat API"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Heartbeat Api
    - Javascript
    - Repos
---

# Heartbeat API

Heartbeat API 是 WordPress 内置的简易服务器轮询 API，可实现近乎实时的前端更新。

## 工作原理

页面加载时，客户端心跳代码会设置一个间隔（称为"tick"），每 15-120 秒运行一次。运行时，heartbeat 会收集数据并通过 jQuery 事件发送，然后将数据发送到服务器并等待响应。在服务器端，admin-ajax 处理器接收传递的数据，准备响应，过滤响应，最后以 JSON 格式返回数据。客户端接收数据后触发最终的 jQuery 事件，表示数据已接收。

自定义 Heartbeat 事件的基本流程：

1. 向待发送数据添加额外字段（JS `heartbeat-send` 事件）
2. 在 PHP 中检测发送字段，并添加响应字段（`heartbeat_received` 过滤器）
3. 在 JS 中处理返回数据（JS `heartbeat-tick` 事件）

（根据功能需求，可选择仅使用其中一或两个事件）

## Using the API

Using the heartbeat API requires two separate pieces of functionality: send and receive callbacks in JavaScript, and a server-side filter to process passed data in PHP.

### Sending Data to the Server

When Heartbeat sends data to the server, you can include custom data. This can be any data you want to send to the server, or a simple true value to indicate you are expecting data.

```
jQuery( document ).on( 'heartbeat-send', function ( event, data ) {
  // Add additional data to Heartbeat data.
  data.myplugin_customfield = 'some_data';
});
```

### Receiving and Responding on the Server

On the server side, you can then detect this data, and add additional data to the response.

```
/**
 * Receive Heartbeat data and respond.
 *
 * Processes data received via a Heartbeat request, and returns additional data to pass back to the front end.
 *
 * @param array $response Heartbeat response data to pass back to front end.
 * @param array $data     Data received from the front end (unslashed).
 *
 * @return array
 */
function myplugin_receive_heartbeat( array $response, array $data ) {
  // If we didn't receive our data, don't send any back.
  if ( empty( $data['myplugin_customfield'] ) ) {
    return $response;
  }

  // Calculate our data and pass it back. For this example, we'll hash it.
  $received_data = $data['myplugin_customfield'];

  $response['myplugin_customfield_hashed'] = sha1( $received_data );
  return $response;
}
add_filter( 'heartbeat_received', 'myplugin_receive_heartbeat', 10, 2 );
```

Back on the frontend, you can then handle receiving this data back.

```
jQuery( document ).on( 'heartbeat-tick', function ( event, data ) {
  // Check for our data, and use it.
  if ( ! data.myplugin_customfield_hashed ) {
    return;
  }

  alert( 'The hash is ' + data.myplugin_customfield_hashed );
});
```

Not every feature will need all three of these steps. For example, if you don’t need to send any data to the server, you can use just the latter two steps.