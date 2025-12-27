---
title: "JavaScript WebSocket: Giao Tiếp Real-Time Đầy Đủ Duplex"
date: 2025-12-27T18:00:00+07:00
description: "Tìm hiểu WebSocket API trong JavaScript - giao thức mạnh mẽ cho phép giao tiếp hai chiều real-time giữa client và server"
tags: ["javascript", "websocket", "real-time", "socket", "browser", "nodejs"]
featured_image: ""
---

Chào mừng bạn đến với bài viết về **JavaScript WebSocket**! Sau Fetch API cho các request HTTP truyền thống, giờ chúng ta sẽ khám phá **WebSocket** - công cụ lý tưởng cho các ứng dụng cần giao tiếp **real-time** như chat, game online, notification live, stock ticker...




WebSocket cho phép kết nối **full-duplex** (hai chiều) liên tục giữa client và server, khác hẳn với mô hình request-response của HTTP.

## WebSocket Là Gì?

### 🔗 Giao thức giao tiếp real-time

WebSocket là giao thức tầng ứng dụng (RFC 6455) cho phép:
- **Full-duplex**: Client và server có thể gửi dữ liệu cho nhau bất kỳ lúc nào
- **Persistent connection**: Kết nối duy trì mở sau handshake
- **Low overhead**: Sau handshake, header rất nhỏ (2-10 bytes/frame)
- **Bidirectional**: Không cần polling liên tục




### 🎯 HTTP Polling vs Long Polling vs SSE vs WebSocket

| Phương thức                  | Hướng dữ liệu            | Overhead   | Real-time     | Use case tốt nhất               |
| ---------------------------- | ------------------------ | ---------- | ------------- | ------------------------------- |
| **Short Polling**            | Client → Server → Client | Cao        | Không         | Dữ liệu cập nhật chậm           |
| **Long Polling**             | Client → Server → Client | Trung bình | Gần real-time | Notification đơn giản           |
| **Server-Sent Events (SSE)** | Server → Client chỉ      | Thấp       | Có            | Live feed, news ticker          |
| **WebSocket**                | Hai chiều đầy đủ         | Rất thấp   | Có            | Chat, multiplayer game, trading |




### 💡 Khi Nào Dùng WebSocket?

**Sử dụng WebSocket cho:**
- Chat applications (Zalo, Messenger style)
- Multiplayer online games
- Real-time collaboration (Google Docs)
- Live sports scores, stock prices
- IoT dashboard
- Notification hệ thống phức tạp

**KHÔNG dùng WebSocket cho:**
- Request đơn giản (dùng Fetch)
- Chỉ cần dữ liệu từ server → client (dùng SSE)
- Browser cũ không hỗ trợ (cần fallback)

## WebSocket Handshake

### Quá trình thiết lập kết nối

1. Client gửi HTTP request với header `Upgrade: websocket`
2. Server trả về `101 Switching Protocols` nếu chấp nhận
3. Kết nối chuyển sang giao thức WebSocket

```http
GET /chat HTTP/1.1
Host: example.com:8000
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

## WebSocket API Cơ Bản (Client-side)

### Tạo kết nối WebSocket

```javascript
const socket = new WebSocket('wss://example.com/socket');

// Các event listener
socket.onopen = (event) => {
  console.log('Kết nối WebSocket đã mở!');
  socket.send('Hello Server!');
};

socket.onmessage = (event) => {
  console.log('Tin nhắn từ server:', event.data);
  
  // event.data có thể là string, Blob, ArrayBuffer
  if (typeof event.data === 'string') {
    const message = JSON.parse(event.data);
    console.log('Parsed:', message);
  }
};

socket.onclose = (event) => {
  console.log('Kết nối đã đóng', event.code, event.reason);
};

socket.onerror = (error) => {
  console.error('Lỗi WebSocket:', error);
};
```




### Gửi dữ liệu

```javascript
// Gửi string
socket.send('Hello from client');

// Gửi JSON
socket.send(JSON.stringify({
  type: 'message',
  text: 'Xin chào!',
  user: 'John'
}));

// Gửi binary (Blob hoặc ArrayBuffer)
const blob = new Blob(['binary data']);
socket.send(blob);
```

### Đóng kết nối

```javascript
// Đóng bình thường
socket.close();

// Đóng với code và reason
socket.close(1000, 'Client rời khỏi');
```

## Simple Chat Application

### Client HTML + JS

```html
<!DOCTYPE html>
<html>
<head>
  <title>WebSocket Chat</title>
</head>
<body>
  <h1>Real-time Chat</h1>
  <div id="messages"></div>
  <input id="input" placeholder="Nhập tin nhắn..." />
  <button id="send">Gửi</button>

  <script>
    const socket = new WebSocket('wss://your-server.com/chat');
    
    const messagesDiv = document.getElementById('messages');
    const input = document.getElementById('input');
    const sendBtn = document.getElementById('send');
    
    socket.onopen = () => {
      addMessage('Đã kết nối đến server!');
    };
    
    socket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      addMessage(`${data.user}: ${data.text}`);
    };
    
    socket.onclose = () => {
      addMessage('Kết nối đã đóng');
    };
    
    function addMessage(text) {
      const p = document.createElement('p');
      p.textContent = text;
      messagesDiv.appendChild(p);
      messagesDiv.scrollTop = messagesDiv.scrollHeight;
    }
    
    function sendMessage() {
      const text = input.value.trim();
      if (text && socket.readyState === WebSocket.OPEN) {
        const message = {
          user: 'You',
          text: text
        };
        socket.send(JSON.stringify(message));
        addMessage(`Bạn: ${text}`);
        input.value = '';
      }
    }
    
    sendBtn.onclick = sendMessage;
    input.addEventListener('keypress', (e) => {
      if (e.key === 'Enter') sendMessage();
    });
  </script>
</body>
</html>
```

### Server ví dụ với Node.js + ws library

```javascript
// npm install ws
const WebSocket = require('ws');

const wss = new WebSocket.Server({ port: 8080 });

console.log('WebSocket server running on ws://localhost:8080');

wss.on('connection', (ws) => {
  console.log('Client mới kết nối');
  
  ws.on('message', (data) => {
    console.log('Received:', data.toString());
    
    // Broadcast đến tất cả clients
    wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(data);
      }
    });
  });
  
  ws.on('close', () => {
    console.log('Client ngắt kết nối');
  });
});
```

## Xử Lý Reconnection (Quan trọng!)

```javascript
function createWebSocket(url) {
  let socket = new WebSocket(url);
  let reconnectInterval = 1000;
  let maxReconnectInterval = 30000;

  socket.onclose = (event) => {
    console.log('Kết nối đóng, thử kết nối lại...');
    
    setTimeout(() => {
      reconnectInterval = Math.min(reconnectInterval * 2, maxReconnectInterval);
      createWebSocket(url);
    }, reconnectInterval);
  };

  socket.onerror = (error) => {
    console.error('WebSocket error:', error);
    socket.close(); // Trigger reconnect
  };

  return socket;
}

// Sử dụng
const socket = createWebSocket('wss://example.com/socket');
```

## Ping/Pong Heartbeat

```javascript
function startHeartbeat(socket, interval = 30000) {
  const heartbeat = setInterval(() => {
    if (socket.readyState === WebSocket.OPEN) {
      socket.send(JSON.stringify({ type: 'ping' }));
    }
  }, interval);

  socket.onmessage = (event) => {
    if (event.data === 'pong') {
      console.log('Heartbeat OK');
    }
  };

  return () => clearInterval(heartbeat);
}
```

## WebSocket với Authentication

```javascript
// Thêm token vào URL
const socket = new WebSocket('wss://api.example.com/socket?token=abc123');

// Hoặc dùng subprotocol
const socket = new WebSocket('wss://api.example.com/socket', ['access_token', 'abc123']);
```

## Best Practices

### 1. Luôn xử lý reconnection và heartbeat
### 2. Sử dụng wss:// (secure) trong production
### 3. Validate và sanitize message trên server
### 4. Giới hạn kích thước message
### 5. Graceful degradation (fallback sang polling nếu WebSocket fail)

```javascript
// Kiểm tra hỗ trợ
if ('WebSocket' in window) {
  // Dùng WebSocket
} else {
  // Fallback sang long polling
}
```

### 6. Sử dụng thư viện nếu cần phức tạp (Socket.io, SignalR)

## Troubleshooting

### Problem: Kết nối bị từ chối (101 → không)
**Giải pháp:**
- Server chưa hỗ trợ Upgrade header
- CORS policy (WebSocket không dùng CORS thông thường nhưng cần đúng origin)
- Proxy (Nginx, Cloudflare) chưa config WebSocket

**Nginx config ví dụ:**
```nginx
location /socket {
  proxy_pass http://backend;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
}
```

### Problem: Kết nối đóng đột ngột
- Network không ổn định → implement reconnect
- Server timeout → gửi heartbeat

### Problem: Message không đến
- Kiểm tra `readyState === WebSocket.OPEN` trước khi send
- Server có broadcast đúng không?

## Kết Luận

Trong bài viết này, chúng ta đã học:
- Sự khác biệt giữa WebSocket và các phương thức real-time khác
- Cách thiết lập và sử dụng WebSocket API trong JavaScript
- Xây dựng ứng dụng chat đơn giản
- Xử lý reconnection, heartbeat, authentication
- Best practices và troubleshooting thường gặp

WebSocket là nền tảng cho hầu hết các ứng dụng real-time hiện đại. Hãy bắt đầu xây dựng một dự án chat hoặc dashboard live để thực hành nhé!

## Tài Liệu Tham Khảo

- MDN Web Docs: [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- RFC 6455 - The WebSocket Protocol
- Socket.io (thư viện phổ biến với fallback)
- ws library cho Node.js

---
Happy real-time coding! 🚀 Chia sẻ dự án WebSocket của bạn nhé!