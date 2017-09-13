---
title: iframe和父窗口通信
date: 2017-09-02 16:28:52
categories: js
tags: js
---
之前做过一个需求，合作方要求以iframe的形式将我们的页面嵌入到他们的页面中，就涉及到了iframe和父窗口的通信，一般需要通信的内容就是发送高度，对方接受到信息后设置iframe的高度，保证iframe正常显示，不会遮挡内容也不会出现滚动条。
### 实现方案
> window.postMessage() 方法可以安全地实现跨源通信。通常，对于两个不同页面的脚本，只有当执行它们的页面位于具有相同的协议（通常为https），端口号（443为https的默认值），以及主机  (两个页面的模数 Document.domain设置为相同的值) 时，这两个脚本才能相互通信。window.postMessage() 方法提供了一种受控机制来规避此限制，只要正确的使用，这种方法就很安全

### 语法介绍
#### iframe内使用postMessage
>postMessage(message, targetOrigin, [transfer]);

**message**

将要发送到其他 window的数据。它将会被结构化克隆算法序列化。这意味着你可以不受什么限制的将数据对象安全的传送给目标窗口而无需自己序列化。

**targetOrigin**

通过窗口的origin属性来指定哪些窗口能接收到消息事件，其值可以是字符串"\*"（表示无限制）或者一个URI。在发送消息的时候，如果目标窗口的协议、主机地址或端口这三者的任意一项不匹配targetOrigin提供的值，那么消息就不会被发送；只有三者完全匹配，消息才会被发送。这个机制用来控制消息可以发送到哪些窗口；例如，当用postMessage传送密码时，这个参数就显得尤为重要，必须保证它的值与这条包含密码的信息的预期接受者的orign属性完全一致，来防止密码被恶意的第三方截获。如果你明确的知道消息应该发送到哪个窗口，那么请始终提供一个有确切值的targetOrigin，而不是\*。不提供确切的目标将导致数据泄露到任何对数据感兴趣的恶意站点。

**transfer** 可选

是一串和message 同时传递的 Transferable 对象. 这些对象的所有权将被转移给消息的接收方，而发送一方将不再保有所有权。

#### 父窗口监听message
```
window.addEventListener("message", receiveMessage, false);

function receiveMessage(event)
{
  // For Chrome, the origin property is in the event.originalEvent
  // object.
  var origin = event.origin || event.originalEvent.origin; 
  if (origin !== "http://example.org:8080")
    return;

  // ...
}
```
message 的属性有:

**data**

从其他 window 中传递过来的对象。

**origin**

调用 postMessage  时消息发送方窗口的 origin . 这个字符串由 协议、“://“、域名、“ : 端口号”拼接而成。例如 “https://example.org (implying port 443)”、“http://example.net (implying port 80)”、“http://example.com:8080”。请注意，这个origin不能保证是该窗口的当前或未来origin，因为postMessage被调用后可能被导航到不同的位置。

**source**

对发送消息的窗口对象的引用; 您可以使用此来在具有不同origin的两个窗口之间建立双向通信。

### 实际使用经验
* post高度的时候，一般使用document.body.clientHeight获取iframe内的内容的高度
* 为了快速设置高度，可以在未加载完成的时候，就先post一个高度，等页面onload的时候在post一次高度
* 注意在页面高度变化的时候需要重新设置高度，页面高度变化一般都是由于发生点击重绘页面，因此点击的时候需要post高度，有时会异步请求一些资源，如果不能在callback中设置高度，可以通过设置定时器不断post高度来解决
* postMessage不仅仅是用来设置高度，可以处理任何消息，不要思维定视

### 参考资料
[MDN web docs](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage)