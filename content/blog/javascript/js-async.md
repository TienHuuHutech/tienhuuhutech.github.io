---
title: "JavaScript Async: Bất Đồng Bộ Từ Cơ Bản Đến Thực Tế"
date: 2024-12-27T18:00:00+07:00
description: "Hiểu rõ lập trình bất đồng bộ trong JavaScript với Callback, Promise và Async/Await kèm ví dụ thực tế"
tags: ["javascript", "async", "promise", "async-await", "web"]
featured_image: ""
---

## Giới Thiệu

JavaScript là ngôn ngữ **single-thread**, nhưng lại xử lý rất tốt các tác vụ **bất đồng bộ (Asynchronous)** như:
- Gọi API
- Đọc/ghi file
- Xử lý sự kiện
- Timer, animation

Bài viết này sẽ giúp bạn hiểu:
- Async là gì và tại sao cần
- Callback, Promise, Async/Await
- Best practices khi làm việc với async

---

## 1. Đồng Bộ vs Bất Đồng Bộ

### Ví dụ đồng bộ (Synchronous)

```js
console.log("Bước 1");
console.log("Bước 2");
console.log("Bước 3");
```
### Ví dụ bất đồng bộ (Asynchronous)

```js
console.log("Bắt đầu");

setTimeout(() => {
    console.log("Xử lý xong sau 2 giây");
}, 2000);

console.log("Kết thúc");
```

## 2. Callback – Cách Async Cổ Điển
### Ví dụ Callback
```js
function fetchData(callback) {
    setTimeout(() => {
        callback("Dữ liệu đã tải xong");
    }, 1000);
}

fetchData(data => {
    console.log(data);
});
```
### Nhược điểm: Callback Hell

```js
doTask1(() => {
    doTask2(() => {
        doTask3(() => {
            console.log("Hoàn thành");
        });
    });
});
```

## 3. Promise – Giải Pháp Hiện Đại Hơn
### Tạo Promise


```js
const fetchData = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve("Dữ liệu từ server");
    }, 1000);
});

```

### Sử dụng Promise

```js
fetchData
    .then(data => {
        console.log(data);
    })
    .catch(error => {
        console.error(error);
    });


```

### Promise Chain

```js
fetchUser()
    .then(user => fetchPosts(user.id))
    .then(posts => fetchComments(posts[0].id))
    .then(comments => console.log(comments))
    .catch(err => console.error(err));
```

## 4. Async / Await – Cách Viết Khuyến Nghị
### Cú pháp cơ bản
```js
async function loadData() {
    const data = await fetchData();
    console.log(data);
}

```
### Ví dụ gọi API với fetch
```js
async function getUsers() {
    try {
        const response = await fetch("https://jsonplaceholder.typicode.com/users");
        const users = await response.json();
        console.log(users);
    } catch (error) {
        console.error("Lỗi:", error);
    }
}

getUsers();

```
## 5. Chạy Song Song Với Promise.all
```js
async function loadAll() {
    const [users, posts] = await Promise.all([
        fetch("/api/users"),
        fetch("/api/posts")
    ]);

    console.log(users, posts);
}

```
## 6. Event Loop – Hiểu Bản Chất Async

**Call Stack → Web APIs → Callback Queue → Event Loop**

Ví dụ
```js
console.log("A");

setTimeout(() => {
    console.log("B");
}, 0);

console.log("C");
```
Kết quả:

```css
A
C
B
``` 
## 7. Lỗi Phổ Biến
## Quên await
```js 
const data = fetchData();
console.log(data);
```
## Đúng
```js
const data = await fetchData();
```
## Không bắt lỗi
```js
await fetchData();
```
## Đúng
```js
try {
    await fetchData();
} catch (e) {
    console.error(e);
}
```
## 8. Best Practices
- Ưu tiên async/await
- Luôn dùng try/catch
- Tránh await trong vòng lặp
- Dùng Promise.all khi cần
- Không block UI
## 9. Ứng Dụng Thực Tế
- Gọi API REST
- Xử lý form
- Realtime (WebSocket)
- Node.js backend
- React / Vue / Next.js
## So Sánh
| Cách        | Dễ đọc | Khuyến nghị |
| ----------- | ------ | ----------- |
| Callback    | ❌      | Không       |
| Promise     | ⚠      | Có          |
| Async/Await | ✔      | Rất nên     |
## Kết Luận

Async là nền tảng cốt lõi của JavaScript hiện đại, giúp xây dựng ứng dụng web hiệu quả, mượt mà và dễ bảo trì.

---