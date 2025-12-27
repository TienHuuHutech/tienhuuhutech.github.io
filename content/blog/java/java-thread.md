---
title: "Thread trong Java: Lập Trình Đa Luồng Cơ Bản Đến Nâng Cao"
date: 2024-12-27T18:00:00+07:00
description: "Tìm hiểu Thread trong Java: khái niệm, cách tạo, đồng bộ hoá, thread pool và best practices trong lập trình đa luồng"
tags: ["java", "thread", "multithreading", "concurrency"]
featured_image: ""
---

Chào mừng bạn đến với bài viết về **Thread trong Java**! Đây là nền tảng quan trọng giúp ứng dụng Java xử lý **nhiều tác vụ song song**, cải thiện hiệu năng và khả năng phản hồi.

---

## Thread Là Gì?

### 🧵 Khái Niệm Thread

**Thread (luồng)** là đơn vị thực thi nhỏ nhất trong một tiến trình (Process):

- Một process có thể chứa nhiều thread
- Các thread **chia sẻ chung bộ nhớ**
- Thực thi song song (hoặc giả song song)
- Giúp chương trình chạy nhanh và mượt hơn

📌 Ví dụ thực tế:
- Trình duyệt: 1 thread load trang, 1 thread xử lý UI
- Game: render, xử lý input, network chạy song song
- Server: mỗi client request là một thread

---

## Process vs Thread

| Tiêu chí | Process | Thread |
|--------|--------|--------|
| Bộ nhớ | Riêng biệt | Chia sẻ |
| Tạo mới | Nặng | Nhẹ |
| Giao tiếp | Phức tạp | Dễ |
| Lỗi | Ít ảnh hưởng | Có thể crash toàn bộ |

---

## Cách Tạo Thread Trong Java

### Cách 1: Kế Thừa Lớp `Thread`

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread đang chạy: " + 
            Thread.currentThread().getName());
    }

    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        t1.start(); // KHÔNG gọi run()
    }
}
```
📌 Lưu ý quan trọng:
Luôn gọi start() thay vì run() để tạo thread mới.
### Cách 2: Implement Runnable (Khuyến nghị)
```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread đang chạy");
    }

    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.start();
    }
}
```
Ưu điểm:
  - Không bị giới hạn kế thừa
  - Phù hợp với thread pool
  - Thiết kế tốt hơn

 ### Cách 3: Lambda Expression (Java 8+)
 ```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread đang chạy");
    }

    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.start();
    }
}
```
### Vòng Đời Của Thread
**NEW → RUNNABLE → RUNNING → BLOCKED/WAITING → TERMINATED**
| Trạng thái | Ý nghĩa         |
| ---------- | --------------- |
| NEW        | Thread được tạo |
| RUNNABLE   | Sẵn sàng chạy   |
| RUNNING    | Đang chạy       |
| BLOCKED    | Bị khóa         |
| WAITING    | Đang chờ        |
| TERMINATED | Kết thúc        |

### Thread Sleep & Join
**Thread.sleep()**
```java
try {
    Thread.sleep(1000); // ngủ 1 giây
} catch (InterruptedException e) {
    e.printStackTrace();
}
```
**Thread.join()**
```java
Thread t = new Thread(() -> {
    System.out.println("Thread con");
});

t.start();
t.join(); // Chờ thread t kết thúc

System.out.println("Main thread tiếp tục");
```
## Vấn Đề Race Condition
Ví dụ lỗi Race Condition
```java
class Counter {
    int count = 0;

    void increment() {
        count++;
    }
}
```
**Nếu nhiều thread gọi increment() → kết quả sai.**
## Đồng Bộ Hóa (Synchronization)
Sử dụng ```synchronized```
```java
class Counter {
    int count = 0;

    synchronized void increment() {
        count++;
    }
}
```
Hoặc:
```java
synchronized (this) {
    count++;
}
```
**📌 Chỉ một thread được truy cập tại một thời điểm.**
## Volatile Keyword
```java
volatile boolean running = true;
```
- Đảm bảo đọc/ghi trực tiếp từ bộ nhớ chính

- Không đảm bảo atomicity

- Dùng cho flag đơn giản
## Thread Pool với ExecutorService
Tạo Fixed Thread Pool
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolDemo {
    public static void main(String[] args) {
        ExecutorService executor =
            Executors.newFixedThreadPool(3);

        for (int i = 1; i <= 5; i++) {
            int taskId = i;
            executor.submit(() -> {
                System.out.println(
                    "Task " + taskId +
                    " chạy bởi " +
                    Thread.currentThread().getName()
                );
            });
        }

        executor.shutdown();
    }
}
```
**Ưu điểm**
- Quản lý thread hiệu quả
- Tránh tạo quá nhiều thread
- Tối ưu hiệu năng server

## Callable & Future
```java 
import java.util.concurrent.*;

public class CallableDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService executor =
            Executors.newSingleThreadExecutor();

        Callable<Integer> task = () -> {
            Thread.sleep(1000);
            return 42;
        };

        Future<Integer> future = executor.submit(task);

        System.out.println("Kết quả: " + future.get());

        executor.shutdown();
    }
}
```
## Deadlock Trong Java
Ví dụ Deadlock
```java
synchronized (lockA) {
    synchronized (lockB) {
        // ...
    }
}
```
Thread khác:
```java
synchronized (lockB) {
    synchronized (lockA) {
        // DEADLOCK
    }
}
```
**Cách Tránh Deadlock**
- Lock theo thứ tự cố định
- Tránh nested locks
- Sử dụng `tryLock()`

## Best Practices Khi Làm Việc Với Thread
**1. Ưu tiên ExecutorService**
- Tránh tạo thread thủ công
- Dễ quản lý và scale
**2. Không Block Thread Không Cần Thiết**
- Tránh `sleep` dài
- Tránh `synchronized` quá rộng
**3. Luôn Xử Lý InterruptedException**
```java
catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```
**4. Thread An Toàn (Thread-Safe)**
- Sử dụng immutable object
- Concurrent collections
## So Sánh Thread vs Async
| Tiêu chí    | Thread    | Async      |
| ----------- | --------- | ---------- |
| Kiểm soát   | Cao       | Trung bình |
| Độ phức tạp | Cao       | Thấp       |
| Hiệu năng   | Tốt       | Rất tốt    |
| Phù hợp     | CPU-bound | IO-bound   |
## Kết Luận
Trong bài viết này, chúng ta đã tìm hiểu:

- Khái niệm Thread và Multithreading
- Cách tạo và quản lý Thread
- Đồng bộ hóa và tránh race condition
- Thread pool và ExecutorService
- Deadlock và best practices
  
📌 Thread là nền tảng cốt lõi của Java, đặc biệt quan trọng trong lập trình mạng, server và hệ thống phân tán.
## Tài Liệu Tham Khảo
- Java Concurrency in Practice
- Oracle Java Thread Documentation
- Effective Java – Item Concurrency
- `java.util.concurrent` Package
---