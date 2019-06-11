# 使用 postMessage 在跨域页面间传数据

搞了几个微服务，各自都有后端数据和前端页面，不同服务都需要获取主服务提供的 JWT 令牌信息用于后端的用户鉴权。由于主服务和各个微服务之间都是配置为不同的端口，属于跨域页面之间的信息传递，可以使用 `window.postMessage` 方法安全地实现跨源通信。

## 1. 语法：

### 1.1. 发送方调用

```
otherWindow.postMessage(message, targetOrigin, [transfer])
```

|  名称 | 说明
|---------|--------
| otherWindow | 窗口的引用，如全局 `window` 对象、 `iframe.contentWindow`、[window.open] 的返回值
| message     | 发送的信息，值为字符或数据对象
| targetOrigin | 限定可以接收 `message` 的窗口范围，值为一个 URL 或者字符串 `*`，`*` 代表无限制，要慎用；设置为一个 URL 时，则只有与此 URL 同域的窗口才能接收到 `message` 消息
| transfer       | 可选参数，没用过

> `postMessage` 的浏览器兼容性请点击 [这里](https://caniuse.com/#search=postMessage) 查看，mozilla 官方文档点 [这里](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage)。

### 1.2. 接收方监听

```
window.addEventListener("message", receiveMessage, false);

function receiveMessage(event) {
  // 可以通过 origin 控制只接收特定源的消息
  var origin = event.origin
  if (origin !== "http://example.org:8080") return;

  // 处理发送过来的消息
  var data = event.data
  ...
}
```

`addEventListener` 用于监听其它窗口通过 `postMessage` 发送过来的消息，`receiveMessage` 为监听函数，该函数的参数 `event` 含有如下属性：

|  名称 | 说明
|---------|--------
| data     | `postMessage` 函数的 `message` 参数值
| origin   | `postMessage`  函数调用者的 [origin]，如 "http:/ /example.com:8080"
| source | 对发送消息的窗口对象的引用

## 2. 实例

主页面 A 通过 iframe 加载跨域页面 B，页面 B 加载完毕后需要获取主页面 A 保存在 localStorage 内的 jwt 令牌。

页面 A ：（http:/ /a.com/index.html）

```
...
<iframe id="iframe" src="http://b.com/index.html"></iframe>
...
<script>
  iframe.addEventListener("load", function(){
    // 主动发送 jwt 令牌给页面 B
    iframe.contentWindow.postMessage(localStorage.jwt, "http://b.com");
    // or window.postMessage(localStorage.jwt, "http://b.com");
  });
</script>
...
```

页面 B ：（http:/ /b.com/index.html）

```
...
<script>
  window.addEventListener("message", function(event){
    // 接收页面 A 发送过来的 jwt 令牌
    let jwt = event.data;
    ...
  });
</script>
...
```

[origin]: https://developer.mozilla.org/en-US/docs/Origin
[window.open]: https://developer.mozilla.org/en-US/docs/DOM/window.open "DOM/window.open"
[window.open]: https://developer.mozilla.org/en-US/docs/DOM/window.open "DOM/window.open"