---
title: gRPC-web 入门
date: 2024-08-1
categories: [Front-end, utils]
tags: [frontend, gRPC]     # TAG names should always be lowercase
# comments: false 
pin: false
math: true
mermaid: true
toc: true
---

# [gRPC-web](https://github.com/grpc/grpc-web)

## Description 描述

- gRPC是google于2015年开源的一个RPC框架。它是基于protoBuf和HTTP/2实现的。它是一个现代化、开源、基于HTTP/2协议的 RPC 框架，使用强类型二进制数据（protobuf）进行通讯。

- 由于浏览器没有直接暴露 HTTP/2，web不能直接使用 gRPC，gRPC-web 是一个JavaScript 客户端库，解决了这个问题，可以在浏览器中使用gRPC。
- gRPC-Web协议跟原生的gRPC协议有所不同。gRPC-web协议设计原则是让浏览器和gRPC服务之间的代理服务器可以更容易的对协议进行翻译。

<img src="https://pf6xrzskv9.feishu.cn/space/api/box/stream/download/asynccode/?code=Njc2NzkxYmViOTYxZDY0YWUyOGU1M2NlMjJmYTQxMGZfRlNnbkwzcHpRYXhwTzNQc0pUZ3Q3a1UyMThLMDZVN0RfVG9rZW46TXNiS2I4cXg5bzJoOWh4bEVwVWM4eUlwbkVkXzE3MjI1NjQ4MzY6MTcyMjU2ODQzNl9WNA" alt="img" style="zoom:50%;" />

### Protobuf

**Protobuf** 是一种平台无关、语言无关、可扩展且轻便高效的序列化数据结构的**协议**，可以用于**网络通信**和**数据存储**。

客户端和服务端需要同时维护一份 `.proto` 文件来进行序列化 = `[JSON.stringify]`，反序列化 = `[JSON.parse]`

主要的优势 

> 使用二进制格式进行传输，编码体积被压缩，排列简单，解析算法复杂度小，在大数据量，高频率的交互场景下被广泛使用

- 序列化速度快
- 传输速度快

约定 `.proto` 文件

```protobuf
syntax = "proto3";

package write;


service WriteService {
  rpc Writing (WriteRequest) returns (ResponseMessage) {}
}

message WriteRequest {
  //类型 | 字段名字|  标号
  string input = 1;
}

// message WriteResponse {
//   repeated int64 ids = 1; // repeated 表示数组
//   Value info = 2;         // 可嵌套对象
//   map<int, Value> values = 3;    // 可输出map映射
// }

message WriteResult {
  string output = 2;
}

// 定义一个通用的响应消息
message ResponseMessage {
  string type = 1;
  int32 code = 2;
  string message = 3;
  WriteResult result = 4;
}
```

## 通信模式

### Unary (单一调用) [支持](https://github.com/grpc/grpc-web/blob/master/doc/streaming-roadmap.md)

- 客户端发送一个请求，服务器返回一个响应

- ```protobuf
  rpc UnaryCall (UnaryRequest) returns (UnaryResponse);
  ```

- ```javascript
  /**
   * @param {WriteRequest} request
   * @param {Metadata} metadata
   * @param {function(Error, ResponseMessage)} callback
   */
  client.unaryCall(request, {}, (err, response) => {
    if (err) {
      console.error('Error:', err.message);
      return;
    }
    console.log('Response:', response.getReply());
  });
  ```

### Server Streaming（服务器流式）支持

- 客户端发送一个请求，服务器返回一个数据流，客户端可以多次接收响应

- ```protobuf
  rpc ServerStreamingCall (ServerStreamingRequest) returns (stream ServerStreamingResponse);
  ```

- ```javascript
  /**
   * @param {WriteRequest} request
   * @param {Metadata} metadata
   */
  const stream = client.serverStreamingCall(request, {});
  stream.on('data', (response) => {
    console.log('Response:', response.getReply());
  });
  stream.on('end', () => {
    console.log('Stream ended.');
  });
  stream.on('error', (err) => {
    console.error('Error:', err.message);
  });
  ```

### ~~Client Streaming（客户端流式）~~ 不支持

- 客户端发送一个数据流到服务器，服务器返回一个响应。

- 适用于需要客户端发送多个请求的场景，例如上传大文件。

- ```
  rpc ClientStreamingCall (stream ClientStreamingRequest) returns (ClientStreamingResponse);
  ```

- ```javascript
  const stream = client.clientStreamingCall((err, response) => {
    if (err) {
      console.error('Error:', err.message);
      return;
    }
    console.log('Response:', response.getReply());
  });
  
  // 发送多个请求
  stream.write(request1);
  stream.write(request2);
  stream.end();
  ```

### ~~Bidirectional Streaming（双向流式）~~ 不支持

- 客户端和服务器之间建立一个双向的数据流，双方可以独立地发送和接收消息

- ```protobuf
  rpc BidirectionalStreamingCall (stream BidirectionalStreamingRequest) returns (stream BidirectionalStreamingResponse);
  ```

- ```javascript
  const stream = client.bidirectionalStreamingCall();
  
  // 监听服务器的响应
  stream.on('data', (response) => {
    console.log('Response:', response.getReply());
  });
  stream.on('end', () => {
    console.log('Stream ended.');
  });
  stream.on('error', (err) => {
    console.error('Error:', err.message);
  });
  
  // 发送多个请求
  stream.write(request1);
  stream.write(request2);
  stream.end();
  ```

  

## 配置 gRPC-web 客户端

### 安装环境

1. 必要依赖
   install (`grpc-web`) ：用于在浏览器中使用gRPC
   install (`google-protobuf`)：用于处理 Protocol Buffers 消息

   ```shell
   npm install grpc-web google-protobuf
   ```

2. 编译工具
   install Protobuf (`protoc`)

   ```shell
   # macOS
   brew install protobuf
   # download
   https://github.com/protocolbuffers/protobuf/releases
   ```

   install Protobuf-javascript (`protoc-gen-js`)

   ```shell
   npm install -g protoc-gen-js
   ```

   install gRPC-Web code generator

   ```shell
   # macOS
   sudo mv protoc-gen-grpc-web-1.5.0-darwin-aarch64 \
       /usr/local/bin/protoc-gen-grpc-web
   
   chmod +x /usr/local/bin/protoc-gen-grpc-web
   ```

### gRPC-web 客户端 代码生成

- `protoc` Code Generator Plugins 代码生成插件

- 生成 javascript 中间代码命令 Options
  ```shell
  # js
  protoc -I=$DIR [filename].proto \
    --js_out=import_style=commonjs:$OUT_DIR \
    --grpc-web_out=import_style=commonjs,mode=grpcwebtext:$OUT_DIR
    
  # ts
  protoc -I=. [filename].proto \
    --js_out=import_style=commonjs:./generate[name] \
    --grpc-web_out=import_style=typescript,mode=grpcwebtext:./generate[name]
  ```
  
  - `--js_out`
    - 支持
      - The default generated code has [Closure](https://developers.google.com/closure/library/) `goog.require()` import style.
      - The [CommonJS](https://requirejs.org/docs/commonjs.html) style `require()` is also supported.
  - `--grpc-web_out`
    - 支持
      - Closure
      - CommonJS
      - commonjs+dts
      - typescript
  - `mode`
    - `mode=grpcwebtext`: The default generated code sends the payload in the `grpc-web-text` format.
      - `Content-type: application/grpc-web-text`
      - Payload are base64-encoded.
      - Both unary and server streaming calls are supported.
    - `mode=grpcweb`: A binary protobuf format is also supported.
      - `Content-type: application/grpc-web+proto`
      - Payload are in the binary protobuf format.
      - Only unary calls are supported.

## 封装连接

```javascript
// grpcClient.js
import { ExampleServiceClient } from './generated/example_grpc_web_pb';

const client = new ExampleServiceClient('http://localhost:8080', null, null);

// 封装一个 promise 请求
const grpcCallAsync = (methodName, request) => {
  return new Promise((resolve, reject) => {
    client[methodName](request, {}, (err, response) => {
      if (err) {
        reject(err);
      } else {
        resolve(response);
      }
    });
  });
};

export { grpcCallAsync };
```

```javascript
// 使用
try {
  const response = await grpcCallAsync('unaryCall', request);
  setResponse1(response);
} catch (err) {
  console.error('Error:', err.message);
}
```

### setting deadline header

```javascript
var deadline = new Date();
deadline.setSeconds(deadline.getSeconds() + 1);

client.sayHelloAfterDelay(request, {deadline: deadline.getTime().toString()},
  (err, response) => {
    // err will be populated if the RPC exceeds the deadline
    ...
  });
```

### 使用拦截器 interceptor

可以实现以下功能：

1. **日志记录**：记录请求和响应的详细信息。
2. **认证**：在请求头中添加认证信息。
3. **错误处理**：捕获和处理错误。
4. **重试机制**：实现请求的重试逻辑。
