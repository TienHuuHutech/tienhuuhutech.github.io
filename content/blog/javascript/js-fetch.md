---
title: "JavaScript Fetch API: Gọi HTTP Request Hiện Đại Và Mạnh Mẽ"
date: 2025-12-27T18:00:00+07:00
description: "Tìm hiểu chi tiết Fetch API - cách tiêu chuẩn hiện đại để thực hiện HTTP requests trong JavaScript, thay thế XMLHttpRequest cũ kỹ"
tags: ["javascript", "fetch", "api", "http", "promise", "async"]
featured_image: ""
---

Chào mừng bạn đến với bài viết về **JavaScript Fetch API**! Sau khi đã nắm vững Event Loop và cơ chế bất đồng bộ, giờ là lúc khám phá công cụ mạnh mẽ nhất để giao tiếp với server: **Fetch API**.

Fetch API cung cấp giao diện đơn giản, dựa trên Promise để thực hiện các HTTP requests (GET, POST, PUT, DELETE...) một cách hiện đại và dễ đọc.

## Fetch API Là Gì?

### 🌐 Modern HTTP Client trong JavaScript

Fetch API là giao diện toàn cục (`fetch()`) có sẵn trong:
- Tất cả các trình duyệt hiện đại
- Node.js (từ v18 trở lên, trước đó cần polyfill hoặc thư viện như node-fetch)

### 🎯 XMLHttpRequest vs Fetch API - So Sánh Chi Tiết

| Đặc điểm            | XMLHttpRequest (XHR)          | Fetch API                               |
| ------------------- | ----------------------------- | --------------------------------------- |
| **Giao diện**       | Callback-based                | Promise-based                           |
| **Đọc response**    | Phức tạp (onreadystatechange) | Dễ dàng với `.json()`, `.text()`        |
| **Error handling**  | Chỉ catch network error       | Cần kiểm tra `response.ok`              |
| **Abort request**   | `abort()`                     | AbortController + AbortSignal           |
| **Streaming**       | Hạn chế                       | Hỗ trợ Response.body (ReadableStream)   |
| **Headers/Body**    | String chỉ                    | Headers object, Body mixins             |
| **CORS**            | Có                            | Có, mặc định credentials: 'same-origin' |
| **Progress events** | Có                            | Không (trừ streaming)                   |

### 💡 Khi Nào Dùng Fetch?

**Sử dụng Fetch cho:**
- Gọi REST APIs
- Upload/Download files
- Real-time data (kết hợp với Server-Sent Events)
- Modern web apps (React, Vue, Angular...)
- Server-side rendering (Next.js, Node.js)

**KHÔNG dùng Fetch khi cần:**
- Progress bar chi tiết (upload/download) → dùng XHR hoặc thư viện như axios
- Hỗ trợ browser cũ (IE) → cần polyfill

## Cấu Trúc Cơ Bản Của Fetch

### HTTP Request/Response Cycle




### Cú pháp cơ bản

```javascript
fetch(url, options)
  .then(response => {
    // Kiểm tra status
    if (!response.ok) throw new Error('Network error');
    return response.json(); // hoặc .text(), .blob(), .arrayBuffer()
  })
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.error('Fetch failed:', error);
  });
```

## Fetch GET Request

### Simple GET

```javascript
// GET JSON từ API
fetch('https://jsonplaceholder.typicode.com/posts/1')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return response.json();
  })
  .then(post => {
    console.log('Post title:', post.title);
  })
  .catch(err => console.error(err));
```

### GET với Query Parameters

```javascript
const params = new URLSearchParams({
  userId: 1,
  _limit: 5
});

fetch(`https://jsonplaceholder.typicode.com/posts?${params}`)
  .then(res => res.json())
  .then(posts => console.log(posts));
```

## Fetch POST/PUT/DELETE

### POST JSON Data

```javascript
const newPost = {
  title: 'Bài viết mới',
  body: 'Nội dung bài viết',
  userId: 1
};

fetch('https://jsonplaceholder.typicode.com/posts', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    // 'Authorization': 'Bearer token'
  },
  body: JSON.stringify(newPost)
})
.then(res => res.json())
.then(createdPost => console.log('Created:', createdPost));
```

### PUT (Update)

```javascript
fetch('https://jsonplaceholder.typicode.com/posts/1', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    id: 1,
    title: 'Updated title',
    body: 'Updated body'
  })
})
.then(res => res.json())
.then(updated => console.log(updated));
```

### DELETE

```javascript
fetch('https://jsonplaceholder.typicode.com/posts/1', {
  method: 'DELETE'
})
.then(res => {
  if (res.ok) console.log('Deleted successfully');
});
```

## Sử Dụng Async/Await (Khuyến nghị)

```javascript
async function getUser(userId) {
  try {
    const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const user = await response.json();
    return user;
  } catch (error) {
    console.error('Fetch error:', error);
    throw error;
  }
}

// Sử dụng
getUser(1).then(user => console.log(user.name));
```

## Abort Request Với AbortController




```javascript
const controller = new AbortController();
const signal = controller.signal;

// Bắt đầu fetch với timeout 5s
const timeoutId = setTimeout(() => controller.abort(), 5000);

fetch('https://slow-api.example.com/data', { signal })
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => {
    if (err.name === 'AbortError') {
      console.log('Request bị hủy do timeout');
    } else {
      console.error(err);
    }
  })
  .finally(() => clearTimeout(timeoutId));

// Hủy thủ công
// controller.abort();
```

## Upload File Với FormData

```javascript
const formData = new FormData();
formData.append('username', 'john_doe');
formData.append('avatar', fileInput.files[0]);

fetch('/api/upload', {
  method: 'POST',
  body: formData
  // Không set Content-Type → browser tự set với boundary
})
.then(res => res.json())
.then(result => console.log('Upload success:', result));
```

## Custom Request Headers & CORS

```javascript
fetch('https://api.example.com/data', {
  method: 'GET',
  headers: {
    'Authorization': 'Bearer ' + token,
    'Accept': 'application/json',
    'X-Custom-Header': 'value'
  },
  credentials: 'include' // Gửi cookies (CORS cần allow)
})
.then(res => res.json());
```

## Xử Lý Response Streaming (Advanced)

```javascript
fetch('https://api.example.com/large-file')
  .then(response => {
    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    
    return new ReadableStream({
      start(controller) {
        function pump() {
          return reader.read().then(({ done, value }) => {
            if (done) {
              controller.close();
              return;
            }
            controller.enqueue(value);
            console.log(decoder.decode(value));
            return pump();
          });
        }
        return pump();
      }
    });
  });
```

## Error Handling Chi Tiết

```javascript
async function safeFetch(url) {
  try {
    const response = await fetch(url);
    
    // Network error → throw tự động
    // HTTP error (4xx, 5xx) → không throw, phải check
    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      throw new Error(errorData.message || `HTTP ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    if (error.name === 'TypeError') {
      console.error('Network error hoặc CORS');
    } else if (error.name === 'AbortError') {
      console.error('Request bị hủy');
    }
    throw error;
  }
}
```

## Best Practices

### 1. Luôn kiểm tra `response.ok`
```javascript
if (!response.ok) throw new Error(...);
```

### 2. Sử dụng async/await thay vì .then chain dài
### 3. Wrap fetch thành helper function
```javascript
const api = {
  get: (url) => fetch(url).then(handleResponse),
  post: (url, data) => fetch(url, {method: 'POST', body: JSON.stringify(data), headers: {'Content-Type': 'application/json'}}).then(handleResponse)
};
```

### 4. Timeout mặc định cho mọi request
```javascript
function fetchWithTimeout(url, options = {}, timeout = 8000) {
  const controller = new AbortController();
  const id = setTimeout(() => controller.abort(), timeout);
  
  return fetch(url, { ...options, signal: controller.signal })
    .finally(() => clearTimeout(id));
}
```

### 5. Không lạm dụng streaming trừ khi thật sự cần

Streaming response rất mạnh mẽ nhưng phức tạp và tốn tài nguyên. Chỉ sử dụng khi:
- Xử lý file lớn (download/upload progressive)
- Server-Sent Events hoặc real-time data stream
- Cần hiển thị tiến độ từng phần (chat streaming, log tailing...)

Với các API thông thường (JSON, text nhỏ), hãy dùng `.json()` hoặc `.text()` để code đơn giản và dễ bảo trì hơn.

## Troubleshooting Thường Gặp

### Problem: Lỗi CORS (Cross-Origin Resource Sharing)

**Nguyên nhân phổ biến:**
- Server không cho phép origin của bạn truy cập

**Giải pháp:**
- **Production**: Server phải set header phù hợp:
 
```http
Access-Control-Allow-Origin: https://your-domain.com
  // hoặc * (không khuyến khích nếu có credentials)
  Access-Control-Allow-Credentials: true (nếu cần cookie)
```

- **Development**: 
  - Dùng proxy trong dev server (Vite, Create React App, Next.js đều hỗ trợ)
  - Hoặc dùng browser extension như "CORS Unblock" (chỉ để test)

### Problem: 401 Unauthorized

**Kiểm tra:**
- Token có hợp lệ và chưa hết hạn không?
- Header `Authorization: Bearer <token>` đã được gửi đúng chưa?
- Nếu dùng cookie/session: đã set `credentials: 'include'` chưa?
- Server có yêu cầu preflight OPTIONS không (CORS với custom headers)?

### Problem: Request không gửi cookie / session

```javascript
fetch(url, {
  method: 'POST',
  credentials: 'include'  // Bắt buộc để gửi cookie cross-origin
  // hoặc 'same-origin' nếu chỉ cần cùng domain
})
```

**Lưu ý**: Khi dùng `credentials: 'include'`, server phải set:
```http
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: https://your-domain.com  // KHÔNG dùng *
```

### Problem: Response rỗng hoặc lỗi khi parse JSON

**Nguyên nhân:**
- Server trả về HTML error page (404, 500) thay vì JSON
- Content-Type không phải application/json

**Giải pháp an toàn:**

```javascript
async function parseJsonSafely(response) {
  const contentType = response.headers.get('content-type');
  
  if (contentType && contentType.includes('application/json')) {
    return await response.json();
  }
  
  // Nếu không phải JSON, trả về text để debug
  const text = await response.text();
  console.warn('Expected JSON but got:', text);
  throw new Error('Invalid JSON response');
}
```

Hoặc xử lý tổng quát:

```javascript
const response = await fetch(url);
if (!response.ok) {
  const text = await response.text();
  throw new Error(`HTTP ${response.status}: ${text}`);
}

try {
  return await response.json();
} catch (e) {
  const text = await response.text();
  throw new Error(`JSON parse error: ${text}`);
}
```

## Kết Luận

Trong bài viết này, chúng ta đã học:
- Sự khác biệt giữa Fetch và XHR
- Các method HTTP cơ bản với Fetch
- Async/await và Promise handling
- Abort requests, upload file, custom headers
- Streaming response và error handling tốt
- Best practices để code sạch và an toàn

Fetch API là công cụ mạnh mẽ, hiện đại và là lựa chọn mặc định cho mọi HTTP request trong JavaScript ngày nay.

## Tài Liệu Tham Khảo

- MDN Web Docs: [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- WHATWG Fetch Standard
- "Using Fetch" - MDN Guide
- Jake Archibald: [Fetch basics](https://jakearchibald.com/2015/thats-so-fetch/)

---
Happy fetching! 🚀 Hãy thử build một mini app gọi API và chia sẻ kết quả nhé!