---
title: SSE总结
created: 2025-12-16T06:38:42.173Z
updated: 2025-12-16T06:38:42.173Z
slug: ssezong-jie
tags: []
description: ""
---

---
title: SSE总结
created: 2023-12-10 02:40:10
updated: 2023-12-10 04:43:51
---

Server-Sent Events 服务器推送**事件**，是一种在客户端和服务器之间实现**单向**、实时通信的Web技术
**特点**

- 只允许服务器**主动**推送数据到客户端（单向）
- 基于 HTTP 协议，比较**轻量**
- 内置断线重连和消息追踪功能
- 只支持传送文本（二进制数据需要编码后传送）
## 协议
[SSE规范-W3C](https://html.spec.whatwg.org/multipage/server-sent-events.html#server-sent-events)
### **请求头**

- MIME 类型为`text/event-stream`
- 指定浏览器不缓存服务端发送的数据，确保数据的实时性
- 保持连接打开，允许服务器持续发送数据
```http
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
```
### **消息格式**
服务器响应的数据必须为 UTF-8 编码的文本
**事件**指服务器发送到客户端的数据单元，每个事件由一行或多行字段组成，每行格式为`filed: value`，每个事件以`\n\n`结束
```http
: this is a test stream\n\n
event: custom
data: some text\n\n

event: foo
data: another message\n
data: with two lines \n\n
```
**字段**

- `id`事件的唯一标识符
   - 浏览器会用`lastEventId`记录该值，通过`event.lastEventId`可读取
   - 发生断连时浏览器会进行重试，重试发起的请求头中会携带 `Last-Event-Id`字段将`lastEventId`发送给服务端，帮助服务器端重建连接
- `event`事件类型，浏览器接收到数据后会触发 `EventSource`实例以该字段为名的事件
- `data`消息数据
   - 数据内容只能以一个字符串的文本形式进行发送
   - 数据很长可以分成多行发送，最后一行用`\n\n`结尾
- `retry`发生断联时浏览器等待指定时间进行重连
   - 整数值(单位 ms)，如果该字段不是整数值，会被忽略
   - 服务端没有指定时由浏览器自行决定重连时间
- 注释行
   - 冒号开头的行，表示注释
   - 通常用来防止连接中断（服务器可以定时发送注释行，保持连接不中断）
- 其他情况
   - 出现其他所有字段都会被忽略
   - 如果一行字段中不包含冒号，则整行文本将被视为字段名，字段值为空
## 客户端 API
浏览器提供了`EventSource`接口用于创建与服务端的连接并接受服务端事件的推送
```typescript
const eventSource = new EventSource('/api/sse', { withCredentials: true })
```
### 基本使用

- 实例化`EventSource`之后会立即向服务器发起连接（GET 请求）
- `EventSource`也可以跨域，跨域时传递第二个参数 `{ withCredentials: true }`携带`cookie`
- 浏览器按照**事件**（以两个换行符为结束符）来接收数据，如果一个事件包含了多个`data`字段，`event.data`为所有的`data`字段合并的值
- `readyState`属性
   - `0`: 连接还未建立，或者断线正在重连。等于常量`EventSource.CONNECTING`
   - `1`: 连接已经建立，可以接受数据。等于常量`EventSource.OPEN`
   - `2`: 连接已断，且不会重连。等于常量`EventSource.CLOSED`
- `close`方法用于关闭 SSE 连接
- 内置事件（可以通过`on`、`addEventListener`两种方式监听）
   - `open`成功连接到服务端时触发
   - `message`接收到服务器发送的消息时触发，`e.data`为服务器发送的消息内容（data字段内容）
   - `error`发生通信错误时触发（比如连接中断），`e.event`包含了错误信息
### 自定义事件
服务器发来的数据默认触发`message`事件，但是对于自定义事件（存在自定义 event 字段）不会触发 `message`事件
> 或者说服务器发来数据会触发该消息对应的 event 事件，默认 event 名称为 message

自定义事件需要使用 `addEventListener`进行监听，并且只能使用 `addEventListener`监听
```typescript
eventSource.addEventListener('log', e => { })
```
### 自定义实现
原生的`EventSource`API 不能自定义请求头、只能发出 GET 请求，如果需要更多的能力可以用原生的 ajax 的方式模拟实现

本质上 SSE 是借助于 HTTP 保持长连接传输流来实现的
## 服务端实现
```typescript
import http from 'node:http'

interface SendMessageOptions {
  initId: number
  event: string
}

const createSendMessage = (response: http.ServerResponse<http.IncomingMessage>, options: SendMessageOptions) => {
  let _id = options.initId
  const event = options.event
  return (data?: any) => {
    const id = _id++
    response.write(`event: ${event}\n`)
    response.write(`id: ${id}\n`)
    response.write(`retry: 30000\n`)
    response.write(`data: ${JSON.stringify({ id, time: new Date().toLocaleString(), data })}\n\n`)
  }
}

const server = http.createServer((request, response) => {
  if (request.url === '/sse') {
    response.writeHead(200, {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
      'Access-Control-Allow-Origin': '*',
    })

    const initId = typeof request.headers['Last-Event-Id'] === 'string' ? parseInt(request.headers['Last-Event-Id']) : 0

    const sendMessage = createSendMessage(response, { initId, event: 'custome' })
    
    // 每隔 1 秒发送一条消息
    const intervalId = setInterval(() => sendMessage('hello'), 1000)

    // 当客户端关闭连接时停止发送消息
    request.on('close', () => {
      clearInterval(intervalId)
      response.end()
    })
  } else {
    response.writeHead(404)
    response.end()
  }
})

server.listen(3000, () => console.log('Server listening on port 3000'))
```
## 本质
> **以流的形式进行响应，完成一次用时很长的下载**

SSE 和**长连接轮询**有些类似，区别在于长连接一个连接只发送一次数据，而 SSE 会一直保持连接，在一个连接中多次推送消息。
SSE 本质上服务端发送的一个数据流（不是一次性的数据包），所以客户端会一直保持连接，等待服务端发送新的数据流。

**参考**

- [一文读懂即时更新方案：SSE](https://juejin.cn/post/7221125237500330039)
- [Server-Sent Events 教程](https://www.ruanyifeng.com/blog/2017/05/server-sent_events.html)
- ChatGPT
