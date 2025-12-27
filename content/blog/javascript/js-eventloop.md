---
title: "JavaScript Event Loop: Hiểu Rõ Cơ Chế Bất Đồng Bộ Của JS"
date: 2025-12-27T18:00:00+07:00
description: "Tìm hiểu sâu về JavaScript Event Loop - cơ chế quan trọng giúp JavaScript xử lý bất đồng bộ hiệu quả dù chỉ có một luồng thực thi"
tags: ["javascript", "event-loop", "asynchronous", "nodejs", "browser"]
featured_image: ""
---

Chào mừng bạn đến với bài viết về **JavaScript Event Loop**! Đây là một trong những khái niệm quan trọng nhất để hiểu cách JavaScript xử lý các tác vụ bất đồng bộ (asynchronous) như setTimeout, Promise, async/await, hay các callback từ mạng, file I/O.

JavaScript là ngôn ngữ **single-threaded**, nhưng nhờ Event Loop mà nó vẫn có thể thực hiện các tác vụ không blocking một cách mượt mà.

## Event Loop Là Gì?

### 🔄 Cơ chế cốt lõi của JavaScript Runtime

Event Loop là một vòng lặp vô tận chịu trách nhiệm:
- Thực thi mã đồng bộ (synchronous code)
- Quản lý hàng đợi các tác vụ bất đồng bộ (callback queue)
- Đẩy các callback đã sẵn sàng vào Call Stack để thực thi khi Stack trống

### 🎯 Tại sao cần Event Loop?

JavaScript được thiết kế để chạy trong môi trường **single-threaded** (một luồng duy nhất):
- **Không blocking**: Một tác vụ dài (ví dụ: request mạng) không làm treo toàn bộ ứng dụng
- **Non-blocking I/O**: Các tác vụ I/O được giao cho hệ thống (browser hoặc Node.js) xử lý bên ngoài
- **Callback-based**: Khi tác vụ hoàn thành, callback được đưa vào hàng đợi để xử lý sau

### 💡 Call Stack vs Task Queue vs Microtask Queue

| Thành phần          | Mô tả                                                                | Ví dụ                        |
| ------------------- | -------------------------------------------------------------------- | ---------------------------- |
| **Call Stack**      | Ngăn xếp gọi hàm (LIFO) - nơi thực thi mã JavaScript                 | Hàm gọi hàm, recursion       |
| **Web APIs**        | Các API của browser (setTimeout, DOM events, XMLHttpRequest, fetch)  | Timer, network requests      |
| **Task Queue**      | Hàng đợi các macro-task (callback từ timer, I/O, events)             | setTimeout, setInterval, I/O |
| **Microtask Queue** | Hàng đợi ưu tiên cao hơn (Promise, MutationObserver, queueMicrotask) | Promise.then, async/await    |

**Thứ tự ưu tiên**:
1. Call Stack phải trống
2. Thực thi hết **Microtask Queue**
3. Lấy một **Task** từ Task Queue để thực thi
4. Lặp lại

## Cách Event Loop Hoạt Động

### Minh họa bằng sơ đồ

```
                     +------------------+
                     |   Call Stack     |
                     +--------+---------+
                              ^
                              |
                +--------------+--------------+
                |                             |
      +---------v---------+         +---------v----------+
      |   Microtask Queue |         |    Task Queue      |
      | (Promise, etc.)   |         | (setTimeout, etc.) |
      +-------------------+         +--------------------+
                ^                           ^
                |                           |
      +---------+---------+     +-----------+------------+
      |   Web APIs /      |     |  Node.js libuv thread  |
      |   Thread Pool     |     |        pool            |
      +-------------------+     +------------------------+
```

### Ví dụ minh họa đơn giản

```javascript
console.log('1. Start');

setTimeout(() => {
  console.log('4. setTimeout callback');
}, 0);

Promise.resolve().then(() => {
  console.log('3. Promise then');
});

console.log('2. End');
```

**Kết quả in ra:**
```
1. Start
2. End
3. Promise then
4. setTimeout callback
```

**Giải thích từng bước:**
1. `console.log('1. Start')` → vào Call Stack → in ra → pop
2. `setTimeout(...)` → đăng ký timer với Web API → callback được đẩy vào **Task Queue** sau 0ms
3. `Promise.resolve().then(...)` → Promise resolved ngay → callback `.then` vào **Microtask Queue**
4. `console.log('2. End')` → vào Call Stack → in ra → pop
5. Call Stack trống → Event Loop kiểm tra Microtask Queue → thực thi Promise callback → in "3."
6. Microtask Queue trống → lấy một task từ Task Queue → thực thi setTimeout callback → in "4."

## Các Thành Phần Chi Tiết

### Macro Tasks (Tasks)
- `setTimeout`, `setInterval`
- I/O operations (network, file)
- DOM events (click, load)
- `setImmediate` (Node.js)
- `requestAnimationFrame` (browser)

### Micro Tasks
- `Promise.then/.catch/.finally`
- `queueMicrotask()`
- `MutationObserver`
- `process.nextTick` (Node.js)

### Ví dụ về thứ tự ưu tiên Microtask > Macrotask

```javascript
console.log('Start');

setTimeout(() => {
  console.log('setTimeout 1');
  
  Promise.resolve().then(() => {
    console.log('Promise inside setTimeout');
  });
  
}, 0);

Promise.resolve().then(() => {
  console.log('Promise 1');
  
  setTimeout(() => {
    console.log('setTimeout inside Promise');
  }, 0);
});

console.log('End');
```

**Output:**
```
Start
End
Promise 1
setTimeout 1
Promise inside setTimeout
setTimeout inside Promise
```

### Starvation của Macro Tasks

Nếu Microtask liên tục sinh ra Microtask mới → Macro Tasks bị "đói" (starvation):

```javascript
// NGUY HIỂM - có thể làm treo UI
Promise.resolve().then(function recurse() {
  console.log('Microtask running...');
  // Làm việc nặng
  return Promise.resolve().then(recurse);
});

setTimeout(() => {
  console.log('Tôi sẽ chạy khi nào?'); // Có thể rất lâu!
}, 100);
```

## Event Loop trong Browser vs Node.js

| Đặc điểm          | Browser                     | Node.js                                 |
| ----------------- | --------------------------- | --------------------------------------- |
| Runtime           | V8 + Web APIs               | V8 + libuv                              |
| Task sources      | Timers, DOM events, network | Timers, I/O, setImmediate               |
| Microtasks        | Promise, MutationObserver   | Promise, process.nextTick               |
| Additional phases | Rendering, RAF              | Timers → Pending → Poll → Check → Close |
| process.nextTick  | Không có                    | Ưu tiên cao hơn cả Microtask            |

### Node.js Event Loop Phases

1. **Timers**: thực thi callback của setTimeout/setInterval đã hết hạn
2. **Pending Callbacks**: I/O callbacks bị trì hoãn
3. **Idle, Prepare**: nội bộ
4. **Poll**: lấy I/O events mới, thực thi I/O callbacks
5. **Check**: thực thi setImmediate
6. **Close Callbacks**: socket.on('close'), etc.

## Ví dụ Thực Tế

### Async/Await và Event Loop

```javascript
async function fetchData() {
  console.log('1. Bắt đầu fetch');
  
  const response = await fetch('https://api.example.com/data');
  console.log('3. Đã nhận response');
  
  const data = await response.json();
  console.log('4. Đã parse JSON');
  
  return data;
}

console.log('2. Gọi fetchData');
fetchData().then(() => console.log('5. Hoàn thành'));
console.log('6. Sau khi gọi');
```

**Output:**
```
2. Gọi fetchData
1. Bắt đầu fetch
6. Sau khi gọi
3. Đã nhận response
4. Đã parse JSON
5. Hoàn thành
```

### Xử lý DOM Events

```javascript
document.getElementById('btn').addEventListener('click', () => {
  console.log('Click handler');
});

setTimeout(() => {
  console.log('Timeout');
}, 0);

Promise.resolve().then(() => {
  console.log('Promise');
});

console.log('Sync code');
```

Khi click → callback click được đẩy vào **Task Queue** → chờ đến khi Call Stack trống và Microtask xong.

## Best Practices

### 1. Tránh blocking Call Stack
```javascript
// ❌ BAD - blocking
function heavyTask() {
  let i = 0;
  while (i < 1e9) i++; // treo UI
}

// ✅ GOOD - chia nhỏ bằng setTimeout hoặc Web Workers
function heavyTaskChunked(i = 0) {
  const chunk = 1e7;
  while (i < 1e9 && i < chunk) i++;
  
  if (i < 1e9) {
    setTimeout(() => heavyTaskChunked(i), 0);
  }
}
```

### 2. Không lạm dụng Microtasks
```javascript
// Tránh tạo chuỗi Microtask vô tận
```

### 3. Sử dụng queueMicrotask cho tác vụ ưu tiên cao
```javascript
queueMicrotask(() => {
  // Chạy ngay sau code đồng bộ hiện tại, trước các macrotask
});
```

### 4. Hiểu rõ thứ tự thực thi
```javascript
setTimeout(() => console.log('timeout'), 0);
process.nextTick?.(() => console.log('nextTick')); // Node.js
Promise.resolve().then(() => console.log('promise'));
```

## Troubleshooting Thường Gặp

### Callback không chạy?
- Kiểm tra Call Stack có bị blocking không
- Kiểm tra lỗi trong callback trước đó (Promise reject không catch)

### Thứ tự thực thi sai kỳ vọng?
- Nhớ Microtask luôn chạy trước Macrotask
- Trong Node.js: `process.nextTick` > Microtask > Macrotask

### UI bị treo dù dùng async?
- Có thể đang có vòng lặp đồng bộ hoặc Microtask liên tục

## Kết Luận

Event Loop là "trái tim" của cơ chế bất đồng bộ trong JavaScript. Hiểu rõ nó giúp bạn:
- Viết code async hiệu quả hơn
- Tránh các bug khó hiểu liên quan đến thứ tự thực thi
- Tối ưu hiệu năng ứng dụng (đặc biệt là UI responsiveness)
- Debug tốt hơn trong môi trường Node.js và browser

Hãy thực hành bằng cách viết các ví dụ nhỏ và quan sát thứ tự log để củng cố kiến thức!

## Tài Liệu Tham Khảo

- [MDN: Concurrency model and the event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop)
- [WHATWG Specification - Event Loops](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)
- [Node.js Event Loop Documentation](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
- Video nổi tiếng: "What the heck is the event loop anyway?" - Philip Roberts (JSConf EU)

---
Happy coding với Event Loop! 🚀 Hãy thử tạo một ví dụ thú vị và chia sẻ nhé!